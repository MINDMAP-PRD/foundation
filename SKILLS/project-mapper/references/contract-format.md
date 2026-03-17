# Build Contract Format Reference

## What a Build Contract Is

A build contract is a binding technical agreement between the frontend and backend
teams (or the frontend and backend halves of a solo build). It defines:
- Every API endpoint and its exact schema
- Every UI component and its exact props/state/wiring
- Every cross-module integration and its exact protocol
- Every build phase and its exact gate conditions

The contract is derived from the project spec, not invented. Every field,
every endpoint, every constraint must trace back to a documented source.

---

## BACKEND.md

### Endpoint block format
```markdown
### <METHOD> /api/<path>

**Spec reference:** Section <N> - <section title>
**Purpose:** <One sentence>
**Auth required:** <None | Session | Role: [role list]>
**Rate limit:** <N requests per M minutes per IP|user, or "Default">
**RBAC permission:** <permission.string>

**Request body:**
| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|

**Response (200):**
| Field | Type | Notes |
|-------|------|-------|

**Error responses:**
| HTTP Code | Code string | Condition |
|-----------|-------------|-----------|

**Lifecycle / ordering rules:**
<Any sequencing constraints from the spec, e.g. "CAPTCHA must be verified
before validation" - quote verbatim>

**Calls out to:**
- <service or module this endpoint invokes>

**Called by:**
- <frontend component or other service that calls this>

**Acceptance condition:**
<Gate condition from the spec - verbatim>

**State transitions triggered:**
- <state transition from FILE_MAP.md> - <what causes it>

**Skill notes:**
- <skill-name> - <how this endpoint should follow that skill's guidance>
```

---

## FRONTEND.md

### Component block format
```markdown
### <ComponentName>

**File:** `<admin-ui|user-ui>/src/components/<ComponentName>.tsx`
**Renders in page(s):** <PageName>
**Role gate:** `RoleGate minRole="<role>"` or `None`
**Spec reference:** Section <N>

**Props:**
| Prop | Type | Required | Default | Description |
|------|------|----------|---------|-------------|

**Local state:**
| Key | Type | Initial | Description |
|-----|------|---------|-------------|

**States displayed:**
- <state name from FILE_MAP.md> - <where/how this component shows it>

**Actions rendered:**
- <action label from FILE_MAP.md> - <where/how this component renders it and which `RoleGate` protects it>

**API calls made:**
| Endpoint | Trigger | On success | On error |
|----------|---------|------------|----------|

**Emits (events/callbacks):**
| Callback prop | When | Payload |
|---------------|------|---------|

**Depends on components:**
- <ComponentName> - reason

**Acceptance condition:**
<Gate from spec>

**Skill notes:**
- <skill-name> - <how this component should follow that skill's guidance>
```

---

## INTEGRATIONS.md

### Integration block format
```markdown
### <Module A> <-> <Module B>

**Type:** <API call | Event | Shared DB | Webhook | Background job>
**Direction:** <A -> B | B -> A | Bidirectional>
**Spec reference:** Section <N> (Module A), Section <M> (Module B)

**Trigger:**
<What causes this integration to fire>

**Payload:**
| Field | Type | Notes |
|-------|------|-------|

**Contract rules:**
- <Rule 1 from spec>
- <Rule 2 from spec>

**Failure handling:**
<What happens if this integration fails - from spec>

**Acceptance condition:**
<Both modules must satisfy their individual acceptance conditions AND
this integration contract>

**State changes across boundary:**
- <state change in Module A or Module B> - <what this integration causes>

**Skills invoked:**
- <skill-name> - <why this integration needs it>
```

---

## PHASES.md

### Phase block format
```markdown
## Phase <N> - <Name>

**Goal:** <What this phase delivers>
**Must complete before:** Phase <N+1>
**Depends on phases:** <N-1, N-2, ...>
**Spec sections / derivation:** Section <list> or `Derived from FILE_MAP.md dependency ordering`

**What gets built:**
- <Deliverable 1>
- <Deliverable 2>

**Skills invoked:**
- <skill-name> - <why this phase needs it>

**Key constraints (non-negotiable):**
1. <Constraint from spec - verbatim>
2. <Constraint from spec - verbatim>

**Gate checklist:**
- [ ] <Verifiable item - can be checked YES/NO>
- [ ] <Verifiable item>

**Stubs allowed:**
<List any placeholders that may be left for later phases, and which phase resolves them>

**Drift signals (do not do these in this phase):**
- <Thing that would constitute build drift>

**Acceptance condition:**
All gate checklist items are YES. No drift signals fired. Handoff summary produced.
```

---

## General Principles

1. **Trace everything.** Every contract entry must cite either a spec section
   or an explicitly allowed mapping derivation source defined by this skill
   (for example `FILE_MAP.md` dependency ordering for inferred phases).
2. **Sequence is absolute.** If the spec defines an ordering (like a 10-step
   submission lifecycle), reproduce it exactly in BACKEND.md.
3. **No invented fields.** If a field isn't in the spec, it isn't in the contract.
   Exception: mapping-derived fields sourced from FILE_MAP.md, HIERARCHY.md,
   or the Skill Registry are required when the template includes them
   (for example `States displayed`, `Actions rendered`, `State transitions triggered`,
   `State changes across boundary`, `Skills invoked`, and `Skill notes`).
4. **Failure paths are required.** Every endpoint block must have error responses.
   Do not leave error responses blank.
5. **Acceptance conditions propagate.** If a backend endpoint has an acceptance
   condition, the frontend component that calls it inherits that condition too.
