---
name: cleanup-worktree
description: Manually remove git worktrees after PRs are merged
type: command
---

# Cleanup Worktree Command

**Usage:**
```bash
/cleanup-worktree ENG-123        # Clean specific ticket worktree
/cleanup-worktree all            # Clean all merged worktrees
/cleanup-worktree --list         # List all worktrees
```

---

## Command Implementation

### Clean Specific Ticket

**Input:** Ticket identifier (e.g., "ENG-123")

**Steps:**

1. **Find worktree for ticket**
```bash
# Extract identifier from argument
IDENTIFIER="$1"

# Find worktree path
WORKTREE_PATH=$(git worktree list | grep -i "$IDENTIFIER" | awk '{print $1}')

if [ -z "$WORKTREE_PATH" ]; then
  echo "❌ No worktree found for $IDENTIFIER"
  exit 1
fi

echo "Found worktree: $WORKTREE_PATH"
```

2. **Safety check - uncommitted changes**
```bash
# Check for uncommitted changes
cd "$WORKTREE_PATH"
if ! git diff-index --quiet HEAD --; then
  echo "❌ Worktree has uncommitted changes. Options:"
  echo "  1) Commit changes first"
  echo "  2) Stash changes"
  echo "  3) Force remove (loses changes)"
  # Use AskUserQuestion tool
  exit 1
fi
```

3. **Remove worktree**
```bash
# Return to main repo
cd "$(git rev-parse --show-toplevel)"

# Remove worktree
git worktree remove "$WORKTREE_PATH"

# Remove lock file if exists
LOCK_PATH=".worktrees/.locks/$IDENTIFIER.lock"
rm -f "$LOCK_PATH"

echo "✅ Removed worktree for $IDENTIFIER"
```

4. **Clean up local branch if merged**
```bash
# Extract branch name from worktree path
BRANCH_NAME=$(basename "$WORKTREE_PATH" | sed 's/^feat-/feat\//')

# Check if branch is merged
if git branch --merged | grep -q "$BRANCH_NAME"; then
  git branch -d "$BRANCH_NAME"
  echo "✅ Deleted local branch $BRANCH_NAME (already merged)"
else
  echo "ℹ️  Branch $BRANCH_NAME not merged, keeping local branch"
fi
```

---

### Clean All Merged Worktrees

**Input:** "all"

**Steps:**

1. **List all worktrees**
```bash
# Get all worktrees in .worktrees/
WORKTREES=$(git worktree list | grep ".worktrees/" | awk '{print $1}')

if [ -z "$WORKTREES" ]; then
  echo "No worktrees found in .worktrees/"
  exit 0
fi

echo "Found worktrees:"
echo "$WORKTREES"
```

2. **Check each for merged status**
```bash
REMOVED_COUNT=0

for WORKTREE_PATH in $WORKTREES; do
  # Extract branch name
  BRANCH_NAME=$(git worktree list | grep "$WORKTREE_PATH" | awk '{print $3}' | tr -d '[]')

  # Check if merged
  if git branch --merged | grep -q "$BRANCH_NAME"; then
    echo "Removing merged worktree: $WORKTREE_PATH"

    # Remove worktree
    git worktree remove "$WORKTREE_PATH"

    # Delete branch
    git branch -d "$BRANCH_NAME"

    REMOVED_COUNT=$((REMOVED_COUNT + 1))
  else
    echo "Skipping unmerged worktree: $WORKTREE_PATH"
  fi
done

echo "✅ Cleaned up $REMOVED_COUNT merged worktrees"
```

3. **Clean up lock files**
```bash
# Remove stale lock files
find .worktrees/.locks -type f -name "*.lock" -exec rm -f {} \;
echo "✅ Cleaned up lock files"
```

---

### List All Worktrees

**Input:** "--list"

**Steps:**

```bash
echo "Active worktrees:"
echo ""

git worktree list | grep ".worktrees/" | while read -r line; do
  WORKTREE_PATH=$(echo "$line" | awk '{print $1}')
  BRANCH_NAME=$(echo "$line" | awk '{print $3}' | tr -d '[]')

  # Check if merged
  MERGED_STATUS="Not merged"
  if git branch --merged | grep -q "$BRANCH_NAME"; then
    MERGED_STATUS="✅ Merged"
  fi

  # Check for uncommitted changes
  cd "$WORKTREE_PATH"
  CHANGES_STATUS="Clean"
  if ! git diff-index --quiet HEAD --; then
    CHANGES_STATUS="⚠️ Uncommitted changes"
  fi
  cd - > /dev/null

  echo "- $WORKTREE_PATH"
  echo "  Branch: $BRANCH_NAME"
  echo "  Status: $MERGED_STATUS | $CHANGES_STATUS"
  echo ""
done
```

---

## Error Handling

### Worktree Not Found
```markdown
❌ No worktree found for ENG-123

Possible causes:
- Worktree already removed
- Ticket identifier incorrect
- Worktree created in different location

Use '/cleanup-worktree --list' to see active worktrees
```

### Uncommitted Changes
```markdown
❌ Worktree has uncommitted changes

Options:
1. Commit changes:
   cd <worktree-path>
   git add .
   git commit -m "message"

2. Stash changes:
   cd <worktree-path>
   git stash

3. Force remove (loses changes):
   /cleanup-worktree ENG-123 --force
```

### Permission Denied
```markdown
❌ Permission denied removing worktree

This may occur if:
- Worktree directory is in use
- Insufficient filesystem permissions

Try:
1. Close any programs using the worktree directory
2. Check directory permissions
3. Run with appropriate permissions
```
