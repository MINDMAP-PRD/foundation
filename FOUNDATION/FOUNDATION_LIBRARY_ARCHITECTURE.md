# FOUNDATION LIBRARY ARCHITECTURE

Library/Plugin Operating Model for `GeneralApp` + `FormBuilder` + Product Modules

> **Purpose:** This document defines how the foundations are meant to be used like a shared library/framework, with product modules loaded on top through manifests, registries, hooks, and capability checks instead of direct foundation rewrites.
>
> **Implementation note:** For the practical repo layout of where the real code should be built, read [FOUNDATION_IMPLEMENTATION_BLUEPRINT.md](FOUNDATION_IMPLEMENTATION_BLUEPRINT.md).
>
> **Dependency note:** For the recommended third-party stack and the rules for keeping providers swappable, read [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md).

---

## 1. Core Idea

Treat the foundations the way a product would treat a framework such as Bootstrap:

- the framework is shared
- the app consumes and configures it
- the app extends it through supported interfaces
- the app does not rewrite the framework during ordinary feature work

For this repo:

- `GeneralApp` is the shared platform library
- `FormBuilder` is the shared reusable form-engine library
- each product spec is a module/plugin package built on top of `GeneralApp`, with `FormBuilder` used only when the module needs reusable form-engine capability

---

## 2. Runtime Shape

The intended architecture is:

1. The foundation libraries expose stable shared contracts.
2. A module declares what it owns and what it registers through a manifest.
3. The platform loads the module through registries and capability checks.
4. The module can be enabled or disabled without requiring the foundations themselves to change.

This means modules are consumers of the platform, not forks of it.

### 2.1 Repo topology options

The same operating model works in both single-repo and multi-repo setups.

#### Single-repo

Use a single repo when the foundations are still evolving quickly or when shared platform work and module work need tight coordination.

Typical shape:

- `foundations/`
- `skills/`
- `apps/`
- `packages/`
- `modules/`
- `supabase/`

In this model:

- the foundations are built once
- modules live in separate folders/packages
- each module builds only itself
- shared code is imported from shared packages
- shared platform behavior is consumed through shared services and registries

#### Multi-repo

Use multiple repos when the shared contracts are stable enough to be published and consumed externally.

Typical split:

- one repo for `GeneralApp` platform services and shared packages
- one repo for `FormBuilder` packages
- one repo per product module

In this model:

- modules install shared packages through npm
- modules call shared hosted platform APIs/services
- modules still register through manifests and extension interfaces
- modules do not copy the foundations into their own repo as a forked baseline

In both models, the core rule is the same: the foundation is shared once, and modules attach to it.

---

## 3. What the Foundations Expose

### 3.1 GeneralApp exposes

- shared identity/session contracts
- shared storage/file contracts
- shared audit contracts
- shared connectors and capability registry
- shared RBAC registry
- shared settings hierarchy
- shared webhook/API infrastructure
- shared rate limiting, security, and cross-module contracts

### 3.2 FormBuilder exposes

- form-definition schema
- submission and draft contracts
- field/runtime engine contracts
- reusable validation and submission lifecycle hooks
- form-specific plugin interfaces

---

## 4. Provider and Dependency Model

The foundation should move quickly by reusing strong libraries and hosted services, but it must not leak those vendor choices into the shared module contract.

### 4.1 Core rule

- the foundation contract is ours
- provider implementations are replaceable
- modules should call foundation packages, services, and capabilities
- modules should not import vendor SDKs directly for foundation-owned concerns

### 4.2 Adapter rule

Shared concerns should be implemented behind adapters or capability bindings such as:

- auth adapters
- policy adapters
- storage adapters
- payment adapters
- email adapters
- search adapters
- job/workflow adapters
- CAPTCHA adapters

### 4.3 Paid-service rule

Paid or hosted products may be used to accelerate delivery.

They must remain:

- optional from the perspective of the architecture
- isolated from canonical manifest and registry contracts
- replaceable without requiring modules to change their shared integration contract

### 4.4 Recommended defaults

The recommended default stack and acceptable alternatives are documented in [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md).

---

## 5. What Modules Register

A product module should register itself through a manifest and module-owned extension definitions.

Typical module registrations:

- routes
- settings keys
- module-owned permissions registered into the shared RBAC registry
- module-owned event namespaces registered into the shared audit/event registry
- bridge handlers
- adapters
- form actions or field plugins
- UI navigation and admin surfaces
- module-owned migrations and tables

All of those must be module-scoped.

---

## 6. Isolation Model

The architecture must prevent one module from contaminating another.

### 6.1 Data isolation

- shared entities remain in shared foundation tables
- module-specific fields remain in module-specific tables
- cross-module reads must be explicit

### 6.2 Behavior isolation

- a module must not change shared defaults for all modules
- optional logic must be activated only when the module is present
- missing module handlers must fail safely

### 6.3 Naming isolation

- module routes should be namespaced
- module permissions should be namespaced
- module events should be namespaced
- module tables should be clearly module-owned

---

## 7. Capability Model

The foundation should not assume every module supports every feature.

Capability names must come from the canonical registry defined in [CAPABILITY_REGISTRY_SPEC.md](CAPABILITY_REGISTRY_SPEC.md). Modules may not invent ad hoc shared capability names in manifests or registrations without extending that registry.

Instead, shared code should use capability checks such as:

- is the module installed?
- is the module enabled?
- is the handler registered?
- is the optional capability available?
- is the required adapter configured?

Anything optional must be checked before use.

### 7.1 Extension interface versions

Modules should also declare the version of the extension interfaces they register against, such as:

- bridge handler contracts
- form action contracts
- field plugin contracts
- adapter contracts
- UI extension surface contracts

Foundation version compatibility alone is not enough if the hook/interface contract has evolved independently.
Valid interface names and version lines must come from [INTERFACE_REGISTRY_SPEC.md](INTERFACE_REGISTRY_SPEC.md).

If a module does not register against any shared extension interfaces, an empty interface declaration is valid.

---

## 8. Install / Enable / Disable / Uninstall

### 8.1 Install

Installing a module should:

- register the manifest
- create module-owned tables
- register module routes, handlers, permissions, and settings
- attach UI and workflow extensions through supported hooks

### 8.2 Enable

Enabling a module should turn on its registrations and optional capabilities.

### 8.3 Disable

Disabling a module should turn off its registrations without breaking shared platform behavior or removing module-owned data.

### 8.4 Uninstall

Uninstalling a module should remove or archive:

- module routes
- module handlers
- module permissions/events/settings
- module-owned tables or module-owned data

Uninstalling a module must not require reverting foundation changes that were only made for that module.

---

## 9. Extension Versus Promotion

The default path is extension, not promotion.

Use extension when:

- the feature is product-local
- the feature fits an existing hook
- the feature does not need a new shared contract

Use promotion only when:

- the feature is reusable across multiple future modules
- existing hooks are not enough
- the change belongs clearly to `GeneralApp` or `FormBuilder`
- the change can be made as a deliberate shared-contract update

---

## 10. Safe Foundation Evolution

To keep the foundations stable:

- module work should happen in module files and manifests
- foundation edits should happen only as explicit promotion work
- every promotion must evaluate adjacent impact:
  - RBAC
  - audit/events
  - settings
  - hooks/bridges
  - APIs/webhooks
  - security
  - compatibility

---

## 11. Documentation Expectations

Every new product module should provide:

- a product/module spec
- a module manifest
- module-owned data declarations
- extension registrations
- explicit references to shared foundation contracts instead of re-declaring them

The foundation docs should remain generic and stable; the product docs should carry the domain-specific change.

---

## 12. Bottom Line

The foundations are meant to behave like a framework/library layer.

Modules:

- extend the foundations
- register into the foundations
- depend on the foundations
- can be removed independently

Modules should not:

- fork the foundations
- redefine shared contracts
- hard-wire global changes into the foundations during ordinary module work
