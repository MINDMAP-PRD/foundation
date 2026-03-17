# Audit Checklists by Element Type

Quick-reference checklists for auditing specific element types. Use alongside the 10 Lenses in SKILL.md.

---

## API Endpoint / Route Handler

- [ ] Route is registered in router
- [ ] Route is reachable (no conflicting middleware blocking it)
- [ ] Authentication middleware applied → if missing, see `common-findings.md` `SEC-01`
- [ ] Authorization/RBAC check present (server-side, not UI-only) → if UI-only, see `common-findings.md` `SEC-01`
- [ ] Request validation: required fields, types, formats
- [ ] Controller calls correct service/function → if missing, see `common-findings.md` `DEPGRAPH-01`
- [ ] Service exists and is implemented (not a stub)
- [ ] Service returns expected shape
- [ ] Response shape matches what the consumer expects (frontend, mobile client, or calling service) → if mismatch, see `common-findings.md` `FETCH-02`
- [ ] HTTP status codes are correct (201 for create, 204 for delete, 400 for bad input, 403 for forbidden)
- [ ] Error responses are structured and consistent → if silent catch found, see `common-findings.md` `ERROR-01`
- [ ] Database query is correct (right table, right fields, right WHERE)
- [ ] Database query is protected against injection
- [ ] Query is scoped to the correct tenant/user → if not, see `common-findings.md` `SEC-02`
- [ ] N+1 query problem checked
- [ ] Response does not expose sensitive fields (passwords, tokens, internal IDs)
- [ ] Rate limiting applied
- [ ] Logging present (request, response, errors)
- [ ] Audit trail written for sensitive operations (update, delete, access)

---

## React / Frontend Component

- [ ] Component renders without crashing when props are minimal/empty
- [ ] Required props are documented and provided by all parents
- [ ] Optional props have sensible defaults
- [ ] Loading state rendered while data is fetching → if missing, see `common-findings.md` `STATE-03`
- [ ] Error state rendered when fetch fails → if missing, see `common-findings.md` `STATE-01`
- [ ] Empty state rendered when data is empty array or null → if missing, see `common-findings.md` `STATE-02`
- [ ] Data state renders correctly with realistic data
- [ ] All user interactions have handlers (`onClick`, `onChange`, `onSubmit`)
- [ ] Form submissions are prevented from default browser behaviour
- [ ] Fetch calls are not triggered on every render (useEffect deps checked) → if unchecked, see `common-findings.md` `FETCH-01`
- [ ] No memory leaks (cleanup on unmount for subscriptions, timers, listeners)
- [ ] Accessibility: labels, roles, keyboard navigation
- [ ] No hardcoded strings that should be i18n or admin-configurable → if present, see `common-findings.md` `CONFIG-01`
- [ ] Component is actually imported and rendered somewhere → if not, see `common-findings.md` `DEAD-01`
- [ ] No console.log left in production code

---

## Database Table / Schema

- [ ] Table exists (migration has been run)
- [ ] All columns referenced in code exist in schema
- [ ] Column types match what code expects (string vs integer vs boolean vs JSON)
- [ ] Required columns have NOT NULL constraints
- [ ] Foreign keys are defined and reference correct tables → if missing dependency, see `common-findings.md` `DEPGRAPH-01`
- [ ] Foreign key ON DELETE behaviour is correct (CASCADE vs RESTRICT vs SET NULL)
- [ ] Unique constraints exist where business logic requires uniqueness
- [ ] Indices exist on all foreign key columns
- [ ] Indices exist on columns used in frequent WHERE clauses
- [ ] Soft-delete pattern: `deleted_at` used consistently if implemented
- [ ] Multi-tenant isolation: `tenant_id` or `org_id` present and indexed → if missing, see `common-findings.md` `SEC-02`
- [ ] Timestamp columns: `created_at`, `updated_at` — auto-managed?
- [ ] Seed data: does required config/lookup data exist in the table? → if missing, see `common-findings.md` `CONFIG-01`
- [ ] No orphaned records: FK integrity maintained
- [ ] No columns storing comma-separated values (should be junction table)

---

## Service Class / Module

- [ ] Class is instantiated / injected somewhere (not just defined) → if not, see `common-findings.md` `WIRE-01`
- [ ] Constructor dependencies are provided (not missing injections) → if missing, see `common-findings.md` `DEPGRAPH-01`
- [ ] All public methods are called from at least one place → if not, see `common-findings.md` `DEAD-01`
- [ ] Private methods are only used internally
- [ ] State managed within the service is thread-safe / not shared incorrectly
- [ ] External API calls have timeouts
- [ ] External API calls handle 4xx and 5xx separately → if not, see `common-findings.md` `FETCH-03`
- [ ] Retry logic present for transient failures
- [ ] Service does not hold business logic that belongs in the domain model
- [ ] Service does not do too many things (single responsibility)
- [ ] Interfaces/contracts are versioned if consumed externally → if duplicated contract, see `common-findings.md` `DUPE-02`

---

## Cron Job / Background Task

- [ ] Job is registered/scheduled (not just defined) → if not, see `common-findings.md` `DEAD-01`
- [ ] Schedule is correct (cron expression verified)
- [ ] Job has a lock mechanism to prevent concurrent runs
- [ ] Job is idempotent (safe to run twice with same effect)
- [ ] Job has a timeout
- [ ] Job logs start, end, and any errors → if silent on failure, see `common-findings.md` `ERROR-01`
- [ ] Job handles partial failures (doesn't silently skip failed records)
- [ ] Job respects multi-tenancy (processes all tenants or a specific one) → if unscoped, see `common-findings.md` `SEC-02`
- [ ] Job has an alerting/monitoring hook
- [ ] Dead-letter / retry mechanism for failed items

---

## Webhook Handler

- [ ] Webhook endpoint is publicly accessible
- [ ] Signature verification implemented (HMAC check) → if missing, this is 🔴 CRITICAL — unauthenticated callers can send arbitrary events
- [ ] Idempotency: duplicate webhook deliveries handled (event ID stored)
- [ ] Handler responds 200 quickly, processes async
- [ ] Processing failures do not return 500 to the sender (causes resend loops) → if not, see `common-findings.md` `ERROR-01`
- [ ] Handler is scoped to the correct event types (ignores unknown events gracefully)
- [ ] Sensitive payload data is not logged

---

## Auth / Permission Check

- [ ] Auth check happens server-side (not just UI-gated) → if UI-only, see `common-findings.md` `SEC-01`
- [ ] Token validation: signature, expiry, issuer
- [ ] Refresh token flow implemented
- [ ] Session revocation supported
- [ ] Permission check uses the correct resource ID (not a generic check) → if generic, see `common-findings.md` `SEC-02`
- [ ] Ownership check: does the requesting user own this resource?
- [ ] Super-admin bypass is explicit and logged
- [ ] Failed auth attempts are logged and rate-limited

---

## Config / Settings System

- [ ] All config keys are documented
- [ ] Default values are sensible and safe
- [ ] Required config causes application to fail fast on startup if missing
- [ ] Config is not logged in plaintext (secrets redacted)
- [ ] Per-tenant config is isolated → if not, see `common-findings.md` `SEC-02`
- [ ] Config changes take effect without requiring a full restart (where appropriate)
- [ ] Admin panel changes write to the correct config store → if broken roundtrip, see `common-findings.md` `CONFIG-02`
- [ ] Config reads are cached appropriately (not hitting DB on every request)
- [ ] Hardcoded values that should be configurable are flagged → see `common-findings.md` `CONFIG-01`

---

## Environment File / .env

- [ ] All required env vars are documented (e.g. `.env.example` present and current) → if missing docs, see `common-findings.md` `CONFIG-01`
- [ ] No secrets committed to source control → if present, this is 🔴 CRITICAL — secrets in git history are permanently exposed even after deletion; rotate all exposed secrets immediately and use secret scanning
- [ ] Each env var is actually read by the application — no orphaned vars → if orphaned, see `common-findings.md` `DEAD-01`
- [ ] Each var read by the application is present in the env file — no missing vars causing silent undefined → if missing, see `common-findings.md` `DEPGRAPH-01`
- [ ] Prod / staging / dev values are segregated (no prod secrets in dev `.env`)
- [ ] Sensitive vars are excluded from logging at startup
- [ ] Default fallback values for optional vars are safe (not `password: ''`)

---

## Docker / Container Config

- [ ] Base image is pinned to a specific version (not `:latest`) → if not, see `common-findings.md` `CONFIG-01` (uncontrolled dependency version is a hardcoded assumption)
- [ ] Image is not running as root → if it is, this is 🔴 CRITICAL — process escalation risk; add `USER nonroot` to Dockerfile
- [ ] Secrets are not baked into the image (use runtime env injection) → if present, this is 🔴 CRITICAL — secrets persist in image layers and image history; move to runtime env vars or secret manager
- [ ] `.dockerignore` excludes `.env`, `node_modules`, secrets
- [ ] Multi-stage build used if applicable (build image ≠ runtime image)
- [ ] Health check defined → if missing, this is 🟠 HIGH — orchestrators (Docker Compose, Kubernetes) cannot detect or restart an unresponsive container without it
- [ ] Port exposure matches what the application actually listens on → if mismatch, this is 🔴 CRITICAL — container is unreachable; verify `EXPOSE` matches `PORT` env var and app binding
- [ ] Volumes for persistent data are declared

---

## CI/CD Pipeline (GitHub Actions / Jenkins / etc.)

- [ ] Pipeline is actually triggered (push/PR/tag — correct branch filters) → if never triggers, see `common-findings.md` `DEAD-01`
- [ ] Secrets are injected via CI secret store — not hardcoded in YAML → if hardcoded, this is 🔴 CRITICAL — secrets in pipeline YAML are visible to anyone with repo access and persist in git history; move to the CI provider's secret store immediately
- [ ] Build step succeeds before deploy step (deploy is gated on build) → if not gated, see `common-findings.md` `STATE-03` (no failure state)
- [ ] Tests run before deploy → if tests skipped, treat as `HIGH` — unverified functionality reaching production
- [ ] Failed pipeline does not deploy (exit-on-error / `set -e` present) → if missing, see `common-findings.md` `ERROR-01`
- [ ] Deploy targets are correct per branch (main → prod, develop → staging) → if wrong, this is 🔴 CRITICAL — production deployment to wrong environment or vice versa; audit branch filter expressions explicitly
- [ ] Rollback mechanism exists
- [ ] Pipeline notifications configured (Slack, email on failure)

---

## GraphQL Schema

- [ ] All types are used (no orphaned type definitions) → if orphaned, see `common-findings.md` `DEAD-01`
- [ ] All resolvers are implemented — no stub resolvers returning null → if stubs, see `common-findings.md` `WIRE-01`
- [ ] Resolver is protected by auth middleware where data is sensitive → if missing, see `common-findings.md` `SEC-01`
- [ ] N+1 query problem addressed (DataLoader or batching used for nested resolvers) → if missing, treat as `HIGH`
- [ ] Input types have validation → if missing, this is 🟠 HIGH — unvalidated inputs enable injection attacks and corrupt data; add field-level validation rules to all mutation/query input types
- [ ] Mutations return the updated resource (not just a success boolean) → if not, see `common-findings.md` `FETCH-02`
- [ ] Subscriptions have cleanup on client disconnect
- [ ] Schema is not introspectable in production (or introspection is intentional) → if exposed, treat as `HIGH`

---

## Infrastructure as Code (Terraform / Kubernetes YAML / CloudFormation)

- [ ] Resources are named consistently and descriptively
- [ ] Secrets are not hardcoded in IaC files (use secret manager references) → if hardcoded, this is 🔴 CRITICAL — IaC files are typically committed to source control; use Vault, AWS Secrets Manager, or equivalent; reference secrets by ARN/path, never by value
- [ ] Resource limits (CPU/memory) defined for all containers → if missing, treat as `HIGH`
- [ ] Replicas / scaling config is environment-appropriate (not `replicas: 1` in prod) → if hardcoded, see `common-findings.md` `CONFIG-01`
- [ ] Network policies restrict inter-service access to what is necessary → if absent, this is 🟠 HIGH — without network policies all services can reach all other services by default; any compromised pod can communicate laterally across the cluster; define explicit ingress/egress rules per service
- [ ] Persistent volumes have correct storage class and access mode
- [ ] Rollout strategy defined (RollingUpdate with sane maxUnavailable)
- [ ] Health/readiness probes configured on all services → if missing, see `common-findings.md` `WIRE-01`
- [ ] State files / plan outputs are not committed to source control → if committed, this is 🔴 CRITICAL — Terraform state and plan files contain resource IDs, IP addresses, and potentially plaintext secrets; add to `.gitignore` and use remote state backends (S3, Terraform Cloud)
