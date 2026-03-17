# CAPABILITY REGISTRY SPEC

Canonical Vocabulary for Shared Module Capabilities

> **Purpose:** This document defines how shared capability names are owned, named, and used so product modules do not invent incompatible capability vocabulary.

---

## 1. Ownership

The canonical shared capability registry is owned by `GeneralApp`.

`GeneralApp` owns the registry mechanics, publication surface, and canonical list location.

`FormBuilder` owns the semantic definitions for reusable form-engine capabilities such as `forms`, `form_drafts`, and `form_actions`, but those capabilities must still be published through the shared registry model rather than invented ad hoc inside individual product modules.

Product modules may:

- consume registered capabilities
- require registered capabilities
- optionally use registered capabilities

Product modules may not:

- invent new shared capability names silently
- treat private product flags as if they were shared platform capabilities

---

## 2. Naming Rules

Capability names should be neutral, reusable, and product-agnostic.

Use names such as:

- `files`
- `audit`
- `rbac`
- `settings`
- `webhooks`
- `connectors`
- `forms`

Avoid names such as:

- `familyvault-memorial-flow`
- `client-portal-domains`
- any capability name tied to a single module's business domain

If a capability is product-local, it should stay in the module's own settings or manifest metadata instead of entering the shared capability registry.

---

## 3. Capability Types

### 3.1 Core platform capabilities

Owned by `GeneralApp`, for example:

- `users`
- `sessions`
- `files`
- `audit`
- `rbac`
- `settings`
- `webhooks`
- `connectors`

### 3.2 Form-engine capabilities

Semantically owned by `FormBuilder`, and published through the shared registry, for example:

- `forms`
- `form_drafts`
- `form_submissions`
- `form_actions`
- `field_plugins`

### 3.3 Module-local flags

These are not shared capabilities.

If a module needs a purely local toggle or product-specific behavior flag, it should keep that in:

- module settings
- module manifest metadata
- module-owned adapter configuration

not in the shared capability registry.

---

## 4. How Manifests Use Capabilities

Manifests should split capabilities into:

- `required`
- `optional`

`required` means the module cannot operate correctly without that capability.

`optional` means the module can use the capability if present and must capability-check before invoking it.

---

## 5. Extending the Registry

A new capability should be added only when:

- it is reusable across multiple current or future modules
- it represents a real shared integration surface
- it is not just a product-local business feature

Adding a new capability should include:

- capability name
- owning foundation
- short definition
- whether it is core or optional
- related extension interfaces, if any

---

## 6. Interface Relationship

Capabilities declare that a broad shared surface exists.

They do **not** replace interface versioning.

For example:

- capability: `forms`
- interface contract: `form_action_contract: 1.x`

Modules should declare both capability requirements and interface requirements where applicable.
