# FOUNDATION COMPREHENSIVE WIRING PLAN

What the foundation should absorb from `BUILD-PLAN/`, and how the promoted pieces should interconnect.

## 1. Canonical promoted domains

The shared foundations should absorb the following comprehensive domains into `GeneralApp`:

1. owner/admin control model
2. user management
3. RBAC and capability management
4. settings hierarchy
5. owner-configurable UI and element configuration
6. notifications
7. payments/billing
8. MCP/integrations/capability adapters
9. audit/error logs/incidents/alerts

These come primarily from:

- `admin.md`
- `ADMIN_CAPABILITIES.md`
- `OWNER_CONTROL_CONTRACT.md`
- `comprehensive-user-management-flow.md`
- `comprehensive-rbac-roles-capabilities-flow.md`
- `comprehensive-roles-capabilities-rbac-flow.md`
- `rbac-api-endpoints.md`
- `rbac-complete-integration-guide.md`
- `rbac-implementation-examples.md`
- `rbac-implementation-part2.md`
- `comprehensive-settings-management-flow.md`
- `comprehensive-element-configuration-flow.md`
- `comprehensive-notification-flow.md`
- `comprehensive-payment-system-flow.md`
- `comprehensive-mcp-integrations-flow.md`
- `comprehensive-error-logs-flow.md`

## 2. What should become canonical foundation surfaces

### 2.1 Foundation admin shell

The platform should expose a real owner/admin shell, not scattered settings pages.

Canonical surfaces:

- Owner Control Center
- User Management
- RBAC + Capabilities
- Settings + Branding
- Element Configuration
- Notifications
- Integrations/MCP
- Billing + Plans
- Audit + Errors + Incidents

### 2.2 Registries

The foundation should make the following registries explicit:

- capability registry
- interface registry
- settings registry
- owner-config registry
- UI element registry
- notification channel/template registry
- payment provider registry
- integration provider registry
- audit event registry

### 2.3 Runtime services

The foundation runtime should own:

- identity/user lifecycle service
- policy evaluation service
- settings resolution service
- owner-config resolution service
- UI configuration resolution service
- notification orchestration service
- payment orchestration service
- integration/capability dispatch service
- observability/audit service

## 3. Interconnection model

These domains should not be built as isolated admin pages.

They must interconnect through shared runtime authority.

### 3.1 Owner-config chain

Every important configurable element should follow:

`element -> registry key -> owner UI -> persisted config -> runtime resolver -> enforcement/rendering`

This must apply to:

- branding
- sidebar/header labels
- nav visibility
- role access
- capability bindings
- feature flags
- notification behavior
- integration selection
- billing defaults
- audit retention and alerting policies

### 3.2 RBAC + capabilities + UI

RBAC must drive more than API allow/deny checks.

It must connect to:

- route visibility
- nav visibility
- page section visibility
- component visibility/state
- action buttons
- settings tabs
- data access
- admin inspection tools

Capabilities must connect to:

- provider adapters
- module registrations
- settings dependencies
- UI affordances
- audit events

### 3.3 Settings + owner config

Settings must be authoritative, not decorative.

A changed setting should propagate into:

- runtime behavior
- API behavior
- UI rendering
- module availability
- provider selection
- audit trails

### 3.4 Notifications + audit + users

Notifications should interconnect with:

- user preferences
- org/platform settings
- RBAC/capability checks
- audit events
- provider adapters
- admin templates/rules

### 3.5 Payments + org/plans + RBAC

Payments should interconnect with:

- organization plans
- feature/capability entitlements
- billing settings
- invoices/subscriptions
- audit trails
- admin overrides

### 3.6 Integrations/MCP + capabilities

Integrations should never be hardwired straight into business logic.

They should route through:

- capability bindings
- provider adapters
- settings hierarchy
- audit logging
- owner/admin control surfaces

## 4. Suggested package and app ownership

### `packages/foundation-types`

- canonical types for users, roles, permissions, settings, UI config, notifications, payments, integrations, and audit events

### `packages/foundation-core`

- runtime resolvers
- orchestration services
- policy evaluation
- settings resolution
- capability dispatch
- owner-config resolution

### `packages/foundation-sdk`

- client/server APIs for:
  - users
  - RBAC
  - settings
  - owner config
  - notifications
  - payments
  - integrations
  - audit/errors

### `packages/foundation-ui`

- admin UI primitives
- owner-control forms
- registry-driven navigation
- configurable element renderers
- inspection/debug components

### `apps/platform-api`

- authoritative backend APIs
- policy-enforced endpoints
- admin inspection endpoints
- provider-adapter endpoints

### `apps/admin-shell`

- owner/admin control center
- all comprehensive management surfaces
- inspection/debug flows
- bulk actions

### `supabase/`

- shared tables for:
  - users
  - sessions
  - roles
  - permissions
  - settings
  - UI config
  - notification rules/templates
  - payment state
  - integrations/capability bindings
  - audit events
  - error logs/incidents/alerts

## 5. What should not be promoted wholesale

Do not copy these into the base foundation as mandatory runtime scope:

- mindmap systems
- kanban systems
- industry profiles
- training-data generation pipelines
- project-profiler/recommendation engine as required core runtime

Those should stay:

- module-specific
- optional capability modules
- or reference corpora

## 6. Build order

### Phase 1: control authority

- user management
- RBAC + capabilities
- settings hierarchy
- owner-config chain

### Phase 2: operational foundation

- notifications
- integrations/MCP
- audit/error logs/incidents

### Phase 3: commercial/entitlement layer

- payments/billing
- plan entitlements
- capability-plan linkage

### Phase 4: optional intelligence

- profiler/recommendation engine
- AI agent orchestration
- scouts/ops intelligence

## 7. Bottom line

The right extraction from `BUILD-PLAN/` is not “copy everything into the foundation.”

The right extraction is:

- promote the universal wiring
- promote the authoritative owner/admin control surfaces
- promote the registries and adapters
- promote the enforcement and audit model
- keep product-specific systems and meta/planning corpora outside the shared runtime
