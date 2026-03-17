# FOUNDATION MODULE RULES

Shared Operating Rules for `GeneralApp` + `FormBuilder` + Any Future Product Module

> **Read-first rule:** Any AI or engineer creating, editing, or auditing a product/module spec on top of the foundations **MUST read this file first** before making changes elsewhere.
>
> **Library model:** The foundations are to be treated as a **shared library/framework**. Product modules consume and register against them; they do not directly rewrite them during normal module work.
>
> **Entry point:** If the reader does not yet know how the foundations are structured, start with [README.md](README.md) and then return to this file.

---

## 1. Purpose

This file explains how the foundations work so new modules can build on them safely without:

- re-declaring foundation-owned contracts
- mixing one module's data with another module's data
- extending shared behavior in a way that breaks modules that do not use that extension
- creating parallel platform systems inside a product spec

The foundations are intended to stay stable. Product modules are the layer that changes.
The detailed architecture for this model is documented in [FOUNDATION_LIBRARY_ARCHITECTURE.md](FOUNDATION_LIBRARY_ARCHITECTURE.md). Dependency and provider rules are documented in [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md). Direct-edit guardrails for the foundations are documented in [FOUNDATION_CHANGE_POLICY.md](FOUNDATION_CHANGE_POLICY.md).

---

## 2. Platform Model

Use this mental model:

- `GeneralApp` = shared platform library/runtime and shared contracts
- `FormBuilder` = shared reusable form-engine library
- `Product Module` = product-specific plugin/package built on top of `GeneralApp`, with `FormBuilder` used only when the module needs reusable form-engine capability

The foundations are **not copied into each module**. They are shared once and consumed by many modules.
Modules are installed, registered, enabled, disabled, and removed without mutating the foundation contracts by default.

Modules should still build quickly by using proven frameworks and libraries for module-local concerns, but those choices must remain module implementation details unless explicitly promoted into the foundations.

---

## 3. Ownership Rules

### 3.1 GeneralApp owns

- shared `users`
- shared `sessions`
- shared `file_uploads`
- shared audit infrastructure
- shared connector and capability registry mechanics
- shared RBAC registry and permission registration
- shared settings hierarchy
- shared webhooks, API infrastructure, rate limits, and platform security contracts

### 3.2 FormBuilder owns

- reusable form definitions
- reusable submissions and drafts
- reusable field/runtime behavior
- form-specific extension points
- form-specific permissions registered into the shared RBAC system
- semantic definitions for reusable form-engine capabilities published through the shared capability registry

### 3.3 Product modules own

- product-specific entities
- product-specific workflows
- product-specific statuses and lifecycle states
- product-specific tables
- product-specific routes and UI behavior
- product-specific bridge handlers and adapters using foundation extension points
- module manifests and installation metadata
- module-owned permission keys and event namespaces that are registered into the shared foundation registries
- module-local framework and library choices for product-only implementation concerns

### 3.4 Product modules do not own

- alternative `users` or `sessions` authority models
- alternative audit platforms
- alternative connector registries
- alternative shared RBAC authorities
- direct mutation of foundation behavior for all modules
- direct edits to foundation contracts during ordinary module work

If a module declares one of those as if it owns it, that is a foundation conflict.

---

## 4. Data Isolation Rules

Modules must be isolated from one another by contract.

### 4.1 Shared data stays in shared tables

A module may reference shared entities such as:

- `user_id`
- `session_id`
- `file_upload_id`
- `connector_id`

But a module must not redefine or fork the shared table that owns those IDs.

### 4.2 Module data stays in module-owned tables

Module-specific fields must live in module-specific tables.

Examples:

- good: `familyvault_profiles.user_id -> users.id`
- bad: adding many FamilyVault-only columns directly to `users`

### 4.3 Cross-module access must be explicit

One module must not read another module's private tables by assumption.

Cross-module interaction must go through one of:

- shared foundation contracts
- documented bridge handlers
- documented public APIs
- documented capability bindings

---

## 5. Extension Rules

Modules may **register** behavior. They may not silently rewrite shared foundation behavior.

Allowed extension patterns:

- plugin registration
- module manifests
- bridge handlers
- capability bindings
- module-specific settings
- module-specific permissions and events under the shared registry
- optional adapters enabled only when the module is installed
- module-owned migrations that touch only module-owned storage or formally approved extension tables

Disallowed extension patterns:

- changing a shared schema for one module's private needs
- changing shared lifecycle rules globally without promotion
- assuming all other modules support the same optional behavior
- introducing product-specific enum values into a shared contract without formal foundation expansion
- editing a foundation file as a side effect of ordinary module implementation
- turning a module's preferred framework into an implied shared foundation standard without explicit promotion

### 5.1 Extension tables

An extension table is a shared, foundation-approved integration table intended for module registrations or bridge mappings.

An extension table is **not**:

- a module-owned business table
- a replacement for a shared foundation table
- a shortcut for adding product-local fields into shared contracts

Extension tables may be used only when:

- the table is explicitly documented as a shared extension surface
- the owning foundation is named
- the table's allowed module interaction is defined

The canonical list of approved extension tables is documented in [EXTENSION_TABLE_REGISTRY_SPEC.md](EXTENSION_TABLE_REGISTRY_SPEC.md).

If those conditions are not met, the storage should be treated as either module-owned or foundation-owned, not as an implicit hybrid.

---

## 6. Compatibility Rules

Every module must be safe to enable or disable without breaking unrelated modules.

That means:

- optional features must be capability-checked
- missing module handlers must fail safely
- shared code must not hardcode joins to module tables
- shared code must not assume a module-specific field exists
- shared code must not assume a module-specific workflow is present

If a new module feature requires changing shared behavior for everyone, that feature is not product-local anymore and must be reviewed for promotion.

### 6.1 Module removal rule

Disabling a module must remove only:

- its routes
- its handlers
- its UI registrations
- its module-owned settings
- its active module-owned permissions/events from runtime use

Disabling a module must **not** remove or archive module-owned tables or data.

Uninstalling a module may remove or archive:

- its routes
- its handlers
- its UI registrations
- its module-owned settings
- its module-owned permissions/events registrations
- its module-owned tables or archived module data

Disabling or uninstalling a module must **not** require the foundations themselves to be reverted.

---

## 7. Promotion Rules

Sometimes a product module introduces a feature that is generic enough to belong in a foundation.

Promote a module feature only if it is:

- reusable across multiple future products
- not tightly bound to one product domain
- better as a shared contract than as repeated product logic
- not already expressible through existing extension hooks

If promoted, the feature must be added to the correct foundation:

- `GeneralApp` for shared platform primitives
- `FormBuilder` for reusable form engine behavior

If it does not clearly belong to either, stop and treat it as an architecture decision rather than forcing it into the wrong place.

Promotion is the exception path, not the default module path.

---

## 8. Safe Module Expansion Pattern

When a new product spec adds something the foundations do not yet cover:

1. Identify the overlapping part, if any, with existing foundation contracts.
2. Keep the overlapping part referenced from the foundation rather than re-declared.
3. Keep product-only behavior inside the product module.
4. Promote only the reusable missing capability upward.
5. Ensure promotion includes adjacent dependency updates:
   - RBAC keys
   - audit/event namespaces
   - settings behavior
   - bridge hooks
   - API/webhook contracts
   - security constraints

If the feature can be expressed through existing extension points, keep it in the module instead of promoting it.

---

## 9. Module Manifest Rule

Every product module should define a module manifest. The manifest contract is specified in [MODULE_MANIFEST_SPEC.md](MODULE_MANIFEST_SPEC.md).

At minimum, a module manifest should declare:

- module key
- module version
- required foundation versions
- required interface versions
- owned tables
- owned routes
- owned permissions and event namespaces
- required capabilities
- optional capabilities
- registered hooks, handlers, and adapters
- install, enable, disable, uninstall expectations
- module-local metadata

The manifest is how the module is "called" by the shared foundations without editing them directly.
Capabilities and interface names used by the manifest must come from documented shared registries rather than ad hoc module vocabulary.
Valid shared interface names and version lines are documented in [INTERFACE_REGISTRY_SPEC.md](INTERFACE_REGISTRY_SPEC.md).

---

## 10. AI Working Rules

Before creating or editing any new module/spec file:

1. Read [README.md](README.md).
2. Read this file.
3. Read [FOUNDATION_LIBRARY_ARCHITECTURE.md](FOUNDATION_LIBRARY_ARCHITECTURE.md).
4. Read [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md).
5. Read [MODULE_MANIFEST_SPEC.md](MODULE_MANIFEST_SPEC.md).
6. Read [CAPABILITY_REGISTRY_SPEC.md](CAPABILITY_REGISTRY_SPEC.md).
7. Read [INTERFACE_REGISTRY_SPEC.md](INTERFACE_REGISTRY_SPEC.md).
8. Read [EXTENSION_TABLE_REGISTRY_SPEC.md](EXTENSION_TABLE_REGISTRY_SPEC.md).
9. Read [FOUNDATION_CHANGE_POLICY.md](FOUNDATION_CHANGE_POLICY.md).
10. Read `GeneralApp_Build_Documentation.md`.
11. Read `FormBuilder_Build_Documentation_v6.md` only when the module uses or proposes reusable form-engine capability.
12. Read `../skills/foundation-extension-auditor/SKILL.md` before auditing ownership, duplicates, conflicts, or promotion candidates.

Any AI working on a new module must answer these questions first:

- Is this concept already owned by `GeneralApp`?
- Is this concept already owned by `FormBuilder`?
- Is this only product-local?
- Is this a reusable missing feature that should be promoted?
- If promoted, what adjacent contracts must change too?

---

## 11. Foundation Change Policy

During ordinary module work, AI should edit the module spec and manifest, not the foundations.

Foundation edits are allowed only when one of these is true:

- the user explicitly asked for shared-contract or foundation work
- an audit concluded that promotion into a foundation is required
- the change is a cross-foundation correction already approved as a shared contract fix

If a foundation change is proposed, the change must state:

- why extension hooks are insufficient
- which foundation owns the promoted change
- what modules are affected
- what compatibility checks are required

An explicit user request does **not** by itself justify placing product-local behavior into a foundation. Extension-first and promotion rules still apply.

---

## 12. Non-Negotiable Guardrails

- Do not redefine foundation-owned schemas in product specs.
- Do not add product-only columns to shared tables unless the foundations are intentionally expanded.
- Do not let one module's optional feature become a hidden dependency for another module.
- Do not let product docs act like parallel platforms.
- Do not introduce product-specific naming into shared contracts when a neutral shared term is available.
- Do not directly edit foundation files during ordinary module work.
- Do not make a module uninstall depend on reverting foundation edits.

The goal is one shared platform, many isolated modules, and deliberate promotion of reusable capabilities.
