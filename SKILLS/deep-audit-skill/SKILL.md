---
name: deep-audit
description: >
  Exhaustive, recursive, multi-dimensional audit of any codebase element — components, API endpoints, database schemas, service classes, cron jobs, webhooks, auth systems, config systems, environment files, Docker configs, CI/CD pipelines, GraphQL schemas, and IaC. Trigger on: "audit", "review", "inspect", "drill into", "does this actually work?", "what's wrong with this?", "trace this flow", "are there hardcoded values", "is this wired up", "does this fetch actually fetch", "check all the states", "leave no stone unturned". Goes far beyond surface review — recurses every sub-path to terminal ends, applies 10 lenses per element, verifies each element actually does what it claims, checks all states/branches/edges, traces dependency graph upstream and downstream, verifies configuration ownership, finds hardcoded values, detects disconnected elements, confirms fetches execute, and surfaces every error, mismatch, and gap with severity ratings. Use for any deep technical review.
---

# Deep Audit Skill

## Read First

Before starting, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. `../../foundations/FOUNDATION_MODULE_RULES.md` when auditing a foundation-based module or package
3. `../../foundations/GeneralApp_Build_Documentation.md` when the audited project builds on the foundations
4. `../../foundations/FormBuilder_Build_Documentation_v6.md` only if the audited project uses reusable form-engine capability

If a governed package already exists, read that package first and treat it as the working continuity layer.

## Intake Routing Rule

This skill is for audit work, not project intake.

If the user actually needs:

- a new project/module package
- phase planning
- folderized build specs
- progress continuity

route to `foundation-module-factory` first, then return here for implementation or spec audits.

When the audit scope includes API routes, endpoint contracts, pagination, response shapes, versioning, or rate limiting, use `api-design` as the contract-design lens and `secure-coding-practices` as the security lens rather than folding both concerns into a generic audit pass.

When the audit scope includes repository/service layering, middleware composition, query efficiency, caching, background jobs, or logging structure, use `backend-patterns` as the backend-architecture lens rather than treating backend shape as a generic implementation concern.

You are performing a **forensic, recursive, exhaustive audit**. The goal is not to summarise — it is to find everything that is wrong, incomplete, disconnected, hardcoded, missing, duplicated, or mismatched. You drill until you hit bedrock. You never accept "it exists" as proof that "it works."

---

## Static Analysis Constraint — Read This First

You are performing **static analysis only**. You cannot run the code, observe runtime behaviour, or confirm execution in a live environment. This constraint must never be hidden from findings.

**Three Verification Levels — use exactly one on every functional claim:**

| Level | Label | Meaning |
|-------|-------|---------|
| ✅ STATIC-CONFIRMED | Code path provably executes | The call chain is unambiguous: the function is called, the condition evaluates to true, the data is available. No runtime surprises possible from the code alone. |
| ⚠️ STATIC-INFERRED | Likely executes based on structure | The code appears wired correctly but depends on runtime values (env vars, DB records, user state) that cannot be verified statically. Flag as inferred, not confirmed. |
| ❌ UNVERIFIED | Cannot confirm without runtime evidence | Execution depends on external state, feature flags, or dynamic dispatch that is not traceable through static analysis. Treat as suspect until proven otherwise. |

Apply these labels everywhere the 5 Questions ask "Does it ACTUALLY do it?" — write the label, not just your opinion.

Never write a finding as CRITICAL based on STATIC-INFERRED alone — downgrade to HIGH and note that runtime confirmation is needed.

---

## Core Audit Philosophy

**The Three Laws of Deep Audit:**

1. **Existence ≠ Functionality.** A function being defined does not mean it is called. A fetch being written does not mean it executes. A config key being present does not mean it is read. Always verify the actual runtime behaviour, not just the declaration.

2. **Every Path Has an End.** When you find that component A calls service B, you audit service B. When service B queries table C, you audit that query. You do not stop until you have traced every sub-path to its terminal node.

3. **Every Element Has an Owner.** Who configures this? Is it configurable at all? Are the values hardcoded? Should the platform owner be able to change this from an admin panel? Is that admin panel wired to this value?

---

## Audit Scope — Determine First

Before beginning, identify the audit scope.

**Scope is clear when:** (a) a specific file, component, feature, or table is named AND (b) the audit type is inferrable from context. If either is missing, ask once before proceeding.

| Scope Type | Description | Checklist |
|------------|-------------|-----------|
| **Component Audit** | Single UI/backend component, class, or function | `references/audit-checklists.md` → match element type |
| **Module Audit** | A feature folder, service layer, or domain | `references/audit-checklists.md` → apply per element type within module |
| **Flow Audit** | A complete user journey end-to-end (e.g., "user books appointment") | All checklists for each element type in the flow |
| **Endpoint Audit** | An API route and its full execution chain | `references/audit-checklists.md` → API Endpoint |
| **Schema Audit** | Database tables, relations, constraints, indices | `references/audit-checklists.md` → Database Table |
| **System Audit** | Full application or subsystem | `references/system-audit-phasing.md` (required — phases the work) |
| **Config Audit** | Environment, admin settings, feature flags, and their consumers | `references/audit-checklists.md` → Config / Settings System |
| **Infrastructure Audit** | Docker, CI/CD pipelines, .env files, IaC, GraphQL schema | `references/audit-checklists.md` → match specific infra type; apply 10 lenses using the infra lens mapping below |

For **Infrastructure Audit**, the 10 lenses map to infrastructure elements as follows — do not skip a lens because it "seems like a code concept":

| Lens | Infrastructure Application |
|------|---------------------------|
| L1 Identity | Does the file's name and structure match its actual purpose? Is a `Dockerfile` actually building what it claims? Is a pipeline YAML named `deploy.yml` actually deploying? |
| L2 Functional | Is this file actually used? Is the Dockerfile referenced in a compose file? Is the pipeline actually triggered? Is the IaC module instantiated? |
| L3 State Machine | For **CI/CD pipelines**: does the pipeline handle all states — build success, build failure, test failure, deploy success, deploy failure, rollback? Are all branch conditions covered? For **IaC (K8s/Terraform)**: are rolling update, rollback, and degraded states handled? For **Dockerfile** and **.env files**: no state machine applies — mark ✅ Clean and move on. |
| L4 Dependencies | What does this infra element depend on? Base images, external secret stores, other pipeline jobs, upstream modules. Do those exist? |
| L5 Configuration | Are environment-specific values hardcoded? Should replica counts, resource limits, image tags be configurable? Who controls them? |
| L6 Data Flow | For CI/CD: does the artifact produced in the build step actually reach the deploy step? For IaC: does output from module A correctly wire into module B? |
| L7 Interconnection | Is the Dockerfile's exposed port actually what the load balancer routes to? Is the health check URL the one the app actually responds on? |
| L8 Duplicates | Are there two Dockerfiles, two pipeline files, two IaC modules doing the same thing? Are there conflicting env var definitions? |
| L9 Error Handling | Does the pipeline fail loudly on error? Does the IaC rollout strategy handle failures? Is there a rollback path? |
| L10 Security | Are secrets in plaintext? Is the image running as root? Are network policies open by default? Are state files committed? |

For **System Audit**, always open `references/system-audit-phasing.md` before any analysis begins.

---

## The 10 Audit Lenses

Apply ALL 10 lenses to every element. Do not skip a lens because it "seems inapplicable" — state explicitly that it was checked and found clean if so.

---

### LENS 1 — IDENTITY & PURPOSE

> *What is this element? Does its name, documentation, and implementation agree?*

- What does this element **claim** to do? (name, comments, docs, type signature)
- What does it **actually** do? (read the implementation, not just the interface)
- Does the name accurately reflect the behaviour?
- Are comments/docstrings accurate, stale, or missing?
- Is this a god-object doing too many things? Does it violate single responsibility?
- Is this element actually used anywhere, or is it dead code?

**⚡ Dead Code Short-Circuit:** If Lens 1 confirms the element has zero callers, zero imports, and is never rendered or scheduled — log `DEAD-01` (see `references/common-findings.md`), set VERDICT to `🔘 DEAD CODE`, and **skip Lenses 2–10 entirely**. There is no runtime behaviour to audit.

**Drill trigger:** If claimed purpose ≠ actual behaviour → CRITICAL finding. Trace what callers expect vs. what they get.

---

### LENS 2 — FUNCTIONAL VERIFICATION

> *Does it actually do what it promises? Prove it.*

Check `references/common-findings.md` patterns: `FETCH-01` (fetch function defined but never called — feature silently non-functional) and `DEAD-01` (a function or method *within* this element that is never invoked from any execution path — distinct from top-level dead code, which is caught by the Lens 1 short-circuit before reaching this lens).

- Is this element **invoked**? Where? When? Under what conditions?
- Walk the execution path: entry point → logic → output. Does each step actually execute?
- Is there a conditional that could prevent it from ever running? (feature flag, role check, env guard)
- Does it produce its **promised output**? (return value, side effect, emitted event, DB write)
- Is the output consumed correctly by callers?
- Are there tests verifying it works? Do those tests cover the real path or mock everything away?

**Drill trigger:** For every function called inside, recurse — does THAT function actually work?

---

### LENS 3 — STATE MACHINE COMPLETENESS

> *What are all possible states? Are all transitions and branches handled?*

**Lens 3 owns:** Which states exist, which are reachable, and whether all transitions and branches are defined. It does NOT own error handling quality or recovery UX — those belong to Lens 9. It does NOT own fetch error handling — that belongs to Lens 6. If you find a missing error state, log it here; if you find that the error message is unhelpful, log that in Lens 9.

Check `references/common-findings.md` patterns: `STATE-01` (missing error state), `STATE-02` (missing empty state), `STATE-03` (in-flight indicator never resets — loading spinner, job stuck at "processing", queue message stuck "in-flight").

Ask for every stateful element:

**States:**
- What states does this element have? (loading, idle, success, error, empty, partial, locked, expired, pending, cancelled…)
- Are ALL states defined and reachable?
- Are there implicit states that are never explicitly handled? (e.g., "what if the list is empty?")
- Is there a default/initial state? Is it correct?

**Transitions:**
- What events trigger state transitions?
- Are all transitions bi-directional where needed? (can you go back from error to idle?)
- Are there missing transitions? (e.g., no way to retry after error)
- Are there impossible transitions that are nonetheless reachable in code?

**Branches:**
- Audit every `if`, `else`, `switch`, `ternary`, `guard clause`
- For `try/catch`: Lens 3 owns whether all branches inside `try` are reachable and handled. Lens 9 owns whether the `catch` block correctly handles or propagates the error. Do not duplicate — note the try/catch exists in L3 as a branch structure, and check the catch body quality in L9.
- Is every branch reachable?
- Is every branch handled?
- What happens in the `else` case when there is no explicit `else`?
- What happens when an array is empty? When a value is null/undefined/0/false/""?

**Race Conditions:**
- Can two state transitions fire simultaneously and corrupt state?
- Are async operations guarded against stale responses?

---

### LENS 4 — DEPENDENCY GRAPH

> *What does this need? What needs this? What happens if either side breaks?*

Check `references/common-findings.md` patterns: `DEPGRAPH-01` (missing dependency — app won't compile) and `DEPGRAPH-02` (circular import OR circular DI constructor dependency — undefined runtime behaviour or container failure).

**Upstream (what this element depends on):**
- External APIs / microservices
- Database tables, collections, views
- Other components, modules, classes, functions
- Config values, environment variables, feature flags
- Auth/session context, user object, permissions
- Third-party libraries
- File system, queues, caches

For each dependency:
- Does it exist?
- Is the correct version/interface being used?
- What happens if it is unavailable? Is there a fallback?
- Is it being imported/injected correctly?
- Is it tested in isolation or mocked away everywhere?

**Downstream (what depends on this element):**
- Which components consume this element's output?
- Which routes, jobs, crons, webhooks call this?
- Which other elements would break if this element was removed or changed?
- Are callers protected from breaking changes? (versioned APIs, typed contracts)

**Circular Dependencies:**

Check `references/common-findings.md` `DEPGRAPH-02` which covers both forms:
- **Circular imports:** Does A import B which imports A? Look for "Cannot access before initialization" runtime errors.
- **Circular DI:** Does Service A's constructor require Service B, which requires Service A? Look for DI container errors or infinite recursion at startup.
- Does A → B → A exist anywhere in the broader dependency graph?

---

### LENS 5 — CONFIGURATION AUDIT

> *How is the owner supposed to configure this? Are they actually able to?*

Check `references/common-findings.md` patterns: `CONFIG-01` (hardcoded business value — owner cannot change without a deploy) and `CONFIG-02` (admin panel writes to wrong field — admin control is an illusion).

This lens is critical for multi-tenant, SaaS, and admin-managed systems.

**Configurability check:**
- What values in this element SHOULD be owner-configurable vs. system-fixed?
- Examples of values that are almost always wrong to hardcode: timeouts, rate limits, prices, feature thresholds, email addresses, URLs, copy text, colours, pagination sizes, role names, permission sets, notification templates.

**Hardcoded value detection:**
- Scan for string literals, numeric literals, and boolean literals that represent business logic.
- For each: should this be in DB config? ENV? Admin panel? Feature flag?
- Flag: `const MAX_RETRIES = 3` — is 3 correct? Can the admin change it? Should they be able to?

**Admin wiring:**
- If this value is configurable via the admin panel, is the admin panel actually reading/writing the right DB field?
- Is there a config table/document/record that stores this? Is it seeded? Is it missing?
- Is the config loaded at startup or per-request? Is it cached correctly?
- What happens with missing or invalid config? (crash? silent default? wrong default?)

**Multi-tenant isolation:**
- Are config values properly scoped to tenant/org/user?
- Is there a risk of one tenant's config leaking to another?

---

### LENS 6 — DATA FLOW & FETCH AUDIT

> *Is there a fetch here? Does it actually fetch? What does it do with the response?*

**Lens 6 owns:** Everything about the fetch itself — trigger, headers, request shape, response shape, HTTP status handling, loading/stale state, and cache invalidation. For fetch error handling (try/catch, timeout, retry), log findings here. For business logic errors and recovery UX, use Lens 9.

Before auditing, check `references/common-findings.md` patterns: `FETCH-01`, `FETCH-02`, `FETCH-03` — these are the three most common fetch failures.

For every fetch, HTTP call, or external network read/write:

**Scope note:** Database queries are not in Lens 6 scope — they have no HTTP method, headers, or status codes. For DB query concerns (injection, tenant scoping, N+1, result shape), use the Database Table checklist and Lens 10. Lens 6 covers: REST API calls, GraphQL queries, webhook deliveries, message queue publishes, any call that goes over a network socket.

**Request audit:**

Note: The questions below are written for HTTP/REST calls. For non-HTTP network calls (message queue publishes, gRPC, WebSocket sends), skip HTTP-specific questions (method, status code) and focus on: message/payload shape, auth credentials, endpoint configurability, error handling, and whether the operation is idempotent.

- What is the method? (GET/POST/PUT/DELETE/PATCH) *(HTTP only)*
- What are the required headers? (auth token, content-type, tenant ID) *(HTTP only; for queues: equivalent auth/credential mechanism)*
- Are headers actually set?
- What is the request body/params shape?
- Is required data available at the time of the fetch?
- Is the fetch triggered at the right time? (on mount? on event? on demand? on a schedule?)
- Can the fetch be triggered multiple times simultaneously? (debounce/deduplicate needed?)

**Response audit:**
- What shape does the response have?
- What shape does the code ASSUME the response has? Do they match?
- Is the response validated/typed before use?
- Is the entire response used, or only part of it? Are unused fields a concern?
- Is pagination handled?

**State management:**

Note: Lens 3 owns whether loading/error/empty states *exist* in the element. Lens 6 owns whether the fetch *correctly triggers and releases* those states. Both lenses look at in-flight state, but from different angles — log "state is missing entirely" under L3, log "fetch doesn't set the in-flight indicator before firing" or "in-flight indicator is never cleared" under L6.

- Does the fetch set a loading/in-flight indicator before it fires? Is that indicator cleared in *both* success and error paths? (If stuck forever → `STATE-03` in common-findings.md)
- Is the previous result cleared or preserved before the new fetch starts? Is that intentional?
- Is stale data served or displayed while refreshing? Is that intentional?
- Is the result cached? Should it be? For how long?
- After a mutation (POST/PUT/DELETE), is related data re-fetched/invalidated?

**Error handling:**
- Is the fetch wrapped in try/catch or `.catch()`?
- Is the HTTP error status checked before parsing the body? *(HTTP only — for non-HTTP calls, check whether error/failure signals from the transport layer are handled: queue nack, gRPC error code, WebSocket close event)*
- Is there a retry mechanism for transient failures?
- Is there a timeout on the request?

Note: Whether the caught error is then *correctly propagated or surfaced* to the caller or user is Lens 9's concern — do not audit that here.

---

### LENS 7 — INTERCONNECTION AUDIT

> *How does this connect to the rest of the system? Is it accidentally isolated?*

Check `references/common-findings.md` patterns: `WIRE-01` (element exists but is disconnected from its data source or consumer — UI component, service, job, or IaC resource that is a shell) and `WIRE-02` (callback or handler defined but never invoked — flow silently breaks at the handoff point).

**Wiring check:**
- **Frontend:** Are props passed correctly? Are required props actually provided by the parent? Are event handlers attached to the right functions? Are callback props passed AND called (e.g. `onSuccess` that is never invoked)?
- **Backend/services:** Are constructor dependencies injected? Are interface contracts implemented? Are callback/handler parameters actually invoked at the right point? Are event listeners registered on the correct event source?
- Are signals/observables/stores subscribed to correctly?
- Are emitted events (frontend: DOM events, custom events; backend: message queue publications, webhook fires) listened to somewhere?

**Isolation detection:**
- Is this element instantiated, registered, or rendered but producing no output — effectively a shell?
- Is a service instantiated but never injected anywhere?
- Is a database table populated but never queried?
- Is a config value set but never read?
- Is a route registered but never linked to from anywhere?

**Contract verification:**

For response shape mismatches, check `references/common-findings.md` pattern `FETCH-02`. For request body or field-name mismatches, log as a contract finding with the specific mismatch described — there is no exact pattern ID but severity is 🔴 CRITICAL (runtime crash or silent data corruption).

Examples — adapt to your element's context:
- Frontend sends field `user_id` — backend expects field `userId` → request contract mismatch ❌
- Service A emits queue message `{ "order_id": 123 }` — Service B consumer expects `{ "orderId": "..." }` → message schema mismatch ❌
- API returns `response.results` — consumer accesses `response.data.items` → response shape mismatch → `FETCH-02` ❌
- Database column is `INTEGER` — service reads it as `STRING` without casting → type contract mismatch ❌
- Any mismatch between what one element produces and what the consumer expects is a contract finding.

---

### LENS 8 — DUPLICATE & CONFLICT DETECTION

> *Is this logic duplicated? Are there mismatches? Which version wins?*

Check `references/common-findings.md` patterns: `DUPE-01` (logic implemented twice — will diverge) and `DUPE-02` (two routes handling the same resource action).

- Is the same logic implemented in two or more places? (especially: validation, formatting, calculation)
- Are there two functions with the same name in different files?
- Are there two database columns storing the same data?
- Are there two config keys representing the same setting?
- Are there two routes that handle the same endpoint?
- Is the same API called in multiple places with different parameters/assumptions?
- Are there conflicting migrations? Schema drift between environments?
- Is a constant defined in multiple places with different values?
- Are there two RBAC checks for the same action that could disagree?

---

### LENS 9 — ERROR HANDLING AUDIT

> *What can go wrong? What actually happens when it does?*

**Lens 9 owns:** Business logic errors, runtime exceptions, recovery UX, silent catches, global error boundaries, and error message quality. Fetch-specific error handling (HTTP status, timeout, retry) is owned by Lens 6 — do not duplicate those findings here.

Before auditing, check `references/common-findings.md` pattern `ERROR-01` (silent catch — the most dangerous error handling failure).

For every failure mode:

**Identification:**

Lens 9 identifies failures that are NOT fetch/HTTP errors (those are Lens 6). Focus on:
- What runtime errors can occur? (null dereference, type mismatch, division by zero, index out of bounds, promise rejection not caught)
- What business logic errors can occur? (insufficient balance, expired token, duplicate record, constraint violation, invalid state transition)
- What third-party service failures can occur? (payment gateway down, email provider unreachable, storage quota exceeded, rate limit hit)
- What data integrity errors can occur? (orphaned record after partial write, stale cache serving wrong data, concurrent modification conflict)

**Handling:**
- Is each error caught at the right level?
- Are errors swallowed silently anywhere? (`catch(e) {}` with no body = CRITICAL)
- Is the error surfaced to the appropriate consumer in a meaningful way? For UI: does the user see a visible error? For services: is an error response returned to the caller? For jobs: is a failed status set and logged? For APIs: is a non-2xx response returned with a structured error body?
- Is the error message useful to the developer? To the end consumer?
- Is a generic "something went wrong" the only message surfaced to the consumer? (acceptable in prod, flag for improvement — applies to user-facing UI messages, error response bodies in APIs, and logged messages in jobs equally)
- Are errors logged with enough context to debug? (request ID, user ID, timestamp, stack trace)
- Is there a global error boundary / handler?

**Recovery:**
- Can the caller/user recover from the error? (retry button for UI; retry logic for services; dead-letter queue for jobs; alternative path for any consumer)
- Does the application state become corrupted after an error? (partial writes, orphaned records)
- Are failed async operations rolled back?

---

### LENS 10 — SECURITY & ACCESS AUDIT

> *Who can reach this? Who should be able to? Are inputs safe?*

**⚠️ SECRET REPRODUCTION RULE:** When a hardcoded secret is found (API key, password, token, private key), describe its **location and nature only** — never reproduce its value in audit output. Write: `Hardcoded API key at [file:line] — REDACTED. Move to environment variable.` Reproducing the secret in findings output defeats the purpose of flagging it.

Check `references/common-findings.md` patterns `SEC-01` (permission check missing at the enforcement layer — covers UI-only gatekeeping, resolver-level gaps, internal service trust, and background job scope) and `SEC-02` (missing tenant isolation).

- Is this element gated by authentication? Should it be?
- Is this element gated by authorization (RBAC, permission check)? Should it be?
- Does the permission check happen at the right layer? (not just UI — must also be server-side)
- Is data filtered per-user / per-tenant? (can user A see user B's data?)
- Are inputs sanitised? (SQL injection, XSS, path traversal, command injection)
- Are file uploads validated? (type, size, content)
- Are secrets (API keys, passwords) hard-coded or logged?
- Are sensitive fields excluded from logs and error messages?
- Are tokens short-lived? Are refresh flows implemented?
- Is the rate limit enforced on this endpoint?

---

## Recursive Drilling Protocol

When a lens reveals a dependency, call, or reference — you recurse into it. Follow this protocol:

```
FINDING: [ElementA] calls [ElementB.methodX()]
  → DRILL: Does ElementB exist?
    → DRILL: Does methodX() exist on ElementB?
      → DRILL: Does methodX() actually do what ElementA expects?
        → DRILL: What does methodX() depend on?
          → DRILL: Are those dependencies available at runtime?
            → TERMINAL: [Finding / Clean]
```

**Audited Registry — maintained throughout the entire audit:**

Before drilling into any element, check whether it is already in the Audited Registry. If it is, write `→ [ALREADY AUDITED: see [element-id]]` and stop — do not re-drill.

The registry has two states per element:

- `[PARTIAL]` — Lenses 3–6, 9–10 complete (added after Step 3). Prevents re-drilling during recursive sub-path traversal.
- `[COMPLETE]` — All 10 lenses done including L7+L8 (upgraded after Step 4). Used in final report.

Add elements as `PARTIAL` immediately after Step 3 finishes on them. Upgrade to `COMPLETE` once Step 4 fills in their L7/L8 rows.

```
AUDITED REGISTRY:
  [AE-001] AuthService          — PARTIAL  (L3-6,9-10: ✅ CLEAN)
  [AE-002] UserRepository       — PARTIAL  (L3-6,9-10: 🟠 HIGH-003, HIGH-007)
  [AE-003] /api/auth/login      — COMPLETE (L3-6,9-10: 🔴 CRITICAL-001 | L7: ✅ Clean | L8: ✅ Clean)
```

Stop drilling only when you reach:
- A well-tested, verified external library
- A native platform API
- A confirmed working, in-production external service
- An element already in the Audited Registry at PARTIAL or COMPLETE state (cite registry ID)

---

## The 5 Questions Rule

For EVERY element audited — no exceptions — answer these 5 questions explicitly:

1. **What does it CLAIM to do?** (declared intent — name, comments, type signature)
2. **Does it ACTUALLY do it?** (use a verification label: `STATIC-CONFIRMED` / `STATIC-INFERRED` / `UNVERIFIED` — never just state "yes")
3. **What happens when it FAILS?** (all error/edge paths — unhandled failure modes are findings)
4. **Who CONFIGURES it, and how?** (owner control, admin wiring, hardcoded values that block owner control)
5. **What BREAKS if this element is removed?** (downstream impact — every caller, every consumer)

---

## Output Format

**For elements WITH findings** — use this full structure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 AUDIT: [Element Name]
📍 Location: [file path or layer]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

5 QUESTIONS:
  Claimed purpose : [what it says it does]
  Actual function : [STATIC-CONFIRMED / STATIC-INFERRED / UNVERIFIED] — [what it does]
  Failure modes   : [what happens when it fails]
  Configuration   : [who configures it, how, hardcoded values]
  Removal impact  : [what breaks]

LENS FINDINGS:
  L1 Identity       : ✅ Clean / ⚠️ [finding] / ❌ [finding]
  L2 Functional     : ✅ Clean / ⚠️ [finding] / ❌ [finding]
  L3 State Machine  : ✅ Clean / ⚠️ [finding] / ❌ [finding]
  L4 Dependencies   : ✅ Clean / ⚠️ [finding] / ❌ [finding]
  L5 Configuration  : ✅ Clean / ⚠️ [finding] / ❌ [finding]
  L6 Data Flow      : ✅ Clean / ⚠️ [finding] / ❌ [finding]
  L7 Interconnect   : ⏳ Pending (cross-element — filled after Step 4)
  L8 Duplicates     : ⏳ Pending (cross-element — filled after Step 4)
  L9 Error Handling : ✅ Clean / ⚠️ [finding] / ❌ [finding]
  L10 Security      : ✅ Clean / ⚠️ [finding] / ❌ [finding]

Note: L7 and L8 rows show ⏳ Pending during Step 3. After Step 4 (cross-element pass) completes, update these rows to ✅ Clean / ⚠️ / ❌ and add any findings to FINDINGS DETAIL.

FINDINGS DETAIL:
  🔴 [CRITICAL-001] [Description] → [Sub-path drilled] → [Terminal finding]
  🟠 [HIGH-002]     [Description] → [Drill result]
  🟡 [MEDIUM-003]   [Description]
  🔵 [LOW-004]      [Description]
  ⚪ [INFO-005]     [Observation]

SUB-PATHS DRILLED:
  → [ElementA] calls [ElementB] → [ElementB] calls [ServiceC] → [ServiceC] MISSING ❌
  → [fetch /api/users] → [response shape mismatch: expects {data}, receives {users}] ❌
  → [config.timeout] → [hardcoded as 5000ms, not owner-configurable] ⚠️

VERDICT: 🔴 BROKEN / 🟠 INCOMPLETE / 🟡 FUNCTIONAL WITH GAPS / ✅ CLEAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**For elements with ALL LENSES CLEAN** — use this compact form. **Only valid after Step 4 completes and L7+L8 resolved to Clean for this element.** Do not write this during Step 3:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 AUDIT: [Element Name] — ✅ ALL LENSES CLEAN
📍 Location: [file path or layer]
  Actual function : [STATIC-CONFIRMED / STATIC-INFERRED] — [one-line description]
  L1–L10 (all passes): No findings.
VERDICT: ✅ CLEAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**For dead code** — use this after Lens 1 short-circuit:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 AUDIT: [Element Name] — 🔘 DEAD CODE
📍 Location: [file path or layer]
  DEAD-01: Zero callers / zero imports / never rendered or scheduled.
  Lenses 2–10 skipped — no runtime behaviour to audit.
VERDICT: 🔘 DEAD CODE — Remove or wire up.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Severity Scale

| Symbol | Level    | Meaning |
|--------|----------|---------|
| 🔴     | CRITICAL | Element is broken, non-functional, causes data loss, or is a security vulnerability. Must fix immediately. |
| 🟠     | HIGH     | Element is functional but incomplete — missing states, disconnected wiring, hardcoded values that block owner control, missing error handling on critical paths. |
| 🟡     | MEDIUM   | Logic duplication, non-critical missing states, partial error handling, schema drift, stale docs. |
| 🔵     | LOW      | Minor naming issues, cosmetic mismatches, non-blocking improvements. |
| ⚪     | INFO     | Observations, architectural notes, technical debt flags with no immediate impact. |
| 🔘     | DEAD CODE | Element exists but has zero callers, zero imports, is never rendered or scheduled. Not a bug — a removal or wiring decision. |

Note: `⚪ INFO` and `🔘 DEAD CODE` are distinct symbols. Do not confuse them in output or summaries.

**Per-lens row symbols** (used in the LENS FINDINGS grid of the output template):

| Symbol | Meaning |
|--------|---------|
| ✅ Clean | This lens found nothing wrong. State this explicitly — do not leave it blank. |
| ⚠️ | Finding of MEDIUM or LOW severity on this lens. |
| ❌ | Finding of CRITICAL or HIGH severity on this lens. |

The per-lens row is a quick-scan indicator. The full detail, ID, and drill path goes in FINDINGS DETAIL below it.

---

## Audit Summary Report

After all elements are audited, produce:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 AUDIT SUMMARY REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Scope       : [what was audited]
Elements    : [N] audited  ([N] with findings / [N] clean / [N] dead code 🔘)
Sub-paths   : [N] drilled
Depth       : [max recursion depth reached]

FINDING COUNTS:
  🔴 Critical : [N]
  🟠 High     : [N]
  🟡 Medium   : [N]
  🔵 Low      : [N]
  ⚪ Info     : [N]
  🔘 Dead Code: [N] elements — no runtime behaviour, removal/wiring decision pending
  ✅ Clean    : [N] elements — no findings

TOP CRITICAL ISSUES (must fix first):
  1. [CRITICAL-XXX] [Element] — [one-line description]
  2. ...

DISCONNECTED ELEMENTS (exist but are wired to nothing):
  - [Element] at [path]

HARDCODED VALUES REQUIRING ADMIN WIRING:
  - [Value] in [Element] — should be configurable via [suggested mechanism]

MISSING STATE HANDLERS:
  - [Element] has no [error/empty/loading] state

DEAD CODE CANDIDATES:
  - [Element] — defined but never called/imported/rendered

SCHEMA/CONTRACT MISMATCHES:
  - [ElementA sends X, ElementB expects Y — e.g. service returns {data}, consumer accesses .items]

VERIFICATION COVERAGE:
  STATIC-CONFIRMED : [N] elements — high confidence
  STATIC-INFERRED  : [N] elements — runtime confirmation recommended
  UNVERIFIED       : [N] elements — cannot confirm without execution

RECOMMENDED FIX ORDER:
  Phase 1 (Critical)  : [list]
  Phase 2 (High)      : [list]
  Phase 3 (Medium)    : [list]

OVERALL SYSTEM HEALTH: 🔴 / 🟠 / 🟡 / ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Audit Execution Protocol

### Step 0: Code Acquisition (REQUIRED before all else)

If no code is present in context:
- Ask the user to paste the relevant code, upload files, or specify exact file paths.
- Do not begin any analysis until you have the actual source.
- Do not infer, guess, or generate placeholder findings for code you have not read.

If partial code is present (some files referenced but not provided):
- List exactly which files you have and which are missing.
- State which lenses cannot be fully applied without the missing files.
- Proceed with what you have, flagging gaps explicitly as `UNVERIFIED — source not provided`.

**Context Window Overflow:** If the provided codebase is larger than can be fully analysed in a single pass:
- Immediately state this limitation before beginning.
- List which modules will be covered in this pass (prioritise critical path: auth, payments, core data).
- Specify exactly what to provide for the continuation pass: `"Next: provide [module names] to continue the audit."`
- Never silently truncate your analysis — a partial audit presented as complete is worse than no audit.

---

### Step 1: Orient

- Read the entry point. Understand the declared scope.
- **Initialise the Audited Registry** — an empty list that grows as each element is fully audited. Format: `[AE-XXX] ElementName — VERDICT`. Check this registry before drilling into any element.
- Map the top-level elements (list them before diving in).
- Identify the critical path (user-facing, revenue-critical, auth-critical).
- Audit critical path first.

### Step 2: First Pass (Lenses 1 & 2)

- Apply Lenses 1 and 2 to all elements.
- Apply dead code short-circuit where triggered (see Lens 1).
- Flag broken implementations immediately.
- Do not go deep yet — build the full map first.
- Load `references/audit-checklists.md` and identify which checklist applies to each element type.

### Step 3: Deep Pass (Lenses 3–6, 9–10 per element)

- Apply Lenses 3, 4, 5, 6, 9, and 10 to each element in critical-path order.
- **On Lenses 7 and 8 during Step 3:** You may log *per-element* wiring and duplication findings inline as you encounter them. The rule: if the finding is visible entirely within the code of the element you are currently reading, log it now. If confirming the finding requires reading a *second* element and comparing the two, that is Step 4. Examples — valid in Step 3: "this component accepts an `onSuccess` prop but never calls it" (visible in this component alone); "this element expects `user_id` but the API contract above says `userId`" (visible from the element + its immediate caller already in context). Belongs in Step 4: "this validation logic appears to be duplicated from another file" (requires reading that other file to confirm).
- Before each element, check the Audited Registry — cite and skip if already audited.
- Recurse into sub-paths as discovered; add each audited element to the registry **as PARTIAL** immediately after completing its L3–6, 9–10 pass.
- For any finding, cross-reference `references/common-findings.md` for a matching pattern and fix template.
- Mark each drilled sub-path as CLEAN or log the finding with ID. Format for CLEAN sub-paths: `→ [ElementA] calls [ElementB] → [ElementB]: ✅ STATIC-CONFIRMED`

### Step 4: Cross-Element Pass (Lenses 7 & 8 system-wide)

**This is a different mode of analysis** — not repeating per-element checks but scanning the whole system at once for patterns that only emerge when comparing elements against each other.

- **Lens 7 cross-element:** Look for disconnected islands — elements that exist but produce no output and receive no input from any other element; services instantiated but injected nowhere; routes registered but linked to from nowhere; config values set but never read by anything. Also compare contracts: does the shape one element *sends* match what the *consuming* element expects, across all call sites?
- **Lens 8 cross-element:** Compare all elements against each other for duplicated logic, duplicate function names across files, duplicate API routes, constants defined in multiple files with different values, conflicting RBAC rules for the same action.
- Identify disconnected islands, contract mismatches, and duplicated logic that only become visible with the full picture.

### Step 5: Finalise and Report

**First — finalise per-element output blocks** (written during Step 3 with L7/L8 pending):
- Go back to each element's output block.
- Fill in L7 Interconnect and L8 Duplicates rows with results from Step 4.
- Add any new L7/L8 findings to that element's FINDINGS DETAIL section.
- Upgrade elements with all lenses now clean to the compact clean format.
- Upgrade elements in the Audited Registry from PARTIAL to COMPLETE.

**Then — produce the Audit Summary Report** (see template above).

**Finally — produce the recommended fix order** by phase.

---

## Focused Audit Shortcuts

These are abbreviated single-concern audits for when a user asks one specific question ("does this fetch work?", "what states is this missing?") rather than a full 10-lens pass. They are **not** replacements for the full lenses — they are entry points when scope is narrow.

### Shortcut: "Does this fetch actually fetch?"

Apply Lens 6 only. Walk this specific checklist:
1. Is the fetch inside a function? Is that function called anywhere?
2. Is there a condition blocking it? (`if (enabled)`, role check, feature flag)
3. Is it tied to a lifecycle hook or trigger condition that may never fire? (React: `useEffect` with stale or missing deps; backend: an event listener registered on an event that never fires; scheduled job: trigger condition never met in the current environment)
4. Is the result awaited or fire-and-forget?
5. Is the destination correct and environment-configurable? (HTTP: URL; queue: topic/queue name; gRPC: service address — whichever applies)
6. Are credentials/auth attached and current? (HTTP: auth headers; queues: IAM role or connection credentials; gRPC: channel credentials)
7. Is the error/failure signal from the transport layer checked? (HTTP: response status before parsing body — see `FETCH-03`; queues: nack/dead-letter handling; gRPC: error code; WebSocket: close event)
8. Is the response actually consumed by the caller — stored, returned, passed downstream, or rendered? Or is it silently discarded?

### Shortcut: "What states is this missing?"

Apply Lens 3 only. Check `references/common-findings.md` STATE-01 (missing error state), STATE-02 (missing empty state), STATE-03 (in-flight indicator never resets). Enumerate all state combinations — adapt the names to the element's language/paradigm:

- **In-flight + has prior data** — what is shown while refreshing? (UI: spinner over stale data? backend: "processing" status while old result still visible?)
- **Error + has prior data** — is the error surfaced? Is the old data still shown?
- **Empty result + not loading** — is an explicit empty/no-results state rendered or returned?
- **In-flight + previous error** — is the old error cleared when a retry starts?

For frontend: check `isLoading`, `isError`, `isEmpty` boolean combinations. For backend/jobs: check status enums (`PENDING`, `PROCESSING`, `FAILED`, `COMPLETE`) — are all transitions defined? For state machines: map every state node and every transition edge.

### Shortcut: "Is this configurable by the owner?"

Apply Lens 5 only. First load `references/common-findings.md` — check `CONFIG-01` (hardcoded value blocks owner control) and `CONFIG-02` (admin panel writes to wrong field — owner control is an illusion). Then for each business-logic value:
1. Is it a literal in code? (string, number, boolean)
2. Would a platform admin want to change it without a deploy?
3. If yes: is there a config table, env var, or admin panel? Is it wired?
4. If admin panel exists: trace the full roundtrip — does write go to the right field? Does the application read from that same field? → `CONFIG-02` if broken.
5. If no mechanism exists: log `CONFIG-01`.

### Shortcut: "What depends on this?"

Apply Lens 4 (downstream only). Reverse-reference search:
1. All imports of this module
2. All usages of this component name
3. All calls to this function
4. All foreign keys referencing this table
5. Build downstream impact list — what breaks if this is removed or changed?

### Shortcut: "Where is the error being swallowed?"

Apply Lens 9 only. Check `references/common-findings.md` ERROR-01 (silent catch) first. Then:
1. Search for all `catch` / `rescue` / `except` blocks — is the body empty, or does it only log without rethrowing or returning an error value?
2. Search for all `.catch()` / `.catchError()` promise handlers — same check.
3. Is there a global error boundary or handler? Is it actually connected?
4. For every error found: is it surfaced to the appropriate consumer (user, caller, or monitoring system)? Logged with enough context? Does it leave application state corrupted?

---

## Reference Files — When to Load Each

| File | Load When | Contains |
|------|-----------|----------|
| `references/audit-checklists.md` | Step 2 — identify which checklist matches each element type. Re-load on demand during Step 3 when entering a new element type. | API Endpoint, React Component, Database Table, Service Class, Cron Job, Webhook Handler, Auth/Permission, Config/Settings, .env File, Docker, CI/CD Pipeline, GraphQL Schema, IaC |
| `references/common-findings.md` | Step 3 — before writing any finding's fix recommendation, check for a matching pattern and use its fix template. Also referenced inline from Lenses 2–10. | 18 named finding patterns (FETCH-01–03, STATE-01–03, CONFIG-01–02, WIRE-01–02, DEPGRAPH-01–02, DUPE-01–02, SEC-01–02, ERROR-01, DEAD-01) |
| `references/system-audit-phasing.md` | Immediately when scope is System Audit — before any analysis begins. | Phase 0 (system map) + 7 numbered audit phases covering foundation, services, API, frontend, integrations, cross-cutting concerns, and infrastructure |

Do not defer loading these files until after analysis — they shape the analysis itself.
