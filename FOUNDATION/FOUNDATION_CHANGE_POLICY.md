# FOUNDATION CHANGE POLICY

Rules for Editing `GeneralApp` and `FormBuilder`

> **Purpose:** The foundations are shared library contracts. They should not be directly edited during ordinary module work unless a change is being handled as explicit shared-contract promotion or correction.

---

## 1. Default Rule

During ordinary module work:

- edit the module spec
- edit the module manifest
- register through extension points
- do not directly edit the foundations

---

## 2. Direct Foundation Edits Are Allowed Only When

- the user explicitly asked for shared-contract or foundation work
- an audit concluded that a reusable capability must be promoted
- a shared-contract bug or stale cross-reference in a foundation must be corrected

---

## 3. Required Questions Before Editing a Foundation

Before changing a foundation, answer:

1. Why is an existing extension hook insufficient?
2. Which foundation owns this change: `GeneralApp` or `FormBuilder`?
3. Is the change reusable across multiple modules?
4. What adjacent contracts must also change?
5. What modules could be affected by the change?
6. Can the change remain backward-compatible?

If those questions are not answered, do not patch the foundation.

An explicit user request does **not** by itself justify placing product-local behavior into a foundation. Extension-first and promotion rules still apply.

---

## 4. Required Impact Sweep

Any foundation edit should check for impact on:

- RBAC keys
- audit/event namespaces
- capability registry entries
- interface registry entries
- extension-table registry entries
- settings hierarchy
- routes/APIs/webhooks
- bridge handlers and extension hooks
- security constraints
- module manifests
- cross-document references

---

## 5. Promotion Rule

Promotion into a foundation is the exception path.

If a feature is:

- product-local
- already expressible through existing extension hooks
- or tightly coupled to one module's domain

then it should remain in the module rather than be promoted.

Even when the user asks for a foundation edit directly, AI should still say when the cleaner design is module-local extension rather than shared-contract change.

---

## 6. No Silent Side-Effect Rule

AI must not change a foundation as a hidden side effect of implementing or auditing a product module.

Foundation changes must be explicit, intentional, and documented as shared-contract work.
