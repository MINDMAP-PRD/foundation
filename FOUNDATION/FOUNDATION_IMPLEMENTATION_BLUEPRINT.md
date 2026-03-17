# FOUNDATION IMPLEMENTATION BLUEPRINT

Where the Actual Foundation Code Lives and How It Should Be Structured

> **Purpose:** This document turns the foundation architecture into a practical repo layout. It explains where the real foundation runtime should be built, where shared packages belong, where modules belong, and how the same model works in both single-repo and multi-repo setups.
>
> **Dependency strategy:** For the recommended library and provider stack, plus the rules for keeping vendors swappable, read [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md).

---

## 1. Core Rule

The `foundations/` folder is the documentation and contract layer.

It is **not** where the actual runtime implementation should be built.

The actual foundation should be built as:

- shared backend/runtime apps
- shared packages/libraries
- shared database/storage/auth support
- separate module folders or repos that consume those shared pieces

---

## 2. Single-Repo Blueprint

Use this when the foundations are still evolving or when you want one coordinated workspace.

```text
repo/
|-- foundations/
|   |-- README.md
|   |-- GeneralApp_Build_Documentation.md
|   |-- FormBuilder_Build_Documentation_v6.md
|   |-- FOUNDATION_MODULE_RULES.md
|   |-- FOUNDATION_LIBRARY_ARCHITECTURE.md
|   |-- FOUNDATION_IMPLEMENTATION_BLUEPRINT.md
|   |-- FOUNDATION_DEPENDENCY_STRATEGY.md
|   |-- MODULE_MANIFEST_SPEC.md
|   |-- CAPABILITY_REGISTRY_SPEC.md
|   |-- INTERFACE_REGISTRY_SPEC.md
|   |-- EXTENSION_TABLE_REGISTRY_SPEC.md
|   `-- FOUNDATION_CHANGE_POLICY.md
|
|-- skills/
|   |-- SKILL_INTERCONNECTION_MAP.md
|   |-- foundation-extension-auditor/
|   `-- foundation-module-factory/
|
|-- apps/
|   |-- platform-api/
|   |   |-- src/
|   |   |-- package.json
|   |   `-- README.md
|   |-- admin-shell/
|   |   |-- src/
|   |   |-- package.json
|   |   `-- README.md
|   `-- module-host/
|       |-- src/
|       |-- package.json
|       `-- README.md
|
|-- packages/
|   |-- foundation-types/
|   |   |-- src/
|   |   `-- package.json
|   |-- foundation-sdk/
|   |   |-- src/
|   |   `-- package.json
|   |-- foundation-core/
|   |   |-- src/
|   |   `-- package.json
|   |-- foundation-ui/
|   |   |-- src/
|   |   `-- package.json
|   |-- form-builder-core/
|   |   |-- src/
|   |   `-- package.json
|   |-- form-builder-ui/
|   |   |-- src/
|   |   `-- package.json
|   `-- module-manifest/
|       |-- src/
|       `-- package.json
|
|-- modules/
|   |-- client-portal/
|   |   |-- module.manifest.json
|   |   |-- docs/
|   |   |-- db/
|   |   |-- backend/
|   |   |-- frontend/
|   |   `-- package.json
|   `-- another-module/
|       |-- module.manifest.json
|       |-- docs/
|       |-- db/
|       |-- backend/
|       |-- frontend/
|       `-- package.json
|
|-- supabase/
|   |-- migrations/
|   |-- functions/
|   `-- seeds/
|
|-- package.json
`-- workspace config
```

---

## 3. What Each Top-Level Folder Does

### `foundations/`

Owns:

- contracts
- governance
- architecture rules
- manifest and registry specs

Does not own:

- runtime implementation code
- module-specific business code

### `apps/`

Owns the real foundation runtime entry points.

Recommended app roles:

- `platform-api/`
  - shared backend runtime for `GeneralApp`
  - auth/session orchestration
  - shared API/webhook layer
  - shared RBAC/audit/settings/connectors logic
- `admin-shell/`
  - optional shared admin host shell
  - module-mounted admin surfaces
- `module-host/`
  - optional module composition host for frontend/runtime loading

### `packages/`

Owns reusable code libraries.

Recommended packages:

- `foundation-types/`
  - shared types and schemas
- `foundation-sdk/`
  - typed client for calling shared foundation services
- `foundation-core/`
  - reusable platform logic shared by apps/modules
- `foundation-ui/`
  - shared UI utilities, tokens, wrappers, and `shadcn/ui` baseline helpers
- `form-builder-core/`
  - reusable FormBuilder engine/runtime
- `form-builder-ui/`
  - reusable FormBuilder UI layer
- `module-manifest/`
  - manifest validation, loaders, registrars, and capability checks

Provider adapter packages may begin inside `foundation-core/` and later be extracted into dedicated packages such as:

- `foundation-auth/`
- `foundation-policy/`
- `foundation-storage/`
- `foundation-email/`
- `foundation-payment/`
- `foundation-search/`
- `foundation-jobs/`
- `foundation-captcha/`

The exact packaging may evolve. The adapter boundary should not.

### `modules/`

Owns product-specific work.

Each module should contain:

- `module.manifest.json`
- module docs/specs
- module DB/migrations
- module backend code
- module frontend code
- module-local settings, events, routes, and integrations

Modules should consume the foundation, not copy it.

### `supabase/`

Owns the shared platform backing services when Supabase is used.

Typical responsibilities:

- Postgres migrations
- auth/storage integration support
- edge/server functions where appropriate
- shared seed/bootstrap data

---

## 4. What Gets Built Where

### Build the foundation here

If the user says "build the foundation," the implementation should primarily land in:

- `apps/platform-api/`
- `packages/foundation-*`
- `packages/form-builder-*`
- `supabase/`

The foundation should **not** be built inside:

- `foundations/`
- `modules/<module-name>/`

### Build modules here

If the user says "build a module," the implementation should land in:

- `modules/<module-name>/`

and consume:

- `packages/foundation-sdk/`
- `packages/foundation-types/`
- `packages/foundation-core/`
- `packages/foundation-ui/`
- `packages/form-builder-*` when needed
- shared platform services exposed by `apps/platform-api/`

---

## 5. Recommended Build Stack

To build faster without starting from scratch, prefer a strong default stack behind foundation-owned adapters.

Recommended defaults:

- Supabase for Postgres, Auth, Storage, Realtime, and Functions where appropriate
- Kysely for type-safe SQL access
- Hono or Fastify for foundation APIs
- Casbin behind a foundation policy adapter
- `shadcn/ui` as the shared UI baseline
- Resend + React Email for outbound email
- Stripe behind foundation payment adapters
- Cloudflare Turnstile behind the shared CAPTCHA capability
- Trigger.dev or BullMQ for jobs and workflows
- Meilisearch for shared search
- dnd-kit, TanStack Form, Zod, Ajv, and Tiptap for FormBuilder implementation
- optional Yjs + Hocuspocus for collaborative form editing

Optional accelerators:

- Permit.io as a hosted policy provider behind the foundation policy adapter
- SurveyJS, Form.io, JSON Forms, or `react-jsonschema-form` as accelerators behind foundation-owned form contracts and adapters

Do not make any paid or hosted provider the canonical truth of the architecture. The foundation contract must remain portable.

---

## 6. Single-Repo Build Rule

Even in a single repo:

- each module builds only itself
- the shared foundation is built once
- the repo is shared organization, not one merged application build

That means:

- module builds target `modules/<module-name>/`
- shared package builds target `packages/<package-name>/`
- shared platform builds target `apps/platform-api/` and related apps

---

## 7. Multi-Repo Blueprint

Use this when the shared contracts are stable enough to publish through npm and shared hosting.

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

In this model:

- `GeneralApp` shared packages are published through npm
- `FormBuilder` shared packages are published through npm
- shared platform services are hosted once
- modules stay in their own repos
- modules install the shared packages and call the hosted services

The multi-repo model still follows the same ownership rules.

---

## 8. How Modules Attach To the Foundation

Modules should attach through:

- package imports
- API/service calls
- manifest registration
- capability and interface registration
- module-scoped DB tables and migrations

Modules should not attach by:

- copying foundation code into the module
- redefining shared schemas locally
- patching shared foundation behavior during ordinary module work

---

## 9. UI Structure Rule

For UI-bearing foundation or module work:

- `foundation-ui/` should hold the shared UI baseline
- the default design system is `shadcn/ui`
- module frontend code should consume that shared baseline
- custom UI components should only exist when no suitable `shadcn/ui` primitive or composition path exists

That keeps admin and user surfaces aligned across modules without forcing every module to reinvent its own component system.

---

## 10. Recommended Starting Point

While the foundations are still being designed:

1. keep everything in a single repo
2. keep contracts/governance in `foundations/`
3. build actual shared runtime code in `apps/`, `packages/`, and `supabase/`
4. build each product in `modules/<module-name>/`

Later, once the shared contracts are stable:

1. publish shared packages through npm
2. split modules into their own repos if desired
3. keep the ownership and manifest model unchanged

---

## 11. Bottom Line

If the user says "build the foundation":

- build the real implementation in `apps/`, `packages/`, and `supabase/`
- keep `foundations/` as the canonical contract layer

If the user says "build a module":

- build it in `modules/<module-name>/`
- make it consume the shared foundation instead of copying it

One shared foundation. Many isolated modules.
