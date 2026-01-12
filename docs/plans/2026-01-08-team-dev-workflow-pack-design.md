# Team Development Workflow PAI Pack - Implementation Plan

**Created:** 2026-01-08
**Goal:** Create a PAI pack that integrates Linear, feature-dev, frontend-design, and other Anthropic plugins into a deterministic, TDD-enforced team workflow

---

## Context

I have an existing `team-workflow` Claude Code plugin at:
- `/home/jeremyodell/.claude/plugins/marketplaces/claude-assistant-marketplace/plugins/team-workflow/`

**What it currently does:**
- Phase-gated workflow: Setup → Brainstorm → Plan → Execute → Quality → Ship
- Linear MCP integration for ticket sync
- Uses superpowers plugin for brainstorming/planning
- TDD enforcement
- Parallel subagent orchestration via `/team:feature`

**What I want to change:**
- Convert to PAI pack (Option 2 - Full PAI Pack)
- Replace brainstorm/plan with `feature-dev` plugin (Phases 1-4)
- Use `frontend-design` plugin for UI implementation
- Use `ralph-wiggum` plugin for TDD iteration loops
- Use `pr-review-toolkit` code-review agents
- Enforce TDD: write tests FIRST, then iterate with ralph until all pass

---

## The New Workflow Design

### Phase Decision Tree

```
START: User says "work on ENG-123"
    ↓
Fetch Linear ticket
    ↓
Check for "pre-planned" label?
    ├─ YES → Skip to Implementation (Phase 5)
    └─ NO → Continue
         ↓
    Check for "UI" or "frontend" label?
         ├─ YES → Run feature-dev Phases 1-4, then frontend-design for Phase 5
         └─ NO → Run full feature-dev Phases 1-4, then standard TDD implementation
              ↓
         IMPLEMENTATION PHASE (With ralph-loop for TDD)
              ↓
         Code Review → Quality Gates → Ship → Sync Linear
```

---

## Detailed Phase Specifications

### Phase 0: Setup & Routing

**Actions:**
1. Fetch Linear ticket: `mcp__linear__get_issue(id: "$ISSUE_ID")`
2. Extract: `title`, `description`, `labels[]`, `assignee`, `priority`
3. Check labels array:
   - `pre-planned` → Set `SKIP_PHASES_1_TO_4 = true`
   - `UI` or `frontend` or `design` → Set `USE_FRONTEND_DESIGN = true`
4. Create feature branch: `feat/$ISSUE_ID-$SLUGIFIED_TITLE`
5. Update Linear status → "In Progress"

**Output:**
```
✅ Phase 0 Complete - Ticket Analysis
- Issue: ENG-123 - Add user dashboard
- Labels: [UI, pre-planned]
- Routing: SKIP to Implementation with frontend-design
- Branch: feat/ENG-123-add-user-dashboard
```

---

### Phases 1-4: Discovery & Architecture (Conditional)

**Skip if ticket has "pre-planned" label**

**If NOT pre-planned:**

Invoke `/feature-dev` which runs:

**Phase 1: Discovery**
- Understand what needs to be built
- Clarify feature request
- Summarize and confirm

**Phase 2: Codebase Exploration**
- Launch 2-3 `code-explorer` agents in parallel
- Explore similar features, architecture, patterns
- Read all identified files
- Present comprehensive findings

**Phase 3: Clarifying Questions**
- Identify underspecified aspects
- Ask user ALL questions in organized list
- Wait for answers before proceeding

**Phase 4: Architecture Design**
- Launch 2-3 `code-architect` agents with different focuses:
  - Minimal changes
  - Clean architecture
  - Pragmatic balance
- Present approaches with trade-offs
- Get user approval on chosen approach

**STOP BEFORE Phase 5 (Implementation)**

**Post to Linear:**
```
mcp__linear__create_comment(
  issueId: "$ISSUE_ID",
  body: "## Architecture Design Complete\n\n$ARCHITECTURE_SUMMARY"
)
```

---

### Phase 5: Implementation (Modified for TDD + Ralph)

#### Option A: Frontend Implementation (If UI Label Present)

1. **Invoke frontend-design skill** with architecture from Phase 4
2. **Design thinking:**
   - Choose bold aesthetic direction (brutalist, editorial, minimal, etc.)
   - Define typography, colors, motion, spatial composition
3. **Generate production-grade UI code**
4. **Write tests FIRST:**
   - Component rendering tests
   - User interaction tests
   - Accessibility tests (WCAG 2.1 AA)
   - Visual regression (optional)

5. **Enter ralph-loop:**
```bash
/ralph-loop "Implement the frontend design based on architecture.

REQUIREMENTS:
- All tests must pass
- No console errors
- Meets accessibility standards (WCAG 2.1 AA)
- Matches aesthetic vision

Output <promise>UI_IMPLEMENTATION_COMPLETE</promise> when:
- All tests passing (npm test shows 0 failures)
- Visual matches design intent
- Code follows project conventions
" --completion-promise "UI_IMPLEMENTATION_COMPLETE" --max-iterations 30
```

#### Option B: Standard Implementation (Non-UI)

1. **Write failing tests FIRST** (based on acceptance criteria)
2. **Enter ralph-loop for TDD:**

```bash
/ralph-loop "Implement feature following TDD methodology.

ACCEPTANCE CRITERIA:
$ACCEPTANCE_CRITERIA_FROM_ARCHITECTURE

TDD RULES:
1. Write failing test first
2. Implement minimum code to pass
3. Refactor for quality
4. Repeat for next requirement

COMPLETION REQUIREMENTS:
- All acceptance criteria met
- All tests passing (npm test: 0 failures)
- Code coverage >= 80%
- No linting errors (npm run lint)
- No type errors (npm run typecheck)

Output <promise>IMPLEMENTATION_COMPLETE</promise> when ALL requirements met.
" --completion-promise "IMPLEMENTATION_COMPLETE" --max-iterations 50
```

**Why Ralph for TDD:**
- Iterates until ALL tests pass
- Self-corrects based on failures
- Max iterations prevents infinite loops
- Sensible defaults: 30 for UI, 50 for backend

---

### Phase 6: Code Review

**After ralph-loop completes:**

1. **Use pr-review-toolkit code-review agent:**
   - Review for simplicity, DRY, elegance
   - Check for bugs and functional correctness
   - Verify project conventions
   - Focus on high-confidence issues (≥80%)

2. **Auto-fix high-confidence issues** (≥90%)

3. **Present medium-confidence issues** (80-89%) to user for decision

**Output:**
```
✅ Code Review Complete
- High-confidence issues: 3 found, 3 fixed
- Medium-confidence: 2 found, awaiting user decision
```

---

### Phase 7: Quality Gates

**Run ALL gates (must ALL pass):**

```bash
# 1. Tests
npm test
# Expected: 0 failures

# 2. Linting
npm run lint
# Expected: 0 errors

# 3. Type checking
npm run typecheck
# Expected: 0 errors

# 4. Build
npm run build
# Expected: successful build
```

**If ANY gate fails:**
```
❌ QUALITY GATE FAILED: $GATE_NAME
$ERROR_OUTPUT

BLOCKED: Cannot proceed to ship.
```

**Auto-fix with ralph-loop:**
```bash
/ralph-loop "Fix quality gate failures:
$FAILURES

Output <promise>GATES_PASSING</promise> when all gates pass.
" --completion-promise "GATES_PASSING" --max-iterations 20
```

---

### Phase 8: Ship

1. **Commit with conventional message:**
```bash
git add .
git commit -m "feat(ENG-123): $TITLE

$IMPLEMENTATION_SUMMARY

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

2. **Push branch:**
```bash
git push origin feat/ENG-123-$SLUG
```

3. **Create PR:**
```bash
gh pr create \
  --title "feat(ENG-123): $TITLE" \
  --body "$PR_BODY" \
  --base main
```

4. **Update Linear → "In Review"**

5. **Post PR link to Linear**

---

## PAI Pack Structure

**Location:** `/home/jeremyodell/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/`

```
team-dev-workflow/
├── README.md           # Architecture overview (this document)
├── INSTALL.md          # Step-by-step installation
├── VERIFY.md           # Verification checklist
└── src/
    ├── skills/
    │   └── TeamDev/
    │       ├── SKILL.md              # Main skill routing
    │       ├── Workflows/
    │       │   ├── WorkOnTicket.md       # Main orchestrator
    │       │   ├── RunQualityGates.md    # Quality gate runner
    │       │   └── ShipFeature.md        # Git + Linear sync
    │       └── Tools/
    │           ├── LinearHelpers.ts      # Linear MCP wrappers
    │           ├── TicketRouter.ts       # Label-based routing
    │           └── RalphOrchestrator.ts  # Ralph-loop management
    ├── hooks/
    │   ├── linear-sync-on-complete.ts    # Auto-sync on Stop
    │   └── quality-gate-blocker.ts       # Block ship if gates fail
    └── config/
        └── workflow-config.yaml          # Label→Plugin mappings
```

---

## Plugin Integration Details

### Required Plugins (Already Installed)
- `feature-dev@claude-plugins-official` - Discovery & architecture (Phases 1-4)
- `frontend-design@claude-plugins-official` - UI implementation
- `ralph-wiggum@claude-plugins-official` - TDD iteration loops
- `pr-review-toolkit@claude-plugins-official` - Code review agents

### Linear MCP Configuration
```json
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.linear.app/mcp"]
    }
  }
}
```

### PAI Integration Points

| PAI System | How It Integrates |
|------------|------------------|
| **CORE Routing** | "work on ENG-123" auto-triggers TeamDev skill |
| **History System** | Captures ALL ticket work for future reference |
| **Voice System** | Announces ticket phases and completion |
| **Response Format** | 📋 SUMMARY, 🔍 ANALYSIS, ✅ RESULTS, 🎯 COMPLETED |
| **Hooks** | Auto-sync Linear on Stop, capture learnings |

---

## Configuration Answers

1. **Other plugins needed:**
   - `pr-review-toolkit` (code review)
   - `ralph-wiggum` (iteration loops)
   - Linear MCP integration (already configured)

2. **Quality gates:**
   - Use standard npm scripts: `test`, `lint`, `typecheck`, `build`
   - All must pass (0 errors/failures)

3. **Ralph max iterations:**
   - UI: 30 iterations
   - Backend: 50 iterations
   - Quality fixes: 20 iterations

4. **Linear state names:**
   - Use default Linear states: "In Progress", "In Review", etc.

5. **Future labels:**
   - Will add label for story creation from ideas
   - For now, handle: `pre-planned`, `UI`, `frontend`, `design`

---

## Example Flows

### Example 1: Pre-Planned Backend Feature
```
User: "work on ENG-456"
Ticket: "Add rate limiting to API endpoints"
Labels: [pre-planned, backend]

Flow:
Phase 0: Detect "pre-planned" → Skip to Phase 5
Phase 5: TDD with ralph-loop (write tests → iterate → all pass)
Phase 6: Code review → Auto-fix
Phase 7: Quality gates → All pass
Phase 8: Ship → PR + Linear sync
```

### Example 2: UI Feature (Not Pre-Planned)
```
User: "work on ENG-789"
Ticket: "Create user profile dashboard"
Labels: [UI, frontend]

Flow:
Phase 0: Detect "UI" → Set USE_FRONTEND_DESIGN = true
Phases 1-4: Feature-dev discovery & architecture
Phase 5: Frontend-design + ralph-loop for tests
Phase 6: Code review
Phase 7: Quality gates
Phase 8: Ship
```

### Example 3: Complex Feature (No Labels)
```
User: "work on ENG-111"
Ticket: "Add real-time notifications system"
Labels: []

Flow:
Phase 0: No special labels → Full workflow
Phases 1-4: Feature-dev discovery & architecture
Phase 5: TDD with ralph-loop
Phase 6: Code review
Phase 7: Quality gates
Phase 8: Ship
```

---

## Implementation Tasks

**To build this pack, you need to:**

1. **Create pack structure** following PAI v2.0 directory format
2. **Write SKILL.md** with routing logic for TeamDev
3. **Create WorkOnTicket.md** workflow that orchestrates all phases
4. **Build helper tools:**
   - `LinearHelpers.ts` - MCP wrapper functions
   - `TicketRouter.ts` - Label detection & routing logic
   - `RalphOrchestrator.ts` - Ralph-loop invocation wrapper
5. **Create hooks:**
   - `linear-sync-on-complete.ts` - Auto-sync on Stop event
   - `quality-gate-blocker.ts` - Prevent ship if gates fail
6. **Write INSTALL.md** with installation steps
7. **Write VERIFY.md** with verification checklist
8. **Test end-to-end** with real Linear tickets

---

## Key Design Principles

1. **TDD is non-negotiable** - Tests MUST be written first
2. **Ralph enforces completion** - Iterate until ALL tests pass
3. **Plugin composition** - Use official plugins, don't reimplement
4. **Linear as source of truth** - All status flows through Linear
5. **PAI integration** - Memory, voice, routing, hooks all connected
6. **Quality gates are hard stops** - Cannot ship if gates fail
7. **Label-based routing** - Smart detection of ticket type

---

## Next Steps

Run this prompt in a fresh session:

```
I need to build the team-dev-workflow PAI pack based on the design in:
/home/jeremyodell/dev/projects/personal-ai/PAI/docs/plans/2026-01-08-team-dev-workflow-pack-design.md

Please read that file and help me implement it as a complete PAI pack following the v2.0 directory structure.

Start by creating the pack structure and SKILL.md, then we'll build the workflows and tools.
```

---

**End of Design Document**
