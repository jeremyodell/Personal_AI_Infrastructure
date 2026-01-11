# Team Development Workflow - Installation Guide

**Step-by-step installation for the team-dev-workflow PAI pack**

Follow these steps carefully to set up the complete workflow.

---

## Prerequisites

Before installing, ensure you have:

- [ ] **Claude Code CLI** installed and working
- [ ] **PAI v2.0** infrastructure set up
- [ ] **Linear account** with API access
- [ ] **GitHub CLI** (`gh`) installed and authenticated
- [ ] **Node.js** and **npm** installed (for quality gates)
- [ ] **Git** configured with user.name and user.email

---

## Installation Steps

### Step 1: Install Required Anthropic Plugins

```bash
# Install all required plugins
claude plugin install feature-dev@claude-plugins-official
claude plugin install frontend-design@claude-plugins-official
claude plugin install ralph-wiggum@claude-plugins-official
claude plugin install pr-review-toolkit@claude-plugins-official
```

**Verify installation:**
```bash
claude plugin list

# Should show:
# - feature-dev
# - frontend-design
# - ralph-wiggum
# - pr-review-toolkit
```

---

### Step 2: Configure Linear MCP Integration

**1. Add Linear MCP to Claude config:**

Edit `~/.claude/config.json`:
```json
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.linear.app/mcp"]
    }
  }
}
```

**2. Test Linear connection:**
```bash
# Start Claude Code and run:
/list-tools

# Should include tools like:
# - mcp__linear__get_issue
# - mcp__linear__list_issues
# - mcp__linear__create_comment
```

**3. Test fetch a ticket:**
```bash
# In Claude Code session:
mcp__linear__list_issues({ assignee: "me", limit: 5 })

# Should return your Linear tickets
```

---

### Step 3: Copy Pack to PAI Directory

```bash
# Copy the team-dev-workflow pack
cp -r /path/to/team-dev-workflow $HOME/dev/projects/personal-ai/PAI/Packs/

# Verify structure
ls -la $HOME/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/

# Should show:
# README.md, INSTALL.md, VERIFY.md, src/
```

---

### Step 4: Configure GitHub CLI

**1. Authenticate GitHub CLI:**
```bash
gh auth login

# Follow prompts:
# - GitHub.com
# - HTTPS
# - Login with browser
```

**2. Verify authentication:**
```bash
gh auth status

# Should show: Logged in to github.com as yourusername
```

**3. Test PR creation (dry run):**
```bash
# In your project repo:
gh pr create --dry-run \
  --title "Test PR" \
  --body "Testing gh CLI" \
  --base main \
  --head test-branch

# Should show what would be created
```

---

### Step 5: Configure Project npm Scripts

**Ensure your project has these scripts in `package.json`:**

```json
{
  "scripts": {
    "test": "jest",  // or vitest, mocha, etc.
    "lint": "eslint src/",
    "typecheck": "tsc --noEmit",
    "build": "vite build"  // or webpack, next build, etc.
  }
}
```

**Test each script works:**
```bash
npm test
npm run lint
npm run typecheck
npm run build

# All should complete successfully (or with expected results)
```

---

### Step 6: Create Linear Labels

**In your Linear workspace, create these labels:**

```yaml
# Routing labels
pre-planned:
  description: "Architecture pre-planned, skip discovery"
  color: "#3b82f6"

# Feature type labels
ui:
  description: "User interface implementation"
  color: "#8b5cf6"

frontend:
  description: "Frontend code"
  color: "#8b5cf6"

backend:
  description: "Backend/API code"
  color: "#10b981"

# Urgency labels
bug:
  description: "Bug fix"
  color: "#ef4444"

hotfix:
  description: "Urgent production fix"
  color: "#dc2626"

# Research labels
spike:
  description: "Research/exploration spike"
  color: "#f59e0b"

research:
  description: "Research task"
  color: "#f59e0b"
```

**Create via Linear UI:**
1. Go to Settings → Labels
2. Click "Create Label"
3. Enter name, description, color
4. Save

**Or create via MCP:**
```typescript
await mcp__linear__create_issue_label({
  name: "pre-planned",
  description: "Architecture pre-planned, skip discovery",
  color: "#3b82f6"
});
// Repeat for other labels
```

---

### Step 7: Configure Workflow Settings

**Edit `src/config/workflow-config.yaml` for your project:**

```yaml
# Adjust quality gate commands for your project
quality_gates:
  gates:
    test:
      command: npm test  # Or: yarn test, pnpm test
    lint:
      command: npm run lint  # Or: yarn lint
    typecheck:
      command: npm run typecheck  # Or: yarn typecheck
    build:
      command: npm run build  # Or: yarn build

# Adjust Linear state names if different
linear:
  states:
    start: "In Progress"  # Or: "Started", "Doing"
    review: "In Review"   # Or: "Review", "Reviewing"
    done: "Done"          # Or: "Complete", "Closed"

# Adjust base branch if not 'main'
pull_request:
  settings:
    base_branch: main  # Or: master, develop
```

---

### Step 8: Install TeamDev Skill Files

**CRITICAL: Copy skill files so Claude Code can discover them.**

```bash
# Set paths
PACK_DIR="$(pwd)"
PAI_DIR="${PAI_DIR:-$HOME/.claude}"

# Create skill directory structure
mkdir -p "$PAI_DIR/skills/TeamDev/Workflows"
mkdir -p "$PAI_DIR/skills/TeamDev/Tools"

# Copy skill files from pack to PAI skills directory
cp "$PACK_DIR/src/skills/TeamDev/SKILL.md" "$PAI_DIR/skills/TeamDev/"
cp "$PACK_DIR/src/skills/TeamDev/Workflows/WorkOnTicket.md" "$PAI_DIR/skills/TeamDev/Workflows/"
cp "$PACK_DIR/src/skills/TeamDev/Workflows/RunQualityGates.md" "$PAI_DIR/skills/TeamDev/Workflows/"
cp "$PACK_DIR/src/skills/TeamDev/Workflows/ShipFeature.md" "$PAI_DIR/skills/TeamDev/Workflows/"
cp "$PACK_DIR/src/skills/TeamDev/Tools/LinearHelpers.md" "$PAI_DIR/skills/TeamDev/Tools/"
cp "$PACK_DIR/src/skills/TeamDev/Tools/TicketRouter.md" "$PAI_DIR/skills/TeamDev/Tools/"
cp "$PACK_DIR/src/skills/TeamDev/Tools/RalphOrchestrator.md" "$PAI_DIR/skills/TeamDev/Tools/"

# Verify installation
ls -la "$PAI_DIR/skills/TeamDev/"
echo "✓ TeamDev skill files installed"
```

**Files installed:**
- `SKILL.md` - Main skill routing and workflow orchestration
- `Workflows/WorkOnTicket.md` - End-to-end ticket workflow (Phases 0-8)
- `Workflows/RunQualityGates.md` - Quality gate runner with auto-fix
- `Workflows/ShipFeature.md` - Git + PR + Linear sync
- `Tools/LinearHelpers.md` - Linear MCP wrapper patterns
- `Tools/TicketRouter.md` - Label detection and routing logic
- `Tools/RalphOrchestrator.md` - Ralph-loop management utilities

---

### Step 9: Link Pack to PAI CORE

**Add TeamDev skill to PAI routing:**

Edit `$HOME/.claude/skills/CORE/SKILL.md`:

```markdown
## Skill Routing

**Team Development Workflow:**

When user says:
- "work on ENG-123"
- "work on ticket ENG-456"
- "implement ENG-789"
- "start ENG-111"
- "start ticket ENG-222"

→ Invoke TeamDev skill from team-dev-workflow pack

**Path:** `~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/skills/TeamDev/SKILL.md`
```

**Note:** This routing is already in CORE if you followed the initial setup. Verify it exists:

```bash
grep -A 5 "Team Development Workflow" ~/.claude/skills/CORE/SKILL.md
```

---

### Step 10: Test Basic Workflow

**Create a test ticket in Linear:**

1. Go to Linear
2. Create new issue
3. Title: "Test team-dev-workflow pack"
4. Description: "Simple test to verify pack installation"
5. Labels: ["pre-planned", "backend"]  (to skip discovery)
6. Note the ticket ID (e.g., ENG-999)

**Run workflow:**
```bash
# Start Claude Code
claude code

# In session:
work on ENG-999

# Should output:
# ✅ Phase 0 Complete - Ticket Analysis
# - Issue: ENG-999 - Test team-dev-workflow pack
# - Labels: [pre-planned, backend]
# - Routing: SKIP to Implementation
# - Branch: feat/ENG-999-test-team-dev-workflow-pack
```

**Cancel before implementation:**
```bash
# Press Ctrl+C to exit
# Linear should have a comment about work starting
```

---

### Step 11: Verify Installation

**Run through VERIFY.md checklist:**

```bash
# See VERIFY.md for full verification steps
cat $HOME/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/VERIFY.md
```

---

## Optional: Configure Hooks

**If you want hooks enabled (recommended):**

Hooks are defined in `src/hooks/HOOKS.md` but need to be implemented in PAI's hook system.

**For now, hooks are documentation-only.** When PAI v2.0 hook system is ready:

1. Review `src/hooks/HOOKS.md`
2. Implement hooks in PAI hook system
3. Configure in `src/config/workflow-config.yaml`:
   ```yaml
   hooks:
     linear_sync_on_complete:
       enabled: true
     quality_gate_blocker:
       enabled: true
   ```

---

## Troubleshooting

### Issue: "Linear MCP not found"

**Solution:**
```bash
# Check config.json syntax
cat ~/.claude/config.json | jq .

# Restart Claude Code
claude code --restart

# Test MCP connection
npx -y mcp-remote https://mcp.linear.app/mcp
```

### Issue: "feature-dev plugin not found"

**Solution:**
```bash
# Reinstall plugin
claude plugin uninstall feature-dev
claude plugin install feature-dev@claude-plugins-official

# Verify
claude plugin list
```

### Issue: "gh: command not found"

**Solution:**
```bash
# Install GitHub CLI
# macOS:
brew install gh

# Ubuntu/Debian:
sudo apt install gh

# Windows:
winget install --id GitHub.cli

# Authenticate
gh auth login
```

### Issue: "Quality gates fail - command not found"

**Solution:**
```bash
# Ensure npm scripts exist
cat package.json | jq .scripts

# Add missing scripts
npm install --save-dev jest eslint typescript
# Then add scripts to package.json
```

### Issue: "Cannot create branch - already exists"

**Solution:**
```bash
# Delete old branch
git branch -D feat/ENG-123-old-feature

# Or checkout and continue work
git checkout feat/ENG-123-old-feature
```

---

## Post-Installation

**After successful installation:**

1. ✅ **Test with a real ticket** - Pick a small, pre-planned ticket
2. ✅ **Verify all phases work** - Run through Phases 0-8
3. ✅ **Check Linear integration** - Ensure status updates work
4. ✅ **Test PR creation** - Verify GitHub integration
5. ✅ **Customize config** - Adjust `workflow-config.yaml` for your team
6. ✅ **Document team conventions** - Add to project CLAUDE.md

---

## Updating the Pack

**To update to a new version:**

```bash
# Backup your config
cp src/config/workflow-config.yaml ~/workflow-config.backup.yaml

# Pull latest pack
cd PAI/Packs/team-dev-workflow
git pull  # If in git repo

# Restore your config
cp ~/workflow-config.backup.yaml src/config/workflow-config.yaml

# Verify changes
diff ~/workflow-config.backup.yaml src/config/workflow-config.yaml
```

---

## Uninstallation

**To remove the pack:**

```bash
# 1. Remove pack directory
rm -rf $HOME/dev/projects/personal-ai/PAI/Packs/team-dev-workflow

# 2. Remove PAI routing (if added)
# Edit PAI CORE SKILL.md and remove TeamDev routing

# 3. Optionally uninstall plugins
claude plugin uninstall feature-dev
claude plugin uninstall frontend-design
claude plugin uninstall ralph-wiggum
claude plugin uninstall pr-review-toolkit

# 4. Optionally remove Linear MCP
# Edit ~/.claude/config.json and remove linear entry
```

---

## Getting Help

**Resources:**
- **README.md** - Pack overview and usage
- **VERIFY.md** - Verification checklist
- **Workflow docs** - `src/skills/TeamDev/Workflows/`
- **Tool docs** - `src/skills/TeamDev/Tools/`
- **Config reference** - `src/config/workflow-config.yaml`

**Debugging:**
```bash
# Enable debug logging
DEBUG=team-dev-workflow:* work on ENG-123

# Check logs
tail -f ~/.claude/logs/team-dev-workflow.log
```

---

## Next Steps

After installation:

1. 📖 **Read README.md** - Understand workflow phases
2. ✅ **Complete VERIFY.md** - Verify installation
3. 🎯 **Test with small ticket** - Try pre-planned backend feature
4. ⚙️ **Customize config** - Adjust for your project
5. 🚀 **Use in production** - Integrate into daily workflow

---

**Installation Complete!** 🎉

You're ready to use the team-dev-workflow pack. Try:
```bash
work on ENG-123
```

---

**Version:** 1.0.0
**Last Updated:** 2026-01-08
