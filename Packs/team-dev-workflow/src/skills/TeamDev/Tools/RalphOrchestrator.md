# Ralph-Wiggum Integration

**TDD enforcement using ralph-wiggum iteration loops**

This document explains how to use the ralph-wiggum plugin for deterministic TDD enforcement in the team-dev-workflow.

---

## What is Ralph-Wiggum?

Ralph-wiggum is an Anthropic plugin that provides **iteration loops with completion detection**:
- Keeps trying until a specific goal is met
- Uses `<promise>` tags for completion detection
- Self-corrects based on failures
- Max iterations prevent infinite loops
- Returns control when promise fulfilled

**Perfect for:**
- TDD: Iterate until all tests pass
- Quality gates: Iterate until all gates pass
- Bug fixes: Iterate until bug is resolved
- Refactoring: Iterate until quality metrics met

---

## Basic Ralph-Loop Syntax

```bash
/ralph-loop "TASK_DESCRIPTION

REQUIREMENTS:
- Requirement 1
- Requirement 2

Output <promise>COMPLETION_TAG</promise> when done.
" --completion-promise "COMPLETION_TAG" --max-iterations 30
```

**Key parameters:**
- Task description with requirements
- `<promise>TAG</promise>` in task for completion signal
- `--completion-promise "TAG"` matches the promise tag
- `--max-iterations N` prevents infinite loops

---

## Pattern: TDD Implementation Loop

**For standard backend/feature implementation:**

```bash
/ralph-loop "Implement feature following TDD methodology.

TICKET: ${identifier} - ${title}

ACCEPTANCE CRITERIA:
${acceptance_criteria}

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
- ✅ Follows project conventions (check CLAUDE.md)

DO NOT:
- Skip writing tests to save time
- Comment out failing tests
- Implement without tests first
- Use 'any' types to bypass type errors

Output <promise>IMPLEMENTATION_COMPLETE</promise> when ALL requirements met.
" --completion-promise "IMPLEMENTATION_COMPLETE" --max-iterations 50
```

**Why 50 iterations for backend?**
- Complex features may need multiple test→implement→refactor cycles
- Accounts for edge cases and error handling
- Allows time for quality improvements

---

## Pattern: UI Implementation Loop

**For frontend/UI features using frontend-design:**

```bash
/ralph-loop "Implement the frontend design based on architecture.

REQUIREMENTS:
- All tests must pass (npm test: 0 failures)
- No console errors in development (npm run dev)
- Meets accessibility standards (WCAG 2.1 AA)
- Matches aesthetic vision from design
- Follows project UI conventions
- Responsive design (mobile, tablet, desktop)

ACCESSIBILITY CHECKLIST:
- [ ] Semantic HTML elements
- [ ] ARIA labels where needed
- [ ] Keyboard navigation works
- [ ] Focus indicators visible
- [ ] Color contrast >= 4.5:1
- [ ] Screen reader friendly

OUTPUT REQUIREMENTS:
- Component tests passing
- User interaction tests passing
- Accessibility tests passing
- Visual matches design intent

Output <promise>UI_IMPLEMENTATION_COMPLETE</promise> when:
- ✅ All tests passing
- ✅ Visual matches design intent
- ✅ Code follows project conventions
- ✅ No accessibility violations
" --completion-promise "UI_IMPLEMENTATION_COMPLETE" --max-iterations 30
```

**Why 30 iterations for UI?**
- UI implementation is typically more straightforward than backend
- Focuses on visual fidelity and accessibility
- Most complexity is in the design phase (already done)

---

## Pattern: Quality Gates Auto-Fix Loop

**For fixing failing quality gates:**

```bash
/ralph-loop "Fix quality gate failures:

FAILED GATES:
${failed_gates_output}

REQUIREMENTS:
- Fix ALL failures listed above
- Ensure tests pass (npm test: 0 failures)
- Ensure linting passes (npm run lint: 0 errors)
- Ensure type checking passes (npm run typecheck: 0 errors)
- Ensure build succeeds (npm run build: successful)

DO NOT:
- Skip or comment out failing tests
- Disable linting rules to pass
- Use 'any' types to bypass type errors
- Modify build config to ignore errors
- Create workarounds instead of real fixes

APPROACH:
1. Analyze root cause of each failure
2. Fix root cause (not symptoms)
3. Verify fix doesn't break other tests
4. Re-run all gates to confirm

Output <promise>GATES_PASSING</promise> when:
- ✅ npm test passes (0 failures)
- ✅ npm run lint passes (0 errors)
- ✅ npm run typecheck passes (0 errors)
- ✅ npm run build succeeds
" --completion-promise "GATES_PASSING" --max-iterations 20
```

**Why 20 iterations for gate fixes?**
- Gate failures are usually specific and fixable
- Most fixes should work within 10 iterations
- 20 provides buffer for cascading failures

---

## Pattern: Bug Fix Loop

**For fixing specific bugs:**

```bash
/ralph-loop "Fix bug: ${bug_description}

BUG DETAILS:
${bug_context}

REPRODUCTION STEPS:
1. ${step1}
2. ${step2}
3. ${step3}

EXPECTED BEHAVIOR:
${expected}

ACTUAL BEHAVIOR:
${actual}

REQUIREMENTS:
- Bug is fixed (reproduction steps no longer produce error)
- Root cause addressed (not just symptom)
- Regression test added (test that would have caught this bug)
- All existing tests still pass
- No new linting or type errors introduced

APPROACH:
1. Reproduce the bug consistently
2. Write failing test that captures the bug
3. Fix root cause
4. Verify test now passes
5. Ensure no regressions

Output <promise>BUG_FIXED</promise> when:
- ✅ Bug reproduction steps no longer trigger error
- ✅ Regression test added and passing
- ✅ All tests passing
- ✅ No new errors introduced
" --completion-promise "BUG_FIXED" --max-iterations 30
```

---

## Pattern: Refactoring Loop

**For code refactoring with quality metrics:**

```bash
/ralph-loop "Refactor ${component_name} for improved quality.

REFACTORING GOALS:
- Reduce complexity (cyclomatic complexity < 10)
- Improve maintainability (clear, self-documenting code)
- Eliminate duplication (DRY principle)
- Follow project patterns (check CLAUDE.md)

REQUIREMENTS:
- All existing tests still pass
- Code coverage maintained or improved
- No new linting errors
- No new type errors
- Performance not degraded

REFACTORING SAFETY:
1. Ensure comprehensive test coverage BEFORE refactoring
2. Make small, incremental changes
3. Run tests after each change
4. Keep commits small and focused

Output <promise>REFACTORING_COMPLETE</promise> when:
- ✅ Code quality improved (less complexity, more readable)
- ✅ All tests passing
- ✅ No functionality changed (behavior preserved)
- ✅ Follows project conventions
" --completion-promise "REFACTORING_COMPLETE" --max-iterations 25
```

---

## Iteration Recommendations

| Task Type | Max Iterations | Rationale |
|-----------|---------------|-----------|
| UI Implementation | 30 | Design already done, focus on tests & accessibility |
| Backend Implementation | 50 | Complex logic, multiple test cycles needed |
| Bug Fix | 30 | Focused fix with regression test |
| Quality Gate Fixes | 20 | Specific failures to address |
| Refactoring | 25 | Safety-first with frequent test runs |
| Research/Spike | 15 | Exploration, not production code |

---

## Monitoring Ralph-Loop Progress

**Ralph provides progress updates:**
```
Ralph iteration 1/50: Writing initial tests...
Ralph iteration 5/50: Implementing core functionality...
Ralph iteration 12/50: Fixing edge case in validation...
Ralph iteration 18/50: All tests passing, checking coverage...
Ralph iteration 20/50: Coverage at 85%, adding missing tests...
Ralph iteration 23/50: <promise>IMPLEMENTATION_COMPLETE</promise>

✅ Ralph-loop completed in 23 iterations
```

**Key metrics to watch:**
- **Iterations used** - If approaching max, may need intervention
- **Progress patterns** - Should see steady progress, not loops
- **Test status** - Should trend toward passing over time
- **Completion signal** - `<promise>` tag indicates success

---

## When Ralph Reaches Max Iterations

```markdown
⚠️ WARNING: Ralph-loop reached max iterations

Phase: Implementation
Iterations used: 50 (max)

Current status:
- Tests passing: 45/50 (90%)
- Coverage: 78% (target: 80%)
- Linting: 2 errors remaining
- Type checking: 0 errors

NEXT STEPS:
1. Review progress - 90% tests passing is close!
2. Options:
   a) Manually fix remaining 2 linting errors
   b) Increase max iterations to 60 and retry
   c) Lower coverage requirement to 78%
3. Consider if requirements are too strict
```

**Common causes of max iterations:**
- Requirements too strict (e.g., 95% coverage on complex code)
- External dependencies failing intermittently
- Insufficient context (Ralph doesn't understand codebase)
- Conflicting requirements (can't satisfy all at once)

---

## Ralph Best Practices

### 1. Clear Completion Criteria

**Bad:**
```bash
"Make it work and be good quality"
```

**Good:**
```bash
"Requirements:
- All tests passing (npm test: 0 failures)
- Code coverage >= 80%
- No linting errors
- No type errors

Output <promise>COMPLETE</promise> when all requirements met."
```

### 2. Specific Requirements

**Bad:**
```bash
"Fix the tests"
```

**Good:**
```bash
"Fix test failures in:
- src/components/Dashboard.test.tsx (3 failing)
- src/utils/validation.test.ts (2 failing)

Each test must pass without:
- Commenting out assertions
- Mocking away the actual behavior
- Changing test expectations to match buggy behavior"
```

### 3. Avoid Escape Hatches

**Prevent shortcuts:**
```bash
"DO NOT:
- Comment out failing tests
- Disable linting rules
- Use 'any' types
- Mock everything to pass tests
- Change requirements to match buggy behavior"
```

### 4. Verify Externally

**Don't trust Ralph's word alone:**
```bash
# After ralph-loop completes, verify manually
npm test  # Should show 0 failures
npm run lint  # Should show 0 errors

# Check ralph actually fixed the issue
git diff  # Review changes made
```

### 5. Incremental Iterations

**Start conservative, increase if needed:**
```bash
# First attempt
--max-iterations 30

# If it reaches 30 but making good progress
--max-iterations 50

# If still not enough
--max-iterations 75  # But investigate why!
```

---

## Debugging Ralph-Loop Issues

### Issue: Ralph keeps looping without progress

**Diagnosis:**
```bash
# Check if requirements are contradictory
Ralph iteration 10: Fixed linting, but broke tests
Ralph iteration 11: Fixed tests, but broke linting
Ralph iteration 12: Fixed linting, but broke tests
...
```

**Solution:**
- Simplify requirements
- Split into multiple ralph-loops
- Provide more context about codebase patterns

### Issue: Ralph completes but requirements not met

**Diagnosis:**
```bash
✅ Ralph completed: <promise>COMPLETE</promise>

# But when you check:
npm test  # Still has failures!
```

**Solution:**
- Ralph may have hallucinated completion
- Always verify externally after ralph-loop
- Make requirements more specific and verifiable

### Issue: Ralph introduces regressions

**Diagnosis:**
```bash
# Tests that were passing now fail
Ralph iteration 15: Fixed target bug, but broke auth tests
```

**Solution:**
- Add requirement: "All EXISTING tests must still pass"
- Run full test suite, not just new tests
- Provide context about related systems

---

## Integration with Workflows

### In WorkOnTicket.md (Phase 5):

```typescript
// After writing tests, use ralph-loop for implementation
const ralphCompleted = await invokeRalphLoop({
  task: "Implement feature with TDD",
  acceptanceCriteria: criteria,
  maxIterations: 50,
  completionPromise: "IMPLEMENTATION_COMPLETE"
});

if (!ralphCompleted) {
  console.log("⚠️ Ralph reached max iterations");
  // Decide: continue manually, retry, or adjust requirements
}
```

### In RunQualityGates.md (Phase 7):

```typescript
// If gates fail, use ralph-loop to auto-fix
if (!allGatesPassed) {
  const ralphCompleted = await invokeRalphLoop({
    task: "Fix quality gate failures",
    failures: gateFailures,
    maxIterations: 20,
    completionPromise: "GATES_PASSING"
  });

  // Re-run gates after ralph completes
  allGatesPassed = await runGates();
}
```

---

## Ralph vs Manual Implementation

| Aspect | Ralph-Loop | Manual |
|--------|-----------|--------|
| **TDD enforcement** | ✅ Automatic | ⚠️ Requires discipline |
| **Iteration until pass** | ✅ Built-in | ❌ Must remember |
| **Self-correction** | ✅ Learns from failures | ⚠️ Manual debugging |
| **Max iterations safety** | ✅ Prevents infinite loops | N/A |
| **Progress visibility** | ✅ Shows iterations | ⚠️ Less visible |
| **Context retention** | ⚠️ Limited by model | ✅ Full context |
| **Complex decisions** | ⚠️ May struggle | ✅ Better judgment |

**When to use Ralph:**
- Straightforward TDD implementation
- Quality gate auto-fixes
- Repetitive refinement tasks
- Time-consuming iteration loops

**When to skip Ralph:**
- Highly complex architectural decisions
- Novel/unusual patterns in codebase
- External dependencies/integration issues
- Tasks requiring deep judgment

---

## Advanced: Custom Completion Detection

**Multiple completion conditions:**
```bash
/ralph-loop "Implement feature with multiple checkpoints.

Output <promise>TESTS_WRITTEN</promise> when tests are complete.
Output <promise>IMPLEMENTATION_COMPLETE</promise> when implementation done.
Output <promise>DOCUMENTATION_COMPLETE</promise> when docs added.

" --completion-promise "DOCUMENTATION_COMPLETE" --max-iterations 40
```

**Conditional completion:**
```bash
/ralph-loop "Fix bug, but only if reproducible.

If bug cannot be reproduced:
  Output <promise>CANNOT_REPRODUCE</promise>

If bug fixed successfully:
  Output <promise>BUG_FIXED</promise>

" --completion-promise "BUG_FIXED|CANNOT_REPRODUCE" --max-iterations 25
```

---

## Performance Optimization

**Reduce iterations by providing context:**

```bash
/ralph-loop "Implement ${feature}

CODEBASE CONTEXT:
- Similar feature: ${similar_feature_path}
- Test pattern: ${test_pattern_example}
- Project conventions: ${claude_md_summary}

EXAMPLE IMPLEMENTATION:
${example_code}

This context should help you implement correctly in fewer iterations.

Output <promise>COMPLETE</promise> when done.
" --completion-promise "COMPLETE" --max-iterations 30
```

---

## Testing Ralph Integration

```typescript
// Mock ralph-loop for testing workflows
const mockRalphLoop = async (
  task: string,
  maxIterations: number
): Promise<boolean> => {
  console.log(`Mock ralph-loop: ${task}`);
  console.log(`Would run for max ${maxIterations} iterations`);

  // Simulate success
  return true;
};

// Test workflow with mock
await testWorkflow({ ralphLoop: mockRalphLoop });
```

---

## Common Gotchas

1. **Forgetting `<promise>` tags** - Ralph needs explicit completion signal
2. **Mismatched promise tags** - Tag in task must match `--completion-promise`
3. **Too strict requirements** - Impossible to satisfy, hits max iterations
4. **Too loose requirements** - Ralph completes but work incomplete
5. **No external verification** - Trust but verify ralph's completion
6. **Context limits** - Ralph has limited context, may not see full codebase
7. **Non-deterministic tests** - Flaky tests confuse ralph's iteration logic

---

## Future Enhancements

1. **Dynamic iteration limits** - Adjust based on task complexity
2. **Progress checkpoints** - Save state every N iterations
3. **Parallel ralph-loops** - Multiple loops for independent tasks
4. **Ralph telemetry** - Track iteration patterns for optimization
5. **Smart completion** - AI-detected completion without promise tags
