---
name: api-design
description: Use when designing, reviewing, normalizing, or documenting REST APIs; when defining endpoint contracts, resource naming, status codes, pagination, filtering, sorting, search, error responses, versioning, or rate limiting; or when a governed package reaches backend contract and public/internal API design work.
---

# API Design

Conventions and best practices for designing consistent, developer-friendly REST APIs.

This skill focuses on API contract shape and consistency.

Use it alongside:

- `foundation-module-factory` when a governed package reaches backend/API contract design
- `project-mapper` when a documented or implemented project needs API mapping or normalization
- `deep-audit` when endpoint or API-contract review needs a contract-design lens
- `secure-coding-practices` when auth, authorization, abuse resistance, or sensitive access patterns are in scope
- `foundation-extension-auditor` when a module may be redefining foundation-owned API behavior

## Read First

Before doing substantive work for a foundation-based project, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. the governed package, if one already exists
3. `../../foundations/README.md` when the project builds on the foundations
4. `../../foundations/FOUNDATION_MODULE_RULES.md` when the project builds on the foundations
5. `../../foundations/FOUNDATION_IMPLEMENTATION_BLUEPRINT.md` when the task affects actual API implementation layout
6. [`references/rest-api-patterns.md`](references/rest-api-patterns.md)

## When To Activate

Activate this skill when the task involves any of the following:

- designing new API endpoints
- reviewing existing API contracts
- defining pagination, filtering, sorting, or search conventions
- defining API error response shape
- choosing status codes and method semantics
- planning API versioning
- defining rate-limit contract behavior
- preparing public, partner, or internal API contracts inside a governed package

## Core Role

This skill should:

- keep REST resources and endpoint contracts consistent
- make response and error formats predictable
- make list endpoints deliberate about pagination and filtering
- make status code usage semantic instead of flat `200` responses everywhere
- record versioning and rate-limit decisions explicitly

This skill should not:

- redefine foundation ownership
- replace security review
- override governed package decisions without updating the package
- silently invent platform-wide API conventions if the foundation already owns them

## Foundation-Aware Rule

If the API belongs to a module built on the foundations:

- consume foundation-owned auth, session, RBAC, audit, connector, and shared-contract behavior instead of redefining it
- use this skill to shape module endpoint contracts cleanly
- if the module proposes a reusable shared API contract change, surface it through the governed package and ownership audit instead of quietly normalizing it into the foundations

## Operating Rules

1. Prefer plural noun resource paths and consistent URL structure.
2. Use HTTP methods semantically.
3. Use status codes semantically.
4. Keep success and error response shapes consistent within the project/package.
5. Validate input boundaries explicitly.
6. Use pagination for list endpoints and document which pagination mode is chosen.
7. Make filtering, sorting, and search rules explicit instead of ad hoc.
8. Record authentication and authorization expectations per endpoint.
9. Define versioning strategy deliberately for public or partner-facing APIs.
10. Define rate-limit expectations for exposed APIs.
11. When the package already defines API patterns, follow the package rather than introducing a parallel contract style.

## Output Expectations

When this skill is used, it should leave behind one or more of the following:

- normalized endpoint contract guidance
- backend contract notes for governed packages
- API review findings tied to specific contract issues
- explicit decisions for response shapes, error formats, pagination, filtering, versioning, or rate limits

## Reference Guide

Use [`references/rest-api-patterns.md`](references/rest-api-patterns.md) as the detailed rulebook for:

- resource naming and URL design
- method and status code semantics
- response envelopes and error shape
- pagination
- filtering, sorting, and search
- auth and authorization patterns
- rate limiting
- versioning

Load only the sections needed for the current task.
