# System Audit Phasing Guide

For large systems, a full audit must be phased to remain manageable and produce actionable output. Do not attempt to audit everything simultaneously.

---

## Phasing Strategy

### Phase Ordering Principles

1. **Critical path first.** Always audit authentication, authorization, and core data models before peripheral features.
2. **Follow the user journey.** Audit flows in the order a user would experience them (registration → onboarding → core action → payment → offboarding).
3. **Bottom-up dependencies.** Audit the database schema and core services before the API layer. Audit the API layer before the frontend.

---

## Standard Phase Sequence

### Phase 0 — System Map (Before Drilling)
- List all modules/features
- List all database tables
- List all API endpoint groups
- List all external integrations
- List all background jobs and webhooks
- List all infrastructure components present: Dockerfile(s), CI/CD pipeline files, .env files, IaC configs (Terraform/K8s/CloudFormation), GraphQL schema (if applicable)
- Mark which are critical path vs. secondary
- Identify the highest-risk areas (auth, payments, data access, secrets handling)

Output: a flat list. No drilling yet. Every element in this list is a candidate for the Audited Registry.

---

### Phase 1 — Foundation Audit
Scope: database schema + core data models + auth system

1. Database schema audit (all tables)
2. Auth system: registration, login, token lifecycle, role assignment
3. Permission/RBAC model: is it DB-driven or hardcoded? Is it enforced server-side?
4. Core data models: the 3-5 tables that everything else references

Critical findings here block all subsequent phases.

---

### Phase 2 — Service Layer Audit
Scope: all service classes / modules / domain logic

1. Does each service exist and is it instantiated?
2. Does each service method do what its name claims?
3. Are services tested in isolation?
4. Are services communicating correctly with the database?
5. Are inter-service contracts consistent?

---

### Phase 3 — API Layer Audit
Scope: all API routes and their request/response contracts

1. Route registration (all routes exist and are reachable)
2. Auth middleware applied to all protected routes
3. Request validation on all POST/PUT routes
4. Response shapes match what clients expect
5. Error responses are consistent and structured
6. Rate limiting applied

---

### Phase 4 — Frontend / UI Audit
Scope: all pages, components, and data-fetching

1. All pages route correctly
2. All data-fetching components have loading/error/empty states
3. Form submissions handled and validated
4. No hardcoded values that should come from API or config
5. No components rendering without data (disconnected shells)

---

### Phase 5 — Integration & Background Audit
Scope: webhooks, crons, external API integrations, email, payments

1. External API calls have auth, error handling, and timeouts
2. Webhooks verify signatures and handle duplicates
3. Cron jobs are scheduled, idempotent, and have locks
4. Payment flows have idempotency keys and webhook confirmation

---

### Phase 6 — Cross-Cutting Concerns
Scope: logging, monitoring, config system, error handling, security

1. Structured logging throughout
2. Error tracking connected
3. Config system: all values configurable, no secrets in code
4. Global error boundaries / handlers
5. Input sanitisation
6. Data isolation (multi-tenant checks)

---

### Phase 7 — Infrastructure & DevOps Audit
Scope: .env files, Docker configs, CI/CD pipelines, IaC, GraphQL schema

Use `references/audit-checklists.md` sections: Environment File, Docker, CI/CD Pipeline, GraphQL Schema, Infrastructure as Code.

1. `.env` / environment files — all vars documented, no secrets in source control, no orphaned vars
2. Docker — base image pinned, not running as root, secrets not baked in, health checks defined
3. CI/CD — pipeline triggers correct, secrets injected via CI store, deploy gated on passing tests
4. IaC (Terraform/K8s/CloudFormation) — no hardcoded secrets, resource limits set, state files not committed
5. GraphQL (if applicable) — all resolvers implemented, auth middleware present, N+1 addressed

Note: Infrastructure findings rarely block application functionality directly, but CRITICAL findings here (hardcoded secrets, root containers, deploy without tests) can compromise the entire system regardless of application-layer quality.

---

## Parallel Tracking

Maintain a running findings register across phases:

```
| ID          | Element        | Lens | Severity | Phase Found | Status  |
|-------------|----------------|------|----------|-------------|---------|
| CRIT-001    | authLogin      | L6   | 🔴       | Phase 1     | OPEN    |
| HIGH-002    | UserList       | L3   | 🟠       | Phase 4     | OPEN    |
| CRIT-003    | Dockerfile     | L10  | 🔴       | Phase 7     | OPEN    |
| HIGH-004    | .github/ci.yml | L2   | 🟠       | Phase 7     | OPEN    |
```

At end of each phase, re-prioritize and confirm nothing in previous phases was missed.
