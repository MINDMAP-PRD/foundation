## Loveable Build Instructions

Read these folders in this order:

1. `FOUNDATION/`
2. `SKILLS/`
3. `BUILD-PLAN/`
4. `MODULE/`

Do not treat `BUILD-PLAN_ARCHIVE_NON_FOUNDATION/` as mandatory base-foundation scope unless explicitly asked to promote something from it.

If the source arrives as a zip or other archive, use the `archive-intake` skill first so the archive is preserved, unpacked into a stable folder, and then routed into the normal foundation intake flow.

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
- owner-configurable-by-default UI and behavior for any meaningful platform element
- use of mature library/framework features where they already solve the problem well
- complete management flows, not isolated screens
- real policy, audit, settings, roles, notifications, and integration handling where the build plan requires them
- rich admin and operator tooling when the build plan defines them

Read `BUILD-PLAN/OWNER_CONTROL_CONTRACT.md` before building any configurable UI, branding, settings, role, capability, or admin surface.

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
- make owner-facing settings authoritative: changing a setting must change runtime behavior, not just update a form

### Build behavior

- Build the shared foundation first.
- Then build only the module described in `MODULE/`.
- Use `BUILD-PLAN/` as the depth and coverage baseline for admin, RBAC, settings, notifications, integrations, users, AI, payments, and related operator flows.
- If the documented scope is incomplete, contradictory, or weak, stop and discuss before building.
- If implementation reveals missing logic or shallow assumptions, stop and discuss instead of silently downgrading the build.
