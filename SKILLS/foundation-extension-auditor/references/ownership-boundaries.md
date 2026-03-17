# Ownership Boundaries

Use this map to decide whether the target file is extending the foundations or redefining them.

## GeneralApp Owns

- shared platform entities and tables
- users, sessions, files, audit, tenancy, settings
- shared RBAC tables and the cross-platform canonical permission registry
- connectors, capability bindings, webhook infrastructure
- shared capability registry mechanics and publication surface
- shared security invariants and browser/server safety rules
- cross-module bridge contracts owned at the platform layer
- canonical cross-platform event and audit namespace registration across the platform
- canonical extension-table approval registry

## FormBuilder Owns

- form definitions, sections, fields, submissions, drafts
- reusable field behavior and validation rules
- render/runtime lifecycle for forms
- after-submit actions
- the semantic definition of form-specific RBAC keys and form-specific events within the `fb.*` namespace
- the semantic definition of form-engine capabilities published through the shared capability registry
- reusable form theming, embeds, analytics, exports
- generic extension hooks for product integrations

## Product Files Own

- product-specific entities
- product workflows and lifecycle states
- adapter mappings into shared bridge contracts
- vendor-specific or vertical-specific orchestration
- product-local behavior and examples that consume their namespace without becoming the canonical registry

## Namespace Rule

Product files may describe their own permissions, events, and audit actions for local behavior, but they should not become the canonical registry for those namespaces.

- Product docs may define intended product-local keys and events.
- GeneralApp remains the canonical place where shared registries, cross-system adoption, and authoritative platform-wide catalogs are consolidated.
- Module foundations like FormBuilder may still be the semantic source of truth for their own module namespace details; GeneralApp should consolidate and adopt those keys rather than silently redefining them.
- If a product introduces a reusable or cross-system permission/event pattern, the auditor should flag it as a promotion candidate or architecture decision rather than accepting a product-local canonical definition.

## Red Flags

- a product file declares a table already canonically defined in a foundation
- a product file restates a foundation schema with different fields
- a product file introduces a new name for an existing shared primitive
- a product file invents new shared capability names outside the shared capability registry
- a product file assumes a shared extension table that is not approved in the extension-table registry
- a foundation doc starts documenting product-only workflow logic
- a supposedly generic spec hardcodes one product's domain model

## Safe Extensions

- product-local `productConfig` payloads behind documented extension hooks
- bridge handler implementations for one product
- product-specific entities that consume shared primitives without redefining them
- vendor adapters behind capability abstractions
- correctly scoped product-local permissions/events that are explicitly non-canonical until promoted
