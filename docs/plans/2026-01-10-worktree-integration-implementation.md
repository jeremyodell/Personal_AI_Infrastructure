# Worktree Integration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Integrate git worktrees into team-dev-workflow to enable parallel ticket development without branch conflicts.

**Architecture:** Modify Phase 0 to create worktrees instead of checking out branches, add cleanup logic to Phase 8, create manual cleanup command, add configuration section.

**Tech Stack:** Markdown workflows, YAML configuration, Bash scripts (embedded in markdown)

---

## Task 1: Add Worktree Configuration

**Files:**
- Modify: `Packs/team-dev-workflow/src/config/workflow-config.yaml:415` (after line 415, before environment variables section)

**Step 1: Add worktree configuration section**

Add this section after line 415 in workflow-config.yaml:

```yaml
# =============================================================================
# WORKTREE CONFIGURATION
# =============================================================================

worktree:
  # Enable worktree creation for isolated workspaces
  enabled: true

  # Base directory for worktrees (relative to repository root)
  base_path: ".worktrees"

  # Cleanup behavior
  cleanup:
    # Auto-cleanup after PR creation (unless --keep-worktree flag used)
    auto_cleanup: true

    # Warn when this many worktrees exist
    warn_threshold: 3

    # Auto-cleanup merged worktrees older than N days
    auto_cleanup_merged_after_days: 7

  # Lock files to prevent concurrent work on same ticket
  locking:
    enabled: true
    lock_path: ".worktrees/.locks"

  # Fallback behavior
  fallback:
    # Fall back to main repo if worktree creation fails
    on_disk_error: main_repo
    on_permission_error: main_repo
```

**Step 2: Verify configuration**

Run: `cat Packs/team-dev-workflow/src/config/workflow-config.yaml | grep -A 20 "WORKTREE CONFIGURATION"`
Expected: Configuration section appears with proper YAML formatting

**Step 3: Commit**

```bash
git add Packs/team-dev-workflow/src/config/workflow-config.yaml
git commit -m "config: add worktree configuration section

Added worktree settings for parallel ticket development:
- Enable/disable worktree creation
- Base path configuration (.worktrees)
- Auto-cleanup after PR creation
- Lock files for concurrent agent prevention
- Fallback to main repo on errors

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 2: Modify WorkOnTicket Phase 0 - Worktree Creation

**Files:**
- Modify: `Packs/team-dev-workflow/src/skills/TeamDev/Workflows/WorkOnTicket.md:79-86` (Replace "4. Create Feature Branch" section)

**Step 1: Replace branch creation with worktree logic**

Replace lines 79-86 with:

```markdown
4. **Create Feature Branch with Worktree**

Read worktree configuration:
```bash
# Load config (pseudocode - actual implementation reads YAML)
WORKTREE_ENABLED=$(grep "enabled: true" workflow-config.yaml | grep worktree)
WORKTREE_BASE_PATH=$(grep "base_path:" workflow-config.yaml | awk '{print $2}' | tr -d '"')
```

If worktrees enabled:
```bash
# Slugify title (lowercase, hyphens, alphanumeric only)
SLUG=$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//' | sed 's/-$//')

# Store original repo path
ORIGINAL_REPO_PATH=$(pwd)

# Create worktree path
WORKTREE_PATH="$WORKTREE_BASE_PATH/feat-$IDENTIFIER-$SLUG"

# Check for existing worktree
if git worktree list | grep -q "$WORKTREE_PATH"; then
  echo "⚠️  Worktree already exists for $IDENTIFIER"
  echo ""
  echo "Options:"
  echo "  1) Resume work in existing worktree"
  echo "  2) Remove and recreate (loses uncommitted changes)"
  echo "  3) Cancel"
  # Prompt user for choice (use AskUserQuestion tool)
  # If option 1: cd to existing worktree
  # If option 2: git worktree remove + recreate
  # If option 3: exit
fi

# Check for lock file (concurrent agent detection)
LOCK_PATH=".worktrees/.locks/$IDENTIFIER.lock"
if [ -f "$LOCK_PATH" ]; then
  echo "❌ ERROR: Another agent is already working on $IDENTIFIER"
  echo "Lock file: $LOCK_PATH"
  exit 1
fi

# Create lock file
mkdir -p ".worktrees/.locks"
echo "$WORKTREE_PATH" > "$LOCK_PATH"
trap "rm -f $LOCK_PATH" EXIT

# Create worktree with new branch
if git worktree add "$WORKTREE_PATH" -b "feat/$IDENTIFIER-$SLUG"; then
  cd "$WORKTREE_PATH"

  # Track worktree in workflow state
  echo "WORKTREE_PATH=$WORKTREE_PATH" >> .workflow-state
  echo "ORIGINAL_REPO_PATH=$ORIGINAL_REPO_PATH" >> .workflow-state
  echo "KEEP_WORKTREE=false" >> .workflow-state

  echo "✅ Worktree created at $WORKTREE_PATH"
else
  # Fallback to main repo workflow
  echo "❌ Failed to create worktree"
  echo "Falling back to main repo workflow"
  git checkout -b "feat/$IDENTIFIER-$SLUG"
  echo "WORKTREE_ENABLED=false" >> .workflow-state
fi
```

If worktrees disabled:
```bash
# Traditional branch checkout
SLUG=$(echo "$TITLE" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//' | sed 's/-$//')
git checkout -b "feat/$IDENTIFIER-$SLUG"
echo "WORKTREE_ENABLED=false" >> .workflow-state
```

**Orphaned worktree detection:**
```bash
# Warn if too many worktrees exist
if [ "$WORKTREE_ENABLED" = "true" ]; then
  WORKTREE_COUNT=$(git worktree list | grep -c "$WORKTREE_BASE_PATH/")
  WARN_THRESHOLD=3  # From config

  if [ "$WORKTREE_COUNT" -gt "$WARN_THRESHOLD" ]; then
    echo "⚠️  You have $WORKTREE_COUNT active worktrees."
    echo "Run '/cleanup-worktree all' to remove merged ones."
  fi
fi
```
```

**Step 2: Verify markdown formatting**

Run: `cat Packs/team-dev-workflow/src/skills/TeamDev/Workflows/WorkOnTicket.md | grep -A 50 "Create Feature Branch with Worktree"`
Expected: New section appears with proper markdown formatting

**Step 3: Commit**

```bash
git add Packs/team-dev-workflow/src/skills/TeamDev/Workflows/WorkOnTicket.md
git commit -m "feat: add worktree creation to Phase 0

Modified WorkOnTicket Phase 0 to create git worktrees:
- Creates isolated workspace in .worktrees/ directory
- Detects existing worktrees and prompts user
- Creates lock files to prevent concurrent agents
- Falls back to main repo on worktree creation failure
- Warns when worktree count exceeds threshold

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 3: Modify ShipFeature Workflow - Add Cleanup Logic

**Files:**
- Modify: `Packs/team-dev-workflow/src/skills/TeamDev/Workflows/ShipFeature.md` (add new section after PR creation)

**Step 1: Read current ShipFeature workflow**

Run: `cat Packs/team-dev-workflow/src/skills/TeamDev/Workflows/ShipFeature.md | head -100`
Expected: Understand current structure

**Step 2: Add worktree cleanup section after PR creation**

Add this section after the `gh pr create` step:

```markdown
### Worktree Cleanup

**Condition:** Only if worktree was created (check .workflow-state)

Read workflow state:
```bash
# Load worktree state
if [ -f .workflow-state ]; then
  source .workflow-state
fi
```

If worktree enabled and auto-cleanup enabled:
```bash
# Check if --keep-worktree flag was used
if [ "$KEEP_WORKTREE" != "true" ]; then
  echo "🧹 Cleaning up worktree..."

  # Return to original repo
  cd "$ORIGINAL_REPO_PATH"

  # Remove worktree
  git worktree remove "$WORKTREE_PATH"

  # Remove lock file
  LOCK_PATH=".worktrees/.locks/$IDENTIFIER.lock"
  rm -f "$LOCK_PATH"

  echo "✅ Worktree cleaned up. Branch preserved on remote."
else
  echo "⚠️ Worktree preserved at $WORKTREE_PATH"
  echo "Use '/cleanup-worktree $IDENTIFIER' when ready to remove"
fi
```
```

**Step 3: Verify changes**

Run: `cat Packs/team-dev-workflow/src/skills/TeamDev/Workflows/ShipFeature.md | grep -A 20 "Worktree Cleanup"`
Expected: New cleanup section appears

**Step 4: Commit**

```bash
git add Packs/team-dev-workflow/src/skills/TeamDev/Workflows/ShipFeature.md
git commit -m "feat: add worktree cleanup to ShipFeature

Added automatic worktree cleanup after PR creation:
- Cleans up worktree by default after PR
- Respects --keep-worktree flag to preserve
- Removes lock files
- Returns to original repo path

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 4: Create Cleanup Worktree Command

**Files:**
- Create: `Packs/team-dev-workflow/src/skills/CleanupWorktree/COMMAND.md`

**Step 1: Create command directory**

Run: `mkdir -p Packs/team-dev-workflow/src/skills/CleanupWorktree`
Expected: Directory created

**Step 2: Create command definition file**

Create file with:

```markdown
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
```

**Step 3: Verify command file**

Run: `cat Packs/team-dev-workflow/src/skills/CleanupWorktree/COMMAND.md | head -50`
Expected: Command definition with proper frontmatter

**Step 4: Commit**

```bash
git add Packs/team-dev-workflow/src/skills/CleanupWorktree/
git commit -m "feat: add cleanup-worktree command

Created manual worktree cleanup command:
- /cleanup-worktree <ticket-id> - Remove specific worktree
- /cleanup-worktree all - Remove all merged worktrees
- /cleanup-worktree --list - List active worktrees
- Safety checks for uncommitted changes
- Auto-deletes merged local branches

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 5: Update TeamDev SKILL.md - Add Flag Parsing

**Files:**
- Modify: `Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md:19-26` (Update Quick Start section)

**Step 1: Add --keep-worktree flag documentation**

Replace lines 19-26 with:

```markdown
## Quick Start

```bash
# Work on a Linear ticket (any of these formats work)
work on ENG-123
work on ticket ENG-456
implement ENG-789

# Keep worktree after PR creation (for making PR fixes)
work on ENG-123 --keep-worktree
```

**Flags:**
- `--keep-worktree` - Preserve worktree after PR creation (useful for quick PR fixes)

The workflow automatically:
1. Fetches ticket from Linear
2. Creates isolated git worktree (enables parallel ticket work)
3. Routes based on labels (pre-planned, UI, frontend)
4. Runs discovery & architecture (if needed)
5. Implements with TDD enforcement
6. Reviews code for quality
7. Runs quality gates (test, lint, typecheck, build)
8. Ships (commit, PR, Linear sync)
9. Cleans up worktree (unless --keep-worktree used)
```

**Step 2: Add flag parsing section**

Add after Quick Start section:

```markdown
---

## Flag Parsing

**Worktree Flags:**

When user invokes workflow with flags:
```typescript
// Parse user input for flags
const userInput = "work on ENG-123 --keep-worktree";

// Extract ticket ID and flags
const ticketMatch = userInput.match(/\b([A-Z]+-\d+)\b/);
const ticketId = ticketMatch ? ticketMatch[1] : null;

// Check for --keep-worktree flag
const keepWorktree = userInput.includes('--keep-worktree');

// Store in workflow state
workflowState.keep_worktree = keepWorktree;
```

**Flag propagation:**
- Phase 0: Reads `keep_worktree` from state
- Phase 8: Respects flag during cleanup
- Cleanup command: Reads from state file
```

**Step 3: Verify changes**

Run: `cat Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md | grep -A 30 "Quick Start"`
Expected: New flag documentation and parsing section

**Step 4: Commit**

```bash
git add Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md
git commit -m "docs: add --keep-worktree flag to TeamDev skill

Added documentation and parsing logic for --keep-worktree flag:
- Updated Quick Start with flag examples
- Added Flag Parsing section
- Documented flag propagation through workflow phases

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 6: Update CORE SKILL.md - Route Worktree Flags

**Files:**
- Modify: `~/.claude/skills/CORE/SKILL.md:88-99` (Update Skill Routing section)

**Step 1: Update routing documentation to include flags**

Replace lines 88-99 with:

```markdown
## Skill Routing

**Team Development Workflow:**

When user says:
- "work on ENG-123"
- "work on ENG-123 --keep-worktree"
- "work on ticket ENG-456"
- "implement ENG-789"
- "start ENG-111"
- "start ticket ENG-222"

Parse and route:
```typescript
// Extract ticket ID
const ticketMatch = userInput.match(/\b([A-Z]+-\d+)\b/);
const ticketId = ticketMatch ? ticketMatch[1] : null;

// Extract flags
const flags = {
  keep_worktree: userInput.includes('--keep-worktree')
};

// Invoke skill with parsed data
await Skill({
  skill: "TeamDev",
  args: JSON.stringify({ ticket_id: ticketId, ...flags })
});
```

→ Invoke TeamDev skill from team-dev-workflow pack

**Path:** `~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md`

**Supported flags:**
- `--keep-worktree` - Preserve worktree after PR creation
```

**Step 2: Verify changes**

Run: `cat ~/.claude/skills/CORE/SKILL.md | grep -A 30 "Team Development Workflow"`
Expected: Updated routing with flag parsing

**Step 3: Commit (in main PAI repo)**

```bash
cd ~/dev/projects/personal-ai/PAI
git add .claude/skills/CORE/SKILL.md
git commit -m "feat: add worktree flag routing to CORE skill

Updated CORE skill routing to parse --keep-worktree flag:
- Extracts ticket ID and flags from user input
- Passes flags to TeamDev skill
- Documents supported flags

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 7: Add Worktree Documentation

**Files:**
- Create: `Packs/team-dev-workflow/docs/worktrees.md`

**Step 1: Create docs directory if not exists**

Run: `mkdir -p Packs/team-dev-workflow/docs`
Expected: Directory created

**Step 2: Create worktree documentation**

Create file with:

```markdown
# Git Worktrees in Team-Dev-Workflow

**What are git worktrees?**

Git worktrees allow you to work on multiple branches simultaneously by creating isolated workspaces that share the same `.git` database.

---

## Benefits

1. **Parallel Development** - Work on multiple tickets simultaneously without branch switching
2. **Clean Isolation** - Each ticket has its own workspace, preventing cross-contamination
3. **No Context Loss** - Switching between tickets doesn't require stashing or committing incomplete work
4. **Efficient Disk Usage** - Worktrees share the `.git` database, so you don't duplicate the entire repository

---

## How It Works

### Traditional Workflow (Without Worktrees)

```
/repo/
├── .git/
├── src/
└── package.json

# Switch between tickets
git checkout feat/ENG-123-dashboard  # Work on ticket 123
git checkout feat/ENG-456-api       # Lose context, switch to ticket 456
git checkout feat/ENG-123-dashboard  # Switch back, need to remember context
```

### Worktree Workflow

```
/repo/
├── .git/                        # Shared git database
├── .worktrees/
│   ├── feat-ENG-123-dashboard/  # Ticket 123 workspace
│   └── feat-ENG-456-api/       # Ticket 456 workspace
├── src/                         # Main repo stays on main branch
└── package.json
```

You can have Claude working on ENG-123 in one session and ENG-456 in another, simultaneously.

---

## Usage

### Starting Work with Worktree

```bash
# Standard workflow - auto-creates worktree
work on ENG-123

# Keep worktree after PR for quick fixes
work on ENG-123 --keep-worktree
```

### Manual Cleanup

```bash
# Clean specific worktree after PR is merged
/cleanup-worktree ENG-123

# Clean all merged worktrees
/cleanup-worktree all

# List active worktrees
/cleanup-worktree --list
```

---

## Automatic Cleanup

By default, worktrees are automatically cleaned up after PR creation. This keeps your workspace tidy.

**When auto-cleanup happens:**
- After `gh pr create` succeeds in Phase 8
- Only if `--keep-worktree` flag was NOT used
- Removes worktree directory
- Removes lock file
- Preserves remote branch (for PR review)

**When to use `--keep-worktree`:**
- You anticipate needing to make quick PR fixes
- You want to test the code locally after PR creation
- You're working on a feature that needs iteration

---

## Edge Cases

### Existing Worktree

If you restart work on a ticket with an existing worktree:

```
⚠️  Worktree already exists for ENG-123

Options:
  1) Resume work in existing worktree
  2) Remove and recreate (loses uncommitted changes)
  3) Cancel
```

**Recommendation:** Choose option 1 to resume work.

### Disk Space Issues

If worktree creation fails due to disk space:

```
❌ Failed to create worktree
Falling back to main repo workflow
```

The workflow automatically falls back to traditional branch checkout.

### Concurrent Agents

If another agent is already working on the same ticket:

```
❌ ERROR: Another agent is already working on ENG-123
Lock file: .worktrees/.locks/ENG-123.lock
```

Wait for the other agent to complete or manually remove the lock file if it's stale.

---

## Configuration

Edit `Packs/team-dev-workflow/src/config/workflow-config.yaml`:

```yaml
worktree:
  # Disable worktrees entirely (use traditional workflow)
  enabled: false

  # Change worktree base directory
  base_path: "worktrees"  # No leading dot

  cleanup:
    # Disable auto-cleanup (always keep worktrees)
    auto_cleanup: false

    # Increase warning threshold
    warn_threshold: 5
```

---

## Troubleshooting

### "Worktree not found" error

```bash
# List all worktrees to verify
git worktree list

# Check if worktree was already removed
ls -la .worktrees/
```

### Uncommitted changes preventing cleanup

```bash
# Commit changes
cd .worktrees/feat-ENG-123-dashboard
git add .
git commit -m "WIP: save progress"

# Or stash changes
git stash

# Then cleanup
/cleanup-worktree ENG-123
```

### Orphaned worktrees

If worktrees accumulate:

```bash
# List all worktrees and their merge status
/cleanup-worktree --list

# Clean all merged worktrees
/cleanup-worktree all

# Force remove specific worktree (loses uncommitted changes)
git worktree remove --force .worktrees/feat-ENG-123-dashboard
```

---

## Best Practices

1. **Let auto-cleanup work** - Unless you need quick PR fixes, let worktrees auto-cleanup
2. **Run `/cleanup-worktree all` weekly** - Clean up any merged worktrees
3. **Use `--keep-worktree` sparingly** - Only when you genuinely need the worktree preserved
4. **Check worktree count** - If you see the warning "You have N active worktrees", clean up
5. **Commit or stash before cleanup** - Avoid losing work by committing/stashing first

---

## Performance Impact

**Disk Space:**
- Each worktree is approximately the size of your working directory
- `.git` database is shared (no duplication)
- Example: If your repo is 100MB, each worktree adds ~100MB

**Speed:**
- Worktree creation: ~1-2 seconds
- Branch checkout in worktree: Instant (no switching needed)
- Cleanup: ~1 second per worktree

**Recommended Limits:**
- Keep 3-5 active worktrees maximum
- Clean up merged worktrees regularly
- Monitor disk space on smaller drives
```

**Step 3: Verify documentation**

Run: `cat Packs/team-dev-workflow/docs/worktrees.md | head -100`
Expected: Documentation with proper formatting

**Step 4: Commit**

```bash
git add Packs/team-dev-workflow/docs/
git commit -m "docs: add comprehensive worktree documentation

Created worktrees.md explaining:
- What git worktrees are and their benefits
- How the worktree workflow works
- Usage examples and commands
- Edge cases and troubleshooting
- Configuration options
- Best practices

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 8: Update README with Worktree Features

**Files:**
- Modify: `Packs/team-dev-workflow/README.md` (add worktree section)

**Step 1: Read current README**

Run: `cat Packs/team-dev-workflow/README.md | head -150`
Expected: Understand current structure

**Step 2: Add worktree section after Features section**

Add this section:

```markdown
### Git Worktrees for Parallel Development

**Run multiple tickets simultaneously without conflicts:**

```bash
# Session 1: Work on feature
work on ENG-123

# Session 2: Work on bug fix (parallel!)
work on ENG-456
```

Each ticket gets its own isolated workspace in `.worktrees/`. No branch switching, no context loss.

**Features:**
- ✅ Automatic worktree creation and cleanup
- ✅ Lock files prevent concurrent work on same ticket
- ✅ `--keep-worktree` flag for quick PR fixes
- ✅ Manual cleanup with `/cleanup-worktree` command
- ✅ Automatic fallback to main repo on errors

**Learn more:** [docs/worktrees.md](./docs/worktrees.md)
```

**Step 3: Verify changes**

Run: `cat Packs/team-dev-workflow/README.md | grep -A 20 "Git Worktrees"`
Expected: New section appears

**Step 4: Commit**

```bash
git add Packs/team-dev-workflow/README.md
git commit -m "docs: add worktree features to README

Highlighted git worktree support in README:
- Parallel ticket development examples
- Feature list
- Link to detailed documentation

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 9: Create Integration Test

**Files:**
- Create: `Packs/team-dev-workflow/tests/worktree-integration-test.md`

**Step 1: Create tests directory**

Run: `mkdir -p Packs/team-dev-workflow/tests`
Expected: Directory created

**Step 2: Create test specification**

Create file with:

```markdown
# Worktree Integration Test

**Purpose:** Verify worktree creation, cleanup, and edge case handling.

---

## Test Setup

**Prerequisites:**
1. Clean git repository with no existing worktrees
2. Linear MCP configured with test workspace
3. Two test tickets created: TEST-001, TEST-002

---

## Test 1: Basic Worktree Creation

**Steps:**
1. Run: `work on TEST-001`
2. Verify worktree created: `git worktree list | grep TEST-001`
3. Verify lock file created: `ls .worktrees/.locks/TEST-001.lock`
4. Verify working directory changed: `pwd | grep .worktrees/feat-TEST-001`

**Expected:**
- ✅ Worktree created at `.worktrees/feat-TEST-001-<slug>/`
- ✅ Lock file exists
- ✅ Working directory is worktree path
- ✅ Branch `feat/TEST-001-<slug>` exists

---

## Test 2: Parallel Ticket Development

**Steps:**
1. Start Session 1: `work on TEST-001`
2. Start Session 2: `work on TEST-002`
3. Verify both worktrees exist: `git worktree list`
4. Verify no conflicts: Check both sessions working independently

**Expected:**
- ✅ Two worktrees created
- ✅ No branch checkout conflicts
- ✅ Each session isolated from the other

---

## Test 3: Auto-Cleanup After PR

**Steps:**
1. Run: `work on TEST-001` (without --keep-worktree)
2. Complete workflow through Phase 8 (PR creation)
3. Verify worktree removed: `ls .worktrees/ | grep TEST-001`
4. Verify lock file removed: `ls .worktrees/.locks/TEST-001.lock`
5. Verify branch still on remote: `git ls-remote | grep TEST-001`

**Expected:**
- ✅ Worktree directory removed
- ✅ Lock file removed
- ✅ Remote branch preserved (for PR review)
- ✅ Local branch preserved (not merged yet)

---

## Test 4: Keep Worktree Flag

**Steps:**
1. Run: `work on TEST-001 --keep-worktree`
2. Complete workflow through Phase 8 (PR creation)
3. Verify worktree still exists: `ls .worktrees/ | grep TEST-001`
4. Run: `/cleanup-worktree TEST-001`
5. Verify worktree removed: `ls .worktrees/ | grep TEST-001`

**Expected:**
- ✅ Worktree preserved after PR creation
- ✅ User message shows preservation notice
- ✅ Manual cleanup removes worktree successfully

---

## Test 5: Existing Worktree Detection

**Steps:**
1. Run: `work on TEST-001` (creates worktree)
2. Run: `work on TEST-001` again (should detect existing)
3. Choose option 1 (Resume work)
4. Verify working directory is existing worktree

**Expected:**
- ✅ Prompt displayed with 3 options
- ✅ Option 1 resumes in existing worktree
- ✅ No new worktree created
- ✅ Working directory is correct

---

## Test 6: Concurrent Agent Detection

**Steps:**
1. Session 1: `work on TEST-001` (creates lock)
2. Session 2: `work on TEST-001` (should be blocked)
3. Verify Session 2 shows error message
4. Complete Session 1 workflow (removes lock)
5. Session 2: `work on TEST-001` (should succeed now)

**Expected:**
- ✅ Session 2 blocked with clear error message
- ✅ Lock file path shown in error
- ✅ After Session 1 completes, Session 2 can proceed

---

## Test 7: Cleanup All Merged Worktrees

**Steps:**
1. Create 3 worktrees: TEST-001, TEST-002, TEST-003
2. Merge TEST-001 and TEST-002 PRs on GitHub
3. Run: `/cleanup-worktree all`
4. Verify only TEST-003 worktree remains: `git worktree list`

**Expected:**
- ✅ TEST-001 worktree removed (merged)
- ✅ TEST-002 worktree removed (merged)
- ✅ TEST-003 worktree preserved (not merged)
- ✅ Summary shows "Cleaned up 2 merged worktrees"

---

## Test 8: Uncommitted Changes Safety

**Steps:**
1. Run: `work on TEST-001`
2. Make changes but don't commit: `echo "test" > test.txt`
3. Run: `/cleanup-worktree TEST-001`
4. Verify error message about uncommitted changes
5. Commit changes and retry cleanup

**Expected:**
- ✅ Cleanup blocked with error message
- ✅ Error suggests commit or stash options
- ✅ After commit, cleanup succeeds

---

## Test 9: Disk Space Fallback

**Steps:**
1. Simulate disk full condition (difficult to test)
2. Run: `work on TEST-001`
3. Verify fallback message: "Falling back to main repo workflow"
4. Verify traditional branch checkout: `git branch | grep TEST-001`

**Expected:**
- ✅ Worktree creation fails gracefully
- ✅ Fallback message displayed
- ✅ Traditional branch checkout succeeds
- ✅ Workflow continues normally

---

## Test 10: Orphaned Worktree Warning

**Steps:**
1. Create 4 worktrees (exceeds threshold of 3)
2. Run: `work on TEST-005` (creates 5th worktree)
3. Verify warning message displayed

**Expected:**
- ✅ Warning: "You have 5 active worktrees"
- ✅ Suggestion to run `/cleanup-worktree all`
- ✅ Workflow continues normally

---

## Test 11: List Worktrees Command

**Steps:**
1. Create 2 worktrees: TEST-001 (merged), TEST-002 (not merged)
2. Make uncommitted changes in TEST-002
3. Run: `/cleanup-worktree --list`
4. Verify output shows status for each worktree

**Expected:**
- ✅ TEST-001 shows "✅ Merged" status
- ✅ TEST-002 shows "Not merged" status
- ✅ TEST-002 shows "⚠️ Uncommitted changes"
- ✅ Worktree paths and branch names displayed

---

## Test 12: Configuration Toggle

**Steps:**
1. Disable worktrees in config: `worktree.enabled: false`
2. Run: `work on TEST-001`
3. Verify traditional branch checkout (no worktree)
4. Re-enable worktrees: `worktree.enabled: true`
5. Run: `work on TEST-002`
6. Verify worktree created

**Expected:**
- ✅ With disabled config, no worktree created
- ✅ Traditional branch checkout works
- ✅ With enabled config, worktree created
- ✅ No errors in either mode

---

## Manual Verification Checklist

After running automated tests, manually verify:

- [ ] Worktree directories are properly isolated (changes in one don't affect others)
- [ ] Lock files prevent true concurrent access (not just a warning)
- [ ] Auto-cleanup doesn't remove worktrees with uncommitted changes
- [ ] `--keep-worktree` flag parsing works from CORE skill routing
- [ ] Worktree paths are consistent and predictable
- [ ] Error messages are clear and actionable
- [ ] Main repo stays on `main` branch throughout
- [ ] `.worktrees/` directory is gitignored (not tracked)

---

## Success Criteria

**All tests must pass:**
- ✅ Basic creation and cleanup
- ✅ Parallel development without conflicts
- ✅ Flag parsing and propagation
- ✅ Safety checks (uncommitted changes, concurrent agents)
- ✅ Edge case handling (existing worktrees, disk failures)
- ✅ Manual commands work correctly
- ✅ Configuration changes respected
```

**Step 3: Verify test file**

Run: `cat Packs/team-dev-workflow/tests/worktree-integration-test.md | head -100`
Expected: Test specification with proper structure

**Step 4: Commit**

```bash
git add Packs/team-dev-workflow/tests/
git commit -m "test: add worktree integration test specification

Created comprehensive test plan covering:
- Basic worktree creation and cleanup
- Parallel ticket development
- Flag parsing (--keep-worktree)
- Edge cases (existing worktrees, concurrent agents)
- Manual cleanup commands
- Configuration toggles
- Safety checks

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Task 10: Final Verification

**Files:**
- All modified/created files

**Step 1: Verify all files exist**

Run:
```bash
ls -la Packs/team-dev-workflow/src/config/workflow-config.yaml
ls -la Packs/team-dev-workflow/src/skills/TeamDev/Workflows/WorkOnTicket.md
ls -la Packs/team-dev-workflow/src/skills/TeamDev/Workflows/ShipFeature.md
ls -la Packs/team-dev-workflow/src/skills/CleanupWorktree/COMMAND.md
ls -la Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md
ls -la ~/.claude/skills/CORE/SKILL.md
ls -la Packs/team-dev-workflow/docs/worktrees.md
ls -la Packs/team-dev-workflow/tests/worktree-integration-test.md
```

Expected: All files exist

**Step 2: Verify git status is clean**

Run: `git status`
Expected: "nothing to commit, working tree clean" (all changes committed)

**Step 3: Verify commit history**

Run: `git log --oneline -10`
Expected: 10 commits following conventional commit format

**Step 4: Run final grep verification**

```bash
# Verify worktree config added
grep -A 5 "WORKTREE CONFIGURATION" Packs/team-dev-workflow/src/config/workflow-config.yaml

# Verify Phase 0 modified
grep -A 10 "Create Feature Branch with Worktree" Packs/team-dev-workflow/src/skills/TeamDev/Workflows/WorkOnTicket.md

# Verify cleanup command exists
grep -A 5 "cleanup-worktree" Packs/team-dev-workflow/src/skills/CleanupWorktree/COMMAND.md

# Verify flag documented
grep -A 5 "keep-worktree" Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md
```

Expected: All greps return results

**Step 5: Create summary commit**

```bash
git commit --allow-empty -m "feat: complete worktree integration for team-dev-workflow

Integrated git worktrees for parallel ticket development:

Configuration:
- Added worktree section to workflow-config.yaml
- Configurable base path, cleanup behavior, locking

Workflow Changes:
- Phase 0: Creates worktrees instead of branch checkout
- Phase 8: Auto-cleanup after PR (respects --keep-worktree)
- Added lock files to prevent concurrent agents

Commands:
- /cleanup-worktree <ticket> - Manual cleanup
- /cleanup-worktree all - Clean merged worktrees
- /cleanup-worktree --list - List active worktrees

Documentation:
- Comprehensive worktree guide in docs/worktrees.md
- Updated README with worktree features
- Created integration test specification

Benefits:
- Run multiple tickets simultaneously
- No branch switching or context loss
- Automatic cleanup keeps workspace tidy
- Lock files prevent conflicts

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Implementation Complete

**Files Created:**
1. ✅ `Packs/team-dev-workflow/src/skills/CleanupWorktree/COMMAND.md`
2. ✅ `Packs/team-dev-workflow/docs/worktrees.md`
3. ✅ `Packs/team-dev-workflow/tests/worktree-integration-test.md`

**Files Modified:**
1. ✅ `Packs/team-dev-workflow/src/config/workflow-config.yaml`
2. ✅ `Packs/team-dev-workflow/src/skills/TeamDev/Workflows/WorkOnTicket.md`
3. ✅ `Packs/team-dev-workflow/src/skills/TeamDev/Workflows/ShipFeature.md`
4. ✅ `Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md`
5. ✅ `~/.claude/skills/CORE/SKILL.md`
6. ✅ `Packs/team-dev-workflow/README.md`

**Total Commits:** 11 (one per task + final summary)

**Next Steps:**
1. Run integration tests (Task 9 test specification)
2. Test with real Linear tickets
3. Gather user feedback
4. Iterate based on usage patterns
