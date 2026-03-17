# SKILL INTERCONNECTION MAP

How Skills Compose Around the Foundations

> **Purpose:** This file is the shared coordination map for local foundation skills and any session-available specialist skills. It exists to stop AI drift by making skill handoffs explicit.

---

## 1. Core Rule

No skill should act like it is the only skill in the system.

When a task crosses boundaries, the active skill should explicitly pull in the next relevant skill rather than improvising outside its specialty.

No skill should blindly continue when the idea or package is materially incomplete, logically broken, or missing a key decision.

If intake, audit, mapping, or implementation work reveals:

- contradictory requirements
- missing architectural decisions
- weak or incomplete logic
- unresolved ownership conflicts
- gaps that would force major guesswork

the active skill must surface that and discuss it with the user before continuing substantive build work.

## 1.1 Path Portability Rule

Local skills must be portable across different checkout folders and machines.

That means:

- do not hardcode machine-specific absolute repo paths
- prefer relative paths between local skills, local references, and the `foundations/` folder
- write local path guidance so it still works if the repo root is renamed or moved
- only use absolute paths when the environment explicitly requires them for clickable file references in user-facing responses

If a local skill contains a machine-specific path to the repo, that is drift and should be patched.

## 1.2 UI System Default

For foundation-based UI work, the default component and styling baseline is `shadcn/ui`.

That means:

- use `shadcn/ui` primitives first for admin and user surfaces
- use custom components only when `shadcn/ui` has no suitable primitive or composition path
- keep typography, spacing, radius, color tokens, and motion tokens aligned to a shared `shadcn`-compatible design-token layer
- do not invent a parallel custom component system by default

Skills that shape UI, package UI, map UI, or review UI should assume this baseline unless the user explicitly overrides it.

---

## 2. Local Foundation Skills

### `foundation-extension-auditor`

Role:

- decides whether a concept belongs in `GeneralApp`
- decides whether a concept belongs in `FormBuilder`
- decides what stays module-local
- identifies promotion candidates and conflicts

Use first when:

- auditing a new module/spec against the foundations
- checking ownership drift
- validating that a planned module does not redefine shared contracts

Hands off to:

- `foundation-module-factory` when the user wants a full build package, phased plan, or comprehensive module folder

### `foundation-module-factory`

Role:

- turns an idea, module concept, or source spec into a comprehensive build package folder
- creates phased plans, inventories, contracts, specs, and progress tracking
- keeps the package aligned to the foundations using the auditor skill
- establishes `shadcn/ui` as the default UI baseline for module/admin/user surfaces unless the user explicitly chooses otherwise

Use first when:

- the user wants to turn an idea into a module package
- the user wants phased build documentation
- the user wants a folder AI can return to without losing progress
- the user wants to scaffold foundation work while the foundations are still evolving

Must invoke:

- `foundation-extension-auditor` during foundation-fit and ownership review

---

## 3. Project Intake Routing

Before any skill starts substantive project-building work, it must classify the request into one of these entry states:

1. `idea-only`
   - there is no build package yet
   - there may be only a rough concept, prompt, brainstorm, or short spec
   - route to `foundation-module-factory`
2. `spec-backed but no package`
   - there are source docs, notes, schemas, or project files
   - there is no governed package folder yet
   - route to `foundation-module-factory`
3. `existing governed package`
   - a package folder already exists with the foundation-module-factory structure
   - minimum markers:
     - `00_intake/INPUT_STATE.md`
     - `04_build_contract/`
     - `06_phases/`
     - `07_progress/PROGRESS.md`
   - resume from that package before generating new planning artifacts
4. `audit-only`
   - the user wants review, ownership analysis, gap finding, or verification
   - route to the relevant audit skill first
5. `specialist-only implementation`
   - the user wants a narrowly scoped artifact inside an already-defined project
   - use the specialist skill, but still read the package first if one exists

If the user says "build a project" or equivalent, do not guess.

Check:

- is this only an idea?
- is there already a governed package folder?
- if not, create or resume one through `foundation-module-factory`

No skill should start freeform project planning when the package should exist but does not.

No skill should turn an `idea-only` or weak `spec-backed but no package` request into blind implementation if the concept still has major unresolved logic. Package and discussion come before build.

---

## 4. Local Specialist Skills

### `project-mapper`

Role:

- maps an existing implemented or heavily documented project into hierarchy, UI, and build artifacts

Use when:

- the user explicitly wants the project-mapper outputs
- the project already exists in code or rich documentation and the user wants a mapped package of that specific style

Do not use first when:

- the request is just an idea
- the request is to create the governed package for a foundation-based module

Hands off to:

- `foundation-module-factory` when there is no governed package yet
- `foundation-extension-auditor` when ownership against the foundations must be checked first

### `deep-audit`

Role:

- performs exhaustive implementation or documentation audits

Use when:

- the user wants deep review, trace analysis, broken-logic finding, mismatch detection, or technical forensics

Do not use first when:

- the user wants to package or scaffold a new project/module

### `owner-dashboard-mapper`

Role:

- derives owner/admin control surfaces from a real backend/frontend project
- maps dashboard/control surfaces onto `shadcn/ui`-first component choices by default

Use when:

- the project already has enough implementation or detailed source material to map owner controls

Do not use first when:

- the project is still in idea stage
- there is no governed package and the user is still shaping the module

Hands off to:

- `foundation-module-factory` when the project still needs the main package first

### `design-auditor`

Role:

- reviews UI quality, usability, accessibility, and visual design
- validates UI against the package's default `shadcn/ui` baseline unless an approved override is recorded

Use after:

- package architecture/spec work exists
- or the user provides a concrete UI, screenshot, Figma file, or code artifact

### `email-html-mjml`

Role:

- builds or reviews email templates

Use after:

- the module/package identifies an email surface, template, or notification flow

### `secure-coding-practices`

Role:

- injects secure-by-default guidance into backend/auth/API/security-sensitive work

Use after:

- the package identifies security-sensitive build areas
- or the user explicitly asks for a security-focused pass

### `coding-standards`

Role:

- applies general TypeScript, JavaScript, React, and Node implementation standards
- keeps code readable, typed, validated, testable, and structurally consistent

Use after:

- the package reaches implementation planning or active code-writing phases
- or the user explicitly asks for coding conventions, refactoring quality, or maintainability guidance

### `api-design`

Role:

- applies REST API contract design standards
- keeps endpoint naming, methods, status codes, response shapes, pagination, filtering, versioning, and rate-limit expectations consistent

Use after:

- the package reaches backend contract or endpoint design phases
- or the user explicitly asks for API design, endpoint normalization, or API contract review

### `backend-patterns`

Role:

- applies backend architecture patterns for repositories, services, middleware, data access, caching, queues, logging, and operational structure
- keeps backend composition proportional, maintainable, and implementation-ready

Use after:

- the package reaches backend implementation architecture phases
- or the user explicitly asks for backend structure, repository/service layering, caching, jobs, query optimization, or backend operational patterns

### `continuous-agent-loop`

Role:

- designs autonomous and semi-autonomous agent workflows
- chooses between sequential pipelines, persistent loops, PR loops, cleanup passes, and DAG orchestration
- keeps loop state, quality gates, and exit conditions explicit

Use after:

- the governed package exists and the user wants continuous implementation, review, CI-style iteration, or multi-agent execution
- or the user explicitly asks for autonomous loops, agent orchestration, or continuous agent workflows

### `article-writing`

Role:

- writes polished long-form content such as guides, tutorials, launch posts, newsletters, and articles
- keeps voice, structure, and credibility grounded in supplied sources or governed package material

Use after:

- the package or source materials already exist and the user wants polished narrative content
- or the user explicitly asks for long-form writing, voice-matched writing, or article-level rewriting

### `playwright-skill`

Role:

- drives browser automation, E2E planning, UI verification, and test implementation

Use after:

- acceptance flows exist in the governed package
- or the user explicitly asks for browser automation/testing

### `skill-creator`

Role:

- creates or upgrades skills when the package or audit reveals a reusable capability gap

Use when:

- a new repeatable workflow should become a skill
- an imported local skill needs to be updated to fit the foundation ecosystem

---

## 5. Specialist Skill Routing

Use these specialist skills when the build package touches their domain and they are available in the current session.

### Security

- `coding-standards`
  - use for general TS/JS/React/Node implementation quality, naming, structure, typing, validation, and testing conventions
- `api-design`
  - use for REST API contract shape, resource naming, method semantics, response envelopes, pagination, filtering, versioning, and rate-limit conventions
- `backend-patterns`
  - use for backend layering, data-access structure, query optimization, caching, queues, and logging patterns once API contract shape is already understood
- `secure-coding-practices`
  - use for secure-by-default backend/API/auth guidance in build contracts and implementation specs
- `security-best-practices`
  - use when the session-available skill is preferred or local secure-coding coverage is insufficient
- `security-threat-model`
  - use when the module has meaningful trust boundaries, sensitive data, or abuse-path risk

### Document Outputs

- `doc`
  - use for `.docx` deliverables
- `pdf`
  - use for PDF outputs, export flows, or PDF review
- `article-writing`
  - use for long-form narrative content, guides, tutorials, newsletters, launch posts, and article-style deliverables

### UI / Browser / Verification

- `design-auditor`
  - use when UI screens, components, or design checkpoints need quality review
- `playwright-skill`
  - use when the package includes browser flows, E2E verification planning, or test implementation work
- `playwright`
  - use when the session-available browser automation skill is preferred for direct terminal/browser work
- `screenshot`
  - use when desktop capture is needed for UI review or progress evidence

### Product / Mapping Skills

- `project-mapper`
  - use when the user explicitly wants mapped hierarchy/UI/build outputs from an implemented or richly documented project
- `owner-dashboard-mapper`
  - use when owner/admin configuration surfaces must be derived from a real project
- `deep-audit`
  - use for exhaustive technical review after the package/spec/implementation exists
- `email-html-mjml`
  - use when the package includes transactional or marketing email surfaces

### Automation / Orchestration

- `continuous-agent-loop`
  - use when the project needs autonomous implementation cycles, PR/CI loops, staged agent pipelines, or multi-agent orchestration after package intake is established

### Skill Work

- `skill-creator`
  - use when the package reveals a capability gap that should become a new skill

### OpenAI Platform

- `openai-docs`
  - use when the planned module depends on OpenAI APIs or products and official-current docs are required

---

## 6. Mandatory Handoff Patterns

### Idea -> Governed Module Package

1. `foundation-module-factory`
2. `foundation-extension-auditor`
3. specialist skills as needed

### New Spec Audit

1. `foundation-extension-auditor`
2. `foundation-module-factory` only if the user wants a structured package afterward

### Foundation Promotion Work

1. `foundation-extension-auditor`
2. `foundation-module-factory` in foundation mode if the user wants phased shared-contract work
3. specialist skills as needed

### Build A Project

1. classify the entry state from Section 3
2. if no governed package exists, use `foundation-module-factory`
3. if a governed package exists, resume from it
4. once implementation planning or code-writing starts, pull in `coding-standards` for TS/JS/React/Node quality guidance
5. once backend contracts or endpoint design work starts, pull in `api-design`
6. once backend implementation structure, caching, query optimization, or job orchestration work starts, pull in `backend-patterns`
7. if the user wants polished long-form docs, tutorials, launch content, or narrative package outputs, pull in `article-writing`
8. if the user wants autonomous execution, CI-style iteration, or multi-agent orchestration, pull in `continuous-agent-loop` after package intake is established
9. only use `project-mapper` when the user explicitly wants its mapping outputs or when a legacy project must be mapped into that format
10. only use specialist skills directly when the task is narrower than project/package creation

---

## 7. Availability Rule

If a referenced skill is unavailable in the current session:

- do not block the work
- note `Capability gap - skill unavailable in current session`
- continue with the best available local or built-in workflow

---

## 8. Anti-Drift Rule

If a skill is planning, packaging, or auditing any module built on the foundations, it must explicitly anchor itself to:

- `../foundations/FOUNDATION_MODULE_RULES.md`
- `../foundations/FOUNDATION_LIBRARY_ARCHITECTURE.md`
- `../foundations/MODULE_MANIFEST_SPEC.md`
- `../foundations/CAPABILITY_REGISTRY_SPEC.md`
- `../foundations/INTERFACE_REGISTRY_SPEC.md`
- `../foundations/EXTENSION_TABLE_REGISTRY_SPEC.md`
- `../foundations/FOUNDATION_CHANGE_POLICY.md`

The local skills are not allowed to bypass those documents.

---

## 9. Interconnection Rule

Every local skill that can be an entry point must:

- read this map before substantial work
- know whether it is an intake skill, audit skill, mapping skill, or downstream specialist skill
- hand off to `foundation-module-factory` when project/package intake is required
- hand off to `foundation-extension-auditor` when foundation ownership or promotion analysis is required

If a skill cannot satisfy those conditions, it is not fully integrated into the local foundation workflow yet.
