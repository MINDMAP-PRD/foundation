---
name: foundation-extension-auditor
description: Use when a new documentation file, module spec, or product-layer build plan is meant to build on top of the foundation documents in `foundations/`, especially `GeneralApp_Build_Documentation.md`, and optionally `FormBuilder_Build_Documentation_v6.md` when reusable form-engine capability is needed. Audit the new file against the foundation and governance docs, prevent duplicate or conflicting declarations, and identify which new concepts should stay product-specific versus be promoted into the foundations.
---

# Foundation Extension Auditor

This skill protects the foundation and governance docs from silent drift while still allowing them to expand through new product files.

Use this skill before creating, editing, or auditing any new spec that builds on:

- `../../foundations/GeneralApp_Build_Documentation.md`
- `../../foundations/FormBuilder_Build_Documentation_v6.md` when the target module uses reusable form-engine capability

## Goal

Classify every declaration in the target file into one of five buckets:

- `duplicate-of-foundation`
- `conflicts-with-foundation`
- `valid-product-extension`
- `promote-to-foundation-candidate`
- `correct-foundation-reference`

The skill assumes the foundations are runtime-extensible without core code changes, but still need a stable ownership boundary.

It also acts as a stop signal when the target idea or spec is too incomplete, contradictory, or logically weak to build safely without further discussion.

## Read First

Before auditing the target file, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. [`references/ownership-boundaries.md`](references/ownership-boundaries.md)
3. [`references/audit-decision-matrix.md`](references/audit-decision-matrix.md)

Then read the current governance docs in the repo root:

1. `../../foundations/README.md`
2. `../../foundations/FOUNDATION_MODULE_RULES.md`
3. `../../foundations/FOUNDATION_LIBRARY_ARCHITECTURE.md`
4. `../../foundations/FOUNDATION_DEPENDENCY_STRATEGY.md`
5. `../../foundations/MODULE_MANIFEST_SPEC.md`
6. `../../foundations/CAPABILITY_REGISTRY_SPEC.md`
7. `../../foundations/INTERFACE_REGISTRY_SPEC.md`
8. `../../foundations/EXTENSION_TABLE_REGISTRY_SPEC.md`
9. `../../foundations/FOUNDATION_CHANGE_POLICY.md`

Then read the current foundation docs in the repo root:

1. `../../foundations/GeneralApp_Build_Documentation.md`
2. `../../foundations/FormBuilder_Build_Documentation_v6.md` only if the target module uses or proposes reusable form-engine capability

Read the sections needed for the target file's declared scope, plus any adjacent foundation sections required to verify aliases, ownership, RBAC, events/audit, capabilities, extension-table usage, security, bridge contracts, lifecycle rules, and cross-document references.

## Default Workflow

1. Identify the target file's claimed role and whether it depends on `GeneralApp` only or on `GeneralApp` plus `FormBuilder`.
2. Build a compact contract map from the governance and foundation docs:
   - canonical entities and tables
   - shared APIs and registries
   - RBAC keys
   - event and audit namespaces
   - capability names and ownership
   - interface names and version lines
   - approved extension-table surfaces
   - bridge and extension points
   - module manifest and interface requirements
   - security and lifecycle invariants
3. Build the same map for the target file.
4. Normalize aliases before comparison:
   - detect renamed-but-equivalent entities, fields, tables, namespaces, and lifecycle states
   - compare semantics, not just exact names
5. Compare the target declarations against the foundation map.
6. For every promotion candidate, run a dependency sweep:
   - RBAC keys and seeded permissions
   - audit/event namespaces
   - capability registry entries
   - extension-table approvals
   - security rules and invariants
   - bridge hooks and extension points
   - interface version expectations
   - lifecycle/state model impacts
   - cross-document references that must move with the promoted concept
7. Produce findings under the required output shape below.

## Audit Rules

- If the target file redefines a foundation-owned schema, table, runtime contract, or namespace, flag it as `duplicate-of-foundation`.
- If the target file correctly references a foundation-owned concept without redefining it, classify it as `correct-foundation-reference` and do not escalate it as a finding unless the reference is stale, partial, or misleading.
- If the target file both changes foundation semantics and adds new capabilities, treat the semantic change as primary:
  - classify the changed portion as `conflicts-with-foundation`
  - only split out additional reusable or product-local capabilities after the conflict is surfaced
  - do not let a split-decision downgrade a semantic conflict
- If the target file restates a foundation-owned concept but also adds capabilities the foundations do not yet have, split the judgment:
  - classify the already-owned portion as `duplicate-of-foundation`
  - classify any new reusable capability as `promote-to-foundation-candidate`
  - classify any remaining product-only addition as `valid-product-extension`
  - do not let a richer restatement silently replace the foundation's canonical ownership
- If the target file changes the meaning, field shape, authority, lifecycle, or security rule of a foundation-owned concept, flag it as `conflicts-with-foundation`.
- If the target file introduces product-local entities, workflows, mappings, or adapter behavior that do not fork the foundations, classify them as `valid-product-extension`.
- If the target file introduces a capability that is already supported by existing extension hooks, keep it product-local unless it also requires a new shared contract. In that case:
  - classify the feature usage as `valid-product-extension`
  - classify only the missing shared contract as `promote-to-foundation-candidate`
- If the target file introduces a reusable cross-product primitive that is generic enough for future modules, classify it as `promote-to-foundation-candidate`.
- If the target file invents a shared capability name outside `../../foundations/CAPABILITY_REGISTRY_SPEC.md`, surface that as `conflicts-with-foundation` unless the audit is explicitly recommending registry expansion.
- If the target file claims use of a shared extension table not approved in `../../foundations/EXTENSION_TABLE_REGISTRY_SPEC.md`, surface that as `conflicts-with-foundation`.
- Prefer references to the foundation over local restatement. Repeated recap tables are usually drift risks unless they are explicitly marked non-canonical and stay minimal.
- If the target idea or spec is materially incomplete, internally contradictory, or missing a key architectural decision required for safe implementation, surface that explicitly and treat it as a discussion gate rather than something to be silently guessed through.

## Split-Decision Rule

When a target file defines the same conceptual object as a foundation-owned object but does so more comprehensively, do not classify the entire object with a single label.

Break it into parts:

1. Existing foundation contract
   - `duplicate-of-foundation`
   - action: `reference foundation instead`
2. New generic capability missing from the foundations
   - `promote-to-foundation-candidate`
   - action: `promote to GeneralApp` or `promote to FormBuilder`
3. New product-local capability that only serves the target module
   - `valid-product-extension`
   - action: `keep as product-specific`

Use this split whenever the target file says, in effect, "the foundation already has this concept, but this file adds missing pieces."

Do not use this split to hide semantic conflicts. If the target file changes authority, validation, lifecycle meaning, or security expectations, surface `conflicts-with-foundation` first.

## Alias Normalization Rule

Before classifying a declaration as new, check whether it is a renamed or reworded version of an existing foundation concept.

Alias checks must include:

- table and entity renames
- field renames
- permission/event namespace renames
- state/lifecycle label renames
- bridge/adapter terminology drift
- capability-name drift
- extension-table naming drift

If the semantics match an existing foundation concept, classify it as duplicate, conflict, or split-decision rather than treating it as net-new.

## Promotion Criteria

Promote a target declaration into a foundation only when most of the following are true:

- It is useful across multiple future products.
- It is not tied to one vendor or one business vertical.
- It is structural, contractual, or security-relevant.
- It cannot already be expressed cleanly through existing extension hooks.
- Keeping it product-local would likely cause future duplication.

If the feature can already be implemented cleanly through the current extension hooks, prefer `valid-product-extension` over promotion.

Promotion destination:

- Promote to `GeneralApp` for shared platform primitives.
- Promote to `FormBuilder` for reusable form-engine capabilities.
- If a reusable concept does not fit cleanly into either foundation, do not force promotion. Classify it as `promote-to-foundation-candidate` with action `needs architecture decision`.

## Dependency Sweep Rule

No promotion recommendation is complete until the auditor checks whether the promoted concept also requires coordinated updates to:

- RBAC registries and seeded roles
- audit/action catalogs
- webhook or event namespaces
- security requirements and safety constraints
- bridge contracts and extension APIs
- capability registry entries
- extension-table registry entries
- manifest/interface requirements
- lifecycle states and retention/deletion rules
- canonical quick links and section references

If any dependent contract would also need to change, list that explicitly under `Foundation Update Recommendations`.

## Skill Handoff Rule

If the user wants more than an audit and is asking for:

- a phased build plan
- a folderized module package
- a resumable build package
- inventories, contracts, specs, and progress files

then hand off to `foundation-module-factory` after the audit findings are established.

## Focus Areas

Always scan for:

- canonical ownership drift
- duplicate schemas or entity definitions
- field-level mismatches
- RBAC key mismatches
- event / audit namespace mismatches
- capability-name mismatches
- unapproved extension-table claims
- hardcoded vendor or vertical assumptions in supposedly generic layers
- stale section references to foundation-owned material
- repeated "usage" tables that silently become competing sources of truth

## Required Output

Return findings in this order:

1. `Conflicts`
2. `Duplicates`
3. `Valid Extensions`
4. `Promotion Candidates`
5. `Foundation Update Recommendations`
6. `Correct Foundation References (Optional)`

If the target is not build-ready, add:

7. `Discussion Required Before Build`

Each finding should say one of:

- `remove from target`
- `reference foundation instead`
- `keep as product-specific`
- `promote to GeneralApp`
- `promote to FormBuilder`
- `correctly references foundation`
- `needs architecture decision`
- `brainstorm with user before build`

## Constraints

- Treat the governance docs (`../../foundations/FOUNDATION_MODULE_RULES.md`, `../../foundations/FOUNDATION_LIBRARY_ARCHITECTURE.md`, `../../foundations/FOUNDATION_DEPENDENCY_STRATEGY.md`, `../../foundations/MODULE_MANIFEST_SPEC.md`, `../../foundations/CAPABILITY_REGISTRY_SPEC.md`, `../../foundations/INTERFACE_REGISTRY_SPEC.md`, `../../foundations/EXTENSION_TABLE_REGISTRY_SPEC.md`, and `../../foundations/FOUNDATION_CHANGE_POLICY.md`) plus the foundation docs as the current authority set; if they conflict anywhere, surface that first.
- Do not assume recap appendices are authoritative unless the document says so explicitly.
- Do not promote vendor-specific flows into the foundations just because they are well documented.
- Keep the audit scoped to repository files unless the user explicitly expands scope.
