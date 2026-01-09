# Team-Dev-Workflow Hooks

**PAI hooks for Linear sync and quality enforcement**

This document specifies the hooks that should be implemented for the team-dev-workflow pack.

---

## Hook Overview

Hooks are event handlers that run automatically at specific points in the workflow:

| Hook | Event | Purpose |
|------|-------|---------|
| `linear-sync-on-complete` | `Stop` | Auto-sync Linear ticket status when session ends |
| `quality-gate-blocker` | `Stop` | Prevent session end if quality gates failing |

---

## Hook 1: linear-sync-on-complete

**Event:** `Stop` (session end)

**Purpose:** Automatically update Linear ticket status and post final summary when work session ends

### Behavior

```typescript
// Hook triggered when user ends session (Ctrl+C, /exit, etc.)
onStop(async (context) => {
  // 1. Check if we're working on a Linear ticket
  const ticketId = context.getVariable('CURRENT_TICKET_ID');

  if (!ticketId) {
    // Not working on a ticket, skip
    return;
  }

  // 2. Determine current phase
  const currentPhase = context.getVariable('CURRENT_PHASE');

  // 3. Update Linear based on phase
  if (currentPhase >= 5) {  // Implementation or later
    await mcp__linear__create_comment({
      issueId: ticketId,
      body: `## Session Ended

Work paused at: Phase ${currentPhase} - ${getPhaseName(currentPhase)}

Progress:
${context.getVariable('PROGRESS_SUMMARY')}

Next session should resume from: ${getNextStep(currentPhase)}
`
    });
  }

  // 4. If work complete, update status
  const isComplete = context.getVariable('WORK_COMPLETE');
  if (isComplete) {
    await mcp__linear__update_issue({
      id: ticketId,
      state: "In Review"
    });
  }
});
```

### Configuration

```yaml
# In workflow-config.yaml
hooks:
  linear_sync_on_complete:
    enabled: true
    sync_on_stop: true
    post_summary: true
    update_status: true
```

### Use Cases

1. **User forgets to ship** - Hook posts progress even if user didn't complete Phase 8
2. **Work in progress** - Captures what was done for next session
3. **Context preservation** - Linear becomes source of truth for progress

### Implementation Notes

```typescript
// Hook implementation should:
// 1. Be non-blocking (don't prevent session end)
// 2. Handle errors gracefully (log but don't fail)
// 3. Be fast (<2 seconds)
// 4. Respect user privacy (don't leak sensitive data)

// Pseudocode structure:
export default {
  event: 'Stop',
  async handler(context) {
    try {
      const ticketId = getCurrentTicketId(context);
      if (!ticketId) return;

      const progress = getProgressSummary(context);
      await syncLinear(ticketId, progress);

      console.log(`✅ Linear ticket ${ticketId} synced`);
    } catch (error) {
      console.error(`⚠️ Linear sync failed:`, error);
      // Don't throw - allow session to end
    }
  }
};
```

---

## Hook 2: quality-gate-blocker

**Event:** `Stop` (session end)

**Purpose:** Prevent session end if quality gates are failing, ensuring code quality

### Behavior

```typescript
onStop(async (context) => {
  // 1. Check if we're in shipping phase
  const currentPhase = context.getVariable('CURRENT_PHASE');

  if (currentPhase < 7) {
    // Not at quality gates phase yet, allow stop
    return;
  }

  // 2. Check quality gate status
  const gatesStatus = await checkQualityGates();

  if (!gatesStatus.allPassed) {
    // 3. Block session end
    throw new Error(`
❌ BLOCKED: Cannot end session - quality gates failing

Failed gates:
${gatesStatus.failures.map(f => `- ${f.name}: ${f.error}`).join('\n')}

NEXT STEPS:
1. Fix failing gates:
   npm test       # ${gatesStatus.test ? '✅' : '❌'}
   npm run lint   # ${gatesStatus.lint ? '✅' : '❌'}
   npm run typecheck  # ${gatesStatus.typecheck ? '✅' : '❌'}
   npm run build  # ${gatesStatus.build ? '✅' : '❌'}

2. Or use ralph-loop to auto-fix:
   /ralph-loop "Fix quality gate failures" --max-iterations 20

3. Then try ending session again

Cannot proceed until all gates pass.
`);
  }

  console.log('✅ Quality gates passing - session can end');
});
```

### Configuration

```yaml
# In workflow-config.yaml
hooks:
  quality_gate_blocker:
    enabled: true
    block_on_failure: true
    allow_override: false  # If true, user can force-exit with flag
    grace_period: 300  # Seconds before blocking (allow quick exits for emergencies)
```

### Use Cases

1. **Prevent shipping broken code** - Can't end session with failing tests
2. **Enforce quality standards** - No shortcuts allowed
3. **Catch last-minute breaks** - User might introduce bug right before shipping

### Override Mechanism

**For emergencies only:**
```bash
# Allow force-exit if absolutely necessary
/exit --force --reason "Production incident, need to switch tasks"

# Hook should log override but allow it
console.warn(`⚠️ Quality gate blocker overridden: ${reason}`);
```

### Implementation Notes

```typescript
// Hook implementation:
export default {
  event: 'Stop',
  async handler(context) {
    const currentPhase = getCurrentPhase(context);

    // Only check if we're past implementation
    if (currentPhase < 7) return;

    // Check quality gates
    const gatesPass = await runQuickGateCheck();

    if (!gatesPass) {
      const allowOverride = context.getConfig('quality_gate_blocker.allow_override');

      if (allowOverride && context.hasFlag('--force')) {
        const reason = context.getFlag('--reason') || 'No reason provided';
        console.warn(`⚠️ Quality gates bypassed: ${reason}`);
        await logOverride(reason);
        return;  // Allow exit
      }

      // Block exit
      throw new Error(generateGateFailureMessage(gatesPass));
    }
  }
};
```

---

## Hook 3: context-capture (Future Enhancement)

**Event:** Multiple (tool use, phase completion, etc.)

**Purpose:** Automatically capture important context for PAI history system

### Behavior

```typescript
onPhaseComplete(async (phase, context) => {
  // Capture key decisions and outcomes
  await context.history.capture({
    event: `Phase ${phase} complete`,
    ticketId: context.ticketId,
    phase: phase,
    decisions: context.getDecisions(),
    outcomes: context.getOutcomes(),
    files_modified: context.getModifiedFiles(),
    tests_written: context.getTestsWritten(),
    timestamp: Date.now()
  });
});
```

---

## Hook 4: notification-sender (Future Enhancement)

**Event:** `ShipComplete`

**Purpose:** Send notifications when work is shipped

### Behavior

```typescript
onShipComplete(async (context) => {
  // Notify team via Slack, Discord, etc.
  await sendNotification({
    channel: '#engineering',
    message: `🚀 ${context.identifier} shipped by ${context.user}`,
    prUrl: context.prUrl,
    ticketUrl: context.ticketUrl
  });
});
```

---

## Testing Hooks

### Manual Testing

```bash
# Test linear-sync-on-complete
1. Start workflow: "work on ENG-123"
2. Complete Phase 5 (implementation)
3. End session: Ctrl+C
4. Check Linear for progress comment

# Test quality-gate-blocker
1. Start workflow: "work on ENG-456"
2. Complete implementation with failing tests
3. Try to end session: Ctrl+C
4. Should be blocked with error message
5. Fix tests and try again
6. Should succeed
```

### Automated Testing

```typescript
// Mock context for testing
const mockContext = {
  getVariable: (key) => testData[key],
  getConfig: (key) => config[key],
  hasFlag: (flag) => flags.includes(flag)
};

// Test hook
describe('quality-gate-blocker', () => {
  it('blocks exit when gates failing', async () => {
    mockContext.currentPhase = 7;
    mockContext.gateStatus = { allPassed: false };

    await expect(
      qualityGateBlocker.handler(mockContext)
    ).rejects.toThrow('BLOCKED');
  });

  it('allows exit when gates passing', async () => {
    mockContext.currentPhase = 7;
    mockContext.gateStatus = { allPassed: true };

    await expect(
      qualityGateBlocker.handler(mockContext)
    ).resolves.not.toThrow();
  });
});
```

---

## Hook Best Practices

1. **Fast execution** - Hooks should complete in <2 seconds
2. **Graceful errors** - Don't crash the system on hook failure
3. **Idempotent** - Safe to run multiple times
4. **Minimal side effects** - Don't modify user's code without explicit permission
5. **User feedback** - Always show what the hook is doing
6. **Configurable** - Allow users to disable/configure hooks
7. **Privacy-conscious** - Don't leak sensitive data to external services

---

## Hook Configuration Schema

```yaml
# workflow-config.yaml
hooks:
  # Linear sync on session end
  linear_sync_on_complete:
    enabled: true
    sync_on_stop: true
    post_summary: true
    update_status: true
    timeout_ms: 5000

  # Quality gate enforcement
  quality_gate_blocker:
    enabled: true
    block_on_failure: true
    allow_override: false
    grace_period: 300
    check_gates:
      - test
      - lint
      - typecheck
      - build

  # Context capture (future)
  context_capture:
    enabled: false
    capture_on:
      - phase_complete
      - decision_made
      - error_occurred

  # Notifications (future)
  notification_sender:
    enabled: false
    channels:
      slack:
        webhook_url: "${SLACK_WEBHOOK_URL}"
        channel: "#engineering"
      discord:
        webhook_url: "${DISCORD_WEBHOOK_URL}"
```

---

## Disabling Hooks

**Temporarily disable a hook:**
```bash
# Via environment variable
DISABLE_QUALITY_GATE_BLOCKER=true work on ENG-123

# Via config override
work on ENG-123 --no-quality-blocker
```

**Permanently disable:**
```yaml
# In workflow-config.yaml
hooks:
  quality_gate_blocker:
    enabled: false
```

---

## Hook Development

**Creating a new hook:**

1. Define hook in `src/hooks/`
2. Specify event trigger
3. Implement handler function
4. Add configuration schema
5. Write tests
6. Document in HOOKS.md
7. Update INSTALL.md with setup instructions

**Example hook structure:**
```typescript
// src/hooks/my-custom-hook.ts
export default {
  name: 'my-custom-hook',
  description: 'Does something useful',
  event: 'PhaseComplete',  // Event to listen for

  // Configuration schema
  config: {
    enabled: true,
    custom_option: 'value'
  },

  // Hook handler
  async handler(context) {
    const config = context.getConfig('my-custom-hook');

    if (!config.enabled) return;

    try {
      // Do something useful
      await doSomething();
    } catch (error) {
      console.error(`Hook error:`, error);
      // Don't throw - allow workflow to continue
    }
  }
};
```

---

## Available Events

| Event | When Triggered | Context Available |
|-------|---------------|-------------------|
| `SessionStart` | Session begins | User, settings |
| `SessionEnd` / `Stop` | Session ends | Full session context |
| `PhaseComplete` | Workflow phase done | Phase number, outcomes |
| `ToolUse` | Before/after tool use | Tool name, parameters |
| `Error` | Error occurs | Error details, stack |
| `Decision` | User makes decision | Question, answer |

---

## Security Considerations

1. **API keys** - Never log or expose API keys in hook output
2. **Sensitive data** - Don't post sensitive code/data to external services
3. **User consent** - Hooks that send data externally need explicit consent
4. **Rate limiting** - Respect API rate limits (especially Linear)
5. **Injection** - Sanitize user input before using in hook logic

---

## Debugging Hooks

**Enable hook debugging:**
```bash
DEBUG=hooks:* work on ENG-123

# Output:
# hooks:linear-sync-on-complete Triggered on Stop event
# hooks:linear-sync-on-complete Ticket ID: abc123
# hooks:linear-sync-on-complete Posting comment to Linear...
# hooks:linear-sync-on-complete ✅ Complete
```

**Hook execution log:**
```bash
# View hook execution history
cat ~/.claude/logs/hooks.log

# Recent entries:
# 2026-01-08T19:30:45Z [linear-sync-on-complete] SUCCESS (1.2s)
# 2026-01-08T19:30:46Z [quality-gate-blocker] BLOCKED (0.3s)
```

---

## Future Hook Ideas

1. **Auto-assign reviewers** - Based on code ownership (CODEOWNERS)
2. **Slack status updates** - Show "Working on ENG-123" in Slack status
3. **Time tracking** - Automatically log hours to time tracking system
4. **Metrics collection** - Track workflow performance and bottlenecks
5. **Auto-rebase** - Keep feature branch up to date with main
6. **Dependency alerts** - Warn if using deprecated packages
