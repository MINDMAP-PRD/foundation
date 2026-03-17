# Output Package Spec

Exact Folder Structure for a Comprehensive Foundation-Governed Build Package

---

## 1. Package Root

Use one package root:

- `<module-slug>-package/`
- `<foundation-change-slug>-foundation-package/`

Treat a folder as an existing governed package only if it contains at least these markers:

- `00_intake/INPUT_STATE.md`
- `04_build_contract/`
- `06_phases/`
- `07_progress/PROGRESS.md`

A package is not automatically trustworthy just because those markers exist.

Before reuse, validate:

- the package shape still matches the current mode and dependency set
- the source files summarized in `00_intake/SOURCE_SUMMARY.md` still match the active source set
- `02_foundation_fit/*` exists and is still relevant
- `07_progress/*` is coherent and not obviously stale
- unresolved blockers and scope changes are still visible

Inside it, create all of the following unless a file is explicitly marked optional.

---

## 2. Package Shapes

Choose one package shape before creating or resuming a package:

- `module-generalapp-only`
  - product/module work built on `GeneralApp` without reusable FormBuilder dependency
- `module-generalapp-plus-formbuilder`
  - product/module work built on `GeneralApp` plus reusable FormBuilder capability
- `foundation-generalapp`
  - shared-contract work for `GeneralApp`
- `foundation-formbuilder`
  - shared-contract work for `FormBuilder`

The package must only require files that belong to the chosen shape.

---

## 3. Baseline Structure

```text
<package-root>/
|-- 00_intake/
|   |-- IDEA.md
|   |-- SOURCE_SUMMARY.md
|   |-- GOALS_AND_SUCCESS.md
|   `-- INPUT_STATE.md
|-- 01_inventory/
|   |-- INVENTORY.md
|   |-- ENTITY_INVENTORY.md
|   |-- ROLE_PERMISSION_INVENTORY.md
|   |-- STATE_BRANCH_FLOW_INVENTORY.md
|   |-- API_ROUTE_INVENTORY.md
|   |-- SETTINGS_CAPABILITIES_INTERFACES.md
|   |-- EVENTS_JOBS_FILES_WEBHOOKS.md
|   `-- SKILL_ROUTING.md
|-- 02_foundation_fit/
|   |-- FOUNDATION_FIT.md
|   |-- OWNERSHIP_MATRIX.md
|   |-- DUPLICATES_AND_CONFLICTS.md
|   |-- PROMOTION_CANDIDATES.md
|   |-- PACKAGE_SHAPE.md
|   `-- MODULE_MANIFEST_DRAFT.md
|-- 03_architecture/
|   |-- CONTEXT.md
|   |-- MODULE_GRAPH.md
|   |-- DATA_ARCHITECTURE.md
|   |-- INTEGRATION_ARCHITECTURE.md
|   |-- SECURITY_ARCHITECTURE.md
|   `-- UI_ARCHITECTURE.md
|-- 04_build_contract/
|   |-- BACKEND_CONTRACT.md
|   |-- FRONTEND_CONTRACT.md
|   |-- DATA_CONTRACT.md
|   |-- INTEGRATIONS_CONTRACT.md
|   |-- SECURITY_CONTRACT.md
|   |-- TESTING_CONTRACT.md
|   |-- DEPLOYMENT_CONTRACT.md
|   `-- OPERATIONS_CONTRACT.md
|-- 05_specs/
|   |-- DOMAIN_SPEC.md
|   |-- ADMIN_SPEC.md
|   |-- USER_SPEC.md
|   |-- API_SPEC.md
|   |-- RBAC_SPEC.md
|   |-- SETTINGS_SPEC.md
|   |-- EVENT_SPEC.md
|   |-- FILES_STORAGE_SPEC.md
|   |-- AUTOMATIONS_AND_JOBS_SPEC.md
|   `-- EDGE_CASES_AND_FAILURES.md
|-- 06_phases/
|   |-- PHASES.md
|   |-- PHASE_CHECKLISTS.md
|   |-- ACCEPTANCE_MATRIX.md
|   |-- DEPENDENCY_ORDER.md
|   `-- MILESTONE_GATES.md
|-- 07_progress/
|   |-- PROGRESS.md
|   |-- DECISIONS.md
|   |-- OPEN_QUESTIONS.md
|   |-- CHANGE_LOG.md
|   `-- NEXT_ACTIONS.md
|-- 08_quality/
|   |-- RISK_REGISTER.md
|   |-- AUDIT_LOG.md
|   |-- DESIGN_REVIEW.md
|   |-- SECURITY_REVIEW.md
|   |-- TEST_COVERAGE_PLAN.md
|   `-- DRIFT_PREVENTION.md
`-- 09_handoff/
    |-- BUILD_ORDER.md
    |-- IMPLEMENTATION_NOTES.md
    |-- RUNBOOK.md
    `-- HANDOFF_SUMMARY.md
```

---

## 4. File Purposes

### `00_intake`

- `IDEA.md`
  - the idea in plain language
  - who it is for
  - why it exists
- `SOURCE_SUMMARY.md`
  - source files read
  - what each source contributes
- `GOALS_AND_SUCCESS.md`
  - business goals
  - operational goals
  - acceptance goals
- `INPUT_STATE.md`
  - raw idea / rough spec / full spec / foundation change
  - known gaps

### `01_inventory`

- `INVENTORY.md`
  - master inventory overview
- `ENTITY_INVENTORY.md`
  - entities, tables, objects, aggregates
- `ROLE_PERMISSION_INVENTORY.md`
  - roles, permissions, restrictions
- `STATE_BRANCH_FLOW_INVENTORY.md`
  - states, transitions, branches, lifecycle flows
- `API_ROUTE_INVENTORY.md`
  - endpoints, handlers, webhooks, route contracts
- `SETTINGS_CAPABILITIES_INTERFACES.md`
  - settings keys, capabilities, interface usage, extension points
- `EVENTS_JOBS_FILES_WEBHOOKS.md`
  - events, scheduled jobs, files, notifications, webhooks
- `SKILL_ROUTING.md`
  - which skills apply to which module areas

### `02_foundation_fit`

- `FOUNDATION_FIT.md`
  - summary of how the package fits `GeneralApp` and optionally `FormBuilder`
- `OWNERSHIP_MATRIX.md`
  - what belongs to foundations vs module
- `DUPLICATES_AND_CONFLICTS.md`
  - auditor findings that must be resolved
- `PROMOTION_CANDIDATES.md`
  - reusable features that may need foundation promotion
- `PACKAGE_SHAPE.md`
  - chosen package shape
  - why that shape applies
  - which files are required vs optional for this package
- `MODULE_MANIFEST_DRAFT.md`
  - manifest draft grounded in foundation rules
  - optional for foundation-mode packages unless a manifest is part of the proposed change

### `03_architecture`

- `CONTEXT.md`
  - system context and boundaries
- `MODULE_GRAPH.md`
  - module-to-module and component-to-component dependency graph
- `DATA_ARCHITECTURE.md`
  - storage model and boundaries
- `INTEGRATION_ARCHITECTURE.md`
  - external services, adapters, bridge surfaces
- `SECURITY_ARCHITECTURE.md`
  - trust boundaries, sensitive flows, authz assumptions
- `UI_ARCHITECTURE.md`
  - admin vs user surfaces, navigation, core UI systems
  - default UI baseline is `shadcn/ui` for any meaningful UI surface unless the package explicitly records a justified override
  - record the shared token model for typography, spacing, color, radius, and motion
  - optional when the package has no meaningful UI surface

### `04_build_contract`

Each contract file should say what must be built, what is out of scope, what must not drift, and what specialist skills should be invoked.

Shape rules:

- `FRONTEND_CONTRACT.md`
  - optional for backend-only modules and some foundation packages
  - when present, it must state `shadcn/ui`-first component usage and define the rule for when custom components are allowed
- `DEPLOYMENT_CONTRACT.md`
  - optional when deployment is explicitly out of scope for the package
- `OPERATIONS_CONTRACT.md`
  - optional when the package is purely conceptual and operations ownership is not yet defined

### `05_specs`

These are the implementation-facing specs.

They should be specific enough that a future AI can build from them without re-inventing missing contract details.

Shape rules:

- `ADMIN_SPEC.md`
  - optional when there is no admin/owner surface
  - when present, it should map screens and controls to `shadcn/ui` primitives first
- `USER_SPEC.md`
  - optional when there is no user-facing surface
  - when present, it should map screens and controls to `shadcn/ui` primitives first
- `API_SPEC.md`
  - optional when no API surface exists
- `RBAC_SPEC.md`
  - optional when RBAC is truly out of scope
- `SETTINGS_SPEC.md`
  - optional when no settings/config surface exists
- `EVENT_SPEC.md`
  - optional when events/audit hooks are out of scope
- `FILES_STORAGE_SPEC.md`
  - optional when no file/storage behavior exists
- `AUTOMATIONS_AND_JOBS_SPEC.md`
  - optional when there are no automations/jobs

### `06_phases`

These files define build order, dependencies, gates, and acceptance.

### `07_progress`

These files are the continuity layer.

They must be updated whenever work advances.

### `08_quality`

These files track risks, reviews, anti-drift measures, and testing expectations.

### `09_handoff`

These files explain how to continue implementation safely.

---

## 5. Re-Baseline Rule

If the source idea/spec changes after a package already exists, do not continue silently.

Record a re-baseline in:

- `00_intake/SOURCE_SUMMARY.md`
- `02_foundation_fit/PACKAGE_SHAPE.md`
- `07_progress/CHANGE_LOG.md`
- `07_progress/NEXT_ACTIONS.md`

The package must explicitly say:

- what changed in the source set
- whether the package shape changed
- for UI-bearing packages, whether the UI-system baseline changed
- which sections became stale
- whether implementation planning can continue safely

---

## 6. Content Rule

Every file should contain:

- traced facts from sources or foundations
- explicit `TBD - no source found` markers where detail is missing
- no silent invention of shared contracts

---

## 7. Anti-Drift Rule

At minimum, the package must preserve:

- ownership boundaries
- manifest shape
- phase status
- unresolved blockers
- specialist skill routing
- foundation fit decisions
- for UI-bearing packages, continuity of the chosen `shadcn/ui`-first design-system baseline

If any of those are missing, the package is incomplete.

The package is also incomplete if:

- it uses the wrong package shape
- it requires files that do not fit the current mode or dependency set
- it resumes from a stale source baseline without recording that drift
