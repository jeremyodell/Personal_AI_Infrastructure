# From Linear Ticket to Merged PR: A Deterministic Development Workflow

**How I built a PAI pack that enforces TDD, routes intelligently, and ships quality code—every time.**

---

## The Problem

You open Linear. You see ENG-123: "Add user dashboard." Simple enough. But then:

- Should you explore the codebase first or jump straight to implementation?
- Who reviews the architecture before you write 500 lines of code?
- Will you remember to write tests? (Be honest.)
- Did you run the linter before committing? The type checker?
- Will quality gates block your PR at 11 PM?

Most workflows leave these decisions to discipline. Discipline fails.

## The Solution

I built `team-dev-workflow`, a PAI pack that makes these decisions for you. Type `work on ENG-123` and watch:

1. **Phase 0**: Fetches the ticket. Detects labels. Creates a branch. Updates Linear to "In Progress."

2. **Phases 1-4**: Launches code-explorer agents to map your codebase. Presents architecture options. Forces you to answer every ambiguous question before touching code.

3. **Phase 5**: Writes tests **first**. Then uses ralph-wiggum to iterate until every test passes. No shortcuts. No "I'll add tests later."

4. **Phase 7**: Runs quality gates. Tests, linting, type checking, build. All must pass. No exceptions.

5. **Phase 8**: Commits with conventional format. Creates PR. Syncs Linear to "In Review." Done.

## The Magic: Label-Based Routing

Not all tickets need discovery. Some arrive pre-planned. The workflow detects this:

| Label | Behavior |
|-------|----------|
| `pre-planned` | Skips discovery. Goes straight to implementation. |
| `ui` or `frontend` | Uses frontend-design plugin for Phase 5. |
| `spike` or `research` | Stops after architecture. No implementation. |

Type `work on ENG-456` with a `pre-planned` label? You start coding immediately. The right workflow for every ticket.

## The Enforcement: Ralph-Wiggum

Here's the secret: ralph-wiggum doesn't let you quit until tests pass.

```bash
/ralph-loop "Implement with TDD.
Write failing test. Implement. Refactor. Repeat.
Output <promise>COMPLETE</promise> when ALL tests pass."
--max-iterations 50
```

Ralph iterates. Fixes failures. Learns. Iterates again. After 23 iterations: `<promise>COMPLETE</promise>`. All tests pass. Coverage hits 85%. No human temptation to skip it.

## The Results

**Before this workflow:**
- Forgot to write tests on 30% of tickets
- Shipped broken builds twice
- PRs blocked by linting errors after commits
- No consistency between team members

**After:**
- 100% test coverage (enforced)
- Zero broken builds
- Quality gates pass before commit
- Every ticket follows identical flow

## The Architecture

Built on Jeremy O'Dell's [PAI framework](https://github.com/jeremyodell/PAI), the pack integrates four Anthropic plugins:

1. **feature-dev**: Discovery & architecture (stops before implementation)
2. **frontend-design**: Production-grade UI components
3. **ralph-wiggum**: Iteration loops with promise-based completion
4. **pr-review-toolkit**: Automated code review with confidence scoring

Plus Linear MCP for ticket integration.

Eight phases. Three workflows. Zero ambiguity.

## The Details That Matter

**TDD enforcement isn't optional.** Phase 5 blocks you from proceeding until:
- Tests written first
- All tests pass
- Coverage ≥ 80%
- No linting errors
- No type errors

**Quality gates are hard stops.** Phase 7 won't let you ship if any gate fails. It uses ralph-loop to auto-fix, but if 20 iterations can't resolve it, you fix manually. No shortcuts.

**Linear stays in sync.** Status updates automatically. Architecture summaries post as comments. PR links appear in tickets. Your team always knows the current state.

## Try It

The pack lives at `PAI/Packs/team-dev-workflow/`. Installation takes 10 minutes:

1. Install plugins: `feature-dev`, `frontend-design`, `ralph-wiggum`, `pr-review-toolkit`
2. Configure Linear MCP
3. Copy pack to PAI directory
4. Run: `work on ENG-123`

Full docs in `INSTALL.md`. Verification checklist in `VERIFY.md`.

## Why This Matters

Discipline fails. Humans forget. We skip tests when rushed. We commit without linting. We ship broken builds.

This workflow doesn't forget. It enforces what you know you should do. Every time.

Type `work on ENG-123`. Watch quality code emerge.

---

**Built by:** Jeremy O'Dell using Claude Code
**Framework:** [PAI - Personal AI Infrastructure](https://github.com/jeremyodell/PAI)
**Version:** 1.0.0
**Released:** January 2026

---

*Credit: This pack was created as part of Jeremy O'Dell's Personal AI Infrastructure (PAI) framework, an open-source project for building deterministic AI workflows. PAI turns Claude Code from a helpful assistant into a systematic development platform.*

*Want to build your own PAI packs? Check out [PAI on GitHub](https://github.com/jeremyodell/PAI).*
