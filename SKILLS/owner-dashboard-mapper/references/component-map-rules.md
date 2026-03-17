# Component Map Rules

Detailed rules for each of the 8 dashboard groups. Read this during Phase 2.
SKILL.md defines the section block format and orphan check rules — this file defines
the specifics of what belongs in each group, what connects to what, and what the user experiences.

**Section block template is in SKILL.md Phase 2. This file adds group-specific rules only.**

---

## Handling UI Surface Scan Entries (Category: ui-surface)

UI surface entries from Phase 1 (category `ui-surface`) are handled differently from backend entries.

**Gated surfaces (`Gate:` is not "none" and not "partial"):**
These are already controlled — they have a permission, flag, or plan check. Cross-reference them against the backend entries. Check whether the gate they rely on is itself valid:
- If the permission they check is orphaned (exists in DB but never checked in code) → this is a 🔴 CRITICAL compound gap: the surface appears gated but the gate has no effect. File an orphaned permission gap (see gap-rules.md Orphaned Permission). Find or create the Roles & Permissions section block for this permission and add the surface note there: `[GATED — but gate is broken: permission orphaned]`. If no Roles & Permissions block covers this permission yet, treat the permission as an unlinked scan entry (see orphan check in SKILL.md) — file a 🔵 SUGGESTION for the missing section, then add the CRITICAL compound gap to it.
- If the feature flag they check is env-only → this is a 🔴 CRITICAL compound gap: the surface appears gated but the toggle has no runtime effect. File an env-only feature flag gap in the Feature Access section block for that flag. Note the surface in that gap's `Blocks:` field as a blocked section.
- If the surface checks a plan tier (`requirePlan('pro')`, `checkSubscription(`) but the 1.5 Billing scan found no matching plan or billing integration → this is a 🟡 WARNING compound gap: the plan check runs but may always fail or always pass if the billing system is absent or misconfigured. File a WARNING in the Plans & Billing section: "Surface [X] is plan-gated against '[plan]' but no matching billing integration was found in the scan."
- If the gate is valid and working → note the surface as `[EXISTING — gated]` and move on.

**Partially gated surfaces (`Gate: partial`):**
The route is reachable by any authenticated user, but some UI elements inside it are conditionally shown based on role, permission, or flag checks. This is a partial gate — better than nothing, but the owner dashboard cannot control who reaches the route, only what they see once there.

For each partially gated surface:
1. Determine which dashboard group owns the feature area.
2. Generate a section block in that group for "add route-level gate to [surface name]".
3. Set `Source:` to the Phase 1 ui-surface scan entry Name + Source.
4. Set `Backend enforces:` based on the risk level recorded in the Phase 1 scan entry's `Notes:` field:
   - If `Notes:` contains "HIGH RISK" → set `Backend enforces: no` — the backend endpoint has no permission check; frontend inline checks are the only (bypassable) barrier.
   - Otherwise (low-risk surface) → set `Backend enforces: partial` — some content is gated at the component level, though the route itself has no backend guard.
5. Set `Gaps:` based on the risk level recorded in the Phase 1 scan entry's `Notes:` field:
   - If `Notes:` contains "HIGH RISK" (write/delete/sensitive access, backend endpoint unguarded) → file a 🔴 CRITICAL using the CRITICAL template: "Partially gated high-risk surface — [surface]. Backend endpoint is unguarded; inline frontend checks can be bypassed via direct API call."
   - Otherwise (read-only or low-risk surface) → file a 🟡 WARNING using the WARNING template: "Surface [X] has inline permission checks but no route-level gate. Authenticated users can reach the route regardless of role; only rendered content differs."
6. `Owner can:` — describe what the owner will control once the gap is resolved, branching on risk level:
   - If HIGH RISK: describe the future dashboard actions once both the backend endpoint and the route-level gate are fixed, e.g. "• Grant or revoke [permission name] per role to control access to [route]." Follow the same verb pattern as ungated surfaces — name the specific permission, flag, or plan tier. Do not describe the backend fix (that belongs in `Gaps:`).
   - Otherwise (low-risk): describe what the owner will control once a route-level gate is added, e.g. "• Restrict access to [route] to users with [permission name]." Do not claim full control is currently possible.
7. `UI component:` — gate-adding sections document a real gap and a real remediation path, but the owner does not resolve it through a new dashboard screen of their own. The gate is configured through an existing section. Set this field to reflect where the gate will live:
   - Permission-based gate → `none — owner grants/revokes via Roles & Permissions › Permissions Matrix`
   - Feature flag gate → `none — owner enables/disables via Feature Access › Role × Feature Matrix`
   - Plan-based gate → `none — owner sets minimum plan tier via Plans & Billing`
   If the gate type is unknown at this stage, write `none — owner configures via [target section name]`.
8. `Interconnects with:` — add at minimum one connection to the section whose permission, flag, or plan will be used as the route-level gate. Format: `[Section name] → route-level gate for [surface] will require [permission/flag/plan] from here`. Same logic as ungated surfaces.
9. `Depends on:` — write `"none"`. Gate-adding sections do not technically break without the permission/flag/plan section — they are simply not yet useful. The relationship is expressed through `Interconnects with:` (step 8) rather than the dependency graph, which avoids a required `Required by:` entry on the depended-upon section.
10. `Required by:` — write `"none"`.
11. `Affects roles:` — write `"all roles (partially gated — route reachable by all authenticated users; inline checks limit rendered content only)"`. This documents the current state accurately: all roles reach the route today. Once a route-level gate is added, this field will be updated to reflect which roles are restricted.
12. `User sees:` — describe both the current state and the post-fix state, branching on risk level:
    - If HIGH RISK: "→ Currently, all authenticated users can reach [route] and the backend endpoint [endpoint] is callable directly by anyone — the frontend inline checks can be bypassed entirely. Once the backend endpoint guard and route-level gate are both in place, only users with [permission] will be able to reach the route."
    - Otherwise (low-risk): "→ Currently, all authenticated users can reach [route]; only [role X] sees [element Y]. Once a route-level gate is added, users without [permission] will be blocked at the route."

Do not generate a separate section block for every individual partially gated surface if multiple surfaces belong to the same feature area. Group them using the same rules as ungated surfaces: one section block per feature area where a single route-level gate would cover all of them. If partially gated surfaces span different permissions or flags, keep them in separate blocks.

**Ungated surfaces (`Gate: none`):**
These are the core finding. For each ungated surface:
1. Determine which dashboard group owns the feature area (e.g. an export button → Feature Access or Roles & Permissions; a billing page → Plans & Billing).
2. Generate a section block in that group for "add a gate to [surface name]".
3. Set `Source:` to the Phase 1 ui-surface scan entry Name + Source (e.g. `ExportButton — src/components/ExportButton.tsx:14`). This is the same as any other section block — the source is the Phase 1 entry that justifies the section's existence.
4. Set `Backend enforces: no` — there is currently no backend check.
5. Set `Gaps:` to the appropriate ungated surface gap (see gap-rules.md Ungated UI Surface).
6. `Owner can:` — list the specific dashboard actions the owner will have once the gate is built. Use concrete verbs. Examples:
   - `• Grant or revoke [permission name] per role` (if gate is permission-based)
   - `• Enable or disable [feature name] per role` (if gate is flag-based)
   - `• Restrict [surface name] to users on [plan tier] or above` (if gate is plan-based)
   Do not write "Owner can control who has access" — that is too vague. Name the specific permission, flag, or plan tier involved.
7. `UI component:` — gate-adding sections document a real gap and a real remediation path, but the owner does not resolve it through a new dashboard screen of their own. The gate is configured through an existing section. Set this field to reflect where the gate will live:
   - Permission-based gate → `none — owner grants/revokes via Roles & Permissions › Permissions Matrix`
   - Feature flag gate → `none — owner enables/disables via Feature Access › Role × Feature Matrix`
   - Plan-based gate → `none — owner sets minimum plan tier via Plans & Billing`
   If the gate type is unknown at this stage, write `none — owner configures via [target section name]`.
8. `Interconnects with:` — add at minimum one connection to the section that the gate will depend on. Format: `[Section name] → gate for this surface requires [permission/flag/plan] to be configured here`. If the ungated surface also relates to audit logging (e.g. write operations), add: `Audit & Compliance → write operations on [surface] should be logged once gated`.
9. `Depends on:` — write `"none"`. Gate-adding sections do not technically break without the permission/flag/plan section — they are simply not yet useful. The relationship is expressed through `Interconnects with:` (step 8), which maintains bidirectionality without requiring an asymmetric `Required by:` entry on the depended-upon section. Note: if you want to record the conceptual dependency for documentation purposes, do so in the `Notes:` field of the section block.
10. `Required by:` — write `"none"`. Gate-adding sections are terminal: no other section depends on a gate existing before it can function.
11. `Affects roles:` — write `"all roles (ungated — currently no role-based distinction exists)"`. This signals to the Key Rules checker that "all roles" here is intentional and reflects the current ungated state, not a vague omission. Once gated, this field will be updated to the specific roles the owner controls.
12. `User sees:` — describe the current problem and what changes once the gate exists: "→ All users: Currently, [surface name] is accessible to all authenticated users regardless of role. Once gated, access will be restricted to users with [permission/flag/plan]." If the surface is high-risk (Notes contains "HIGH RISK"), add: "The backend endpoint [endpoint] is also unguarded — until that is fixed, direct API calls bypass any frontend gate."

Do not generate a separate section block for every individual ungated surface if multiple surfaces belong to the same feature area. Group them: one section block per ungated feature area (e.g. "Reports Access Gate" covers all ungated reports surfaces, not one block per reports page).

**Feature area grouping rules** (apply to both ungated and partially gated surface blocks):
- Same feature area = surfaces that would share the same permission, flag, or plan check once gated. If `/reports/overview` and `/reports/detail` would both be controlled by `reports.read`, they belong in the same block.
- Different feature area = surfaces that would need *different* permissions or flags. `/reports/overview` (needs `reports.read`) and `/admin/users/export` (needs `users.export`) go in separate blocks.
- When unsure, ask: "Would a single owner toggle control all of these surfaces?" If yes → same block. If different toggles would be needed → separate blocks.
- High-risk (write/delete) and low-risk (read-only) surfaces must not be merged into the same block even if they are in the same feature area, because they produce different gap severities (CRITICAL vs WARNING). This applies to both ungated and partially gated surfaces.

---

## Group 1 — Authentication & Login

**Source categories:** 1.1 Auth, 1.7 Env (provider keys)
**One sub-section per provider** — do not collapse all OAuth into one section.

### Sub-sections to generate (only if scan found evidence)
- Google OAuth
- GitHub OAuth
- Microsoft / Azure AD OAuth
- Apple OAuth
- Email + Password
- Magic Link / Passwordless
- SSO / SAML
- MFA Settings (enforce for all / enforce per role / optional)

### Owner can (typical actions)
- Enable or disable the provider
- Configure client ID / secret (if runtime-changeable)
- Set the default role assigned to users who sign up via this provider
- Require email verification before first login
- Enforce MFA for specific roles

### User sees (complete the loop)
When owner disables Google OAuth:
→ The "Continue with Google" button disappears from the login screen.
→ Existing users who only have a Google account are locked out — owner should be warned.

When owner enables magic link:
→ A "Email me a link" option appears on the login screen.
→ Email sender config (App Settings) must be working or the email will silently fail.

When owner enforces MFA for Admins:
→ Admins are prompted to set up an authenticator on next login.
→ Admins who skip it cannot proceed past the MFA setup screen.

### Critical check — login screen mirror
After generating all auth sub-sections, verify:
- Every button on the frontend login screen has a corresponding active backend provider
- Every active backend provider has a corresponding button (or the owner consciously omitted it)
- Any mismatch is a gap — severity depends on direction (see gap-rules.md Login Screen Mismatch)

### Interconnects with
- Roles & Permissions → first-login default role assignment per provider
- User Management → user's linked auth methods are visible in user detail
- App Settings → email sender config is required for magic link and email verification
- Audit & Compliance → login events (success, failure, provider used) logged here

### Depends on
- Nothing — this is a foundation section

### Required by
- User Management (users must be able to log in before they can be managed)
- Roles & Permissions (role assignment only matters once auth exists)

---

## Group 2 — Roles & Permissions

**Source categories:** 1.2 RBAC
**Always two sub-sections:** Role Builder + Permissions Matrix

### Sub-section: Role Builder
Owner can:
- Create a new role (name, description, parent role for inheritance)
- Edit a role's name or description
- Set or change the parent role (changes all inherited permissions immediately)
- Clone a role as a starting point
- Deprecate / delete a role (with warning: N users will lose this role)

User sees:
→ When a role is deleted and users had it assigned, they fall back to a default role or lose access.
→ When a parent role changes, all child roles' inherited permissions change immediately on next auth check.

### Sub-section: Permissions Matrix
Owner can:
- Toggle a directly-granted permission on or off for any role
- View which permissions are inherited (locked — must edit parent to change)
- Navigate to the parent role directly from an inherited cell
- See a cascade warning when removing a permission from a parent role that has children

**Three cell states — all three must be implemented:**
| State | Visual | Editable? | What owner can do |
|---|---|---|---|
| Directly granted | ✅ green | Yes | Click to revoke |
| Inherited | ↑ blue/muted | No | Click → navigates to parent role |
| Not granted | ○ empty | Yes | Click to grant directly |

User sees:
→ When owner grants `projects.export` to Editor:
   Editors can now click the Export button in the projects view. It was hidden before.
→ When owner removes `billing.read` from Admin:
   Admins no longer see the Billing tab. Any hardcoded Admin billing checks bypass this — flag those.

### Cascade warning rule
When owner clicks to remove a permission from a role:
- Count how many other roles inherit this permission from the current role
- If count > 0: show "Removing this will affect [N] roles: [list]. Continue?"
- Do not silently cascade

### Hardcoded check rule
If Phase 1 found any `role === 'x'` inline checks:
→ Add a banner to this section: "⚠️ [N] permissions in this app are hardcoded and cannot be
   controlled here. See gap report for details."
→ List each with its file:line and what it controls

### Interconnects with
- Authentication & Login → default role assigned at first login per provider
- User Management → role options available in invite and edit-user flows
- Feature Access → roles are the columns in the feature × role matrix
- Constraints & Limits → limits are applied per role (limits table uses role IDs)
- Audit & Compliance → every permission change is logged with actor + old/new value
- App Settings → default role on first-login may be stored as a configurable setting
- Plans & Billing → plan-gated features are ultimately granted to roles; roles must exist for plan-feature relationships to render

### Depends on
- Authentication & Login (roles are only meaningful once users can log in)

### Required by
- User Management, Feature Access, Constraints & Limits (all use roles as their primary axis)
- Plans & Billing (plan-gated features are ultimately granted to roles — plans gate the feature, roles determine who gets it)
- Audit & Compliance (permission and role changes are the primary events the audit log records — nothing to log until roles and permissions exist)

---

## Group 3 — User Management

**Source categories:** 1.2 RBAC (roles), 1.1 Auth (linked accounts)

### Sub-sections (only include those with backend evidence)
- Invite users (set role at invite time; triggers invite email)
- User list with role filter and search
- Edit user: change role, deactivate, reset MFA
- Deactivate / reactivate user
- Bulk actions: role change, deactivation, re-invite
- Active sessions view (only if session table or JWT revocation exists in backend)

### Owner can
- Invite a new user with a pre-assigned role
- Change a user's role at any time
- Deactivate a user (revokes all access immediately if session invalidation exists)
- Bulk-change roles across selected users

### User sees
→ When owner invites a user as Editor:
   The user receives an invite email. On accepting, they log in with Editor permissions active immediately.
→ When owner deactivates a user:
   The user's session is invalidated (if backend supports it) or their next request is rejected.
   They cannot log in again until reactivated.
→ When owner changes a user's role from Admin to Viewer:
   On their next page load or auth check, the user loses Admin-level menu items and routes.

### If no invite API exists
→ 🔵 SUGGESTION: "Backend has a User model but no invite endpoint.
   Build POST /api/admin/users/invite before adding the invite form to the dashboard."

### Interconnects with
- Authentication & Login → which auth methods the user has linked (shown in user detail)
- Roles & Permissions → role options available when inviting or editing a user
- Plans & Billing → plan's seat limit gates how many users can be invited
- Audit & Compliance → user lifecycle events (invite sent, role changed, deactivated) logged
- App Settings → invite emails are sent via the email sender configured in App Settings
- Constraints & Limits → changing a user's role changes which constraint tier (rate limits, data scope) applies to that user

### Depends on
- Authentication & Login (users need a login method)
- Roles & Permissions (roles must exist before they can be assigned to users)

### Required by
- Audit & Compliance (user lifecycle events — invites, role changes, deactivations — are the primary log source)
- Plans & Billing (seat usage count comes from active users; seat limit enforcement gates invitations here)

---

## Group 4 — Feature Access

**Source categories:** 1.3 Flags, 1.2 RBAC (role list), 1.5 Billing (plan-gated flags)

### Layout: Role × Feature matrix
Rows = features (from scan entries). Columns = roles (from RBAC scan).
Each cell = whether that role has that feature enabled.

### Flag type distinction (mandatory — four types)
| Flag type | Cell appearance | Owner can edit? | Notes |
|---|---|---|---|
| Global flag | Single toggle above matrix | Yes (affects all roles) | Turning it off overrides all per-role settings |
| Per-role flag | Individual cell per role | Yes | Each role independently configurable |
| Per-plan flag | Cell shows plan name | Only by changing the plan | Owner cannot override without upgrading |
| Env-only flag | Greyed out cell + lock icon | No | Attach 🟡 WARNING: "Requires redeploy to change" |

### User sees
→ When owner turns off `analytics_dashboard` for Viewer:
   The Analytics tab disappears from Viewers' navigation. Existing analytics pages return 403.
→ When a plan-gated feature is toggled (owner upgrades plan):
   All users on that account gain the feature simultaneously on next page load.

### Affects roles guidance
- Per-role flag toggled → `Affects roles: [the specific role whose cell was changed]`
- Global flag toggled → `Affects roles: all roles`
- Plan-gated feature changed via plan upgrade → `Affects roles: all users regardless of role`
- Env-only flag (greyed out) → `Affects roles: n/a — not owner-controllable at runtime`

### Interconnects with
- Roles & Permissions → roles are the column source; role changes here must sync
- Plans & Billing → plan-gated features must match what the plan allows; conflicts are gaps
- Constraints & Limits → features and their associated limits are coupled (e.g. export feature ↔ export row limit)
- Audit & Compliance → feature flag changes are logged

### Depends on
- Roles & Permissions (roles must exist before they can be columns)
- Plans & Billing (plan-gated features require plan data to render correctly)

### Required by
- Nothing directly — Feature Access is a consumer of roles and plans, not a source for other sections

---

## Group 5 — Constraints & Limits

**Source categories:** 1.4 Constraints, 1.2 RBAC (roles), 1.5 Billing (plan quotas)

### Sub-sections (only if scan found evidence)
- Rate limits per role (requests/min or /hour)
- Data visibility scope (date range, ownership filter, tenant isolation)
- Export limits (max rows, max file size per export)
- API quotas (if API access is a feature)
- Storage caps (if file storage is used)

### Per-role table format (for rate limits and quotas)
```
| Constraint          | Owner     | Admin  | Editor | Viewer |
|---------------------|-----------|--------|--------|--------|
| API requests/hour   | unlimited | 1000   | 500    | 100    |
| Export max rows     | unlimited | 50000  | 10000  | 1000   |
| Data visible since  | all time  | 1 year | 90 days| 30 days|
```

Owner edits a cell → PATCH /api/admin/constraints with role + constraint type + new value.

### User sees
→ When owner lowers Editor's API limit from 500 to 200:
   Editors hitting the API after the 200th request/hour receive 429 Too Many Requests.
→ When owner restricts Viewer data visibility to 30 days:
   Viewers' data tables and charts only show records from the last 30 days.
   Older data is not deleted — just scoped out of their queries.

### If limits are env-only
→ 🟡 WARNING: "Rate limits are read from env vars at startup. Owner saves are not reflected
   until the server restarts. Show a 'Changes take effect on next deploy' notice until this is fixed."

### Interconnects with
- Roles & Permissions → limits apply per role; role list is the row source
- Plans & Billing → plan may set base quotas that per-role limits cannot exceed
- Feature Access → features and their associated limits are coupled (e.g. export feature ↔ export row limit)
- Audit & Compliance → constraint changes logged
- User Management → changing a user's role (User Management) changes which constraint tier applies to them

### Depends on
- Roles & Permissions (limits are keyed to roles)
- Plans & Billing (plan may set base quotas that per-role limits cannot exceed — plan data must exist before constraints can be fully defined)

### Required by
- Nothing directly — Constraints & Limits is a consumer of roles and plans

---

## Group 6 — Plans & Billing

**Source categories:** 1.5 Billing, 1.3 Flags (plan-gated), 1.4 Constraints (plan quotas)

### Sub-sections
- Plan overview (what each plan includes — read-heavy)
- Seat management (current usage vs seat limit)
- Upgrade / downgrade flow (link to billing portal or in-app)
- Billing history (read-only, sourced from Stripe/Paddle)
- Plan × Feature matrix (which features unlock at which plan)

### Owner can
- View current plan and usage
- Initiate upgrade or downgrade (may redirect to billing portal)
- View billing history and invoices
- Manage seat assignments if plan has per-seat billing

### User sees
→ When owner upgrades from Starter to Pro:
   Pro-gated features immediately become available to all users.
   Feature Access matrix cells that were locked as "plan-gated" become editable.
→ When seat limit is reached:
   Owner cannot invite more users until they upgrade. Invite button shows upgrade prompt.

### Note on ownership
Plans & Billing is often mostly read for the owner (Stripe controls the data).
The owner dashboard is a view + trigger layer, not the source of truth.
Do not build write controls for plan data that Stripe manages.

### Interconnects with
- Feature Access → plan gates features; plan changes unlock or lock feature cells
- Constraints & Limits → plan sets base quotas
- User Management → plan's seat limit constrains how many users can be invited
- App Settings → billing email notifications go through email sender config
- Roles & Permissions → plan-gated features are granted to roles; plan-feature relationships require roles to exist

### Depends on
- App Settings (billing notification emails require email sender to be configured)
- Roles & Permissions (plan-gated features are granted to roles — roles must exist before plan-feature relationships can be rendered or enforced)

### Required by
- Feature Access (plan gates which feature cells are editable — Feature Access needs plan data to render)
- Constraints & Limits (plan sets the base quotas that per-role limits cannot exceed)
- User Management (seat limit enforcement depends on knowing the active plan's seat cap)

---

## Group 7 — App Settings

**Source categories:** 1.6 Settings, 1.7 Env (Configurable vars), 1.1 Auth (email sender)

### Sub-sections (only include those with backend evidence — check source before adding)
- Branding (app name, logo URL, primary color) — only if BRAND_* vars or settings model exists
- Email config (sender name, reply-to address, provider) — only if SMTP/email vars exist
- Notifications (which events send emails, to whom) — only if notification model exists
- Webhooks (endpoint URL, subscribed events, signing secret) — only if WebhookEndpoint model
- Localization (default timezone, language, currency) — only if DEFAULT_* vars exist
- Data retention (how long to keep logs, exports, deleted records) — only if RETENTION config exists

### Owner can
Depends on sub-section. Examples:
- Set app display name and logo URL
- Configure SMTP sender for all transactional emails
- Choose which user events trigger notification emails
- Add and manage webhook endpoints with event subscriptions
- Set the default timezone shown to all users

### User sees
→ When owner updates app name:
   The new name appears in the browser tab, email subjects, and app header for all users.
→ When owner sets default timezone to UTC+2:
   All dates and timestamps across the app display in that timezone for users who haven't
   set their own preference.
→ When owner disables a webhook endpoint:
   Events stop being sent to that URL immediately. No queued events are flushed.

### Interconnects with
- Authentication & Login → email sender config powers magic link and verification emails
- Audit & Compliance → data retention policy controls how long audit logs are stored
- Plans & Billing → billing notification emails require email sender config
- Roles & Permissions → default role on first-login may be a configurable setting stored here
- User Management → invite emails are sent via the email sender configured here

### Depends on
- Nothing — App Settings is typically a foundation section

### Required by
- Authentication & Login (email-based auth methods depend on email sender being configured)
- Audit & Compliance (retention settings live here)
- Plans & Billing (billing notification emails go through the email sender configured here)

---

## Group 8 — Audit & Compliance

**Source categories:** 1.2 RBAC (change log), 1.1 Auth (login events), 1.6 Settings (retention)

### Sub-sections
- Permission change log (who changed what role/permission, old value, new value, timestamp)
- User activity log (login events, role changes, deactivations)
- App settings change log (who changed which setting, when)
- Data export (GDPR compliance — owner can download all data for a user or account)
- Log retention settings (how long to keep each log type — links to App Settings)

### Owner can
- View and filter audit logs by actor, date range, event type
- Export audit log as CSV
- Trigger a user data export (GDPR request)
- Configure log retention period (stored in App Settings)

### User sees
→ General: Users do not directly see audit logs or permission change history.
→ When an admin's permissions are changed by the owner, the admin may notice missing UI elements on their next page load — this is the audit log's source event.
→ GDPR data export: When the owner triggers a data export for a specific user, that user receives a download link or file containing all their stored data. This is the one Audit & Compliance action that produces a direct user-facing output.

### If no audit log exists
→ 🔵 SUGGESTION: "No audit log table found. Strongly recommend adding before launch.
   Every write to roles, permissions, and settings should create an entry with:
   actor (who), target (what), old_value, new_value, timestamp."

### Interconnects with
- Roles & Permissions → logs every permission and role change
- User Management → logs every invite, role change, and deactivation
- App Settings → retention policy stored there controls how long these logs are kept
- Authentication & Login → logs login events (provider used, success/failure)

### Depends on
- Roles & Permissions (nothing to log until permissions exist)
- User Management (nothing to log until users exist)
- App Settings (retention settings stored there)

### Required by
- Nothing — Audit & Compliance is the terminal section. No other section depends on it to function.

---

## Bidirectionality Enforcement Checklist

After writing all section blocks, verify every pair listed here has the connection declared in both:

| Section A | Section B | Connection reason |
|---|---|---|
| Auth & Login | Roles & Permissions | First-login role assignment |
| Auth & Login | User Management | User's linked auth methods |
| Auth & Login | App Settings | Email sender for magic link / verification |
| Auth & Login | Audit & Compliance | Login events logged |
| Roles & Permissions | User Management | Role options for invite/edit |
| Roles & Permissions | Feature Access | Roles are the feature matrix columns |
| Roles & Permissions | Constraints & Limits | Limits keyed to roles |
| Roles & Permissions | Audit & Compliance | Permission changes logged |
| Roles & Permissions | App Settings | Default role on first login stored in App Settings |
| User Management | Plans & Billing | Seat limit gates invitations |
| User Management | Audit & Compliance | User lifecycle events logged |
| User Management | App Settings | Invite emails sent via App Settings email sender |
| User Management | Constraints & Limits | Role changes determine which constraint tier applies to a user |
| Feature Access | Plans & Billing | Plan gates feature availability |
| Feature Access | Constraints & Limits | Features and their associated limits are coupled (e.g. export feature ↔ export row limit) |
| Feature Access | Audit & Compliance | Feature flag changes logged |
| Constraints & Limits | Plans & Billing | Plan sets base quotas |
| Plans & Billing | Roles & Permissions | Plan-gated features are granted to roles — roles must exist for plan-feature relationships to render |
| Plans & Billing | App Settings | Billing emails use email sender |
| App Settings | Audit & Compliance | Retention policy controls log lifespan |
| App Settings | Auth & Login | Email sender required for email auth |

Any pair in this table where one side is missing the connection → one-way connection orphan.
Fix it before Phase 3.
