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
