---
name: backend-patterns
description: Use when designing or reviewing backend architecture for Node.js, Express, or Next.js server code; when defining repository, service, controller, middleware, caching, queue, database-access, logging, or operational server patterns; or when a governed package reaches backend implementation planning beyond raw API contract shape.
---

# Backend Patterns

Backend architecture patterns and best practices for scalable server-side applications.

This skill focuses on backend composition and operational structure.

It does not replace:

- `api-design` for REST contract shape
- `secure-coding-practices` for security hardening
- `coding-standards` for general implementation quality

## Read First

Before doing substantive work for a foundation-based project, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. the governed package, if one already exists
3. `../../foundations/README.md` when the project builds on the foundations
4. `../../foundations/FOUNDATION_MODULE_RULES.md` when the project builds on the foundations
5. `../../foundations/FOUNDATION_IMPLEMENTATION_BLUEPRINT.md` when the task affects actual backend implementation layout
6. [`references/server-architecture-patterns.md`](references/server-architecture-patterns.md)

## When To Activate

Activate this skill when the task involves any of the following:

- designing repository, service, controller, or middleware layers
- structuring backend modules and data-access boundaries
- optimizing database queries, batching, or connection use
- adding caching layers or cache invalidation behavior
- designing background jobs, queues, or async processing flows
- defining centralized backend error handling
- structuring logging or backend observability patterns
- turning backend sections of a governed package into implementation architecture

## Core Role

This skill should:

- choose backend patterns proportional to the system's real complexity
- keep business logic, transport logic, and data access clearly separated
- reduce duplication in query, cache, and middleware behavior
- make backend operational concerns explicit rather than scattered
- help packages turn backend contracts into maintainable implementation structure

This skill should not:

- redefine foundation ownership
- redefine API contract shape already owned by `api-design`
- replace security review or permission design
- invent enterprise layering when the system is simple enough for a smaller pattern

## Foundation-Aware Rule

If the backend belongs to a module built on the foundations:

- consume foundation-owned auth, sessions, RBAC, audit, connector, and shared settings behavior instead of rebuilding them
- keep module-specific repositories, services, jobs, caches, and handlers inside the module
- surface reusable backend primitives through the governed package and ownership audit instead of silently promoting them into the foundations

## Operating Rules

1. Separate transport concerns, business logic, and data access when that improves clarity.
2. Do not create repository/service/controller layers by reflex; use them when the complexity justifies them.
3. Prefer explicit query shaping and batching over hidden N+1 behavior.
4. Introduce caching only with a clear invalidation strategy.
5. Keep background jobs idempotent and failure-aware where possible.
6. Centralize operational error handling rather than scattering ad hoc responses.
7. Keep logging structured and useful for backend diagnosis.
8. Follow the governed package and active project structure rather than inventing a parallel architecture.
9. Pair with `api-design` when endpoint contract shape is in scope.
10. Pair with `secure-coding-practices` when auth, permissions, secrets, abuse resistance, or sensitive data are in scope.

## Output Expectations

When this skill is used, it should leave behind one or more of the following:

- backend architecture guidance for the governed package
- implementation notes for repositories, services, controllers, middleware, caches, queues, or logging
- review findings tied to backend structure or operational-pattern issues
- explicit rationale for choosing a simple or layered backend design

## Reference Guide

Use [`references/server-architecture-patterns.md`](references/server-architecture-patterns.md) as the detailed rulebook for:

- repository and service patterns
- middleware composition
- query optimization and batching
- transactions
- caching
- background jobs and queues
- centralized error handling
- logging and backend observability

Load only the sections needed for the current task.
