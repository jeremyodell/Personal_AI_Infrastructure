# plan-to-linear Pack Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create plan-to-linear pack that transforms feature ideas into Linear stories using feature-dev/frontend-design skills and smart labeling.

**Architecture:** Skill-based pack that orchestrates existing skills (feature-dev, frontend-design, writing-plans) to analyze requirements, generate designs, decompose into stories, and upload to Linear with smart labels.

**Tech Stack:** Markdown-based skill file with YAML frontmatter, Linear MCP for API integration

---

## Task 1: Create Pack Directory Structure

**Files:**
- Create: `Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md`
- Create: `Packs/plan-to-linear/README.md`
- Create: `Packs/plan-to-linear/INSTALL.md`
- Create: `Packs/plan-to-linear/VERIFY.md`

**Step 1: Create directory structure**

```bash
mkdir -p Packs/plan-to-linear/src/skills/PlanToLinear
```

**Step 2: Verify directories created**

Run: `ls -la Packs/plan-to-linear/src/skills/PlanToLinear`
Expected: Directory exists

**Step 3: Commit directory structure**

```bash
git add Packs/plan-to-linear/
git commit -m "chore: create plan-to-linear pack structure"
```

---

## Task 2: Write SKILL.md Core Structure

**Files:**
- Create: `Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md`

**Step 1: Write YAML frontmatter and skill header**

```markdown
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
```

**Step 2: Verify file created**

Run: `cat Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md | head -20`
Expected: Shows frontmatter and header

**Step 3: Commit SKILL.md foundation**

```bash
git add Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md
git commit -m "feat: add SKILL.md foundation for plan-to-linear"
```

---

## Task 3: Add Workflow Documentation to SKILL.md

**Files:**
- Modify: `Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md` (append)

**Step 1: Add workflow phases section**

```markdown
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
```

**Step 2: Verify workflow section added**

Run: `grep -A 5 "## Workflow Phases" Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md`
Expected: Shows workflow phases header and content

**Step 3: Commit workflow documentation**

```bash
git add Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md
git commit -m "docs: add workflow phases to plan-to-linear SKILL.md"
```

---

## Task 4: Add Smart Labeling Documentation

**Files:**
- Modify: `Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md` (append)

**Step 1: Add labeling strategy section**

```markdown
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
```

**Step 2: Verify labeling section**

Run: `grep -A 3 "## Smart Labeling Strategy" Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md`
Expected: Shows labeling section

**Step 3: Commit labeling documentation**

```bash
git add Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md
git commit -m "docs: add smart labeling strategy to plan-to-linear"
```

---

## Task 5: Add Configuration and Usage Examples

**Files:**
- Modify: `Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md` (append)

**Step 1: Add configuration section**

```markdown
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
```

**Step 2: Verify configuration and examples**

Run: `tail -30 Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md`
Expected: Shows configuration and examples

**Step 3: Commit configuration and examples**

```bash
git add Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md
git commit -m "docs: add configuration and usage examples"
```

---

## Task 6: Complete SKILL.md with Integration Details

**Files:**
- Modify: `Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md` (append)

**Step 1: Add integration and artifacts sections**

```markdown
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
```

**Step 2: Verify SKILL.md is complete**

Run: `wc -l Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md`
Expected: ~250+ lines

**Step 3: Commit complete SKILL.md**

```bash
git add Packs/plan-to-linear/src/skills/PlanToLinear/SKILL.md
git commit -m "feat: complete SKILL.md with integration details"
```

---

## Task 7: Write README.md

**Files:**
- Create: `Packs/plan-to-linear/README.md`

**Step 1: Write README content**

```markdown
# plan-to-linear Pack

**Transform feature ideas into structured Linear stories with dependencies.**

Streamlined workflow that:
- Detects frontend/backend/both automatically
- Uses feature-dev and frontend-design for design phase
- Stops before implementation (creates pre-planned stories)
- Applies smart labels automatically
- Handles full-stack features with separate story groups

---

## What It Does

1. **Analyzes** your feature idea to detect frontend, backend, or both
2. **Designs** using existing skills (feature-dev, frontend-design)
3. **Decomposes** designs into stories with natural dependencies
4. **Uploads** to Linear with nested hierarchy and smart labels

---

## Quick Start

```bash
# Command format
/plan-to-linear "Add user authentication with JWT"

# Natural language
"I want to plan this feature"
"create stories for user dashboard"
```

---

## Example Output

### Input
```
/plan-to-linear "Add user authentication with JWT"
```

### Linear Structure Created
```
Parent: "Feature: User Authentication" [pre-planned]
├─ Frontend: "User Authentication" [pre-planned, frontend]
│   ├─ Login form component [pre-planned, frontend, forms]
│   └─ Auth state management [pre-planned, frontend]
└─ Backend: "User Authentication" [pre-planned, backend]
    ├─ JWT generation endpoint [pre-planned, backend, api, authentication, security]
    ├─ Auth middleware [pre-planned, backend, api, security]
    └─ User session storage [pre-planned, backend, database]
```

---

## Features

✅ **Smart Detection** - Automatically identifies frontend/backend needs
✅ **Design-Only** - Stops before implementation, creates pre-planned tickets
✅ **Smart Labels** - Base labels + keyword detection (database, auth, forms, etc.)
✅ **Natural Dependencies** - writing-plans determines blocking relations
✅ **Full-Stack Support** - Separate story groups with cross-dependencies

---

## Installation

See [INSTALL.md](./INSTALL.md)

## Verification

See [VERIFY.md](./VERIFY.md)

---

## Configuration

Create `.claude/plan-to-linear.local.md`:

```yaml
---
linear_team: "Engineering"
linear_project: "Product Development"
default_labels:
  - "pre-planned"
frontend_labels:
  - "frontend"
backend_labels:
  - "backend"
---
```

---

## Dependencies

- `feature-dev` plugin
- `frontend-design` plugin
- `superpowers` plugin (writing-plans skill)
- Linear MCP server

---

## License

MIT
```

**Step 2: Verify README created**

Run: `cat Packs/plan-to-linear/README.md | head -30`
Expected: Shows README header and overview

**Step 3: Commit README**

```bash
git add Packs/plan-to-linear/README.md
git commit -m "docs: add README for plan-to-linear pack"
```

---

## Task 8: Write INSTALL.md

**Files:**
- Create: `Packs/plan-to-linear/INSTALL.md`

**Step 1: Write installation instructions**

```markdown
# plan-to-linear Installation

## Prerequisites

Ensure you have:
- ✅ Claude Code installed
- ✅ Linear MCP server configured
- ✅ `feature-dev` plugin installed
- ✅ `frontend-design` plugin installed
- ✅ `superpowers` plugin installed

---

## Installation Steps

### 1. Install Pack

```bash
# Copy pack to your .claude/plugins directory
cp -r Packs/plan-to-linear ~/.claude/plugins/plan-to-linear
```

### 2. Verify Skill Loaded

Start Claude Code and check:

```bash
claude
# In Claude session, type:
/help
```

Expected: See `plan-to-linear` in available commands

### 3. Configure Linear Settings

Create `.claude/plan-to-linear.local.md`:

```yaml
---
linear_team: "Your Team Name"
linear_project: "Your Project Name"
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

Customize labels and keywords for your team.
```

### 4. Test Installation

Run:

```bash
claude
# In Claude session:
/plan-to-linear "Add a simple contact form"
```

Expected: Claude analyzes the idea, detects frontend requirement, and begins planning.

---

## Configuration Options

### Linear Team

Find your team name:
```bash
# In Claude session with Linear MCP:
list Linear teams
```

### Linear Project

Find your project name:
```bash
# In Claude session with Linear MCP:
list Linear projects in <team-name>
```

### Custom Labels

Add project-specific labels to `smart_label_keywords`:

```yaml
smart_label_keywords:
  database: ["db", "schema", "migration"]
  myproject_specific: ["special", "keywords"]
```

---

## Troubleshooting

### "Linear MCP not found"

**Problem:** Linear MCP server not configured

**Solution:**
1. Install Linear MCP: `npm install -g @linear/mcp`
2. Configure in `.claude/settings.json`:
```json
{
  "mcpServers": {
    "linear": {
      "command": "linear-mcp",
      "env": {
        "LINEAR_API_KEY": "your-api-key"
      }
    }
  }
}
```

### "feature-dev skill not found"

**Problem:** feature-dev plugin not installed

**Solution:** Install from marketplace or manually

### "Permission denied creating Linear issues"

**Problem:** Linear API key lacks permissions

**Solution:** Regenerate API key with "Write" permissions in Linear settings

---

## Uninstallation

```bash
rm -rf ~/.claude/plugins/plan-to-linear
rm .claude/plan-to-linear.local.md
```

---

## Next Steps

After installation:
1. ✅ Run verification (see VERIFY.md)
2. ✅ Create your first planned feature
3. ✅ Check Linear for created tickets
4. ✅ Customize labels for your team
```

**Step 2: Verify INSTALL.md**

Run: `cat Packs/plan-to-linear/INSTALL.md | grep "## Prerequisites" -A 5`
Expected: Shows prerequisites section

**Step 3: Commit INSTALL.md**

```bash
git add Packs/plan-to-linear/INSTALL.md
git commit -m "docs: add installation instructions"
```

---

## Task 9: Write VERIFY.md

**Files:**
- Create: `Packs/plan-to-linear/VERIFY.md`

**Step 1: Write verification checklist**

```markdown
# plan-to-linear Verification

Complete this checklist to verify installation.

---

## Phase 1: Skill Loading

### Test 1.1: Skill Available

**Action:** Start Claude Code, type `/plan-to-linear`

**Expected:** Claude recognizes command and asks for feature idea

**Status:** ⬜ Pass ⬜ Fail

---

## Phase 2: Detection Logic

### Test 2.1: Frontend Detection

**Action:**
```bash
/plan-to-linear "Create a user registration form with email validation"
```

**Expected:**
- ✅ Detects: Frontend only
- ✅ Shows reasoning: "form", "validation" keywords
- ✅ Asks for confirmation

**Status:** ⬜ Pass ⬜ Fail

### Test 2.2: Backend Detection

**Action:**
```bash
/plan-to-linear "Add rate limiting middleware to API endpoints"
```

**Expected:**
- ✅ Detects: Backend only
- ✅ Shows reasoning: "API", "middleware" keywords
- ✅ Asks for confirmation

**Status:** ⬜ Pass ⬜ Fail

### Test 2.3: Full-Stack Detection

**Action:**
```bash
/plan-to-linear "Build user authentication with login page and JWT tokens"
```

**Expected:**
- ✅ Detects: Both frontend and backend
- ✅ Shows reasoning: "login page" (frontend) + "JWT" (backend)
- ✅ Asks for confirmation

**Status:** ⬜ Pass ⬜ Fail

---

## Phase 3: Design Phase Integration

### Test 3.1: Frontend-Design Skill Invoked

**Action:** Continue from Test 2.1, confirm detection

**Expected:**
- ✅ Invokes `frontend-design` skill
- ✅ Stops at design phase (no implementation)
- ✅ Saves to `docs/features/YYYY-MM-DD-slug/frontend/design.md`

**Status:** ⬜ Pass ⬜ Fail

### Test 3.2: Feature-Dev Skill Invoked

**Action:** Continue from Test 2.2, confirm detection

**Expected:**
- ✅ Invokes `feature-dev` skill
- ✅ Stops at design phase (no implementation)
- ✅ Saves to `docs/features/YYYY-MM-DD-slug/backend/design.md`

**Status:** ⬜ Pass ⬜ Fail

### Test 3.3: Both Skills for Full-Stack

**Action:** Continue from Test 2.3, confirm detection

**Expected:**
- ✅ Invokes `frontend-design` first
- ✅ Invokes `feature-dev` second
- ✅ Saves to respective folders

**Status:** ⬜ Pass ⬜ Fail

---

## Phase 4: Story Decomposition

### Test 4.1: Writing-Plans Invoked

**Action:** After design phase completes

**Expected:**
- ✅ Invokes `superpowers:writing-plans` skill
- ✅ Creates stories with acceptance criteria
- ✅ Identifies dependencies
- ✅ Saves to `stories.md` in concern folder

**Status:** ⬜ Pass ⬜ Fail

### Test 4.2: Cross-Concern Dependencies

**Action:** Check full-stack feature stories

**Expected:**
- ✅ Frontend stories list backend dependencies
- ✅ Backend stories marked as blocking frontend
- ✅ Dependencies use story titles/IDs

**Status:** ⬜ Pass ⬜ Fail

---

## Phase 5: Linear Upload

### Test 5.1: Nested Hierarchy Created

**Action:** After stories generated, Linear upload runs

**Expected in Linear:**
- ✅ Parent issue: "Feature: [name]"
- ✅ Sub-parent(s): "Frontend/Backend: [name]"
- ✅ Child stories under sub-parents

**Status:** ⬜ Pass ⬜ Fail

### Test 5.2: Base Labels Applied

**Action:** Check Linear tickets

**Expected:**
- ✅ Parent has `["pre-planned"]`
- ✅ Frontend sub-parent/stories have `["pre-planned", "frontend"]`
- ✅ Backend sub-parent/stories have `["pre-planned", "backend"]`

**Status:** ⬜ Pass ⬜ Fail

### Test 5.3: Smart Labels Applied

**Action:** Check Linear tickets from authentication feature

**Expected:**
- ✅ JWT story has `"authentication"` label
- ✅ Database story has `"database"` label
- ✅ Form story has `"forms"` label
- ✅ Security-related stories have `"security"` label

**Status:** ⬜ Pass ⬜ Fail

### Test 5.4: Blocking Relations Set

**Action:** Check Linear blocking relations

**Expected:**
- ✅ Frontend stories blocked by backend stories (if dependencies exist)
- ✅ Sequential stories have within-concern blocking
- ✅ Can view dependency graph in Linear

**Status:** ⬜ Pass ⬜ Fail

---

## Phase 6: Artifacts Saved

### Test 6.1: Design Documents

**Action:** Check filesystem

**Expected:**
```bash
docs/features/YYYY-MM-DD-feature-slug/
├── frontend/design.md (if frontend)
└── backend/design.md (if backend)
```

**Status:** ⬜ Pass ⬜ Fail

### Test 6.2: Story Documents

**Action:** Check filesystem

**Expected:**
```bash
docs/features/YYYY-MM-DD-feature-slug/
├── frontend/stories.md (if frontend)
└── backend/stories.md (if backend)
```

**Status:** ⬜ Pass ⬜ Fail

### Test 6.3: Linear Summary

**Action:** Check filesystem

**Expected:**
```bash
docs/features/YYYY-MM-DD-feature-slug/linear-summary.md
```

Contains:
- ✅ Parent issue ID and link
- ✅ Sub-parent issue IDs and links
- ✅ All story IDs and links
- ✅ Summary of labels applied

**Status:** ⬜ Pass ⬜ Fail

---

## Phase 7: Configuration

### Test 7.1: Custom Labels

**Action:** Edit `.claude/plan-to-linear.local.md`, add custom label:

```yaml
smart_label_keywords:
  custom_keyword: ["special", "unique"]
```

Run plan-to-linear with "special feature"

**Expected:**
- ✅ Story gets `"custom_keyword"` label

**Status:** ⬜ Pass ⬜ Fail

### Test 7.2: Team/Project Config

**Action:** Check Linear upload uses configured team/project

**Expected:**
- ✅ Issues created in correct Linear team
- ✅ Issues created in correct Linear project

**Status:** ⬜ Pass ⬜ Fail

---

## Overall Status

**All tests passing?** ⬜ Yes ⬜ No

If no, see INSTALL.md troubleshooting or open an issue.

---

## Success Criteria

✅ Detection correctly identifies frontend/backend/both
✅ Design skills invoked and stop before implementation
✅ Stories decomposed with dependencies
✅ Linear hierarchy created correctly
✅ Base labels + smart labels applied
✅ Blocking relations set properly
✅ All artifacts saved to docs/features/
```

**Step 2: Verify VERIFY.md**

Run: `cat Packs/plan-to-linear/VERIFY.md | grep "## Phase" | wc -l`
Expected: 7 phases

**Step 3: Commit VERIFY.md**

```bash
git add Packs/plan-to-linear/VERIFY.md
git commit -m "docs: add verification checklist"
```

---

## Task 10: Final Pack Assembly and Documentation

**Files:**
- Create: `Packs/plan-to-linear/LICENSE`
- Create: `Packs/plan-to-linear/.gitignore`

**Step 1: Create LICENSE**

```text
MIT License

Copyright (c) 2026 PAI Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

**Step 2: Create .gitignore**

```
# Local configuration
*.local.md

# Test artifacts
test-output/
.cache/
```

**Step 3: Verify pack structure is complete**

Run: `tree Packs/plan-to-linear -L 3`
Expected:
```
Packs/plan-to-linear/
├── .gitignore
├── INSTALL.md
├── LICENSE
├── README.md
├── VERIFY.md
└── src/
    └── skills/
        └── PlanToLinear/
            └── SKILL.md
```

**Step 4: Final commit**

```bash
git add Packs/plan-to-linear/
git commit -m "feat: complete plan-to-linear pack

Includes:
- SKILL.md with full workflow documentation
- README with quick start guide
- INSTALL with step-by-step setup
- VERIFY with comprehensive test checklist
- LICENSE and .gitignore

Ready for use and distribution."
```

---

## Summary

**Implementation complete! The plan-to-linear pack includes:**

1. ✅ SKILL.md with frontmatter, workflow, examples
2. ✅ README with quick start and features
3. ✅ INSTALL with setup instructions
4. ✅ VERIFY with comprehensive test checklist
5. ✅ LICENSE and .gitignore
6. ✅ All documentation following PAI standards

**Pack ready for:**
- Installation to `.claude/plugins`
- Testing with Linear integration
- Distribution via PAI marketplace
- Use in feature planning workflows
