## Loveable Build Instructions

Read these folders in this order:

1. `FOUNDATION/`
2. `SKILLS/`
3. `BUILD-PLAN/`
4. `MODULE/`

### What `BUILD-PLAN/` means

`BUILD-PLAN/` defines what "comprehensive" means for implementation depth.

Do not interpret comprehensive as:

- basic CRUD only
- a thin admin page with a few toggles
- partial library usage
- placeholder support for advanced features
- simplified permission models when the build plan defines richer capability coverage

Interpret comprehensive as:

- full capability coverage for the documented surface
- use of mature library/framework features where they already solve the problem well
- complete management flows, not isolated screens
- real policy, audit, settings, roles, notifications, and integration handling where the build plan requires them
- rich admin and operator tooling when the build plan defines them

### Library usage rule

When the foundation or module uses a proven library/framework, use the library deeply enough to cover the documented requirements.

Do not:

- install a strong library and then implement only its most basic subset
- replace advanced built-in capabilities with shallow custom shortcuts
- ignore available library features that directly satisfy documented requirements

Do:

- use the full practical feature set of the chosen library/framework where the docs call for it
- keep vendors/frameworks behind adapters or capability boundaries when they are platform-level concerns
- keep module-local framework choices inside module boundaries unless explicitly promoted to the foundation

### Build behavior

- Build the shared foundation first.
- Then build only the module described in `MODULE/`.
- Use `BUILD-PLAN/` as the depth and coverage baseline for admin, RBAC, settings, notifications, integrations, users, AI, payments, and related operator flows.
- If the documented scope is incomplete, contradictory, or weak, stop and discuss before building.
- If implementation reveals missing logic or shallow assumptions, stop and discuss instead of silently downgrading the build.
