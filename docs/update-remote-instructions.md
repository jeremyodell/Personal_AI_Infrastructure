# Update PAI Remote Repository Instructions

These instructions will help you point your PAI project to the new remote repository and get the latest commits.

## Prerequisites

- You have a local PAI project directory
- You have git installed
- You have SSH access configured for GitHub (or will use HTTPS)

## Step-by-Step Instructions

### 1. Backup Your Current Work (Optional but Recommended)

If you have uncommitted changes you want to keep:

```bash
# Check current status
git status

# If you have changes, stash them
git stash save "Backup before remote update"
```

### 2. Check Your Current Remote

```bash
# View current remote configuration
git remote -v
```

### 3. Update the Remote URL

Choose **one** of the following options:

#### Option A: Update Existing Remote (Recommended)

```bash
# Update the origin remote to point to the new repository
git remote set-url origin git@github.com:jeremyodell/Personal_AI_Infrastructure.git
```

#### Option B: Remove and Re-add Remote

```bash
# Remove the old remote
git remote remove origin

# Add the new remote
git remote add origin git@github.com:jeremyodell/Personal_AI_Infrastructure.git
```

#### Option C: Use HTTPS Instead of SSH

If you don't have SSH keys configured:

```bash
# Update to HTTPS URL
git remote set-url origin https://github.com/jeremyodell/Personal_AI_Infrastructure.git
```

### 4. Verify the Remote Update

```bash
# Confirm the remote is now pointing to the correct repository
git remote -v
```

You should see:
```
origin  git@github.com:jeremyodell/Personal_AI_Infrastructure.git (fetch)
origin  git@github.com:jeremyodell/Personal_AI_Infrastructure.git (push)
```

### 5. Fetch the Latest Changes

```bash
# Fetch all branches and tags from the new remote
git fetch origin
```

### 6. Update Your Local Branch

#### If You're on the Main Branch:

```bash
# Make sure you're on main
git checkout main

# Pull the latest changes
git pull origin main
```

#### If You Have Local Commits to Preserve:

```bash
# Rebase your local commits on top of the remote changes
git pull --rebase origin main
```

#### If You Want a Clean Slate:

```bash
# Reset your local branch to match the remote exactly (CAUTION: loses local commits)
git checkout main
git reset --hard origin/main
```

### 7. Restore Your Stashed Changes (If You Stashed Earlier)

```bash
# List your stashes
git stash list

# Apply the most recent stash
git stash pop
```

## Verification

Confirm everything is working:

```bash
# Check your current branch and status
git status

# View recent commits to confirm you have the latest
git log --oneline -5

# Test that you can fetch updates
git fetch origin
```

## Recent Commits You Should See

After updating, you should see these recent commits:

- `2c8d1c7` - chore: merge stashed changes - add skill invocation enforcement to TeamDev
- `554734d` - Merge feat/worktree-integration: Add git worktree support to team-dev-workflow
- `4185d3f` - feat: complete worktree integration implementation
- `1cd134e` - test: add worktree integration test specification
- `6020fe5` - docs: highlight worktree features in README

## Troubleshooting

### SSH Key Issues

If you get a "Permission denied (publickey)" error:

```bash
# Test your SSH connection
ssh -T git@github.com

# If it fails, you may need to add your SSH key to GitHub
# Or use the HTTPS URL instead (see Option C above)
```

### Merge Conflicts

If you encounter merge conflicts when pulling:

```bash
# View conflicted files
git status

# Edit the files to resolve conflicts
# Then stage the resolved files
git add <resolved-files>

# Complete the merge
git commit
```

### Need Help?

- Check the repository: https://github.com/jeremyodell/Personal_AI_Infrastructure
- Review git documentation: `git help <command>`

## Summary

After following these steps, your PAI project will be:
- ✅ Pointing to the new remote repository
- ✅ Updated with the latest commits from the main branch
- ✅ Ready to push your own changes (if you have push access)

---

**Quick Reference One-Liner:**

For experienced users who just need a quick update:

```bash
git remote set-url origin git@github.com:jeremyodell/Personal_AI_Infrastructure.git && git fetch origin && git pull origin main
```
