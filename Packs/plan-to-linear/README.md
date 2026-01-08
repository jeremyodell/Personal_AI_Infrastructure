# plan-to-linear Pack

**Transform feature ideas into structured Linear stories with dependencies.**

Streamlined workflow that:
- Detects frontend/backend/both automatically
- Uses feature-dev and frontend-design for design phase
- Stops before implementation (creates pre-planned stories)
- Applies smart labels automatically
- Handles full-stack features with separate story groups

---

## What It Does

1. **Analyzes** your feature idea to detect frontend, backend, or both
2. **Designs** using existing skills (feature-dev, frontend-design)
3. **Decomposes** designs into stories with natural dependencies
4. **Uploads** to Linear with nested hierarchy and smart labels

---

## Quick Start

```bash
# Command format
/plan-to-linear "Add user authentication with JWT"

# Natural language
"I want to plan this feature"
"create stories for user dashboard"
```

---

## Example Output

### Input
```
/plan-to-linear "Add user authentication with JWT"
```

### Linear Structure Created
```
Parent: "Feature: User Authentication" [pre-planned]
├─ Frontend: "User Authentication" [pre-planned, frontend]
│   ├─ Login form component [pre-planned, frontend, forms]
│   └─ Auth state management [pre-planned, frontend]
└─ Backend: "User Authentication" [pre-planned, backend]
    ├─ JWT generation endpoint [pre-planned, backend, api, authentication, security]
    ├─ Auth middleware [pre-planned, backend, api, security]
    └─ User session storage [pre-planned, backend, database]
```

---

## Features

✅ **Smart Detection** - Automatically identifies frontend/backend needs
✅ **Design-Only** - Stops before implementation, creates pre-planned tickets
✅ **Smart Labels** - Base labels + keyword detection (database, auth, forms, etc.)
✅ **Natural Dependencies** - writing-plans determines blocking relations
✅ **Full-Stack Support** - Separate story groups with cross-dependencies

---

## Installation

See [INSTALL.md](./INSTALL.md)

## Verification

See [VERIFY.md](./VERIFY.md)

---

## Configuration

Create `.claude/plan-to-linear.local.md`:

```yaml
---
linear_team: "Engineering"
linear_project: "Product Development"
default_labels:
  - "pre-planned"
frontend_labels:
  - "frontend"
backend_labels:
  - "backend"
---
```

---

## Dependencies

- `feature-dev` plugin
- `frontend-design` plugin
- `superpowers` plugin (writing-plans skill)
- Linear MCP server

---

## License

MIT
