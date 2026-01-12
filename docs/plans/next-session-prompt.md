# Next Session: Implement plan-to-linear Pack

## Context

In the previous session, I successfully installed the **team-dev-workflow** pack with:
- ✅ All required plugins (feature-dev, frontend-design, pr-review-toolkit)
- ✅ Linear MCP integration configured
- ✅ PAI CORE routing linked
- ✅ GitHub CLI verified

## Task for This Session

**Implement the plan-to-linear pack** using the existing implementation plan.

## Files You Need

- **Design:** `docs/plans/2026-01-08-plan-to-linear-design.md`
- **Implementation Plan:** `docs/plans/2026-01-08-plan-to-linear-implementation.md`

## What to Do

Use the superpowers:executing-plans skill to implement the plan-to-linear pack from the implementation plan document.

The pack should:
1. Transform feature ideas into Linear stories
2. Auto-detect frontend/backend/both needs
3. Use feature-dev and frontend-design skills for design phase
4. Use writing-plans skill for story decomposition
5. Apply smart labels (pre-planned + concern + keywords)
6. Upload to Linear with proper hierarchy and dependencies

## Installation After Implementation

Once implemented, install it by:
1. Adding routing to `~/.claude/skills/CORE/SKILL.md`
2. Testing with a sample feature idea
3. Creating verification checklist

## Start Command

```
Read docs/plans/2026-01-08-plan-to-linear-implementation.md and implement the plan-to-linear pack using superpowers:executing-plans
```
