# Git Worktrees in Team-Dev-Workflow

## Overview

Git worktrees enable parallel development by creating isolated workspaces that share the same `.git` database. Each worktree can have a different branch checked out, allowing you to work on multiple tickets simultaneously without conflicts.

## Benefits

**Parallel Development**
- Work on multiple Linear tickets simultaneously
- Each ticket gets its own isolated workspace
- No branch switching conflicts
- No lost context when switching tasks

**Clean Isolation**
- Each worktree has independent:
  - Working directory
  - Staged files
  - Branch state
- Shared `.git` database (efficient disk usage)

**Automatic Management**
- Worktrees created automatically in Phase 0
- Cleaned up automatically in Phase 8 (by default)
- Lock files prevent concurrent work on same ticket

## Usage

### Basic Workflow

```bash
# Start work on a ticket (creates worktree automatically)
work on ENG-123

# The workflow:
# 1. Fetches ticket from Linear
# 2. Creates worktree at .worktrees/feat-ENG-123-title
# 3. Creates branch feat/ENG-123-title
# 4. Changes directory to worktree
# 5. Implements feature
# 6. Ships (commit, PR)
# 7. Cleans up worktree (returns to main repo)
```

### Keep Worktree for PR Fixes

```bash
# Preserve worktree after PR creation
work on ENG-123 --keep-worktree

# Make PR fixes in the preserved worktree
cd .worktrees/feat-ENG-123-title

# When done, clean up manually
/cleanup-worktree ENG-123
```

## Manual Cleanup

### Clean Specific Ticket

```bash
# Remove worktree for a specific ticket
/cleanup-worktree ENG-123
```

### Clean All Merged Worktrees

```bash
# Remove all worktrees for merged branches
/cleanup-worktree all
```

### List Active Worktrees

```bash
git worktree list
```

## Directory Structure

```
/main/repo/
├── .git/                        # Shared git database
├── .worktrees/                  # Worktree container
│   ├── .locks/                  # Lock files (prevent conflicts)
│   │   └── ENG-123.lock
│   ├── feat-ENG-123-title/     # Ticket 1 workspace
│   └── feat-ENG-456-other/     # Ticket 2 workspace
├── src/                         # Main repo stays on main branch
└── package.json
```

## Edge Cases

### Worktree Already Exists

If you start work on a ticket that already has a worktree, you'll see:

```
⚠️  Worktree already exists for ENG-123

Options:
  1) Resume work in existing worktree
  2) Remove and recreate (loses uncommitted changes)
  3) Cancel
```

### Concurrent Work Protection

Lock files prevent multiple agents from working on the same ticket:

```
❌ ERROR: Another agent is already working on ENG-123
```

The lock is automatically released when the workflow completes.

### Disk Space Issues

If worktree creation fails due to disk space, the workflow falls back to main repo:

```
❌ Failed to create worktree
Falling back to main repo workflow
```

## Configuration

Settings in `config/workflow-config.yaml`:

```yaml
worktree:
  enabled: true                    # Enable worktree creation
  base_path: ".worktrees"          # Where to create worktrees

  cleanup:
    auto_cleanup: true             # Cleanup after PR creation
    warn_threshold: 3              # Warn when N worktrees exist
    auto_cleanup_merged_after_days: 7  # Auto-cleanup merged after N days

  locking:
    enabled: true                  # Prevent concurrent work
    lock_path: ".worktrees/.locks" # Lock file location
```

## Troubleshooting

### Orphaned Worktrees

List all worktrees:
```bash
git worktree list
```

Remove orphaned worktree:
```bash
git worktree remove .worktrees/feat-ENG-123-title
```

### Uncommitted Changes Warning

```
❌ ERROR: Worktree has uncommitted changes
```

**Solution:** Commit or stash changes first:
```bash
cd .worktrees/feat-ENG-123-title
git add .
git commit -m "wip: save progress"
# or
git stash
```

### Lock File Stuck

If a lock file is stuck (workflow crashed):
```bash
rm .worktrees/.locks/ENG-123.lock
```

## Best Practices

1. **Let the workflow manage worktrees** - Don't create them manually
2. **Use --keep-worktree sparingly** - Only for quick PR fixes
3. **Clean up merged worktrees** - Run `/cleanup-worktree all` periodically
4. **Don't commit worktree contents** - Ensure `.worktrees/` is in `.gitignore`
5. **One agent per ticket** - Lock files enforce this automatically

## FAQ

**Q: Do worktrees use extra disk space?**
A: Yes, but only for the working directory. The `.git` database is shared, so the overhead is minimal (typically 10-50MB per worktree).

**Q: Can I have multiple agents on different tickets?**
A: Yes! That's the whole point. Each ticket gets its own worktree.

**Q: What happens if I forget --keep-worktree?**
A: The worktree is automatically cleaned up after PR creation. You'll need to check out the branch again if you want to make changes.

**Q: Can I use worktrees outside this workflow?**
A: Yes, but manual worktrees aren't managed by the workflow. Use `git worktree add` directly.

**Q: What if I delete a worktree manually?**
A: Git will detect it. Run `git worktree prune` to clean up references.
