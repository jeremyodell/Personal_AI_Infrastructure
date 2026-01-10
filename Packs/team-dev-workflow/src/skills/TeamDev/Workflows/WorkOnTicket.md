---
name: WorkOnTicket
description: Main orchestrator workflow for team development - handles all phases from Linear ticket to merged PR
type: workflow
---

# WorkOnTicket Workflow

**End-to-end workflow: Linear ticket → Production-ready PR**

**Input:** `ticket_id` (e.g., "ENG-123")

**Output:** Merged PR with Linear ticket updated to "In Review"

---

## Workflow Overview

```
Phase 0: Setup & Routing
   ↓
Phases 1-4: Discovery & Architecture (conditional - skip if pre-planned)
   ↓
Phase 5: TDD Implementation (conditional routing: UI → frontend-design, else → standard TDD)
   ↓
Phase 6: Code Review (pr-review-toolkit)
   ↓
Phase 7: Quality Gates (test, lint, typecheck, build - ALL must pass)
   ↓
Phase 8: Ship (commit, PR, Linear sync)
```

---

## Phase 0: Setup & Routing

**Goal:** Fetch ticket, detect routing, create branch, update Linear status

### Actions

1. **Fetch Linear Ticket**
```typescript
const issue = await mcp__linear__get_issue({
  id: ticket_id,
  includeRelations: false
});

// Extract data
const {
  id,
  identifier,       // "ENG-123"
  title,
  description,
  labels,           // Array of label objects
  assignee,
  priority,
  state
} = issue;
```

2. **Extract Label Names**
```typescript
const labelNames = labels.map(label => label.name.toLowerCase());
```

3. **Routing Logic**
```typescript
// Check for special routing labels
const isPrePlanned = labelNames.includes('pre-planned');
const isUIFeature = labelNames.some(label =>
  ['ui', 'frontend', 'design'].includes(label)
);

// Set routing flags
let SKIP_PHASES_1_TO_4 = isPrePlanned;
let USE_FRONTEND_DESIGN = isUIFeature;
```

4. **Create Feature Branch with Worktree**

Read worktree configuration:
```bash
# Load config (pseudocode - actual implementation reads YAML)
WORKTREE_ENABLED=$(grep "enabled: true" workflow-config.yaml | grep worktree)
WORKTREE_BASE_PATH=$(grep "base_path:" workflow-config.yaml | awk '{print $2}' | tr -d '"')
```

If worktrees enabled:
```bash
# Slugify title (lowercase, hyphens, alphanumeric only)
SLUG=$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//' | sed 's/-$//')

# Store original repo path
ORIGINAL_REPO_PATH=$(pwd)

# Create worktree path
WORKTREE_PATH="$WORKTREE_BASE_PATH/feat-$IDENTIFIER-$SLUG"

# Check for existing worktree
if git worktree list | grep -q "$WORKTREE_PATH"; then
  echo "⚠️  Worktree already exists for $IDENTIFIER"
  echo ""
  echo "Options:"
  echo "  1) Resume work in existing worktree"
  echo "  2) Remove and recreate (loses uncommitted changes)"
  echo "  3) Cancel"
  # Prompt user for choice (use AskUserQuestion tool)
  # If option 1: cd to existing worktree
  # If option 2: git worktree remove + recreate
  # If option 3: exit
fi

# Check for lock file (concurrent agent detection)
LOCK_PATH=".worktrees/.locks/$IDENTIFIER.lock"
if [ -f "$LOCK_PATH" ]; then
  echo "❌ ERROR: Another agent is already working on $IDENTIFIER"
  echo "Lock file: $LOCK_PATH"
  exit 1
fi

# Create lock file
mkdir -p ".worktrees/.locks"
echo "$WORKTREE_PATH" > "$LOCK_PATH"
trap "rm -f $LOCK_PATH" EXIT

# Create worktree with new branch
if git worktree add "$WORKTREE_PATH" -b "feat/$IDENTIFIER-$SLUG"; then
  cd "$WORKTREE_PATH"

  # Track worktree in workflow state
  echo "WORKTREE_PATH=$WORKTREE_PATH" >> .workflow-state
  echo "ORIGINAL_REPO_PATH=$ORIGINAL_REPO_PATH" >> .workflow-state
  echo "KEEP_WORKTREE=false" >> .workflow-state

  echo "✅ Worktree created at $WORKTREE_PATH"
else
  # Fallback to main repo workflow
  echo "❌ Failed to create worktree"
  echo "Falling back to main repo workflow"
  git checkout -b "feat/$IDENTIFIER-$SLUG"
  echo "WORKTREE_ENABLED=false" >> .workflow-state
fi
```

If worktrees disabled:
```bash
# Traditional branch checkout
SLUG=$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//' | sed 's/-$//')
git checkout -b "feat/$IDENTIFIER-$SLUG"
echo "WORKTREE_ENABLED=false" >> .workflow-state
```

**Orphaned worktree detection:**
```bash
# Warn if too many worktrees exist
if [ "$WORKTREE_ENABLED" = "true" ]; then
  WORKTREE_COUNT=$(git worktree list | grep -c "$WORKTREE_BASE_PATH/")
  WARN_THRESHOLD=3  # From config

  if [ "$WORKTREE_COUNT" -gt "$WARN_THRESHOLD" ]; then
    echo "⚠️  You have $WORKTREE_COUNT active worktrees."
    echo "Run '/cleanup-worktree all' to remove merged ones."
  fi
fi
```

5. **Update Linear Status**
```typescript
await mcp__linear__update_issue({
  id: ticket_id,
  state: "In Progress"
});
```

### Output

```markdown
✅ Phase 0 Complete - Ticket Analysis

📋 SUMMARY:
- Issue: ENG-123 - Add user dashboard
- Labels: [UI, pre-planned]
- Routing: SKIP to Implementation with frontend-design
- Branch: feat/ENG-123-add-user-dashboard

🔍 ROUTING DECISIONS:
- SKIP_PHASES_1_TO_4: true (pre-planned label detected)
- USE_FRONTEND_DESIGN: true (UI label detected)

📁 CAPTURE: Starting work on $IDENTIFIER - $TITLE
```

---

## Phases 1-4: Discovery & Architecture

**Condition:** Skip if `SKIP_PHASES_1_TO_4 === true`

### If NOT Pre-Planned:

**Invoke feature-dev skill:**
```typescript
// This runs feature-dev Phases 1-4 and stops before implementation
await Skill({
  skill: "feature-dev:feature-dev",
  args: `Implement Linear ticket ${identifier}: ${title}

TICKET DESCRIPTION:
${description}

ACCEPTANCE CRITERIA:
[Extract or derive from ticket description]

CONSTRAINTS:
- Follow project conventions in CLAUDE.md
- Write comprehensive tests
- Ensure accessibility (if UI)
- Consider performance implications
`
});
```

**feature-dev will execute:**
1. **Phase 1: Discovery** - Clarify requirements
2. **Phase 2: Codebase Exploration** - Launch 2-3 code-explorer agents
3. **Phase 3: Clarifying Questions** - Ask ALL questions before design
4. **Phase 4: Architecture Design** - Launch 2-3 code-architect agents, present options

**feature-dev stops at Phase 4** - waits for user approval before Phase 5

### After Architecture is Approved:

**Post architecture to Linear:**
```typescript
await mcp__linear__create_comment({
  issueId: ticket_id,
  body: `## Architecture Design Complete

${ARCHITECTURE_SUMMARY}

### Approach Chosen:
${CHOSEN_APPROACH}

### Files to Modify:
${FILES_LIST}

### Implementation Plan:
${IMPLEMENTATION_PHASES}

Ready to proceed with implementation.
`
});
```

### Output

```markdown
✅ Phases 1-4 Complete - Architecture Designed

📋 SUMMARY: Architecture approved for ${identifier}

🔍 ANALYSIS:
- Approach: ${CHOSEN_APPROACH}
- Files to modify: ${FILE_COUNT}
- Estimated complexity: ${COMPLEXITY}

📁 CAPTURE: Architecture for $IDENTIFIER approved - ${APPROACH_NAME}
```

---

## Phase 5: TDD Implementation

**Goal:** Implement feature with TDD enforcement using ralph-wiggum

### Route A: UI/Frontend Implementation

**Condition:** `USE_FRONTEND_DESIGN === true`

**1. Invoke frontend-design skill:**
```typescript
await Skill({
  skill: "frontend-design:frontend-design",
  args: `Design and implement UI for: ${title}

ARCHITECTURE CONTEXT:
${ARCHITECTURE_FROM_PHASE_4}

DESIGN REQUIREMENTS:
- Choose bold aesthetic direction (brutalist, editorial, minimal, etc.)
- Define typography, colors, motion, spatial composition
- Generate production-grade UI code
- Ensure accessibility (WCAG 2.1 AA)

TECHNICAL REQUIREMENTS:
- Follow project UI patterns
- Use existing component library
- Implement responsive design
- Write component tests
`
});
```

**2. Write Tests FIRST (before UI code):**
```typescript
// Component rendering tests
// User interaction tests
// Accessibility tests (WCAG 2.1 AA)
// Visual regression (optional)
```

**3. Enter ralph-loop for TDD enforcement:**
```bash
/ralph-loop "Implement the frontend design based on architecture.

REQUIREMENTS:
- All tests must pass (npm test: 0 failures)
- No console errors in development
- Meets accessibility standards (WCAG 2.1 AA)
- Matches aesthetic vision
- Follows project conventions

Output <promise>UI_IMPLEMENTATION_COMPLETE</promise> when:
- ✅ All tests passing
- ✅ Visual matches design intent
- ✅ Code follows project conventions
- ✅ No accessibility violations
" --completion-promise "UI_IMPLEMENTATION_COMPLETE" --max-iterations 30
```

### Route B: Standard Implementation

**Condition:** `USE_FRONTEND_DESIGN === false`

**1. Write Failing Tests FIRST:**
```typescript
// Based on acceptance criteria from ticket/architecture
// Write comprehensive test suite covering:
// - Happy path
// - Edge cases
// - Error handling
// - Integration points
```

**2. Enter ralph-loop for TDD:**
```bash
/ralph-loop "Implement feature following TDD methodology.

TICKET: ${identifier} - ${title}

ACCEPTANCE CRITERIA:
${ACCEPTANCE_CRITERIA_FROM_ARCHITECTURE}

TDD RULES:
1. Write failing test first
2. Implement minimum code to pass test
3. Refactor for quality and maintainability
4. Repeat for next requirement

COMPLETION REQUIREMENTS:
- ✅ All acceptance criteria met
- ✅ All tests passing (npm test: 0 failures)
- ✅ Code coverage >= 80%
- ✅ No linting errors (npm run lint: 0 errors)
- ✅ No type errors (npm run typecheck: 0 errors)
- ✅ Follows project conventions

Output <promise>IMPLEMENTATION_COMPLETE</promise> when ALL requirements met.
" --completion-promise "IMPLEMENTATION_COMPLETE" --max-iterations 50
```

### Why Ralph-Loop for TDD?

- **Deterministic enforcement** - Iterates until ALL tests pass
- **Self-correction** - Learns from test failures
- **Max iterations** - Prevents infinite loops (30 UI, 50 backend)
- **Promise detection** - Uses `<promise>` tags for completion

### Output

```markdown
✅ Phase 5 Complete - Implementation

📋 SUMMARY: Feature implemented with TDD for ${identifier}

🔍 ANALYSIS:
- Tests written: ${TEST_COUNT}
- All tests passing: ✅
- Code coverage: ${COVERAGE}%
- Ralph iterations used: ${ITERATIONS}

⚡ ACTIONS:
- Wrote ${TEST_COUNT} tests covering all acceptance criteria
- Implemented feature with TDD discipline
- Refactored for maintainability
- Verified all tests pass

📁 CAPTURE: Implementation complete for $IDENTIFIER - all tests passing
```

---

## Phase 6: Code Review

**Goal:** Review code for quality, bugs, and conventions using pr-review-toolkit

### Actions

**1. Launch code-reviewer agent:**
```typescript
await Task({
  subagent_type: "pr-review-toolkit:code-reviewer",
  description: "Review implementation code",
  prompt: `Review the code changes for Linear ticket ${identifier}.

Focus on unstaged changes (git diff).

Review for:
- Simplicity, DRY principles, elegance
- Bugs and functional correctness
- Project conventions (CLAUDE.md)
- Security vulnerabilities
- Performance issues

Use confidence-based filtering:
- Only report issues with confidence >= 80%
- Auto-fix issues with confidence >= 90%
- Present issues 80-89% for user decision
`
});
```

**2. Process Review Findings:**
```typescript
// High-confidence issues (≥90%): Auto-fix
if (highConfidenceIssues.length > 0) {
  console.log(`Auto-fixing ${highConfidenceIssues.length} high-confidence issues...`);
  // Apply fixes
}

// Medium-confidence issues (80-89%): Ask user
if (mediumConfidenceIssues.length > 0) {
  console.log(`Found ${mediumConfidenceIssues.length} medium-confidence issues:`);
  // Present to user for decision
}
```

### Output

```markdown
✅ Phase 6 Complete - Code Review

📋 SUMMARY: Code review complete for ${identifier}

🔍 ANALYSIS:
- High-confidence issues: ${HIGH_COUNT} found, ${HIGH_COUNT} fixed
- Medium-confidence issues: ${MEDIUM_COUNT} found, awaiting user decision
- Code quality rating: ${QUALITY_SCORE}/10

⚡ ACTIONS:
- Auto-fixed ${HIGH_COUNT} high-confidence issues
- Presented ${MEDIUM_COUNT} issues for user review

📁 CAPTURE: Code review complete for $IDENTIFIER - ${ISSUE_COUNT} issues addressed
```

---

## Phase 7: Quality Gates

**Goal:** Run ALL quality gates - all must pass before shipping

### Actions

**Invoke RunQualityGates workflow:**
```typescript
await invokeWorkflow({
  workflow: "RunQualityGates",
  params: {
    ticket_id: ticket_id,
    auto_fix: true  // Use ralph-loop to auto-fix failures
  }
});
```

**RunQualityGates executes:**
```bash
# Gate 1: Tests
npm test
# Expected: 0 failures

# Gate 2: Linting
npm run lint
# Expected: 0 errors

# Gate 3: Type Checking
npm run typecheck
# Expected: 0 errors

# Gate 4: Build
npm run build
# Expected: successful build
```

**If ANY gate fails:**
- Block progression to Phase 8
- Use ralph-loop to auto-fix (max 20 iterations)
- Re-run all gates until all pass

### Output

```markdown
✅ Phase 7 Complete - Quality Gates

📋 SUMMARY: All quality gates passed for ${identifier}

🔍 GATE RESULTS:
- ✅ Tests: 0 failures
- ✅ Linting: 0 errors
- ✅ Type checking: 0 errors
- ✅ Build: successful

📁 CAPTURE: Quality gates passed for $IDENTIFIER - ready to ship
```

---

## Phase 8: Ship

**Goal:** Commit, create PR, update Linear

### Actions

**Invoke ShipFeature workflow:**
```typescript
await invokeWorkflow({
  workflow: "ShipFeature",
  params: {
    ticket_id: ticket_id,
    identifier: identifier,
    title: title,
    branch: `feat/${identifier}-${slug}`,
    implementation_summary: IMPLEMENTATION_SUMMARY
  }
});
```

**ShipFeature executes:**
1. Commit with conventional message
2. Push branch to remote
3. Create PR with `gh pr create`
4. Update Linear → "In Review"
5. Post PR link to Linear

### Output

```markdown
✅ Phase 8 Complete - Shipped

📋 SUMMARY: Feature shipped for ${identifier}

🔍 SHIP DETAILS:
- Branch: feat/${identifier}-${slug}
- Commit: ${COMMIT_SHA}
- PR: ${PR_URL}
- Linear status: In Review

⚡ ACTIONS:
- Committed changes with conventional message
- Pushed branch to origin
- Created PR: ${PR_URL}
- Updated Linear ticket to "In Review"
- Posted PR link to Linear

✅ RESULTS:
Feature ${identifier} ready for team review

📁 CAPTURE: Shipped $IDENTIFIER - PR created at ${PR_URL}

🎯 COMPLETED: ${identifier} implemented, reviewed, and shipped
```

---

## Error Handling

### If Linear Fetch Fails:
```markdown
❌ ERROR: Could not fetch Linear ticket ${ticket_id}

Possible causes:
- Ticket ID incorrect (check format: ENG-123)
- Linear API not configured
- Network connectivity issue

Please verify ticket ID and try again.
```

### If Quality Gates Fail After Max Iterations:
```markdown
❌ QUALITY GATE FAILURE: Cannot proceed to ship

Failed gates:
${FAILED_GATES}

Ralph-loop reached max iterations (20) without fixing all issues.

NEXT STEPS:
1. Review gate failures above
2. Manually fix remaining issues
3. Re-run workflow from Phase 7
```

### If Ralph-Loop Reaches Max Iterations:
```markdown
⚠️ WARNING: Ralph-loop reached max iterations

Phase: ${PHASE_NAME}
Iterations used: ${MAX_ITERATIONS}

Current status:
${CURRENT_STATUS}

NEXT STEPS:
1. Review implementation progress
2. Decide: continue manually, adjust requirements, or increase iterations
```

---

## Workflow State

**Track state throughout workflow:**
```typescript
const workflowState = {
  ticket_id: string,
  identifier: string,
  title: string,
  labels: string[],

  // Routing flags
  skip_phases_1_4: boolean,
  use_frontend_design: boolean,

  // Architecture (from Phase 4)
  architecture_summary: string,
  chosen_approach: string,

  // Implementation (from Phase 5)
  tests_written: number,
  tests_passing: boolean,
  coverage: number,

  // Review (from Phase 6)
  issues_found: number,
  issues_fixed: number,

  // Quality (from Phase 7)
  gates_passed: boolean,

  // Ship (from Phase 8)
  pr_url: string,
  commit_sha: string
};
```

---

## Integration Points

### Linear MCP:
- `mcp__linear__get_issue()` - Fetch ticket
- `mcp__linear__update_issue()` - Update status
- `mcp__linear__create_comment()` - Post updates

### Anthropic Plugins:
- `feature-dev:feature-dev` - Discovery & architecture (Phases 1-4)
- `frontend-design:frontend-design` - UI implementation
- `ralph-wiggum` (via `/ralph-loop`) - TDD enforcement
- `pr-review-toolkit:code-reviewer` - Code review

### PAI Systems:
- **TodoWrite** - Track phase progress
- **History** - Capture workflow context
- **Voice** - Announce phase transitions
- **Hooks** - Linear sync, quality gate enforcement

---

## Usage

**Invoke from SKILL.md:**
```markdown
When user says "work on ENG-123", invoke this workflow with ticket_id="ENG-123"
```

**Direct invocation:**
```typescript
await invokeWorkflow({
  workflow: "WorkOnTicket",
  params: {
    ticket_id: "ENG-123"
  }
});
```
