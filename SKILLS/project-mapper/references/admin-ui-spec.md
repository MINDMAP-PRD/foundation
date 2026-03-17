# Admin UI Page Specifications

All pages are part of a **Vite + React + TypeScript + shadcn/ui project**. See
`component-patterns.md` for the full scaffold, component library, and data types.
All page content derives from `data.ts` (which is populated from FILE_MAP.md and
HIERARCHY.md) — no invented data.

Use shadcn/ui primitives by default. Go custom only when shadcn has no matching
primitive (dependency graph SVG lines, state machine node layout). See section 15
of `component-patterns.md` for the full custom-only rule.

---

## Required Pages

### Sidebar Navigation
One nav item per top-level module from FILE_MAP.md. No invented items.

### Role Selector (demo mode)
Lets the viewer switch between the exact roles documented in the source files to
see what each role can access. Lives in the top bar.

### RoleGate Component
Use the `RoleGate` component from `component-patterns.md` (section 7) — that is
the canonical definition. It must work from source-defined role names rather
than a fixed owner/super-admin/admin/user ladder. Do not redefine it here.

### Dashboard Page
Overview cards: total modules, permissions registered, governance config keys,
dependencies.

Also surface flagged gaps from HIERARCHY.md as a separate warning panel when any
permission or role decision is still TBD.

### Feature Map Page
Visual interconnection map. Render each module as a card in a CSS grid. Draw
dependency arrows using SVG `<line>` elements with absolute positioning. Color
by the source-defined minimum role using a deterministic role-color mapping.
Clicking a module highlights its dependencies and dependents.

### State Flow Page
For every module from FILE_MAP.md, render its state machine:
- States as rounded nodes. Color: initial=blue, active=green, terminal=gray, error=red.
- Transitions as labeled arrows between states.
- Branch points as diamond nodes with condition labels on each exit arrow.
- Module selector dropdown to switch between modules.
- Clicking a state node shows entry condition, exit condition, and description.
- Clicking a transition arrow shows the trigger and any side effects.

No state may be orphaned (unreachable from initial state). No transition may
reference a state not defined in FILE_MAP.md.

### Per-Module Pages (one per module in FILE_MAP.md)
Each module page shows:
- Module name + one-line description
- Skill tags (badges showing which skills apply)
- Dependency badges: `Depends on: X, Y` / `Used by: A, B`
- Acceptance condition banner
- **States tab** — table of states with entry/exit conditions
- **Branches tab** — table of decision points and outcomes
- **Flow tab** — numbered steps from trigger to side effects
- Feature table: all configurable options, their min role, their status, and acceptance condition
- Role-gated action buttons (derived from documented module actions and gated by HIERARCHY.md permissions)

### Governance Config Page (conditional; top-authority-only when documented, RoleGate enforced)
An interactive configuration form. Must include:
- Role delegation toggles only when the docs define delegatable permissions
- Settings surfaces found in the docs (not invented)
- "Save Configuration" button — writes to `localStorage` under a governance config key
- "Export Config" button — downloads current config as a JSON file
- On load: reads from `localStorage` to restore previously saved config

Include this page only when the source docs define a governance/delegation
surface that belongs in the admin UI. If no such surface is documented, omit
the page entirely.

Note: `localStorage` is intentional — this file runs in the user's own browser,
not inside a Claude artifact. This is not subject to Claude.ai artifact restrictions.

### Hierarchy Page
Full permission registry table from HIERARCHY.md. Show assigned roles exactly
as documented. Flagged gaps shown in amber.

---

## Interconnection Enforcement Rules

For every dependency edge in FILE_MAP.md:
- Dependent module page must show: `Requires: <dependency>`
- Dependency page must show: `Used by: <dependent>`
- Feature Map must draw a visible edge between them

These are mandatory. If FILE_MAP.md says Module A depends on Module B, both
pages and the Feature Map must reflect it.
