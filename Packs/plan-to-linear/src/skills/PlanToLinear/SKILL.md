---
name: PlanToLinear
description: Transform feature ideas into structured Linear stories with dependencies. Activates when user wants to plan a feature, create Linear stories, or mentions "plan this feature", "I want to build", "create stories for".
---

# PlanToLinear - Feature Planning to Linear Stories

**Streamlined workflow from idea to pre-planned Linear tickets.**

Integrates:
- **feature-dev** - Backend architecture and design
- **frontend-design** - UI/UX design and component planning
- **superpowers:writing-plans** - Story decomposition with dependencies
- **Linear MCP** - Ticket creation with nested hierarchy

---

## Overview

Replaces ideate with a faster workflow that:
- Detects frontend/backend/both automatically
- Uses existing design skills (stops before implementation)
- Creates separate story groups for full-stack features
- Applies smart labels (pre-planned + concern + detected keywords)
- Maintains natural dependencies between stories

---

## Quick Start

```bash
# Command format
/plan-to-linear "feature idea description"

# Natural language (auto-triggers)
"I want to plan this feature"
"plan X for Linear"
"create stories for user authentication"
```

---

## Workflow Phases

### Phase 1: Detection & Confirmation
```
User provides feature idea
    ↓
Analyze keywords for frontend/backend indicators
    ↓
Present analysis: "I detected this needs [X] because..."
    ↓
Ask: "Does that sound right?"
    ↓
Adjust based on user feedback
```

**Detection Keywords:**
- **Frontend:** UI, page, component, form, design, styling, button, layout, responsive
- **Backend:** API, database, service, logic, authentication, data model, endpoint, schema

### Phase 2: Design
**Frontend only:**
- Invoke `frontend-design` skill
- Specify: "Design phase only, stop before implementation"
- Save: `docs/features/YYYY-MM-DD-slug/frontend/design.md`

**Backend only:**
- Invoke `feature-dev` skill
- Specify backend focus
- Save: `docs/features/YYYY-MM-DD-slug/backend/design.md`

**Both (Full-Stack):**
- Run `frontend-design` first (UI/UX)
- Run `feature-dev` second (backend architecture)
- Save to respective folders

### Phase 3: Story Decomposition
**Uses `superpowers:writing-plans` skill:**

- Frontend only: Decompose UI design → `frontend/stories.md`
- Backend only: Decompose API design → `backend/stories.md`
- Both: Run twice, analyze cross-dependencies

**Story Format:**
- Title
- Summary/Description
- Acceptance Criteria (checkboxes)
- Technical Notes
- Dependencies (blocked by X, blocks Y)
- Suggested labels (from smart analysis)

### Phase 4: Linear Upload
**Creates nested hierarchy:**

```
Parent: "Feature: [name]" (labels: [pre-planned])
├─ Sub-parent: "Frontend: [name]" (labels: [pre-planned, frontend])
│   ├─ Story 1 (labels: [pre-planned, frontend, forms])
│   └─ Story 2 (labels: [pre-planned, frontend, api])
└─ Sub-parent: "Backend: [name]" (labels: [pre-planned, backend])
    ├─ Story 1 (labels: [pre-planned, backend, database, authentication])
    └─ Story 2 (labels: [pre-planned, backend, api])
```

**Blocking Relations:**
- Uses Linear's native `blockedBy` fields
- Cross-concern: Frontend stories blocked by Backend stories
- Within-concern: Sequential dependencies within frontend or backend

---

## Smart Labeling Strategy

### Base Labels (Always Applied)
- Parent issue: `["pre-planned"]`
- Frontend sub-parent + stories: `["pre-planned", "frontend"]`
- Backend sub-parent + stories: `["pre-planned", "backend"]`

### Smart Analysis Labels (Auto-Detected)
Scans story title, description, and technical notes for keywords:

| Label | Keywords |
|-------|----------|
| `database` | db, schema, migration, query, sql, postgres, mysql, mongo |
| `authentication` | auth, login, session, permission, jwt, oauth, rbac |
| `api` | endpoint, route, rest, graphql, request, response |
| `forms` | form, validation, input, submit, field, error handling |
| `testing` | test, spec, e2e, unit, integration, coverage |
| `security` | security, vulnerability, encryption, sanitize, xss, csrf |
| `performance` | performance, optimize, cache, lazy, memoize, debounce |
| `documentation` | docs, documentation, readme, guide, comment |

**Example:**
Story: "Create user login endpoint with JWT authentication"
→ Detected labels: `["pre-planned", "backend", "api", "authentication", "security"]`

---

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

# Plan to Linear Configuration

This file configures the plan-to-linear pack for this project.
```

---

## Usage Examples

### Example 1: Frontend-Only Feature
```
User: "/plan-to-linear Add user profile dashboard with avatar upload"

Flow:
✅ Phase 1: Detect "dashboard", "upload" → Frontend only
✅ Phase 2: Run frontend-design (design phase only)
✅ Phase 3: Decompose into stories
✅ Phase 4: Upload to Linear
  └─ Parent: "Feature: User Profile Dashboard"
      └─ Sub-parent: "Frontend: User Profile Dashboard"
          ├─ Create profile layout component (labels: [pre-planned, frontend])
          ├─ Implement avatar upload widget (labels: [pre-planned, frontend, forms])
          └─ Add responsive styling (labels: [pre-planned, frontend])
```

### Example 2: Backend-Only Feature
```
User: "I want to plan a rate limiting system for API endpoints"

Flow:
✅ Phase 1: Detect "API", "rate limiting" → Backend only
✅ Phase 2: Run feature-dev (backend focus, design only)
✅ Phase 3: Decompose into stories
✅ Phase 4: Upload to Linear
  └─ Parent: "Feature: Rate Limiting System"
      └─ Sub-parent: "Backend: Rate Limiting System"
          ├─ Design rate limit storage (labels: [pre-planned, backend, database])
          ├─ Create middleware (labels: [pre-planned, backend, api, performance])
          └─ Add monitoring (labels: [pre-planned, backend, performance])
```

### Example 3: Full-Stack Feature
```
User: "create stories for user authentication with JWT"

Flow:
✅ Phase 1: Detect "authentication" + implicit UI → Both
✅ Phase 2a: Run frontend-design (login UI)
✅ Phase 2b: Run feature-dev (auth backend)
✅ Phase 3: Decompose both designs, analyze dependencies
✅ Phase 4: Upload to Linear with blocking relations
  └─ Parent: "Feature: User Authentication"
      ├─ Sub-parent: "Frontend: User Authentication"
      │   ├─ Login form component (blocked by: Backend JWT endpoint)
      │   └─ Auth state management (blocked by: Backend JWT endpoint)
      └─ Sub-parent: "Backend: User Authentication"
          ├─ JWT token generation (labels: [pre-planned, backend, authentication, security])
          ├─ Auth middleware (labels: [pre-planned, backend, api, security])
          └─ User session storage (labels: [pre-planned, backend, database])
```

---

## Artifact Organization

All designs and stories saved to:
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

---

## Dependencies

| Dependency | Purpose |
|------------|---------|
| `feature-dev` | Backend architecture and design |
| `frontend-design` | UI/UX design and component planning |
| `superpowers:writing-plans` | Story decomposition with dependencies |
| Linear MCP server | Linear API integration |

---

## Integration with PAI Systems

| PAI System | Integration |
|------------|-------------|
| **CORE Routing** | "/plan-to-linear" and natural language triggers |
| **History** | Captures all planning sessions for reference |
| **Voice** | Announces phase transitions |
| **Response Format** | Uses 📋 SUMMARY, 🔍 ANALYSIS, ✅ RESULTS, 🎯 COMPLETED |

---

## Design Principles

1. **Smart detection** - Analyze requirements, confirm with user
2. **Stop before implementation** - Design phase only, no code
3. **Natural dependencies** - Let writing-plans determine blocking relations
4. **Smart labeling** - Base labels + keyword-based detection
5. **Nested hierarchy** - Umbrella parent → concern sub-parents → stories
6. **Skill composition** - Orchestrate existing skills, don't reimplement

---

## Installation

See `INSTALL.md` in pack root.

## Verification

See `VERIFY.md` in pack root.
