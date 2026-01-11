# Worktree Integration Test Specification

**Date:** 2026-01-10
**Feature:** Git worktree integration in team-dev-workflow
**Test Type:** Manual integration testing

---

## Test Environment

- Clean git repository (no worktrees)
- Linear tickets available: ENG-111, ENG-222, ENG-333
- Team-dev-workflow pack installed
- worktree.enabled: true in config

---

## Test Scenarios

### Test 1: Basic Worktree Creation

**Objective:** Verify worktree created automatically

**Steps:**
1. Run: `work on ENG-111`
2. Verify worktree created: `git worktree list`
3. Verify directory exists: `.worktrees/feat-ENG-111-title`
4. Verify branch created: `git branch | grep feat/ENG-111-title`

**Expected Results:**
- ✅ Worktree appears in `git worktree list`
- ✅ Directory `.worktrees/feat-ENG-111-title` exists
- ✅ Branch `feat/ENG-111-title` exists
- ✅ Current directory changed to worktree

---

### Test 2: Parallel Development

**Objective:** Verify multiple worktrees work simultaneously

**Steps:**
1. Run: `work on ENG-111` (complete and ship)
2. Run: `work on ENG-222` (start, don't complete)
3. Run: `work on ENG-333` (start, don't complete)
4. Verify: `git worktree list` shows 2 active worktrees (ENG-222, ENG-333)

**Expected Results:**
- ✅ ENG-111 worktree cleaned up (shipped)
- ✅ ENG-222 worktree exists and active
- ✅ ENG-333 worktree exists and active
- ✅ No branch conflicts
- ✅ Each worktree on correct branch

---

### Test 3: Auto-Cleanup After Ship

**Objective:** Verify worktree auto-cleanup (default behavior)

**Steps:**
1. Run: `work on ENG-111`
2. Complete workflow (implement, test, ship)
3. After PR creation, verify: `git worktree list`
4. Verify directory removed: `ls .worktrees/`

**Expected Results:**
- ✅ Worktree removed after ship
- ✅ Directory `.worktrees/feat-ENG-111-title` does not exist
- ✅ Branch still exists: `git branch | grep feat/ENG-111-title`
- ✅ User returned to main repo directory

---

### Test 4: Keep Worktree Flag

**Objective:** Verify --keep-worktree flag preserves worktree

**Steps:**
1. Run: `work on ENG-111 --keep-worktree`
2. Complete workflow (ship PR)
3. Verify worktree still exists: `git worktree list`
4. Verify directory exists: `.worktrees/feat-ENG-111-title`

**Expected Results:**
- ✅ Worktree NOT removed after ship
- ✅ Directory `.worktrees/feat-ENG-111-title` exists
- ✅ User sees message: "Worktree preserved (--keep-worktree)"
- ✅ User sees cleanup instructions

---

### Test 5: Manual Cleanup (Specific)

**Objective:** Verify /cleanup-worktree command for specific ticket

**Steps:**
1. Create worktree: `work on ENG-111 --keep-worktree`
2. Ship PR
3. Run: `/cleanup-worktree ENG-111`
4. Verify: `git worktree list`

**Expected Results:**
- ✅ Worktree removed
- ✅ Directory deleted
- ✅ Lock file removed
- ✅ User sees success message

---

### Test 6: Manual Cleanup (All Merged)

**Objective:** Verify /cleanup-worktree all removes merged branches

**Steps:**
1. Create 3 worktrees for ENG-111, ENG-222, ENG-333
2. Ship PRs for ENG-111 and ENG-222 with --keep-worktree
3. Merge ENG-111 PR on GitHub
4. Run: `/cleanup-worktree all`
5. Verify: `git worktree list`

**Expected Results:**
- ✅ ENG-111 worktree removed (merged)
- ✅ ENG-222 worktree NOT removed (unmerged)
- ✅ ENG-333 worktree NOT removed (unmerged)
- ✅ User sees summary: "Removed 1 worktree(s)"

---

### Test 7: Lock File Protection

**Objective:** Verify lock files prevent concurrent work

**Steps:**
1. Run: `work on ENG-111` (in terminal 1)
2. While running, in terminal 2: `work on ENG-111`
3. Verify error message in terminal 2

**Expected Results:**
- ✅ Terminal 2 shows error: "Another agent is already working on ENG-111"
- ✅ Lock file exists: `.worktrees/.locks/ENG-111.lock`
- ✅ Terminal 1 continues normally
- ✅ Lock removed when terminal 1 completes

---

### Test 8: Worktree Already Exists

**Objective:** Verify behavior when worktree exists

**Steps:**
1. Run: `work on ENG-111`
2. Cancel/exit workflow
3. Run: `work on ENG-111` again

**Expected Results:**
- ✅ User prompted with options:
  - Resume work
  - Remove and recreate
  - Cancel
- ✅ Selection respected
- ✅ No data loss

---

### Test 9: Disk Space Fallback

**Objective:** Verify fallback to main repo on disk error

**Steps:**
1. Simulate low disk space (platform-specific)
2. Run: `work on ENG-111`
3. Verify fallback behavior

**Expected Results:**
- ✅ Error message about worktree creation failure
- ✅ Fallback message: "Falling back to main repo workflow"
- ✅ Workflow continues in main repo
- ✅ No crash/hang

---

### Test 10: Configuration Disabled

**Objective:** Verify workflow works with worktree.enabled: false

**Steps:**
1. Edit `config/workflow-config.yaml`: Set `worktree.enabled: false`
2. Run: `work on ENG-111`
3. Verify behavior

**Expected Results:**
- ✅ No worktree created
- ✅ Branch checked out in main repo
- ✅ Workflow completes normally
- ✅ No errors

---

### Test 11: Lock File Cleanup on Exit

**Objective:** Verify lock removed on signal (Ctrl+C)

**Steps:**
1. Run: `work on ENG-111`
2. During Phase 0, press Ctrl+C
3. Verify lock file: `.worktrees/.locks/ENG-111.lock`

**Expected Results:**
- ✅ Lock file removed
- ✅ Worktree removed (if created)
- ✅ Clean state

---

### Test 12: .worktrees in .gitignore

**Objective:** Verify .worktrees directory is ignored

**Steps:**
1. Create worktree: `work on ENG-111`
2. Run: `git status`
3. Verify `.worktrees/` not shown as untracked

**Expected Results:**
- ✅ `.worktrees/` in `.gitignore`
- ✅ `git status` shows clean (except staged files)
- ✅ No worktree content tracked

---

## Success Criteria

All 12 test scenarios must pass for the worktree integration to be considered complete and production-ready.

**Test Execution:**
- Run tests in order (1-12)
- Document any failures
- Re-test after fixes
- Sign-off when all pass
