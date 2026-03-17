# EXTENSION TABLE REGISTRY SPEC

Canonical Index of Approved Shared Extension Tables

> **Purpose:** This document is the single lookup source for determining whether a table is an approved shared extension surface that modules may legally interact with outside their own module-owned storage.

---

## 1. Default Rule

If a table is not listed in this registry, treat it as **not approved** as a shared extension table.

In that case, the table should be treated as either:

- foundation-owned
- or module-owned

but not as an implicit hybrid.

---

## 2. Required Fields for Any Approved Extension Table

Every approved extension table entry must declare:

- table name
- owning foundation
- purpose
- allowed module interaction
- whether modules may read it
- whether modules may write to it
- whether modules may only register rows through a managed API/hook

---

## 3. Current Registry

At this time, there are **no globally approved shared extension tables** documented in this registry.

Until a table is added here, modules should assume that module-specific data belongs in module-owned tables and that shared foundation tables remain foundation-owned.

---

## 4. Adding an Extension Table

Add a table here only when:

- the table is genuinely shared across multiple modules
- the owning foundation is clear
- direct module interaction is safer than forcing duplicate module-owned copies
- the allowed interaction model is explicitly bounded

Adding a new extension table should be handled as explicit foundation work, not as ordinary module implementation.
