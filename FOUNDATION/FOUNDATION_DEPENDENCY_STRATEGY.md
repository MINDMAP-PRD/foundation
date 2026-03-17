# FOUNDATION DEPENDENCY STRATEGY

Build Faster With Proven Libraries While Keeping the Foundation Swappable

> **Purpose:** This document defines the implementation strategy for using third-party libraries and hosted services to accelerate delivery without hardwiring the foundation to any single vendor. The foundation contracts stay ours. Providers stay replaceable.

---

## 1. Core Rule

The foundation should reuse proven infrastructure wherever possible.

Do **not** rebuild commodity infrastructure from scratch if a stable library or service already solves it well.

Examples:

- identity and session infrastructure
- policy evaluation
- storage
- search
- jobs and workflows
- email delivery
- payments
- CAPTCHA
- form-canvas interaction
- validation engines
- rich-text editing

But reuse must follow one non-negotiable rule:

- modules consume **foundation-owned contracts**
- modules do **not** consume vendor SDKs directly for foundation-owned capabilities

The same acceleration principle applies to product modules:

- modules should also use proven frameworks and libraries for module-local concerns
- but they must keep those choices behind module-owned boundaries
- and they must not turn module framework choices into new shared foundation truth

---

## 2. Non-Negotiable Architecture Rules

### 2.1 The canonical contract stays ours

The shared source of truth remains:

- foundation docs in `foundations/`
- foundation types and packages in `packages/foundation-*`
- manifest, registry, interface, and extension contracts owned by the foundation

### 2.2 Providers sit behind adapters

Every external dependency used for a shared platform concern must sit behind one of:

- a foundation package
- a foundation service boundary
- a capability binding
- a connector/provider adapter

Examples:

- `@foundation/auth`
- `@foundation/policy`
- `@foundation/storage`
- `@foundation/payment`
- `@foundation/email`
- `@foundation/search`
- `@foundation/jobs`
- `@foundation/captcha`

These may start life inside `foundation-core` or `platform-api`, but the design must remain adapter-based.

### 2.3 Paid products may accelerate delivery, but may not become the core contract

Paid or hosted services are allowed when they speed up delivery.

They must **not** become:

- the canonical identity model
- the canonical RBAC model
- the canonical module contract
- the canonical manifest format
- the canonical registry vocabulary

If a provider is replaced, module contracts should stay stable.

### 2.4 Vendor-specific details must stay isolated

Keep vendor-specific items out of module-facing contracts whenever possible:

- vendor SDK calls
- vendor object IDs as shared canonical IDs
- provider-specific payload shapes
- provider-specific role models
- provider-specific UI assumptions

Store provider-specific configuration in:

- connector bindings
- capability bindings
- adapter config
- tenant/provider settings

The same isolation rule applies to module-local frameworks:

- module UI libraries
- module editor engines
- module workflow engines
- module graph or media tooling

Those may be used freely, but they should remain module implementation details unless deliberately promoted into the foundation.

---

## 3. Recommended Default Stack

These are the recommended defaults for moving quickly.

They are defaults, not hard locks.

### 3.1 Platform runtime and data

| Domain | Recommended Default | Acceptable Alternatives | Foundation Rule |
|--------|---------------------|-------------------------|-----------------|
| Database | Postgres via Supabase | self-hosted Postgres | canonical schemas stay ours |
| Query layer | Kysely | Prisma, Drizzle, direct SQL | keep shared entity contracts unchanged |
| API runtime | Hono | Fastify | modules call foundation APIs, not framework internals |

### 3.2 Identity, sessions, and policy

| Domain | Recommended Default | Acceptable Alternatives | Foundation Rule |
|--------|---------------------|-------------------------|-----------------|
| Identity / auth | Supabase Auth | Better Auth, Auth.js, custom OIDC | use a foundation auth adapter |
| Session management | foundation session contract backed by provider/session store | app-owned session layer | session schema stays canonical |
| Authorization / RBAC | Casbin behind `@foundation/policy` | Permit.io provider, table-backed local engine | policy checks remain foundation-owned |

### 3.3 Shared platform capabilities

| Domain | Recommended Default | Acceptable Alternatives | Foundation Rule |
|--------|---------------------|-------------------------|-----------------|
| File storage | Supabase Storage | S3, R2, GCS, Azure Blob | use storage capability/adapters |
| Email delivery | Resend + React Email | SMTP, Postmark, SendGrid | templates and notification contracts stay ours |
| Payments | Stripe | other payment providers via adapters | payment capability contracts stay ours |
| CAPTCHA | Cloudflare Turnstile | hCaptcha | integrate through `captcha.verify` capability |
| Search | Meilisearch | Typesense, Postgres FTS | search contracts stay ours |
| Jobs / workflows | Trigger.dev | BullMQ, server-side queues | job contracts stay ours |
| Webhook delivery | foundation webhook engine | Svix as accelerator | webhook contracts and event keys stay ours |

### 3.4 FormBuilder implementation defaults

| Domain | Recommended Default | Acceptable Alternatives | Foundation Rule |
|--------|---------------------|-------------------------|-----------------|
| Drag/drop canvas | dnd-kit | custom drag system | form layout schema stays ours |
| Form state | TanStack Form | React Hook Form, app-owned state | form definition stays canonical |
| Validation | Zod + Ajv | Yup + JSON Schema validator | validation contracts stay ours |
| Rich text | Tiptap | Lexical, raw ProseMirror | rich-text payload contract stays ours |
| Real-time collaboration | Yjs + Hocuspocus | app-owned collaboration layer | collaboration is optional and swappable |

### 3.5 Optional accelerators, not foundation truth

These may speed delivery, but must be isolated behind adapters and imports that the foundation owns:

- Permit.io for hosted authorization administration
- SurveyJS for form rendering/builder acceleration
- Form.io for full form-platform acceleration
- JSON Forms or `react-jsonschema-form` for schema-driven form surfaces

If used:

- preserve the foundation manifest, registry, and capability model
- preserve the foundation form-definition and submission contracts
- do not let product modules depend directly on those vendors

### 3.6 Module implementation defaults

Product modules should also prefer proven frameworks instead of rebuilding generic product-layer mechanics from scratch.

Good candidates for module-local reuse:

- graph and visualization libraries
- media players and upload helpers
- collaborative editors
- rich document viewers
- search UI packages
- workflow/state libraries
- charting libraries
- accessibility helpers

Module rule:

- use the best-fit framework for the module
- keep the choice behind the module boundary
- do not leak it into shared foundation contracts unless promotion is explicit

---

## 4. Capability-Driven Integration Rule

When a shared concern can vary by provider, prefer capability-driven integration over hardcoded product branches.

Examples:

- `storage.upload`
- `storage.sign_url`
- `captcha.verify`
- `payment.create_intent`
- `payment.create_checkout`
- `payment.verify_webhook`
- `email.send`
- `search.index`
- `search.query`
- `policy.check`
- `policy.assign_role`

If a capability is optional:

- the runtime must detect whether it is registered
- the UI must adapt safely
- the platform must fail safely when the capability is unavailable

Capability names must stay aligned with the canonical registry in `CAPABILITY_REGISTRY_SPEC.md`.

---

## 5. Build-Fast Guidance

To move quickly without wasting time:

1. choose a strong default provider in each shared domain
2. wire it through a foundation adapter from day one
3. keep vendor-specific code inside the adapter or connector layer
4. keep module contracts, manifests, and shared schemas vendor-neutral
5. only build custom infrastructure where the foundation itself is the differentiator

Good candidates to build ourselves:

- module manifest system
- capability registry
- interface registry
- runtime registry
- extension/plugin loading model
- foundation bridge contracts
- shared entity contracts

Good candidates to borrow:

- auth infrastructure
- email delivery
- payments
- storage
- search
- jobs
- rich-text editing
- drag/drop canvas primitives
- module-local UX and implementation frameworks where they reduce delivery time without contaminating shared contracts

---

## 6. What Modules Must Never Do

Modules must not:

- call provider SDKs directly for foundation-owned concerns when a foundation adapter exists
- invent parallel auth/session/policy systems
- redefine provider-specific schemas as module-owned truth
- encode paid vendor assumptions into manifests or shared contracts

Modules should:

- call the foundation SDK
- rely on foundation capability names
- register their own module-specific permissions, routes, handlers, and tables
- use proven frameworks for module-local concerns when that speeds delivery and does not violate shared ownership boundaries

---

## 7. Bottom Line

The foundation should build faster by standing on proven libraries and services.

The foundation should stay portable by making those dependencies:

- adapter-based
- capability-driven where appropriate
- isolated from module-facing contracts

Use the best available tools.
Do not let them own the architecture.
