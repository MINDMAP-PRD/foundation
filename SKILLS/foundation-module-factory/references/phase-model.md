# Phase Model

Build Phases for Foundation Module Factory

---

## Phase 0 - Intake and Mode Selection

Decide:

- module mode vs foundation mode
- `GeneralApp` only vs `GeneralApp + FormBuilder`
- idea-only vs spec-backed
- create new package vs resume existing package
- choose and record the package shape before writing downstream files
- validate whether an existing package is still safe to reuse

Outputs:

- `00_intake/*`

---

## Phase 1 - Inventory Extraction

Extract every concrete element from the sources:

- entities
- roles
- permissions
- states
- branches
- flows
- routes
- settings
- capabilities
- interfaces
- events
- jobs
- files
- integrations

Outputs:

- `01_inventory/*`

---

## Phase 2 - Foundation Fit

Run foundation ownership analysis and package the results.

This phase must incorporate `foundation-extension-auditor`.

Outputs:

- `02_foundation_fit/*`

---

## Phase 3 - Architecture Synthesis

Turn the raw inventory into a system design package.

Outputs:

- `03_architecture/*`

---

## Phase 4 - Build Contract

Write the actual implementation contract across backend, frontend, data, integrations, security, testing, deployment, and operations.

Outputs:

- `04_build_contract/*`

---

## Phase 5 - Detailed Specs

Write the implementation-facing specs that future AI can build from directly.

Outputs:

- `05_specs/*`

---

## Phase 6 - Build Phasing

Turn the specs into concrete implementation phases with dependency order and acceptance gates.

Outputs:

- `06_phases/*`

---

## Phase 7 - Progress and Continuity

Initialize and maintain the continuity files:

- progress
- decisions
- questions
- changes
- next actions
- re-baseline when the source set or package shape changes

Outputs:

- `07_progress/*`

---

## Phase 8 - Quality and Anti-Drift

Write the risk, review, audit, and anti-drift files.

Outputs:

- `08_quality/*`

---

## Phase 9 - Handoff

Write the documents that let another AI or engineer resume work cleanly.

Outputs:

- `09_handoff/*`

---

## Completion Rule

Do not consider the package complete until:

- all required folders exist
- the chosen package shape is recorded and the required/optional files match that shape
- progress reflects current status
- foundation-fit decisions are recorded
- specialist skill routing is recorded
- unresolved gaps are explicitly visible
- any reused package has been validated against the current source set
- any source drift since package creation has been recorded through a re-baseline
