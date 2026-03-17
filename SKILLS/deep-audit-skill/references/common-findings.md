# Common Critical Findings — Pattern Library

Each pattern includes: symptom, diagnosis, and fix template.

---

## FETCH-01: Fetch Defined But Never Called

**Symptom:** A function containing a fetch/HTTP call exists but is never invoked.
**Diagnosis:** Search for all call sites of the function. If zero results → dead fetch.
**Finding Level:** 🟠 HIGH (feature silently non-functional)
**Fix Template:**
> Identify where this fetch should be triggered and wire it there. Common triggers: on element/component initialisation, on a user action (click, submit), on a schedule, or on receipt of an upstream event. After wiring, verify the response is consumed correctly by the caller — not just executed but its result actually used.

---

## FETCH-02: Response Shape Mismatch

**Symptom:** Code accesses `response.data.items` but the API returns `response.results`.
**Diagnosis:** Compare the actual API response shape against what the code destructures/accesses.
**Finding Level:** 🔴 CRITICAL (runtime crash or silent undefined)
**Fix Template:**
> Align the consumer to the actual API contract, or update the API to match the expected contract. Add a typed interface/schema for the response.

---

## FETCH-03: HTTP Error Not Caught

**Symptom:** An HTTP call is made but the response status is never checked — only the body is parsed, regardless of whether the request succeeded.
**Diagnosis:** HTTP errors (400, 403, 500) do not throw exceptions by default in most HTTP clients. The response body will be parsed as if it were a success response, silently corrupting application state.
**Finding Level:** 🔴 CRITICAL (errors silently ignored — the caller receives and processes an error body as if it were a success response, silently corrupting state)
**Fix Template:**
> Always check the HTTP response status *before* parsing the body. If the status indicates failure, extract any error message from the body and throw/return an error — do not proceed as if the request succeeded.
>
> JavaScript (`fetch` API) example:
```javascript
if (!res.ok) {
  const error = await res.json().catch(() => ({}));
  throw new Error(error.message || `HTTP ${res.status}`);
}
```
> The principle is the same in all languages: status check → error branch → don't parse as success.

---

## STATE-01: Missing Error State

**Symptom:** An element has a success/data path but no explicit error path — failures are silently ignored or produce ambiguous output. Common forms: (a) UI component shows data or spinner but has no error UI; (b) REST endpoint returns 200 for all outcomes including failures; (c) job/worker completes with exit code 0 even when processing failed; (d) state machine has no FAILED/ERROR node.
**Diagnosis:** Identify all paths that can result in failure. Check whether each failure path produces a distinct, observable error state. For UI: check render logic for `isError`/`error` handling. For APIs: check that error responses use appropriate status codes (4xx/5xx). For jobs: check exit codes and logging. For state machines: verify a FAILED state exists and is reachable.
**Finding Level:** 🟠 HIGH (failures are invisible to the system or the user — debugging is impossible, retries cannot be triggered)
**Fix Template:**
> Add an explicit error state for every failure mode. For UI: render a meaningful error message with a retry action. For APIs: return appropriate HTTP error codes with structured error bodies. For jobs: log the failure, set a failed status, and optionally raise an alert. For state machines: add a FAILED/ERROR node with defined transitions into and out of it.

---

## STATE-02: Missing Empty State

**Symptom:** An element receives a valid but empty result and produces no distinct response for that case. Common forms: (a) UI list/table renders blank space when data array is empty; (b) API returns 200 with empty array when the resource doesn't exist (should be 404 or explicit empty body); (c) job logs "Processed 0 records" with no alert or distinct outcome; (d) search returns no results with no user-facing "nothing found" signal.
**Diagnosis:** Check whether the element explicitly handles the case where the result set is empty/zero/null. For UI: check for `if (items.length === 0)` render branch. For APIs: check status codes and response body when no records match. For jobs: check whether 0-record runs are logged distinctly and whether an alert threshold exists.
**Finding Level:** 🟡 MEDIUM (ambiguous — the caller cannot distinguish "no data" from "not loaded yet" or "error")
**Fix Template:**
> Add an explicit empty state for the zero-result case. For UI: render an empty state component with a message and call to action. For APIs: return 404 when a resource doesn't exist, or an explicit `{ "items": [], "total": 0 }` for empty collections. For jobs: log a distinct "no records to process" outcome and consider whether an alert is appropriate.

---

## STATE-03: Loading Never Resolved

**Symptom:** An in-flight/processing indicator is set to true but never reset after the operation completes or fails — it stays true indefinitely. Common forms: UI loading spinner stuck forever; a job status permanently stuck at "processing"; a queue message stuck "in-flight"; a transaction stuck at "pending"; a progress bar that never reaches 100%.
**Diagnosis:** Check that the in-flight indicator is reset to false/idle in both the success AND error/failure path. If it is only reset on success, any failure will leave the state stuck permanently.
**Finding Level:** 🔴 CRITICAL (the operation appears in-progress indefinitely — the caller, user, or downstream system has no recourse and cannot trigger a retry)
**Fix Template:**
> Always reset the in-flight indicator in both the success path and the error/failure path — it must never remain active after the operation settles.
>
> Languages with `finally`: put the reset there — `finally { isLoading = false; }`
>
> Languages without `finally` (e.g. Go) or patterns that don't use it: reset explicitly in both branches:
> ```
> isLoading = true
> result, err = doOperation()
> isLoading = false        // reset in success branch
> if err != nil {
>   isLoading = false      // reset in error branch too
>   handleError(err)
>   return
> }
> ```
> The principle applies universally: the in-flight state must reach a terminal value (idle/complete/failed) after every possible exit path from the operation.

---

## CONFIG-01: Hardcoded Business Value

**Symptom:** `const FREE_PLAN_LIMIT = 10` as a constant in code.
**Diagnosis:** Is this a value a platform operator might want to change? If yes → hardcoded.
**Finding Level:** 🟠 HIGH (requires code deploy to change, blocks owner control)
**Fix Template:**
> Move to a config table, environment variable, or admin-configurable setting. Seed a default value. Wire the admin panel to read/write the correct field.

---

## CONFIG-02: Admin Panel Writes to Wrong Field

**Symptom:** Admin sets "max seats" to 50, but the application still enforces the old value.
**Diagnosis:** Trace the admin panel form submission → API call → DB write → application read. Find where the chain breaks.
**Finding Level:** 🔴 CRITICAL (admin control is an illusion, system behaves unpredictably)
**Fix Template:**
> Verify the full config roundtrip: write path and read path use the same field name and table. Clear any cache on write.

---

## WIRE-01: Element Exists But Is Disconnected From Its Data Source or Consumer

**Symptom:** An element is instantiated or registered but receives no data, makes no calls, and produces no output — it exists but is effectively a shell. Common forms: (a) UI component renders markup but is never wired to its data source (no props, no store subscription, no fetch); (b) service class is instantiated but never injected into the components/routes that need it; (c) event handler is registered but the event it listens for is never emitted; (d) cron job is defined but not scheduled; (e) IaC resource is declared but no service is configured to use it.
**Diagnosis:** Trace from the element to its data source and to its consumers. For UI: check if props are passed, if the store is subscribed, if fetch is invoked. For services: check injection points — is this service used anywhere? For event handlers: check whether the triggering event is ever emitted. For jobs: check scheduler registration.
**Finding Level:** 🟠 HIGH (feature appears present but is a shell — produces no output, delivers no value)
**Fix Template:**
> Trace the full wiring path: data source → element → consumer. Identify the broken link and connect it. For UI: pass props, connect to store, or invoke fetch. For services: inject at the correct dependency injection point. For events: verify the emitter and listener are on the same channel. For jobs: register the schedule.

---

## WIRE-02: Callback / Handler Defined But Never Invoked

**Symptom:** A callback, handler, or hook is passed to or registered within an element, but the element never calls it — the caller's response to the operation is silently dropped. Common forms: (a) UI parent passes `onSuccess={handleSuccess}` to child; child never calls `props.onSuccess()`; (b) service receives an `onComplete` callback parameter but never invokes it; (c) event listener is registered but the host never emits that event; (d) middleware is registered but its `next()` is never called, leaving the request hanging.
**Diagnosis:** Search for all usages of the callback/handler within the receiving element. If it is accepted as a parameter/prop but never invoked in any code path, that is the failure.
**Finding Level:** 🟠 HIGH (the caller's state is never updated after the operation — the flow silently breaks at this handoff point)
**Fix Template:**
> Invoke the callback at the correct point in the element's execution — after the operation succeeds, after an error is handled, or at the end of the middleware chain. Verify all code paths (success, error, early exit) invoke it appropriately.

---

## DEPGRAPH-01: Missing Dependency

**Symptom:** An element depends on something that does not exist at the expected path, name, or injection point. Common forms: (a) module imports a file that doesn't exist; (b) service constructor expects an injected dependency that is never registered; (c) function calls another function that has been removed or renamed; (d) IaC module references an output from another module that doesn't exist.
**Diagnosis:** Resolve the import/reference path. Check if the file, module, class, or registration exists. Check for typos, renamed exports, or missing registrations in the DI container.
**Finding Level:** 🔴 CRITICAL (application will not compile or will crash at startup/runtime when the missing dependency is first accessed)
**Fix Template:**
> Create the missing module or dependency, correct the import/reference path, register the missing dependency in the DI container, or rename to match the expected identifier.

---

## DEPGRAPH-02: Circular Import / Circular Dependency

**Symptom:** Module A imports Module B which imports Module A. Or: Service A's constructor requires Service B, and Service B's constructor requires Service A.
**Diagnosis:** For circular imports — trace import chains; look for "Cannot access before initialization" or "undefined is not a constructor" runtime errors. For circular DI — look for constructor injection cycles in your DI container's error output, or trace constructor signatures manually.
**Finding Level:** 🔴 CRITICAL (undefined behaviour at module load time; DI circular deps cause container failures or infinite recursion)
**Fix Template:**
> **Circular imports:** Extract the shared dependency into a third module C. A and B both import C. Neither imports the other.
>
> **Circular DI:** Use lazy injection (inject a factory or provider rather than the instance directly), extract the shared logic into a third service that neither A nor B depends back on, or refactor so the dependency flows in one direction only.

---

## DUPE-01: Logic Implemented Twice

**Symptom:** Two functions in different files both calculate the same value (e.g., user's display name).
**Diagnosis:** Search for similar function names and similar logic patterns.
**Finding Level:** 🟡 MEDIUM (divergence risk — they will eventually produce different results)
**Fix Template:**
> Extract into a single shared utility. Update both call sites.

---

## DUPE-02: Two API Routes for the Same Resource Action

**Symptom:** `POST /api/v1/users` and `POST /api/users` both exist and both create users.
**Diagnosis:** List all registered routes and look for semantic duplicates.
**Finding Level:** 🟠 HIGH (confusion, inconsistent behaviour, maintenance burden)
**Fix Template:**
> Deprecate one route. Redirect or consolidate. Document the canonical route.

---

## SEC-01: Permission Check Missing at Enforcement Layer

**Symptom:** An operation that should be gated by authentication or authorisation is only protected at the presentation or routing layer — not at the layer where the operation is actually executed. Common forms: (a) a "Delete" button is hidden in the UI, but the underlying API endpoint has no server-side auth check; (b) a GraphQL resolver is blocked at the schema level but not validated within the resolver itself; (c) an internal microservice trusts a caller's identity claim without re-verifying it; (d) a background job skips permission checks because "it's an internal process."
**Diagnosis:** Identify where the operation executes. Check that an authorisation check exists at that layer — not just above it. For REST endpoints: test the endpoint directly as an unauthorised user. For internal services: check whether callers can be impersonated. For resolvers: check whether the auth check is inside the resolver function, not only in middleware.
**Finding Level:** 🔴 CRITICAL (security vulnerability — any caller who bypasses the presentation layer can perform the operation without restriction)
**Fix Template:**
> Add authorisation at the execution layer — never rely on a layer above (UI, middleware, schema) as the sole gatekeeper. For REST: add auth middleware directly to the route. For GraphQL: verify inside the resolver. For services: validate the caller's identity and permissions at the service boundary. For jobs: check whether tenant/user context is correctly scoped before execution.

---

## SEC-02: Missing Tenant Isolation

**Symptom:** `SELECT * FROM orders WHERE id = :id` — no tenant/user filter.
**Diagnosis:** A user can retrieve any record by guessing IDs (IDOR — Insecure Direct Object Reference).
**Finding Level:** 🔴 CRITICAL (IDOR — any authenticated user can access any other user's or tenant's records by guessing IDs)
**Fix Template:**
> Add `AND tenant_id = :current_tenant_id` to all queries. Verify no query retrieves cross-tenant data.

---

## ERROR-01: Silent Catch

**Symptom:** An error is caught but nothing is done with it — the catch block is empty, or only logs without rethrowing, causing failures to disappear silently.
**Diagnosis:** Search for all catch/rescue/except blocks. Identify any where: the body is empty, the error is only logged but not rethrown or returned, or the error is suppressed with a placeholder value.
**Finding Level:** 🔴 CRITICAL (errors disappear — debugging becomes impossible, the failure is invisible to callers and monitoring systems, recovery cannot be triggered)
**Fix Template:**
> At minimum, a catch block must do one of: (a) rethrow the error so a higher-level handler can act on it, (b) convert it to a meaningful error response/return value, or (c) log it AND surface the failure to the appropriate consumer (user, caller, or monitoring system). An empty catch block or a log-only catch that swallows the error is always wrong.
>
> JavaScript/Java example: `catch (e) { logger.error(e); throw e; }`
> Python example: `except Exception as e: logger.error(e); raise`
> Go example: `if err != nil { log.Error(err); return err }`
>
> The principle is universal: catch, log with context, then propagate or surface — never swallow.

---

## DEAD-01: Unreachable Code Path

**Symptom:** A function, component, or module is defined but never reaches execution. Common forms: (a) defined but never imported anywhere, (b) imported but the imported reference is never called or rendered, (c) a route registered in the router but nothing ever navigates to it — no UI link, no redirect, no internal service call.
**Diagnosis:** Search for all import sites of the module. If zero → never imported. If imported, search for all usages of the imported name. If zero usages → imported but unused. Check route links for registered-but-unreachable routes.
**Finding Level:** 🔵 LOW to 🟡 MEDIUM depending on context (🟡 if the feature was intended to be live; 🔵 if it is known cleanup)
**Fix Template:**
> Either wire the element to where it should be used (import it, render it, link to it), or remove it entirely if it is genuinely obsolete. Leaving dead code in place creates confusion about system scope and grows maintenance burden.
