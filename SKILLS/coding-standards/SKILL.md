---
name: coding-standards
description: Use when building, reviewing, refactoring, or scaffolding TypeScript, JavaScript, React, or Node.js code; when translating specs into implementation; when setting naming, structure, typing, validation, or testing conventions; or when a governed package needs implementation guidance that should stay readable, maintainable, and consistent.
---

# Coding Standards

Use this skill to keep TS/JS/React/Node work clean, consistent, and maintainable.

This skill is not a replacement for security or foundation-ownership review.

Use it alongside:

- `foundation-module-factory` when a governed package is being turned into implementation work
- `project-mapper` when build contracts and UI/backend outputs need implementation-quality conventions
- `secure-coding-practices` when code quality work also touches auth, permissions, sensitive data, or security-critical endpoints

## Read First

Before doing substantive work for a foundation-based project, read:

1. [`../SKILL_INTERCONNECTION_MAP.md`](../SKILL_INTERCONNECTION_MAP.md)
2. the governed package, if one already exists
3. `../../foundations/README.md` when the project builds on the foundations
4. `../../foundations/FOUNDATION_MODULE_RULES.md` when the project builds on the foundations
5. `../../foundations/FOUNDATION_IMPLEMENTATION_BLUEPRINT.md` when the task affects actual implementation layout
6. [`references/typescript-javascript-react-node.md`](references/typescript-javascript-react-node.md)

## When To Activate

Activate this skill when the task involves any of the following:

- writing or refactoring TypeScript or JavaScript code
- building React components, hooks, routes, or UI flows
- building Node.js backend modules, APIs, handlers, or services
- turning specs, contracts, or package phases into implementation code
- setting or reviewing naming, typing, validation, file organization, or test conventions
- preparing a package or module for implementation-ready work

## Core Role

This skill should:

- keep code readable before it becomes clever
- prefer simple, explicit patterns over speculative abstraction
- reduce drift by making implementation follow one consistent set of conventions
- enforce typed, validated, testable code paths for TS/JS/React/Node work

This skill should not:

- redefine foundation ownership
- silently promote module-local code into the foundations
- replace security review where security-specific guidance is needed

## Operating Rules

1. Prefer readability, explicit naming, and simple control flow.
2. Prefer small functions and components with clear responsibilities.
3. Avoid direct mutation unless there is a documented reason.
4. Avoid `any` unless the user explicitly accepts a typed gap and it is recorded.
5. Use schema validation for external input boundaries when appropriate.
6. Keep file structure and naming consistent with the active project/package.
7. For React work, keep state, rendering, and hooks predictable and typed.
8. For backend/API work, keep request handling, validation, error handling, and response shape consistent.
9. When the governed package already defines implementation constraints, follow the package rather than inventing a parallel style system.
10. When a task is security-sensitive, bring in `secure-coding-practices` or the appropriate security specialist instead of treating code-quality guidance as sufficient.

## Foundation-Aware Rule

If the work belongs to a module built on the foundations:

- consume foundation-owned contracts instead of re-implementing them
- keep module-local logic inside the module
- use this skill to improve implementation quality, not to change ownership
- if a code pattern suggests a reusable gap in the ecosystem, flag it for `skill-creator` or the governed package instead of improvising a foundation change

## Output Expectations

When this skill is used during active work, it should leave behind one or more of the following:

- implementation code that follows the standards in the reference file
- review findings that point to specific violations
- package notes or build-contract notes that capture coding conventions for the module
- explicit rationale when deviating from the default conventions

## Reference Guide

Use [`references/typescript-javascript-react-node.md`](references/typescript-javascript-react-node.md) as the detailed rulebook for:

- naming and readability
- immutability and state updates
- error handling and async patterns
- React component and hook structure
- API design and validation
- file organization
- testing standards
- code-smell detection

Load only the sections needed for the current task.
