# FOUNDATIONS README

Entry Point for `GeneralApp` + `FormBuilder` + Module Architecture

> **Purpose:** This is the first file an AI or engineer should read to understand how the foundations work, how product modules build on top of them, and how the same foundation model applies in both single-repo and multi-repo setups.

---

## 1. What This Folder Is

This folder contains the canonical operating model for the shared foundations:

- `GeneralApp_Build_Documentation.md` = shared platform contracts
- `FormBuilder_Build_Documentation_v6.md` = reusable form-engine contracts
- `FOUNDATION_MODULE_RULES.md` = day-to-day operating rules for modules
- `FOUNDATION_LIBRARY_ARCHITECTURE.md` = library/plugin architecture model
- `FOUNDATION_IMPLEMENTATION_BLUEPRINT.md` = practical repo layout for where the real foundation and module code should be built
- `FOUNDATION_DEPENDENCY_STRATEGY.md` = recommended libraries, services, and adapter rules for building fast without hardwiring vendors into the foundation
- `BUILD_PLAN_EXTRACTS/` = extracted foundation-owned guidance from the broader `BUILD-PLAN/` corpus, including what should be promoted into `GeneralApp` and how the comprehensive pieces interconnect
- `MODULE_MANIFEST_SPEC.md` = module registration contract
- `CAPABILITY_REGISTRY_SPEC.md` = canonical shared capability names
- `INTERFACE_REGISTRY_SPEC.md` = canonical shared interface contracts and version lines
- `EXTENSION_TABLE_REGISTRY_SPEC.md` = approved shared extension-table surfaces
- `FOUNDATION_CHANGE_POLICY.md` = guardrails for editing shared foundations

---

## 2. Core Mental Model

Treat the foundations as a shared library/framework:

- `GeneralApp` is the shared platform layer
- `FormBuilder` is the shared optional form-engine layer
- product modules are plugin packages built on top of `GeneralApp`, with `FormBuilder` used only when needed

Modules do **not** copy the foundations into themselves.

Modules should:

- reference shared contracts
- install/register through manifests
- import shared packages
- call shared APIs/services
- keep module-specific data in module-owned tables

Modules should **not**:

- redefine shared `users`, `sessions`, files, audit, connectors, or RBAC contracts
- create parallel platform systems
- patch the foundations during ordinary module work

---

## 3. How the Foundations Work

The foundations are meant to be built once and consumed many times.

The normal model is:

1. Build the shared foundation runtime and shared packages once.
2. Let modules register themselves through manifests, hooks, capabilities, and interfaces.
3. Let each module own only its own domain tables, routes, UI, and workflows.
4. Let modules be enabled, disabled, and removed without rewriting the foundations.

To move faster:

- prefer proven libraries and services for commodity infrastructure
- keep those dependencies behind foundation-owned adapters and capability bindings
- keep paid or hosted vendors optional, not core to the shared contract model
- use proven frameworks for module-local implementation too, as long as those choices stay inside the module boundary unless explicitly promoted
- use [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md) as the implementation rule for this

This is why the foundations are described as a library/framework rather than a set of copied starter files.

---

## 4. Single-Repo Model

Use a single repo when the foundations are still evolving quickly or when you want tight coordination between shared platform work and module work.

Recommended shape:

```text
repo/
|-- foundations/
|-- skills/
|-- apps/
|   |-- platform-api/
|   |-- admin-shell/
|   `-- module-host/
|-- packages/
|   |-- foundation-types/
|   |-- foundation-sdk/
|   |-- foundation-ui/
|   |-- form-builder-core/
|   |-- form-builder-ui/
|   `-- module-manifest/
|-- modules/
|   |-- client-portal/
|   `-- another-module/
`-- supabase/
```

In a single repo:

- the foundations are shared once
- each module lives in its own folder/package
- each module builds only itself
- modules import shared packages instead of copying shared code
- modules call shared foundation APIs/services instead of re-implementing them

This means a monorepo is shared organization, not one giant merged build.

---

## 5. Multi-Repo Model

Use a multi-repo model when the foundation contracts are stable enough to be shared through package publishing and hosted APIs.

Recommended split:

```text
generalapp-platform/
|-- foundations/
|-- apps/platform-api/
`-- packages/foundation-*

formbuilder/
|-- foundations/
`-- packages/form-builder-*

product-module/
|-- module.manifest.json
|-- docs/
|-- db/
|-- backend/
`-- frontend/
```

In a multi-repo setup:

- shared packages are installed through npm
- shared backend/platform services are hosted once
- each product module remains in its own repo
- each module imports the shared SDK/packages and calls the shared platform API
- each module still owns only its own data and registrations

The multi-repo model should not turn into copy-pasting the foundations into every product repo.

---

## 6. What "Library" Means In Practice

The foundation is not primarily a CDN asset.

The practical library model is:

- shared packages for types, SDKs, UI, and reusable engines
- shared backend services for auth, sessions, files, audit, settings, connectors, and other platform concerns
- manifests and registries for installing module-specific extensions

That means modules usually attach to the foundation through:

- package imports
- API/service calls
- manifest registration
- hook/bridge/interface registration

Not by rebuilding the foundations inside every module.

---

## 7. Build and Deployment Rule

Even in a single repo, each module should build only its own project.

The shared foundation can be built and deployed once as:

- platform API/runtime
- shared packages
- shared registries

Then each module builds independently against those shared contracts.

Deleting a module should remove:

- its own code
- its own manifest registrations
- its own module-owned tables/data according to uninstall rules

Deleting a module should **not** require deleting or rewriting the foundation.

---

## 8. Read Order For AI

Before building or auditing a module, read in this order:

1. `README.md`
2. `FOUNDATION_MODULE_RULES.md`
3. `FOUNDATION_LIBRARY_ARCHITECTURE.md`
4. `FOUNDATION_IMPLEMENTATION_BLUEPRINT.md`
5. `FOUNDATION_DEPENDENCY_STRATEGY.md`
6. `MODULE_MANIFEST_SPEC.md`
7. `CAPABILITY_REGISTRY_SPEC.md`
8. `INTERFACE_REGISTRY_SPEC.md`
9. `EXTENSION_TABLE_REGISTRY_SPEC.md`
10. `FOUNDATION_CHANGE_POLICY.md`
11. `BUILD_PLAN_EXTRACTS/BUILD_PLAN_FOUNDATION_PROMOTION_MAP.md` when mapping broader comprehensive plans into the shared foundations
12. `BUILD_PLAN_EXTRACTS/FOUNDATION_COMPREHENSIVE_WIRING_PLAN.md` when building owner/admin comprehensive wiring into `GeneralApp`
13. `GeneralApp_Build_Documentation.md`
14. `FormBuilder_Build_Documentation_v6.md` only when the module uses or proposes reusable form-engine capability
15. `../skills/foundation-extension-auditor/SKILL.md` before ownership audits or promotion decisions
16. `../skills/foundation-module-factory/SKILL.md` when turning an idea or early spec into a governed build package

---

## 9. Bottom Line

If a module needs the foundation:

- do not copy the foundation into the module
- do not rebuild the platform inside the module
- do not let the module redefine shared contracts

Instead:

- import the shared packages
- call the shared services
- register the module through manifests and extension interfaces
- keep the module's own code and data isolated

The foundations are shared once. Modules attach to them.
