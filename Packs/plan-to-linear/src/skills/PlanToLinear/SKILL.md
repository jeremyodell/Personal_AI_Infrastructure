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
