# Ticket Routing Logic

**Label-based routing system for intelligent workflow selection**

This document explains how the team-dev-workflow pack routes tickets based on their Linear labels.

---

## Routing Overview

```
Fetch Linear Ticket
        ↓
  Extract Labels
        ↓
   Detect Patterns
        ↓
   Set Routing Flags
        ↓
Select Workflow Path
```

---

## Routing Labels

| Label | Effect | Workflow Impact |
|-------|--------|-----------------|
| `pre-planned` | Skip discovery/architecture (Phases 1-4) | → Jump to Phase 5 (Implementation) |
| `UI`, `frontend`, `design` | Use frontend-design plugin | → Phase 5 uses frontend-design + ralph-loop |
| `backend`, `api` | Standard TDD implementation | → Phase 5 uses standard TDD + ralph-loop |
| `bug`, `hotfix` | Fast-track (skip if pre-planned) | → Phases 1-4 use faster exploration |
| `spike`, `research` | Discovery only (no implementation) | → Stop after Phase 4 (architecture) |

---

## Routing Flags

```typescript
interface RoutingFlags {
  // Skip Phases 1-4 (discovery & architecture)
  skip_phases_1_4: boolean;

  // Use frontend-design for Phase 5
  use_frontend_design: boolean;

  // Use fast-track workflow (reduced exploration)
  fast_track: boolean;

  // Stop after architecture (no implementation)
  architecture_only: boolean;

  // TDD enforcement level
  tdd_enforcement: 'strict' | 'standard' | 'flexible';
}
```

---

## Pattern: Extract and Normalize Labels

```typescript
const extractLabels = (issue: any): string[] => {
  if (!issue.labels || !Array.isArray(issue.labels)) {
    return [];
  }

  return issue.labels
    .map(label => label.name?.toLowerCase())
    .filter(Boolean);
};

// Usage
const labels = extractLabels(issue);
console.log(`Detected labels: ${labels.join(', ')}`);
```

---

## Pattern: Detect Routing Flags

```typescript
const detectRoutingFlags = (labels: string[]): RoutingFlags => {
  // Helper: check if label exists
  const hasLabel = (...labelsToCheck: string[]) =>
    labelsToCheck.some(l => labels.includes(l.toLowerCase()));

  return {
    skip_phases_1_4: hasLabel('pre-planned'),

    use_frontend_design: hasLabel('ui', 'frontend', 'design'),

    fast_track: hasLabel('bug', 'hotfix', 'urgent'),

    architecture_only: hasLabel('spike', 'research', 'exploration'),

    tdd_enforcement: hasLabel('bug', 'hotfix') ? 'strict' :
                     hasLabel('spike', 'research') ? 'flexible' :
                     'standard'
  };
};

// Usage
const flags = detectRoutingFlags(labels);
console.log(`Routing flags:`, flags);
```

---

## Pattern: Route to Workflow Path

```typescript
const routeWorkflow = (flags: RoutingFlags): WorkflowPath => {
  // 1. Architecture-only path
  if (flags.architecture_only) {
    return {
      phases: [0, 1, 2, 3, 4],  // Stop after architecture
      implementation: null,
      quality_gates: false,
      ship: false
    };
  }

  // 2. Pre-planned path
  if (flags.skip_phases_1_4) {
    return {
      phases: [0, 5, 6, 7, 8],  // Skip discovery/architecture
      implementation: flags.use_frontend_design ? 'frontend-design' : 'standard-tdd',
      quality_gates: true,
      ship: true
    };
  }

  // 3. Full workflow path
  return {
    phases: [0, 1, 2, 3, 4, 5, 6, 7, 8],  // All phases
    implementation: flags.use_frontend_design ? 'frontend-design' : 'standard-tdd',
    quality_gates: true,
    ship: true
  };
};

// Usage
const path = routeWorkflow(flags);
console.log(`Workflow path: ${path.phases.join(' → ')}`);
```

---

## Routing Decision Tree

```
START: Fetch ticket labels
   ↓
┌──────────────────────────┐
│ Has "spike" or "research"?│
└──────────────────────────┘
   ↓ YES                ↓ NO
   │                    │
   V                    V
Stop after         ┌────────────────┐
architecture       │Has "pre-planned"│
(Phases 0-4)       └────────────────┘
                      ↓ YES      ↓ NO
                      │          │
                      V          V
                  Skip to     Run full
                  Phase 5     discovery
                              (Phases 1-4)
                      │          │
                      └────┬─────┘
                           ↓
                    ┌──────────────────┐
                    │Has UI/frontend/  │
                    │design label?     │
                    └──────────────────┘
                       ↓ YES      ↓ NO
                       │          │
                       V          V
                   frontend-   Standard
                   design      TDD
                   Phase 5     Phase 5
                       │          │
                       └────┬─────┘
                            ↓
                    Quality Gates
                    (Phase 7)
                            ↓
                    Ship (Phase 8)
```

---

## Routing Scenarios

### Scenario 1: Pre-Planned UI Feature

```typescript
// Ticket: ENG-123
// Labels: ["pre-planned", "UI"]

const labels = ["pre-planned", "ui"];
const flags = detectRoutingFlags(labels);

// Results:
// - skip_phases_1_4: true
// - use_frontend_design: true
// - fast_track: false
// - architecture_only: false

// Workflow: 0 → 5 (frontend-design) → 6 → 7 → 8
```

### Scenario 2: Standard Backend Feature

```typescript
// Ticket: ENG-456
// Labels: ["backend", "api"]

const labels = ["backend", "api"];
const flags = detectRoutingFlags(labels);

// Results:
// - skip_phases_1_4: false
// - use_frontend_design: false
// - fast_track: false
// - architecture_only: false

// Workflow: 0 → 1 → 2 → 3 → 4 → 5 (standard TDD) → 6 → 7 → 8
```

### Scenario 3: Research Spike

```typescript
// Ticket: ENG-789
// Labels: ["spike", "research"]

const labels = ["spike", "research"];
const flags = detectRoutingFlags(labels);

// Results:
// - skip_phases_1_4: false
// - use_frontend_design: false
// - fast_track: false
// - architecture_only: true

// Workflow: 0 → 1 → 2 → 3 → 4 (STOP - no implementation)
```

### Scenario 4: Urgent Bug Fix

```typescript
// Ticket: ENG-111
// Labels: ["bug", "hotfix", "pre-planned"]

const labels = ["bug", "hotfix", "pre-planned"];
const flags = detectRoutingFlags(labels);

// Results:
// - skip_phases_1_4: true
// - use_frontend_design: false
// - fast_track: true
// - architecture_only: false
// - tdd_enforcement: 'strict'

// Workflow: 0 → 5 (strict TDD) → 6 → 7 → 8
```

---

## Pattern: Generate Routing Summary

```typescript
const generateRoutingSummary = (
  identifier: string,
  labels: string[],
  flags: RoutingFlags
): string => {
  const path = routeWorkflow(flags);

  return `
✅ Routing Analysis: ${identifier}

📋 DETECTED LABELS:
${labels.map(l => `- ${l}`).join('\n') || '- None'}

🔍 ROUTING FLAGS:
- Skip discovery/architecture: ${flags.skip_phases_1_4 ? 'YES' : 'NO'}
- Use frontend-design: ${flags.use_frontend_design ? 'YES' : 'NO'}
- Fast-track mode: ${flags.fast_track ? 'YES' : 'NO'}
- Architecture only: ${flags.architecture_only ? 'YES' : 'NO'}
- TDD enforcement: ${flags.tdd_enforcement}

⚡ WORKFLOW PATH:
Phases: ${path.phases.join(' → ')}
Implementation: ${path.implementation || 'N/A'}
Quality gates: ${path.quality_gates ? 'Enabled' : 'Disabled'}
Ship: ${path.ship ? 'Yes' : 'No'}
`.trim();
};

// Usage in Phase 0
console.log(generateRoutingSummary(issue.identifier, labels, flags));
```

---

## Pattern: Validate Routing Configuration

```typescript
const validateRouting = (flags: RoutingFlags): ValidationResult => {
  const errors: string[] = [];
  const warnings: string[] = [];

  // Cannot skip phases AND do architecture only
  if (flags.skip_phases_1_4 && flags.architecture_only) {
    errors.push("Conflicting flags: skip_phases_1_4 and architecture_only");
  }

  // Frontend design with architecture only doesn't make sense
  if (flags.use_frontend_design && flags.architecture_only) {
    warnings.push("Frontend design flag ignored for architecture-only workflow");
  }

  // Fast-track with architecture only is unusual
  if (flags.fast_track && flags.architecture_only) {
    warnings.push("Fast-track flag ignored for architecture-only workflow");
  }

  return {
    valid: errors.length === 0,
    errors,
    warnings
  };
};

// Usage
const validation = validateRouting(flags);
if (!validation.valid) {
  console.error(`❌ Invalid routing configuration:`, validation.errors);
  throw new Error("Cannot proceed with invalid routing");
}
if (validation.warnings.length > 0) {
  console.warn(`⚠️ Routing warnings:`, validation.warnings);
}
```

---

## Custom Routing Rules

Add custom routing logic based on project needs:

```typescript
const customRoutingRules = (
  labels: string[],
  issue: any
): Partial<RoutingFlags> => {
  const customFlags: Partial<RoutingFlags> = {};

  // High-priority bugs always fast-track
  if (issue.priority <= 2 && labels.includes('bug')) {
    customFlags.fast_track = true;
    customFlags.skip_phases_1_4 = true;
  }

  // UI bugs use frontend-design
  if (labels.includes('bug') && labels.includes('ui')) {
    customFlags.use_frontend_design = true;
  }

  // Documentation changes are architecture-only
  if (labels.includes('documentation')) {
    customFlags.architecture_only = true;
  }

  return customFlags;
};

// Merge with default routing
const baseFlags = detectRoutingFlags(labels);
const customFlags = customRoutingRules(labels, issue);
const finalFlags = { ...baseFlags, ...customFlags };
```

---

## Label Recommendations

**Recommended label structure:**

```yaml
# Workflow routing
pre-planned:
  description: "Architecture pre-planned, skip discovery"
  color: "#3b82f6"

# Feature type
ui:
  description: "User interface implementation"
  color: "#8b5cf6"

frontend:
  description: "Frontend code"
  color: "#8b5cf6"

backend:
  description: "Backend/API code"
  color: "#10b981"

# Urgency
bug:
  description: "Bug fix"
  color: "#ef4444"

hotfix:
  description: "Urgent production fix"
  color: "#dc2626"

# Research
spike:
  description: "Research/exploration spike"
  color: "#f59e0b"

research:
  description: "Research task"
  color: "#f59e0b"
```

---

## Testing Routing Logic

```typescript
// Test routing for different label combinations
const testRoutingScenarios = () => {
  const scenarios = [
    {
      name: "Pre-planned UI",
      labels: ["pre-planned", "ui"],
      expected: {
        skip_phases_1_4: true,
        use_frontend_design: true,
        phases: [0, 5, 6, 7, 8]
      }
    },
    {
      name: "Standard backend",
      labels: ["backend"],
      expected: {
        skip_phases_1_4: false,
        use_frontend_design: false,
        phases: [0, 1, 2, 3, 4, 5, 6, 7, 8]
      }
    },
    {
      name: "Research spike",
      labels: ["spike"],
      expected: {
        skip_phases_1_4: false,
        architecture_only: true,
        phases: [0, 1, 2, 3, 4]
      }
    }
  ];

  scenarios.forEach(scenario => {
    console.log(`\nTesting: ${scenario.name}`);
    const flags = detectRoutingFlags(scenario.labels);
    const path = routeWorkflow(flags);

    const passed =
      flags.skip_phases_1_4 === scenario.expected.skip_phases_1_4 &&
      (!scenario.expected.use_frontend_design || flags.use_frontend_design) &&
      (!scenario.expected.architecture_only || flags.architecture_only) &&
      JSON.stringify(path.phases) === JSON.stringify(scenario.expected.phases);

    console.log(passed ? "✅ PASS" : "❌ FAIL");
    if (!passed) {
      console.log("Expected:", scenario.expected);
      console.log("Got:", { flags, phases: path.phases });
    }
  });
};
```

---

## Integration with Configuration

**Load routing rules from config:**

```typescript
// Read from workflow-config.yaml
const routingConfig = {
  labels: {
    skip_discovery: ["pre-planned", "hotfix"],
    use_frontend_design: ["ui", "frontend", "design"],
    fast_track: ["bug", "hotfix", "urgent"],
    architecture_only: ["spike", "research", "exploration"]
  },
  defaults: {
    tdd_enforcement: "standard",
    quality_gates: true
  }
};

const detectRoutingFlagsFromConfig = (
  labels: string[],
  config: typeof routingConfig
): RoutingFlags => {
  const hasAnyLabel = (labelList: string[]) =>
    labelList.some(l => labels.includes(l.toLowerCase()));

  return {
    skip_phases_1_4: hasAnyLabel(config.labels.skip_discovery),
    use_frontend_design: hasAnyLabel(config.labels.use_frontend_design),
    fast_track: hasAnyLabel(config.labels.fast_track),
    architecture_only: hasAnyLabel(config.labels.architecture_only),
    tdd_enforcement: config.defaults.tdd_enforcement
  };
};
```

---

## Best Practices

1. **Keep routing simple** - Too many rules become hard to understand
2. **Document label meanings** - Clear descriptions in Linear
3. **Use consistent naming** - Lowercase, hyphenated labels
4. **Test routing logic** - Verify all label combinations work as expected
5. **Provide defaults** - Handle unlabeled tickets gracefully
6. **Log routing decisions** - Show user which path was chosen and why
7. **Allow overrides** - Let user manually override routing if needed

---

## Common Gotchas

1. **Case sensitivity** - Always normalize to lowercase
2. **Label vs identifier confusion** - Label = "ui", Identifier = "ENG-123"
3. **Conflicting labels** - "pre-planned" + "spike" doesn't make sense
4. **Missing labels** - Handle empty label arrays
5. **Multiple routing labels** - "ui" + "backend" - which takes precedence?

---

## Future Enhancements

1. **Project-based routing** - Different routing per Linear project
2. **Team-based routing** - Team-specific workflow customization
3. **Priority-based routing** - Urgent tickets skip phases
4. **Assignee-based routing** - Different paths for different team members
5. **Time-based routing** - Different routing during on-call/incidents
