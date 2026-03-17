# Audit Decision Matrix

Use this matrix when classifying target-file declarations.

## 1. Already owned by a foundation?

- Yes, and the target only references it accurately:
  - classify `correct-foundation-reference`
  - action: `correctly references foundation`
- Yes, but the target changes semantics and also adds extra capabilities:
  - surface the semantic change first as `conflicts-with-foundation`
  - then split any remaining additions into reusable vs product-local only after the conflict is stated
- Yes, same meaning and same contract:
  - classify `duplicate-of-foundation`
  - action: `reference foundation instead`
- Yes, same core concept, but the target adds extra capabilities:
  - split the concept
  - foundation-owned portion: `duplicate-of-foundation`
  - reusable missing capability: `promote-to-foundation-candidate`
  - product-only addition: `valid-product-extension`
- Yes, but changed meaning or shape:
  - classify `conflicts-with-foundation`
  - action: `needs architecture decision`
- Yes, but the target invents a shared capability name or shared extension-table usage outside the canonical registries:
  - classify `conflicts-with-foundation`
  - action: `needs architecture decision`

## 2. Not owned by a foundation?

- Product-only and tied to one vertical:
  - classify `valid-product-extension`
  - action: `keep as product-specific`
- Already supported by existing extension hooks:
  - classify `valid-product-extension`
  - action: `keep as product-specific`
- Generic across future products:
  - classify `promote-to-foundation-candidate`
  - action: `promote to GeneralApp` or `promote to FormBuilder`
- Generic, but the correct foundation target is unclear:
  - classify `promote-to-foundation-candidate`
  - action: `needs architecture decision`

## 3. Promotion Routing

Promote to `GeneralApp` when the declaration is about:

- shared entities or tables
- auth, audit, files, security, tenancy, connectors
- cross-module contracts

Promote to `FormBuilder` when the declaration is about:

- field types
- submission/runtime behavior
- form bridge hooks
- form rendering, validation, or export contracts

If the concept spans both foundations or does not clearly belong to either:

- do not force a destination
- use `needs architecture decision`

## 3.1 Split-Decision Examples

- A product file redefines a shared table but adds two new cross-product-safe fields:
  - existing shared fields -> `duplicate-of-foundation`
  - the two reusable fields -> `promote-to-foundation-candidate`
- A product file redefines a shared table and adds one vendor-specific field:
  - existing shared fields -> `duplicate-of-foundation`
  - vendor-specific field -> `valid-product-extension`
- A product file redefines a shared concept and changes validation, authority, or semantics:
  - `conflicts-with-foundation`
- A product file changes shared semantics and also adds reusable features:
  - `conflicts-with-foundation` first
  - then evaluate any remaining additions separately
- A product file uses a renamed label for an existing foundation concept:
  - normalize alias first
  - then classify as `correct-foundation-reference`, `duplicate-of-foundation`, or `conflicts-with-foundation`

## 3.2 Promotion Dependency Check

Before recommending promotion, also verify whether the concept requires updates to:

- RBAC keys
- audit/event catalogs
- capability registry
- extension-table registry
- manifest/interface contracts
- lifecycle/state rules
- security requirements
- bridge contracts
- cross-document quick links

If yes, include those dependencies in the recommendation instead of promoting the concept in isolation.

## 4. Severity Hints

- `Critical`: breaks canonical ownership, security, or schema authority
- `High`: creates active implementation ambiguity across docs
- `Medium`: causes reference drift or duplicate documentation risk
- `Low`: weakens genericity or introduces avoidable hardcoding

## 5. Final Recommendation Style

Prefer short, decisive recommendations:

- "Remove and reference foundation"
- "Keep local; product-specific"
- "Promote into GeneralApp canonical schema section"
- "Promote into FormBuilder extension contract"
- "Resolve foundation conflict before adding product doc"
