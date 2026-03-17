# BUILD-PLAN

This folder defines the expected implementation depth for comprehensive platform features.

## Purpose

Use these documents to understand what "comprehensive" means in practice.

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
- If a build request mentions admin, capabilities, users, roles, settings, notifications, AI, integrations, payments, or profiling, check the matching `BUILD-PLAN` docs first.
- When a library/framework already supports advanced requirements in these docs, use those capabilities instead of rebuilding shallow custom versions.
- If a requested build surface is simpler than the relevant build-plan doc, ask whether the user wants a reduced scope before cutting features.

## Important rule

Do not silently reduce a comprehensive requirement into a basic MVP if the project materials ask for a comprehensive system.

If time, technical constraints, or source ambiguity require a smaller slice, pause and discuss that downgrade explicitly.
