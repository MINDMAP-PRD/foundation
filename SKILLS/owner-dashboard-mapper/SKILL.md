---
name: owner-dashboard-mapper
description: >
  Scans a project's backend (routes, middleware, auth, RBAC, feature flags, env vars, schema,
  OAuth providers) AND frontend UI (pages, routes, components) to generate a complete Owner
  Dashboard spec — every section the owner needs to configure the app, what each role can/can't
  do, auth options, feature toggles, data visibility rules, and which UI surfaces are ungated and
  invisible to owner control. Use whenever the user wants to build an admin config dashboard, map
  their backend to a UI, audit RBAC coverage, find gaps in config or frontend gating, or says
  things like "owner should configure", "admin dashboard", "map my backend to a UI", "what can
  owners control", "role-based config", "generate settings screens", "what's missing from my
  config UI", "scan my frontend for unprotected screens", or "check my UI for ungated pages".
  MUST be used any time the user mentions scanning their project for owner-configurable settings
  or auditing their UI for missing access controls. Do not use first for foundation-based project
  intake when the project is still an idea or has no governed package; route to
  foundation-module-factory first.
---

# Owner Dashboard Mapper

## Read First

Before starting, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. `../../foundations/FOUNDATION_MODULE_RULES.md` when the project builds on the foundations
3. `../../foundations/GeneralApp_Build_Documentation.md` when the project builds on the foundations
4. `../../foundations/FormBuilder_Build_Documentation_v6.md` only if reusable form capability is in scope

If a governed package already exists, read it first so dashboard work follows the active contracts, phases, and progress state.

## Intake Routing Rule

This skill maps owner/admin control surfaces from a real or well-specified project.

If the request is still:

- an idea
- an early concept with no governed package
- a module that still needs its build phases/specs/package created

use `foundation-module-factory` first, then return here once the project/package has enough structure to map owner controls safely.

Scans a project's backend and frontend to produce a complete, gap-free Owner Dashboard specification.
Every dashboard section maps to something real — a backend definition, a frontend surface without a gate, or a missing control that the owner should have.
Nothing is invented. Missing controls, mismatched enforcement, and ungated surfaces are flagged loudly, not quietly skipped.

For foundation-based projects, owner/admin UI mapping should assume a `shadcn/ui`-first component baseline unless the governed package explicitly records a different approved UI system.

**Reference files — load only when directed:**
- `references/scan-targets.md` — grep patterns and file lists for each scan category
- `references/component-map-rules.md` — rules for building section blocks from scan data
- `references/gap-rules.md` — severity criteria, templates, and examples for every gap type

---

## Core Principle

> **The dashboard can only surface what is actually enforced — and must expose what isn't.**

If the backend has no concept of Google OAuth, the dashboard has no Google OAuth toggle.
If the backend has a `MAX_SEATS_PER_PLAN` env var but no API to change it, that is a gap.
If a permission exists in the DB but is never checked in code, it is orphaned — 🔴 CRITICAL.
If a UI surface exists with no RBAC gate, no feature flag, and no plan check, the owner has no control over it — that is also a gap.

---

## Entry States — Start Here

Before any scanning, determine which state the user is in:

### State A — "Scan my project"
User provides files, a repo, or pastes code.
→ Proceed to **Phase 1 — Scan**.

### State B — "What's missing from my config UI?"
User has a partial dashboard already and wants gap analysis only.
→ Ask: *"Can you share or describe your existing dashboard sections — what config screens do you already have?"*
→ Proceed to **Phase 1** to scan the backend fully, including 1.8 UI Surfaces.
→ Proceed to **Phase 2** in modified mode (detailed rules at the top of Phase 2):
  - For `ui-surface` entries: check `Gate:` field. Specific named gate → `[EXISTING — gated]`. `Gate: partial` → generate section block with partial gate gap (WARNING for low-risk, CRITICAL for high-risk with unguarded backend). `Gate: none` → generate section block with ungated surface gap.
  - For backend entries: covered with no enforcement problems → `[EXISTING — no gaps detected]`. Covered with `Backend enforces: no/partial`, `Runtime-tunable: no`, or `API endpoint: none` → generate section block to surface the mismatch. Uncovered → generate section block as new.
→ Proceed to **Phase 3** in full — the gap report is the primary deliverable for State B.
→ Phase 4.2 shows both `[EXISTING]` and new sections. Phase 4.6 shows only new sections and mismatch fixes.

### State C — "Re-scan — I've made changes"
User previously ran this skill and has since updated their code.
→ Ask: *"What changed — new tables, endpoints, providers, roles, or env vars?"*
→ Map their answer to specific scan categories using this guide:

| They say | Re-scan these categories |
|---|---|
| "Added a table" or "schema change" | 1.2 RBAC, 1.5 Billing, 1.6 Settings (whichever fits the table) |
| "Added an endpoint" or "new API route" | 1.1 Auth (if auth route), 1.2 RBAC (if permission-checked), all categories for coverage |
| "Added a provider" or "new OAuth" | 1.1 Auth only |
| "Added roles" or "changed permissions" | 1.2 RBAC only |
| "Added feature flags" | 1.3 Flags only |
| "Changed limits or quotas" | 1.4 Constraints only |
| "Changed billing or plans" | 1.5 Billing only |
| "Changed settings or branding" | 1.6 Settings, 1.7 Env |
| "Added new pages" or "added routes" or "added UI screens" | 1.8 UI Surfaces only |
| "Added or changed frontend gating" or "added permission checks to UI" | 1.8 UI Surfaces, 1.2 RBAC (if new permissions added) |
| "I refactored a lot" or vague answer | Re-run all 8 categories — treat as State A full scan |

→ Re-run Phase 2 for sections whose source data changed.
→ Re-run Phase 3 in full — gaps can be resolved or introduced by any change.
→ Output a diff: gaps closed, gaps remaining, gaps newly introduced.

### State D — "No repo yet, planning stage"
User is designing before building.
→ Tell them: *"This skill maps real backend code. Without a backend to scan, I can give you the
  Backend Readiness Checklist — the minimum your backend needs before this dashboard can enforce
  anything. Want that?"*
→ If yes, output the **Backend Readiness Checklist** at the bottom of this file. Stop there.

---

## Phase 1 — Scan

**Load `references/scan-targets.md` now** for file lists and grep patterns per category.

Scan all 8 categories. For each item found, record a **scan entry** in this exact format.
This structured output is what Phase 2 consumes — do not skip it or summarize informally.

```
SCAN ENTRY
Category:        [auth | rbac | flags | constraints | billing | settings | env | ui-surface]
Name:            [human-readable identifier — provider name, role name, flag key, route path, component name, etc.]
Source:          [filename:line — or table name — or env var key]
Type:            [provider | role | permission | flag | limit | setting | env-var | page | component | nav-item | api-route]
Class:           [for env-vars only: Secret | Infrastructure | Configurable | Unclassified — leave blank for all other types]
Value/Default:   [current configured value, or "none" — for ui-surface entries use "none"]
Runtime-tunable: [yes | no | unknown — for ui-surface entries use "n/a"]
API endpoint:    [the endpoint that reads/writes this, or "none" or "unknown"]
UI equivalent:   [if a dashboard section for this already exists in the frontend, name it — otherwise "none yet"]
Gate:            [ui-surface entries only: the RBAC check, permission guard, feature flag, or plan check controlling access — or "none" if ungated — or "partial" if inline checks exist but no route-level guard]
Notes:           [hardcoded? inherited? partially enforced? env-only? ungated? note it here]
```

Note: the `Gate:` field applies only to `Category: ui-surface` entries. Leave blank for all other categories.

Note: `UI equivalent` refers to an *existing* frontend section, not one being designed.
If the project has no owner dashboard yet, this field will almost always be "none yet".
Phase 2 will create the section blocks — do not pre-invent section names here.

Output all scan entries for each category before moving to the next.
This lets the user catch misreadings incrementally.

**Pause after all categories:** Show a scan summary table:

| Category | Items found | Runtime-tunable | Has UI | Needs attention |
|---|---|---|---|---|
| Auth & Login | N | N | N | N |
| RBAC | N | N | N | N |
| ... | | | | |
| UI Surfaces | N | n/a | n/a | N (ungated + partial count) |

**`Needs attention` column definition:** For backend categories (1.1–1.7): count items where `Runtime-tunable: no/unknown`, `Notes` contains "HARDCODED"/"always-on"/"inline check", or `API endpoint: none`. For UI Surfaces (1.8): count entries where `Gate: none` OR `Gate: partial` — ungated surfaces will become gaps; partially gated surfaces will become 🟡 WARNINGs or 🔴 CRITICALs depending on risk level (see scan-targets.md 1.8 Critical checks).

**`Runtime-tunable` column definition:** For backend categories: count items where `Runtime-tunable: yes`. Higher count = more owner-controllable at runtime. For UI Surfaces: always "n/a".

**`Has UI` column definition:** For backend categories: count items where `UI equivalent` is set to a section name (anything other than "none yet"). For UI Surfaces: always "n/a" — the surfaces ARE the UI being catalogued.

Then ask — wording depends on entry state:
- **State A:** *"I found [total] items. Anything missing or wrong before I build the component map?"*
- **State B:** *"I found [total] items. Anything missing or wrong before I run the gap analysis?"*
- **State C:** *"I found [total] items in the re-scan. Anything to correct before I diff against the previous run?"*

Wait for confirmation before proceeding.

### Scan Categories

| # | Category | What to look for (detail in scan-targets.md) |
|---|---|---|
| 1.1 | Auth & Login | OAuth providers, session type, MFA, email verify, SSO/SAML |
| 1.2 | RBAC | Role definitions, permissions, inheritance, guards, inline checks |
| 1.3 | Feature Flags | Flag systems, ENABLE_* vars, plan-gates, per-role overrides |
| 1.4 | Constraints | Rate limiters, quota vars, data scoping middleware, export limits |
| 1.5 | Billing | Plan models, Stripe/Paddle, plan-gating logic, seat limits |
| 1.6 | App Settings | Branding, email, webhooks, locale, notifications, retention |
| 1.7 | Env Vars | Classify every var: Secret / Infrastructure / Configurable / Unclassified |
| 1.8 | UI Surfaces | Pages, routes, nav items, and components that exist in the frontend but lack RBAC gates, feature flag checks, plan guards, or owner-configurable controls |

### Branching — missing or broken categories

| What you find | What to do |
|---|---|
| No auth config at all | 🔴 CRITICAL: "No auth system detected." Do not proceed to Phase 2 — a dashboard cannot exist without auth enforcement. Output the Backend Readiness Checklist and stop. |
| No RBAC at all | 🔴 CRITICAL: "No role/permission system found." Continue scan, but every section in Phase 2 gets `Backend enforces: no`. Entire dashboard is speculative until RBAC is added. |
| No auth AND no RBAC | Both criticals apply. The project is not dashboard-ready at all. Do not proceed to Phase 2. Output the Backend Readiness Checklist, list all other scan findings as context, and stop. There is nothing to map — the owner dashboard has no foundation to sit on. |
| No feature flags | Skip 1.3. Do not generate a Feature Access section in Phase 2. |
| No billing | Skip 1.5. Do not generate a Plans & Billing section in Phase 2. |
| Partial files only | Mark affected categories as incomplete. Those sections get `Backend enforces: unknown` in Phase 2. |
| RBAC entirely hardcoded | Scan and record all inline checks. Every permission entry gets 🔴 CRITICAL in Phase 3. |
| `Unclassified` env vars found | Do not proceed past Phase 1 until these are resolved. Show the unclassified vars and ask: *"I found [N] env vars I can't classify (listed below). Are these secrets, infrastructure config, or owner-configurable settings? Please classify each so I can continue."* Wait for user response before Phase 2. If user cannot classify, mark them `Unclassified` and flag each as 🟡 WARNING in Phase 3: "Env var [X] could not be classified — verify manually whether it is owner-configurable." |
| No frontend code found (1.8) | Skip 1.8. Note it in the summary table. Do not generate ungated surface gaps. This is normal for API-only projects. |

---

## Phase 2 — Build the Component Map

**Load `references/component-map-rules.md` now.**

**State B modified mode:** If this is State B ("What's missing from my config UI?"), apply these rules:
- Before generating any section block, check whether the user's described existing dashboard already covers that scan entry.
- **For `Category: ui-surface` entries:** The "does your dashboard cover this?" check does not apply — the surface is already in the frontend. Instead, check the `Gate:` field:
  - `Gate:` set to a specific guard (not "none" or "partial") → note as `[EXISTING — gated]`.
  - `Gate: partial` → generate the section block and file the gap per the partially gated surface rules in `component-map-rules.md` (🟡 WARNING for low-risk surfaces, 🔴 CRITICAL for high-risk surfaces with unguarded backend endpoints).
  - `Gate: none` (ungated) → generate the section block as normal using the ungated surface handling rules in `component-map-rules.md`.
- **For all other entries:** If covered: set `UI equivalent` on the scan entry, generate a section block marked `[EXISTING]`, and set `Backend enforces` based on the scan findings. Generate this section block — even for covered items — if any of these mismatch conditions are true:
  - `Backend enforces: no` (UI exists but backend doesn't actually enforce it)
  - `Backend enforces: partial` (enforcement is incomplete — e.g. middleware guards but also inline hardcoded checks)
  - `Runtime-tunable: no` (owner saves changes but they have no effect until redeploy)
  - `API endpoint: none` (UI would write to an endpoint that doesn't exist)
  If none of these are true, mark the section `[EXISTING — no gaps detected]` and move on.
- If not covered: generate the section block normally as a new section to build.
- The gap report (Phase 3) is the primary deliverable for State B.
- The dashboard map (Phase 4.2) shows both `[EXISTING]` and new sections, clearly labelled.
- The implementation order (Phase 4.6) shows only new sections and mismatch fixes, not already-correct existing sections.

For all other states, proceed with standard section block generation below.

---

For each scan category that produced entries, generate **section blocks**.
One section block = one screen or sub-screen in the owner dashboard.

**Section block format — every field is required:**

```
SECTION: [Section Name]
Group:              [one of the 8 groups listed below]
Source:             [Phase 1 scan entry Name + Source — must be exact references, not invented]
Backend enforces:   [yes | no | partial | unknown]
Owner can:          [bullet list of specific actions with clear verbs]
Affects roles:      [which roles change behavior when owner saves a change here]
User sees:          [what a user of each affected role actually experiences after the change]
UI component:       [toggle | table | permissions matrix | form | select | matrix | none (gate-adding sections only — see component-map-rules.md step 7) | etc.]
Interconnects with: [SectionName → reason; one line per connection]
Depends on:         [section names that must be built/configured before this one works]
Required by:        [section names that break or lose meaning without this one]
Gaps:               [gap entries using gap-rules.md format — or "none"]
```

When assigning `UI component`, prefer a `shadcn/ui` primitive or a clear composition of `shadcn/ui` primitives first. Only mark a section as needing a custom component when there is no suitable `shadcn/ui` primitive or composition path.

**Field rules:**
- `Source` must reference a Phase 1 scan entry name. No scan entry = no section. Delete it.
- A single scan entry may be cited as `Source` by more than one section block when it genuinely feeds multiple groups (e.g. `DATA_RETENTION_DAYS` feeds both App Settings — where the owner sets the value — and Audit & Compliance — which is governed by it). When this happens, note the relationship: the section that *controls* the value owns it as primary source; the section that *is governed by* it lists it as secondary. Do not duplicate the same `Owner can` action in both.
- `Interconnects with` is **bidirectional**: if A lists B, B must list A. Enforce this.
- `Depends on` / `Required by` must be symmetric: if A depends on B, B's `Required by` must list A.
- `Affects roles` lists which roles change behavior when the owner saves a change here.
  Valid values: specific role names, "all roles", or "all users regardless of role" (for settings like branding, timezone, email sender where the effect is universal).
- `User sees` must be paired with `Affects roles`. Format:
  - If `Affects roles` lists specific roles → one `→ [Role name]: [effect]` entry per role.
  - If `Affects roles` is "all roles" or "all users" → one `→ All users: [effect]` entry is sufficient.
  If a role is listed in `Affects roles` but has no `User sees` entry, the effect on that role is unverified — flag it.
- `Gaps` filed here are also collected into the Phase 3 report. They appear in both places.

**The 8 top-level dashboard groups:**
1. Authentication & Login
2. Roles & Permissions
3. User Management
4. Feature Access
5. Constraints & Limits
6. Plans & Billing
7. App Settings
8. Audit & Compliance

Only generate a group if at least one section block belongs to it.
An empty group means that scan category had no entries — do not fabricate sections.

### Orphan Check (run immediately after all section blocks are written)

Check for all **five** orphan types before Phase 3:

| Orphan type | Detection | Action |
|---|---|---|
| Unlinked scan entry | Phase 1 entry not referenced by any section's `Source` — **excluding** (a) entries with `Type: env-var` and `Class: Secret` or `Class: Infrastructure` (these are correctly non-configurable and should never generate dashboard sections), and (b) `Category: ui-surface` entries with a `Gate:` value that is a specific named guard (not "none" and not "partial") — these are fully gated surfaces correctly handled as `[EXISTING — gated]` without generating a section block. Note: `Gate: partial` entries ARE expected to have a source section block and are NOT excluded from this check. | File 🔵 SUGGESTION: "Backend has [X] with no dashboard section" |
| Sourceless section | Section block `Source` can't be traced to any Phase 1 entry | Delete the section. Log it as removed. |
| One-way connection | A lists B in `Interconnects with` but B doesn't list A | Add reverse to B, or document why it's intentionally one-way |
| Isolated section | Section has zero `Interconnects with` entries | 🟡 WARNING: "Section [X] has no connections — verify it belongs in the dashboard" |
| Asymmetric dependency | A lists B in `Depends on` but B doesn't list A in `Required by` — or B lists A in `Required by` but A doesn't list B in `Depends on` | Add the missing reverse entry. The dependency graph must be symmetric: every `Depends on` has a matching `Required by` and vice versa. |

When the orphan check is complete, all removals are logged and all one-way connections are resolved.
**Proceed immediately to Phase 3.**

---

## Phase 3 — Gap Analysis

**Load `references/gap-rules.md` now** for severity criteria, full templates, and examples.

Collect all gaps from Phase 2 section blocks. Add gaps found during the orphan check.
Deduplicate. Assign final severity.

**Severity levels — summary (full criteria in gap-rules.md):**
- 🔴 **CRITICAL** — Owner has no control: either the dashboard shows a control the backend ignores, or a high-risk UI surface has no gate at all. Do not build UI until fixed.
- 🟡 **WARNING** — Partial or absent owner control: backend has the concept but owner can't fully control it, there's a UI/backend mismatch, or a read-only surface exists with no owner-configurable gate.
- 🔵 **SUGGESTION** — Backend fully ready. Only the UI is missing. Safe to build now.

**Additional gap triggers (check these beyond what Phase 2 already caught):**

| Condition | Severity |
|---|---|
| Permission defined in DB, no middleware/guard/route checks it | 🔴 CRITICAL — orphaned permission |
| `role === 'x'` found outside the central permission function | 🔴 CRITICAL — hardcoded check |
| Login screen shows a button for a provider the backend doesn't have | 🔴 CRITICAL — login mismatch |
| Env-only **feature flag** (controls on/off access to a feature) | 🔴 CRITICAL — toggle has zero effect on user access |
| Config value **hardcoded in source** (not even in env — cannot be changed without code deploy) | 🔴 CRITICAL — owner has no path to change this |
| UI surface exists with `Gate: none` AND the surface provides write access, modifies data, or exposes sensitive content | 🔴 CRITICAL — ungated surface with high-risk access |
| Backend provider active, no login screen button | 🟡 WARNING — unreachable provider |
| Env-only **configuration value** (branding, limits, locale, email) | 🟡 WARNING — owner can change but only via redeploy |
| Role inheritance exists in backend, but matrix only shows on/off | 🟡 WARNING — misleading UI |
| UI surface exists with `Gate: none` AND the surface is read-only or low-risk | 🟡 WARNING — ungated read-only surface |
| UI surface exists with `Gate: partial` AND surface is read-only or low-risk — route reachable by all authenticated users, only inline checks limit what they see | 🟡 WARNING — partially gated surface, no route-level control |
| UI surface exists with `Gate: partial` AND surface has write/delete/sensitive access AND backend endpoint is unguarded | 🔴 CRITICAL — partially gated high-risk surface, backend bypassable |
| Fully enforced feature or setting, no owner toggle exists | 🔵 SUGGESTION — UI missing |
| Audit log table exists, nothing surfaces it to owner | 🔵 SUGGESTION — UI missing |
| `Configurable` env var with no DB-backed equivalent and no owner UI | 🔵 SUGGESTION — UI missing |

**Readiness score:** Calculate using the formula in `gap-rules.md` (Gap Report Format section).
Include this score in the Phase 4 executive summary.

---

## Phase 4 — Output the Dashboard Spec

Output in this exact order. Do not skip sections.

**State C only — output a diff before 4.1 (mandatory, not optional):** Before the executive summary, output the gap diff using the format defined in `gap-rules.md` (Gap Lifecycle section). This diff is required for State C — it is what makes the re-scan useful. Show gaps closed, remaining, regressed, and new — then the readiness score delta. Then continue with 4.1–4.6 as normal, using the current (post-diff) gap list.

### 4.1 Executive Summary
Paragraph covering: tech stack detected, categories scanned (including whether 1.8 UI Surfaces was scanned and how many ungated and partially gated surfaces were found), section blocks built, gap counts (🔴/🟡/🔵), readiness score, and a one-sentence verdict:
*"X sections are ready to build. Y are blocked. Z need warnings resolved first."*
If ungated or partially gated surfaces were found, add a second sentence: *"[N] ungated UI surfaces were found — [N] high-risk (write/sensitive) and [N] read-only. [N] partially gated surfaces were also found (route reachable by all authenticated users). All are included in the gap counts above."* Omit the ungated or partially gated clause if the count is zero.

### 4.2 Dashboard Map
All section blocks from Phase 2, grouped by top-level area.
Within each area, order sections by `Depends on` chain — dependencies appear first.

### 4.3 Gap Report
All gaps from Phase 3: 🔴 first, 🟡 second, 🔵 third.
**Assign a sequential number to each gap as it is output: GAP-1, GAP-2, GAP-3...**
These numbers are used in Phase 4.6 to cross-reference blocked sections.
End with gap summary table and readiness score. (Table format in gap-rules.md.)

Gap entry format with number — use the matching template from gap-rules.md for each severity:
```
GAP-N  🔴 CRITICAL: [title]
Backend: ...
UI status: ...
Impact: ...
Recommended fix: ...
Blocks: [section names] — reference as GAP-N in Phase 4.6 implementation order

GAP-N  🟡 WARNING: [title]
Backend: ...
UI status: ...
Impact: ...
Recommended fix: ...
Affects: [section names or "none"] — note which sections this warning relates to (does not block building them)

GAP-N  🔵 SUGGESTION: [title]
Backend: ...
UI status: ...
Effort estimate: [low / medium / high]
Recommended action: ...
Affects: [section names or "none"]
```

The `GAP-N` identifier assigned here is what Phase 4.6 uses in `[BLOCKED — fix GAP-N first]`.
Only 🔴 CRITICAL gaps block sections in Phase 4.6. WARNING and SUGGESTION gaps inform but do not block.

### 4.4 Orphan List
Backend config items that are correctly classified as `Secret` or `Infrastructure` — deliberately excluded from dashboard coverage and from gap generation. These are items like `DATABASE_URL`, `JWT_SECRET`, `PORT`, `NODE_ENV`: never owner-configurable, correctly absent from the dashboard.

List each with its Phase 1 scan entry details and its classification reason.

**Sequencing note:** The body of 4.4 is output here, in order, between 4.3 and 4.5. However, the closing prompt below is deferred until after 4.6 has been output. Write the orphan entries now, hold the closing prompt, then output 4.5 and 4.6, then append the prompt.

After Phase 4.6, add this closing prompt:
> *"The orphan list contains [N] items classified as Secret or Infrastructure — not owner-configurable. If any of these should actually be configurable at runtime, tell me and I'll reclassify them and re-run the gap check."*

Note: `Configurable` env vars with no UI are handled as gaps (🔵 SUGGESTION), not orphans. `Unclassified` vars are flagged for review during Phase 1 and must be reclassified before Phase 2 proceeds.

### 4.5 Interconnection Map
Text graph of all connections. Format:
```
[Section A] ──[reason]──> [Section B]
[Section B] ──[reason]──> [Section A]
```
Every `Interconnects with` entry from every section block appears here.
Every edge appears twice (forward and reverse) unless intentionally one-directional (note why).
After the graph: list any section that appears in zero edges — isolation warning.

### 4.6 Implementation Order
Derived from `Depends on` / `Required by` across all sections.
Numbered stages. Each stage = sections that can be built in parallel.
Sections with 🔴 gaps are listed but marked **[BLOCKED — fix GAP-N first]**.
Use the exact GAP-N identifier assigned in Phase 4.3. Do not use "gap #N" or any other format.

**Gate-adding sections** (sections created for ungated or partially gated UI surfaces) have `Depends on: none` and `Required by: none` by design, but they cannot be usefully built until their primary `Interconnects with` target exists. The primary target is the section whose permission, flag, or plan the gate will rely on (e.g. Roles & Permissions if a permission gate is being added). When ordering gate-adding sections, place them in the same stage as their primary `Interconnects with` target section — or the stage after it if that section is in a later stage. Do not place them in Stage 1 alongside foundation sections.

```
Stage 1 — Foundation (no dependencies):
  - Auth & Login › Email + Password  ✓ ready
  - Roles & Permissions › Role Builder  ✓ ready

Stage 2 — Depends on Stage 1:
  - Roles & Permissions › Permissions Matrix  ✓ ready
  - User Management › Invite Users  [BLOCKED — fix GAP-2 first: no invite endpoint]

Stage 3 — Depends on Stage 2:
  - Feature Access › Role × Feature Matrix  ✓ ready
  - Constraints & Limits › Rate Limits  ✓ ready
```

**If all or most sections are blocked (readiness score < 40%):**
The build order is not useful — the backend is the bottleneck, not the frontend.
Switch to a **Backend Fix Order** instead:

For each 🔴 CRITICAL gap, count how many section names appear in its `Blocks:` field. This is the unblock count for that gap.

```
Backend Fix Order — resolve these before building any UI:

Fix 1 (unblocks N sections): [gap title] — [specific backend change]
Fix 2 (unblocks N sections): [gap title] — [specific backend change]
...
```

Sort by unblock count, highest first. A gap whose `Blocks:` field lists 4 sections goes before one listing 1. Ties in unblock count are broken alphabetically by gap title — all gaps here are CRITICAL by definition. This gives the user an actionable backend sprint plan instead of a wall of `[BLOCKED]` items.

---

## Phase 5 — Visual Output

If the visualizer tool is available, render after Phase 4:

**5.1 Interconnection graph** (SVG structural diagram)
Nodes = section blocks. Edges = connections from Phase 4.5.
Node color by gap status: gray = ready, amber = has warnings, red = has critical gaps.

**5.2 Permissions matrix** (interactive HTML widget)
Roles as columns with inheritance chain in header (e.g. "Editor ← Admin").
Permissions grouped by resource as rows.
Three cell states: ✅ directly granted (editable) / ↑ inherited (locked, shows source on hover) / ○ not granted (clickable to grant).
Cascade warning when removing a permission from a parent role that has child roles.

**5.3 Gap report table** (color-coded HTML)
Columns: severity, title, sections blocked/affected, recommended fix.
Red rows for critical, yellow for warnings, blue for suggestions.
For critical rows, "sections blocked/affected" lists sections that cannot be built.
For warning and suggestion rows, it lists sections that are related or will benefit — not blocked.

**If the visualizer is not available**, output these as plain text instead:

*5.1 fallback* — ASCII-style connection list grouped by section, already covered by Phase 4.5.

*5.2 fallback* — Markdown table: rows = permissions, columns = roles, cells show D (direct) / I (inherited) / — (not granted).

*5.3 fallback* — the gap report table in Phase 4.3 already serves this purpose. No additional output needed.

---

## Key Rules

**Never invent.** No Phase 1 scan entry = no section block. Suggest it as a gap instead.
*Exception:* Gate-adding section blocks are valid even though the gate being added doesn't yet exist — they derive directly from Phase 1 `ui-surface` scan entries with `Gate: none` (ungated) or `Gate: partial` (partially gated), not from invention. The section exists to expose and remediate what the scan found.

**Never orphan.** Every scan entry must land somewhere:
- `Secret` or `Infrastructure` env vars → appear in Phase 4.4 Orphan List.
- `Configurable` env vars with no UI → become 🔵 SUGGESTION gaps in Phase 3.
- `Unclassified` vars → resolved in Phase 1 before Phase 2 begins.
- `ui-surface` entries with a specific named gate (not "none" and not "partial") → noted as `[EXISTING — gated]` in Phase 2; cross-check their gate for backend enforcement.
- `ui-surface` entries with `Gate: partial` → generate a section block with a partial gate gap (🟡 WARNING for low-risk; 🔴 CRITICAL for high-risk with unguarded backend endpoint). `Backend enforces:` is set to `no` for high-risk and `partial` for low-risk — see component-map-rules.md step 4.
- `ui-surface` entries with `Gate: none` (ungated) → generate a section block in the appropriate group with `Backend enforces: no` and an ungated surface gap.
- All other scan entries (providers, roles, permissions, flags, constraints, settings) → cited as `Source` in at least one section block, or filed as a gap if no section is appropriate.
Nothing falls through without an explicit destination.

**Never isolate.** Every section has at least one `Interconnects with`. Zero connections = 🟡 WARNING.

**Connections are bidirectional.** A→B requires B→A. Enforce this during orphan check.

**`Required by` is the reverse of `Depends on`.** If section A's `Depends on` lists section B, then B's `Required by` must list A. Mismatches break the dependency graph. Verify this during the orphan check.

**`Affects roles` must be specific.** "All roles" is acceptable only for universal settings (branding, timezone, email sender). For auth, permissions, features, and constraints — name the exact roles affected. Vague entries here mean the `User sees` field will also be vague and unverifiable.

**`User sees` must match `Affects roles`.** One entry per named role. If a role is listed in `Affects roles` with no corresponding `User sees` entry, the effect on that role is undocumented — flag it as incomplete.

**Inheritance is explicit.** Every permission cell is tagged direct or inherited. Untagged = gap.

**Hardcoded = unmanageable.** Inline role checks are 🔴 CRITICAL — the dashboard can't govern them.

**Login mirrors auth.** Provider in backend → button on login screen. Mismatch either way = gap.

**GAP-N identifiers are assigned once in Phase 4.3 and reused everywhere.** Never use "gap #N" or any other format. The identifier assigned during Phase 4.3 output is canonical.

---

## Backend Readiness Checklist (State D output)

Minimum backend requirements before the owner dashboard can enforce anything:

**Auth**
- [ ] Provider configs stored in a DB table (not env-only) — enables runtime toggle
- [ ] Each provider has an `enabled` boolean checked before the strategy initializes
- [ ] First-login default role is configurable, not hardcoded

**RBAC**
- [ ] Roles and permissions stored in DB tables, not hardcoded arrays
- [ ] All permission checks go through one central function — no inline `role === 'x'`
- [ ] Role inheritance stored in a `role_hierarchy` or `parent_role_id` field
- [ ] API endpoint to read and write role-permission assignments (owner-only guard)

**Feature Flags**
- [ ] Flags stored in DB or a runtime-configurable system (not env-only)
- [ ] Per-role flag overrides are possible

**Constraints**
- [ ] Rate limits and quotas stored in a `role_constraints` table
- [ ] API endpoint to read and write constraint values per role

**App Settings**
- [ ] `settings` table for runtime-configurable keys
- [ ] API endpoint to read and write settings (owner-only guard)

**Audit**
- [ ] Every write to roles, permissions, and settings creates an audit log entry
- [ ] Audit log records: actor, target, old value, new value, timestamp
