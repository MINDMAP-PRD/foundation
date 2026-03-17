# GENERAL-PURPOSE APPLICATION

Platform Foundation & Shared Core Contracts

> **Foundation Entry Point:** If an AI or engineer needs the high-level explanation of how the foundations work, how modules attach to them, or how the model applies in single-repo versus multi-repo setups, read [README.md](README.md) first.

> **Document role:** This document is the **canonical source of truth** for all shared platform contracts. Form Builder and all future product modules **MUST** reference these definitions rather than redefine them. This document owns: users, sessions, file storage, audit primitives, MCP connectors, RBAC permissions, and settings hierarchy.
>
> **Architecture Rule:** No product-specific business schemas, field models, or workflow logic (for example client-record business rules, registrar purchase flows, or form-runtime behavior) belong in this document. Shared cross-module registries that the platform owns centrally — such as RBAC action keys, event namespaces, capability bindings, and shared entity contracts — MAY enumerate product namespaces without redefining product behavior.

---


Element Interconnection Mesh • Multi-Tenancy • Sessions • Settings • Help & Support • Onboarding • Search • Feature Flags • Webhooks • API Keys • i18n • Theming • Rate Limiting • Analytics • Accessibility

Version 6.1 (Clean Architecture Refactor) — Platform Foundation Specification  
Part of Unified SaaS Platform: General Application + Form Builder + Product Modules

> **Integration Notice:** This document specifies the General Application foundation module. Cross-system integration points with Form Builder and product modules are marked throughout (see §1.3, §23, §24).
>
> **Foundation Operating Rules:** Before any AI or engineer creates, edits, or audits a product/module spec on top of the foundations, it **MUST first read** [FOUNDATION_MODULE_RULES.md](FOUNDATION_MODULE_RULES.md). That file defines the operating model for shared foundations, module isolation, extension boundaries, and promotion rules.
>
> **Library Architecture:** The foundations are meant to operate like a shared library/framework consumed by product modules. Read [FOUNDATION_LIBRARY_ARCHITECTURE.md](FOUNDATION_LIBRARY_ARCHITECTURE.md), [MODULE_MANIFEST_SPEC.md](MODULE_MANIFEST_SPEC.md), [CAPABILITY_REGISTRY_SPEC.md](CAPABILITY_REGISTRY_SPEC.md), [INTERFACE_REGISTRY_SPEC.md](INTERFACE_REGISTRY_SPEC.md), [EXTENSION_TABLE_REGISTRY_SPEC.md](EXTENSION_TABLE_REGISTRY_SPEC.md), and [FOUNDATION_CHANGE_POLICY.md](FOUNDATION_CHANGE_POLICY.md) before proposing module-level extensions or foundation promotions.
>
> **AI Extension Guardrail:** Before any AI creates, edits, or audits a new module/spec file that builds on this foundation, it **MUST first read** [skills/foundation-extension-auditor/SKILL.md](../skills/foundation-extension-auditor/SKILL.md). Use that skill to decide whether a declaration belongs in GeneralApp, belongs in FormBuilder, should remain product-local, or should be promoted into a foundation instead of being duplicated.
>
> **Module Package Workflow:** If an AI is turning an idea, module concept, or early spec into a comprehensive phased build package folder, it **SHOULD use** [skills/foundation-module-factory/SKILL.md](../skills/foundation-module-factory/SKILL.md) after ownership review so inventories, contracts, specs, phases, and progress tracking remain aligned to the foundations.
>
> **Implementation acceleration rule:** Build shared platform concerns on top of proven libraries and services wherever possible, but keep every provider behind foundation-owned adapters and capability bindings so the architecture remains swappable. The recommended stack and substitution rules are defined in [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md).

> **Role mapping note:** In the single-tenant deployment profile used by the current product docs, platform-wide authority responsibilities map to the reserved `system_owner` assignment used for the initial owner/bootstrap account. Day-to-day workspace administration maps to delegated `super_admin` / `admin` permissions. Any legacy authority wording below refers to capability scope, not to an additional fixed runtime role enum.
>
> **Auth Authority:** The identity and session contract is foundation-owned. Supabase Auth is the default provider for fast delivery, but it must remain behind a swappable foundation auth adapter so the platform can move to another provider later without changing module contracts. No product module shall maintain parallel `password_hash` fields.

---

## TABLE OF CONTENTS

1. Element Interconnection Framework
2. Extension Architecture & Dynamic Registration
3. Multi-Tenancy
4. Session Management
5. Settings System
6. Help & Support System
7. Onboarding & First-Run Experience
8. Global Search
9. Feature Flags
10. Webhooks
11. API Key Management
12. Rate Limiting
13. Analytics & Reporting
14. File & Storage Management
15. Internationalisation & Localisation (i18n)
16. Theming & Branding
17. Notification System
18. Accessibility (A11Y)
19. Logout, Account Security & Security Centre
20. Changelog & In-App Product Updates
21. Maintenance Mode
22. Presence & Real-Time Infrastructure
23. Canonical Entity Schemas (Source of Truth)
    23.1 users Table
    23.2 sessions Table
    23.3 file_uploads Table
    23.4 audit_log Table (with Extension Point)
    23.5 connectors & capability_bindings Tables
    23.6 RBAC Tables (with Extension Point)
    23.7 Webhook Endpoints (with Extension Point)
    23.8 Custom Entity Registry
    23.9 Form-to-Entity Bridge
24. Cross-System Integration Reference

---

## 1. ELEMENT INTERCONNECTION FRAMEWORK

Every element introduced into this system must immediately be evaluated against the Interconnection Mesh. The moment an element exists, it does not exist in isolation — it sends signals, receives signals, fires branches, is governed by rules, and can fail.

### 1.1 The Interconnection Scan Protocol (ISP)

When any new element is introduced — a feature, a UI component, a data entity, or a background job — the following twelve questions are asked and answered before the element is considered specified:

| Scan | Question |
|------|----------|
| **SCAN-01 AUTH** | Does this element require the user to be authenticated? What happens if they are not? |
| **SCAN-02 SESSION** | Does this element depend on an active session? Can it operate in a read-only state? |
| **SCAN-03 TENANT** | Does this element scope its data per tenant? Can it leak cross-tenant data? |
| **SCAN-04 RBAC** | What action string does this element register? Who is allowed to trigger it? |
| **SCAN-05 EMAIL** | Does this element need to send an email? What triggers it? |
| **SCAN-06 MCP** | Does this element depend on any external capability (AI, SMS, payment, storage)? |
| **SCAN-07 AUDIT** | Does this element's state changes need to be logged immutably? |
| **SCAN-08 NOTIF** | Does this element generate a notification? In-app, email, or both? |
| **SCAN-09 RATE-LIMIT** | Is this element a vector for abuse? What rate-limit rule governs it? |
| **SCAN-10 SEARCH** | Are this element's records discoverable via global search? |
| **SCAN-11 SETTINGS** | Does this element expose any configuration to the user, admin, or `system_owner`? |
| **SCAN-12 FAILURE** | What is every error path? What is the empty path? Loading path? |

### 1.2 Interconnection Calling Convention

Instead of re-specifying a branch from scratch every time, we use parameterised templates:

```typescript
// EMAIL_BRANCH
EMAIL_BRANCH({
  trigger: 'user.invite.sent',
  template: 'invite-new-user',
  recipient: 'invitee.email',
  requiredVars: ['inviterName', 'workspaceName', 'acceptUrl', 'expiresIn'],
  fallbackBehaviour: 'queue_retry_3x_then_dead_letter',
  adminCanDisable: true,
})

// RBAC_BRANCH
RBAC_BRANCH({
  action: 'user.invite.send',
  default: 'allow',
  subjects: ['admin', 'editor'],
  conditions: [{ field: 'tenant.plan', op: '!=', value: 'free' }],
})

// AUDIT_BRANCH
AUDIT_BRANCH({
  action: 'user.invite.sent',
  capture: ['actorId', 'targetEmail', 'role', 'expiresAt'],
})

// NOTIF_BRANCH
NOTIF_BRANCH({
  type: 'invite-accepted',
  recipient: 'actor',
  channels: ['in-app', 'email'],
  userCanMute: true,
})
```

### 1.3 Master Interconnection Mesh

The table below lists every core element in this system and its primary interconnections:

| Element | Connects To | Receives From | Failure Mode |
|---------|-------------|---------------|--------------|
| **Auth / Session** | RBAC, Audit, Email (OTP/verify), MCP (SSO), Notif, Settings | — (root element) | Session store down → read from DB replica |
| **Tenant / Workspace** | Auth, RBAC, Billing, MCP, Settings, Audit | Auth, Billing | Tenant deleted → force-logout; data export queued |
| **User Profile** | Auth, Email, Notif, RBAC, Audit, Avatar→Storage | Auth, Tenant | Email provider down → queue change |
| **RBAC Engine** | Every element | All elements | Engine unavailable → fail-closed (deny all) |
| **Email System** | MCP, Queue, Templates, Notif | Every element that sends email | Primary down → fallback; all down → dead-letter |
| **MCP Registry** | Every external capability | Settings, RBAC | Provider down → capability fallback |
| **Session Manager** | Auth, Audit, Notif, RBAC, Tenant | Auth | Session store full → evict oldest idle |
| **Settings** | RBAC, Audit, Notif | All elements | Setting write fails → old value retained |
| **Help & Support** | Email, RBAC, Audit, Notif, MCP | User | Chat provider down → fallback to email |
| **Search** | Auth, Tenant, RBAC, Index | All indexed entities | Index unavailable → DB fallback |
| **Notifications** | Email, WebSocket, Queue, RBAC | All elements | WebSocket down → badge delayed |
| **Audit Log** | RBAC, Search, Export, Tenant | Every state-changing element | Write failure → retry queue |
| **File / Storage** | MCP, RBAC, Audit, Notif, Tenant | Profile, Maps, Attachments | Provider down → upload blocked |
| **Feature Flags** | RBAC, Audit, Settings, Tenant | All features | Flag service down → last cached value |
| **Rate Limiting** | RBAC, Tenant, Redis | Every API endpoint | Redis down → in-memory fallback |
| **Webhooks** | RBAC, Audit, MCP, Queue, Notif | All event emitters | Delivery failure → retry 3× |
| **API Keys** | Auth, RBAC, Audit, Rate Limiting, Tenant | Auth | Compromised key → revoke instantly |
| **Onboarding** | Auth, Tenant, RBAC, Notif, Email | Auth | Onboarding incomplete → reminder email |
| **Analytics** | Auth, Tenant, RBAC, Audit, Search | All elements | Collector down → buffer events |
| **i18n / Locale** | Settings, Tenant, Auth | All UI elements | Missing translation → fallback to en-US |
| **Theming** | Settings, Auth | All UI elements | Theme load fails → fallback to default |
| **Changelog** | Auth, RBAC, Notif, Email | Product updates | No entries → empty state |
| **Feedback Widget** | Auth, Email, RBAC, Audit, Notif | User action | Submission fails → local draft saved |

---

## 2. EXTENSION ARCHITECTURE & DYNAMIC REGISTRATION

> **Core Principle:** The General Application is a **foundation, not a closed system**. All major subsystems support dynamic extension by product modules without schema changes or core code modification.

### 2.1 Extension Philosophy

| Aspect | Design Rule |
|--------|-------------|
| **Schema Extensibility** | All registry tables use `VARCHAR` keys, not enums. New entries require only INSERT, not ALTER. |
| **Runtime Discovery** | Product modules register handlers at startup; core resolves by key lookup. |
| **Namespace Isolation** | Each module uses its prefix (`ga.*`, `fb.*`, `cp.*`, `ext.*`) to avoid collisions. |
| **Backward Compatibility** | Core never removes columns; new features are additive. |
| **Capability Over Configuration** | Prefer capability-based extension over hardcoded conditionals. |

### 2.2 Extension Points Summary

| Subsystem | Extension Mechanism | Registration Pattern | Documentation |
|-----------|---------------------|---------------------|---------------|
| **MCP Capabilities** | `capability_bindings` table | Dynamic INSERT | §23.5 |
| **RBAC Permissions** | `role_permissions` table | INSERT with action_key | §23.6.2 |
| **Settings** | `settings` table with ext. prefix | INSERT with namespaced key | §4.7 |
| **Webhooks** | `webhook_endpoints.events` array | Event key subscription | §10, §23.7.1 |
| **API Routes** | `/ext/:module/*` namespace | Route registration | §10.5 |
| **Audit Actions** | `audit_log.action` VARCHAR | Log with any namespaced key | §23.4.1 |
| **Feature Flags** | `feature_flags` table | INSERT with module prefix | §9.5 |
| **i18n Strings** | `i18n_strings` table | INSERT with namespace | §14.5 |
| **Custom Entities** | `entity_registry` + auto-tables | Dynamic entity types | §23.8 |
| **Rate Limiting** | `rate_limit_policies` table | INSERT with namespaced key | §12 |
| **Search** | `search_index` configuration | Entity type registration | §8.5 |

### 2.3 Dynamic Registration API

Product modules register extensions at system initialization:

```typescript
// Extension registration pattern
interface ExtensionRegistry {
  // MCP Capabilities
  registerCapability(capabilityKey: string, schemas: CapabilitySchemas): void;
  
  // RBAC Permissions
  registerPermission(actionKey: string, metadata: PermissionMeta): void;
  
  // Webhook Events
  registerEvent(eventKey: string, payloadSchema: JSONSchema): void;
  
  // API Routes
  registerRoutes(basePath: string, routes: RouteDefinition[]): void;
  
  // Custom Entities
  registerEntity(entityType: string, schema: EntitySchema): void;
  
  // Settings Schema
  registerSettings(settingsKey: string, schema: SettingsSchema): void;
}
```

### 2.4 Namespace Conventions

| Prefix | Owner | Example |
|--------|-------|---------|
| `ga.*` | GeneralApp | `ga.user.created`, `ga.workspace.suspended` |
| `fb.*` | FormBuilder | `fb.form.submitted`, `fb.payment.succeeded` |
| `pm.{module}.*` | Product Module | `pm.crm.record.created`, `pm.billing.invoice.paid` |
| `ext.{vendor}.*` | Third-party extensions | `ext.salesforce.sync`, `ext.zapier.trigger` |

### 2.5 Extension Validation

The system validates extensions at registration time:

```typescript
// Validation rules
- action_key must match /^[a-z0-9_]+(\.[a-z0-9_]+)+$/
- event_key must be namespaced (contain at least one dot)
- capability_key must be unique across all modules
- entity_type must not conflict with core entities (user, session, tenant, etc.)
```

### 2.6 Extension Discovery

All extensions are discoverable via API:

```typescript
// List all registered capabilities
GET /api/ext/capabilities

// List all RBAC permissions
GET /api/ext/permissions

// List all webhook events
GET /api/ext/events

// List all custom entities
GET /api/ext/entities
```

---

## 3. MULTI-TENANCY

The platform is tenant-capable by default. Every tenant-aware data record is scoped to a tenant. A product built on this platform may still run in a single-tenant deployment profile by provisioning exactly one active tenant/workspace while retaining the same core scoping primitives.

### 3.1 Tenancy Models

| Model | Description |
|-------|-------------|
| **multi-tenant** (default) | Multiple isolated workspaces. Each workspace has its own users, data, settings, billing, and MCP configuration. |
| **single-tenant** | One workspace for the entire application. All registered users join the default workspace. |
| **hybrid** | Multiple workspaces exist, but the `system_owner` can designate shared resource pools. |

### 3.2 Tenant (Workspace) Entity

```typescript
interface Tenant {
  id: string;              // UUID. Immutable.
  slug: string;            // URL-safe identifier. Unique globally.
  name: string;            // Display name. Editable by an authorized workspace administrator.
  plan: string;            // Subscription plan.
  status: 'active' | 'suspended' | 'deleted' | 'pending-setup';
  settings: JSON;          // Tenant-level settings
  createdBy: string;       // User who created the workspace.
  memberCount: number;     // Cached count.
  storageUsed: number;    // Bytes used.
}
```

### 3.3 ISP Scan — Tenant

- **SCAN-01 AUTH:** Tenant context is resolved from the subdomain or path segment on every request.
- **SCAN-02 SESSION:** Session token contains tenant_id claim.
- **SCAN-03 TENANT:** All DB queries append WHERE tenant_id = $1.
- **SCAN-04 RBAC:** workspace.create, workspace.delete, workspace.suspend evaluated for every action.
- **SCAN-05 EMAIL:** New tenant → EMAIL_BRANCH(welcome-workspace).
- **SCAN-06 MCP:** Each tenant can have their own MCP provider overrides.
- **SCAN-07 AUDIT:** All tenant-level config changes logged.
- **SCAN-08 NOTIF:** `system_owner` or delegated elevated admins are notified on member join, plan change, suspension.
- **SCAN-09 RATE-LIMIT:** Per-tenant rate limits enforced separately.
- **SCAN-10 SEARCH:** Search scoped to tenant_id.
- **SCAN-11 SETTINGS:** Tenant has its own settings panel.
- **SCAN-12 FAILURE:** Tenant record missing → 404 with 'Workspace not found'.

### 3.4 Workspace Provisioning Flow

1. User clicks 'Create Workspace'.
2. Enters: workspace name, slug (auto-suggested).
3. Billing: if plan requires payment, payment step shown.
4. POST /workspaces → tenant record created → initiating user assigned as the controlling administrator for that workspace. In the single-tenant deployment profile, this is the bootstrap/initial-owner `system_owner` assignment.
5. EMAIL_BRANCH: welcome-workspace template sent.
6. Onboarding flow triggered.
7. Redirect to new workspace URL.

| State | Description |
|-------|-------------|
| idle | Create Workspace button visible. |
| slug-checking | Async uniqueness check. |
| slug-taken | Inline error. |
| creating | POST in progress. |
| success | Workspace created. |
| plan-limit | User has reached max workspaces. |
| error | Server error. |

### 3.5 Tenant Isolation Enforcement

- **Database:** PostgreSQL Row-Level Security policy on every tenant-scoped table.
- **API Middleware:** tenant_id extracted from JWT claim, set as DB session variable.
- **Storage:** Object key prefixed with tenant_id.
- **Search Index:** All documents indexed with tenant_id field.
- **WebSocket:** Socket room scoped to tenant_id.
- **MCP:** Per-tenant API keys stored encrypted.

### 3.6 Tenant Suspension & Deletion (Admin / `system_owner`)

| State | Description |
|-------|-------------|
| suspend | All users force-logged out. Login blocked. Data preserved. |
| reinstate | Suspension lifted. |
| delete-request | `system_owner` initiates. 30-day grace period. |
| grace-period | Workspace inaccessible. Data queued for deletion. |
| deleted | All data purged. Audit log entries anonymised. |

---

## 4. SESSION MANAGEMENT

Sessions are first-class entities. Every active login is a session record.

### 4.1 Session Entity

```typescript
interface Session {
  id: string;
  userId: string;
  tenantId: string;
  token: string;          // hashed in DB
  device: {
    browser: string;
    os: string;
    type: 'mobile' | 'tablet' | 'desktop';
  };
  ip: string;
  location: string;       // reverse-geocoded
  createdAt: Date;
  lastActiveAt: Date;
  expiresAt: Date;
  rememberMe: boolean;
  terminatedAt: Date | null;
  terminatedBy: string | null;
  terminationReason: string | null;
}
```

### 4.2 Session States

| State | Description |
|-------|-------------|
| active | Session valid. Token not expired. |
| idle | No activity for idle-timeout window. Warning shown. |
| idle-warning | Banner shown: 'You will be logged out in N minutes.' |
| expired | TTL elapsed. Next request returns 401. |
| terminated-self | User manually ended this session. |
| terminated-admin | Admin or system ended this session. |
| revoked-security | Session revoked because of a security event. |
| suspicious | IP changed beyond geo-threshold. Flagged. |

### 4.3 User Session Management (Self-Service)

Profile > Security > Active Sessions. List of all sessions.

| Element | States | Description |
|---------|--------|-------------|
| Current session badge | idle, active | Cannot terminate own current session here |
| Other session row | idle, active, suspicious-flag | Terminate button |
| Terminate All Others | idle, hover, loading | Terminates every session except current |
| Session list | loading, loaded, empty, error | AUDIT_BRANCH logged |

**ISP Scan — Session:**
- **SCAN-01 AUTH:** User must be authenticated.
- **SCAN-04 RBAC:** session.view.own, session.terminate.own.
- **SCAN-07 AUDIT:** session.terminated.self logged.
- **SCAN-08 NOTIF:** EMAIL_BRANCH fires on 'Terminate All Others'.
- **SCAN-09 RATE-LIMIT:** session.terminate limited to 20/hour.

### 4.4 Admin Session Management

Admin Panel > Users > [User] > Active Sessions

| Action | Description |
|--------|-------------|
| End Single Session | DELETE /admin/sessions/:id |
| End All User Sessions | All future requests return 401 |
| End Sessions — Global | `system_owner` only by default; may be delegated to `super_admin` by policy. All sessions across all users. |
| End Sessions by Device | Filter by device type → bulk terminate |
| End Sessions by IP | Terminate all sessions from specific IP/CIDR |
| Block & Terminate | Terminate and flag IP in blocklist |

| State | Description |
|-------|-------------|
| confirmation-modal | Action requires confirmation. |
| processing | DELETE in progress. |
| success | Sessions terminated. |
| partial-failure | Some failed to terminate. |
| error | Full failure. |

### 4.5 Admin Configuration — Sessions

- **Session TTL:** default (24h), remember-me (30d). Configurable per tenant.
- **Idle timeout:** default: 25 min, then auto-terminate window (5 min).
- **Max concurrent sessions per user:** unlimited | 1 | 3 | 5 | N.
- **New device notification:** email user when login from new device.
- **Geo-anomaly detection:** flag when login IP country differs.
- **Session binding:** bind session to IP.
- **Force re-auth events:** password change always invalidates sessions.
- **Session fixation guard:** authentication state changes (login, password reset completion, email change confirmation, privilege elevation) rotate the effective session/refresh-token chain immediately. Old refresh tokens are invalidated and any stale session identifiers are revoked rather than reused.

**RBAC Branch:** session.view.any, session.terminate.any, session.terminate.all, session.terminate.global.

---

## 5. SETTINGS SYSTEM

Settings exist at three tiers: User Preferences, Workspace Settings, and System Settings.

### 5.1 Settings Hierarchy

| Tier | Description |
|------|-------------|
| **System Settings** (Tier 1) | `system_owner` only. Governs defaults and hard limits for all tenants. |
| **Workspace Settings** (Tier 2) | `super_admin` / delegated `admin` workspace settings access. Governs defaults for all users. Can be more restrictive. |
| **User Preferences** (Tier 3) | Individual user. Cannot exceed workspace limits. |

### 5.2 Setting Entity

```typescript
interface Setting {
  key: string;
  scope: 'system' | 'workspace' | 'user';
  value: any;
  type: 'boolean' | 'number' | 'string' | 'enum' | 'json';
  allowedValues?: any[];
  min?: number;
  max?: number;
  locked: boolean;
  lockedBy: string | null;
  description: string;
  requiresRestart: boolean;
}
```

### 5.3 User Preferences

| Element | States | Interconnects / Branches |
|---------|--------|--------------------------|
| Language / Locale | idle, saving, saved, error, locked-by-workspace | i18n_BRANCH |
| Timezone | idle, saving, saved, error | None |
| Date / Time Format | idle, saving, saved | i18n_BRANCH |
| Theme | idle, saving, saved | THEME_BRANCH |
| Notification Prefs | saving, saved | NOTIF_BRANCH |
| Email Digest | saving, saved | EMAIL_BRANCH |
| Keyboard Shortcuts | enabled/disabled, customise | SHORTCUT_BRANCH |
| Accessibility | reduce-motion, high-contrast, text-size | A11Y_BRANCH |
| Two-Factor Auth | enabled/disabled | AUTH_BRANCH |
| Active Sessions | list | SESSION_BRANCH |

| State | Description |
|-------|-------------|
| loading | GET /users/me/preferences. |
| loaded | All current values shown. |
| editing | Unsaved state indicator. |
| saving | PATCH /users/me/preferences. |
| saved | Green tick. |
| error | Red border. |
| locked | Greyed. Tooltip explains. |

### 5.4 Workspace Settings

| Section | Description | Interconnects / Branches |
|---------|-------------|--------------------------|
| General | name, slug, logo, brand colours | MCP STORAGE |
| Auth & Security | allowed login methods, 2FA, password policy | AUTH_BRANCH |
| Members & Roles | default role, invite permissions | RBAC_BRANCH |
| Email | MCP email connector, templates | EMAIL_BRANCH, MCP_BRANCH |
| Integrations / MCP | all external provider configs | MCP_BRANCH |
| AI | AI feature toggles, provider priorities | MCP_BRANCH |
| Billing | plan, payment methods, invoices | PAYMENT_BRANCH |
| Notifications | default notification preferences | NOTIF_BRANCH |
| Storage | MCP storage provider, quotas | MCP_BRANCH, STORAGE_BRANCH |
| Feature Flags | per-workspace feature overrides | FLAG_BRANCH |
| Webhooks | outbound webhook endpoints | WEBHOOK_BRANCH |
| API | API key management, allowed scopes | APIKEY_BRANCH |
| Danger Zone | suspend, delete, transfer ownership | AUDIT_BRANCH |

| State | Description |
|-------|-------------|
| loading | Skeleton per section. |
| no-permission | Section greyed. Tooltip explains. |
| locked-system | Section greyed. Locked by system admin. |
| editing | Unsaved changes indicator. |
| saving | Loading state. |
| saved | Green toast. |
| change-impact | Warning modal for impactful changes. |

### 5.5 System Settings

`system_owner` only. Accessible at `/system-admin/settings`.

- **Maintenance Mode:** take system offline with custom message.
- **Registration:** open / invite-only / closed globally.
- **Multi-tenancy mode:** multi / single / hybrid.
- **Global rate-limit defaults.**
- **Feature flag master toggles.**
- **Data retention policies.**
- **Audit log export schedule.**

### 5.5.1 Cross-System Settings Resolution

**Connector governance note:** although consumer product modules may expose the main integration UI, the authoritative shared model across the platform is the common `connectors` registry plus per-capability `capability_bindings` (primary + fallback routing). GeneralApp and FormBuilder consume resolved capabilities from that shared registry rather than maintaining separate connector authorities.

Settings across GeneralApp, product modules, and Form Builder follow a unified hierarchy:

```
┌─────────────────────────────────────────────────────────────────┐
│                    SETTINGS HIERARCHY                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TIER 1: SYSTEM SETTINGS (system_owner only)                    │
│  ───────────────────────────────────                            │
│  Location: /system-admin/settings (GeneralApp)                  │
│                                                                  │
│  • Maintenance mode (system-wide)                               │
│  • Multi-tenancy mode                                           │
│  • Registration policy                                          │
│  • Global rate limit defaults                                   │
│  • Feature flag masters                                         │
│  • Data retention minimums (cannot be overridden lower)         │
│  • Maximum workspace limits                                     │
│                                                                  │
│  TIER 2: WORKSPACE/TENANT SETTINGS                              │
│  ────────────────────────────────                               │
│  Location: /settings/workspace (GeneralApp)                     │
│           /settings/* (Product Module)                          │
│                                                                  │
│  • Workspace name, branding                                     │
│  • MCP connector configurations                                 │
│  • Auth methods, 2FA policy                                     │
│  • Email templates & branding                                   │
│  • Default form theme                                           │
│  • Notification defaults                                        │
│  • Data retention (≥ system minimum)                            │
│                                                                  │
│  TIER 3: FORM/ENTITY SETTINGS                                   │
│  ─────────────────────────────                                  │
│  Location: FormBuilder > Form Settings                          │
│           Product Module > Entity/Form Schemas                  │
│                                                                  │
│  • Form visibility, rate limits                                 │
│  • Form theme overrides                                         │
│  • After-submit actions                                         │
│  • Data retention (≥ system minimum, longest effective wins)    │
│                                                                  │
│  TIER 4: USER PREFERENCES                                       │
│  ─────────────────────                                          │
│  Location: /settings/profile (GeneralApp)                       │
│           /module/settings (Product Module)                     │
│                                                                  │
│  • Locale/timezone                                              │
│  • Theme preference                                             │
│  • Notification preferences                                     │
│  • 2FA method selection                                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Resolution Rules

| Conflict | Resolution |
|----------|------------|
| System setting vs Workspace setting | System wins (enforced minimum/maximum) |
| Workspace setting vs Form setting | Security-sensitive settings resolve to the most restrictive effective value. Data-retention settings resolve to the longest effective retention that still satisfies legal minimums and system floors. |
| User preference vs Workspace default | User wins unless locked by admin |
| Product module vs GeneralApp tenant | Longer retention wins (conservative) |
| FormBuilder vs Product module theme | Form-level overrides tenant default |

#### Setting Propagation

```typescript
// Example: Rate Limit Resolution
const effectiveRateLimit = {
  // System provides the maximum permitted throughput for this surface.
  // Lower values are more restrictive.
  systemMaxAllowed: systemSettings.rateLimitMax,
  
  // Tenant sets their preference
  preferred: tenantSettings.rateLimit,
  
  // Form can be more restrictive
  form: formSettings.rateLimit,
  
  // Final value: most restrictive (lowest allowed request volume)
  value: Math.min(systemMaxAllowed, tenantPreferred, formSetting)
};

// Example: Data Retention Resolution
const effectiveRetention = {
  // System sets minimum (legal requirement)
  minimum: systemSettings.retentionMinimum, // e.g., 7 years for payments
  
  // Tenant sets their preference
  preferred: tenantSettings.retentionDays,
  
  // Form can request longer or shorter, but it cannot undercut the system/legal floor.
  form: formSettings.dataRetentionDays,
  
  // Final value: longest effective retention (conservative)
  value: Math.max(systemMinimum, tenantPreferred, formSetting)
};
```

#### Cross-System Settings Reference

| Setting | System of Record | Used By |
|---------|-----------------|---------|
| `tenant_id` | GeneralApp | All systems |
| `timezone` | Product-module tenant config | All timestamps |
| `default_currency` | Product-module tenant config | Pricing, payments |
| `email_connectors` | Shared `connectors` + `capability_bindings` registry | Product modules, FormBuilder, GeneralApp |
| `storage_connector` | Shared `connectors` + `capability_bindings` registry | Product modules, FormBuilder, GeneralApp |
| `form_themes` | FormBuilder → synced to consuming product module | Form rendering |
| `rate_limits` | GeneralApp (floor) + product module | All API endpoints |
| `data_retention` | System minimum + product module | All data stores |

### 5.6 ISP Scan — Settings

- **SCAN-01 AUTH:** All settings endpoints require authentication. System settings require `system_owner` auth.
- **SCAN-02 SESSION:** Settings reads can be cached. Cache invalidated on PATCH.
- **SCAN-03 TENANT:** Workspace settings strictly scoped. System settings have no tenant_id.
- **SCAN-04 RBAC:** settings.read.user, settings.write.user, settings.read.workspace, settings.write.workspace, settings.read.system, settings.write.system.
- **SCAN-07 AUDIT:** Every settings write logged.
- **SCAN-08 NOTIF:** Workspace-wide setting changes broadcast in-app notification.
- **SCAN-09 RATE-LIMIT:** settings.write limited to 60/hour per user.
- **SCAN-12 FAILURE:** Settings read failure → show stale cached values.

### 5.7 Settings Extension Point

Product modules can register custom settings that integrate with the platform settings hierarchy.

#### Extension Schema

```typescript
// Product module registers custom settings
interface SettingsExtension {
  settingKey: string;           // Must be namespaced: 'ext.{module}.{name}'
  scope: 'workspace' | 'user';
  type: 'boolean' | 'number' | 'string' | 'enum' | 'json' | 'encrypted';
  defaultValue: any;
  validation?: {
    min?: number;
    max?: number;
    pattern?: string;           // Regex for string validation
    allowedValues?: any[];      // For enum type
  };
  ui?: {
    label: string;              // Display label (i18n key)
    description: string;        // Help text (i18n key)
    component: 'toggle' | 'input' | 'select' | 'textarea' | 'json-editor';
    category: string;           // Grouping in settings UI
    icon?: string;              // Optional icon identifier
  };
  rbac?: {
    read: string;               // Required permission to read
    write: string;              // Required permission to modify
  };
  resolution?: 'min' | 'max' | 'override';  // How to resolve conflicts
}
```

#### Registration Example

```typescript
// Example product module registers a custom setting
settingsRegistry.register({
  settingKey: 'ext.product_module.record_followup_days',
  scope: 'workspace',
  type: 'number',
  defaultValue: 30,
  validation: { min: 7, max: 90 },
  ui: {
    label: 'settings.domain_renewal_days.label',
    description: 'settings.domain_renewal_days.help',
    component: 'input',
    category: 'notifications',
    icon: 'calendar'
  },
  rbac: {
    read: 'pm.settings.read',
    write: 'pm.settings.write'
  },
  resolution: 'override'
});
```

#### Storage

Extended settings are stored in the `settings` table with `setting_key` prefixed by module namespace:

```sql
-- Example extended settings
INSERT INTO settings (setting_key, scope, tenant_id, value, type) VALUES
('ext.product_module.record_followup_days', 'workspace', 'tenant-123', '30', 'number'),
('ext.form_builder.max_upload_size', 'workspace', 'tenant-123', '26214400', 'number'),
('ext.my_module.api_endpoint', 'workspace', 'tenant-123', '"https://api.example.com"', 'encrypted');
```

#### Resolution with Core Settings

Extended settings participate in the standard resolution hierarchy:

```
system.setting.{key} → workspace.ext.{module}.{key} → user.ext.{module}.{key}
```

#### API Access

Extended settings are accessible via the standard settings API:

```typescript
// Read extended setting
GET /api/settings/ext.product_module.record_followup_days

// Write extended setting (requires appropriate RBAC)
PATCH /api/settings/ext.product_module.record_followup_days
{ "value": 45 }

// List all extended settings for a module
GET /api/settings?prefix=ext.product_module
```

---

## 6. HELP & SUPPORT SYSTEM

The support system provides: Knowledge Base, Support Tickets, Live Chat, and in-app Feedback Widget.

### 6.1 Knowledge Base

| State | Description |
|-------|-------------|
| article-list | Articles grouped by category. Search within KB. |
| article-view | Rendered Markdown. Table of contents. 'Was this helpful?' feedback. |
| search-results | Full-text search across KB. Highlighted matches. |
| empty | No articles published. |
| draft | Admin-only visibility. |

**Knowledge Base Admin:**

| Element | States | Interconnects / Branches |
|---------|--------|--------------------------|
| Article Editor | idle, editing, saving, saved, published, draft, archived | MCP_BRANCH (rich text / AI-assist) |
| Category Manager | idle, creating, editing, reordering, deleting | AUDIT_BRANCH |
| Article Feedback | helpful-count, not-helpful-count, comment thread | NOTIF_BRANCH |
| Publish Toggle | draft→published, published→draft | AUDIT_BRANCH |

### 6.2 Support Tickets

User clicks Help > Submit a Request.

- **Form:** Subject, Description, Category, Attachments.
- **POST /support/tickets.**
- **EMAIL_BRANCH:** ticket-created template to user. Ticket number included.
- **EMAIL_BRANCH:** ticket-new-admin template to support team.
- **NOTIF_BRANCH:** in-app to user.

| State | Description |
|-------|-------------|
| open | Newly created. No agent assigned. |
| assigned | Agent picked up ticket. |
| pending-user | Agent replied, awaiting user response. Auto-close after N days. |
| pending-agent | User replied. Waiting for agent. |
| resolved | Agent marked resolved. User can reopen within 72 h. |
| closed | Resolved + user confirmation. |
| spam | Admin flagged. Hidden from user. |

**Admin Ticket Management:**

| Element | States | Interconnects / Branches |
|---------|--------|--------------------------|
| Ticket Queue | open, assigned, pending-user, filtered | RBAC_BRANCH |
| Assign Ticket | idle, assigning, assigned | NOTIF_BRANCH, EMAIL_BRANCH |
| Reply | idle, composing, sending, sent, error | EMAIL_BRANCH, AUDIT_BRANCH |
| Internal Note | idle, composing, saved | Visible only to agents |
| Priority | low, normal, high, urgent | NOTIF_BRANCH |
| Merge Tickets | select source, select target, confirm | AUDIT_BRANCH |
| Close / Resolve | idle, confirming, closed/resolved | EMAIL_BRANCH, NOTIF_BRANCH |
| Macros | Pre-written responses | None |
| SLA Indicator | within-SLA, warning, breached | NOTIF_BRANCH |

**ISP Scan — Tickets:**
- **SCAN-01 AUTH:** Ticket creation requires auth.
- **SCAN-03 TENANT:** Tickets scoped to tenant.
- **SCAN-04 RBAC:** support.ticket.create, support.ticket.view.own, support.ticket.view.any.
- **SCAN-05 EMAIL:** All ticket state changes fire emails.
- **SCAN-07 AUDIT:** Every ticket state change logged.
- **SCAN-09 RATE-LIMIT:** ticket.create limited to 5/hour per user.

### 6.3 Live Chat

Live chat is an MCP capability: chat.session. Default providers: Intercom, Crisp, Tawk.to.

| State | Description |
|-------|-------------|
| unavailable | No MCP chat provider configured. |
| offline | Provider configured but agents offline. Bot handles. |
| online | Agent available. Chat widget opens. |
| in-progress | Active chat session. |
| ended | Chat ended. Transcript emailed. |

### 6.4 In-App Feedback Widget

A small floating button on every screen. Thumbs up/down + optional text + current page URL.

| State | Description |
|-------|-------------|
| idle | Small button in corner. |
| open | Feedback popover open. |
| submitting | POST /feedback. Loading. |
| success | Thank you message. Auto-closes after 3s. |
| error | Submission failed. Draft retained. |

**Admin Feedback Dashboard:**
- List of all feedback with rating, comment, URL, user, timestamp.
- Aggregate: satisfaction score over time.
- Export CSV.
- RBAC: feedback.view — admin and `system_owner` only.

---

## 7. ONBOARDING & FIRST-RUN EXPERIENCE

Onboarding is triggered on first login to a new workspace. It is a state machine.

### 7.1 Onboarding Flow States

| State | Description |
|-------|-------------|
| not-started | User has never completed onboarding. First login redirects. |
| in-progress | User started onboarding. Progress saved per step. |
| skipped | User clicked 'Skip for now'. Reminder shown in header. |
| completed | All required steps done. Confetti animation. |
| admin-reset | Admin reset onboarding for user. |

### 7.2 Default Onboarding Steps

| Element | States | Interconnects / Branches |
|---------|--------|--------------------------|
| Step 1: Profile Setup | idle, active, filled, skipped, completed | USER_PROFILE_BRANCH |
| Step 2: Invite Team | idle, active, email-entered, sent, skipped, completed | EMAIL_BRANCH, RBAC_BRANCH |
| Step 3: First Action | idle, active, completed | Core feature branch |
| Step 4: Connect Integration | idle, active, skipped, completed | MCP_BRANCH |
| Step 5: Tour / Tooltips | idle, active, step-N-of-M, completed, dismissed | None |

**Admin Configuration — Onboarding:**
- Enable/disable onboarding globally.
- Configure required vs skippable steps.
- Add custom onboarding steps.
- Set onboarding reminder email schedule.
- View onboarding completion rate per step.
- Reset onboarding for user or all users.

**ISP:** SCAN-07 AUDIT: onboarding events logged. SCAN-08 NOTIF: admin notified when completion rate drops.

---

## 8. GLOBAL SEARCH

Global Search accessible from top navigation bar (Cmd/Ctrl+K). Searches across all entity types user has permission to see.

### 8.1 Search States

| State | Description |
|-------|-------------|
| idle | Search bar collapsed. Icon only. |
| focused | Bar expanded. Recent searches shown. |
| typing | Debounced (300ms) query sent. |
| results | Results grouped by entity type. |
| no-results | No matches. Suggestions shown. |
| error | Search API unavailable. DB fallback. |
| loading-more | Infinite scroll. |

### 8.2 Indexed Entity Types

| Entity | Description | RBAC |
|--------|-------------|------|
| Users | name, email, display name | Admin only. |
| Workspace | name, slug | System admin only. |
| Support Tickets | subject, description | Agent/admin or own tickets. |
| KB Articles | title, content | All authenticated users. |
| Audit Events | action, actor email | Admin only. |
| Custom Entities | whatever app-specific feature introduces | Varies |

### 8.3 ISP Scan — Search

- **SCAN-03 TENANT:** ALL search queries append tenant_id filter.
- **SCAN-04 RBAC:** each entity type has a search.{entity} action.
- **SCAN-09 RATE-LIMIT:** search limited to 60 queries/minute per user.
- **SCAN-12 FAILURE:** index unavailable → fall back to DB ILIKE query.

### 8.4 Search Admin Configuration

- Enable/disable global search.
- Configure which entity types are indexed per tenant.
- Set search result visibility rules.
- Search analytics: top queries, zero-result queries.

### 8.5 Search Extension Point

Product modules can register custom entity types for global search indexing.

**Entity Registration:**

```typescript
// Product module registers custom entity for search
searchRegistry.registerEntity({
  entityType: 'ext.my_module.project',
  displayName: 'Project',
  icon: 'folder',
  fields: {
    indexed: ['name', 'description', 'tags'],  // Full-text indexed
    stored: ['status', 'owner_id', 'due_date'], // Stored but not indexed
    filterable: ['status', 'priority']          // Facet/filter fields
  },
  resultTemplate: {
    title: '{name}',
    subtitle: 'Project | {status}',
    url: '/ext/my-module/projects/{id}',
    actions: [
      { label: 'View', action: 'view' },
      { label: 'Edit', action: 'edit', rbac: 'ext.my_module.project.edit' }
    ]
  },
  rbac: {
    search: 'ext.my_module.project.view'  // Required permission to search
  }
});
```

**Indexing Data:**

```typescript
// Index custom entity
await search.index({
  entityType: 'ext.my_module.project',
  id: 'project-123',
  tenantId: 'tenant-456',
  data: {
    name: 'Website Redesign',
    description: 'Complete overhaul of company website',
    status: 'active',
    priority: 'high',
    tags: ['design', 'frontend'],
    owner_id: 'user-789',
    due_date: '2026-06-01'
  }
});
```

**Search Query:**

```typescript
// Search includes custom entities automatically
GET /api/search?q=website&types=ext.my_module.project,user,workspace

// Response
{
  "results": [
    {
      "type": "ext.my_module.project",
      "id": "project-123",
      "title": "Website Redesign",
      "subtitle": "Project | active",
      "url": "/ext/my-module/projects/project-123"
    }
  ],
  "facets": {
    "by_type": {
      "ext.my_module.project": 5,
      "user": 12
    }
  }
}
```

---

## 9. FEATURE FLAGS

Feature flags are the runtime switch for every feature. No feature is permanently hardwired on.

### 9.1 Flag Entity

```typescript
interface FeatureFlag {
  key: string;           // e.g. 'ai.nodeGeneration'
  enabled: boolean;      // global on/off
  rollout: number;       // 0-100: percentage
  scope: 'global' | 'tenant' | 'user';
  tenantOverrides: { tenantId: string; enabled: boolean }[];
  userOverrides: { userId: string; enabled: boolean }[];
  conditions: Condition[];
  description: string;
  expiresAt: Date | null;
}
```

### 9.2 Flag Evaluation

```typescript
const enabled = await flagService.evaluate('ai.nodeGeneration', {
  userId, tenantId, role, subscriptionTier
});

if (!enabled) return <FeatureUnavailable reason='Not available on your plan.' />;
```

### 9.3 Flag States

| State | Description |
|-------|-------------|
| on-global | Flag enabled for all users. |
| off-global | Flag disabled globally. |
| rollout | Flag enabled for X% of users. Sticky. |
| tenant-on | Enabled for specific tenants only. |
| user-on | Enabled for specific users only. |
| expired | Past expiresAt. Evaluated as off. |
| condition-based | Evaluates conditions at runtime. |

### 9.4 Admin Configuration

- Create / edit / delete flags.
- Toggle global on/off.
- Set rollout percentage.
- Add tenant or user overrides.
- Set expiry date.
- View which users currently have a flag enabled.

**ISP:** SCAN-04 RBAC: flag.create, flag.edit, flag.delete — `system_owner` only. SCAN-07 AUDIT: every flag change logged. SCAN-12 FAILURE: flag service down → last cached value from Redis.

### 9.5 Feature Flag Extension Point

Product modules can register custom feature flags that integrate with the platform feature flag system.

**Flag Registration:**

```typescript
// Product module registers custom feature flags
featureFlagRegistry.register([
  {
    flagKey: 'ext.my_module.new_dashboard',
    name: 'New Dashboard UI',
    description: 'Enable the redesigned dashboard interface',
    category: 'My Module',
    defaultValue: false,
    rollout: {
      strategy: 'percentage',
      percentage: 0  // Start at 0%
    },
    targeting: {
      // Target specific tenant types
      tenantTypes: ['enterprise'],
      // Target specific user roles
      userRoles: ['admin', 'super_admin']
    }
  },
  {
    flagKey: 'ext.my_module.beta_api',
    name: 'Beta API v2',
    description: 'Enable experimental API endpoints',
    category: 'My Module',
    defaultValue: false,
    requiresOptIn: true  // Users must explicitly enable
  }
]);
```

**Flag Evaluation:**

```typescript
// Check custom flag (same API as core flags)
const isEnabled = await featureFlag.check({
  flagKey: 'ext.my_module.new_dashboard',
  userId: 'user-123',
  tenantId: 'tenant-456'
});

// Usage in code
if (isEnabled) {
  return <NewDashboard />;
} else {
  return <LegacyDashboard />;
}
```

**Flag Discovery:**

```typescript
// List all flags including extended
GET /api/ext/feature-flags

// Response
{
  "core": [...],
  "extensions": [
    {
      "flagKey": "ext.my_module.new_dashboard",
      "name": "New Dashboard UI",
      "module": "my_module",
      "enabled": true,
      "rollout": { "percentage": 25 }
    }
  ]
}
```

**Admin UI Integration:**

Extended flags appear in the same admin interface as core flags, grouped by module category.

---

## 10. WEBHOOKS

> **Canonical Schema:** §23.7 `webhook_endpoints` (SQL DDL)

Webhooks are outbound HTTP calls sent when events occur. All product modules use the shared webhook infrastructure defined in §23.7.

### 10.1 Webhook Entity (TypeScript Projection)

```typescript
// TypeScript projection of §23.7 webhook_endpoints table
interface WebhookEndpoint {
  id: string;
  tenantId: string;
  name: string;                    // Human-readable label
  url: string;                     // HTTPS required
  signingSecretEncrypted: string;  // Raw secret encrypted at rest for outbound HMAC signing
  secretFingerprint: string;       // Non-sensitive identifier shown in UI/audit logs
  events: string[];                // Subscribed event keys (e.g., ['ga.user.created', 'fb.submission.received'])
  
  // Unified Status Enum (replaces enabled boolean + separate state tracking)
  status: 'unverified' | 'active' | 'disabled' | 'failing';
  verifiedAt: Date | null;
  
  // Delivery tracking
  lastDeliveryAt: Date | null;
  lastDeliveryStatus: number | null;  // HTTP status code
  failureCount: number;
  
  createdAt: Date;
  updatedAt: Date;
}
```

### 10.2 Webhook Status States

| Status | Description |
|--------|-------------|
| `unverified` | Endpoint registered but verification challenge not completed. No deliveries. |
| `active` | Verified and delivering. Receives all subscribed events. |
| `failing` | 3+ consecutive delivery failures. Admin notified. Retries continue. |
| `disabled` | Manually disabled or auto-disabled after 10 consecutive failures. No deliveries. |

**Status Transitions:**
```
unverified → active (verification success)
active → failing (3 consecutive failures)
active → disabled (manual disable)
failing → disabled (10 consecutive failures)
failing → active (successful delivery)
disabled → unverified (re-enable requires re-verification)
```

### 10.3 Supported Events

| Event | Description |
|-------|-------------|
| `ga.user.*` | `ga.user.created`, `ga.user.updated`, `ga.user.deleted`, `ga.user.suspended`, `ga.user.plan_changed` |
| `ga.session.*` | `ga.session.created`, `ga.session.terminated` |
| `ga.ticket.*` | `ga.ticket.created`, `ga.ticket.assigned`, `ga.ticket.replied`, `ga.ticket.resolved`, `ga.ticket.closed` |
| `ga.billing.*` | `ga.billing.subscription_created`, `ga.billing.payment_succeeded`, `ga.billing.payment_failed` |
| `ga.workspace.*` | `ga.workspace.created`, `ga.workspace.updated`, `ga.workspace.suspended`, `ga.workspace.deleted` |
| `ga.custom.*` | Admin-defined GeneralApp event keys must remain namespaced under `ga.` |

**Naming rule:** shared outbound events must use explicit system prefixes (`ga.*`, `cp.*`, `fb.*`) to avoid collisions across modules. Legacy non-prefixed examples are illustrative only and must not be treated as the canonical outbound namespace.

### 10.4 Delivery & Security

- All webhook payloads signed with HMAC-SHA256.
- Timeout: 10 seconds per delivery attempt.
- Payload max size: 100 KB.
- Delivery log: last 50 attempts.

**ISP Scan — Webhooks:**
- **SCAN-01 AUTH:** Webhook management requires auth.
- **SCAN-03 TENANT:** Webhooks strictly scoped to tenant.
- **SCAN-04 RBAC:** webhook.create, webhook.edit, webhook.delete, webhook.view.
- **SCAN-07 AUDIT:** webhook.created, webhook.delivery.failed, webhook.auto-disabled logged.
- **SCAN-08 NOTIF:** Auto-disable triggers in-app + email.
- **SCAN-09 RATE-LIMIT:** Delivery rate-limited if target returns 429.

---

## 11. API KEY MANAGEMENT

API Keys allow programmatic authentication without session cookie. Keys are scoped, rate-limited, and auditable.

### 11.1 API Key Entity

```typescript
interface APIKey {
  id: string;
  userId: string;
  tenantId: string;
  name: string;           // user-given label
  prefix: string;         // first 8 chars, shown in listings
  hashedKey: string;      // bcrypt hash — raw key never stored
  scopes: string[];       // e.g. ['read:maps', 'write:maps']
  expiresAt: Date | null;
  lastUsedAt: Date | null;
  lastUsedIp: string | null;
  revokedAt: Date | null;
  revokedBy: string | null;
  createdAt: Date;
}
```

### 11.2 API Key States

| State | Description |
|-------|-------------|
| created | Key just generated. Raw key shown ONCE. |
| active | Key valid. Requests authenticated. |
| unused | Key never used. Warning badge after 30 days. |
| expired | Past expiresAt. Requests return 401. |
| revoked | Manually or automatically revoked. |
| compromised | Admin flagged. Revoked instantly. |

### 11.3 Key Creation Flow

1. User navigates to Profile > API Keys.
2. Clicks 'Generate New Key'.
3. Enters: name, optional expiry date, selects scopes.
4. POST /api-keys. Server generates cryptographically random 32-byte key.
5. Modal: 'Copy your API key. You will not be able to see it again.'
6. Key stored hashed. Prefix stored plaintext.

**RBAC Branch:** apikey.create, apikey.revoke. Scopes bounded by user's own RBAC role.

### 11.4 Admin Key Management

- View all API keys in workspace.
- Revoke any key with reason.
- View all requests made by a specific key.
- Set workspace-level allowed scopes.
- Require key expiry.

**ISP Scan — API Keys:**
- **SCAN-02 SESSION:** API key auth bypasses session.
- **SCAN-03 TENANT:** Key is tenant-scoped.
- **SCAN-04 RBAC:** Every API request evaluated via rulesEngine.
- **SCAN-07 AUDIT:** apikey.created, apikey.revoked, apikey.used.
- **SCAN-09 RATE-LIMIT:** API key requests have their own rate-limit bucket.

### 11.5 API Extension Point

Product modules can register custom API routes that are accessible via API key authentication.

**Route Registration:**

```typescript
// Product module registers custom API routes
apiRegistry.registerRoutes({
  basePath: '/ext/my-module',
  routes: [
    {
      method: 'GET',
      path: '/tasks',
      handler: 'listTasks',
      rbac: 'ext.my_module.task.view',
      rateLimit: { requests: 100, window: '1m' }
    },
    {
      method: 'POST',
      path: '/tasks',
      handler: 'createTask',
      rbac: 'ext.my_module.task.create',
      rateLimit: { requests: 30, window: '1m' }
    },
    {
      method: 'GET',
      path: '/tasks/:id',
      handler: 'getTask',
      rbac: 'ext.my_module.task.view'
    }
  ]
});
```

**Scope Registration:**

```typescript
// Register custom API key scopes
apiKeyRegistry.registerScopes([
  {
    scope: 'ext.my_module:read',
    description: 'Read access to My Module',
    rbac: ['ext.my_module.task.view']
  },
  {
    scope: 'ext.my_module:write',
    description: 'Write access to My Module',
    rbac: ['ext.my_module.task.view', 'ext.my_module.task.create']
  }
]);
```

**API Key with Custom Scopes:**

```typescript
// Create API key with custom scopes
POST /api/api-keys
{
  "name": "My Module Integration",
  "scopes": ["ext.my_module:read", "ga.user.view"],
  "expiresAt": "2026-12-31"
}
```

**Route Discovery:**

```typescript
// List all available routes including extended
GET /api/ext/routes

// Response
{
  "core": [...],
  "extensions": [
    {
      "basePath": "/ext/my-module",
      "routes": [
        { "method": "GET", "path": "/tasks", "rbac": "ext.my_module.task.view" }
      ]
    }
  ]
}
```

---

## 12. RATE LIMITING

Rate limiting is a multi-dimensional policy evaluated per request. Every endpoint has a default limit.

### 12.1 Rate Limit Dimensions

| Dimension | Description |
|-----------|-------------|
| per-user | Counts requests from a specific userId. |
| per-tenant | Aggregate requests from all users in a tenant. |
| per-ip | Applied to unauthenticated endpoints. |
| per-api-key | Separate bucket per API key. |
| per-endpoint | Some endpoints have tighter limits. |
| burst | Short-window allowance for traffic spikes. |

### 12.2 Rate Limit Entity (Unified)

**Canonical Rate Limit Policy** — used across all modules. Product modules may extend with module-specific fields.

```typescript
interface RateLimitPolicy {
  key: string;                    // e.g. 'auth.login.per-ip', 'form.submit.per-user'
  dimension: RateLimitDimension;  // see below
  endpoint: string;               // '/auth/login' or '*' for global
  limit: number;                  // max requests within window
  windowSeconds: number;          // rolling window (FormBuilder uses windowMinutes; convert: ×60)
  burstLimit: number;             // optional burst allowance
  planOverrides: { plan: string; limit: number }[];
  roleOverrides:  { role: string; limit: number }[];
  action: RateLimitAction;        // see below
  exceededMessage?: LocalisedString;  // Optional custom message (FormBuilder extension)
}

type RateLimitDimension = 
  | 'ip'           // Per IP address
  | 'user'         // Per authenticated user
  | 'session'      // Per session (FormBuilder-specific extension)
  | 'tenant'       // Per tenant/workspace
  | 'api-key'      // Per API key
  | 'global';      // Global rate limit

type RateLimitAction = 
  | 'block'        // Hard block with 429
  | 'throttle'     // Slow down responses
  | 'warn'         // Log but allow
  | 'queue';       // Queue for later processing (FormBuilder-specific for submissions)
```

**Module-Specific Extensions:**

| Module | Extension Field | Description |
|--------|-----------------|-------------|
| FormBuilder | `by: 'session'` | Form submissions rate-limited by session |
| FormBuilder | `exceededAction: 'queue'` | Queue submissions instead of blocking |
| FormBuilder | `exceededMessage: LocalisedString` | Custom message shown to form users |
| GeneralApp | `scope: 'endpoint' \| 'global'` | System-wide vs endpoint-specific |

**Cross-Module Resolution:**
```
System Policy (GeneralApp §12.2)
      ↓ overridden by
Tenant Policy (GeneralApp §5.5.1)
      ↓ overridden by
Form Policy (FormBuilder §14)
      ↓ overridden by
Field Policy (FormBuilder §6)

Resolution: Most restrictive (lowest limit, shortest window) wins for security.
```

### 12.3 Default Policies

| Policy | Description |
|--------|-------------|
| auth.login | 5 attempts / 10 min per IP. |
| auth.register | 10 / hour per IP. |
| auth.password-reset | 5 / hour per email. |
| api.global | Free: 100/min. Pro: 500/min. Enterprise: 2000/min. |
| search | 60 / min per user. |
| file.upload | 20 uploads / hour per user. |
| webhook.delivery | Max 1000 / hour per webhook. |
| email.send | 1000 / hour per tenant. |
| support.ticket.create | 5 / hour per user. |

### 12.4 Rate Limit Response

```http
HTTP 429 Too Many Requests
Headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 0
  X-RateLimit-Reset: 1710000060
  Retry-After: 60
```

```json
{ "error": { "code": "RATE_LIMIT_EXCEEDED", "message": "Too many requests.", "retryAfter": 60 } }
```

### 12.5 UI Rate Limit States

| State | Description |
|-------|-------------|
| warned | User approaching limit (80%). |
| throttled | Requests slowed. |
| blocked | All requests blocked until window resets. |
| lifted | Window reset. Normal operation resumed. |

### 12.6 Rate Limit Extension Point

Product modules can register custom rate limit policies for their API endpoints.

**Policy Registration:**

```typescript
// Product module registers custom rate limit
rateLimitRegistry.registerPolicy({
  policyKey: 'ext.my_module.api',
  description: 'My Module API rate limit',
  defaultLimits: {
    anonymous: { requests: 30, window: '1m' },
    authenticated: { requests: 100, window: '1m' },
    api_key: { requests: 1000, window: '1m' }
  },
  dimensions: ['ip', 'user', 'api_key'],  // Dimension priority
  overrideHierarchy: ['system', 'tenant', 'user'],  // Who can override
  action: 'block',  // block | throttle | queue
  responseHeaders: true  // Include X-RateLimit-* headers
});
```

**Applying to Routes:**

```typescript
// Apply rate limit to custom routes
apiRegistry.registerRoutes({
  basePath: '/ext/my-module',
  routes: [
    {
      method: 'GET',
      path: '/tasks',
      handler: 'listTasks',
      rateLimit: 'ext.my_module.api'  // Reference policy
    }
  ]
});
```

**Tenant Override:**

```typescript
// Tenant can customize rate limits
POST /api/settings/ext.my_module.rate_limit
{
  "policyKey": "ext.my_module.api",
  "limits": {
    "authenticated": { "requests": 200, "window": "1m" }
  }
}
```

**Rate Limit Discovery:**

```typescript
// List all rate limit policies including extended
GET /api/ext/rate-limit-policies

// Response
{
  "core": [...],
  "extensions": [
    {
      "policyKey": "ext.my_module.api",
      "module": "my_module",
      "limits": {
        "anonymous": "30/min",
        "authenticated": "100/min"
      }
    }
  ]
}
```

---

## 13. ANALYTICS & REPORTING

Analytics provides data-driven insight. Split into three surfaces: User Activity, Workspace Analytics, and System Analytics.

### 13.1 Event Collection

```typescript
// Client-side event emission (never blocks UI)
analytics.track('node.created', {
  userId, tenantId, entityType: 'node', mapId,
  properties: { hasAiContent: false, nodeType: 'default' }
});

// Server-side for sensitive events
await analyticsService.track('billing.plan.upgraded', ctx);
```

### 13.2 Workspace Analytics Dashboard

| Element | Description | RBAC |
|---------|-------------|------|
| Active Users | DAU/WAU/MAU chart. Trend line. | analytics.view.workspace |
| Feature Adoption | Which features are used, by what % of users. | analytics.view.workspace |
| Retention | N-day retention cohort table. | analytics.view.workspace |
| Search Queries | Top searches, zero-result queries. | analytics.view.workspace |
| Support Volume | Tickets over time, resolution time, CSAT. | analytics.view.workspace |
| AI Usage | Requests, tokens consumed, cost estimate. | analytics.view.workspace, flag: ai.analytics |

### 13.3 System Analytics (`system_owner`)

| Element | Description | RBAC |
|---------|-------------|------|
| Revenue | MRR, ARR, churn, LTV. Per-plan breakdown. | analytics.view.system |
| Tenant Growth | New workspaces over time. | analytics.view.system |
| Infrastructure | API latency P50/P95/P99. Error rate. Queue depth. | analytics.view.system |
| MCP Health | Per-provider success rate, latency, cost. | analytics.view.system |
| Security | Failed login attempts, session anomalies, rate-limit hits. | analytics.view.system |

### 13.4 Analytics States

| State | Description |
|-------|-------------|
| loading | Skeleton charts. |
| loaded | Data rendered. Last-updated timestamp shown. |
| empty | Not enough data. |
| error | Service unavailable. Stale data shown. |
| date-range | User adjusts date range. |
| export | CSV/PDF generated. Download link shown. |

---

## 14. FILE & STORAGE MANAGEMENT

File storage is an MCP capability: `storage.upload`, `storage.delete`, `storage.get_url`.

### 14.1 Canonical Schema Reference

**The canonical schema for file storage is defined in §23.3 (`file_uploads` table).** This section describes usage patterns and MCP integration.

Product modules **MUST NOT** define alternate file storage schemas. They **MAY** extend the canonical schema via:
- `form_id`, `submission_id`, `draft_id` columns for Form Builder linkage
- `linked_entity_type` and `linked_entity_id` for GeneralApp entity linkage
- `client_id` for consumer-module record linkage where applicable
- `access_roles` array for module-specific permission overlays
- MCP capability bindings for storage provider selection

### 14.2 TypeScript Projection

```typescript
// Projection of §23.3 for TypeScript convenience
// This is NOT a redefinition - all durable storage uses §23.3 SQL schema
interface StoredFile {
  id: string;
  tenantId: string;
  uploadedBy: string;
  originalName: string;
  mimeType: string;
  sizeBytes: number;
  storageKey: string;       // Maps to storage_key in §23.3
  isPublic: boolean;        // Maps to is_public in §23.3
  accessRoles: string[];    // Maps to access_roles in §23.3
  previewUrlExpiresAt: Date | null;
  scanStatus: 'pending' | 'clean' | 'quarantined' | 'error'; // Maps to scan_status in §23.3
  deletedAt: Date | null;
  // Product modules use form_id/submission_id/draft_id (FormBuilder), client_id (consumer record linkage where applicable), or linked_entity_type/linked_entity_id (GeneralApp) for linkage
}
```

**Key Rules:**
- Signed URLs are generated on-demand via MCP `storage.get_url` - never stored in database
- `scan_status` values: `pending` → `clean` (available) OR `quarantined` (threat detected) OR `error` (scan failure)
- Storage path convention: `{tenant_id}/{source_type}/{entity_id}/{file_id}.{ext}`

### 14.3 Upload Flow

1. Client requests upload URL: POST /files/upload-url { fileName, mimeType, sizeBytes, purpose }.
2. RBAC_BRANCH: storage.upload evaluated. Tenant quota checked.
3. If quota exceeded: 413 error.
4. Server generates pre-signed upload URL. Returns: { uploadUrl, fileId }.
5. Client PUTs file directly to storage provider.
6. Client calls POST /files/:id/complete.
7. Server queues virus scan job.
8. File available after virus scan returns 'clean'.

| State | Description |
|-------|-------------|
| idle | Upload zone in idle state. |
| drag-over | File dragged over zone. |
| validating | File type and size checked client-side. |
| type-error | File type not allowed. |
| size-error | File exceeds max size. |
| quota-exceeded | Tenant or user quota reached. |
| uploading | Progress bar. Cancel button. |
| scanning | Virus scan in progress. |
| quarantined | Threat detected. File quarantined. Admin alerted. |
| ready | File clean and available. |
| error | Upload or scan failed. |

### 14.4 Admin File Management

- Storage usage dashboard.
- Set per-tenant quota.
- Configure allowed MIME types globally.
- Configure MCP storage provider (S3, GCS, Cloudflare R2, Azure Blob).
- Enable/disable virus scanning.
- Quarantine management.
- Orphan cleanup.

---

## 15. INTERNATIONALISATION & LOCALISATION (i18n)

The application is built locale-first. Every string passes through the translation layer.

### 15.1 Locale Resolution Order

1. User Preference (explicit selection in Settings)
2. Browser Accept-Language header (on first visit)
3. Workspace default locale
4. System default locale (default: en-US)

### 15.2 What is Localised

| Category | Description |
|----------|-------------|
| UI Strings | All labels, buttons, messages, placeholders, error messages. |
| Email Templates | Each template exists in all supported locales. |
| Dates & Times | All timestamps in user's timezone. Format per locale. |
| Numbers | Decimal/thousands separators per locale. |
| Currency | Plan prices in user's local currency. |
| Right-to-Left | Arabic, Hebrew, Farsi: full RTL layout. |
| Pluralisation | Correct plural forms per locale. |

### 15.3 i18n States

| State | Description |
|-------|-------------|
| loaded | All strings for current locale loaded. |
| fallback | Current locale missing string. en-US used. Admin alerted. |
| missing-locale | Requested locale not supported. Fall back to en-US. |
| loading | Locale bundle fetching (lazy-loaded). |
| rtl-active | RTL locale selected. Layout direction flipped. |

### 15.4 Admin Configuration

- Enable/disable supported locales for the workspace.
- Set workspace default locale.
- Add custom translation overrides.
- Preview any locale in admin panel.
- Export missing translation report.

### 15.5 i18n Extension Point

Product modules can register custom translation strings that integrate with the platform i18n system.

**String Registration:**

```typescript
// Product module registers custom translations
i18nRegistry.registerNamespace({
  namespace: 'ext.my_module',
  defaultLocale: 'en-US',
  strings: {
    'en-US': {
      'task.title': 'Task Management',
      'task.create': 'Create Task',
      'task.name_placeholder': 'Enter task name...',
      'task.priority_high': 'High Priority',
      'task.priority_medium': 'Medium Priority',
      'task.priority_low': 'Low Priority'
    },
    'fr-FR': {
      'task.title': 'Gestion des tâches',
      'task.create': 'Créer une tâche',
      'task.name_placeholder': 'Entrez le nom de la tâche...',
      'task.priority_high': 'Haute priorité',
      'task.priority_medium': 'Priorité moyenne',
      'task.priority_low': 'Faible priorité'
    }
  }
});
```

**Database Storage:**

```sql
-- Extended i18n strings stored in i18n_strings table
INSERT INTO i18n_strings (namespace, locale, key, value, module) VALUES
('ext.my_module', 'en-US', 'task.title', 'Task Management', 'my_module'),
('ext.my_module', 'fr-FR', 'task.title', 'Gestion des tâches', 'my_module');
```

**Usage in Code:**

```typescript
// Use extended translations (same API as core)
const title = i18n.t('ext.my_module:task.title');
const placeholder = i18n.t('ext.my_module:task.name_placeholder');

// With interpolation
const message = i18n.t('ext.my_module:task.created', { 
  taskName: 'Deploy API',
  userName: 'John' 
});
```

**Translation Discovery:**

```typescript
// List all namespaces including extended
GET /api/ext/i18n-namespaces

// Response
{
  "core": ["common", "auth", "settings"],
  "extensions": [
    {
      "namespace": "ext.my_module",
      "module": "my_module",
      "locales": ["en-US", "fr-FR", "de-DE"],
      "stringCount": 42
    }
  ]
}

// Export strings for translation
GET /api/ext/i18n/export?namespace=ext.my_module&format=json
```

**Admin Translation Override:**

Admins can override extended translations via the same interface as core translations:

```typescript
// Override in admin panel
POST /api/admin/i18n/override
{
  "namespace": "ext.my_module",
  "locale": "en-US",
  "key": "task.title",
  "value": "Project Tasks"
}
```

---

## 16. THEMING & BRANDING

The theming system governs visual appearance at three layers: system defaults, workspace brand customisation, and user preferences.

### 16.1 Theme Modes

| Mode | Description |
|------|-------------|
| light | Default. White backgrounds, dark text. |
| dark | Dark backgrounds, light text. |
| system | Follows OS preference. |
| high-contrast | Accessibility mode. Increased contrast. |
| custom | Workspace-branded theme. Tenant colours injected. |

### 16.2 Workspace Brand Customisation

| Element | States | Interconnects / Branches |
|---------|--------|--------------------------|
| Logo | idle, uploading, active, removed | FILE_BRANCH (storage.upload) |
| Primary Colour | idle, picker-open, valid, invalid | Contrast ratio validator |
| Accent Colour | idle, picker-open, valid, invalid | Same contrast validator |
| Favicon | idle, uploading, active | FILE_BRANCH |
| App Name Override | idle, editing, saved, error | Replaces app name |
| Custom CSS | idle, editing, preview, saved, syntax-error | `system_owner` only by default; may be delegated to `super_admin`. RBAC: theme.customCss |

**Contrast Validator:** When a colour is picked, WCAG AA contrast ratio is checked. Warning if ratio < 4.5:1.

### 16.3 Theme States

| State | Description |
|-------|-------------|
| loading | Theme bundle fetching. System light theme shown. |
| applied | Theme tokens injected. Full brand visible. |
| user-override | User has chosen different mode from workspace default. |
| conflict | Custom CSS conflicts with user high-contrast. High-contrast wins. |

---

## 17. NOTIFICATION SYSTEM

Notifications exist at every scope. Emitted by elements, delivered through channels, configured by users and admins.

### 17.1 Notification Entity

```typescript
interface Notification {
  id: string;
  tenantId: string;
  userId: string;         // recipient
  actorId: string | null; // who triggered it
  type: string;           // e.g. 'ticket.reply'
  title: string;
  body: string;
  actionUrl: string | null;
  actionLabel: string | null;
  channels: ('in-app' | 'email' | 'sms')[];
  priority: 'low' | 'normal' | 'high' | 'critical';
  readAt: Date | null;
  dismissedAt: Date | null;
  createdAt: Date;
}
```

### 17.2 Notification Priority & Channel Rules

| Priority | Channels | Description |
|----------|----------|-------------|
| critical | in-app + email | Always delivered. Cannot be muted. |
| high | in-app + email | Default. User can disable email. |
| normal | in-app only | Default. User can add email. |
| low | in-app only | User can disable entirely. |

### 17.3 Delivery Channels

| Channel | Description |
|---------|-------------|
| in-app | Real-time via WebSocket. Persisted. Bell icon badge. |
| email | Queued via EMAIL_BRANCH. Digest mode available. |
| sms | High and critical only. Via SMS_BRANCH. |
| push | Optional. MCP: push.notify. Mobile app only. |

### 17.4 Notification Panel States

| State | Description |
|-------|-------------|
| empty | No notifications. Illustration + 'You are all caught up'. |
| has-unread | Bell badge shows count. Panel shows unread items. |
| loading | Panel opening. Skeleton items. |
| mark-all-read | Button at top. Badge clears. |
| infinite-scroll | User scrolls to bottom. Older notifications load. |
| action-clicked | Notification marked read. Navigates to resource. |

### 17.5 Admin Broadcast Notification

| State | Description |
|-------|-------------|
| compose | Admin writes title, body, optional CTA URL. |
| targeting | Select audience: all, role, or individual. |
| sending | POST /admin/notifications/broadcast. |
| sent | Delivery count shown. |
| failed | Delivery to some users failed. |

### 17.6 User Notification Preferences

- Per notification type: on/off per channel.
- Digest mode: daily or weekly.
- Quiet hours: suppress non-critical notifications.
- Do Not Disturb: temporary mute.

### 17.7 Notification Extension Point

Product modules can register custom notification types that integrate with the platform notification system.

**Notification Registration:**

```typescript
// Product module registers custom notification types
notificationRegistry.registerType({
  typeKey: 'ext.my_module.task_assigned',
  displayName: 'Task Assigned',
  category: 'My Module',
  description: 'Notified when a task is assigned to you',
  channels: ['in_app', 'email'],  // Supported channels
  defaultPreferences: {
    in_app: true,
    email: true,
    digest: false  // Don't include in digests
  },
  priority: 'normal',  // normal | high | urgent
  template: {
    in_app: {
      title: 'ext.my_module:notification.task_assigned.title',
      body: 'ext.my_module:notification.task_assigned.body'
    },
    email: {
      subject: 'ext.my_module:email.task_assigned.subject',
      bodyTemplate: 'task_assigned_email'
    }
  },
  dataSchema: {
    type: 'object',
    properties: {
      task_id: { type: 'string' },
      task_name: { type: 'string' },
      assigner_name: { type: 'string' }
    }
  }
});
```

**Sending Custom Notifications:**

```typescript
// Send custom notification
await notification.send({
  type: 'ext.my_module.task_assigned',
  userId: 'user-123',
  data: {
    task_id: 'task-456',
    task_name: 'Review API documentation',
    assigner_name: 'Jane Smith'
  },
  priority: 'high'
});
```

**Notification Discovery:**

```typescript
// List all notification types including extended
GET /api/ext/notification-types

// Response
{
  "core": [...],
  "extensions": [
    {
      "typeKey": "ext.my_module.task_assigned",
      "displayName": "Task Assigned",
      "module": "my_module",
      "channels": ["in_app", "email"]
    }
  ]
}
```

---

## 18. ACCESSIBILITY (A11Y)

Accessibility is not a feature — it is a baseline requirement. Every element must meet WCAG 2.1 AA.

### 18.1 Universal A11y States

| State | Description |
|-------|-------------|
| focus-visible | All interactive elements show clear focus indicator. |
| aria-label | Every icon-only button has aria-label. |
| aria-live | Status messages announced to screen readers. |
| aria-expanded | Dropdowns, accordions carry aria-expanded. |
| aria-disabled | Disabled elements carry aria-disabled. |
| error-association | Form error messages linked via aria-describedby. |
| skip-nav | 'Skip to main content' link first focusable element. |
| table-headers | Data tables use <th scope>. |

### 18.2 Reduce Motion

When user has 'Reduce Motion' enabled:
- All animations disabled or instant.
- Confetti, progress animations replaced with static.
- Auto-advancing carousels pause.

### 18.3 Keyboard Navigation

| Key | Action |
|-----|--------|
| Tab | Moves forward through interactive elements. |
| Shift+Tab | Moves backward. |
| Enter/Space | Activates buttons, links, checkboxes. |
| Escape | Closes modal, popover, dropdown. |
| Arrow keys | Navigate within lists, menus, radio groups. |
| Cmd/Ctrl+K | Opens global search. |
| Focus trap | Modals trap focus within bounds. |

### 18.4 Admin A11y Configuration

- Enable/disable high-contrast theme workspace-wide.
- Enforce large text minimum (18px).
- Accessibility audit log: periodic WCAG scan results.

---

## 19. LOGOUT, ACCOUNT SECURITY & SECURITY CENTRE

Logout is not just clearing a cookie. Full session termination flow with audit trail.

### 19.1 Logout Flow

**Standard Logout:**
1. User clicks Avatar > Logout.
2. If unsaved changes: navigation guard fires.
3. POST /auth/logout → session invalidated → WebSocket disconnected.
4. Client clears local state.
5. Redirect to /login.

| State | Description |
|-------|-------------|
| idle | Logout button visible in menu. |
| unsaved-guard | Navigation guard modal. |
| logging-out | POST in flight. Brief spinner overlay. |
| logged-out | Session invalidated. Login screen. |
| force-logged-out | Server-initiated (admin session termination). |
| session-expired | TTL elapsed. 'Session expired.' |

### 19.2 Security Centre (User View)

Profile > Security. Consolidates all account security information.

| Element | States | Interconnects / Branches |
|---------|--------|--------------------------|
| Password | last-changed date, change button | AUTH + EMAIL_BRANCH |
| Two-Factor Auth | status, method, backup codes remaining, setup/disable | AUTH_BRANCH |
| Active Sessions | list all, terminate any | SESSION_BRANCH |
| Login History | last 20 logins: date, IP, location, device, outcome | AUDIT_BRANCH |
| Connected Apps | OAuth apps with access, revoke | APIKEY / OAuth |
| Security Alerts | recent security events | NOTIF_BRANCH |
| Account Recovery | backup email, phone, recovery codes | EMAIL + SMS_BRANCH |
| Data Export | download all personal data | STORAGE_BRANCH |
| Delete Account | multi-step deletion | AUTH + EMAIL_BRANCH |

### 19.3 Login History States

| State | Description |
|-------|-------------|
| loading | GET /auth/login-history. Skeleton rows. |
| loaded | Rows: date/time, IP, location, device, method, outcome. |
| suspicious-flag | Row with 'Suspicious' badge. |
| not-me | User clicks 'Not me'. → Terminate all sessions. → EMAIL_BRANCH. |

### 19.4 Admin Security Actions

- Force password reset.
- Force 2FA setup.
- Revoke all API keys for a user.
- View full login history for any user.
- View security events across workspace.
- Security health score for workspace.

### 19.5 Browser Security Headers & Sensitive Response Rules

All web surfaces built on this platform inherit a shared browser-security baseline. Product modules may add stricter headers, but they must not weaken these defaults.

| Header / Control | Baseline Rule |
|------------------|---------------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` on production HTTPS deployments |
| `Content-Security-Policy` | Default deny-by-default posture: `default-src 'self'`; script sources limited to trusted first-party origins/nonces; `object-src 'none'`; `base-uri 'self'`; `frame-ancestors 'none'` unless an explicitly documented embed flow overrides this for a public form surface |
| `X-Content-Type-Options` | Always `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `X-Frame-Options` | `DENY` for non-embed surfaces; embed-capable surfaces rely on CSP `frame-ancestors` allowlisting instead of looser global framing |
| `Cache-Control` on sensitive pages | `no-store` for auth pages, security settings, impersonation sessions, checkout confirmation pages, data export links, and any response containing one-time tokens or signed URLs |
| Cookie flags | Session and refresh cookies are `HttpOnly`, `Secure`, and `SameSite=Strict` unless a documented same-site flow requires `Lax` |

**Rendering rule:** user-controlled data must be escaped or sanitised for its output context. HTML-capable surfaces (rich text, templates, generated documents, previews) must use an allowlist sanitisation strategy rather than trusting stored markup.

### 19.6 Resource Authorization, Redirect Safety & Write-Path Hardening

Shared platform endpoints follow these defensive rules regardless of module:

- **Resource authorization at the data layer:** every read/write operation validates tenant scope, ownership, and role authorization against the target resource itself. Route-level gating alone is never sufficient.
- **Enumeration resistance:** where revealing resource existence would leak sensitive information, unauthorized access resolves as not found (`404`) rather than disclosing the presence of another tenant's or user's record.
- **Mass-assignment prevention:** create/update endpoints use explicit server-side allowlists of writable fields. Unknown, privileged, or system-managed attributes are ignored or rejected; request bodies are never blindly merged into persisted entities.
- **Redirect safety:** user-influenced redirect targets are relative-path only by default. Absolute URLs require explicit allowlisting. `javascript:`, `data:`, protocol-relative (`//...`), credential-bearing, and malformed double-encoded targets are always rejected.
- **Path safety:** file-system and object-storage paths are derived from server-generated identifiers and canonical path builders. User input may influence metadata, but never the trusted storage path root.

---

## 20. CHANGELOG & IN-APP PRODUCT UPDATES

The Changelog is an admin-published feed of product updates, shown in-app.

### 20.1 Changelog Entry

```typescript
interface ChangelogEntry {
  id: string;
  title: string;
  body: string;           // Markdown
  type: 'new' | 'improvement' | 'fix' | 'deprecation';
  publishedAt: Date | null; // null = draft
  audience: {
    plans: string[];
    roles: string[];
    tenantIds: string[];
  };
  notifyEmail: boolean;
}
```

### 20.2 States

| State | Description |
|-------|-------------|
| new-badge | User has unread entries. Badge shows count. |
| read | User opened Changelog panel. Badge clears. |
| empty | No entries for this user's audience. |
| draft | Admin-only. Entry saved but not published. |
| published | Visible to targeted audience. NOTIF_BRANCH: in-app notification. |
| email-sent | EMAIL_BRANCH: changelog-digest sent on publish. |

---

## 21. MAINTENANCE MODE

Maintenance mode takes the application offline for non-admin users.

### 21.1 Maintenance States

| State | Description |
|-------|-------------|
| off | Normal operation. |
| scheduled | Future maintenance window. Countdown banner shown. |
| active | Non-admin users see maintenance page. API returns 503. |
| ending | Admin disabling maintenance. Final checks. |
| ended | Normal operation resumed. |

### 21.2 Enabling Maintenance Mode

1. `system_owner` navigates to System Settings > Maintenance.
2. Sets: start time, estimated duration, custom message.
3. Clicks 'Enable Maintenance Mode'.
4. Confirmation modal: 'Type CONFIRM to proceed.'
5. On confirm: all non-admin sessions invalidated. 503 middleware activated.

### 21.3 Admin Bypass

Admins bypass maintenance via special header token or valid admin session.
Admin banner shown during maintenance: 'Maintenance mode is active. N users blocked.'

### 21.4 ISP Scan — Maintenance Mode

- **SCAN-04 RBAC:** maintenance.enable — `system_owner` only.
- **SCAN-05 EMAIL:** maintenance-scheduled and maintenance-complete templates required.
- **SCAN-07 AUDIT:** maintenance.enabled, maintenance.disabled logged.
- **SCAN-08 NOTIF:** In-app and email notifications on schedule and completion.
- **SCAN-12 FAILURE:** If flag fails to load → default to OFF.

---

## 22. PRESENCE & REAL-TIME INFRASTRUCTURE

Presence tracks which users are online, on which resource, and enables real-time features.

### 22.1 Presence States

| State | Description |
|-------|-------------|
| online | User has active WebSocket. Green indicator. |
| away | User connected but idle > 5 min. Yellow indicator. |
| offline | No WebSocket. Grey indicator. Last-seen shown. |
| do-not-disturb | User has DND enabled. Purple indicator. |

### 22.2 WebSocket Events

| Event | Description |
|-------|-------------|
| presence.join | User connects. Broadcast to room. |
| presence.leave | User disconnects. Last-seen updated. |
| entity.updated | Record was updated by another user. Diff applied. |
| notification.push | New notification delivered. |
| session.terminated | Admin terminated session. Client receives → force logout. |
| maintenance.start | Maintenance mode activated. |

### 22.3 Reconnection Strategy

- Exponential back-off: 1s, 2s, 4s, 8s, 16s, 30s (cap). Jitter applied.
- On reconnect: re-subscribe to all rooms. Fetch missed diffs.
- Offline queue: mutations made while disconnected are queued. Replayed on reconnect.
- Conflict resolution: last-write-wins for positional; operational transform for text.

**ISP:** SCAN-02 SESSION: WebSocket connection validated on handshake. SCAN-03 TENANT: Socket rooms namespaced with tenantId. SCAN-09 RATE-LIMIT: 100 messages/second per connection.


---

## 23. CANONICAL ENTITY SCHEMAS (Source of Truth)

This section defines the authoritative schemas for all shared platform entities. Form Builder and all product modules **MUST** reference these definitions and **MUST NOT** redefine them.

### 23.1 users Table (Foundation Identity Contract; Supabase Auth Default Adapter)

The `users` table stores profile and authorization metadata. **Credential storage (password hashes) is delegated to the active foundation auth provider**. In the default deployment profile, that provider is Supabase Auth. This table does NOT store passwords.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY, -- Matches the active foundation auth provider user ID in the default direct-mapping profile
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    primary_role_id UUID NULLABLE REFERENCES rbac_roles(id),
    tenant_id UUID NULLABLE REFERENCES tenants(id),
    -- 2FA Configuration (TOTP secrets encrypted, never plaintext)
    totp_secret_encrypted TEXT NULLABLE,
    totp_enabled BOOLEAN DEFAULT false,
    email_otp_enabled BOOLEAN DEFAULT false,
    sms_otp_enabled BOOLEAN DEFAULT false,
    backup_codes VARCHAR(255)[] DEFAULT '{}', -- Array of bcrypt hashes
    -- Session/Security
    failed_login_attempts INTEGER DEFAULT 0,
    locked_until TIMESTAMP NULLABLE,
    last_login_at TIMESTAMP NULLABLE,
    last_login_ip INET NULLABLE,
    -- Account lifecycle
    is_active BOOLEAN DEFAULT true,
    deleted_at TIMESTAMP NULLABLE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

**Critical Rules:**
- `password_hash` field **MUST NOT** exist in this table — credentials belong to the active auth provider
- the active auth provider **MUST** map its subject ID to canonical `users.id`
- when the default Supabase deployment profile is active, `id` **MUST** match Supabase Auth `user.id` exactly
- when the default Supabase deployment profile is active, RLS policies use `auth.uid()` from Supabase JWT

### 23.2 sessions Table

Active and historical sessions keyed on JWT JTI for revocation support.

```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    jti_hash VARCHAR(64) UNIQUE NOT NULL, -- HMAC-SHA256 of JWT ID, not raw JTI
    device_fingerprint VARCHAR(255),
    user_agent TEXT,
    ip_address INET,
    geo_country VARCHAR(2),
    geo_city VARCHAR(100),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    last_active_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP NOT NULL, -- JWT expiry
    revoked_at TIMESTAMP NULLABLE,
    revoked_reason VARCHAR(50) NULLABLE -- 'logout', 'logout_all', 'admin_revoke', 'password_change', 'email_change', 'security_event'
);

CREATE INDEX idx_sessions_user_active ON sessions(user_id, is_active) WHERE is_active = true;
CREATE INDEX idx_sessions_jti ON sessions(jti_hash);
```

### 23.3 file_uploads Table (Unified Storage)

Single table for all files across all modules. Supports polymorphic linkage to multiple entity types simultaneously (e.g., a form submission file linked to a client record).

```sql
CREATE TABLE file_uploads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    uploaded_by UUID NOT NULL REFERENCES users(id),
    
    -- Module source tracking
    source_type VARCHAR(50) NOT NULL CHECK (source_type IN ('general_app', 'form_builder', 'product_module')),
    
    -- Consumer-module linkage (nullable, populated when a product projects a client/account record onto the shared file row)
    client_id UUID NULLABLE,
    
    -- Form Builder linkage (nullable, populated when file is from a form)
    form_id UUID NULLABLE,              -- References FormDefinition
    submission_id UUID NULLABLE,        -- References FormSubmission
    draft_id UUID NULLABLE,             -- References FormDraft (save-and-resume)
    field_name VARCHAR(255) NULLABLE,   -- Field that stored this file
    
    -- GeneralApp direct linkage (nullable)
    linked_entity_type VARCHAR(50) NULLABLE,  -- 'avatar', 'export', 'kb_article', etc.
    linked_entity_id UUID NULLABLE,           -- Generic FK for GeneralApp entities
    
    -- Storage reference
    storage_key VARCHAR(500) NOT NULL, -- Path in storage provider (e.g., S3 key)
    original_filename VARCHAR(255) NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_size_bytes INTEGER NOT NULL,
    checksum VARCHAR(64) NULLABLE, -- SHA-256 for integrity verification
    
    -- Connector reference
    connector_id UUID REFERENCES connectors(id),
    
    -- Security scanning
    scan_status VARCHAR(20) DEFAULT 'pending' CHECK (scan_status IN ('pending', 'clean', 'quarantined', 'error')),
    scanned_at TIMESTAMP NULLABLE,
    
    -- Access control (product modules may overlay additional restrictions)
    is_public BOOLEAN DEFAULT false,
    access_roles VARCHAR(50)[] DEFAULT '{}',
    
    -- Lifecycle
    deleted_at TIMESTAMP NULLABLE,
    delete_reason VARCHAR(50) NULLABLE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for common access patterns
CREATE INDEX idx_file_uploads_tenant ON file_uploads(tenant_id);
CREATE INDEX idx_file_uploads_client ON file_uploads(client_id) WHERE client_id IS NOT NULL;
CREATE INDEX idx_file_uploads_form ON file_uploads(form_id) WHERE form_id IS NOT NULL;
CREATE INDEX idx_file_uploads_submission ON file_uploads(submission_id) WHERE submission_id IS NOT NULL;
CREATE INDEX idx_file_uploads_draft ON file_uploads(draft_id) WHERE draft_id IS NOT NULL;
CREATE INDEX idx_file_uploads_scan_status ON file_uploads(scan_status);
CREATE INDEX idx_file_uploads_source ON file_uploads(source_type, linked_entity_type, linked_entity_id) 
    WHERE linked_entity_id IS NOT NULL;
```

**Storage Path Convention:**
```
{tenant_id}/{source_type}/{entity_id}/{file_id}.{ext}
```

**Polymorphic Linkage Semantics:**
- `client_id` set → File visible to the linked consumer record via product-module authorization rules
- `form_id` set → File linked to a specific form definition
- `submission_id` set → File linked to a form submission (also set `form_id`)
- `draft_id` set → File is part of a save-and-resume draft
- Multiple linkages may be set simultaneously (e.g., form submission auto-linked to client)
- `source_type='product_module'` is the neutral token for direct uploads initiated by consuming product modules; product specs should not introduce per-module `source_type` enum variants unless the shared foundation contract is formally expanded.

**Signed URLs:** Generated on-demand via MCP `storage.get_url` capability. Never stored in database.

### 23.4 audit_log Table (Append-Only)

Immutable audit trail for all state-changing operations across all modules.

```sql
CREATE TABLE audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    actor_id UUID NULLABLE REFERENCES users(id),
    actor_role VARCHAR(50) NULLABLE,
    action VARCHAR(100) NOT NULL, -- Namespace: 'ga.*', 'fb.*', 'cp.*', 'ext.*' (see §2.4)
    target_type VARCHAR(50) NULLABLE, -- Table/entity name
    target_id UUID NULLABLE,
    metadata JSONB NULL, -- Action-specific details (old/new values, reasons)
    ip_address INET NULLABLE,
    user_agent TEXT NULLABLE,
    created_at TIMESTAMP DEFAULT NOW() -- Immutable
);

-- Append-only enforcement: No UPDATE or DELETE policies
```

**Action Naming Convention (aligned with §2.4 Namespace Conventions):**
- `ga.*` - GeneralApp/platform-level actions (auth, settings, tenant management)
- `fb.*` - Form Builder actions (defined in FormBuilder spec)
- `pm.*` - Product-module actions (defined in consuming product specs)
- `ext.{vendor}.*` - Third-party extension actions

**Legacy/Deprecated:** Old `system.*`, `form.*`, `client.*` prefixes should be migrated to the canonical prefixes above.

#### 23.4.1 Audit Action Extension Point

Product modules can log custom audit actions that integrate with the platform audit system.

**Action Registration:**

```typescript
// Product module registers custom audit actions
auditRegistry.registerActions([
  {
    actionKey: 'ext.my_module.task.created',
    description: 'Task created in My Module',
    category: 'My Module',
    targetType: 'ext_my_module_task',  // Custom entity type
    metadataSchema: {
      type: 'object',
      properties: {
        task_name: { type: 'string' },
        assignee_id: { type: 'string' },
        priority: { type: 'string', enum: ['low', 'medium', 'high'] }
      }
    }
  },
  {
    actionKey: 'ext.my_module.workflow.completed',
    description: 'Workflow completed',
    category: 'My Module',
    targetType: 'ext_my_module_workflow'
  }
]);
```

**Logging Custom Actions:**

```typescript
// Log custom audit action
await audit.log({
  action: 'ext.my_module.task.created',
  actor_id: 'user-123',
  actor_role: 'admin',
  target_type: 'ext_my_module_task',
  target_id: 'task-456',
  metadata: {
    task_name: 'Deploy to production',
    assignee_id: 'user-789',
    priority: 'high'
  },
  ip_address: req.ip,
  user_agent: req.headers['user-agent']
});
```

**Action Discovery:**

```typescript
// List all audit actions including extended
GET /api/ext/audit-actions

// Response
{
  "core": [
    { "key": "system.user.created", "description": "..." },
    { "key": "ga.workspace.suspended", "description": "..." }
  ],
  "extensions": [
    {
      "key": "ext.my_module.task.created",
      "module": "my_module",
      "description": "Task created in My Module",
      "category": "My Module"
    }
  ]
}
```

**Querying Extended Actions:**

```sql
-- Query audit log for custom actions
SELECT * FROM audit_log 
WHERE action LIKE 'ext.my_module.%'
  AND created_at > NOW() - INTERVAL '7 days'
ORDER BY created_at DESC;
```

### 23.5 connectors & capability_bindings Tables (MCP Registry)

MCP connector configuration and capability routing.

```sql
CREATE TABLE connectors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    connector_type VARCHAR(50) NOT NULL, -- 'smtp', 'resend', 'stripe', 's3', 'supabase_storage', etc.
    capabilities VARCHAR(100)[] NOT NULL, -- Capability keys this connector fulfills
    config JSONB NOT NULL, -- AES-256-GCM encrypted configuration
    is_active BOOLEAN DEFAULT true,
    priority INTEGER DEFAULT 1, -- Display ordering only
    last_tested_at TIMESTAMP,
    last_test_result JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE capability_bindings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    capability VARCHAR(100) NOT NULL UNIQUE, -- e.g., 'email.send', 'payment.create_checkout'
    primary_connector_id UUID NOT NULL REFERENCES connectors(id),
    fallback_connector_id UUID NULLABLE REFERENCES connectors(id),
    is_enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    CHECK (primary_connector_id IS DISTINCT FROM fallback_connector_id)
);
```

### 23.6 RBAC Tables

Role-based access control primitives.

```sql
CREATE TABLE rbac_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    role_key VARCHAR(50) NOT NULL UNIQUE, -- 'system_owner', 'super_admin', 'admin', etc.
    name VARCHAR(255) NOT NULL,
    description TEXT,
    is_system_reserved BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE role_permissions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    role_id UUID NOT NULL REFERENCES rbac_roles(id),
    action_key VARCHAR(100) NOT NULL, -- e.g., 'form.create', 'client.view'
    conditions JSONB NULLABLE, -- Optional conditions (JSON schema)
    UNIQUE(role_id, action_key)
);

CREATE TABLE user_role_assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    role_id UUID NOT NULL REFERENCES rbac_roles(id),
    is_primary BOOLEAN DEFAULT false,
    assigned_by UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, role_id)
);
```

#### 23.6.1 Unified RBAC Permission Registry

**Naming Convention:** `{module}.{resource}.{action}`

**Boundary note:** this registry is the canonical namespace for action keys across the platform. Listing `fb.*` and `pm.*` permissions here does **not** make GeneralApp the owner of FormBuilder or product-module business logic; it only standardizes permission names used by the shared RBAC engine.

| Permission | Module | Description | Default Role |
|------------|--------|-------------|--------------|
| **GeneralApp Permissions** |
| `ga.workspace.create` | GeneralApp | Create new workspace | system_owner |
| `ga.workspace.delete` | GeneralApp | Delete workspace | system_owner |
| `ga.workspace.suspend` | GeneralApp | Suspend workspace | system_owner |
| `ga.user.invite` | GeneralApp | Invite users to workspace | super_admin |
| `ga.user.delete` | GeneralApp | Delete user accounts | system_owner |
| `ga.session.view.own` | GeneralApp | View own sessions | All |
| `ga.session.view.any` | GeneralApp | View any user's sessions | super_admin |
| `ga.session.terminate.any` | GeneralApp | Terminate any session | super_admin |
| `ga.settings.read.system` | GeneralApp | Read system settings | system_owner |
| `ga.settings.write.system` | GeneralApp | Modify system settings | system_owner |
| `ga.audit.view` | GeneralApp | View audit log | super_admin |
| `ga.webhook.create` | GeneralApp | Create webhook endpoints | super_admin |
| `ga.apikey.create` | GeneralApp | Create API keys | admin |
| **FormBuilder Permissions** |
| `fb.form.create` | FormBuilder | Create new forms | admin |
| `fb.form.edit` | FormBuilder | Edit existing forms | admin |
| `fb.form.delete` | FormBuilder | Delete forms | super_admin |
| `fb.form.archive` | FormBuilder | Archive forms | admin |
| `fb.form.publish` | FormBuilder | Publish forms | admin |
| `fb.form.system.edit` | FormBuilder | Edit system forms | system_owner |
| `fb.submissions.view` | FormBuilder | View form submissions | admin |
| `fb.submissions.export` | FormBuilder | Export submissions | admin |
| `fb.submissions.export.elevated` | FormBuilder | Export with sensitive data | system_owner |
| `fb.field.customCss` | FormBuilder | Use custom CSS on fields | system_owner |
| **Product Module Permissions (Example Namespace)** |
| `pm.record.view` | Product Module | View product-owned records | admin |
| `pm.record.create` | Product Module | Create product-owned records | admin |
| `pm.record.delete` | Product Module | Delete product-owned records | super_admin |
| `pm.billing.manage` | Product Module | Manage module billing flows | super_admin |
| `pm.service.provision` | Product Module | Provision module-owned services/resources | admin |

**Permission Inheritance:**
- `system_owner` → All permissions implicitly
- `super_admin` → All permissions except `system_owner`-reserved
- `admin` → Workspace operational permissions
- `client` → Client-facing permissions only

#### 23.6.2 RBAC Extension Point

Product modules can dynamically register custom permissions that integrate with the platform RBAC engine.

**Registration Pattern:**

```typescript
// Product module registers custom permissions
rbacRegistry.registerPermissions([
  {
    actionKey: 'ext.my_module.resource.view',
    description: 'View custom resource',
    category: 'My Module',
    defaultRoles: ['admin', 'super_admin']
  },
  {
    actionKey: 'ext.my_module.resource.manage',
    description: 'Manage custom resource',
    category: 'My Module',
    defaultRoles: ['super_admin'],
    conditions: {              // Optional JSON schema conditions
      type: 'object',
      properties: {
        department: { type: 'string' }
      }
    }
  }
]);
```

**Dynamic Permission Storage:**

```sql
-- Extended permissions are stored in role_permissions with ext. prefix
INSERT INTO role_permissions (role_id, action_key, conditions) VALUES
('role-admin-id', 'ext.my_module.resource.view', NULL),
('role-admin-id', 'ext.my_module.resource.manage', '{"department": "engineering"}');
```

**Permission Discovery:**

```typescript
// List all permissions including extended
GET /api/ext/permissions

// Response includes core and extended permissions
{
  "core": [...],
  "extensions": [
    {
      "actionKey": "ext.my_module.resource.view",
      "module": "my_module",
      "description": "View custom resource",
      "category": "My Module"
    }
  ]
}
```

**Custom Role Creation:**

Product modules can create custom roles with mixed core and extended permissions:

```typescript
// Create custom role with extended permissions
POST /api/ext/roles
{
  "role_key": "my_module_operator",
  "name": "Module Operator",
  "permissions": [
    "ga.workspace.view",
    "ext.my_module.resource.view",
    "ext.my_module.resource.manage"
  ]
}
```

**Runtime Permission Check:**

```typescript
// Check includes extended permissions automatically
const allowed = await rbac.check({
  userId: 'user-123',
  action: 'ext.my_module.resource.manage',
  context: { department: 'engineering' }  // For condition evaluation
});
```

### 23.7 Webhook Endpoints (Shared Delivery Infrastructure)

Outbound webhook configuration. Product modules register endpoints and events they emit.

```sql
CREATE TABLE webhook_endpoints (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(500) NOT NULL, -- HTTPS only
    signing_secret_encrypted TEXT NOT NULL, -- AES-256-GCM, raw secret for HMAC signing
    secret_fingerprint VARCHAR(32) NOT NULL, -- Non-sensitive ID for logs
    events VARCHAR(100)[] NOT NULL, -- Event keys this endpoint subscribes to
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

**Event Namespace:**
- `ga.*` - General Application events
- `fb.*` - Form Builder events  
- `pm.*` - Product-module events
- `ext.*` - Third-party extension events

#### 23.7.1 Webhook Event Extension Point

Product modules can register custom webhook events that integrate with the shared delivery infrastructure.

**Event Registration:**

```typescript
// Product module registers custom events
webhookRegistry.registerEvents([
  {
    eventKey: 'ext.my_module.task.completed',
    description: 'Task completed in My Module',
    payloadSchema: {
      type: 'object',
      properties: {
        task_id: { type: 'string' },
        task_name: { type: 'string' },
        completed_by: { type: 'string' },
        completed_at: { type: 'string', format: 'date-time' }
      },
      required: ['task_id', 'task_name']
    },
    category: 'My Module'
  },
  {
    eventKey: 'ext.my_module.alert.critical',
    description: 'Critical alert triggered',
    payloadSchema: { /* ... */ },
    category: 'Alerts',
    priority: 'high'  // High-priority delivery
  }
]);
```

**Event Emission:**

```typescript
// Emit custom event (delivered to all subscribed endpoints)
await webhook.emit('ext.my_module.task.completed', {
  task_id: 'task-123',
  task_name: 'Deploy production',
  completed_by: 'user-456',
  completed_at: new Date().toISOString()
});
```

**Event Subscription:**

```typescript
// Subscribe to custom events
POST /api/webhook-endpoints
{
  "name": "My Module Integration",
  "url": "https://example.com/webhooks",
  "events": ["ext.my_module.task.completed", "ext.my_module.alert.critical"],
  "signing_secret": "whsec_..."
}
```

**Event Discovery:**

```typescript
// List all available events including extended
GET /api/ext/events

// Response
{
  "core": [
    { "key": "ga.user.created", "description": "..." },
    { "key": "fb.form.submitted", "description": "..." }
  ],
  "extensions": [
    {
      "key": "ext.my_module.task.completed",
      "module": "my_module",
      "description": "Task completed in My Module",
      "category": "My Module"
    }
  ]
}
```

**Payload Delivery:**

```json
{
  "type": "ext.my_module.task.completed",
  "timestamp": "2026-03-14T12:00:00Z",
  "payload": {
    "task_id": "task-123",
    "task_name": "Deploy production",
    "completed_by": "user-456",
    "completed_at": "2026-03-14T12:00:00Z"
  }
}
```

### 23.8 Custom Entity Registry (Extension Point)

Product modules can register custom entity types that integrate with the platform's shared infrastructure (audit, webhooks, search, etc.).

**Entity Registration:**

```typescript
// Product module registers custom entity
entityRegistry.register({
  entityType: 'ext.my_module.project',
  displayName: 'Project',
  description: 'Project management entity',
  schema: {
    type: 'object',
    properties: {
      id: { type: 'string', format: 'uuid' },
      name: { type: 'string', maxLength: 255 },
      status: { type: 'string', enum: ['active', 'archived'] },
      owner_id: { type: 'string', format: 'uuid' },
      metadata: { type: 'object' }
    },
    required: ['name', 'status']
  },
  features: {
    auditable: true,        // Enable audit logging
    searchable: true,       // Enable global search
    webhookEmit: true,      // Emit lifecycle webhooks
    softDelete: true,       // Enable soft delete
    tenantScoped: true,     // Auto-scope to tenant
    rbacControlled: true    // Enable RBAC permissions
  },
  webhooks: {
    emitEvents: [
      'ext.my_module.project.created',
      'ext.my_module.project.updated',
      'ext.my_module.project.deleted'
    ]
  },
  rbac: {
    permissions: [
      'ext.my_module.project.view',
      'ext.my_module.project.create',
      'ext.my_module.project.edit',
      'ext.my_module.project.delete'
    ]
  }
});
```

**Auto-Generated Table:**

```sql
-- System auto-creates table for registered entity
CREATE TABLE ext_my_module_project (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL,
    name VARCHAR(255) NOT NULL,
    status VARCHAR(50) NOT NULL,
    owner_id UUID REFERENCES users(id),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    deleted_at TIMESTAMP NULLABLE,  -- For soft delete
    
    -- Auto-generated RLS policies
    CONSTRAINT fk_tenant FOREIGN KEY (tenant_id) REFERENCES tenants(id)
);

-- Auto-generated indexes
CREATE INDEX idx_ext_my_module_project_tenant ON ext_my_module_project(tenant_id);
CREATE INDEX idx_ext_my_module_project_status ON ext_my_module_project(status);
CREATE INDEX idx_ext_my_module_project_deleted ON ext_my_module_project(deleted_at) WHERE deleted_at IS NULL;
```

**Entity Discovery:**

```typescript
// List all registered entities including extended
GET /api/ext/entities

// Response
{
  "core": [
    { "type": "user", "displayName": "User" },
    { "type": "session", "displayName": "Session" }
  ],
  "extensions": [
    {
      "type": "ext.my_module.project",
      "displayName": "Project",
      "module": "my_module",
      "features": ["auditable", "searchable", "webhookEmit"]
    }
  ]
}
```

**Entity Storage:**

Custom entity data is stored in module-specific tables with full platform integration:

```typescript
// Create entity (auto-audited, webhooks emitted)
POST /api/ext/entities/ext.my_module.project
{
  "name": "Website Redesign",
  "status": "active",
  "owner_id": "user-123"
}

// Query with platform features
GET /api/ext/entities/ext.my_module.project?search=redesign&status=active
```

**Integration Benefits:**

| Platform Feature | Auto-Enabled for Custom Entities |
|------------------|----------------------------------|
| Audit Logging | All CRUD operations logged to `audit_log` |
| Webhooks | Lifecycle events emitted automatically |
| Global Search | Indexed and searchable via `/api/search` |
| RBAC | Permissions auto-generated and enforced |
| Soft Delete | `deleted_at` pattern with recovery support |
| Tenant Scoping | Automatic RLS policies per tenant |
| API Access | REST endpoints auto-generated |

### 23.9 Form-to-Entity Bridge (Cross-Module Integration)

The Form-to-Entity Bridge enables Form Builder submissions to create or update entities in product modules (for example CRM records, customer profiles, or other product-owned entities). This is a **runtime integration pattern** where Form Builder provides the extension mechanism and product modules register handlers.

**Bridge Configuration Schema:**
```typescript
interface FormBridgeConfig {
  enabled: boolean;
  targetSchemaId: string;           // Target entity definition ID (product-owned)
  targetEntityType: string;         // Product entity type key (e.g., 'client_profile')
  fieldMappings: BridgeFieldMapping[];
  productConfig: Record<string, any> | null;  // Opaque payload interpreted by product module
}

interface BridgeFieldMapping {
  formFieldName: string;            // Source field in FormDefinition
  targetFieldKey: string;           // Destination field in product entity
  transform?: 'none' | 'uppercase' | 'lowercase' | 'trim' | 'date_format';
}
```

**Integration Flow:**
```
1. Form Submission Received (FormBuilder)
   ↓
2. Validation & Standard Persistence
   ↓
3. Bridge Config Lookup (if enabled)
   ↓
4. Field Mapping & Transform
   ↓
5. Handler Invocation (registered by product module)
   ↓
6. Entity Created/Updated in Product Module
   ↓
7. Submission Marked with bridgeResult (success/failure)
```

**Handler Registration (Product Module Responsibility):**
```typescript
// Product module registers at runtime
formEngine.registerBridgeHandler('product-module-bridge', {
  async execute(submission, config, context) {
    // Product-specific logic: create/update module-owned record
    const entityId = await createRecordFromSubmission(submission, config);
    return { success: true, entityId, entityType: 'product_record' };
  }
});
```

**Event Emissions:**
| Event | Emitter | Description |
|-------|---------|-------------|
| `fb.submission.received` | FormBuilder | Form submitted |
| `fb.submission.processed` | FormBuilder | After-submit actions complete |
| `fb.bridge.executed` | FormBuilder | Bridge handler completed |
| `pm.record.created_from_form` | Product Module | Product record created via bridge |
| `pm.record.updated_from_form` | Product Module | Product record updated via bridge |
| `pm.payment.received_from_form` | Product Module | Payment linked to a product record from form |

**Handler Registration API (Canonical):**
```typescript
// FormBuilder exposes registration API
interface FormEngine {
  registerBridgeHandler(
    handlerKey: string,
    handler: BridgeHandler
  ): void;
}

interface BridgeHandler {
  execute(
    submission: FormSubmission,
    config: FormBridgeConfig,
    context: BridgeContext
  ): Promise<BridgeResult>;
}

interface BridgeContext {
  tenantId: string;
  formId: string;
  submittedBy: string | null;
  timestamp: Date;
}

interface BridgeResult {
  success: boolean;
  entityId?: string;        // ID of created/updated entity
  entityType?: string;      // Type of entity (e.g., 'client', 'service')
  error?: string;           // Error message if failed
  warnings?: string[];      // Non-fatal warnings
}
```

---

## 24. CROSS-SYSTEM INTEGRATION REFERENCE

This section documents General Application's role as the foundation for the unified platform comprising General App, Form Builder, and consumer product modules.

### 24.1 Platform Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     UNIFIED SAAS PLATFORM                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │                    GENERAL APPLICATION (This Doc)               │   │
│  │  ─────────────────────────────────────────────────────────────  │   │
│  │                                                                 │   │
│  │  Foundation Services:                                           │   │
│  │  • Tenant/Workspace management (§3)                            │   │
│  │  • Session management + auth authority contracts (§4, §23.1–§23.2) │   │
│  │  • Settings hierarchy (§5)                                     │   │
│  │  • File storage abstraction (§14, §23.3)                       │   │
│  │  • Cross-cutting concerns (search, analytics, webhooks)        │   │
│  │                                                                 │   │
│  │  Provides to other modules:                                     │   │
│  │  • Tenant context (tenant_id)                                  │   │
│  │  • User identity (auth.uid())                                  │   │
│  │  • Session management                                          │   │
│  │  • Settings resolution                                         │   │
│  │                                                                │   │
│  └────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│           ┌──────────────────┴──────────────────┐                       │
│           ▼                                      ▼                       │
│  ┌──────────────────┐                ┌──────────────────┐               │
│  │   CLIENT PORTAL  │                │   FORM BUILDER   │               │
│  │   (v6.1)         │◄──────────────►│   (v6.1)         │               │
│  │                  │   §23.9 Bridge │                  │               │
│  └──────────────────┘                └──────────────────┘               │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### 24.2 Module Responsibilities

| Capability | GeneralApp | ProductModule | FormBuilder |
|------------|------------|--------------|-------------|
| **Tenancy** | §3 - Platform authority (`system_owner`) | Consumer | Consumer |
| **User Auth** | §23.1 - Canonical `users` table; Supabase Auth integration | Consumer | Consumer |
| **Sessions** | §23.2 - Canonical `sessions` table | Consumer | Consumer |
| **Settings** | §5 - Hierarchy master | Tenant settings | Form settings |
| **Files** | §23.3 - Canonical `file_uploads` table | Consumer | Consumer |
| **Audit** | §23.4 - Canonical `audit_log` table | Consumer | Consumer |
| **MCP/Connectors** | §23.5 - `connectors` + `capability_bindings` | Consumer | Consumer |
| **RBAC** | §23.6 - `rbac_roles`, `role_permissions` | Consumer | Consumer |
| **Webhooks** | §23.7 - Delivery infrastructure | Consumer | Consumer |
| **Payments** | — | Business logic | Field type (via MCP) |
| **Forms** | — | Consumer (via extension hooks) | §2 - Provider |
| **Business Records** | — | Provider | — |

### 24.2.1 Platform Capability Adoption Matrix

This document defines platform capabilities that product modules may adopt selectively. The table below clarifies the intended status for the current unified product set so the platform doc does not compete with product-module authority.

| Capability | Current Status | Primary Runtime Owner |
|------------|----------------|-----------------------|
| Feature Flags | Active platform capability | GeneralApp |
| API Keys | Active platform capability | GeneralApp |
| Maintenance Mode | Active platform capability | GeneralApp |
| Knowledge Base | Optional support capability | GeneralApp / product support modules |
| Live Chat | Optional connector-backed capability | GeneralApp / support workflows |
| System Analytics | Active platform capability for `system_owner` scope | GeneralApp |
| Changelog / Product Updates | Optional but supported | GeneralApp |
| Presence / Realtime infrastructure | Active shared platform capability | GeneralApp |
| Shared file/storage contract | Active shared platform capability | GeneralApp + module projections |
| Shared webhook delivery infrastructure | Active shared platform capability | GeneralApp |

### 24.3 Data Flow Patterns

**Pattern 1: Tenant Context Propagation**
```
User Login (GeneralApp)
    ↓ JWT with tenant_id claim
All API Requests
    ↓ RLS policies use tenant_id
Database Records scoped to tenant
```

**Pattern 2: Settings Resolution**
```
System Default (GeneralApp)
    ↓ overridden by
Tenant Setting (Product Module)
    ↓ overridden by
Form Setting (FormBuilder)
    ↓ overridden by
User Preference (GeneralApp)
```

**Pattern 3: Form-to-Entity Creation**
```
Form Submission (FormBuilder)
    ↓ bridge configuration
Product Record Created (Product Module)
    ↓ field mapping
Client Field Values Populated
    ↓ optional
Payment Record Created
    ↓ optional
Service Record Created
```

### 24.4 Shared Entity References

| Entity | GeneralApp | ProductModule | FormBuilder |
|--------|------------|--------------|-------------|
| **User** | §2.2, §3.1 | §29.1 | Implied |
| **Session** | §3.1 | §4.6 | Implied |
| **Tenant** | §2.2 | `tenant_config` | `tenantId` field |
| **File** | §23.3 | Reference only | Reference only |
| **Settings** | §5 | Product settings surface | §14 |

### 24.5 Cross-Document Quick Links

| Topic | GeneralApp (Canonical) | Product Module | FormBuilder |
|-------|----------------------|---------------|-------------|
| **users** table | §23.1 | Reference only | Reference only |
| **sessions** table | §23.2 | Reference only | Reference only |
| **file_uploads** table | §23.3 | Reference only | Reference only |
| **audit_log** table | §23.4 | Reference only | Reference only |
| **connectors** + bindings | §23.5 | Reference only | Reference only |
| **RBAC** tables | §23.6 | Reference only | Reference only |
| **webhook_endpoints** | §23.7 | Reference only | Reference only |
| Tenancy/Settings | §3, §5 | Consumer | Consumer |
| Files (usage) | — | §12 | §24 |
| Forms | — | Consumer via §23.9 | Owner |
| Business Records | — | Owner | — |
| Payments | — | Owner | Field type |
| Bridge/Integration | — | §23.9 Consumer | Extension hooks |

### 24.6 Integration Checklist for Platform Setup

```
□ GeneralApp: Core Schema Setup
  □ §23.1 users table (via Supabase Auth)
  □ §23.2 sessions table
  □ §23.3 file_uploads table
  □ §23.4 audit_log table
  □ §23.5 connectors + capability_bindings tables
  □ §23.6 RBAC tables
  □ §23.7 webhook_endpoints table
  □ §5.5 System-level configuration
  □ §2.4 First tenant created
  
□ FormBuilder: Generic Engine Setup
  □ §2 FormDefinition schema
  □ §12 FormSubmission schema
  □ §9 Payment field configuration
  □ §27 CAPTCHA configuration
  □ §30 Theme defaults
  
□ ProductModule: Product Layer Setup
  □ Define product-owned core entities
  □ Define module-owned billing/service entities as needed
  □ Configure module-specific connector usage
  □ Define first-run/product bootstrap flows as needed
  
□ Integration: Form-to-Entity Bridge
  □ FormBuilder: Configure extension hooks
  □ ProductModule: Register bridge handlers
  □ Field mappings configured
```

---

*End of Document — General-Purpose Application v6.1 (Platform Foundation)*

*Canonical source for shared entities: §23*

*Part of Unified SaaS Platform. Cross-system integration specified in §24.*
