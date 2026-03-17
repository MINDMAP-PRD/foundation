# Gap Rules

Classification criteria, templates, and examples for every gap type.

---

## Severity Levels

### 🔴 CRITICAL — Build nothing until this is fixed

Either the owner dashboard would show a control that the backend ignores (false promise), or a UI surface exists with no owner control at all (missing gate). In both cases the owner cannot govern the behaviour.

**Criteria (any one of these = CRITICAL):**
- A UI section would write to an endpoint that doesn't exist
- A permission toggle maps to a permission that isn't checked anywhere in code
- An auth provider toggle exists in the UI but the backend always allows the provider regardless
- A feature flag is stored only in env vars with no runtime-update mechanism
  (env-only *configuration values* like branding or limits are 🟡 WARNING, not CRITICAL —
  the owner can still change them via redeploy. Feature flags are CRITICAL because the dashboard
  toggle has zero effect on user access regardless of what the owner sets.)
- Any business logic uses `role === 'admin'` style hardcoded checks instead of the central permission system
- A "disable feature" toggle exists but no backend guard enforces the disabled state
- A config value is hardcoded directly in source code (not even in env) — the owner has no path to change it without a code deploy
- A UI surface with write access, data mutation, or sensitive data exposure has no RBAC gate, feature flag check, or plan check — any authenticated user can reach it regardless of role
- A UI surface is partially gated (`Gate: partial`) with write/delete/sensitive access AND its backend endpoint has no permission check — the frontend inline checks are the only barrier and can be bypassed via direct API calls

**Template:**
```
🔴 CRITICAL: [Short title]
Backend: [What exists or doesn't exist — be specific, cite files]
UI status: [What the UI does or would do]
Impact: [What happens if you build the UI anyway — false config, security hole, etc.]
Recommended fix: [Specific change needed — may require both a backend guard and a frontend check]
Blocks: [List dashboard sections that cannot be built until this is resolved]
```
When this gap is output in Phase 4.3, prefix it with its GAP-N number. The `Blocks:` field value is what Phase 4.6 uses when marking sections `[BLOCKED — fix GAP-N first]`.

Note for Phase 2 inline filing: when a gap is filed in a section block's `Gaps:` field during Phase 2, use the template format *without* the GAP-N prefix — numbers are assigned only when collecting and outputting in Phase 4.3.

**Example:**
```
🔴 CRITICAL: Google OAuth toggle has no runtime enforcement
Backend: GoogleStrategy is initialized unconditionally in passport.js:14.
  No DB flag or env var gates whether Google login is allowed.
UI status: Owner dashboard "Authentication" section has a Google OAuth on/off toggle.
Impact: Owner disabling Google OAuth in the UI has no effect — Google login remains active.
Recommended fix: Add `oauth_providers` table with `enabled` boolean per provider.
  Wrap GoogleStrategy initialization in a runtime check against this table.
Blocks: Authentication & Login section (entire provider management UI)
```

---

### 🟡 WARNING — Partial coverage or uncontrolled read access

Either the backend has the concept but the owner can't fully control it, there's a mismatch between what the UI implies and what the backend does, or a read-only UI surface exists with no owner-configurable gate.

**Criteria (any one of these = WARNING):**
- Config exists in env vars only — owner must redeploy to change it (no runtime API)
- Permission is enforced in middleware but also hardcoded in 1–2 places (partial escape)
- A role exists in the DB but the permissions list is partially hardcoded
- An API endpoint exists to update a setting but the frontend doesn't surface it
- Inheritance is implemented but the UI doesn't show the inherited state (just shows on/off)
- A feature flag is per-role in the backend but the UI only shows a global toggle
- A UI surface is read-only or non-sensitive and has no RBAC gate, feature flag check, or plan check — all authenticated users can view it regardless of role (owner has no control, but no write risk)
- A UI surface has inline permission/flag/plan checks inside the component but no route-level guard, AND the surface is read-only or low-risk — any authenticated user can reach the route; only the rendered content differs by role (`Gate: partial`, low-risk). Note: if the surface has write/delete/sensitive access and the backend endpoint is unguarded, this escalates to 🔴 CRITICAL — see CRITICAL criteria.

**Template:**
```
🟡 WARNING: [Short title]
Backend: [What partially exists]
UI status: [What the UI does or can't do]
Impact: [Limitations — what the owner thinks they control vs what they actually control]
Recommended fix: [Specific improvement — add API, fix UI, document limitation]
Affects: [Sections where this warning is visible or relevant — does not block building them]
```
Note for Phase 2 inline filing: use this template *without* a GAP-N prefix — numbers are assigned only when collecting and outputting in Phase 4.3.

**Example:**
```
🟡 WARNING: Rate limits are env-only — no runtime update
Backend: RATE_LIMIT_ADMIN=1000 and RATE_LIMIT_VIEWER=100 are set in .env.
  express-rate-limit reads these at server startup only.
UI status: Owner dashboard "Constraints" section shows rate limit inputs per role.
Impact: Owner saves new values → nothing changes until next deploy.
  Owner believes limits are updated; they are not.
Recommended fix: Store limits in `role_constraints` table. Add PATCH /api/admin/constraints endpoint.
  UI should show "Changes take effect on next deploy" notice until this is fixed.
Affects: Constraints & Limits section (show read-only until fixed, or add deploy notice)
```

---

### 🔵 SUGGESTION — Backend ready, UI missing

Safe to build. The backend fully supports this. No owner UI exists yet.
These are build tickets waiting to happen.

**Criteria:**
- OAuth provider is fully wired but no toggle to enable/disable it in the UI
- A setting is stored in the DB with an update API but no config screen exists
- A permission exists and is enforced but isn't shown in the permissions matrix
- An audit log table exists but no UI surfaces it to the owner
- Feature flags exist per-role in the backend but the feature access matrix UI is missing
- An invite API exists but the user management section doesn't have an invite form

**Template:**
```
🔵 SUGGESTION: [Short title]
Backend: [What fully exists]
UI status: [What's missing]
Effort estimate: [low / medium / high]
Recommended action: [Specific UI to build]
Affects: [Sections that will be more complete once this UI is built — or "none"]
```
Note for Phase 2 inline filing: use this template *without* a GAP-N prefix — numbers are assigned only when collecting and outputting in Phase 4.3.

**Example:**
```
🔵 SUGGESTION: GitHub OAuth has no enable/disable toggle in owner UI
Backend: GitHubStrategy fully configured in passport.js. GITHUB_CLIENT_ID + GITHUB_CLIENT_SECRET
  in .env. oauth_providers table has a github row with enabled=true.
  PATCH /api/admin/auth/providers/:name endpoint exists and is tested.
UI status: Authentication section only shows Google OAuth toggle. GitHub is missing.
Effort estimate: low
Recommended action: Add GitHub OAuth row to auth providers list in the dashboard.
  Reuse the same toggle component used for Google OAuth.
Affects: none
```

---

## Special Gap Types

### Orphaned Permission
A permission exists in the permissions matrix (role-permission table) but cannot be found
in any middleware, guard, route decorator, or inline check in the codebase.

> The permission is defined but never checked. Granting or revoking it has no effect.

Always 🔴 CRITICAL. The backend must add enforcement before this permission has any meaning.

```
🔴 CRITICAL: Orphaned permission — reports.export
Backend: Permission `reports.export` exists in the role_permissions table and is granted to Admin and Editor.
  No check for this permission was found in any route handler, middleware, or guard.
UI status: The permissions matrix shows this permission as granted to Admin and Editor.
  Owner can toggle it, but the toggle has no effect on user behavior.
Impact: Granting or revoking `reports.export` currently has no effect — users can access or be blocked from report exports regardless of what the owner sets.
Recommended fix: Add permission check to the report export endpoint/controller.
Blocks: Permissions matrix (reports section), Feature Access (export feature flag)
```

---

### Inherited Permission Not Displayed
The permissions matrix shows a cell as on/off but doesn't distinguish inherited from direct.

Always 🟡 WARNING — the owner will misunderstand what they're editing.

```
🟡 WARNING: Permissions matrix shows no inheritance state
Backend: Editor role inherits 6 permissions from Admin via the role_hierarchy table.
UI status: Permissions matrix shows all 6 as simple green checkboxes.
  Owner removing one of these on the Editor column would have no effect
  because the backend re-grants via inheritance.
Impact: Owner thinks they removed a permission; it's re-granted on next auth check.
Recommended fix: Add three-state cell: directly granted / inherited (locked) / not granted.
  Inherited cells should show source role and link to edit the parent.
Affects: Roles & Permissions section (matrix accuracy)
```

---

### Hardcoded Role Check
Found `role === 'admin'` or similar inline in business logic, outside the central permission system.

Always 🔴 CRITICAL.

```
🔴 CRITICAL: Hardcoded role check bypasses permission system
Backend: src/controllers/billing.controller.ts:88 contains: if (req.user.role !== 'admin') throw new ForbiddenException()
  This check is inline in the controller, outside the central permission system.
UI status: The permissions matrix in the owner dashboard shows billing permissions as owner-configurable.
  Owner granting a custom role 'billing_manager' the billing.read permission appears to succeed.
Impact: Despite what the permissions matrix shows, this billing route blocks anyone who isn't literally 'admin'.
  If owner grants a custom role 'billing_manager' billing.read permission,
  this route will still block them. The permissions matrix has no effect here.
Recommended fix: Replace with `this.permissionsService.check(req.user, 'billing', 'read')`.
  Ensure the billing_read permission is in the permissions matrix.
Blocks: Roles & Permissions (billing permissions), User Management (custom roles)
```

---

### Login Screen Mismatch
The login screen shows buttons that don't match what the backend has enabled,
or the backend has providers that don't appear on the login screen.

Severity depends on direction:
- Button shown, backend provider disabled → 🔴 CRITICAL (user tries to log in, gets error)
- Backend provider active, no button shown → 🟡 WARNING (provider active but unreachable via UI)

```
🔴 CRITICAL: Login screen shows Apple OAuth button, backend has no Apple provider
Backend: No AppleStrategy in passport.js. No APPLE_* env vars. GET /auth/apple returns 404.
UI status: LoginScreen.tsx:42 renders an AppleSignInButton unconditionally.
Impact: User clicks "Sign in with Apple" → immediate 404 error. Auth flow broken.
Recommended fix: Either wire up Apple OAuth in the backend, or remove the button from the login screen.
  Do not add an owner toggle for Apple until the backend strategy is implemented.
Blocks: Authentication & Login section (Apple OAuth sub-section)
```

```
🟡 WARNING: GitHub OAuth is active in backend but has no login button
Backend: GitHubStrategy fully configured in passport.js with GITHUB_CLIENT_ID and GITHUB_CLIENT_SECRET.
  GET /auth/github is a live endpoint that returns tokens.
UI status: The login screen has no "Continue with GitHub" button.
Impact: GitHub OAuth is available and working but users have no way to reach it.
  Any user with a GitHub-linked account cannot log in via that method.
Recommended fix: Add a "Continue with GitHub" button to the login screen, controlled by the owner toggle in Authentication & Login settings.
Affects: Authentication & Login section (GitHub OAuth sub-section)
```

---

### Partially Gated UI Surface
A route or page exists in the frontend and is reachable by any authenticated user, but some UI elements inside it are conditionally shown or hidden by inline permission, flag, or plan checks. The owner dashboard cannot control who *reaches* the route — only what they see once there.

Severity depends on the risk level of what the surface exposes:
- **Route is read-only or low-risk** → 🟡 WARNING. The backend endpoint returning read-only data to all authenticated users is an owner-control gap, but not a write-risk security issue. Fix is adding a route-level guard.
- **Route has write/delete/sensitive access AND backend endpoint is unguarded** → 🔴 CRITICAL. Frontend inline checks are the only barrier and can be bypassed via direct API calls. Backend must be fixed first.

```
🟡 WARNING: Partially gated surface — /admin/reports
Backend: GET /api/reports has no permission check at the endpoint level. Any authenticated user can call it.
UI status: The Reports page renders for all authenticated users. Inside the component, individual chart sections are hidden via hasPermission('reports.analytics') and hasPermission('reports.export') checks, so Viewers see a mostly-empty page.
Impact: Owner cannot restrict access to the Reports page by role. Viewers reach the route and see a degraded view rather than being blocked. The backend endpoint returns all data regardless of role, relying entirely on frontend filtering.
Recommended fix: Add a route-level guard (e.g. requirePermission('reports.read')) to the frontend route and a matching permission check to the backend endpoint. The inline checks can remain for per-feature granularity, but the route itself should require at least one permission.
Affects: Roles & Permissions (reports.read permission), Feature Access (reports feature)
```

```
🔴 CRITICAL: Partially gated high-risk surface — /admin/users/bulk-edit
Backend: PATCH /api/admin/users/bulk has no permission check. Any authenticated user can call it directly.
UI status: The Bulk Edit Users page renders for all authenticated users. Inside the component, the submit button is hidden via hasPermission('users.bulkEdit'), so Viewers see a read-only view — but the backend endpoint is callable regardless.
Impact: Any authenticated user can send a PATCH request directly to /api/admin/users/bulk and modify all users, bypassing the frontend permission check entirely.
Recommended fix: Add a permission check (e.g. users.bulkEdit) to the backend endpoint immediately. Then add a route-level guard to the frontend. The inline check alone is insufficient.
Blocks: Roles & Permissions (users.bulkEdit permission), User Management (bulk actions)
```

---

### Ungated UI Surface
A page, route, component, or nav item exists in the frontend and is accessible to authenticated users regardless of their role, permissions, plan, or feature flag state. The owner dashboard cannot control who sees or uses it because the backend enforces nothing.

Severity depends on what the surface does:
- **Write access, delete, or sensitive data exposed** → 🔴 CRITICAL
- **Read-only or non-sensitive** → 🟡 WARNING

```
🔴 CRITICAL: Ungated high-risk surface — /admin/users/export
Backend: GET /api/admin/users/export has no permission check. Any authenticated user can call it.
UI status: The "Export Users" button is visible and functional for all roles including Viewer.
  No permission guard, feature flag, or plan check controls rendering or execution.
Impact: Any logged-in user can export the full user list. Owner cannot revoke this via the dashboard.
Recommended fix: Add permission check (e.g. users.export) to the backend endpoint and a matching
  frontend guard. Then surface the permission in the permissions matrix so the owner can control it.
Blocks: Roles & Permissions (users.export permission), Feature Access (export feature)
```

```
🟡 WARNING: Ungated read-only surface — /reports/overview
Backend: GET /api/reports/overview returns aggregate data. No permission check on the endpoint.
UI status: The Reports tab is visible in the sidebar for all roles. Any authenticated user can view it.
Impact: Owner cannot restrict report access by role. All users see the same aggregate data.
Recommended fix: Add a permission check (e.g. reports.read) to the endpoint and conditionally render
  the sidebar link based on that permission. Surface reports.read in the permissions matrix.
Affects: Roles & Permissions (reports.read permission), Feature Access (reports feature)
```

---

## Gap Report Format

Output all gaps in this order:
1. All 🔴 CRITICAL gaps (sorted alphabetically by first section name in `Blocks:`; ties broken alphabetically by gap title)
2. All 🟡 WARNING gaps (sorted by the first section name in `Affects:` — alphabetically; gaps with `Affects: none` appear last)
3. All 🔵 SUGGESTION gaps (sorted by effort: low → high; ties broken alphabetically by title)

Note: every CRITICAL gap must name at least one section in `Blocks:`. A CRITICAL gap with an empty `Blocks:` field is incomplete — add the section that cannot be built until this gap is resolved.

End the gap report with this summary table:

| Severity | Count | Sections blocked / affected |
|---|---|---|
| 🔴 Critical | N | list section names (these are blocked) |
| 🟡 Warning | N | list section names (these are affected, not blocked) |
| 🔵 Suggestion | N | list section names (these are affected, not blocked) |
| **Total** | **N** | |

**Dashboard readiness score — single canonical formula:**
```
readiness = max(0, 100 − (critical_count × 20) − (warning_count × 5) − (suggestion_count × 1))
```
Example: 2 critical, 3 warnings, 4 suggestions → 100 − 40 − 15 − 4 = **41% ready**

This is the only definition of the formula. Do not recalculate differently elsewhere.
Attach the score to the Phase 4.1 executive summary.

---

## Gap Lifecycle — Re-scan and Iteration (State C in SKILL.md)

When the user re-scans after making changes, compare new gap list to the previous run.

### Gap states across iterations
| State | Meaning | Output marker |
|---|---|---|
| **Closed** | Condition no longer exists after the fix | ✅ Closed |
| **Remaining** | Gap unchanged | (same entry) |
| **Regressed** | Was closed, now broken again | 🔴 REGRESSED |
| **New** | Introduced by the latest change | 🆕 NEW |

### Diff output format (State C only)
Note on gap numbering in the diff: The diff is output before Phase 4.3, so current-run GAP-N numbers do not yet exist. Use the previous run's GAP-N identifiers for GAPS REMAINING and GAPS REGRESSED entries. GAPS NEW entries will receive their GAP-N numbers when Phase 4.3 runs. Note this explicitly in the diff header: *"GAP-N identifiers below match the previous run. New gaps will be renumbered in the gap report (4.3)."*

```
GAPS CLOSED (N):
  ✅ [previous GAP-N] [gap title] — [what was fixed and verified]

GAPS REMAINING (N):
  [previous GAP-N] [unchanged gap entries]

GAPS REGRESSED (N):
  🔴 REGRESSED: [previous GAP-N] [gap title] — [what changed that broke it again]

GAPS NEW (N):
  🆕 [gap entries — will be assigned GAP-N in section 4.3]

Previous readiness: N%
Current readiness:  N%
Change: +N% / −N%
```

### How to confirm a gap is closed (verify in scan — do not take user's word alone)
- 🔴 CRITICAL (no enforcement): re-scan shows the check/endpoint now exists and is wired to a DB flag
- 🔴 CRITICAL (ungated high-risk surface): re-scan of the frontend shows a permission/flag/plan check on the route or component, AND re-scan of the backend shows the corresponding endpoint now has a matching guard
- 🔴 CRITICAL (partially gated high-risk surface): re-scan of the backend shows the endpoint now has a permission check, AND re-scan of the frontend shows a route-level guard — both must be present; fixing only the frontend is insufficient
- 🟡 WARNING (env-only config): re-scan finds a DB table + runtime update API now handles the value
- 🟡 WARNING (missing inheritance display): re-scan of frontend code shows three-state cell logic
- 🟡 WARNING (ungated read-only surface): re-scan of the frontend shows a conditional render or route guard now controls visibility based on role, flag, or plan
- 🟡 WARNING (partially gated surface): re-scan of the frontend shows a route-level guard (not just inline component checks) now controls access to the route based on role, permission, or flag — the route is no longer reachable by all authenticated users
- 🔵 SUGGESTION (missing UI): re-scan finds the UI section exists and calls the correct endpoint
