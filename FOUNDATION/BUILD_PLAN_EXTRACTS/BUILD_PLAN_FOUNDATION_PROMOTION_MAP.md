# BUILD PLAN FOUNDATION PROMOTION MAP

Classification of every file in `../BUILD-PLAN/` by foundation fit.

## Promotion labels

- `Promote -> GeneralApp`
- `Promote -> FormBuilder`
- `Optional -> GeneralApp capability/module`
- `Keep -> module-specific`
- `Keep -> reference/meta only`

## File-by-file map

| Build-plan file | Decision | Reason | Extraction target |
|---|---|---|---|
| `admin.md` | `Promote -> GeneralApp` | Defines owner/admin operating model, admin hierarchy, RBAC depth, and capability-driven integrations | GeneralApp admin shell + owner control model |
| `ADMIN_CAPABILITIES.md` | `Promote -> GeneralApp` | Defines comprehensive admin surfaces and operator controls | GeneralApp admin capability registry + owner UI surfaces |
| `AI_PROJECT_PROFILER_COMPLETE.md` | `Optional -> GeneralApp capability/module` | Valuable as a cross-product profiling/recommendation service, but not required for the base runtime | Optional project-profiler capability |
| `AI_PROJECT_PROFILER_ENGINE.md` | `Optional -> GeneralApp capability/module` | Same as above; useful for intake and recommendation, not base platform ownership | Optional project-profiler capability |
| `COMPLETE_EXPANSION_FEATURES_INDUSTRIES_AI.md` | `Keep -> reference/meta only` | Large expansion bank; contains mixed domains and optional ideas rather than a clean foundation contract | Reference corpus only |
| `COMPLETE_SYSTEM_OVERVIEW.md` | `Keep -> reference/meta only` | High-level explanation of the recommendation system, not a direct runtime contract | Reference corpus only |
| `comprehensive-ai-agents-flow.md` | `Optional -> GeneralApp capability/module` | Could become a shared AI-agent orchestration capability, but should not be forced into the core foundation first | Optional AI orchestration capability |
| `comprehensive-element-configuration-flow.md` | `Promote -> GeneralApp` | Strong fit for owner-configurable UI, element settings, and runtime configuration | Foundation UI config + owner-control surfaces |
| `comprehensive-error-logs-flow.md` | `Promote -> GeneralApp` | Shared observability, incidents, alerts, and admin error operations belong to the platform | Audit/observability/error foundation |
| `comprehensive-mcp-integrations-flow.md` | `Promote -> GeneralApp` | Shared provider-agnostic integration and capability orchestration fits the platform directly | Integration/capability foundation |
| `comprehensive-mindmap-flow.md` | `Keep -> module-specific` | Mindmap is a product capability, not a universal platform primitive | Module-only |
| `comprehensive-notification-flow.md` | `Promote -> GeneralApp` | Notifications are a shared platform concern with adapters and admin control | Notification foundation |
| `comprehensive-payment-system-flow.md` | `Promote -> GeneralApp` | Payments should live behind shared adapters and plan/billing controls | Payment/billing foundation |
| `comprehensive-rbac-roles-capabilities-flow.md` | `Promote -> GeneralApp` | Core shared authorization and owner control surface | RBAC/capability foundation |
| `comprehensive-roles-capabilities-rbac-flow.md` | `Promote -> GeneralApp` | Same domain; merge with the other RBAC document into canonical shared contracts | RBAC/capability foundation |
| `comprehensive-settings-management-flow.md` | `Promote -> GeneralApp` | Shared settings hierarchy, overrides, and config authority belong to the platform | Settings foundation |
| `comprehensive-user-management-flow.md` | `Promote -> GeneralApp` | Shared identity administration, user lifecycle, and operator flows belong to the platform | User-management foundation |
| `COMPREHENSIVE_INDUSTRY_PROFILES.md` | `Keep -> reference/meta only` | Strong input for project profiling, but not shared runtime ownership | Reference corpus only |
| `FINAL_COMPLETE_SUMMARY.md` | `Keep -> reference/meta only` | Summary document, not canonical contract | Reference corpus only |
| `INTELLIGENT_FEATURE_FILTERING_SYSTEM.md` | `Optional -> GeneralApp capability/module` | Good for an intake/recommendation engine, not required for the base runtime | Optional project-profiler capability |
| `KANBAN_IMPLEMENTATION.md` | `Keep -> module-specific` | Kanban is a product/module surface, not a shared platform primitive by default | Module-only |
| `MASTER_CONSISTENT_FEATURES_LIST.md` | `Keep -> reference/meta only` | Excellent universal reference list; only selected universal features should be promoted | Reference corpus with selective extraction |
| `MASTER_INDEX_ALL_DOCUMENTATION.md` | `Keep -> reference/meta only` | Index only | Reference corpus only |
| `OWNER_CONTROL_CONTRACT.md` | `Promote -> GeneralApp` | Establishes the owner-configurable-by-default platform rule | GeneralApp admin/settings/UI control model |
| `rbac-api-endpoints.md` | `Promote -> GeneralApp` | Shared RBAC API contract material | RBAC/capability foundation |
| `rbac-complete-integration-guide.md` | `Promote -> GeneralApp` | Shared integration guidance for RBAC enforcement | RBAC/capability foundation |
| `rbac-implementation-examples.md` | `Promote -> GeneralApp` | Useful implementation patterns for shared authorization runtime | RBAC/capability foundation |
| `rbac-implementation-part2.md` | `Promote -> GeneralApp` | Same as above | RBAC/capability foundation |
| `README.md` | `Keep -> reference/meta only` | Overview only | Reference corpus only |
| `SCOUTS_IMPLEMENTATION.md` | `Optional -> GeneralApp capability/module` | Could become a reusable operations/scouting capability, but not part of the base runtime first | Optional ops/scouts capability |
| `TRAINING_DATA_GENERATION_SYSTEM.md` | `Keep -> reference/meta only` | ML/training support material, not foundation runtime ownership | Reference corpus only |

## Foundation-level conclusions

### Strong GeneralApp promotions

These are clean shared-platform fits and should feed the canonical `GeneralApp` foundation:

- owner/admin hierarchy
- user management
- RBAC and capabilities
- settings hierarchy
- owner-configurable UI elements
- notifications
- payments/billing adapters
- MCP/integration capability bindings
- error logs, incidents, alerts, and operational observability

### FormBuilder promotions

No `BUILD-PLAN` file is a clean full `FormBuilder` spec.

The closest reusable input is:

- `comprehensive-element-configuration-flow.md`

But that should influence shared owner-configurable UI and builder/admin patterns first, not be promoted wholesale into `FormBuilder`.

### Optional shared capabilities

These should be treated as optional shared capabilities or future modules, not mandatory base-foundation scope:

- project profiler / feature recommendation system
- AI agent orchestration
- scouts / operational intelligence

### Module-only systems

These stay outside the base foundation unless a future multi-product need emerges:

- mindmap
- kanban

## Operational rule

The foundation should promote the universal wiring and contracts, not copy every feature system in full.

Promote:

- registries
- settings authority
- admin/operator shells
- adapters
- enforcement contracts
- auditability

Keep module-only business surfaces out of the shared core unless they become clearly reusable across multiple products.
