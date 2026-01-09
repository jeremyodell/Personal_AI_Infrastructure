# Linear MCP Helper Patterns

**Reference guide for Linear MCP integration patterns**

This document provides reusable patterns for working with Linear tickets via the MCP integration.

---

## Available Linear MCP Tools

```typescript
// Ticket operations
mcp__linear__get_issue(params)
mcp__linear__list_issues(params)
mcp__linear__create_issue(params)
mcp__linear__update_issue(params)

// Comments
mcp__linear__list_comments(params)
mcp__linear__create_comment(params)

// Labels
mcp__linear__list_issue_labels(params)
mcp__linear__create_issue_label(params)

// Teams & Users
mcp__linear__list_teams(params)
mcp__linear__get_team(params)
mcp__linear__list_users(params)
mcp__linear__get_user(params)

// Projects
mcp__linear__list_projects(params)
mcp__linear__get_project(params)
```

---

## Pattern: Fetch Ticket with Full Context

```typescript
const issue = await mcp__linear__get_issue({
  id: ticket_id,  // Can be UUID or identifier (ENG-123)
  includeRelations: false  // Set true if you need blocking/blocked-by relations
});

// Extract commonly used fields
const {
  id,              // UUID
  identifier,      // "ENG-123"
  title,           // "Add user dashboard"
  description,     // Full markdown description
  labels,          // Array of label objects
  assignee,        // User object or null
  priority,        // 0-4 (0=none, 1=urgent, 2=high, 3=normal, 4=low)
  state,           // State object { name, type }
  project,         // Project object or null
  team             // Team object
} = issue;
```

---

## Pattern: Extract Label Names

```typescript
// Labels come as objects: { id, name, color, ... }
const labelNames = labels
  .map(label => label.name.toLowerCase())
  .filter(Boolean);  // Remove any null/undefined

console.log(`Labels: ${labelNames.join(', ')}`);
// Example output: "Labels: ui, pre-planned, frontend"
```

---

## Pattern: Check for Specific Labels

```typescript
// Check for routing labels
const hasLabel = (labelName: string) =>
  labelNames.some(name => name === labelName.toLowerCase());

const isPrePlanned = hasLabel('pre-planned');
const isUI = ['ui', 'frontend', 'design'].some(label => hasLabel(label));
const isBackend = hasLabel('backend');
const isBug = hasLabel('bug');
```

---

## Pattern: Update Ticket Status

```typescript
// Update to "In Progress" when starting work
await mcp__linear__update_issue({
  id: ticket_id,
  state: "In Progress"
});

// Update to "In Review" when PR created
await mcp__linear__update_issue({
  id: ticket_id,
  state: "In Review"
});

// Update to "Done" when merged (usually automated)
await mcp__linear__update_issue({
  id: ticket_id,
  state: "Done"
});
```

**Available state types:**
- `"Backlog"`
- `"Todo"`
- `"In Progress"`
- `"In Review"`
- `"Done"`
- `"Canceled"`

---

## Pattern: Post Progress Updates

```typescript
// Post architecture summary
await mcp__linear__create_comment({
  issueId: ticket_id,
  body: `## Architecture Design Complete

### Approach Chosen:
${approach_name}

### Key Decisions:
- ${decision1}
- ${decision2}

### Files to Modify:
- ${file1}
- ${file2}

Ready to proceed with implementation.
`
});

// Post implementation complete
await mcp__linear__create_comment({
  issueId: ticket_id,
  body: `## Implementation Complete ✅

### Summary:
${implementation_summary}

### Test Coverage:
${coverage}%

### Quality Gates:
- ✅ Tests: 0 failures
- ✅ Linting: 0 errors
- ✅ Type checking: 0 errors
- ✅ Build: successful

Ready for code review.
`
});

// Post PR link
await mcp__linear__create_comment({
  issueId: ticket_id,
  body: `## Pull Request Created 🎉

**PR:** ${pr_url}

Feature ready for team review!
`
});
```

---

## Pattern: Search for Tickets

```typescript
// List tickets assigned to current user
const myIssues = await mcp__linear__list_issues({
  assignee: "me",
  state: "In Progress",
  limit: 50
});

// List tickets by label
const uiTickets = await mcp__linear__list_issues({
  label: "UI",
  state: "Todo",
  limit: 20
});

// List tickets by project
const projectTickets = await mcp__linear__list_issues({
  project: "Q1 2026 Roadmap",
  team: "Engineering",
  limit: 50
});
```

---

## Pattern: Get Team Information

```typescript
// Get team by name or ID
const team = await mcp__linear__get_team({
  query: "Engineering"  // Can be name or UUID
});

const teamId = team.id;
const teamKey = team.key;  // Used in identifiers (e.g., "ENG")
```

---

## Pattern: List Available Labels

```typescript
// List all workspace labels
const labels = await mcp__linear__list_issue_labels({
  team: "Engineering",  // Optional: filter by team
  limit: 100
});

// Extract label names
const availableLabels = labels.map(l => l.name);
console.log(`Available labels: ${availableLabels.join(', ')}`);
```

---

## Pattern: Create New Label

```typescript
// Create team-specific label
await mcp__linear__create_issue_label({
  name: "pre-planned",
  description: "Architecture is pre-planned, skip discovery phases",
  color: "#3b82f6",  // Blue
  teamId: team_id    // Optional: null for workspace label
});

// Create UI label
await mcp__linear__create_issue_label({
  name: "UI",
  description: "User interface implementation",
  color: "#8b5cf6",  // Purple
  teamId: team_id
});
```

---

## Pattern: Handle Ticket Not Found

```typescript
try {
  const issue = await mcp__linear__get_issue({
    id: ticket_id
  });
} catch (error) {
  console.error(`❌ ERROR: Ticket ${ticket_id} not found

Possible causes:
- Ticket ID is incorrect (check format: ENG-123)
- Ticket doesn't exist in Linear
- No permission to access this ticket
- Linear API connection issue

Please verify the ticket ID and try again.`);
  throw error;
}
```

---

## Pattern: Batch Operations

```typescript
// Get multiple tickets
const ticketIds = ["ENG-123", "ENG-124", "ENG-125"];

const tickets = await Promise.all(
  ticketIds.map(id =>
    mcp__linear__get_issue({ id, includeRelations: false })
  )
);

// Post updates to multiple tickets
await Promise.all(
  tickets.map(ticket =>
    mcp__linear__create_comment({
      issueId: ticket.id,
      body: "Bulk update: All tickets reviewed"
    })
  )
);
```

---

## Pattern: Extract Acceptance Criteria

```typescript
// Parse acceptance criteria from description
const extractAcceptanceCriteria = (description: string): string[] => {
  // Look for "Acceptance Criteria" section
  const match = description.match(/## Acceptance Criteria\s+([\s\S]*?)(?=##|$)/i);

  if (!match) return [];

  // Extract bullet points
  const criteria = match[1]
    .split('\n')
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('*'))
    .map(line => line.replace(/^[\s-*]+/, '').trim())
    .filter(Boolean);

  return criteria;
};

// Usage
const criteria = extractAcceptanceCriteria(issue.description);
console.log(`Acceptance Criteria:\n${criteria.map((c, i) => `${i + 1}. ${c}`).join('\n')}`);
```

---

## Pattern: Slugify Title for Branch Name

```typescript
const slugifyTitle = (title: string): string => {
  return title
    .toLowerCase()
    .replace(/[^a-z0-9\s-]/g, '')  // Remove special chars
    .replace(/\s+/g, '-')           // Spaces to hyphens
    .replace(/-+/g, '-')            // Collapse multiple hyphens
    .replace(/^-+|-+$/g, '');       // Trim hyphens
};

// Usage
const slug = slugifyTitle(issue.title);
const branchName = `feat/${issue.identifier}-${slug}`;
console.log(`Branch: ${branchName}`);
// Example: feat/ENG-123-add-user-dashboard
```

---

## Pattern: Format Priority for Display

```typescript
const formatPriority = (priority: number): string => {
  const priorities = {
    0: "No priority",
    1: "🔥 Urgent",
    2: "⬆️ High",
    3: "➡️ Normal",
    4: "⬇️ Low"
  };
  return priorities[priority] || "Unknown";
};

// Usage
console.log(`Priority: ${formatPriority(issue.priority)}`);
```

---

## Pattern: Generate Ticket Summary

```typescript
const generateTicketSummary = (issue: any): string => {
  const labelNames = issue.labels.map(l => l.name).join(', ');

  return `
📋 Ticket Summary: ${issue.identifier}

Title: ${issue.title}
Status: ${issue.state.name}
Priority: ${formatPriority(issue.priority)}
Labels: ${labelNames || 'None'}
Assignee: ${issue.assignee?.name || 'Unassigned'}
Team: ${issue.team.name}
Project: ${issue.project?.name || 'None'}

Description:
${issue.description || 'No description'}
`.trim();
};

// Usage
console.log(generateTicketSummary(issue));
```

---

## Pattern: Check if Ready to Ship

```typescript
const isReadyToShip = (issue: any): boolean => {
  // Check required fields
  if (!issue.description) {
    console.log("❌ Missing: Description");
    return false;
  }

  if (!issue.assignee) {
    console.log("❌ Missing: Assignee");
    return false;
  }

  if (issue.state.name === "Backlog" || issue.state.name === "Todo") {
    console.log("❌ Not started yet");
    return false;
  }

  return true;
};

// Usage
if (!isReadyToShip(issue)) {
  console.log("Ticket not ready to work on yet");
}
```

---

## Error Handling Patterns

```typescript
// Wrap Linear calls with error handling
const safeLinearCall = async <T>(
  operation: () => Promise<T>,
  errorMessage: string
): Promise<T | null> => {
  try {
    return await operation();
  } catch (error) {
    console.error(`❌ ${errorMessage}:`, error);
    return null;
  }
};

// Usage
const issue = await safeLinearCall(
  () => mcp__linear__get_issue({ id: ticket_id }),
  "Failed to fetch ticket"
);

if (!issue) {
  console.log("Cannot proceed without ticket data");
  return;
}
```

---

## Rate Limiting

Linear API has rate limits. Handle gracefully:

```typescript
const delay = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

const linearCallWithRetry = async <T>(
  operation: () => Promise<T>,
  maxRetries = 3
): Promise<T> => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await operation();
    } catch (error) {
      if (error.message.includes('rate limit')) {
        console.log(`Rate limited, retrying in ${(i + 1) * 1000}ms...`);
        await delay((i + 1) * 1000);
      } else {
        throw error;
      }
    }
  }
  throw new Error(`Failed after ${maxRetries} retries`);
};
```

---

## Testing Linear Integration

```typescript
// Test Linear connection
const testLinearConnection = async (): Promise<boolean> => {
  try {
    const teams = await mcp__linear__list_teams({ limit: 1 });
    console.log(`✅ Linear connected (${teams.length} team(s) found)`);
    return true;
  } catch (error) {
    console.error("❌ Linear connection failed:", error);
    return false;
  }
};

// Usage in workflow
const connected = await testLinearConnection();
if (!connected) {
  console.log("Please configure Linear MCP integration first");
  return;
}
```

---

## Best Practices

1. **Always use identifier over ID** - Identifiers (ENG-123) are human-readable
2. **Handle nulls gracefully** - Many fields can be null (assignee, project, etc.)
3. **Use batch operations** - When fetching multiple tickets
4. **Cache team/user data** - Don't fetch repeatedly
5. **Post meaningful updates** - Keep Linear as source of truth
6. **Use markdown formatting** - Linear supports rich markdown in comments
7. **Handle errors explicitly** - Linear API can fail, always have fallbacks
8. **Respect rate limits** - Implement backoff/retry logic

---

## Common Gotchas

1. **State names are case-sensitive** - "In Progress" not "in progress"
2. **Labels are objects not strings** - Must extract .name property
3. **Ticket ID vs Identifier** - ID is UUID, identifier is "ENG-123"
4. **Team required for creation** - Can't create issues without team
5. **Relations are separate query** - Set includeRelations: true if needed
