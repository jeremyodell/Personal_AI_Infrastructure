---
name: TeamDev
description: Team development workflow integrating Linear tickets with feature-dev, frontend-design, ralph-wiggum, and pr-review-toolkit. Enforces TDD and quality gates for production-ready code.
---

# TeamDev - Team Development Workflow

**Full-stack team development workflow from Linear ticket to merged PR.**

Integrates:
- **Linear MCP** - Ticket management & status sync
- **feature-dev** - Discovery, exploration, architecture (Phases 1-4)
- **frontend-design** - Production-grade UI implementation
- **ralph-wiggum** - TDD iteration loops with completion guarantees
- **pr-review-toolkit** - Automated code review

---

## Quick Start

```bash
# Work on a Linear ticket (any of these formats work)
work on ENG-123
work on ticket ENG-456
implement ENG-789
```

The workflow automatically:
1. Fetches ticket from Linear
2. Routes based on labels (pre-planned, UI, frontend)
3. Runs discovery & architecture (if needed)
4. Implements with TDD enforcement
5. Reviews code for quality
6. Runs quality gates (test, lint, typecheck, build)
7. Ships (commit, PR, Linear sync)

---

## Workflow Phases

### Phase 0: Setup & Routing
- Fetch Linear ticket
- Detect labels: `pre-planned`, `UI`, `frontend`, `design`
- Create feature branch: `feat/ENG-123-slugified-title`
- Update Linear → "In Progress"

### Phases 1-4: Discovery & Architecture (Conditional)
**Skip if ticket has "pre-planned" label**

Uses `/feature-dev` plugin:
1. **Discovery** - Understand feature requirements
2. **Exploration** - Launch code-explorer agents in parallel
3. **Clarifying Questions** - Ask ALL questions before design
4. **Architecture Design** - Launch code-architect agents, get user approval

**Stops naturally after Phase 4** - feature-dev waits for approval before implementation

### Phase 5: TDD Implementation
**Enforced with ralph-wiggum iteration loops**

#### IF UI/Frontend Label Present:
```bash
/frontend-design → Design aesthetic + generate UI code
↓
/ralph-loop → Iterate until all tests pass
  - Component rendering tests
  - User interaction tests
  - Accessibility (WCAG 2.1 AA)
  - Visual regression (optional)
```

#### ELSE (Standard Implementation):
```bash
Write failing tests FIRST (based on acceptance criteria)
↓
/ralph-loop → TDD methodology enforcement
  - Write failing test
  - Implement minimum code to pass
  - Refactor
  - Repeat until all requirements met

Completion requirements:
  - All tests passing (npm test: 0 failures)
  - Code coverage >= 80%
  - No linting errors
  - No type errors
```

**Why ralph-loop?**
- Deterministic TDD enforcement
- Self-corrects based on test failures
- Max iterations prevent infinite loops (30 for UI, 50 for backend)
- Uses `<promise>` tags for completion detection

### Phase 6: Code Review
**Uses pr-review-toolkit agents**

1. Review for: simplicity, DRY, elegance, bugs, conventions
2. Auto-fix high-confidence issues (≥90%)
3. Present medium-confidence issues (80-89%) for user decision

### Phase 7: Quality Gates
**ALL must pass (hard stop if any fail):**

```bash
npm test         # 0 failures
npm run lint     # 0 errors
npm run typecheck # 0 errors
npm run build    # successful build
```

If any gate fails → ralph-loop auto-fixes until all pass

### Phase 8: Ship
1. Commit with conventional message
2. Push feature branch
3. Create PR with `gh pr create`
4. Update Linear → "In Review"
5. Post PR link to Linear ticket

---

## Label-Based Routing

| Label | Behavior |
|-------|----------|
| `pre-planned` | Skip Phases 1-4 → Go straight to implementation |
| `UI`, `frontend`, `design` | Use frontend-design for Phase 5 |
| None | Full workflow with standard TDD |

**Combinations:**
- `pre-planned` + `UI` → Skip to frontend-design implementation
- `pre-planned` + (no UI) → Skip to standard TDD implementation

---

## Usage Examples

### Example 1: Pre-Planned Backend Feature
```
User: "work on ENG-456"
Ticket: "Add rate limiting to API endpoints"
Labels: [pre-planned, backend]

Flow:
✅ Phase 0: Detect "pre-planned" → Skip to Phase 5
✅ Phase 5: TDD with ralph-loop (tests → iterate → all pass)
✅ Phase 6: Code review → Auto-fix issues
✅ Phase 7: Quality gates → All pass
✅ Phase 8: Ship → PR + Linear sync
```

### Example 2: UI Feature (Not Pre-Planned)
```
User: "work on ENG-789"
Ticket: "Create user profile dashboard"
Labels: [UI, frontend]

Flow:
✅ Phase 0: Detect "UI" → Set USE_FRONTEND_DESIGN = true
✅ Phases 1-4: feature-dev discovery & architecture
✅ Phase 5: frontend-design + ralph-loop for tests
✅ Phase 6: Code review
✅ Phase 7: Quality gates
✅ Phase 8: Ship
```

### Example 3: Complex Feature (No Labels)
```
User: "work on ENG-111"
Ticket: "Add real-time notifications system"
Labels: []

Flow:
✅ Phase 0: No special labels → Full workflow
✅ Phases 1-4: feature-dev (discovery → architecture)
✅ Phase 5: TDD with ralph-loop
✅ Phase 6: Code review
✅ Phase 7: Quality gates
✅ Phase 8: Ship
```

---

## Command Routing

When you see:
- "work on ENG-123"
- "work on ticket ENG-456"
- "implement ENG-789"
- "start ENG-111"

**Immediately invoke:**
```markdown
[Invoke WorkOnTicket workflow with ticket_id]
```

---

## Integration with PAI Systems

| PAI System | Integration |
|------------|-------------|
| **CORE Routing** | "work on ENG-123" auto-triggers TeamDev |
| **History** | Captures all ticket work for reference |
| **Voice** | Announces phase transitions & completion |
| **Response Format** | Uses 📋 SUMMARY, 🔍 ANALYSIS, ✅ RESULTS, 🎯 COMPLETED |
| **Hooks** | Auto-sync Linear on Stop, quality gate enforcement |

---

## Configuration

See `config/workflow-config.yaml` for:
- Label → Plugin mappings
- Quality gate commands
- Ralph max iterations
- Linear state transitions

---

## Workflows

Main workflows in `Workflows/`:
- **WorkOnTicket.md** - Main orchestrator (Phases 0-8)
- **RunQualityGates.md** - Quality gate runner with auto-fix
- **ShipFeature.md** - Git + PR + Linear sync

---

## Tools

Helper tools in `Tools/`:
- **LinearHelpers.ts** - MCP wrapper functions for Linear API
- **TicketRouter.ts** - Label detection & routing logic
- **RalphOrchestrator.ts** - Ralph-loop management utilities

---

## Design Principles

1. **TDD is non-negotiable** - Tests MUST be written first
2. **Ralph enforces completion** - Iterate until ALL tests pass
3. **Plugin composition** - Use official plugins, don't reimplement
4. **Linear as source of truth** - All status flows through Linear
5. **Quality gates are hard stops** - Cannot ship if gates fail
6. **Label-based routing** - Smart detection of ticket type
7. **PAI integration** - Memory, voice, routing, hooks all connected

---

## Installation

See `INSTALL.md` in pack root for setup instructions.

## Verification

See `VERIFY.md` in pack root for verification checklist.
