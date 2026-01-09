# Team Development Workflow

**End-to-end team development workflow: Linear ticket → Production-ready PR**

A PAI v2.0 pack that integrates Linear, feature-dev, frontend-design, ralph-wiggum, and pr-review-toolkit into a deterministic, TDD-enforced workflow.

---

## Overview

This pack provides an **8-phase workflow** from Linear ticket to merged PR:

```
Phase 0: Setup & Routing → Fetch ticket, detect labels, create branch
Phases 1-4: Discovery & Architecture → feature-dev exploration & design
Phase 5: TDD Implementation → ralph-wiggum enforced test-first development
Phase 6: Code Review → pr-review-toolkit automated review
Phase 7: Quality Gates → test, lint, typecheck, build (all must pass)
Phase 8: Ship → commit, PR, Linear sync
```

**Key Features:**
- ✅ **TDD Enforced** - Tests written first, ralph-wiggum iterates until all pass
- ✅ **Smart Routing** - Label-based workflow selection (pre-planned, UI, etc.)
- ✅ **Quality Gates** - Hard stop if tests/lint/typecheck/build fail
- ✅ **Linear Integration** - Auto-sync status, post updates, track progress
- ✅ **Plugin Composition** - Leverages official Anthropic plugins
- ✅ **PAI Integration** - Works with history, voice, hooks

---

## Quick Start

```bash
# 1. Install pack
./INSTALL.md  # Follow installation steps

# 2. Work on a Linear ticket
work on ENG-123

# 3. Follow the workflow
# - Phases 1-4: Discovery & Architecture (if not pre-planned)
# - Phase 5: TDD Implementation with ralph-loop
# - Phase 6: Code Review
# - Phase 7: Quality Gates
# - Phase 8: Ship (PR + Linear sync)
```

---

## Workflow Phases

### Phase 0: Setup & Routing
Fetches Linear ticket, analyzes labels, creates branch, updates status.

**Routing Logic:**
| Label | Effect |
|-------|--------|
| `pre-planned` | Skip to Phase 5 (implementation) |
| `UI`, `frontend`, `design` | Use frontend-design for Phase 5 |
| `spike`, `research` | Stop after Phase 4 (no implementation) |

### Phases 1-4: Discovery & Architecture
Uses **feature-dev** plugin:
1. **Discovery** - Clarify requirements
2. **Exploration** - Launch code-explorer agents
3. **Questions** - Ask ALL clarifying questions
4. **Architecture** - Launch code-architect agents, present options

**Stops naturally** after Phase 4 - waits for user approval before implementation.

### Phase 5: TDD Implementation
**Enforced with ralph-wiggum iteration loops:**

#### IF UI Label:
```bash
1. /frontend-design → Generate UI code
2. Write tests FIRST (component, interaction, accessibility)
3. /ralph-loop → Iterate until all tests pass (max 30 iterations)
```

#### ELSE (Standard):
```bash
1. Write failing tests FIRST (based on acceptance criteria)
2. /ralph-loop → TDD methodology enforcement (max 50 iterations)
   - Write failing test
   - Implement minimum code to pass
   - Refactor
   - Repeat
```

### Phase 6: Code Review
Uses **pr-review-toolkit**:
- Review for simplicity, DRY, elegance, bugs, conventions
- Auto-fix high-confidence issues (≥90%)
- Present medium-confidence issues (80-89%) for user decision

### Phase 7: Quality Gates
**ALL must pass (hard stop if any fail):**
```bash
npm test         # 0 failures
npm run lint     # 0 errors
npm run typecheck # 0 errors
npm run build    # successful
```

If gates fail → ralph-loop auto-fixes (max 20 iterations)

### Phase 8: Ship
1. Commit with conventional message
2. Push branch to origin
3. Create PR with `gh pr create`
4. Update Linear → "In Review"
5. Post PR link to Linear

---

## Label-Based Routing

```
START: "work on ENG-123"
   ↓
Fetch Linear ticket
   ↓
Check labels
   ├─ "pre-planned" → Skip to Phase 5
   ├─ "UI/frontend" → Phase 5 uses frontend-design
   ├─ "spike/research" → Stop after Phase 4
   └─ None → Full workflow (0-8)
```

---

## Architecture

```
PAI/Packs/team-dev-workflow/
├── README.md           # This file
├── INSTALL.md          # Installation guide
├── VERIFY.md           # Verification checklist
└── src/
    ├── skills/
    │   └── TeamDev/
    │       ├── SKILL.md              # Main skill (routing)
    │       ├── Workflows/
    │       │   ├── WorkOnTicket.md       # Main orchestrator (Phases 0-8)
    │       │   ├── RunQualityGates.md    # Quality gate runner
    │       │   └── ShipFeature.md        # Git + PR + Linear sync
    │       └── Tools/
    │           ├── LinearHelpers.md      # Linear MCP patterns
    │           ├── TicketRouter.md       # Label detection logic
    │           └── RalphOrchestrator.md  # Ralph-loop patterns
    ├── hooks/
    │   └── HOOKS.md            # Hook specifications
    └── config/
        └── workflow-config.yaml    # All configuration
```

---

## Required Plugins

**Must be installed:**
```bash
# Anthropic official plugins
- feature-dev@claude-plugins-official
- frontend-design@claude-plugins-official
- ralph-wiggum@claude-plugins-official
- pr-review-toolkit@claude-plugins-official
```

**Must be configured:**
```bash
# Linear MCP integration
~/.claude/config.json:
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.linear.app/mcp"]
    }
  }
}
```

---

## Configuration

**All configuration in:** `src/config/workflow-config.yaml`

**Key settings:**
```yaml
# Routing
routing.labels.skip_discovery: [pre-planned, hotfix]
routing.labels.use_frontend_design: [ui, frontend, design]

# Ralph max iterations
plugins.ralph_wiggum.max_iterations.ui: 30
plugins.ralph_wiggum.max_iterations.backend: 50

# Quality gates
quality_gates.gates.test.command: npm test
quality_gates.auto_fix.enabled: true

# Linear
linear.states.start: "In Progress"
linear.states.review: "In Review"
```

---

## Usage Examples

### Example 1: Pre-Planned UI Feature
```bash
User: "work on ENG-789"
Ticket: "Create user profile dashboard"
Labels: [pre-planned, UI]

Flow:
✅ Phase 0: Detect labels → Skip to Phase 5 with frontend-design
✅ Phase 5: frontend-design + ralph-loop (tests → iterate → pass)
✅ Phase 6: Code review
✅ Phase 7: Quality gates
✅ Phase 8: Ship

Duration: ~30-45 minutes
```

### Example 2: Complex Backend Feature
```bash
User: "work on ENG-456"
Ticket: "Add rate limiting to API endpoints"
Labels: []

Flow:
✅ Phase 0: No special labels → Full workflow
✅ Phases 1-4: feature-dev (discovery → architecture)
✅ Phase 5: TDD with ralph-loop (50 iterations max)
✅ Phase 6: Code review
✅ Phase 7: Quality gates
✅ Phase 8: Ship

Duration: ~2-3 hours
```

### Example 3: Research Spike
```bash
User: "work on ENG-111"
Ticket: "Research websocket libraries"
Labels: [spike, research]

Flow:
✅ Phase 0: Detect "spike" → Architecture-only workflow
✅ Phases 1-4: feature-dev (exploration & research)
✅ STOP (no implementation, no PR)

Duration: ~1 hour
```

---

## Integration with PAI Systems

| PAI System | How It's Used |
|------------|--------------|
| **CORE** | Routes "work on ENG-123" to TeamDev skill |
| **History** | Captures ALL ticket work for future reference |
| **Voice** | Announces phase transitions and completion |
| **Response Format** | Uses 📋 SUMMARY, 🔍 ANALYSIS, ✅ RESULTS, 🎯 COMPLETED |
| **Hooks** | Auto-sync Linear on Stop, quality gate enforcement |

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

## Quality Guarantees

**Before shipping, ALL of these MUST pass:**
- ✅ All tests passing (0 failures)
- ✅ Linting passes (0 errors)
- ✅ Type checking passes (0 errors)
- ✅ Build succeeds
- ✅ Code coverage ≥ 80% (configurable)
- ✅ Code review complete
- ✅ Linear ticket updated

**No shortcuts allowed** - Quality gate blocker hook prevents shipping with failures.

---

## Troubleshooting

### Ralph reaches max iterations
```bash
⚠️ Ralph-loop reached max iterations (50)

Options:
1. Manually fix remaining issues
2. Increase max iterations in workflow-config.yaml
3. Lower coverage/quality requirements
4. Review if requirements are too strict
```

### Quality gates fail
```bash
❌ Quality gate failed: tests

Fix options:
1. Let ralph-loop auto-fix (automatic)
2. Review test failures and fix manually
3. Check if tests are flaky
```

### Linear connection fails
```bash
❌ Linear API connection failed

Check:
1. MCP integration configured (~/.claude/config.json)
2. Network connectivity
3. Linear API key valid
4. Rate limits not exceeded
```

---

## Performance

**Typical workflow durations:**
- Pre-planned UI feature: 30-45 min
- Pre-planned backend feature: 45-60 min
- Complex feature (full workflow): 2-3 hours
- Bug fix: 20-30 min
- Research spike: 30-60 min

**Parallelism:**
- Code-explorer agents: 2-3 in parallel
- Code-architect agents: 2-3 in parallel
- Code-reviewer agents: 3 in parallel
- Quality gates: Sequential (future: parallel)

---

## Customization

**Add custom routing rules:**
Edit `src/skills/TeamDev/Tools/TicketRouter.md` to add project-specific routing logic.

**Adjust quality thresholds:**
Edit `src/config/workflow-config.yaml`:
```yaml
tdd.enforcement.standard.min_coverage: 80  # Change to 90 for stricter
quality_gates.coverage.threshold: 80
```

**Add custom labels:**
```yaml
routing.labels.custom_behavior:
  - your-label
  - another-label
```

---

## Best Practices

1. **Label tickets consistently** - Use `pre-planned` when architecture is done
2. **Write good acceptance criteria** - Helps ralph-loop implement correctly
3. **Monitor ralph iterations** - If using >40 iterations, investigate
4. **Review auto-fixes** - Always `git diff` after ralph-loop completes
5. **Keep PRs focused** - One feature per ticket per PR
6. **Update Linear manually if needed** - Don't rely solely on automation
7. **Test the workflow** - Use VERIFY.md checklist after installation

---

## Roadmap

**v1.1:**
- [ ] Parallel quality gate execution
- [ ] Dynamic ralph iteration limits
- [ ] Auto-rebase on main before shipping
- [ ] Slack/Discord notifications

**v1.2:**
- [ ] Multi-ticket workflows (dependencies)
- [ ] Story creation from ideas
- [ ] Auto-assign reviewers based on CODEOWNERS
- [ ] Time tracking integration

**v2.0:**
- [ ] Team-specific workflow customization
- [ ] Project-based routing
- [ ] Priority-based fast-tracking
- [ ] Context preservation between sessions

---

## Contributing

This is a personal PAI pack. If you want to use/modify:

1. Fork the pack
2. Modify `src/config/workflow-config.yaml` for your needs
3. Adjust routing logic in `src/skills/TeamDev/Tools/TicketRouter.md`
4. Test with VERIFY.md
5. Document your changes

---

## Support

**Documentation:**
- Installation: `INSTALL.md`
- Verification: `VERIFY.md`
- Configuration: `src/config/workflow-config.yaml`
- Workflows: `src/skills/TeamDev/Workflows/`
- Tools: `src/skills/TeamDev/Tools/`

**Debugging:**
```bash
# Enable debug logging
DEBUG=team-dev-workflow:* work on ENG-123

# Check logs
cat ~/.claude/logs/team-dev-workflow.log
```

---

## License

Personal use only. Not for redistribution without permission.

---

## Credits

**Built with:**
- [Claude Code](https://claude.com/claude-code) - AI coding assistant
- [PAI](https://github.com/yourusername/PAI) - Personal AI infrastructure
- [Linear](https://linear.app) - Issue tracking
- [Anthropic Plugins](https://github.com/anthropics/claude-code) - feature-dev, frontend-design, ralph-wiggum, pr-review-toolkit

**Inspired by:**
- Test-Driven Development (TDD) methodology
- Conventional Commits
- GitHub Flow
- Agile development practices

---

**Version:** 1.0.0
**Last Updated:** 2026-01-08
**Author:** Jeremy O'Dell
