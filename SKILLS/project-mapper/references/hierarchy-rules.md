# Hierarchy Rules Reference

## Source-Defined Role Model

Every project mapped by this skill must use the role model documented in the
source files.

Do not impose canonical role names, a fixed number of tiers, or an inherited
power ladder unless the source files explicitly define them.

If the docs use aliases for the same role, note the mapping in HIERARCHY.md
under a "Role Name Mapping" section. If the docs do not define a complete
hierarchy, render only the documented pieces and flag the rest as `TBD`.

## Deriving Permissions from Docs

### Step 1 — Find all permission strings
Search every file for:
- RBAC tables or registries
- Permission strings (e.g. `form.create`, `user.invite.send`)
- SCAN-04 sections or equivalent
- Role capability lists
- "Only X can..." or "Restricted to X" statements

### Step 2 — Classify each permission
```
Is it explicitly restricted to one role?    → mark that exact role
Is it marked delegatable or inheritable?    → record exactly as documented
Is it assigned to multiple roles?           → list those exact roles
Is it marked public/no auth?                → Not in hierarchy, note separately
No role restriction documented?             → Flag as `⚠️ TBD — role assignment not specified in source`
```

### Step 3 — Delegation rules
- Only record delegation chains that the source files explicitly support
- No role can grant permissions they do not themselves hold unless the docs say otherwise
- Delegation must be modeled from the documented authority surface in the hierarchy config file — not hardcoded

## Authority-only candidates to verify in the docs
- Creating/deleting elevated admin accounts
- Modifying system-level settings
- Viewing all audit logs
- Overriding any role gate
- Configuring capability connectors / integrations

Only include these in HIERARCHY.md when the source files explicitly support
them. If the docs imply one of these controls exists but do not name the
permission or role assignment, flag it as `TBD` in the gaps section instead of
adding it to the permission registry.

## What to Flag
If any of the following are missing from the docs, flag them in HIERARCHY.md
under the canonical `## ⚠️ Gaps — Require Authority Decision` section:
- No explicit permission for a UI action that clearly exists
- A role that appears in the docs but has no defined permissions
- Permissions that appear in two roles' lists with no delegation hierarchy
- Any "TBD" or "TODO" in the original spec's RBAC section
