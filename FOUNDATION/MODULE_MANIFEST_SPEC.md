# MODULE MANIFEST SPEC

Contract for Registering a Product Module on Top of the Foundations

> **Purpose:** This file defines the minimum manifest structure a product module should expose so it can be installed, enabled, disabled, audited, and removed without directly editing the foundation contracts.

---

## 1. Why a Manifest Exists

A module manifest is the module's registration contract with the shared foundations.

It exists so the foundations can discover and load module behavior safely through a declared interface instead of through hidden assumptions or direct foundation edits.

---

## 2. Minimum Required Fields

Every module manifest should declare at least:

- `module_key`
- `module_name`
- `module_version`
- `foundation_requirements`
- `interface_requirements`
- `ownership`
- `registrations`
- `capabilities`
- `lifecycle`
- `metadata`

---

## 3. Recommended Manifest Shape

```yaml
module_key: familyvault
module_name: FamilyVault
module_version: 1.0.0

foundation_requirements:
  general_app: ">=6.1"
  form_builder: ">=6.1"   # include only when the module depends on FormBuilder

interface_requirements:
  bridge_handler_contract: "1.x"
  form_action_contract: "1.x"
  adapter_contract: "1.x"
  ui_surface_contract: "1.x"

ownership:
  tables:
    - familyvault_vaults
    - familyvault_profiles
    - familyvault_relationships
  routes:
    - /familyvault/*
  permissions:
    - fv.vaults.view
    - fv.vaults.manage
  event_namespaces:
    - fv.*
  settings_keys:
    - modules.familyvault.*

registrations:
  bridge_handlers:
    - familyvault-profile-bridge
  form_actions:
    - fv.createVaultRecord
  adapters:
    - fv.memorial-exporter
  ui_surfaces:
    - admin.nav.familyvault

capabilities:
  required:
    - files
    - audit
    - rbac
  optional:
    - webhooks
    - forms
    - connectors

lifecycle:
  install:
    creates_tables: true
    registers_permissions: true
    registers_routes: true
  enable:
    activates_handlers: true
  disable:
    removes_runtime_registrations: true
  uninstall:
    removes_module_registrations: true
    archives_module_data: true

metadata:
  product_local_flags:
    memorial_mode_enabled: true
  notes:
    - "Product-local flags belong here, not in the shared capability registry."
```

---

## 4. Field Rules

### 4.1 `module_key`

- unique, stable, lowercase key
- used for namespacing
- should not collide with another module

### 4.2 `foundation_requirements`

- declares the compatible foundation versions
- prevents a module from assuming contracts that do not exist
- may include only `general_app` for modules that do not depend on FormBuilder

### 4.3 `interface_requirements`

- declares which extension-interface versions the module targets
- prevents a module from claiming compatibility while registering against stale or incompatible hook shapes
- should reference only documented interface contracts exposed by the foundations
- valid interface names and version lines must come from [INTERFACE_REGISTRY_SPEC.md](INTERFACE_REGISTRY_SPEC.md)

### 4.4 `ownership`

Declares what the module owns directly.

The module should list:

- module-owned tables
- module-owned routes
- module-owned permissions
- module-owned event namespaces
- module-owned settings keys

If it is not listed here, it should not be assumed to be module-owned.

Module-owned permissions and event namespaces are still registered into the shared foundation registries. Ownership does not make the module a parallel registry authority.

### 4.5 `registrations`

Declares what the module plugs into.

This includes:

- bridge handlers
- adapters
- form actions
- field plugins
- UI extension points

If a module does not register any shared hooks or extension surfaces, `registrations: {}` is valid.

### 4.6 `capabilities`

Split into:

- `required`: the foundations/services the module cannot run without
- `optional`: integrations the module can use if present

Capability names must come from [CAPABILITY_REGISTRY_SPEC.md](CAPABILITY_REGISTRY_SPEC.md).
Optional capabilities must always be checked before use.

### 4.7 `lifecycle`

Documents what happens when the module is:

- installed
- enabled
- disabled
- uninstalled

This is required for safe removal and operational clarity.

At minimum:

- `disable` should remove runtime registrations only
- `disable` should not delete or archive module-owned data
- `uninstall` may remove registrations and remove or archive module-owned data

### 4.8 `metadata`

- holds product-local flags and other module-local notes that do not belong in the shared capability registry
- may be freely extended by the module as long as it does not redefine shared foundation contracts

## 4.9 Empty-State Guidance

- `interface_requirements: {}` is valid when the module does not target any shared extension interfaces
- `registrations: {}` is valid when the module does not register any shared hooks or extension surfaces
- this is expected for modules that depend only on `GeneralApp` and do not extend shared runtime/plugin surfaces

---

## 5. Manifest Guardrails

A module manifest must not:

- claim ownership of shared foundation tables
- redefine shared foundation schemas
- assume another module is installed
- register product-specific names into shared contracts without namespace discipline
- hide side effects that alter foundation behavior globally

---

## 6. Manifest and Auditing

The manifest should be used during audits to answer:

- what the module owns
- what the module registers
- what shared capabilities it depends on
- whether uninstall is safe
- whether a declared feature is product-local or a promotion candidate

---

## 7. Practical Rule

If a new module cannot explain itself with a manifest, it is likely too entangled with the foundations and needs better boundaries before implementation.
