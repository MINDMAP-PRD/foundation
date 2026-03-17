---
name: project-mapper
description: >
  Reads all project files (docs, specs, code, configs) and produces a fully-mapped,
  interconnected project structure with a role hierarchy, admin UI, user UI, and
  build contracts. Use this skill whenever the user mentions mapping a project,
  generating a project hierarchy, creating build contracts from specs, generating
  an admin dashboard from documentation, mapping file dependencies, or any request
  to map out, interconnect, or structure a project's files and roles. Also trigger
  when a project folder or set of source docs is provided alongside a request for
  hierarchy, UI, or build artifact outputs. Do not trigger for simple summaries,
  basic docs, or README-style writing that does not require hierarchy, UI, or
  build contracts. For foundation-based project intake, do not use this skill
  first when the request is still idea-stage or when no governed package exists;
  route to foundation-module-factory first.
---

# Project Mapper Skill — v3

## Read First

Before starting, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. `../../foundations/README.md` when the project builds on the foundations
3. `../../foundations/FOUNDATION_MODULE_RULES.md` when the project builds on the foundations
4. `../../foundations/FOUNDATION_LIBRARY_ARCHITECTURE.md` when the project builds on the foundations
5. `../../foundations/FOUNDATION_IMPLEMENTATION_BLUEPRINT.md` when the project builds on the foundations
6. `../../foundations/GeneralApp_Build_Documentation.md` when the project builds on the foundations
7. `../../foundations/FormBuilder_Build_Documentation_v6.md` only if the project uses reusable form-engine capability

If a governed package already exists, validate it first so mapping work follows the active contracts, phases, progress state, source freshness, and package continuity.

## Foundation-Aware Intake Rule

This skill is not the default entry point for "build a project" in the foundation ecosystem.

Before using it, check:

- is the request only an idea or rough spec?
- is there already a governed package folder with intake, build-contract, phases, and progress files?
- if yes, is it still valid for reuse under the current source set and task?

If the answer is:

- `idea only` or `no governed package yet` -> use `foundation-module-factory` first
- `governed package exists, it validates cleanly, and the user wants package-driven continuation` -> resume from that package first
- `governed package exists but is stale, incomplete, or inconsistent with the current source set` -> re-baseline through `foundation-module-factory` first
- `implemented or richly documented project and user explicitly wants project-mapper outputs` -> this skill is appropriate

## Changelog

- `v3` adds state/branch/flow mapping, two runnable Vite UI outputs, and build contracts.
- `v3.1` fixes internal cross-references, missing scaffold files, fast-mode timing, UI checkpoint guidance, and unavailable-skill fallbacks.

Produces four interconnected output folders from any project's files:

```
<project-name>-mapped/
├── hierarchy-config/     # FILE_MAP.md + HIERARCHY.md (authority-configurable)
├── admin-ui/             # Single working React app — admin dashboard
├── user-ui/              # Single working React app — client-facing portal
└── build-contract/       # BACKEND.md + FRONTEND.md + INTEGRATIONS.md + PHASES.md
```

**Critical operating rules (read before starting):**
1. STOP and show output after each phase checkpoint. Do not proceed until the user confirms.
2. Before generating any phase output, re-read all previously created files from disk — never work from memory alone.
3. All four output folders must be internally consistent. Run the consistency check in Phase 7 before packaging.
4. Every permission string, module name, and acceptance condition must trace to a source file. If it cannot be traced, mark it `⚠️ TBD — no source found`.
5. UIs are Vite + React + TypeScript + shadcn/ui projects. Each UI is a separate project folder the user runs with `npm install && npm run dev`. Use custom code only when shadcn has no matching primitive — see `references/component-patterns.md` section 15.
6. Every module card must include States, Branches, Features, Actions, and Flow — not just dependencies. This is mandatory for anti-drift.
7. The Skill Registry must be consulted during Phase 1 and applied throughout. Every module that touches a skill-covered domain must be flagged.
8. For foundation-based projects, do not start if the governed package should exist but does not. Route to `foundation-module-factory` first.

---

## SKILL REGISTRY

Before starting any phase, internalize this registry. When a module, integration, or output matches a domain below, apply that skill's approach automatically — do not wait to be asked.

| Skill | Domain | When to invoke |
|-------|--------|---------------|
| `design-auditor` | UI generation and review | All admin-ui and user-ui phases. Apply this skill to review generated UI against the governed package and `shadcn/ui` baseline before each UI checkpoint when it is available. |
| `email-html-mjml` | Email & notifications | Any module with email flows, transactional mail, invite emails, notification systems. Reference MJML approach for email templates in build contract. |
| `doc` | Word documents | When generating deliverables as .docx — reports, specs, memos requested as Word files. |
| `pdf` | PDF output | Any module that generates, exports, or fills PDFs. Reference pdf skill approach in INTEGRATIONS.md. |
| `coding-standards` | Implementation quality | Any TS/JS/React/Node module whose outputs should carry coding, naming, typing, validation, and testing conventions into build contracts or implementation notes. |
| `api-design` | API contracts | Any module with REST endpoints, pagination, filtering, versioning, status-code decisions, or public/internal API contract surfaces. |
| `backend-patterns` | Backend architecture | Any module with repository/service/controller patterns, middleware composition, query optimization, caching, queueing, or logging/operational backend structure. |
| `continuous-agent-loop` | Automation / orchestration | Any module or project that needs autonomous implementation phases, CI-style iteration, PR loops, or multi-agent execution planning in its build outputs. |
| `secure-coding-practices` | Security | Any auth module, backend scaffolding, or endpoint with permission checks. Annotate BACKEND.md with security notes from this skill. |
| `security-best-practices` | Security | Use when the session-available security specialist is preferred or when broader secure-by-default coverage is needed. |
| `security-threat-model` | Threat modeling | Any module with meaningful trust boundaries, sensitive data, or abuse-path risk. |
| `playwright-skill` | E2E testing | Acceptance conditions in PHASES.md should reference Playwright test patterns. Any module with testable UI flows. |
| `playwright` | Browser automation | Use when the session-available Playwright skill is preferred for direct browser execution. |
| `foundation-extension-auditor` | Foundation ownership | Any module or spec that claims to build on `GeneralApp` or `FormBuilder`; use before accepting shared ownership assumptions. |
| `foundation-module-factory` | Governed package intake | Any foundation-based project that is still idea-stage or lacks a governed package folder. |
| `skill-creator` | Capability gap | If mapping reveals a module type with no matching skill — flag it. The user may want a new skill built for it. |

**Availability rule:** If a listed skill is not available in the current session, do not block the mapping. Note it as `Capability gap — skill unavailable in current session` in working notes and in the Phase 8 handoff summary.
Use this exact handoff format: `⚠️ SKILL GAP: [skill-name] — required by [module-list] but not available. Source manually or build with skill-creator.`

**How to apply the registry:**
- During Phase 1 extraction, tag each module with `Skills: [skill-name, ...]` in your working notes
- In FILE_MAP.md module cards, include a `Skills:` field listing which skills apply
- In BACKEND.md and INTEGRATIONS.md, add skill invocation notes where relevant
- In PHASES.md, reference skill names in acceptance conditions and gate checklists
- In Phase 8 handoff summary, list which skills are needed to fully execute the build

### Minimum UI audit checklist

Use this checklist whenever a UI review step is required and `design-auditor` is unavailable:

- Contrast is readable for body text, labels, muted text, and badges
- Layout works at desktop and mobile widths without clipping or horizontal overflow
- Interactive controls have visible hover, focus, disabled, loading, and error states
- Touch targets are comfortably clickable and form labels are present
- Tables, graphs, and state-machine views remain legible when data grows
- Empty, loading, and error states are visible wherever data can be missing or delayed
- User UI exposes no admin-only controls, permission strings, or dependency internals

---

## PHASE 1 — READ & EXTRACT

### 1.1 Ingest every file

Read every file the user has provided. Accept any type:
- `.md / .txt / .pdf / .docx` — primary source of truth
- `.ts / .js / .py / .go` — extract entity names, routes, schemas
- `.json / .yaml / .env.example` — extract settings surfaces, flags, capabilities
- Diagrams or architecture notes — extract system boundaries

For each file, extract into a working note (not yet written to disk):

| Field | What to extract |
|-------|----------------|
| **File/Module name** | What is it called in the docs? |
| **What it is** | One sentence: what does this module do? |
| **Depends on** | Other modules it calls, imports, or requires to exist first |
| **What depends on it** | Other modules that call or import this one |
| **States** | All possible states this module can be in (e.g. `draft \| active \| archived`) |
| **Features** | Feature lists, configurable options, or capability tables for this module; include min role, status, and acceptance when documented |
| **Branches** | Every decision fork (e.g. `authenticated → dashboard \| unauthenticated → login`) |
| **Flow** | Full lifecycle: trigger → processing → output → side effects |
| **Acceptance condition** | The gate/rule that must be true for this to be complete |
| **Permissions found** | Every RBAC/permission string mentioned in this file |
| **Role restrictions** | Any "only X can..." or "restricted to X" statements |
| **Min role** | Lowest role explicitly allowed to access or operate this module; if unclear, record `⚠️ TBD — min role not specified in source` |
| **API endpoints** | Any routes or endpoint shapes clearly described by name in the source docs. If a route is required but not specified, record `⚠️ TBD — endpoint not specified in source`. |
| **Actions** | User or admin actions the UI exposes for this module; include label, min role, and destructive/secondary intent when documented |
| **Skills** | Which skills from the Skill Registry apply to this module |

### 1.2 Build a project model

After reading all files, answer these in your working notes before writing anything:

1. **What is this project?** — 2–3 sentence summary.
2. **What are the top-level modules?** — List them by name.
3. **What roles exist?** — List every role name found, with the exact source quote.
4. **What is the complete permission registry?** — Every permission string found, which file it came from, which role the docs assign it to.
5. **What are the integration points?** — Where do modules call each other?
6. **What is the data lifecycle?** — How does data flow from user → system → admin?
7. **What build phases exist?** — List them if the docs define any.
8. **Which skills apply?** — Map every module to its relevant skills from the Skill Registry.
9. **What are the state machines?** — For each module: list all states, all branches, and the complete flow.

**Fast mode timing:** Only offer fast mode after completing Phase 1.2, because only then is the module count known.

### 1.3 Handling sparse or contradictory sources

**If the source docs are sparse** (fewer than 3 modules can be extracted, or fewer than 2 roles are named):
- Do not attempt to invent structure. Surface what you found at Checkpoint 1.
- Ask: "The docs don't have enough detail for a full mapping. Would you like to add more documentation before I proceed, or should I mark all gaps as TBD and continue?"
- If the user says continue, flag every TBD item prominently throughout all outputs.

**If two source files contradict each other** (e.g. different permission lists for the same role, conflicting module names):
- Note the conflict in your working notes.
- At Checkpoint 1, surface both versions: "File A says X, File B says Y — which is correct?"
- Do not resolve contradictions yourself. Wait for the user to confirm.

### 1.4 ⛔ CHECKPOINT 1 — Show the project model

Present the project model to the user as a clean summary including:
- Module list with skill tags
- Role inventory
- State machine summary (one line per module: states and key branches)

If the project has fewer than 10 modules, you may offer fast mode here:
> "This looks like a small project — I can combine Checkpoints 1 and 2 into one review step if you'd like to move faster."

If the user accepts fast mode:
- prepare the Project Model summary and a draft FILE_MAP preview in the same response
- do not write `hierarchy-config/FILE_MAP.md` yet
- use one combined confirmation before saving either output to disk
- ask this combined review question:
  > "Here's the project model and full file map together. Does everything look right — modules, roles, min roles, features, actions, states, flows, and dependencies? Confirm to proceed to the hierarchy."

Ask:
> "Here's what I found. Does this match your project? Anything missing or wrong in the modules, roles, min roles, features, actions, states, flows, or dependencies before I generate the file map and hierarchy?"

**Do not write any files until the user confirms.**

---

## PHASE 2 — FILE MAP

Re-read all source files from disk before starting this phase.

### 2.1 Write FILE_MAP.md

Write FILE_MAP.md by domain section. Add each domain header once, then place every matching module card beneath it. Use this **exact expanded pattern**:

```markdown
## Backend

### <Module Name>

- **What it is:** <one sentence>
- **Skills:** <comma-separated skill names from the Skill Registry, or "None">
- **Depends on:** <comma-separated module names, or "None">
- **What depends on it:** <comma-separated module names, or "None">
- **Min role:** <lowest documented role allowed to access this module, or "⚠️ TBD — min role not specified in source">
- **Acceptance condition:** <verbatim from docs, or "⚠️ TBD — no source found">
- **Source:** <which file this was extracted from>

#### States
| State | Description | Entry condition | Exit condition |
|-------|-------------|----------------|----------------|
| <state-name> | <what this state means> | <what triggers entry> | <what triggers exit> |

#### Branches
| Decision point | Condition | Path A | Path B |
|----------------|-----------|--------|--------|
| <trigger or event> | <condition being evaluated> | <outcome if true> | <outcome if false> |

#### Features
| Feature ID | Feature | Min role | Status | Acceptance condition |
|------------|---------|----------|--------|----------------------|
| <explicit feature ID from the docs, or deterministic kebab-case slug of the feature name> | <feature name from the docs> | <lowest documented role allowed to use it, or "TBD - min role not specified in source"> | <complete | in_progress | pending | blocked | "TBD - status not specified in source"> | <verbatim acceptance condition, or "TBD - no source found"> |

#### Actions
| Action | Min role | Variant | Source / trigger |
|--------|----------|---------|------------------|
| <button or UI action label from the docs> | <lowest documented role allowed to use it, or "⚠️ TBD — min role not specified in source"> | <default | destructive | outline | secondary | ghost | "⚠️ TBD — variant not specified in source"> | <what causes this action or where it appears in the docs> |

#### Flow
1. **Trigger:** <what initiates this module>
2. **Input:** <what data/event this module receives>
3. **Processing:** <what it does, step by step>
4. **Output:** <what it produces or returns>
5. **Side effects:** <what other modules are notified or changed>
6. **Error paths:** <what happens when things go wrong>
```

Group cards under domain headers written once per section: `## Backend`, `## Frontend`, `## Integrations`, `## Background Jobs`, etc. Do not repeat the domain header for every module.

Populate `Min role` from the Phase 1 `Role restrictions` / `Min role` extraction only. Do not infer it from HIERARCHY.md later. If the source docs do not make the minimum role explicit, mark it `⚠️ TBD — min role not specified in source`.

Populate `Features` from feature lists, configurable options, or capability tables in the source docs. If a module has no documented feature breakdown, add a single `⚠️ TBD` feature row rather than inventing one.

The `Feature ID` column is required because `data.ts` uses stable feature IDs. If the docs do not provide explicit IDs, generate deterministic kebab-case slugs from feature names in FILE_MAP.md and reuse those exact IDs in `data.ts`.

Populate `Actions` from explicitly documented UI actions only. If a module page needs role-gated actions but the source docs do not name them, add a single `⚠️ TBD` action row rather than inventing actions from the permissions list.

At the top of FILE_MAP.md, write a dependency matrix:

```markdown
## Dependency Matrix

| Module | Depends On | Depended On By | Min Role | States | Skills |
|--------|------------|----------------|----------|--------|--------|
```

At the bottom of FILE_MAP.md, write a state inventory:

```markdown
## State Inventory

| Module | States | Initial State | Terminal States |
|--------|--------|--------------|----------------|
```

Save to: `hierarchy-config/FILE_MAP.md`

### 2.2 ⛔ CHECKPOINT 2 — Show the file map

If the user already confirmed the combined fast-mode review in Checkpoint 1, skip this checkpoint and proceed directly to Phase 3.

Show FILE_MAP.md to the user. Ask:
> "Here's the full file map including states, branches, features, actions, and flows for every module. Are the module names right? Are the min roles, feature tables, action rows, states, and flows correct? Any dependencies wrong or missing? This is what the admin UI and build contracts will be built from."

**Do not proceed until confirmed.** If fast mode was accepted at Checkpoint 1, treat Checkpoints 1 and 2 as one combined checkpoint and proceed to Phase 3 only after that combined confirmation.

---

## PHASE 3 — HIERARCHY CONFIG

Re-read `hierarchy-config/FILE_MAP.md` from disk before starting.
Also re-read all original source files to cross-check permissions.
Read `references/hierarchy-rules.md` for role hierarchy formatting, role name mapping rules, and permission classification logic — apply it throughout this phase without inventing a canonical role tree that the source does not define.

### 3.1 Permission verification loop

Before writing HIERARCHY.md, run this check for every permission string you found:

1. Is the permission string quoted verbatim from a source file? → ✅ include
2. Is the role assignment documented explicitly? → ✅ include
3. Is the permission required by the UI/spec but not stated verbatim? → ⚠️ flag it as `TBD` and do not assign a role
4. Did you derive the permission yourself? → ❌ do not include — mark as `TBD`

**You may not invent a single permission string or role assignment.**

### 3.2 Write HIERARCHY.md

Use this exact structure:

```markdown
# Project Hierarchy Configuration
**Project:** <name>
**Version:** <from docs>
**Generated from:** <list of source files read>
**Date:** <today>

---

## Role Architecture

Describe the hierarchy exactly as documented in the source files.

Do not assume:

- `Owner`
- `Super Admin`
- `Admin`
- a three-tier model
- undocumented bypass authority

Any top role, delegated role, override rule, or bootstrap authority must trace to a source file or be flagged as `TBD`.

### Hierarchy
<render the documented hierarchy exactly as found in the source files>
<if the full tree is not documented, render only the documented parts and flag the missing structure as `TBD`>

---

## <Top Role From Source>

### Exclusive permissions (only if documented)
| Permission | Source | What it controls |
|------------|--------|-----------------|

### Delegatable permissions (only if documented)
| Permission | Source |
|------------|--------|

### Configuration surface
List only what the source docs explicitly say this top role controls.

---

## <Delegated Role From Source>

### Permissions
| Permission | Source | Delegatable to lower roles? |
|------------|--------|----------------------|

### Cannot do without higher role
| Blocked action | Reason / source |
|----------------|----------------|

---

## <Additional Roles From Source>

### Permissions
| Permission | Source |
|------------|--------|

### Cannot do without higher role
| Blocked action | Reason / source |
|----------------|----------------|

---

## Permission Registry (complete)

| Permission String | Assigned Roles | Source | Notes |
|-------------------|----------------|--------|-------|

---

## ⚠️ Gaps — Require Authority Decision

| Gap | Description | Decision required from documented top authority |
|-----|-------------|------------------------------------------|
```

### 3.3 ⛔ CHECKPOINT 3 — Show hierarchy

Show HIERARCHY.md to the user. Ask:
> "Here's the full role hierarchy and permission registry. Every entry is traced to a source file. Any gaps are flagged at the bottom. Does this match how you want the ownership to work? Once you confirm, I'll build the admin UI from this."

**Do not proceed until confirmed.**

---

## PHASE 4 — ADMIN UI

Re-read `hierarchy-config/FILE_MAP.md` AND `hierarchy-config/HIERARCHY.md` from disk before writing a single line of UI code.
Read `references/component-patterns.md` for the full Vite + shadcn/ui scaffold, component patterns, and TypeScript types — follow it exactly for all UI generated in this phase.

Apply the `design-auditor` skill criteria and the governed package's `shadcn/ui` baseline for all UI generation in this phase when that skill is available.
If it is unavailable, follow `references/component-patterns.md` exactly and use the Minimum UI audit checklist as the baseline quality bar.
If `design-auditor` is available, run it before the checkpoint. Otherwise apply the inline Minimum UI audit checklist above.

**UI checkpoint write rule:** Phases 4 and 5 write files to disk before review because the output is a runnable Vite project, not inline prose. The review checkpoint still happens before any later phase begins.

### 4.1 Architecture rule

**The admin UI is a Vite + React + TypeScript + shadcn/ui project** at `admin-ui/`. The user runs `npm install && npm run dev` to open it. Read `references/component-patterns.md` sections 1–2 for the full scaffold — generate every file in the tree (`index.html`, package.json, tsconfig files, vite.config.ts, tailwind.config.ts, postcss.config.js, components.json, globals.css, src/lib/utils.ts, src/types.ts, src/data.ts, src/main.tsx, src/App.tsx, all component files).

### 4.2 What to build

Read `references/admin-ui-spec.md` for the complete page-by-page spec. Build every required page defined there, and include conditional pages only when FILE_MAP.md or HIERARCHY.md explicitly justify them. No invented pages, no invented data.

### 4.3 Interconnection enforcement

For every dependency edge in FILE_MAP.md:
- The dependent module's page must show a badge: `Requires: <dependency>`
- The dependency's page must show: `Used by: <dependent>`
- The Feature Map must draw a visible edge between them

For every state in FILE_MAP.md the State Flow page must render every node and transition. No orphaned states. No transitions referencing states not in FILE_MAP.md. See `references/admin-ui-spec.md` for details.

### 4.4 ⛔ CHECKPOINT 4 — Show the admin UI

Write all files for `admin-ui/`. Run a UI audit pass before presenting. Then ask:
> "The admin UI is ready — run `cd admin-ui && npm install && npm run dev` to open it. Check: does the Feature Map show the right connections? Does the State Flow page show correct states for each module? If Governance Config is documented for this project, does that page have the right settings? Does the role switcher in the header correctly gate sections? Any module missing?"

**Do not build the user UI until confirmed.**

---

## PHASE 5 — USER UI

Re-read `hierarchy-config/FILE_MAP.md`, `hierarchy-config/HIERARCHY.md`, `admin-ui/src/App.tsx`, `admin-ui/src/data.ts`, and the files under `admin-ui/src/components/` from disk before starting.
Read `references/component-patterns.md` if you have not already — same Vite + shadcn/ui stack applies here.

Apply the `design-auditor` skill criteria and the governed package's `shadcn/ui` baseline for all UI generation in this phase when that skill is available.
If it is unavailable, follow `references/component-patterns.md` exactly and use the Minimum UI audit checklist as the baseline quality bar.
If `design-auditor` is available, run it before the checkpoint. Otherwise apply the inline Minimum UI audit checklist above.

### 5.1 Architecture rule

Same stack as admin UI: **Vite + React + TypeScript + shadcn/ui project** at `user-ui/`. Scaffold all the same config files. See `references/component-patterns.md` section 14 for what differs in the user UI.

### 5.2 What to build

Only build what the docs describe as user-facing or client-accessible. Do not expose any admin feature.

Extract from FILE_MAP.md: which modules have user-facing interfaces? Build a page for each one.

Every user UI page must:
- Show only fields and actions the docs describe as visible to non-admin users
- Show module state where it is user-visible (e.g. order status, request status) — sourced from FILE_MAP.md States
- Use only API endpoints explicitly documented in the source files - these will be formally recorded in the build contract during Phase 6
- Use the same field types and validation rules described in the spec
- Show no permission strings, no module dependency info, no admin controls

**User portal home** — the user's landing page after login. Show only their data.

**Settings page** — user-level settings only (from docs). No admin settings exposed.

### 5.3 ⛔ CHECKPOINT 5 — Show the user UI

Write all files for `user-ui/`. Ask:
> "The user-facing portal is ready — run `cd user-ui && npm install && npm run dev` to open it. Does it show the right features for your clients? Are the states displayed correctly for user-facing modules? Anything that shouldn't be visible? Anything missing?"

**Do not build contracts until confirmed.**

---

## PHASE 6 — BUILD CONTRACT

Re-read ALL previously created files from disk before starting:
- `hierarchy-config/FILE_MAP.md`
- `hierarchy-config/HIERARCHY.md`
- `admin-ui/src/App.tsx`
- `admin-ui/src/data.ts`
- `admin-ui/src/components/`
- `user-ui/src/App.tsx`
- `user-ui/src/data.ts`
- `user-ui/src/components/`

Read `references/contract-format.md` — it is the **sole authority** for all four output file formats. Follow its block formats exactly. Do not invent fields not defined there.

### 6.1 BACKEND.md

For every API endpoint found or clearly described by name in the source docs, use the endpoint block format from `contract-format.md`. If an endpoint is required for the flow but not actually specified, add `⚠️ TBD — endpoint not specified in source` instead of inventing it. Also include:
- State transitions triggered by the endpoint (from FILE_MAP.md States)
- Skill notes: `secure-coding-practices` security annotations; use `security-best-practices` or `security-threat-model` notes where applicable for sensitive integrations

### 6.2 FRONTEND.md

For every major component in admin-ui and user-ui, use the component block format from `contract-format.md`. Also include:
- Module states it displays (from FILE_MAP.md States)
- Role-gated actions it renders (from FILE_MAP.md Actions where applicable)
- Skill notes: `design-auditor` guidance when that skill is available; otherwise document which `component-patterns.md` and `shadcn/ui` conventions were applied. Add `design-auditor` remediation notes if applicable.

### 6.3 INTEGRATIONS.md

For every cross-module dependency edge in FILE_MAP.md, use the integration block format from `contract-format.md`. Also include:
- State changes across boundary
- Skills invoked: e.g. "`email-html-mjml` for notification templates", "`security-threat-model` for sensitive integration boundaries"

### 6.4 PHASES.md

Build phases derived from the source docs (or inferred from dependency ordering in FILE_MAP.md). Use the phase block format from `contract-format.md`. When a phase is inferred rather than explicitly named in the spec, record that in the `Spec sections / derivation` field instead of inventing a section citation. Reference `playwright-skill` and, when preferred in-session, `playwright` patterns in gate checklists for any testable acceptance gates.

---

## PHASE 7 — CONSISTENCY CHECK

Re-read all ten output files from disk:

1. `hierarchy-config/FILE_MAP.md`
2. `hierarchy-config/HIERARCHY.md`
3. `admin-ui/src/App.tsx`
4. `admin-ui/src/data.ts`
5. `user-ui/src/App.tsx`
6. `user-ui/src/data.ts`
7. `build-contract/BACKEND.md`
8. `build-contract/FRONTEND.md`
9. `build-contract/INTEGRATIONS.md`
10. `build-contract/PHASES.md`

Also inspect the React component files under `admin-ui/src/components/` and `user-ui/src/components/` for the UI-specific checks below.

Run this checklist:

```
For every permission string in HIERARCHY.md:
  [ ] Is it referenced in BACKEND.md with the correct role restriction?
  [ ] Is it enforced by a RoleGate or equivalent gating logic in `admin-ui/src/components/`?
  [ ] If it affects UI access, is it documented in FRONTEND.md?

For every module in FILE_MAP.md:
  [ ] Does it have a page in admin-ui?
  [ ] Are its dependency edges shown in the Feature Map?
  [ ] Are its states shown on the State Flow page?
  [ ] Are all branches represented as transitions in the State Flow?
  [ ] Are its documented features represented in the module page Features tab?
  [ ] Do its feature rows in `admin-ui/src/data.ts` match the Features table in FILE_MAP.md?
  [ ] Are its documented actions represented as role-gated action buttons where applicable?
  [ ] Do its action rows in `admin-ui/src/data.ts` match the Actions table in FILE_MAP.md?
  [ ] Is it in BACKEND.md if it has API endpoints?
  [ ] Is it in INTEGRATIONS.md if it crosses module boundaries?
  [ ] Are its skill tags reflected in PHASES.md?

For every API endpoint in BACKEND.md:
  [ ] If it is frontend-facing, is it called from a component in FRONTEND.md?
  [ ] If it is service-to-service or backend-only, is its caller documented in BACKEND.md or INTEGRATIONS.md?
  [ ] Is its role restriction consistent with HIERARCHY.md?
  [ ] Are the state transitions it triggers consistent with FILE_MAP.md?

For every component in FRONTEND.md:
  [ ] Does it exist in admin-ui or user-ui?
  [ ] Is its role gate consistent with HIERARCHY.md?
  [ ] Does it display the correct states from FILE_MAP.md?
  [ ] If it renders role-gated actions, do its `Actions rendered` entries match FILE_MAP.md and the actual UI component?

For `user-ui/src/components/` specifically:
  [ ] Does it expose zero admin controls, permission strings, or dependency info?
  [ ] Does every user-visible module state (from FILE_MAP.md States) render correctly?
  [ ] Do all API endpoints referenced match those in BACKEND.md?
  [ ] Is the user portal home present and showing only user-scoped data?

For every skill tag in FILE_MAP.md:
  [ ] Is the skill's approach applied in the relevant phase output?
  [ ] Is the skill referenced in PHASES.md for the phase where its output is needed?
```

If any check fails, fix the inconsistency before packaging. List all fixes made.

---

## PHASE 8 — PACKAGE & PRESENT

### 8.1 Verify folder structure
```
<project-name>-mapped/
├── hierarchy-config/
│   ├── FILE_MAP.md
│   └── HIERARCHY.md
├── admin-ui/                    ← Vite + React + TS + shadcn/ui project
│   ├── package.json
│   ├── tsconfig.json
│   ├── tsconfig.app.json
│   ├── tsconfig.node.json
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   ├── postcss.config.js
│   ├── components.json
│   ├── index.html
│   └── src/
│       ├── main.tsx
│       ├── App.tsx
│       ├── globals.css
│       ├── data.ts
│       ├── types.ts
│       ├── lib/utils.ts
│       └── components/
│           ├── ui/              ← shadcn components
│           ├── role-gate.tsx
│           ├── app-sidebar.tsx
│           ├── dashboard-page.tsx
│           ├── feature-map-page.tsx
│           ├── state-flow-page.tsx
│           ├── module-page.tsx
│           ├── governance-config-page.tsx   ← include only when source docs justify governance UI
│           └── hierarchy-page.tsx
├── user-ui/                     ← same base toolchain files, user-facing pages only
│   └── ... (same base toolchain structure only; omit admin-only pages)
└── build-contract/
    ├── BACKEND.md
    ├── FRONTEND.md
    ├── INTEGRATIONS.md
    └── PHASES.md
```

For `user-ui/`, "same base toolchain files" means the same base toolchain files only. Do
not include admin-only pages such as Governance Config, Hierarchy, Feature Map, or
State Flow in the packaged user UI.

### 8.2 Copy to outputs
If filesystem output tools are available, copy the entire output to a user-requested destination or keep it in the current repo workspace.
If they are not available, present the generated files inline or offer them file-by-file instead of failing silently.

### 8.3 Present files
If file-presentation tools are available, present files leading with `hierarchy-config/HIERARCHY.md`.
If they are not available, paste the most important files inline in this order:
1. `hierarchy-config/HIERARCHY.md`
2. `hierarchy-config/FILE_MAP.md`
3. `build-contract/PHASES.md`

### 8.4 Handoff summary

Give the user this handoff summary:

```
MAPPING COMPLETE — <project-name>

Structure
─────────
  N modules mapped
  N permissions registered (N flagged as TBD)
  N features mapped across all modules
  N actions mapped across all modules
  N states defined across all modules
  N branches mapped across all modules
  N API endpoints in backend contract
  N components in frontend contract
  N consistency check fixes applied

Skills to invoke for full build
───────────────────────────────
  [list every skill that was tagged across all modules, with the module(s) that need it]

Capability gaps
----------------
  [for every unavailable registry skill, add one line using the exact format:
   `⚠️ SKILL GAP: [skill-name] — required by [module-list] but not available. Source manually or build with skill-creator.`]

What to open first
──────────────────
  1. hierarchy-config/HIERARCHY.md — verify ownership model before building anything
  2. admin-ui dev server (`cd admin-ui && npm install && npm run dev`) — open the State Flow page and verify all states and branches are correct
  3. in that same admin UI, open the Feature Map page — verify all connections are correct
  4. build-contract/PHASES.md — your build order with gate checklists

⚡ Next step
────────────
  Once the backend is built, revisit INTEGRATIONS.md to verify all
  integration contracts are honoured in the real code. Cross-check
  HIERARCHY.md permission strings against actual middleware/RBAC
  implementations to catch enforcement gaps before launch.
```

To produce the tally counts above, re-read:
- `hierarchy-config/FILE_MAP.md` for module count, feature count, action count, state count, and branch count
- `hierarchy-config/HIERARCHY.md` for permission totals and TBD counts
- `build-contract/BACKEND.md` for endpoint count
- `build-contract/FRONTEND.md` for component count
- the Phase 7 fix log for consistency fixes applied

---

## ABSOLUTE RULES

| Rule | Description |
|------|-------------|
| **No invented permissions** | Every permission in HIERARCHY.md must be quoted from a source file |
| **No invented modules** | Every card in FILE_MAP.md must trace to a documented feature |
| **No invented states** | Every state in a module card must be sourced from docs or explicitly TBD-flagged |
| **Skill Registry is mandatory** | Every module must be checked against the Skill Registry. Missing skill tags = incomplete mapping |
| **Checkpoints are mandatory** | Never bypass user confirmation. In fast mode, Checkpoints 1 and 2 become one combined checkpoint that still requires confirmation before proceeding |
| **Re-read before each phase** | Never rely on memory. Always re-read the files written in previous phases |
| **UIs are Vite + shadcn projects** | Generate the full project scaffold for admin-ui and user-ui. Use shadcn/ui components by default; go custom only when shadcn has no matching primitive |
| **State Flow page is not optional** | Admin UI must include a State Flow page showing every module's state machine |
| **Governance config must be interactive when present** | If the source docs justify a Governance Config page, it must have real form inputs and save/export — not just a table |
| **Consistency check is not optional** | All output files listed in Phase 7 plus the UI component files must be cross-checked before packaging |
| **TBD beats invention** | If something can't be sourced, mark it TBD and flag it — never fill in the gap yourself |

