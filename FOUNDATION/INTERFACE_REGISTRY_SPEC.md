# INTERFACE REGISTRY SPEC

Canonical Index of Shared Extension Interface Contracts

> **Purpose:** This document defines the valid shared extension interface names, their owning foundation, and their version lines so modules do not invent interface contracts ad hoc.

---

## 1. Default Rule

If an interface name is not listed in this registry, treat it as **not approved** as a shared extension interface.

In that case, the interface should be treated as either:

- module-local only
- or a promotion candidate that requires explicit foundation work

but not as an implied shared contract.

---

## 2. Required Fields for Any Approved Interface

Every approved interface entry must declare:

- interface name
- owning foundation
- purpose
- supported version line(s)
- who may register against it
- whether it is optional or required for the modules that use it

---

## 3. Current Registry

### 3.1 GeneralApp-owned interfaces

- `bridge_handler_contract`
  - owner: `GeneralApp`
  - purpose: module bridge handlers that connect product-local workflows to shared platform contracts
  - supported versions: `1.x`
  - registration scope: modules may register against it when using shared bridge surfaces
  - optionality: optional

- `adapter_contract`
  - owner: `GeneralApp`
  - purpose: module adapters that integrate product-local behavior with shared platform capability surfaces
  - supported versions: `1.x`
  - registration scope: modules may register against it when using shared adapter surfaces
  - optionality: optional

- `ui_surface_contract`
  - owner: `GeneralApp`
  - purpose: module registration into shared UI/admin extension surfaces
  - supported versions: `1.x`
  - registration scope: modules may register against it when extending shared UI surfaces
  - optionality: optional

### 3.2 FormBuilder-owned interfaces

- `form_action_contract`
  - owner: `FormBuilder`
  - purpose: reusable post-submit and form-action registrations
  - supported versions: `1.x`
  - registration scope: modules may register against it only when they depend on FormBuilder
  - optionality: optional

- `field_plugin_contract`
  - owner: `FormBuilder`
  - purpose: reusable field/runtime plugin registrations
  - supported versions: `1.x`
  - registration scope: modules may register against it only when they depend on FormBuilder
  - optionality: optional

---

## 4. Empty-State Rule

A module may validly declare:

- `interface_requirements: {}`
- `registrations: {}`

when it does not register against any shared extension interfaces.

This is normal for modules that are primarily data- or workflow-centric and do not extend shared runtime hooks.

---

## 5. Adding an Interface

Add a new interface here only when:

- the contract is reusable across multiple modules
- the owning foundation is clear
- the version line is explicit
- the registration surface is stable enough to be shared

Adding a new shared interface should be handled as explicit foundation work, not as ordinary module implementation.

---

## 6. Version Lifecycle Rule

When an interface version line is no longer the preferred line, this registry should mark it explicitly as one of:

- `active`
- `deprecated`
- `retired`

Interpretation:

- `active`: new module work may target this version line
- `deprecated`: existing modules may remain on it temporarily, but new module work should target the newer active line
- `retired`: modules should not newly target it, and existing usage should be treated as a migration concern

If multiple version lines are supported at once, the registry entry should state which one is preferred for new work.
