# Hierarchy Rules Reference

## The Three-Tier Model

Every project mapped by this skill uses a three-tier ownership model regardless
of what the docs call the roles. The canonical names are:

```
Owner        → maps to: system_owner, root, god, platform_owner, founder
Super Admin  → maps to: super_admin, workspace_admin, org_admin, elevated_admin
Admin        → maps to: admin, editor, manager, operator, staff
```

If the docs use different names, note the mapping in HIERARCHY.md under a
"Role Name Mapping" section.

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
Is it explicitly marked system_owner only?  → Owner exclusive
Is it marked super_admin-capable?           → Super Admin (Owner can also do it)
Is it marked admin-delegatable?             → Admin (Super Admin and Owner can also)
Is it marked public/no auth?                → Not in hierarchy, note separately
No role restriction documented?             → Default to Admin, flag as TBD
```

### Step 3 — Delegation rules
- Owner can always grant any permission to Super Admin
- Super Admin can always grant any of their own permissions to Admin
- No role can grant permissions they do not themselves hold
- Delegation is configured by Owner in the hierarchy config file — not hardcoded

## What Owner Always Controls (regardless of docs)
- Creating/deleting Super Admin accounts
- Modifying system-level settings
- Viewing all audit logs
- Overriding any role gate
- Configuring capability connectors / integrations

## What to Flag
If any of the following are missing from the docs, flag them in HIERARCHY.md
under a "⚠️ Gaps Requiring Owner Decision" section:
- No explicit permission for a UI action that clearly exists
- A role that appears in the docs but has no defined permissions
- Permissions that appear in two roles' lists with no delegation hierarchy
- Any "TBD" or "TODO" in the original spec's RBAC section
