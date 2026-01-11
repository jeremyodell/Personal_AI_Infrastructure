# CORE Skill Modifications for Worktree Integration

**Date:** 2026-01-10

**Purpose:** Document manual changes required to the CORE skill for worktree flag routing

---

## Required Modification

The CORE skill at `~/.claude/skills/CORE/SKILL.md` must be updated to parse and route the `--keep-worktree` flag.

**Location:** Skill Routing section → Team Development Workflow subsection

**Change:**

Find this section:
```markdown
**Team Development Workflow:**

When user says:
- "work on ENG-123"
- "work on ticket ENG-456"
- "implement ENG-789"
- "start ENG-111"
- "start ticket ENG-222"

→ Invoke TeamDev skill from team-dev-workflow pack
```

Update to:
```markdown
**Team Development Workflow:**

When user says:
- "work on ENG-123"
- "work on ticket ENG-456"
- "implement ENG-789"
- "start ENG-111"
- "start ticket ENG-222"

**Flag parsing:**
- Extract `--keep-worktree` flag from user input
- Pass flag state to TeamDev skill invocation
- Example: "work on ENG-123 --keep-worktree" → keep_worktree=true

→ Invoke TeamDev skill from team-dev-workflow pack
```

---

## Status

✅ **Applied:** This modification was applied to the runtime CORE skill on 2026-01-10

---

## Note

The CORE skill resides in `~/.claude/skills/CORE/` (user's runtime configuration directory), which is not under version control. This document serves as a record of the required change for:

1. Installation instructions
2. Future setup documentation
3. Troubleshooting reference
