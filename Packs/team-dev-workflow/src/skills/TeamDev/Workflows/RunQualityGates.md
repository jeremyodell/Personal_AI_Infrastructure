---
name: RunQualityGates
description: Run all quality gates (test, lint, typecheck, build) with auto-fix capability
type: workflow
---

# RunQualityGates Workflow

**Run all quality gates - all must pass before shipping**

**Input:**
- `ticket_id` - Linear ticket ID (for context)
- `auto_fix` - Boolean, whether to use ralph-loop for auto-fixing failures (default: true)

**Output:** All gates pass ✅ or detailed failure report ❌

---

## Quality Gates

All gates must pass (0 errors, 0 failures):

1. **Tests** - `npm test`
2. **Linting** - `npm run lint`
3. **Type Checking** - `npm run typecheck`
4. **Build** - `npm run build`

---

## Workflow

### Step 1: Run All Gates

**Execute gates in sequence:**

```bash
# Gate 1: Tests
echo "🧪 Running tests..."
npm test 2>&1 | tee gate-test.log
TEST_EXIT=$?

# Gate 2: Linting
echo "🔍 Running linter..."
npm run lint 2>&1 | tee gate-lint.log
LINT_EXIT=$?

# Gate 3: Type Checking
echo "📘 Running type checker..."
npm run typecheck 2>&1 | tee gate-typecheck.log
TYPE_EXIT=$?

# Gate 4: Build
echo "🏗️  Running build..."
npm run build 2>&1 | tee gate-build.log
BUILD_EXIT=$?
```

**Collect results:**
```typescript
const gateResults = {
  test: TEST_EXIT === 0,
  lint: LINT_EXIT === 0,
  typecheck: TYPE_EXIT === 0,
  build: BUILD_EXIT === 0
};

const allPassed = Object.values(gateResults).every(passed => passed);
```

---

### Step 2: Report Results

**If all gates pass:**
```markdown
✅ All Quality Gates Passed

📋 SUMMARY: All gates passed for ${ticket_id}

🔍 GATE RESULTS:
- ✅ Tests: 0 failures
- ✅ Linting: 0 errors
- ✅ Type checking: 0 errors
- ✅ Build: successful

📁 CAPTURE: Quality gates passed for ${ticket_id} - ready to ship
```

**If any gates fail:**
```markdown
❌ Quality Gate Failures Detected

📋 SUMMARY: ${failedCount} gate(s) failed for ${ticket_id}

🔍 FAILED GATES:
${failedGates.map(gate => `- ❌ ${gate.name}: ${gate.error}`).join('\n')}

📊 STATUS:
- Tests: ${gateResults.test ? '✅' : '❌'}
- Linting: ${gateResults.lint ? '✅' : '❌'}
- Type checking: ${gateResults.typecheck ? '✅' : '❌'}
- Build: ${gateResults.build ? '✅' : '❌'}
```

---

### Step 3: Auto-Fix (If Enabled and Failures Exist)

**Condition:** `auto_fix === true && !allPassed`

**Prepare failure context:**
```typescript
const failureContext = [];

if (!gateResults.test) {
  const testOutput = readFile('gate-test.log');
  failureContext.push(`TEST FAILURES:\n${testOutput}`);
}

if (!gateResults.lint) {
  const lintOutput = readFile('gate-lint.log');
  failureContext.push(`LINTING ERRORS:\n${lintOutput}`);
}

if (!gateResults.typecheck) {
  const typeOutput = readFile('gate-typecheck.log');
  failureContext.push(`TYPE ERRORS:\n${typeOutput}`);
}

if (!gateResults.build) {
  const buildOutput = readFile('gate-build.log');
  failureContext.push(`BUILD ERRORS:\n${buildOutput}`);
}

const allFailures = failureContext.join('\n\n---\n\n');
```

**Invoke ralph-loop for auto-fix:**
```bash
/ralph-loop "Fix quality gate failures:

${allFailures}

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

Output <promise>GATES_PASSING</promise> when:
- ✅ npm test passes (0 failures)
- ✅ npm run lint passes (0 errors)
- ✅ npm run typecheck passes (0 errors)
- ✅ npm run build succeeds
" --completion-promise "GATES_PASSING" --max-iterations 20
```

**After ralph-loop completes:**
```markdown
🔧 Auto-Fix Complete

📋 SUMMARY: Ralph-loop attempted to fix quality gate failures

⚡ ACTIONS:
- Analyzed failures across ${failedCount} gates
- Applied fixes over ${iterationsUsed} iterations
- Re-running gates to verify...
```

---

### Step 4: Re-Run Gates After Auto-Fix

**If auto-fix was applied, re-run all gates:**

```bash
# Re-run all gates
npm test
npm run lint
npm run typecheck
npm run build
```

**Check results again:**
```typescript
const retryResults = {
  test: /* check exit code */,
  lint: /* check exit code */,
  typecheck: /* check exit code */,
  build: /* check exit code */
};

const allPassedAfterFix = Object.values(retryResults).every(passed => passed);
```

---

### Step 5: Final Report

**If all gates pass after auto-fix:**
```markdown
✅ Quality Gates Passed After Auto-Fix

📋 SUMMARY: All gates now passing for ${ticket_id}

🔍 AUTO-FIX RESULTS:
- Gates failed initially: ${failedCount}
- Ralph iterations used: ${iterationsUsed}
- All gates now passing: ✅

📁 CAPTURE: Quality gates auto-fixed and passed for ${ticket_id}

🎯 COMPLETED: Quality gates passed - ready to ship
```

**If gates still fail after auto-fix:**
```markdown
❌ QUALITY GATE FAILURE: Cannot Proceed

📋 SUMMARY: Auto-fix unable to resolve all failures for ${ticket_id}

🔍 PERSISTENT FAILURES:
${stillFailingGates.map(gate => `- ❌ ${gate.name}`).join('\n')}

⚡ AUTO-FIX STATS:
- Ralph iterations used: 20 (max)
- Gates fixed: ${fixedCount}
- Gates still failing: ${stillFailingCount}

📊 NEXT STEPS:
1. Review failure logs:
   ${stillFailingGates.map(g => `   - ${g.logFile}`).join('\n')}
2. Manually fix remaining issues
3. Re-run: /ralph-loop or RunQualityGates workflow

❌ BLOCKED: Cannot ship until all quality gates pass
```

---

## Configuration

**Gate commands** (from `config/workflow-config.yaml`):
```yaml
quality_gates:
  test:
    command: "npm test"
    timeout: 300000  # 5 minutes
  lint:
    command: "npm run lint"
    timeout: 60000   # 1 minute
  typecheck:
    command: "npm run typecheck"
    timeout: 120000  # 2 minutes
  build:
    command: "npm run build"
    timeout: 300000  # 5 minutes

auto_fix:
  enabled: true
  max_iterations: 20
  ralph_model: "sonnet"  # or "opus" for complex fixes
```

---

## Error Handling

### If Gate Command Not Found:
```markdown
❌ ERROR: Gate command not found

Gate: ${gateName}
Command: ${command}

Possible causes:
- Command not defined in package.json scripts
- npm dependencies not installed (run: npm install)
- Wrong working directory

Please verify package.json scripts and try again.
```

### If Ralph-Loop Fails to Start:
```markdown
❌ ERROR: Ralph-loop failed to start

Possible causes:
- ralph-wiggum plugin not installed
- Invalid ralph-loop syntax
- Context too large for model

Please verify ralph-wiggum is installed and try again.
```

### If Timeout Exceeded:
```markdown
⚠️ WARNING: Gate timeout exceeded

Gate: ${gateName}
Timeout: ${timeout}ms

Gate is still running but exceeded timeout.
Consider increasing timeout in workflow-config.yaml.
```

---

## Usage

**Invoke from WorkOnTicket workflow:**
```typescript
await invokeWorkflow({
  workflow: "RunQualityGates",
  params: {
    ticket_id: "ENG-123",
    auto_fix: true
  }
});
```

**Direct invocation:**
```bash
# Run with auto-fix
/run RunQualityGates --ticket-id ENG-123 --auto-fix true

# Run without auto-fix (manual fix)
/run RunQualityGates --ticket-id ENG-123 --auto-fix false
```

---

## Gate Output Files

**Log files created:**
- `gate-test.log` - Test output
- `gate-lint.log` - Linting output
- `gate-typecheck.log` - Type checking output
- `gate-build.log` - Build output

**Cleanup:**
```bash
# Clean up log files after successful run
rm -f gate-*.log
```

---

## Integration with PAI Hooks

**quality-gate-blocker hook:**
- Prevents shipping if any gate fails
- Triggered on `Stop` event
- Checks gate status before allowing session end

**Usage in hook:**
```typescript
// In quality-gate-blocker.ts hook
const gatesPass = await runQualityGates({
  ticket_id: getCurrentTicketId(),
  auto_fix: false  // Don't auto-fix on Stop
});

if (!gatesPass) {
  throw new Error("Cannot end session - quality gates failing");
}
```

---

## Best Practices

1. **Run gates frequently** - Don't wait until Phase 7
2. **Enable auto-fix** - Let ralph-loop handle simple failures
3. **Monitor iterations** - If ralph uses 15+ iterations, manual review recommended
4. **Review logs** - Always check gate output logs for root causes
5. **Update config** - Adjust timeouts if gates consistently timeout
6. **Fail fast** - Stop at first gate failure to save time

---

## Performance Optimization

**Parallel execution** (future enhancement):
```bash
# Run non-dependent gates in parallel
npm test & TEST_PID=$!
npm run lint & LINT_PID=$!
npm run typecheck & TYPE_PID=$!

# Wait for all
wait $TEST_PID && TEST_EXIT=$?
wait $LINT_PID && LINT_EXIT=$?
wait $TYPE_PID && TYPE_EXIT=$?

# Then run build (depends on typecheck)
npm run build
```

**Caching:**
- Test results cache for unchanged files
- Type checking cache (tsconfig.json: `"incremental": true`)
- Build cache (webpack/vite cache)
