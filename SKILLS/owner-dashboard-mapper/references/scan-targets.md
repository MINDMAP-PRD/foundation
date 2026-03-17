# Scan Targets Reference

Full file lists, grep patterns, and recording rules for each Phase 1 scan category.
Each category ends with a **Feeds into** note — which Phase 2 sections consume its scan entries.
Absence of a category is itself data — note it and apply the branching rules in SKILL.md Phase 1.

---

## 1.1 Auth & Login Providers
**Feeds into:** Authentication & Login sections (one sub-section per provider found), Audit & Compliance (login success/failure events, provider used)
**Also feeds:** User Management (which auth methods users can link), App Settings (email sender)

### Files to check
- `passport.js`, `auth.js`, `auth/index.js`, `auth/providers/*`
- `pages/api/auth/[...nextauth].js`, `auth.config.ts` (Next.js / NextAuth)
- `supabase/config.ts`, `.env` for `SUPABASE_AUTH_*`
- `firebase.config.js` / `firebase-admin.js` for Firebase Auth
- Session middleware: `express-session`, `iron-session`, `lucia`
- `app.use(passport.initialize())` — confirms Passport is active

### Grep patterns
```
GoogleStrategy | GitHubStrategy | MicrosoftStrategy | AppleStrategy
NEXTAUTH_PROVIDERS | providers: [
enableEmailAuth | passwordAuth | magicLink | passwordless
requireEmailVerification | verifyEmail
mfaEnabled | totpEnabled | MFA_REQUIRED | authenticator
SESSION_SECRET | JWT_SECRET | NEXTAUTH_SECRET
sso | saml | SAMLStrategy
```

### What to record per provider
- Provider name, whether it is configured (y/n)
- Env var(s) that drive it (e.g. `GOOGLE_CLIENT_ID`)
- Whether an `enabled` boolean or runtime flag gates it — or is it always-on?
- Which table/collection stores it if DB-backed, and which API endpoint exposes it
- Session type: JWT / server session / cookie

### Critical check
If any provider is initialized unconditionally (no `enabled` check, no DB lookup before init):
→ Record `Runtime-tunable: no` and `Notes: always-on — owner toggle would have no effect`
→ This becomes a 🔴 CRITICAL gap in Phase 3.

---

## 1.2 RBAC — Roles & Permissions
**Feeds into:** Roles & Permissions (role builder + permissions matrix), Feature Access (role columns),
Constraints & Limits (per-role limits), User Management (role assignment), Audit & Compliance (permission + role change log)
**Note:** The audit log in Group 8 is primarily sourced from RBAC writes — every permission grant/revoke and role assignment generates an audit entry. If an audit log table exists, it is fed by 1.2 RBAC events.

### Files to check
- `middleware/auth.js`, `middleware/rbac.js`, `middleware/permissions.js`
- `guards/`, `policies/`, `abilities/` directories (NestJS, Laravel, CASL, Casbin)
- Any `Role`, `Permission`, `UserRole`, `RolePermission` model or schema definition
- Seed files: `seeds/roles.js`, `prisma/seed.ts`, `db/seed.sql`, `migrations/`
- Route decorators: `@Roles(...)`, `@Permissions(...)`, `@UseGuards(...)`
- The central permission-check function — wherever `hasPermission`, `can()`, `checkPermission` is defined

### Grep patterns
```
Role | Permission | UserRole | RolePermission | role_permissions
requireRole | checkPermission | hasPermission | can(
@Roles( | @UseGuards( | @authorize( | @Permissions(
req.user.role === | user.role !== | if.*role.*=== | user.isAdmin
RBAC | rbac | accessControl | acl | policy
casl | casbin | accesscontrol | bouncer | pundit
parent_role | role_hierarchy | inherits_from
```

### What to record per role
- Role name and identifier (e.g. `admin`, `editor`)
- Parent role ID or name if inheritance exists — or "none"
- Whether roles are static (seed-only) or dynamic (owner can create/delete)
- Whether the permissions list is stored in DB or hardcoded in a config array

### What to record per permission
- Resource name (e.g. `projects`, `users`, `billing`, `reports`)
- Action (e.g. `read`, `create`, `update`, `delete`, `export`, `invite`)
- Which roles have it — and whether via direct grant or inheritance
- Where it is checked: middleware filename:line, guard class, or inline in controller
- `Runtime-tunable: yes` only if a DB table stores the assignment AND an API can write to it

### Critical check — hardcoded role checks
Any `role === 'admin'` or `user.isAdmin` that is NOT inside the central permission-check function:
→ Record `Notes: HARDCODED CHECK — outside central permission system`
→ List file:line for every occurrence
→ These become 🔴 CRITICAL gaps in Phase 3

---

## 1.3 Feature Flags
**Feeds into:** Feature Access (role × feature matrix)
**Also feeds:** Plans & Billing (plan-gated flags), Constraints & Limits (flags with associated limits)

### Files to check
- `config/features.js`, `features/index.ts`, `flags.ts`, `feature-flags/`
- LaunchDarkly SDK init (`LDClient`, `initialize(`), Unleash (`new Unleash(`), Statsig, PostHog
- Any `FEATURE_*` or `ENABLE_*` env vars
- DB tables named `feature_flags`, `features`, `settings`, `toggles`
- Any `isEnabled(`, `useFeatureFlag(`, `getFlag(`, `isFeatureActive(`

### Grep patterns
```
FEATURE_ | ENABLE_ | FLAG_ | TOGGLE_
launchdarkly | unleash | statsig | posthog | flipper | flagsmith
featureFlags | feature_flags | isFeatureEnabled | getFeature
plan === 'pro' | plan === 'enterprise' | tier === | planIncludes(
```

### What to record per flag
- Flag name/key and its human description (if available)
- Current default value (on/off)
- Scope: global / per-role / per-plan / per-user
- Runtime-tunable: yes only if stored in DB or a live flag system with an API
- Whether per-role overrides are possible in the backend (not just globally toggled)

---

## 1.4 User Constraints
**Feeds into:** Constraints & Limits (rate limits, data scope, quotas)
**Also feeds:** Plans & Billing (plan-differentiated limits)
**Note on Roles & Permissions feed:** Constraint entries don't generate Roles & Permissions sections,
but the *role list* from 1.2 RBAC is what the Constraints & Limits section uses as its row source.
The data flows from 1.2 into Constraints, not from 1.4 into Roles & Permissions directly.

### Files to check
- Rate limiting: `express-rate-limit`, `@nestjs/throttler`, `redis-rate-limit`, `rack-attack`
- Any `RATE_LIMIT_*`, `MAX_REQUESTS_*`, `QUOTA_*`, `MAX_*` env vars
- Data scoping: Prisma middleware, query interceptors, Postgres RLS policies
- Any `createdBy`, `tenantId`, `organizationId`, `ownerId` used as query filters
- Export/pagination limits: `MAX_EXPORT_ROWS`, `PAGE_SIZE_LIMIT`, `MAX_FILE_SIZE`

### Grep patterns
```
rateLimit | throttle | RateLimit | rate_limit | Throttler
MAX_ | LIMIT_ | QUOTA_ | CAP_
tenantId | organizationId | createdBy | ownerId | scopeToTenant
rowLevelSecurity | enableRls | @TenantScope | filterByOwner
maxExportRows | exportLimit | paginationLimit
```

### What to record per constraint
- Constraint type (rate limit / data scope / export limit / quota)
- Current value and unit (e.g. `1000 requests/hour`, `30 days of data`)
- Which roles or plans it applies to — same for all, or differentiated?
- `Runtime-tunable`: yes only if stored in a `role_constraints` table with an update API

---

## 1.5 Plans & Billing
**Feeds into:** Plans & Billing section
**Also feeds:** Feature Access (plan-gated features), Constraints & Limits (plan-differentiated limits),
User Management (plan sets seat limit)

### Files to check
- `stripe.js`, `billing.service.ts`, `subscription.service.ts`, `subscriptions/`
- `STRIPE_*`, `PADDLE_*` env vars
- Any `Plan`, `Subscription`, `Product`, `PricingTier` model
- Any plan-gating: `requirePlan(`, `checkSubscription(`, `if (user.plan === `
- Seat limit logic: `MAX_SEATS`, `seatLimit`, `checkSeatAvailability`

### Grep patterns
```
Stripe | stripe | STRIPE_SECRET | STRIPE_PUBLISHABLE
Plan | plan | subscription | Subscription | product | Product
requirePlan | checkPlan | planGate | planFeature | planIncludes
FREE | STARTER | PRO | BUSINESS | ENTERPRISE | TEAM
seats | MAX_SEATS | seatLimit | seat_count
```

### What to record
- Plan names and their identifiers (slug or enum value)
- What each plan gates: features, seats, storage, API calls, data retention
- Whether plan config is Stripe-driven (metadata in Stripe products) or DB-driven
- Whether the owner can upgrade/downgrade plans via API, or is this billing-portal-only

---

## 1.6 Application-Level Settings
**Feeds into:** App Settings section
**Also feeds:** Authentication & Login (email sender for magic links and verification),
Audit & Compliance (data retention policy)

### Files to check
- `config/app.js`, `config/settings.ts`, `settings/` directory
- Any `Settings`, `Config`, `AppConfig`, `SiteSettings` database model
- `SMTP_*`, `SENDGRID_*`, `MAILGUN_*`, `RESEND_*`, `POSTMARK_*` env vars
- `APP_NAME`, `APP_URL`, `BRAND_*`, `LOGO_URL`, `PRIMARY_COLOR` env vars
- Webhook config: any `WebhookEndpoint` model or `WEBHOOK_*` vars
- `DEFAULT_TIMEZONE`, `DEFAULT_LOCALE`, `DEFAULT_LANGUAGE`, `DEFAULT_CURRENCY`
- `DATA_RETENTION_DAYS`, `LOG_RETENTION_*`, `GDPR_*`

### Grep patterns
```
AppSettings | SiteSettings | Settings | Config model
SMTP_ | SENDGRID_ | MAILGUN_ | RESEND_ | EMAIL_FROM
APP_NAME | APP_URL | BRAND_ | LOGO_ | PRIMARY_COLOR
webhook | WebhookEndpoint | WEBHOOK_SECRET
RETENTION | DATA_RETENTION | GDPR | purgeAfter
DEFAULT_TIMEZONE | DEFAULT_LOCALE | DEFAULT_LANGUAGE
```

### What to record per setting
- Key name, type (string/bool/number/url), current source (env or DB table)
- Whether a runtime API endpoint exists to update it
- Whether changes require server restart or redeploy
- Mark as `Runtime-tunable: yes` only if DB-backed with an update API

---

## 1.7 Environment Variable Classification
**Feeds into:** All sections (determines which scan findings are actually owner-controllable)
**Also identifies:** Gaps where a Configurable var has no DB-backed equivalent

### Process
Scan `.env`, `.env.example`, `.env.sample`, `config/default.js`, `config/production.js`.
Classify every variable into exactly one of:

| Class | Meaning | Owner-configurable? |
|---|---|---|
| **Secret** | API keys, private keys, DB passwords, signing secrets | Never |
| **Infrastructure** | PORT, NODE_ENV, DATABASE_URL, deployment-specific | Never |
| **Configurable** | Feature flags, limits, branding, email config, timeouts | Should be — flag if no DB/API equivalent |
| **Unclassified** | Unclear purpose — needs human review | Unknown |

For every `Configurable` var:
- If a DB column or `settings` table stores the same value AND an API can write it → `Runtime-tunable: yes`
- If only in `.env` with no DB equivalent → `Runtime-tunable: no`
  - If the var is a **feature flag** (controls whether a feature is on/off) → 🔴 CRITICAL in Phase 3
    Reason: feature flags gate user access to functionality. An env-only flag means the owner dashboard
    toggle has zero effect — users can access or be blocked from features regardless of what the owner sets.
  - If the var is a **configuration value** (branding, email sender, limits, locale) → 🟡 WARNING in Phase 3
    Reason: the value can be changed, but only via redeploy. Owner can still control it, just not at runtime.
- If hardcoded in source (not even in env) → `Runtime-tunable: no` → 🔴 CRITICAL in Phase 3 regardless of type

---

## 1.8 UI Surfaces
**Feeds into:** All dashboard groups — ungated surfaces are flagged as gaps in whichever group owns their feature area.
**Purpose:** Find UI that exists and is accessible but has no RBAC gate, feature flag check, plan check, or owner-configurable control. These are blind spots: the dashboard can't control what it doesn't know about.

### What counts as a UI surface
A UI surface is any user-facing screen, route, page, component, or nav item that:
- Is rendered to the user (not just a utility function or internal helper)
- Accesses data, performs an action, or exposes functionality that could be gated

### Scanning approach — find what's missing, not just what's present

**Critical:** Grep patterns find pages that *have* gate-related code. To find ungated surfaces, you must scan ALL route/page files and check each one for the *presence or absence* of a gate. For every route or page found:
1. Check whether it wraps in a role/permission guard — if yes, it's gated. If no, continue.
2. Check whether it has inline permission/flag/plan checks that conditionally hide content — if yes, it is *partially gated*: record `Gate: partial`. Then check the risk level: if the surface provides write, delete, or sensitive data access and the backend endpoint is unguarded, record `Notes: HIGH RISK — write/delete/sensitive access, backend endpoint unguarded, only frontend inline checks` (this becomes 🔴 CRITICAL in Phase 3). Otherwise record `Notes: inline check gates some content but route is still accessible` (this becomes 🟡 WARNING in Phase 3).
3. If no role/permission/flag/plan check found anywhere in the component or its route guard → record as ungated (`Gate: none`).

Do not rely on grep patterns alone — a surface with zero guard code produces zero grep hits and will be missed entirely if you only search for guard patterns. The Classification rules section below defines exactly what counts as a gate.

### Files to check
- **React / Next.js:** `pages/`, `app/`, `src/pages/`, `src/app/`, any `*.page.tsx`, `*.page.jsx`
- **Vue:** `views/`, `src/views/`, `router/index.js`, `router/index.ts`
- **Angular:** `*.component.ts` with route declarations, `app-routing.module.ts`
- **SvelteKit:** `src/routes/`, `+page.svelte` files
- **Rails / Django / Laravel (server-rendered):** `views/`, `templates/`, route files
- **Navigation:** Sidebar components, nav menus, header components — anything that renders links
- **Route guards / middleware:** `ProtectedRoute`, `AuthGuard`, `requireAuth`, `withAuth`, any HOC or wrapper that gates a route
- **API calls from UI:** `fetch(`, `axios.`, `useQuery(`, `useMutation(` — note what endpoint each page calls and whether those endpoints have backend guards

### Grep patterns
```
<Route | <ProtectedRoute | createBrowserRouter | useRoutes
<Navigate | Redirect | router.push( | useNavigate(
canActivate | authGuard | requiresAuth | isAuthenticated
hasPermission( | checkPermission( | usePermission(
hasRole( | useRole( | can( | cannot(
featureFlag | useFeatureFlag | isEnabled(
requirePlan( | checkSubscription( | isPlanActive(
v-if=".*role\|v-show=".*permission\|\*ngIf=".*auth
```

**Note:** Patterns like `isAuthenticated`, `requiresAuth`, and `authGuard` identify candidate guards that may be login-only. After finding them, apply the Classification rules below — if the guard only checks whether the user is logged in (not which role or permission they have), the surface is still ungated. Do not record it as gated based on these patterns alone.

### What to record per UI surface
- Route path or component name (e.g. `/admin/users`, `BillingPage`, `ExportButton`)
- What the surface does: read-only view / write / delete / sensitive data / admin action
- Gate present: name the specific permission check / feature flag / plan check / role guard — or `none` if only a login check or no check at all — or `partial` if inline checks exist inside the component but there is no route-level guard (the route is reachable by any authenticated user; only some UI elements inside it are conditionally shown). Login-required is NOT a gate — see Classification rules.
- Which backend API endpoint(s) it calls
- **For write/delete/sensitive surfaces:** whether the backend endpoint(s) have a permission check — cross-reference against the RBAC scan (1.2) for any route guards, middleware, or decorators on those endpoints. If 1.2 has not yet been scanned, note `backend guard: unknown` and resolve before Phase 2.
- Risk level if ungated or partially gated: high (write, delete, sensitive data) / low (read-only, non-sensitive)

### Classification rules
A surface is **gated** if it has a route-level guard that checks role or permission before the component renders at all. Specifically, at least one of:
- A route guard / HOC that checks **role or permission** (not just login) before rendering — e.g. `<RoleGuard role="admin">`, `requirePermission('reports.read')`
- A feature flag check at the route level (`isEnabled('export')`) that prevents the route from loading
- A plan check at the route level that blocks access below a certain tier

A surface is **partially gated** (`Gate: partial`) if:
- The route itself has no role/permission/flag/plan guard — any authenticated user can reach it
- BUT the component has inline checks (`can('edit', resource)`, `hasPermission('admin.users')`, `isEnabled('feature')`) that conditionally show or hide elements inside it
- Result: all users reach the route, but they see different content based on their role/permissions

**Important:** An inline permission check inside a component does NOT make the surface fully gated. It only makes it partially gated. A fully gated surface blocks the route entirely for users without the required role/permission/plan.

A surface is **ungated** if it has no role/permission/flag/plan check at any level — neither at the route nor inside the component.

**Login-required ≠ gated.** A surface behind `isAuthenticated` or `requireAuth` (login check only) is still ungated if any logged-in user can reach it regardless of their role, permissions, or plan. The gate must distinguish *which users* can access the surface, not just *whether* the user is logged in.

### Critical check — high-risk ungated surfaces
Any ungated surface where the user can write, delete, or access sensitive data:
→ Record `Gate: none` and `Notes: HIGH RISK — write/delete/sensitive access, no gate`
→ These become 🔴 CRITICAL gaps in Phase 3.

Low-risk ungated surfaces (read-only, non-sensitive):
→ Record `Gate: none` and `Notes: LOW RISK — read-only, no gate`
→ These become 🟡 WARNING gaps in Phase 3.

**Critical check — high-risk partially gated surfaces:**
A partially gated surface with write, delete, or sensitive data access is still dangerous — the backend endpoint typically has no permission check, meaning the inline frontend filtering is the only barrier, and it can be bypassed. If the backend endpoint is unguarded:
→ Record `Gate: partial` and `Notes: HIGH RISK — write/delete/sensitive access, backend endpoint unguarded, only frontend inline checks`
→ These become 🔴 CRITICAL gaps in Phase 3 (the backend must be fixed before a route-level gate alone is sufficient).

---

## Cross-Reference Map

Which scan categories contribute to which Phase 2 section groups:

| Phase 2 Group | Primary source categories | Secondary source categories |
|---|---|---|
| Authentication & Login | 1.1 Auth | 1.7 Env (provider keys), 1.8 UI (login screen, auth flows) |
| Roles & Permissions | 1.2 RBAC | 1.7 Env (any role defaults), 1.8 UI (ungated permission-sensitive screens) |
| User Management | 1.2 RBAC (roles to assign) | 1.1 Auth (linked accounts), 1.8 UI (ungated user-facing screens) |
| Feature Access | 1.3 Flags | 1.2 RBAC (roles), 1.5 Billing (plan gates), 1.8 UI (ungated feature screens) |
| Constraints & Limits | 1.4 Constraints | 1.2 RBAC (per-role), 1.5 Billing (per-plan) |
| Plans & Billing | 1.5 Billing | 1.3 Flags (plan-gated), 1.4 Constraints (quotas), 1.8 UI (ungated billing screens) |
| App Settings | 1.6 Settings | 1.7 Env (configurable vars), 1.1 Auth (email sender) |
| Audit & Compliance | 1.2 RBAC (change log) | 1.6 Settings (retention), 1.1 Auth (login events), 1.8 UI (ungated audit/export screens) |

If a scan category has zero entries, every section group that lists it as "primary" must be
reviewed — that group may not be buildable yet.
