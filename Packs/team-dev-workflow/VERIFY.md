# Team Development Workflow - Verification Checklist

**Complete this checklist to verify your installation is working correctly**

Work through each section systematically. Mark items as complete as you go.

---

## Section 1: Prerequisites

**Verify all prerequisites are met:**

- [ ] Claude Code CLI installed (`claude --version` works)
- [ ] PAI v2.0 directory exists at `~/dev/projects/personal-ai/PAI/`
- [ ] Linear account accessible (can log in to linear.app)
- [ ] GitHub CLI installed (`gh --version` shows version)
- [ ] GitHub CLI authenticated (`gh auth status` shows logged in)
- [ ] Node.js installed (`node --version` shows v16+)
- [ ] npm installed (`npm --version` works)
- [ ] Git configured (`git config user.name` and `git config user.email` set)

---

## Section 2: Plugin Installation

**Verify all plugins installed:**

```bash
claude plugin list
```

- [ ] `feature-dev` listed
- [ ] `frontend-design` listed
- [ ] `ralph-wiggum` listed
- [ ] `pr-review-toolkit` listed

**Test plugin loading:**
```bash
# Start Claude Code
claude code

# Try loading a plugin
/skill feature-dev

# Should show: "Launching skill: feature-dev:feature-dev"
```

- [ ] feature-dev skill loads without errors

---

## Section 3: Linear MCP Integration

**Verify Linear MCP configured:**

```bash
# Check config file exists
cat ~/.claude/config.json | jq '.mcpServers.linear'

# Should output Linear MCP configuration
```

- [ ] Linear MCP entry exists in config.json
- [ ] Config has "command": "npx"
- [ ] Config has correct mcp-remote URL

**Test Linear connection:**

```bash
# In Claude Code session
claude code

# List available tools
/list-tools

# Should see Linear tools:
# - mcp__linear__get_issue
# - mcp__linear__list_issues
# - mcp__linear__create_comment
# - etc.
```

- [ ] Linear MCP tools listed

**Test fetching tickets:**

```bash
# In Claude Code, run:
Fetch my Linear tickets

# Or directly:
mcp__linear__list_issues({ assignee: "me", limit: 5 })

# Should return your tickets
```

- [ ] Can fetch Linear tickets successfully
- [ ] Tickets show correct data (title, status, labels)

---

## Section 4: Pack Installation

**Verify pack structure:**

```bash
ls -la ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/

# Should show:
# README.md
# INSTALL.md
# VERIFY.md (this file)
# src/
```

- [ ] Pack directory exists
- [ ] README.md present
- [ ] INSTALL.md present
- [ ] VERIFY.md present
- [ ] src/ directory present

**Verify skill structure:**

```bash
ls -la ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/skills/TeamDev/

# Should show:
# SKILL.md
# Workflows/
# Tools/
```

- [ ] SKILL.md exists
- [ ] Workflows/ directory exists with 3 files
- [ ] Tools/ directory exists with 3 files

**Verify config exists:**

```bash
cat ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/config/workflow-config.yaml | head -20

# Should show YAML configuration
```

- [ ] workflow-config.yaml exists and is valid YAML

---

## Section 5: Project Configuration

**Verify project has required npm scripts:**

```bash
# In your project directory
cat package.json | jq '.scripts'

# Should have:
# - test
# - lint
# - typecheck
# - build
```

- [ ] `npm test` script exists
- [ ] `npm run lint` script exists
- [ ] `npm run typecheck` script exists
- [ ] `npm run build` script exists

**Test npm scripts run:**

```bash
npm test
# Should run tests (passing or failing)
```

- [ ] `npm test` completes (don't worry about pass/fail for now)

```bash
npm run lint
# Should run linter
```

- [ ] `npm run lint` completes

```bash
npm run typecheck
# Should run type checker
```

- [ ] `npm run typecheck` completes

```bash
npm run build
# Should run build
```

- [ ] `npm run build` completes

---

## Section 6: Linear Labels

**Verify Linear labels exist:**

Go to Linear → Settings → Labels

- [ ] `pre-planned` label exists
- [ ] `ui` label exists
- [ ] `frontend` label exists
- [ ] `backend` label exists
- [ ] `bug` label exists
- [ ] `hotfix` label exists
- [ ] `spike` label exists
- [ ] `research` label exists

**Or verify via MCP:**

```bash
# In Claude Code
mcp__linear__list_issue_labels({ limit: 100 })

# Should list all labels
```

- [ ] All routing labels present in Linear

---

## Section 7: GitHub Integration

**Verify GitHub CLI works:**

```bash
gh auth status

# Should show: Logged in to github.com as yourusername
```

- [ ] GitHub CLI authenticated

**Test PR creation (dry run):**

```bash
# In your project repo
git checkout -b test-verify-pack
echo "test" > test-verify.txt
git add test-verify.txt
git commit -m "test: verify pack installation"

gh pr create --dry-run \
  --title "Test: Verify Pack" \
  --body "Testing gh CLI integration" \
  --base main \
  --head test-verify-pack

# Should show what would be created (don't actually create)
```

- [ ] Dry run PR creation works
- [ ] Shows PR details without creating

**Clean up test branch:**

```bash
git checkout main
git branch -D test-verify-pack
```

- [ ] Test branch cleaned up

---

## Section 8: Phase 0 (Setup & Routing)

**Create a test ticket in Linear:**

1. Create new issue
2. Title: "VERIFY: Test Phase 0 - Setup and Routing"
3. Description: "Testing team-dev-workflow pack Phase 0"
4. Labels: `pre-planned`, `backend`
5. Assign to yourself
6. Note the ticket ID (e.g., ENG-999)

- [ ] Test ticket created in Linear
- [ ] Ticket has both required labels
- [ ] Ticket ID noted: _____________

**Test Phase 0:**

```bash
# Start Claude Code
claude code

# Run workflow
work on ENG-999
```

**Expected output:**
```
✅ Phase 0 Complete - Ticket Analysis

📋 SUMMARY:
- Issue: ENG-999 - VERIFY: Test Phase 0 - Setup and Routing
- Labels: [pre-planned, backend]
- Routing: SKIP to Implementation
- Branch: feat/ENG-999-verify-test-phase-0-setup-and-routing

🔍 ROUTING DECISIONS:
- Skip discovery/architecture: YES (pre-planned label detected)
- Use frontend-design: NO
```

- [ ] Phase 0 completes successfully
- [ ] Correct ticket information displayed
- [ ] Labels detected correctly
- [ ] Routing decision correct (skip to Phase 5)
- [ ] Branch name created correctly

**Verify branch created:**

```bash
git branch | grep feat/ENG-999

# Should show: feat/ENG-999-verify-test-phase-0-setup-and-routing
```

- [ ] Feature branch created
- [ ] Branch name matches expected format

**Verify Linear updated:**

Check Linear ticket - should have:
- Status: "In Progress"
- Comment about workflow starting

- [ ] Linear status updated to "In Progress"
- [ ] (Optional) Comment posted to ticket

**Cancel workflow:**

```bash
# In Claude Code, press Ctrl+C
```

- [ ] Workflow stopped cleanly

---

## Section 9: Routing Logic

**Test different routing scenarios:**

### Test 1: Pre-Planned + UI

Create ticket:
- Title: "VERIFY: Test UI Routing"
- Labels: `pre-planned`, `ui`

```bash
work on ENG-XXX
```

- [ ] Detects "use frontend-design: YES"
- [ ] Skips to Phase 5
- [ ] Plans to use frontend-design plugin

### Test 2: No Special Labels

Create ticket:
- Title: "VERIFY: Test Full Workflow"
- Labels: (none)

```bash
work on ENG-XXX
```

- [ ] Detects "skip discovery: NO"
- [ ] Plans full workflow (Phases 1-4 then 5-8)

### Test 3: Research Spike

Create ticket:
- Title: "VERIFY: Test Research Spike"
- Labels: `spike`

```bash
work on ENG-XXX
```

- [ ] Detects "architecture only: YES"
- [ ] Plans to stop after Phase 4

**Cancel all test workflows after Phase 0**

---

## Section 10: Quality Gates

**Test quality gate execution:**

```bash
# In your project
npm test
npm run lint
npm run typecheck
npm run build
```

- [ ] All gates can run
- [ ] Test output shows pass/fail status
- [ ] Lint output shows errors (if any)
- [ ] Typecheck output shows errors (if any)
- [ ] Build completes

**Test quality gate detection:**

If you have intentional test failures, check:

```bash
npm test
echo $?  # Exit code: 0 = pass, non-zero = fail
```

- [ ] Can detect test failures (non-zero exit code)

---

## Section 11: Git Operations

**Test commit message formatting:**

```bash
# On test branch
echo "test" >> test-file.txt
git add test-file.txt

git commit -m "$(cat <<'EOF'
feat(ENG-999): Test commit message

Testing conventional commit format.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"
```

- [ ] Commit created successfully
- [ ] Commit message follows conventional format
- [ ] Co-author included

**Test branch push:**

```bash
# Don't actually push, but test command syntax
git push --dry-run origin feat/ENG-999-test

# Should show what would be pushed
```

- [ ] Push command syntax correct

---

## Section 12: Linear Comment Posting

**Test posting comment to Linear:**

```bash
# In Claude Code
mcp__linear__create_comment({
  issueId: "ENG-999",  # Your test ticket ID
  body: "## Verification Test\n\nTesting Linear comment posting from team-dev-workflow pack.\n\n✅ Comment posting works!"
})
```

- [ ] Comment posted successfully to Linear
- [ ] Comment visible in Linear ticket
- [ ] Markdown formatted correctly

---

## Section 13: Configuration Customization

**Test editing config:**

```bash
# Backup config
cp ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/config/workflow-config.yaml ~/workflow-config.backup

# Edit a value (e.g., change max iterations)
# Edit: plugins.ralph_wiggum.max_iterations.backend: 50 → 60

# Verify YAML still valid
cat ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/config/workflow-config.yaml | python3 -c "import yaml, sys; yaml.safe_load(sys.stdin)"

# Restore backup
cp ~/workflow-config.backup ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/config/workflow-config.yaml
```

- [ ] Can edit config successfully
- [ ] YAML validation works
- [ ] Config restored from backup

---

## Section 14: Error Handling

**Test Linear connection failure handling:**

```bash
# Temporarily break Linear config
# (Backup first!)
cp ~/.claude/config.json ~/.claude/config.json.backup

# Edit config.json and change Linear URL to invalid
# Then try:
work on ENG-999

# Should show error:
# "❌ ERROR: Linear API connection failed"
```

- [ ] Shows clear error message for Linear failure

**Restore config:**

```bash
cp ~/.claude/config.json.backup ~/.claude/config.json
```

- [ ] Config restored

**Test invalid ticket ID:**

```bash
work on ENG-99999  # Non-existent ticket

# Should show:
# "❌ ERROR: Ticket ENG-99999 not found"
```

- [ ] Shows clear error for invalid ticket

---

## Section 15: Documentation

**Verify all documentation accessible:**

```bash
# README
cat ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/README.md | head -50

# INSTALL
cat ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/INSTALL.md | head -50

# Workflows
ls ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/skills/TeamDev/Workflows/

# Tools
ls ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/skills/TeamDev/Tools/

# Hooks
cat ~/dev/projects/personal-ai/PAI/Packs/team-dev-workflow/src/hooks/HOOKS.md | head -50
```

- [ ] README.md readable and complete
- [ ] INSTALL.md has all steps
- [ ] WorkOnTicket.md exists
- [ ] RunQualityGates.md exists
- [ ] ShipFeature.md exists
- [ ] LinearHelpers.md exists
- [ ] TicketRouter.md exists
- [ ] RalphOrchestrator.md exists
- [ ] HOOKS.md exists

---

## Section 16: Clean Up

**Clean up all test artifacts:**

```bash
# Delete test branches
git branch -D feat/ENG-999-verify-test-phase-0-setup-and-routing
git branch -D test-verify-pack  # If exists

# Clean up test files
git checkout main
rm -f test-verify.txt test-file.txt

# Archive or delete test Linear tickets
# (Or leave them for reference)
```

- [ ] Test branches deleted
- [ ] Test files removed
- [ ] Back on main branch
- [ ] Working directory clean

---

## Verification Results

### Summary

**Count your checkmarks:**

- Total items checked: ______ / ~100+
- Prerequisites: ______ / 8
- Plugins: ______ / 5
- Linear MCP: ______ / 8
- Pack Installation: ______ / 7
- Project Config: ______ / 8
- Linear Labels: ______ / 9
- GitHub Integration: ______ / 5
- Phase 0: ______ / 13
- Routing Logic: ______ / 6
- Quality Gates: ______ / 5
- Git Operations: ______ / 4
- Linear Comments: ______ / 3
- Configuration: ______ / 4
- Error Handling: ______ / 4
- Documentation: ______ / 9
- Clean Up: ______ / 4

### Status

- [ ] ✅ **PASS** - All critical items checked (≥95%)
- [ ] ⚠️ **PARTIAL** - Most items checked (80-94%)
- [ ] ❌ **FAIL** - Many items unchecked (<80%)

---

## Next Steps

### If PASS ✅

**You're ready to use the pack in production!**

Try working on a real ticket:
```bash
work on ENG-<REAL-TICKET-ID>
```

Start with:
1. Small, pre-planned backend feature
2. Then try: pre-planned UI feature
3. Then try: full workflow (no pre-planned label)

### If PARTIAL ⚠️

**Review failed items:**

1. Note which sections had issues
2. Re-read INSTALL.md for those sections
3. Check troubleshooting in INSTALL.md
4. Try verification again

**Common issues:**
- Linear MCP not connecting → Check config.json
- Plugins not loading → Reinstall plugins
- Git operations failing → Check auth

### If FAIL ❌

**Start over with installation:**

1. Review INSTALL.md completely
2. Verify each prerequisite
3. Follow installation steps carefully
4. Run verification after each major step

**Get help:**
- Check INSTALL.md troubleshooting
- Review README.md for architecture
- Check plugin documentation
- Review Linear MCP setup

---

## Troubleshooting

### Issue: Phase 0 doesn't detect labels

**Check:**
```bash
# Verify ticket has labels in Linear UI
# Or check via MCP:
mcp__linear__get_issue({ id: "ENG-999" })

# Check labels property in response
```

### Issue: Quality gates timeout

**Check:**
```bash
# Increase timeout in config:
timeout_ms: 600000  # 10 minutes

# Or test gates manually first
npm test
npm run lint
```

### Issue: GitHub PR creation fails

**Check:**
```bash
gh auth status
gh repo view  # Should show current repo

# Re-authenticate if needed
gh auth login
```

---

## Report Issues

If verification fails and you can't resolve:

1. Document exact error messages
2. Note which verification step failed
3. Check logs: `~/.claude/logs/team-dev-workflow.log`
4. Review configuration files
5. Check plugin versions: `claude plugin list`

---

**Verification Complete!**

Date: __________________
Verified by: __________________
Result: ⬜ PASS ⬜ PARTIAL ⬜ FAIL

---

**Version:** 1.0.0
**Last Updated:** 2026-01-08
