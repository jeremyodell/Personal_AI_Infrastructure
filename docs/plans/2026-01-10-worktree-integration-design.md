# Git Worktree Integration for Team-Dev-Workflow

**Date:** 2026-01-10
**Status:** Approved
**Problem:** Running multiple team-dev-workflow instances causes branch conflicts in shared workspace

---

## Problem Statement

Running two team-dev-workflow instances simultaneously on the same codebase creates conflicts. Both agents attempt to check out different branches in the same working directory. This causes:
- Branch checkout conflicts
- Lost work context when switching branches
- Inability to work on multiple tickets in parallel

Git worktrees eliminate these conflicts. They create isolated workspaces that share the same `.git` database.

---

## Solution Overview

Integrate git worktrees into Phase 0 of WorkOnTicket workflow to:
1. Auto-create isolated workspace for each ticket
2. Enable parallel ticket development
3. Clean up worktrees after PR creation (with escape hatch)
4. Provide manual cleanup commands

---

## Design

### 1. Worktree Creation (Phase 0)

**Current behavior:**
```bash
git checkout -b "feat/ENG-123-title"
```

**New behavior:**
```bash
# Create worktree with new branch
WORKTREE_PATH=".worktrees/feat-ENG-123-title"
git worktree add "$WORKTREE_PATH" -b "feat/ENG-123-title"

# Change to worktree directory
cd "$WORKTREE_PATH"
```

**Directory structure:**
```
/main/repo/
├── .git/                        # Shared git database
├── .worktrees/                  # Worktree container
│   ├── feat-ENG-123-title/     # Ticket 1 workspace
│   └── feat-ENG-456-other/     # Ticket 2 workspace
├── src/                         # Main repo stays on main branch
└── package.json
```

**Workflow state tracking:**
```typescript
workflowState.worktree_path = ".worktrees/feat-ENG-123-title";
workflowState.keep_worktree = false; // Default
workflowState.original_repo_path = "/main/repo";
```

---

### 2. Hybrid Cleanup Strategy

**Default: Auto-cleanup after PR creation**

After `gh pr create` in Phase 8:
```bash
if [ "$KEEP_WORKTREE" != "true" ]; then
  # Return to main repo
  cd "$ORIGINAL_REPO_PATH"

  # Remove worktree
  git worktree remove ".worktrees/feat-ENG-123-title"

  echo "✅ Worktree cleaned up. Branch preserved on remote."
fi
```

**Escape hatch: --keep-worktree flag**

Users can preserve worktree for PR fixes:
```bash
work on ENG-123 --keep-worktree
```

**User messaging when preserved:**
```markdown
✅ Phase 8 Complete - Shipped

🔍 SHIP DETAILS:
- PR: https://github.com/user/repo/pull/42
- Worktree: Preserved at .worktrees/feat-ENG-123-title

⚠️ Note: Worktree kept because --keep-worktree flag was used
Use '/cleanup-worktree ENG-123' when ready to remove
```

---

### 3. Manual Cleanup

**New command: `/cleanup-worktree`**

```bash
# Clean specific ticket
/cleanup-worktree ENG-123

# Clean all merged worktrees
/cleanup-worktree all
```

**Implementation:**
```typescript
import { execFileNoThrow } from '../utils/execFileNoThrow.js';

async function removeWorktree(path: string) {
  // Safety: Check for uncommitted changes
  const hasChanges = await checkUncommittedChanges(path);
  if (hasChanges) {
    throw new Error("Worktree has uncommitted changes. Commit or stash first.");
  }

  // Remove worktree
  await execFileNoThrow('git', ['worktree', 'remove', path]);

  // Clean up local branch if merged
  const branch = extractBranchFromPath(path);
  const isMerged = await isBranchMerged(branch);
  if (isMerged) {
    await execFileNoThrow('git', ['branch', '-d', branch]);
  }
}
```

**Orphaned worktree detection:**

Warn when worktree count exceeds threshold:
```bash
WORKTREE_COUNT=$(git worktree list | grep -c ".worktrees/")

if [ "$WORKTREE_COUNT" -gt 3 ]; then
  echo "⚠️  You have $WORKTREE_COUNT active worktrees."
  echo "Run '/cleanup-worktree all' to remove merged ones."
fi
```

---

### 4. Edge Case Handling

**Case 1: Worktree already exists**

Prompt the user when restarting work on a ticket with existing worktree:
```bash
if git worktree list | grep -q "feat/ENG-123-title"; then
  echo "⚠️  Worktree already exists for ENG-123"
  echo ""
  echo "Options:"
  echo "  1) Resume work in existing worktree"
  echo "  2) Remove and recreate (loses uncommitted changes)"
  echo "  3) Cancel"

  # Prompt user for choice
fi
```

**Case 2: Branch exists but no worktree**

Migrate existing branch to worktree:
```bash
if git show-ref --verify --quiet "refs/heads/feat/ENG-123-title"; then
  if ! git worktree list | grep -q "feat/ENG-123-title"; then
    # Migrate to worktree
    git worktree add ".worktrees/feat-ENG-123-title" "feat/ENG-123-title"
    cd ".worktrees/feat-ENG-123-title"
  fi
fi
```

**Case 3: Disk space issues**

Fall back to main repo workflow when worktree creation fails:
```bash
if ! git worktree add "$WORKTREE_PATH" -b "$BRANCH_NAME"; then
  echo "❌ Failed to create worktree"
  echo "Falling back to main repo workflow"
  git checkout -b "$BRANCH_NAME"
fi
```

**Case 4: Concurrent agents on same ticket**

Prevent conflicts with lock files:
```bash
WORKTREE_LOCK=".worktrees/.locks/ENG-123.lock"

if [ -f "$WORKTREE_LOCK" ]; then
  echo "❌ ERROR: Another agent is already working on ENG-123"
  exit 1
fi

# Create lock
mkdir -p ".worktrees/.locks"
echo "$WORKTREE_PATH" > "$WORKTREE_LOCK"

# Cleanup lock on exit
trap "rm -f $WORKTREE_LOCK" EXIT
```

---

### 5. Configuration

**New config file: `workflow-config.yaml`**

```yaml
worktree:
  # Enable worktree creation
  enabled: true

  # Base directory for worktrees
  base_path: ".worktrees"

  # Cleanup behavior
  cleanup:
    # Auto-cleanup after PR creation (unless --keep-worktree)
    auto_cleanup: true

    # Warn when this many worktrees exist
    warn_threshold: 3

    # Auto-cleanup merged worktrees older than N days
    auto_cleanup_merged_after_days: 7

  # Lock files to prevent conflicts
  locking:
    enabled: true
    lock_path: ".worktrees/.locks"
```

---

## Implementation Files

**Files to create:**
1. `src/utils/worktree-manager.ts` - Core worktree operations
2. `src/commands/cleanup-worktree.ts` - Manual cleanup command
3. `workflow-config.yaml` - Configuration

**Files to modify:**
1. `src/skills/TeamDev/Workflows/WorkOnTicket.md` - Phase 0 and Phase 8
2. `src/skills/TeamDev/SKILL.md` - Add --keep-worktree flag parsing
3. `~/dev/projects/personal-ai/PAI/.claude/skills/CORE/SKILL.md` - Route worktree flags

---

## Benefits

1. **Parallel Development** - Multiple tickets simultaneously without conflicts
2. **Clean Isolation** - Each ticket in separate workspace
3. **Automatic Cleanup** - No orphaned worktrees by default
4. **Flexibility** - Escape hatch for PR fixes
5. **Safety** - Lock files prevent concurrent work on same ticket

---

## Trade-offs

**Pros:**
- Eliminates branch conflicts
- Enables true parallel workflows
- Main repo stays clean on main branch
- Shared `.git` keeps disk usage reasonable

**Cons:**
- Slight disk overhead (N working directories)
- Requires cleanup discipline (mitigated by auto-cleanup)
- Learning curve for worktree concepts
- Need to manage lock files

---

## Testing Plan

1. Create two tickets in Linear
2. Start work on both simultaneously
3. Verify isolated worktrees created
4. Complete one ticket → verify auto-cleanup
5. Complete second with `--keep-worktree` → verify preservation
6. Test `/cleanup-worktree` command
7. Test edge cases (existing worktree, disk failures)

---

## Future Enhancements

1. **GitHub webhook integration** - Auto-cleanup when PR merges
2. **Worktree status dashboard** - Show all active worktrees
3. **Smart disk space management** - Auto-cleanup when disk low
4. **Worktree templates** - Pre-configure new worktrees with environment
