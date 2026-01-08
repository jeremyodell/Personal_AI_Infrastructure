# plan-to-linear Pack Design

**Date:** 2026-01-08
**Purpose:** Transform feature ideas into structured Linear stories with dependencies

## Problem Statement

The ideate pack transforms ideas into Linear stories through five rigid phases. This design replaces ideate with a streamlined workflow that:

- Uses existing feature-dev and frontend-design skills
- Handles full-stack features with separate story groups
- Stops before implementation (creates pre-planned stories)
- Applies smart labels beyond basic concern tagging

## Workflow

```
User provides feature idea
    ↓
Analyze and detect (frontend/backend/both)
    ↓
Confirm detection with user
    ↓
Design phase (feature-dev and/or frontend-design)
    ↓
Story decomposition (writing-plans skill)
    ↓
Upload to Linear (nested hierarchy with smart labels)
```

## Detection Phase

The skill analyzes the feature idea to determine what's needed:

- **Frontend indicators:** UI, page, component, form, design, styling
- **Backend indicators:** API, database, service, logic, authentication, data model

**Process:**
1. Read the feature idea
2. Identify required concerns
3. Present analysis: "I detected this needs [frontend/backend/both] because..."
4. Ask: "Does that sound right?"
5. Adjust based on user correction

## Design Phase

### Frontend Only
- Invoke `frontend-design` skill
- Specify: "Design phase only, stop before implementation"
- Capture UI design, component structure, mockups
- Save to: `docs/features/YYYY-MM-DD-slug/frontend/design.md`

### Backend Only
- Invoke `feature-dev` skill
- Specify backend focus
- Capture architecture, API design, data models
- Save to: `docs/features/YYYY-MM-DD-slug/backend/design.md`

### Both (Full-Stack)
- Run `frontend-design` first → captures UI/UX
- Run `feature-dev` second → captures backend architecture
- Save to respective folders

**Design Requirements:**
- Each skill produces detailed design/plan
- Stop explicitly before implementation
- Provide enough detail for story decomposition

## Story Decomposition

Invoke `superpowers:writing-plans` to decompose designs into stories.

### Frontend Only
- Input: frontend design document
- Output: Stories for UI implementation
- Save: `frontend/stories.md`

### Backend Only
- Input: backend design document
- Output: Stories for API/service implementation
- Save: `backend/stories.md`

### Both (Full-Stack)
- Run writing-plans twice (once per design)
- Frontend stories → `frontend/stories.md`
- Backend stories → `backend/stories.md`
- Analyze cross-concern dependencies

**Natural Dependencies:**
The writing-plans skill identifies:
- Sequential dependencies within each concern
- Cross-concern dependencies (Frontend story needs Backend story)
- Parallel-safe work (independent stories)

**Story Format:**
- Title
- Summary/Description
- Acceptance Criteria (checkboxes)
- Technical Notes
- Dependencies (blocked by X, blocks Y)
- Suggested labels (smart analysis)

## Linear Upload Structure

### Frontend Only
```
Parent: "Feature: [name]"
└─ Sub-parent: "Frontend: [name]"
    ├─ Story 1
    ├─ Story 2
    └─ Story 3
```

### Backend Only
```
Parent: "Feature: [name]"
└─ Sub-parent: "Backend: [name]"
    ├─ Story 1
    ├─ Story 2
    └─ Story 3
```

### Both (Full-Stack)
```
Parent: "Feature: [name]"
├─ Sub-parent: "Frontend: [name]"
│   ├─ Story 1
│   └─ Story 2
└─ Sub-parent: "Backend: [name]"
    ├─ Story 1
    └─ Story 2
```

**Blocking Relations:**
- Uses Linear's native "blocked by" fields
- Frontend stories needing backend: `blockedBy: ["Backend-Story-ID"]`
- Within-concern dependencies: `blockedBy: ["Same-Concern-Story-ID"]`

## Label Strategy

### Base Labels (Always Applied)
- Parent issue: `["pre-planned"]`
- Frontend sub-parent + stories: `["pre-planned", "frontend"]`
- Backend sub-parent + stories: `["pre-planned", "backend"]`

### Smart Analysis Labels
The skill analyzes each story's content and adds relevant labels:

- `"database"` - DB schema, queries, migrations
- `"authentication"` - Auth, permissions, sessions
- `"api"` - API endpoint work
- `"forms"` - Form handling, validation
- `"testing"` - Test infrastructure
- `"security"` - Security considerations
- `"performance"` - Performance optimization
- `"documentation"` - Documentation work

**Detection Method:**
Scan story title, description, technical notes for keywords:
- database: ["db", "schema", "migration", "query", "sql"]
- authentication: ["auth", "login", "session", "permission", "jwt"]
- forms: ["form", "validation", "input", "submit"]
- testing: ["test", "spec", "e2e", "unit"]

## Configuration

**File:** `.claude/plan-to-linear.local.md`

```yaml
---
linear_team: "Engineering"
linear_project: "Product Development"
default_labels:
  - "pre-planned"
frontend_labels:
  - "frontend"
  - "ui"
backend_labels:
  - "backend"
  - "api"
smart_label_keywords:
  database: ["db", "schema", "migration", "query", "sql"]
  authentication: ["auth", "login", "session", "permission", "jwt"]
  forms: ["form", "validation", "input", "submit"]
  testing: ["test", "spec", "e2e", "unit"]
  security: ["security", "vulnerability", "encryption", "sanitize"]
  performance: ["performance", "optimize", "cache", "lazy"]
  documentation: ["docs", "documentation", "readme", "guide"]
---
```

## Artifact Organization

```
docs/features/YYYY-MM-DD-feature-slug/
├── frontend/
│   ├── design.md          (from frontend-design skill)
│   └── stories.md         (from writing-plans)
├── backend/
│   ├── design.md          (from feature-dev skill)
│   └── stories.md         (from writing-plans)
└── linear-summary.md      (upload results, issue IDs, links)
```

## Triggering

### Command
`/plan-to-linear "feature idea"`

### Natural Language
- "I want to plan this feature"
- "plan X for Linear"
- "create stories for Y"

### Skill Description
```
Transform feature ideas into structured Linear stories with dependencies.
Activates when user wants to plan a feature, create Linear stories, or mentions
"plan this feature", "I want to build", "create stories for".
```

## Implementation Notes

### Dependencies
- `feature-dev` plugin (for backend design)
- `frontend-design` plugin (for UI design)
- `superpowers` plugin (writing-plans skill)
- Linear MCP server

### Key Behaviors
1. Always confirm detection with user
2. Stop design phase before implementation
3. Run writing-plans separately for each concern
4. Apply both base labels and smart labels
5. Use Linear's native blocking relations
6. Save comprehensive artifacts for reference

### Success Criteria
- Creates pre-planned Linear stories
- Handles full-stack features with separate groups
- Applies smart labels automatically
- Maintains natural dependencies
- Streamlines workflow compared to ideate
