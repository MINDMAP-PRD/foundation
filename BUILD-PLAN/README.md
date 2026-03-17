# BUILD-PLAN

This folder defines the expected implementation depth for comprehensive platform features.

## Purpose

Use these documents to understand what "comprehensive" means in practice.

This folder is now intentionally limited to the files that fit the shared foundations.

Files that are module-specific, optional future capabilities, or meta/reference corpora have been moved to:

- `../BUILD-PLAN_ARCHIVE_NON_FOUNDATION/`

Comprehensive means:

- end-to-end flows, not isolated pages
- rich admin/operator tooling, not minimal settings screens
- owner-configurable-by-default elements, not hardcoded UI
- full RBAC and capability handling where documented
- auditability, configuration, bulk actions, filters, edge cases, and enforcement paths where documented
- real use of proven libraries/frameworks instead of only the easiest subset

## How to use this folder

- Treat these docs as implementation coverage references.
- Read `OWNER_CONTROL_CONTRACT.md` before building any admin, settings, branding, RBAC, or configurable UI surface.
- Read the canonical foundation docs before treating any build-plan file as implementation authority:
  - `foundations/README.md`
  - `foundations/BUILD_PLAN_EXTRACTS/BUILD_PLAN_FOUNDATION_PROMOTION_MAP.md`
  - `foundations/BUILD_PLAN_EXTRACTS/FOUNDATION_COMPREHENSIVE_WIRING_PLAN.md`
- Use the foundation skills when applying these docs:
  - `skills/foundation-extension-auditor/SKILL.md`
  - `skills/foundation-module-factory/SKILL.md`
- If a build request mentions admin, capabilities, users, roles, settings, notifications, AI, integrations, payments, or profiling, check the matching `BUILD-PLAN` docs first.
- When a library/framework already supports advanced requirements in these docs, use those capabilities instead of rebuilding shallow custom versions.
- If a requested build surface is simpler than the relevant build-plan doc, ask whether the user wants a reduced scope before cutting features.

## Active foundation-fit files

- `admin.md`
- `ADMIN_CAPABILITIES.md`
- `comprehensive-element-configuration-flow.md`
- `comprehensive-error-logs-flow.md`
- `comprehensive-mcp-integrations-flow.md`
- `comprehensive-notification-flow.md`
- `comprehensive-payment-system-flow.md`
- `comprehensive-rbac-roles-capabilities-flow.md`
- `comprehensive-roles-capabilities-rbac-flow.md`
- `comprehensive-settings-management-flow.md`
- `comprehensive-user-management-flow.md`
- `rbac-api-endpoints.md`
- `rbac-complete-integration-guide.md`
- `rbac-implementation-examples.md`
- `rbac-implementation-part2.md`
- `MASTER_CONSISTENT_FEATURES_LIST.md`
- `OWNER_CONTROL_CONTRACT.md`

## Important rule

Do not silently reduce a comprehensive requirement into a basic MVP if the project materials ask for a comprehensive system.

If time, technical constraints, or source ambiguity require a smaller slice, pause and discuss that downgrade explicitly.
