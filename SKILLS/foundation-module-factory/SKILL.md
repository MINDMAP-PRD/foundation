---
name: foundation-module-factory
description: Use when an idea, module concept, project concept, or early source spec needs to be turned into a comprehensive build package on top of the foundation documents in `foundations/`, especially `GeneralApp_Build_Documentation.md`, and optionally `FormBuilder_Build_Documentation_v6.md` when reusable form-engine capability is needed. Creates a folderized package with intake, inventories, foundation-fit analysis, manifest, architecture, build contracts, specs, phases, progress tracking, quality gates, and handoff docs so AI can continue building without drifting. Also use in foundation mode when the user wants to package shared-contract work while the foundations are still being designed.
---

# Foundation Module Factory

This skill turns an idea, module, or evolving project concept into a comprehensive, anti-drift build package folder.

Its job is to produce a package that future AI can reopen and continue from without losing:

- ownership boundaries
- build phases
- progress state
- decisions
- open gaps
- specialist-skill routing

This skill does **not** replace `foundation-extension-auditor`. It composes with it.

It is also the default intake skill whenever a foundation-based project does not already have a governed package.

This skill must not convert a weak or incomplete idea into blind implementation. If the source concept has major logic gaps, contradictions, or missing decisions, it must package the issue and pause for discussion with the user before downstream build work continues.

## Read First

Before doing any work, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. `../../foundations/README.md`
3. `../../foundations/FOUNDATION_MODULE_RULES.md`
4. `../../foundations/FOUNDATION_LIBRARY_ARCHITECTURE.md`
5. `../../foundations/FOUNDATION_DEPENDENCY_STRATEGY.md`
6. `../../foundations/MODULE_MANIFEST_SPEC.md`
7. `../../foundations/CAPABILITY_REGISTRY_SPEC.md`
8. `../../foundations/INTERFACE_REGISTRY_SPEC.md`
9. `../../foundations/EXTENSION_TABLE_REGISTRY_SPEC.md`
10. `../../foundations/FOUNDATION_CHANGE_POLICY.md`
11. `../../foundations/GeneralApp_Build_Documentation.md`
12. `../../foundations/FormBuilder_Build_Documentation_v6.md` only if the module uses or proposes reusable form-engine capability
13. `../foundation-extension-auditor/SKILL.md`
14. [`references/output-package-spec.md`](references/output-package-spec.md)
15. [`references/phase-model.md`](references/phase-model.md)

## Modes

### Module Mode

Use when the target is a product module or project built on the foundations.

### Foundation Mode

Use when the user wants to package shared-contract work for `GeneralApp` or `FormBuilder` themselves.

In foundation mode:

- promotion analysis is mandatory
- shared-contract changes must follow `../../foundations/FOUNDATION_CHANGE_POLICY.md`
- the package must clearly separate current authority from proposed changes

## Core Workflow

1. Identify the input state:
   - raw idea only
   - rough spec/doc
   - detailed product/module doc
   - existing governed package folder
   - foundation change concept
2. Decide the mode:
   - `module`
   - `foundation`
3. Determine dependency shape:
   - `GeneralApp` only
   - `GeneralApp + FormBuilder`
4. Validate package shape before creating new planning artifacts:
   - choose the package shape:
     - `module-generalapp-only`
     - `module-generalapp-plus-formbuilder`
     - `foundation-generalapp`
     - `foundation-formbuilder`
   - only require files that belong to that shape
   - do not create fake UI/module artifacts for foundation work or backend-only work
5. Check package presence before creating new planning artifacts:
   - if a governed package already exists, validate it before trusting it
   - if the package is stale, incomplete, or inconsistent with the current source set, re-baseline it before reuse
   - if no governed package exists, create one
   - package markers and validation rules are defined in [`references/output-package-spec.md`](references/output-package-spec.md)
6. Run `foundation-extension-auditor` logic before packaging ownership-sensitive content:
   - classify duplicates
   - classify conflicts
   - classify valid product extensions
   - classify promotion candidates
7. Check whether the source is build-ready:
   - if the concept is materially incomplete, contradictory, or weak, record the gaps and stop for user discussion before implementation planning goes further
   - if the concept is strong enough to proceed, continue packaging normally
8. Build the package folder using the output spec and the chosen package shape.
9. Populate every required file from traced source information or mark it `TBD - no source found`.
10. Keep a live progress record inside the package as work advances.
11. Re-read package files from disk before each major phase so the package remains the active source of truth.

## Non-Negotiable Rules

1. The package must be folderized and resumable.
2. Every module/package must include inventory, build contract, phases, specs, and progress tracking.
3. Every permission string, capability, interface, setting, event, route, state, and integration must trace either to:
   - a source file
   - a foundation contract
   - or an explicit `TBD - no source found`
4. Do not invent shared ownership. Use `foundation-extension-auditor` decisions.
5. If the idea can stay product-local, keep it product-local.
6. If a shared-contract gap is found, surface it in the foundation-fit section rather than silently patching the foundations.
7. When specialist skills apply, record that in the package and use them if available.
8. If the user says "build a project" and no governed package exists yet, this skill must be the first packaging skill used.
9. If a governed package already exists, do not restart from scratch unless the user explicitly wants a rebuild.
10. Do not treat a package as authoritative until its package shape, source freshness, foundation-fit state, and progress continuity have been validated.
11. If the source idea/spec has materially changed since the package was created, record a re-baseline before continuing implementation planning.
12. When the package includes meaningful UI surfaces, default the UI system to `shadcn/ui` and treat custom components as exceptions that must be justified in the package.
13. If package intake or later implementation work reveals broken logic, unresolved contradictions, or a major missing decision, do not guess silently; record the issue and discuss it with the user before proceeding with substantive build work.

## Specialist Skill Routing

Apply the interconnection map from [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md).

Minimum expectations:

- `foundation-extension-auditor` for ownership and promotion analysis
- `coding-standards` for TypeScript/JavaScript/React/Node implementation conventions once package work reaches implementation-ready contracts or code
- `api-design` for backend contract shape, endpoint conventions, pagination, filtering, error responses, versioning, and rate-limit decisions
- `backend-patterns` for backend layering, data-access patterns, query optimization, caching, queues, logging, and operational structure
- `continuous-agent-loop` when the package needs autonomous implementation flow, CI-style iteration, staged agent pipelines, or multi-agent orchestration
- `article-writing` for polished long-form guides, tutorials, launch docs, newsletters, and narrative deliverables grounded in the package
- `secure-coding-practices` for security-sensitive backend/auth/API areas
- `security-best-practices` when the session-available specialist skill is preferred
- `security-threat-model` for high-risk trust-boundary work when available
- `design-auditor` for UI review and design checkpoints
- `playwright-skill` for browser/E2E acceptance planning and test implementation
- `playwright` when the session-available browser automation skill is preferred for direct execution
- `email-html-mjml` for email-template surfaces
- `owner-dashboard-mapper` when the package reaches owner/admin control-surface mapping
- `deep-audit` for exhaustive review after package/spec work exists
- `doc` / `pdf` for output-format-specific deliverables when needed
- `skill-creator` if the package reveals a new reusable capability gap that should become a skill

If a specialist skill is unavailable, note:

- `Capability gap - skill unavailable in current session`

## Package Construction Rules

- Use the exact package structure in [`references/output-package-spec.md`](references/output-package-spec.md).
- Use the exact phase model in [`references/phase-model.md`](references/phase-model.md).
- Apply the correct package shape for the current mode and dependency set.
- Keep the package internally consistent:
  - package summary must match manifest
  - inventories must match contracts
  - contracts must match specs
  - phases must match acceptance matrix
  - progress must match actual completed files and decisions
- For UI-bearing packages, keep the UI system internally consistent:
  - `UI_ARCHITECTURE.md`, `FRONTEND_CONTRACT.md`, `ADMIN_SPEC.md`, and `USER_SPEC.md` must all reflect the same `shadcn/ui`-first baseline
  - custom components must be explicitly marked as `custom - no suitable shadcn primitive`
- Validate resume safety before reusing an existing package:
  - package markers exist
  - package shape still matches the current task
  - source summary still matches the active source set
  - foundation-fit decisions are present and still valid
  - progress and next actions are coherent
  - unresolved blockers are still visible

## Progress Discipline

The package is the continuity layer.

That means:

- every checkpoint updates `07_progress/PROGRESS.md`
- every meaningful decision updates `07_progress/DECISIONS.md`
- every unresolved blocker updates `07_progress/OPEN_QUESTIONS.md`
- every scope or contract change updates `07_progress/CHANGE_LOG.md`
- every source-set change or package-shape change updates the re-baseline record in `07_progress/CHANGE_LOG.md`

Future AI should be able to resume by reading the package instead of reconstructing context from chat history.

## Output Expectation

Produce a single package folder named like:

- `<module-slug>-package/` for module mode
- `<foundation-change-slug>-foundation-package/` for foundation mode

Do not stop at a prose plan if the user wants the package created. Create the folder structure and write the files.
