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
