# DYNAMIC FORM BUILDER

Generic Form Engine Specification

> **Foundation Entry Point:** If an AI or engineer needs the high-level explanation of how the foundations work, how modules attach to them, or how the model applies in single-repo versus multi-repo setups, read [README.md](README.md) first.

> **Document role:** This document specifies the **generic, reusable form engine**. It is completely product-agnostic and contains no product-specific business logic (no client records, no Namecheap integration, no billing workflows). Product modules integrate via **extension hooks** (`afterSubmitActions`, `fieldTypePlugins`, `paymentAdapters`) registered at runtime. All shared primitives (users, sessions, files, audit) are inherited from GeneralApp §23.

---

> **Foundation Operating Rules:** Before any AI or engineer creates, edits, or audits a product/module spec on top of the foundations, it **MUST first read** [FOUNDATION_MODULE_RULES.md](FOUNDATION_MODULE_RULES.md). That file defines the operating model for shared foundations, module isolation, extension boundaries, and promotion rules.

> **Library Architecture:** The foundations are meant to operate like a shared library/framework consumed by product modules. Read [FOUNDATION_LIBRARY_ARCHITECTURE.md](FOUNDATION_LIBRARY_ARCHITECTURE.md), [MODULE_MANIFEST_SPEC.md](MODULE_MANIFEST_SPEC.md), [CAPABILITY_REGISTRY_SPEC.md](CAPABILITY_REGISTRY_SPEC.md), [INTERFACE_REGISTRY_SPEC.md](INTERFACE_REGISTRY_SPEC.md), [EXTENSION_TABLE_REGISTRY_SPEC.md](EXTENSION_TABLE_REGISTRY_SPEC.md), and [FOUNDATION_CHANGE_POLICY.md](FOUNDATION_CHANGE_POLICY.md) before proposing module-level extensions or foundation promotions.

> **AI Extension Guardrail:** Before any AI creates, edits, or audits a new product/module spec that builds on FormBuilder + GeneralApp, it **MUST first read** [skills/foundation-extension-auditor/SKILL.md](../skills/foundation-extension-auditor/SKILL.md). That skill is the required preflight for detecting duplicate foundation declarations, ownership conflicts, valid product extensions, and reusable features that should be promoted into a foundation.

> **Module Package Workflow:** If an AI is turning an idea, module concept, or early spec into a comprehensive phased build package folder, it **SHOULD use** [skills/foundation-module-factory/SKILL.md](../skills/foundation-module-factory/SKILL.md) after ownership review so inventories, contracts, specs, phases, and progress tracking remain aligned to the foundations.
>
> **Implementation acceleration rule:** Build the reusable form engine on top of proven libraries where possible, but keep canonical form contracts, extension points, and provider boundaries owned by the foundation. The recommended stack and substitution rules are defined in [FOUNDATION_DEPENDENCY_STRATEGY.md](FOUNDATION_DEPENDENCY_STRATEGY.md).

Form Entity • Canvas & Layout Engine • All Field Types & States • Sections & Columns • Conditional Logic • Validation Engine • System Forms • Form Templates • Post-Submission Actions • Preview & Test Runner • RBAC • Interconnection Mesh • Admin Configuration • i18n • DataBinding • Accessibility • Submission Versioning • Concurrency • Embedded Forms • File Storage • Analytics • Forms List • Webhook Registry • Theming • Expression Engine • API Endpoints • Submit Button • Locale Selection • Submission Lifecycle Ordering • File-Upload Resume • Payment Re-charge Guard • Refund Notifications • Repeater Formula Support • GDPR Review Queue • Draft-Visibility Transitions • Digest Delivery Engine • Security Policy Registry • Consent Field Specification • PDF Export • Short-Link & QR Registry • Dark-Mode Theming • Autocomplete Source Security • RateLimitOverride Type

Version 6.1 — Build Specification  
Part of Unified SaaS Platform: General Application + Form Builder + Product Modules

> **Integration Notice:** This document references GeneralApp §23 for all shared entities. Product-specific integrations use extension hooks defined in §2.1 (`extensionHooks`) rather than hardcoded references. See Appendix C for integration patterns.

> **Changelog from v5.0:** 12 issues resolved — duplicate section heading eliminated, `refundNotificationConfig` added to FormDefinition schema, `tenant.defaultFormTheme` type corrected from `ThemeTokens` to `FormTheme`, stale `ConditionGroup` cross-references updated to §51.5, missing §66 cross-reference added to §12.4, PDF export and QR code endpoints added to §44 API reference, `fb.submissions.export.elevated` added to SCAN-04 RBAC registry, spurious §3.1 Build Warnings callout removed (completing the DUP-03 fix), `FormDraft` entity schema added to §55, `captchaFailOpenOnProviderError` field name cross-referenced in §41.2, and elevated export permission named explicitly in §47.3. See Appendix B (§75) for the full v5.0 issue register.

---

## TABLE OF CONTENTS

1. Form Builder — Architecture & Interconnection Scan
2. Extension Architecture & Plugin System
3. Form Definition Entity
4. Form Builder Canvas
5. Field Type Catalogue
6. Field Properties Panel
7. Conditional Logic Engine
8. System Forms — Protected & Configurable
9. Post-Submission Actions
10. Payment Field — Full Specification
11. Form Templates
12. Preview & Test Runner
13. Submission Management
14. Form Settings & Admin Configuration
15. Multi-Step Forms
16. Version History & Rollback
17. Global Error Paths & Edge Cases
18. Internationalisation (i18n)
19. Data Binding
20. Accessibility (A11y)
21. Submission Schema Versioning
22. Concurrent Editing & Optimistic Concurrency
23. Embedded Forms — Auth, CSRF & Cross-Origin
24. File Storage & Lifecycle
25. Form Analytics & Metrics
26. Form Duplication
27. Repeater Field — Full Specification
28. CAPTCHA — Verification Lifecycle
29. After-Submit `trigger-api` — Full Specification
30. Forms List View
31. Form Theming System
32. Validation Rule Schema
33. FieldOption & Dynamic Option Sources
34. Expression Engine — Grammar, Sandbox & Error Handling
35. Submit Deduplication & Once-Per-User Enforcement
36. Publish Pre-Flight Checks
37. Form Archiving
38. Undo / Redo
39. Build Warnings Panel
40. Trigger-Email Template System
41. Webhook Registry
42. CAPTCHA Tenant Configuration Surface
43. Form Render Engine — Serving Architecture & Caching
44. Submission Export — Format Specification
45. REST API Reference
46. Analytics PII & GDPR Handling
47. Address Field — AddressObject Type
48. PaymentResult Type
49. Submission Lifecycle — Authoritative Execution Order
50. Submit Button — Full Specification
51. Locale Selection — Detection & Resolution
52. Supporting Types
53. Field Serialisation Reference
54. Async Validation Endpoints — Security & Architecture
55. Live-Publish Mid-Session Handling
56. Save-and-Resume Draft Expiry
57. File Upload in Multi-Step — Abandonment & Resume Race Condition
58. Payment Re-Charge Guard in Multi-Step
59. Submission Payload Size Limits
60. Search Index Latency
61. Refund Handling
62. Security Policy Registry — SSRF, CORS & Credential Rules
63. Digest Delivery Engine
64. Short-Link & QR Code Registry
65. Consent / Terms Field — Full Specification
66. Single-Submission PDF Export
67. GDPR Review Queue
68. Draft & Deduplication State Transitions on Visibility Change
69. Refund Notification Actions
70. Repeater Field in Payment Formulas
71. Dark-Mode Theme Token System
72. Autocomplete Field — Source Security & Architecture
73. Concurrent Multi-Tab Submission Guard
74. Locale Switcher Position Configuration
75. Appendix A — v4.0 Audit Issue Register
76. Appendix B — v5.0 Audit Issue Register
77. Appendix C — Cross-System Integration Reference

---

## 1. FORM BUILDER — ARCHITECTURE & INTERCONNECTION SCAN

The Form Builder is a first-class system feature, not a plugin. Every form in the application — authentication forms, payment forms, registration flows, profile update forms — is rendered and enforced by this engine. Users with builder permissions — typically `system_owner`, `super_admin`, or delegated `admin` roles — can open forms in the builder, modify fields, layout, validation rules, and post-submission behaviour without writing code or deploying.

### 1.1 Foundational Rules

- **Rule 1:** No form anywhere in the application is hardwired. Every form is a FormDefinition record.
- **Rule 2:** System forms (login, register, etc.) have a protected core that cannot be deleted, but can be extended with additional fields, restyled, and restructured.
- **Rule 3:** Every field on every form is a first-class entity with its own states, validation rules, conditional visibility, and data bindings.
- **Rule 4:** Structural changes to a form (adding a field, removing a column) are versioned. The live form always serves the published version. Drafts exist independently. A form may simultaneously have an active published version AND an active draft; these are tracked separately. See §2.1 and §15.4.
- **Rule 5:** Post-submission logic is declarative — defined in the form's AfterSubmitConfig, not in code.
- **Rule 6:** Any form can be previewed in real-time and test-submitted from inside the builder.
- **Rule 7:** Every submission record stores the formVersion it was collected on. Viewing an old submission always renders against the snapshot of the form at that version — never against the current form definition.
- **Rule 8:** Conditional logic actions (including `set-value`) are re-evaluated server-side on submission using the stored FormDefinition. Client-computed values are display-only and are not trusted.
- **Rule 9:** Two admins may not simultaneously hold an edit lock on the same form. Presence is shown; the second editor is offered read-only or lock-request access.
- **Rule 10:** The Form Render Engine is completely independent of the builder. Builder unavailability does not affect form serving.
- **Rule 11:** The authoritative submission execution order is: CAPTCHA verify → server-side validation → payment checkout/confirmation → store submission → write-back → after-submit actions. See Section 48.
- **Rule 12:** A `FormField.required: true` property and a `ValidationRule` of type `required` are two mechanisms for expressing the same constraint. The authoritative precedence table is in §5.2 and applies to all combinations. Summary: `required: true` on the field property wins over a disabled `required` ValidationRule; an enabled `required` ValidationRule wins even when `required: false` is set on the field. A disabled `required` ValidationRule does NOT override `required: true`. See §5.2 for the complete matrix.
- **Rule 13:** Forms can be configured to create or update product-owned records through a product bridge. The shared Form Builder only defines the bridge surface; consuming products define the target entity semantics. Any consuming product spec may provide its own concrete implementation of this pattern.
- **Rule 14:** Payment fields resolve through the shared payment capability registry and may hand successful payment results to consuming products through bridge mappings or product-specific billing adapters. The shared form engine does not hardcode any single downstream billing product.
- **Rule 15:** File uploads use the unified file/storage contract defined by the platform/core layer (GeneralApp §23.3). Product modules may project additional references onto that shared file identity, but the underlying storage contract is shared.
- **Rule 16:** Auth/security system forms are operational workflows, not generic data-capture submissions. Secrets such as passwords, current-password confirmations, CAPTCHA tokens, and one-time security credentials are never persisted in standard `FormSubmission.data`, never exported, never indexed for search, and never exposed in submission-edit tooling.

### 1.2 ISP Scan — Form Builder System

| Scan | Description |
|------|-------------|
| **SCAN-01 AUTH** | The builder itself requires admin authentication. Rendered forms may be public (login, register) or auth-gated (profile update). Configured per form. |
| **SCAN-02 SESSION** | Builder state (unsaved canvas changes) is saved to a session draft. Lost sessions restore draft from server-side autosave (every 30s). |
| **SCAN-03 TENANT** | Every FormDefinition is tenant-scoped. System form overrides are per-tenant. Tenant A's login form is independent of Tenant B's. |
| **SCAN-04 RBAC** | `fb.form.create`, `fb.form.edit`, `fb.form.delete`, `fb.form.archive`, `fb.form.publish`, `fb.form.view`, `fb.submissions.view`, `fb.submissions.export`, `fb.submissions.export.elevated` (restricted to `system_owner` by default; may be delegated to `super_admin`; includes IP address in JSON exports — see §13.5), `fb.submissions.edit`, `fb.submissions.delete`, `fb.template.create`, `fb.template.delete`, `fb.field.customCss`, `fb.test.run`, `fb.system.edit` (restricted to `system_owner` by default; may be delegated to `super_admin`), `fb.form.duplicate`. Note: `fb.form.archive` is a distinct permission from `fb.form.delete` — archiving is reversible and non-destructive; granting `fb.form.archive` does not imply `fb.form.delete`. |
| **SCAN-05 EMAIL** | Post-submission EMAIL_BRANCH — confirmation email to submitter, notification email to admin. Configurable per form. See Section 39. |
| **SCAN-06 MCP** | Conditional field logic can call `llm.classify` to auto-classify a text field. File upload fields use `storage.upload`. CAPTCHA fields use `captcha.verify`. Payment fields resolve through the shared payment capability registry: embedded payment UIs use `payment.create_intent` and `payment.confirm_intent`; redirect/hosted payment flows use `payment.create_checkout`; webhook-driven settlement uses `payment.verify_webhook`; saved-method flows may use `payment.method.save` when supported by the active provider binding. |
| **SCAN-07 AUDIT** | `fb.form.created`, `fb.form.published`, `fb.form.field.added`, `fb.form.field.removed`, `fb.submission.received`, `fb.form.system.modified`, `fb.submission.edited`, `fb.submission.deleted`, `fb.form.duplicated` — all logged with before/after snapshot. |
| **SCAN-08 NOTIF** | Admin notified on new form submission (configurable per form). User notified on confirmation. |
| **SCAN-09 RATE-LIMIT** | form.submit limited per form (configurable), default 10/hour per IP for public forms, 60/hour per user for auth-gated forms. |
| **SCAN-10 SEARCH** | Form submissions searchable by admin. Fields marked `searchable: true` are indexed asynchronously (see §59 for indexing latency). |
| **SCAN-11 SETTINGS** | Forms have their own settings surface inside the builder (visibility, auth-gate, rate-limit, CORS origins, data retention). |
| **SCAN-12 FAILURE** | Form load fails → show cached version with 'Form may be outdated' banner. Submission fails → save to local draft, retry on reconnect. Builder unavailable → forms still render and submit (builder is separate from render engine). |
| **SCAN-13 I18N** | All user-facing text on a form (labels, placeholders, help text, section titles, error messages, success messages) is locale-keyed. See Section 17. |
| **SCAN-14 A11Y** | All rendered forms target WCAG 2.1 AA. Keyboard navigation, screen reader announcements, focus management, and colour contrast are specified. See Section 19. |
| **SCAN-15 CONCURRENCY** | Edit lock with ETag-based optimistic concurrency for the builder. See Section 22. |
| **SCAN-16 EMBED** | Iframe and popup embeds are supported with explicit CORS, CSRF token fetch, and cookie-alternative auth flows. See Section 23. |
| **SCAN-17 FILE-STORAGE** | Uploaded files are stored as typed StoredFile records with signed URLs, lifecycle rules, and deletion hooks tied to submission retention. See Section 24. |

### 1.3 System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     FORM BUILDER (Admin UI)                 │
│  Forms List │ Canvas │ Field Palette │ Properties │ Preview │
└───────────────────────────┬─────────────────────────────────┘
                            │ CRUD via REST + ETag concurrency
┌───────────────────────────▼─────────────────────────────────┐
│                    FORM DEFINITION API                       │
│  /forms  /forms/:id  /forms/:id/publish  /forms/:id/test    │
│  /forms/:id/lock  /forms/:id/duplicate  /forms/:id/versions │
│  /forms/:id/submissions  /forms/:id/fields/:id/upload-url   │
│  See Section 44 for full REST API reference.                │
└───────────┬──────────────────────┬────────────────┬─────────┘
            │                      │                │
       ┌────▼──────┐    ┌──────────▼──────┐  ┌─────▼──────┐
       │ Form Store │    │ Validation Engine│  │ Rule Engine│
       │ (Postgres) │    │ (server + client)│  │(RBAC+Cond) │
       └──────┬─────┘    └─────────────────┘  └────────────┘
              │
┌─────────────▼───────────────────────────────────────────────┐
│                   FORM RENDER ENGINE                         │
│  React SSR + hydration. Served from /forms/{slug}.          │
│  FormDefinition fetched at request time. CDN-cached.        │
│  Cache invalidated on publish. See Section 43.              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. EXTENSION ARCHITECTURE & PLUGIN SYSTEM

> **Core Principle:** The Form Builder is a **generic form engine** that can be extended at runtime without code changes. Product modules register plugins, field types, validators, and adapters through a unified extension API.

### 2.1 Extension Philosophy

| Aspect | Design Rule |
|--------|-------------|
| **Runtime Registration** | Extensions are registered at system startup, not compile time. |
| **Namespace Isolation** | Each module uses its prefix (`fb.*` for core, `ext.*` for third-party). |
| **Schema Validation** | Extended types are validated against JSON schemas. |
| **Backward Compatibility** | Core never removes extension points; new features are additive. |
| **Capability Detection** | UI adapts to available extensions dynamically. |

### 2.2 Extension Points Summary

| Subsystem | Extension Mechanism | Registration Pattern | Documentation |
|-----------|---------------------|---------------------|---------------|
| **Custom Field Types** | `fieldTypeRegistry` | `registerFieldType()` | §2.3, §4 |
| **Field Validators** | `validatorRegistry` | `registerValidator()` | §2.3, §6 |
| **Post-Submit Actions** | `afterSubmitRegistry` | `registerActionHandler()` | §2.3, §9 |
| **Payment Adapters** | `paymentAdapterRegistry` | `registerPaymentAdapter()` | §2.3, §10 |
| **Data Sources** | `optionSourceRegistry` | `registerOptionSource()` | §2.3, §5 |
| **Bridge Handlers** | `bridgeHandlerRegistry` | `registerBridgeHandler()` | §2.3, GeneralApp §23.9 |
| **Expression Functions** | `expressionRegistry` | `registerFunction()` | §2.3, §51 |
| **Export Formats** | `exportFormatRegistry` | `registerExportFormat()` | §2.3, §44 |
| **i18n Namespaces** | `i18nRegistry` | `registerNamespace()` | §2.3, §17 |
| **Theme Tokens** | `themeRegistry` | `registerTheme()` | §2.3, §30 |
| **Analytics Events** | `analyticsRegistry` | `registerEvent()` | §2.3, §24 |
| **Webhook Events** | `webhookRegistry` | `registerEvent()` | §2.3, §41 |

### 2.3 Dynamic Registration API

Product modules register extensions at system initialization:

```typescript
interface FormBuilderExtensions {
  // Field Types — Custom input components
  registerFieldType(config: FieldTypeConfig): void;
  
  // Validators — Custom validation logic
  registerValidator(config: ValidatorConfig): void;
  
  // After-Submit Actions — Custom post-submission handlers
  registerAfterSubmitAction(config: ActionConfig): void;
  
  // Payment Adapters — Custom payment flows
  registerPaymentAdapter(config: PaymentAdapterConfig): void;
  
  // Option Sources — Dynamic field options
  registerOptionSource(config: OptionSourceConfig): void;
  
  // Bridge Handlers — Product entity creation
  registerBridgeHandler(handlerKey: string, handler: BridgeHandler): void;
  
  // Expression Functions — Custom formula functions
  registerExpressionFunction(name: string, fn: Function, schema: JSONSchema): void;
  
  // Export Formats — Custom export types
  registerExportFormat(format: string, exporter: ExportHandler): void;
  
  // i18n — Custom translations
  registerI18nNamespace(namespace: string, strings: LocaleStrings): void;
  
  // Themes — Custom visual themes
  registerTheme(themeKey: string, theme: FormTheme): void;
  
  // Analytics — Custom events
  registerAnalyticsEvent(eventKey: string, schema: EventSchema): void;
  
  // Webhooks — Custom events
  registerWebhookEvent(eventKey: string, schema: EventSchema): void;
}
```

### 2.4 Namespace Conventions

| Prefix | Owner | Example |
|--------|-------|---------|
| `fb.*` | FormBuilder Core | `fb.text`, `fb.email`, `fb.payment` |
| `ext.{vendor}.*` | Third-party extensions | `ext.salesforce.lookup`, `ext.hubspot.form` |

### 2.5 Extension Validation

Extensions are validated at registration time:

```typescript
// Validation rules
- field_type_key must match /^[a-z0-9_]+(\.[a-z0-9_]+)+$/
- validator_key must be unique across all modules
- action_type must be namespaced (contain at least one dot)
- theme_key must not conflict with core themes
- i18n_namespace must be unique per module
```

### 2.6 Extension Discovery

All extensions are discoverable via API:

```typescript
// List all registered field types
GET /api/ext/form-builder/field-types

// List all validators
GET /api/ext/form-builder/validators

// List all after-submit actions
GET /api/ext/form-builder/actions

// List all payment adapters
GET /api/ext/form-builder/payment-adapters

// List all option sources
GET /api/ext/form-builder/option-sources
```

### 2.7 Extension Examples by Module

**Example Product-Module Extensions:**
```typescript
// Register domain field type
registerFieldType({
  key: 'ext.product_module.custom_record_key',
  component: DomainNameField,
  validators: ['ext.product_module.record_lookup']
});

// Register bridge handler
registerBridgeHandler('product-module-bridge', {
  async execute(submission, config) {
    // Create product-owned record
  }
});
```

**Third-Party Extensions:**
```typescript
// Salesforce integration
registerOptionSource({
  key: 'ext.salesforce.contacts',
  fetcher: async (config) => {
    return await salesforceAPI.query('SELECT Id, Name FROM Contact');
  }
});

// Custom validator
registerValidator({
  key: 'ext.my_company.employee_id',
  validate: async (value) => {
    return await employeeAPI.validateId(value);
  },
  message: 'Invalid employee ID'
});
```

---

## 3. FORM DEFINITION ENTITY

A FormDefinition is the complete specification of a form — its identity, structure, fields, validation, layout, style, behaviour, and post-submission actions. It is a tree of typed nodes stored as JSONB in the database with a versioned publish history.

### 3.1 FormDefinition Schema

```typescript
interface FormDefinition {
  // Identity
  id: string;
  tenantId: string;
  slug: string;             // URL-safe, unique per tenant. e.g. 'user-login'.
                            // Uniqueness is enforced among non-deleted forms.
                            // Slugs of archived forms remain reserved until the form is deleted.
                            // System form slugs (e.g. 'auth-login') are permanently reserved.
  name: string;             // internal label shown in builder
  type: FormType;           // see §3.2
  systemKey: string | null; // 'auth.login' | 'auth.register' | null for custom

  // Status — two independent states
  // A form may simultaneously have a published live version AND an active draft.
  // These are tracked separately and never collapsed into a single enum.
  publishedStatus: 'published' | 'never-published' | 'archived';
  draftStatus: 'clean' | 'has-changes';  // 'clean' = draft matches live. 'has-changes' = unpublished edits exist.
  publishedVersion: number;  // 0 if never published
  draftVersion: number;
  publishedAt: Date | null;
  publishedBy: string | null;

  // Structure (the builder canvas)
  sections: FormSection[];  // ordered array of sections (flat form) or root sections
  steps: FormStep[] | null; // non-null when multi-step is enabled. See Section 51 for FormStep type.

  // Access & Auth
  visibility: 'public' | 'authenticated' | 'role-gated';
  allowedRoles: string[];
  enableCsrf: boolean;      // always true for system forms
  corsOrigins: CorsOrigin[]; // see 2.5 for CorsOrigin type
  ipAllowlist: string[] | null; // CIDR notation or single IPs. null = no restriction. See §13.1.

  // Behaviour
  afterSubmit: AfterSubmitConfig;
  rateLimitPolicy: RateLimitPolicy | null; // see Section 51 for RateLimitPolicy type
  dataRetentionDays: number | null; // null = keep forever; GDPR delete always overrides

  // Submission controls
  allowMultipleSubmissions: boolean; // default: true. If false, one submission per authenticated user.
  multipleSubmissionsErrorMessage: LocalisedString | null;

  // Internationalisation
  defaultLocale: string;    // e.g. 'en'. Fallback when a translation key is missing.
  supportedLocales: string[];
  localeSwitcherPosition: 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right' | 'hidden';
  // 'hidden' suppresses the switcher even if supportedLocales.length > 1.
  // Default: 'top-right'. See §50.3, §73.

  // Style
  theme: FormTheme;         // see Section 30

  // Context binding schema
  contextSchema: Record<string, 'string' | 'number' | 'boolean' | 'date'> | null;

  // Payments
  refundNotificationConfig: RefundNotificationConfig | null; // see §68. null = no refund notifications configured.

  // Product bridge
  bridgeConfig: {
    enabled: boolean;
    targetSchemaId: string; // Product-owned target schema/entity definition ID
    targetEntityType: string; // Product-owned entity type key. Example: 'client_profile'.
    fieldMappings: {
      formFieldName: string;
      targetFieldKey: string;
      transform?: 'none' | 'uppercase' | 'lowercase' | 'trim' | 'date_format';
    }[];
    productConfig: Record<string, any> | null; // product-defined extension payload; interpreted only by the consuming module
  } | null; // Cross-system bridge surface. Product-specific runtime examples are documented in consuming product specs.

  // Sharing
  shortLink: string | null;   // short URL slug. null = not generated. See §63.
  qrCodeToken: string | null; // opaque token used to generate the QR image URL. See §63.

  // Metadata
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
  etag: string;             // opaque version token used for optimistic concurrency
  submissionCount: number;  // cached
  analyticsEnabled: boolean;

  // Submit button configuration — see Section 49
  submitButton: SubmitButtonConfig;

  // Additional extension hooks for product module integration
  extensionHooks: {
    // Custom field type resolvers registered by products
    customFieldTypes?: string[];      // Registered field type plugin keys
    
    // Payment adapter for billing integration
    paymentAdapter?: string | null;   // Registered payment adapter key
  } | null;
}
```

### 3.2 Form Types

| Type | Description |
|------|-------------|
| **system** | Core application forms managed by the engine. Login, Register, Reset Password, Change Email, Payment, Profile Update, Invite Accept, 2FA Setup, Delete Account, Contact Support. Protected core fields cannot be removed but can be supplemented. |
| **custom** | Created by users with `fb.form.create` permission from scratch or from a template. No protected fields. Full CRUD. |
| **template** | Pre-built form blueprints. A template is NOT a live form — it cannot be accessed at `/forms/{slug}`, cannot receive submissions, and has no `publishedStatus`. It exists only as a blueprint. Instantiating a template creates a new `custom` FormDefinition pre-populated from the template's structure. |

### 3.3 FormSection Schema

```typescript
interface FormSection {
  id: string;
  title: LocalisedString | null;       // optional section heading; null hides heading
  description: LocalisedString | null;
  collapsed: boolean;
  columns: 1 | 2 | 3;
  columnGap: 'sm' | 'md' | 'lg';
  fields: FormField[];
  conditions: DisplayCondition[];
  style: SectionStyle;        // see Section 30
  stepId: string | null;      // which FormStep this section belongs to (multi-step only)
}

// LocalisedString — canonical definition lives in §17.2. Used wherever user-facing text appears.
// Do not redefine here. Cross-reference: §17.2.
```

### 3.4 FormField Schema

```typescript
interface FormField {
  id: string;
  type: FieldType;
  name: string;               // internal data key. Unique within form. Used in submission JSON.
  label: LocalisedString;
  placeholder: LocalisedString | null;
  helpText: LocalisedString | null;
  required: boolean;          // AUTHORITATIVE required flag. See §5.2 for interaction with ValidationRule.
  disabled: boolean;          // Static disabled. See §6.6 for interaction with disable/enable conditions.
  hidden: boolean;            // not rendered; value IS included in submission UNLESS the field is in a server-confirmed skipped step (§14.3).
  readOnly: boolean;
  defaultValue: FieldDefaultValue;
  columnSpan: 1 | 2 | 3;
  order: number;
  validation: ValidationRule[];  // see Section 31
  conditions: DisplayCondition[];
  dataBinding: DataBinding | null; // see Section 18
  systemProtected: boolean;
  style: FieldStyle;
  options: FieldOption[] | null;
  optionSource: OptionSource | null;
  analytics: FieldAnalyticsConfig;
  searchable: boolean;        // if true, field value is indexed for admin search
  sensitive: boolean;         // if true, excluded from analytics tracking
}

type FieldDefaultValue =
  | { type: 'static'; value: any }
  | { type: 'expression'; expression: string };

// FieldDefaultValue.hidden behaviour:
// For hidden fields with a dataBinding or expression default, the server ALWAYS
// recomputes the value from the binding/expression at submission time.
// Any client-submitted value for a hidden field is discarded and overwritten.
```

> **`hidden` field vs. skipped step precedence:** If a field has `hidden: true` AND it is inside a server-confirmed skipped step, the skipped-step rule takes precedence: the field's value is discarded (not included in submission). The `hidden: true` flag's "value IS submitted" rule only applies when the containing step is NOT skipped.

> **Protected field + display condition:** A system-protected field MAY have display conditions that hide it from the user's view. However, hiding a protected field does not remove it from validation — the server will still require its value unless the condition is also evaluated server-side and the field is genuinely inapplicable. Admins receive a build warning when a hide condition is applied to a protected field.

### 3.5 CorsOrigin Type

```typescript
interface CorsOrigin {
  origin: string;
  // Exact origin string. Format: scheme + host + optional port.
  // Examples: 'https://example.com', 'https://app.example.com', 'https://example.com:8080'
  // Wildcards are NOT supported. Each origin must be fully specified.
  // Paths are NOT included — only scheme + host + port is matched.
}
```

---

## 4. FORM BUILDER CANVAS

### 4.1 Canvas Layout

| Panel | Description |
|-------|-------------|
| **Left Panel — Field Palette** | All available field types grouped by category: Basic, Selection, Date/Time, File, Layout, Advanced, Payment. Drag any field onto the canvas. |
| **Centre — Canvas** | Live form preview. Sections stacked vertically. Fields in their column grid. Drag handle on each field. Drop zones highlighted on drag. |
| **Right Panel — Properties** | Context-sensitive. No selection: form-level properties. Section selected: section properties. Field selected: field properties, validation, conditions, style. |
| **Top Bar — Toolbar** | Form name (editable), Status badge (see §15.4), Version indicator, Undo/Redo, Preview, Test, Save Draft, Publish buttons, Settings (gear icon), More (kebab menu: Duplicate Form, Save as Template, Archive, Delete). Build Warnings badge (count of active warnings). |
| **Bottom Bar — Status** | Last saved timestamp, warnings count (same count as toolbar badge), unsaved changes indicator, edit-lock status. |

### 4.2 Canvas States

| State | Description |
|-------|-------------|
| **idle** | Form loaded. No element selected. |
| **field-dragging** | Field being dragged. Valid drop zones highlighted green. |
| **field-dropping** | Field released on drop zone. Insertion animation plays. |
| **section-selected** | Section highlighted with blue outline. Section properties shown. |
| **field-selected** | Field highlighted. Field properties panel shown. |
| **multi-selected** | Cmd/Ctrl+Click selects multiple fields. Bulk actions: move to section, duplicate, delete. |
| **resizing-column** | User drags column divider. Snaps to 1/3, 1/2, 2/3 ratios. |
| **unsaved** | Unsaved changes. Toolbar amber dot. Navigation guard active. |
| **autosaving** | 30s autosave in progress. Sync icon in status bar. |
| **saving** | Manual save in progress. Canvas non-interactive. |
| **saved** | Save complete. Timestamp updated. |
| **publishing** | Publish in progress. |
| **published** | `publishedStatus` → 'published'. `draftStatus` → 'clean'. |
| **error-save** | Save failed. Retry. Draft preserved in localStorage as emergency backup. |
| **system-form** | Editing a system form. Protected fields show lock icon. |
| **lock-held** | Current user holds the edit lock. Green padlock in bottom bar. |
| **lock-held-by-other** | Another editor holds the lock. Canvas read-only. Banner: 'Jane is editing this form.' |
| **lock-requested** | Current user sent a lock request. Waiting for response. |
| **lock-request-received** | Toast to active editor: 'Bob has requested edit access. Grant / Deny.' |
| **lock-transferred** | Lock transferred. Previous editor's canvas enters read-only with toast. |

### 4.3 Section Management

**Add Section:** User clicks '+ Add Section' below the last section, or between sections via inter-section drop zone. New section: 1 column, no title, no fields. Immediately selected.

**Section Properties Panel:**

| Element / Field | States | Interconnects |
|-----------------|--------|---------------|
| Section Title | idle, focused, filled, empty | i18n_BRANCH |
| Section Description | idle, focused, rich-text editor | i18n_BRANCH |
| Column Layout | 1 / 2 / 3 toggle. Live preview. | Warns if field has columnSpan > new column count |
| Column Gap | sm / md / lg | None |
| Collapsible | toggle. If ON: collapse-by-default appears | None |
| Background | none / colour picker / preset | THEME_BRANCH |
| Border | none / all-sides / left-accent / rounded | None |
| Padding | xs / sm / md / lg / xl | None |
| Conditions | show/hide section based on field values | CONDITION_BRANCH |
| Duplicate Section | confirm-modal | Creates deep copy with new IDs |
| Delete Section | confirm-modal | Blocked if contains system-protected fields |

### 4.4 Field Drag & Drop

**From Palette to Canvas:** Drag field type → valid drop zones highlight → drop → default properties applied → field selected.

**Reordering Existing Fields:** Grab drag handle (left edge, on hover). Drag within same section (reorder) or to different section (move). If target section has fewer columns than field's `columnSpan`, `columnSpan` auto-adjusted with toast.

**Column Span:** In a 2-column section: 1 or 2. In 3-column: 1, 2, or 3. Set in properties panel. Dragging wide field into narrower section auto-narrows with toast: 'Column span adjusted from 3 to 2.'

---

## 5. FIELD TYPE CATALOGUE

### 5.1 Basic Fields

| Field Type | Data Type | States | Validation / Branches |
|------------|-----------|--------|----------------------|
| Text Input | string | idle, focused, filled, valid, invalid, disabled, read-only, loading | min/max length, regex, email, URL, no-whitespace, custom |
| Textarea | string | idle, focused, filled, at-limit, disabled | min/max length, char counter |
| Rich Text Editor | HTML/md | idle, focused, toolbar-active, fullscreen, loading | max length, tag allowlist, sanitised on submit |
| Number | number | idle, focused, filled, out-of-range, disabled | min, max, step, integer-only, decimal places, currency format |
| Password | string | idle, focused, filled, visible-toggle, strength meter, confirm-mismatch | Intended for auth/security system flows only. Values are operational secrets and are never persisted in standard submission records. |
| Hidden | any | not rendered | Value set by static default, data binding, URL param, or expression. Client-submitted value always discarded and overwritten server-side. |

### 5.2 Selection Fields

| Field Type | Data Type | States | Validation / Branches |
|------------|-----------|--------|----------------------|
| Dropdown / Select | string \| string[] | idle, open, searching, loading-options, selected, cleared, disabled | required, multi-select, option source: static/API/binding. See §32. |
| Radio Group | string | idle, option-hover, option-selected, disabled, error | required, layout: vertical/horizontal/grid |
| Checkbox | boolean | unchecked, checked, indeterminate, disabled | required (must be checked) |
| Checkbox Group | string[] | idle, option-hover, option-checked, min/max-selection | min/max checked, layout |
| Toggle / Switch | boolean | off, on, disabled, loading | required (must be on) |
| Button Group | string | idle, option-hover, option-active, disabled | Exclusive. Max 8 options. |
| Combobox | string | idle, focused, typing, dropdown-open, selected, no-options, loading | Allow custom value, min chars to trigger |

### 5.3 Date & Time Fields

| Field Type | Data Type | States | Validation / Branches |
|------------|-----------|--------|----------------------|
| Date Picker | ISO date string | idle, open, day-hover, selected, month-nav, year-nav, disabled-date, out-of-range | min/max date, disabled specific dates, disabled days-of-week |
| Time Picker | HH:MM string | idle, open, hour-select, minute-select, am/pm-select | min/max time, step (e.g. 15 min), 12h/24h |
| DateTime Picker | ISO datetime string | combines Date + Time states | all Date and Time validations |
| Date Range | { start: ISODate, end: ISODate } | idle, selecting-start, selecting-end, range-hover, range-set, invalid-range | min/max range duration, start cannot be after end |

**Date Range serialisation:** Stored as `{ "start": "2026-03-01", "end": "2026-03-15" }`. In CSV exports: `{fieldName}_start` and `{fieldName}_end` sub-columns.

### 5.4 File Fields

| Field Type | Data Type | States | Validation / Branches |
|------------|-----------|--------|----------------------|
| File Upload | StoredFile[] | idle, drag-over, validating, uploading(progress%), scanning, ready, error, quota-exceeded | allowed MIME types, max file size, max count, virus scan. See §23. |
| Image Upload | StoredFile | idle, drag-over, uploading, cropping, preview, error | File Upload + crop aspect ratio, min dimensions, auto-compress. See §23. |
| Avatar Upload | StoredFile | no-avatar(initials), uploading, cropping(circle), preview, error | Forced circular crop. See §23. |
| Signature | base64 PNG | idle(blank canvas), drawing, completed, cleared | Required: must draw. Min strokes. Output resolution. |

### 5.5 Layout & Structural Fields

Not data-collecting. Presentational only. `data` key is `null` in submission record.

| Field Type | States | Notes |
|------------|--------|-------|
| Section Divider | visible, hidden(condition) | Visual separator. No data. |
| Heading | visible, hidden(condition) | LocalisedString, H2/H3/H4, alignment. |
| Paragraph / Copy | visible, hidden(condition) | Rich text. Instructions, terms. |
| Spacer | visible | Height: xs/sm/md/lg/xl. |
| Alert / Banner | info, warning, error, success — condition-driven | LocalisedString + icon. |
| Progress Bar | rendered when multi-step active | Auto-calculated. Shows step N of M. |

### 5.6 Advanced & Special Fields

| Field Type | Data Type | States | Validation / Branches |
|------------|-----------|--------|----------------------|
| OTP / Code Input | string | idle, focused(each cell), filled, invalid, expired, resend-available | Length (default 6), allowed chars, expiry timer, resend cooldown — EMAIL_BRANCH or SMS_BRANCH |
| Phone Number | E.164 string | idle, focused, country-select, filled, invalid | Country picker, format per country |
| Address | AddressObject | sub-field states | Country drives state/province. Postcode format per country. See §46. |
| Rating | number 1–N | idle, hover-N, selected-N, cleared, read-only | Max stars (default 5), half stars, allow clear |
| Slider / Range | number | idle, dragging, at-min, at-max, disabled | min, max, step, tooltip, tick marks |
| Colour Picker | hex string | idle, open, picking, selected | Preset swatches, custom hex, transparency |
| Autocomplete | string | idle, focused, typing(debounce), loading, results, no-results, selected | Source: OptionSource (§32.3) with SSRF protection (§61). Server-side proxied; client never calls source URL directly. See §71 for full Autocomplete source specification. |
| Repeater | object[] | empty(+add), N-rows, adding, removing, reordering | Min/max rows, sub-form per row. See §26. |
| Consent / Terms | boolean | unchecked, checked, link-clicked, required-error | Required: cannot submit without checking. Configurable link URL. See §64 for full specification. |
| Captcha | token | idle, solving, solved, expired, error | captcha.verify MCP. See §27. |
| Payment | PaymentResult | idle, method-selecting, card-entry, processing, succeeded, failed, requires-auth(3DS) | Embedded flows use `payment.create_intent` + `payment.confirm_intent`; hosted redirect flows use `payment.create_checkout`; async settlement uses `payment.verify_webhook`. See §9 and §47. |

### 5.7 Custom Field Type Extension Point

Product modules can register custom field types that integrate with the Form Builder canvas, validation, and submission systems.

**Field Type Registration:**

```typescript
// Product module registers custom field type
registerFieldType({
  key: 'ext.my_module.sku_selector',
  displayName: 'SKU Selector',
  category: 'Product',
  description: 'Select product SKU with live inventory check',
  
  // React component for rendering
  component: 'ExtSkuSelectorField',
  
  // Server-side validation
  validator: 'ext.my_module.sku_valid',
  
  // Data type in submission
  dataType: 'object',
  dataSchema: {
    type: 'object',
    properties: {
      sku: { type: 'string' },
      name: { type: 'string' },
      price: { type: 'number' },
      quantity: { type: 'integer' }
    }
  },
  
  // Properties panel configuration
  properties: [
    {
      key: 'product_category',
      type: 'select',
      label: 'Product Category',
      options: ['electronics', 'clothing', 'food']
    },
    {
      key: 'show_inventory',
      type: 'boolean',
      label: 'Show Inventory Count'
    }
  ],
  
  // Conditional logic support
  supportsConditions: true,
  
  // Search indexing
  searchable: true,
  searchFields: ['sku', 'name']
});
```

**Field Component Interface:**

```typescript
interface CustomFieldComponentProps {
  field: FormField;
  value: any;
  onChange: (value: any) => void;
  onBlur: () => void;
  error?: string;
  disabled?: boolean;
  readOnly?: boolean;
  context: {
    tenantId: string;
    formId: string;
    submissionId?: string;
    locale: string;
  };
}

// Example custom field component
const ExtSkuSelectorField: React.FC<CustomFieldComponentProps> = ({
  field, value, onChange, error, context
}) => {
  const { product_category, show_inventory } = field.config;
  
  return (
    <SkuSelector
      category={product_category}
      showInventory={show_inventory}
      value={value}
      onChange={onChange}
      tenantId={context.tenantId}
      error={error}
    />
  );
};
```

**Server-Side Validation:**

```typescript
// Register validator for custom field
validatorRegistry.register({
  key: 'ext.my_module.sku_valid',
  validate: async (value, config, context) => {
    const { sku, quantity } = value;
    
    // Check SKU exists
    const product = await productAPI.get(sku);
    if (!product) {
      return { valid: false, message: 'SKU not found' };
    }
    
    // Check inventory
    if (quantity > product.inventory) {
      return { 
        valid: false, 
        message: `Only ${product.inventory} units available` 
      };
    }
    
    return { valid: true };
  }
});
```

**Field Type Discovery:**

```typescript
// List all field types including custom
GET /api/ext/form-builder/field-types

// Response
{
  "core": [
    { "key": "fb.text", "displayName": "Text Input", "category": "Basic" },
    { "key": "fb.payment", "displayName": "Payment", "category": "Advanced" }
  ],
  "extensions": [
    {
      "key": "ext.my_module.sku_selector",
      "displayName": "SKU Selector",
      "category": "Product",
      "module": "my_module"
    }
  ]
}
```

---

## 6. FIELD PROPERTIES PANEL

### 6.1 General Tab

| Element / Field | States | Interconnects |
|-----------------|--------|---------------|
| Label | idle, focused, filled, empty-warning | i18n_BRANCH |
| Field Name (key) | idle, focused, filled, duplicate-error, regex-invalid | Auto-generated from label. Cannot change on system-protected fields. |
| Placeholder | idle, focused, filled | i18n_BRANCH |
| Help Text | idle, focused, filled | Markdown supported. i18n_BRANCH |
| Required toggle | off, on | Sets `FormField.required`. See §5.2. |
| Disabled toggle | off, on | Sets `FormField.disabled`. See §6.6. |
| Read-Only toggle | off, on | Value submitted but not editable. |
| Hidden toggle | off, on | Not rendered; value submitted (with caveats in §2.4). |
| Default Value | idle, filled, expression-mode | Static or expression. See §33. |
| Column Span | 1/2/3 | Conflict state if section resized. |
| Data Binding | none, entity-field dropdown | See §18. |
| Searchable | toggle | Not available for: File, Image, Signature, Payment, CAPTCHA. |
| Sensitive | toggle | Auto-on for Password, Payment card fields. |

### 6.2 Validation Tab — Required Flag Interaction

**Authoritative rule:** `FormField.required: boolean` and a `ValidationRule` of `type: 'required'` in the `validation[]` array are two mechanisms for expressing the same constraint. The complete precedence matrix:

**Precedence and conflict resolution:**

| Scenario | Effective behaviour |
|----------|-------------------|
| `required: true`, no required rule | Field is required. |
| `required: false`, no required rule | Field is not required. |
| `required: false`, required rule with `enabled: true` | Field IS required. The enabled ValidationRule takes effect even when the field property is false. |
| `required: false`, required rule with `enabled: false` | Field is NOT required. Disabled rule has no effect. |
| `required: true`, required rule with `enabled: false` | Field IS required. `required: true` property wins. The disabled rule does not override the property. |
| `required: true`, required rule with `enabled: true` | Field is required. No conflict (additive, same outcome). |

**Summary:** an enabled `required` ValidationRule always makes the field required regardless of the field property. The field property `required: true` cannot be negated by a disabled ValidationRule. Both mechanisms are authoritative when enabled; the field property takes precedence only when it is `true` and the rule is disabled.

**Builder guidance:** Adding a `required` ValidationRule to a field that already has `required: true` triggers a builder info notice: "This field's 'Required' toggle already enforces this rule. The validation rule is redundant but harmless."

**Validation rules in the tab:**

| Rule Type | Description |
|-----------|-------------|
| required | Must have a non-empty value. |
| min / max | String: char count. Number: value. Date: date. |
| pattern (regex) | Must match regex. Admin enters regex + test value. |
| email | Standard email format. Optional domain whitelist. |
| url | Valid URL. Configurable protocols. |
| phone | E.164 format per country. |
| unique (async) | Checked against a server-side endpoint on blur. See §53. |
| match (equality) | Must equal another field's value. |
| custom (server) | Admin-specified endpoint. See §53. |
| ai-classify (MCP) | llm.classify. Admin defines allowed categories and rejection message. |

See **Section 31** for full `ValidationRule` schema.

**Validation Execution Order:**

| Trigger | Rules Applied |
|---------|---------------|
| real-time (keypress, debounced 500ms) | pattern, min, max, email, url |
| on-blur | required, unique (async), match |
| on-submit | All rules in parallel |
| server-side | All rules re-evaluated. Server is authoritative. |

### 6.3 Style Tab

| Element | Description |
|---------|-------------|
| Width | full, 1/2, 1/3, 1/4, auto, custom px/% |
| Label Position | top (default), left (inline), hidden |
| Label Style | font size, weight, colour (from theme tokens) |
| Input Size | sm / md / lg |
| Border Style | default, none, underline-only, rounded, pill |
| Focus Colour | theme primary (default) or custom. A11y warning on low contrast. |
| Error Colour | theme danger (default) or custom. A11y warning on low contrast. |
| Icon | prefix, suffix icon |
| Custom CSS | restricted to `system_owner` by default; may be delegated to `super_admin`; scoped, sanitised. RBAC: form.field.customCss |

---

## 7. CONDITIONAL LOGIC ENGINE

### 7.1 Condition Rule Schema

```typescript
interface DisplayCondition {
  action: 'show' | 'hide' | 'require' | 'disable' | 'enable' | 'set-value';
  logic: 'all' | 'any';
  conditions: ConditionClause[];
  setValue?: SetValueConfig; // only for set-value action
}

// ConditionGroup — canonical definition is in §51.5.
// Used in AfterSubmitAction.condition and DataBinding.writeBackCondition.
// Evaluates to true or false against submitted data WITHOUT an 'action' field.
// Do not redefine here. Cross-reference: §51.5.

interface SetValueConfig {
  value: any;
  expression?: string; // see §33
  // Server applies set-value independently; client-submitted value is discarded.
}

interface ConditionClause {
  subject: ConditionSubject;
  operator: ConditionOperator;
  value: any;
}

type ConditionSubject =
  | { type: 'field'; fieldName: string }
  | { type: 'user'; property: string }
  | { type: 'url-param'; name: string }
  | { type: 'date'; property: 'today' | 'time' }
  | { type: 'tenant'; property: string };

type ConditionOperator =
  'equals' | 'not_equals' | 'contains' | 'not_contains' |
  'starts_with' | 'ends_with' | 'is_empty' | 'is_not_empty' |
  'greater_than' | 'less_than' | 'in' | 'not_in' | 'matches_regex';
```

### 7.2 Conflict Resolution — Contradictory Conditions

1. `hide` beats `show`
2. `disable` beats `enable`
3. `require` is additive — field is required if ANY require condition is met

Enforced identically on client and server.

### 7.3 set-value on Hidden / Multi-Step Fields

A `set-value` condition may target a field in a different section or an unrendered step. Evaluated in full server-side at submission. On client, applied as soon as the target field's section is mounted. If target is `hidden: true`, the server applies the set-value regardless of client state.

### 7.4 Condition Builder UI

1. Select field/section.
2. Conditions tab → '+ Add Condition'.
3. Choose action.
4. Choose logic: All / Any.
5. Add clause(s): Subject → Operator → Value.
6. Live preview updates canvas instantly.

| State | Description |
|-------|-------------|
| idle | No conditions. Field always visible. |
| has-conditions | Blue 'C' badge on field. |
| condition-met | During preview/test: condition active. |
| circular-dep | Circular reference detected. Cannot publish until resolved. |
| orphaned-ref | References a deleted field. Warning badge. |
| conflict | Contradictory actions. Warning shown. Publishing allowed. |

### 7.5 Condition Use-Case Examples

| Scenario | Configuration |
|----------|---------------|
| Show 'Company Name' only if 'Account Type' = 'Business' | action: show, field:accountType equals 'business' |
| Require 'Tax ID' if country = 'US' AND amount > 600 | action: require, logic: all, [country equals 'US', amount greater_than 600] |
| Show warning banner if user selects deprecated option | action: show on Alert field, field:plan equals 'legacy' |
| Show 'Referral Code' if URL has ?ref= | action: show, url-param:ref is_not_empty |
| Auto-set hidden 'leadSource' from URL param | action: set-value, url-param:utm_source is_not_empty → setValue.expression: '{{query.utm_source}}' |

### 7.6 Static `disabled` Property vs. Condition Precedence

`FormField.disabled: boolean` is the static, always-on disabled state. Conditions can also disable/enable a field. Precedence:

| Scenario | Effective State |
|----------|----------------|
| `disabled: true`, no conditions | Always disabled. |
| `disabled: false`, `disable` condition fires | Disabled by condition. |
| `disabled: true`, `enable` condition fires | The `enable` condition overrides the static `disabled: true`. The field becomes enabled when the enable condition is met. |
| `disabled: false`, both `disable` and `enable` conditions fire | `disable` wins (§6.2 Rule 2). |

---

## 8. SYSTEM FORMS — PROTECTED & CONFIGURABLE

### 8.1 System Form Registry

| System Key | Protected Fields | Extensible Fields |
|------------|------------------|-------------------|
| auth.login | Email, Password | Remember Me, CAPTCHA, SSO buttons, Announcement banner |
| auth.register | Email, Password, Confirm Password | Name, Phone, Terms Consent, Marketing Opt-in |
| auth.forgot-password | Email | Customer message, CAPTCHA |
| auth.reset-password | New Password, Confirm Password | Custom copy, terms reminder |
| auth.verify-email | (layout-only) | Instructional copy, help link. Added data fields are stored as supplementary submission data but do not affect verification flow. |
| auth.2fa-setup | QR Code, OTP input, Backup codes | Instructions, help link |
| auth.2fa-verify | OTP Input | Help text, alternative method link |
| profile.update | First Name, Last Name, Display Name | Custom profile fields, sections |
| profile.change-email | New Email, Current Password | Instructions |
| profile.delete-account | Email confirmation input | Reason, CSAT, feedback |
| billing.checkout | Plan selector, Payment field | Promo code, billing address, VAT number |
| billing.update-payment | Payment field | Billing address |
| support.contact | Subject, Category, Message | Priority, custom fields, file attachment |
| user.invite-accept | Invite display, Password | Profile fields to complete |

### 8.2 Protected Field Rules

| Rule | Description |
|------|-------------|
| Cannot delete | No delete button on protected fields. |
| Cannot rename key | `name` property is locked. |
| Can reorder | Can be dragged to different position. |
| Can restyle | Colour, size, border, label text — all editable. |
| Can change label/help | Label text and help text are editable. |
| Can move to any section | Protected fields can move between sections. |
| Can have conditions | With caveat: hiding a protected field triggers a build warning. Server validates the field unless the hide condition is server-confirmed. |

### 8.3 Editing a System Form

1. Admin navigates to Settings > Forms > System Forms.
2. Selects form. Builder opens current published version. Protected fields show lock badge.
3. Admin edits. Preview shows changes.
4. Admin saves draft, then publishes. New version goes live for all tenant users.

**Audit:** `fb.form.system.modified` logged with: formKey, version, changeSet, actor.

### 8.4 support.contact Form — Server-Side Handler

On submission:
1. Support ticket created with subject, category, message, priority, attached files.
2. Ticket assigned auto-incremented ticket number.
3. Confirmation email sent to submitter with ticket number.
4. Admin notification sent to configured support inbox or webhook.
5. If Support module disabled/unavailable: submission stored as standard form submission. Admin notified that ticket creation failed. Can be re-processed when module is restored.

---

## 9. POST-SUBMISSION ACTIONS

### 9.1 AfterSubmitConfig Schema

```typescript
interface AfterSubmitConfig {
  actions: AfterSubmitAction[];
}

interface AfterSubmitAction {
  type: AfterSubmitActionType;
  config: object;               // type-specific config — see per-action sections
  condition: ConditionGroup | null; // fires only if condition evaluates true against submission data
  // ConditionGroup canonical definition: §51.5. Used here — NOT DisplayCondition — because there is no 'action' field.
  // Evaluated against final submission field values AFTER all set-value rules have been applied.
  // Action N's condition CAN reference values set by a prior set-value action in the same submission.
  delay: number;                // milliseconds before action fires (top-level, not inside config)
}

type AfterSubmitActionType =
  | 'show-message'
  | 'show-modal'
  | 'show-popup'
  | 'redirect'
  | 'redirect-back'
  | 'reset-form'
  | 'confetti'
  | 'trigger-email'
  | 'trigger-webhook'
  | 'trigger-api'
  | 'show-next-step';
```

### 9.2 show-message

Replaces form area with inline success message.

- **Config:** `message: LocalisedString` (rich text, variable injection), `icon: 'none'|'check'|'star'|'heart'|'custom'`, `showDuration: number` (ms; 0 = permanent), `style: 'success'|'info'|'custom'`
- **A11y:** `role="status"`, `aria-live="polite"` for success. `role="alert"` for errors.

### 9.3 show-modal

Displays confirmation in a modal. Form remains in background.

- **Config:** `title: LocalisedString`, `body: LocalisedString`, `size: 'sm'|'md'|'lg'|'fullscreen'`, `closable: boolean`, `primaryCTA: CTAButton | null`, `secondaryCTA: CTAButton | null`, `autoCloseMs: number | null`

### 9.4 show-popup

Floating toast notification.

- **Config:** `position: 'top-left'|'top-center'|'top-right'|'bottom-left'|'bottom-center'|'bottom-right'`, `message: LocalisedString`, `type: 'success'|'error'|'info'|'warning'`, `duration: number`, `stacking: boolean`

### 9.5 redirect

| Config field | Description |
|--------------|-------------|
| `url` | Absolute URL or relative path. `{{ }}` variable injection supported (§33). |
| `target` | `'self'` or `'blank'`. |
| `method` | `'GET'` (default) or `'POST'`. |

`delay` is top-level on `AfterSubmitAction`, not inside config.

**Security validation:** relative internal paths are preferred. Absolute URLs are permitted only when the destination host matches an admin-maintained allowlist for the tenant/product deployment. The runtime MUST reject `javascript:`, `data:`, protocol-relative (`//...`) URLs, credential-bearing URLs, control-character injection, and malformed or double-encoded redirect targets. Final redirect destinations are revalidated server-side after expression resolution, not trusted from the client.

### 9.6 redirect-back

- **Config:** `fallbackUrl: string` — if no history entry exists and `fallbackUrl` is not set: stays on form URL.
- **Embedded forms:** Posts `{ type: 'form:redirect-back' }` message to parent window. Parent handles navigation.

### 9.7 reset-form

Clears all field values. Returns to initial empty state. If save-and-resume is enabled: server-side draft for this session is also deleted.

### 9.8 confetti

- **Config:** `duration: number` (ms, default 3000), `colors: string[]` (hex; defaults to theme primary palette), `spread: 'wide'|'normal'|'narrow'` (default: 'normal'), `origin: { x: number, y: number }` (0–1 coords; default: `{ x: 0.5, y: 0.3 }`)
- **A11y:** Container has `aria-hidden="true"`. Not animated if `prefers-reduced-motion` is set.
- Does not block subsequent actions.

### 9.9 trigger-email

- **Config:** See `TriggerEmailConfig` in §40.5.
- See **Section 40** for full template system.
- The `subject` always comes from the referenced `FormEmailTemplate.subject`. There is no subject override field on `TriggerEmailConfig`.

### 9.10 trigger-webhook

- **Config:**
```typescript
interface TriggerWebhookConfig {
  webhookId: string;  // ID of a registered WebhookEndpoint (see §41)
}
```
- Only `active` webhook endpoints can be targeted. Selecting a `failing` or `disabled` webhook in the builder shows a warning.
- Multiple `trigger-webhook` actions can reference different webhook IDs — each fires independently.
- **Retry policy:** Non-2xx or timeout: exponential backoff, 3 attempts over 5 minutes. All retries fail: `processingStatus` → `'failed'` or `'partial'`. Admin notified.
- **Timeout:** 10 seconds per attempt.
- See **Section 41** for Webhook Registry.

### 9.11 trigger-api

See **Section 28** for complete specification.

### 9.12 show-next-step

Advances a multi-step form to the next step programmatically — typically fired after a successful async validation or payment action mid-form.

- **Config:** `{ targetStepId?: string }` — if `targetStepId` is omitted, advances to the immediately next step (accounting for step-skip conditions). If `targetStepId` is specified, jumps to that step directly (skipping intermediate steps; skipped steps' fields are treated as skipped per §14.3).
- **Use case:** After an embedded async check on step 2 succeeds, trigger `show-next-step` to advance to step 3 without the user clicking Next.
- **Only valid** when the form is multi-step (`steps` is non-null). If used on a flat form: **hard publish block** — "show-next-step action is not valid on a flat form. Remove this action or enable multi-step." This is a hard block (not a soft warning) because there is no valid use case for this action on a flat form.

### 9.13 Multi-Action Sequences

Actions execute in declared order. Each has a `delay` and optional `condition`. Visual and navigation actions (show-message, show-modal, redirect) execute immediately. Background actions (trigger-email, trigger-webhook, trigger-api) execute asynchronously. A failing background action does not block subsequent actions in the sequence.

```json
{
  "actions": [
    { "type": "show-message", "delay": 0, "condition": null, "config": { "message": { "default": "Welcome, {{field.firstName}}!" }, "showDuration": 2000 } },
    { "type": "confetti", "delay": 0, "condition": null, "config": { "duration": 3000 } },
    { "type": "trigger-email", "delay": 0, "condition": null, "config": { "to": "submitter", "templateSlug": "form-confirmation" } },
    { "type": "trigger-email", "delay": 0, "condition": null, "config": { "to": "admin", "templateSlug": "form-admin-notification" } },
    { "type": "redirect", "delay": 2000, "condition": null, "config": { "url": "/onboarding", "target": "self" } }
  ]
}
```

---

## 10. PAYMENT FIELD — FULL SPECIFICATION

### 10.0 Payment Integration Modes

The payment field supports two integration modes selected by form configuration and the active capability binding:

| Mode | When Used | Capabilities |
|------|-----------|-------------|
| **embedded** | Inline card / wallet UI rendered inside the form | `payment.create_intent`, `payment.confirm_intent`, optional `payment.method.save`, `payment.verify_webhook` |
| **hosted** | Redirect-based provider checkout experience | `payment.create_checkout`, `payment.verify_webhook` |

**Authoritative rule:** the form runtime must not assume one hardcoded provider flow. The active payment mode is determined by the configured field behavior plus the capabilities exposed by the bound payment connector.

### 10.1 Payment Field Configuration

| Element / Field | States | Interconnects |
|-----------------|--------|---------------|
| Amount Source | fixed / field-derived / formula | Formula uses §33 expression engine. |
| Currency | from tenant billing config, or field-selectable | PAYMENT_BRANCH |
| Payment Methods | card / bank-transfer / wallet | Embedded mode: `payment.create_intent` / `payment.confirm_intent`; Hosted mode: `payment.create_checkout` |
| Billing Address | none / optional / required | If required: address sub-form shown |
| Save Card | off / optional / required | Optional `payment.method.save` capability when the active provider supports vaulting |
| 3D Secure | auto / always / never | MCP handles 3DS challenge |

### 10.2 Payment Field States

| State | Description |
|-------|-------------|
| idle | Payment UI not shown. Order summary visible if amount calculable. |
| method-selecting | User selects payment method. |
| card-entry | Card input rendered. |
| card-valid | All card fields valid. Submit enabled. |
| card-invalid | Luhn fail / expired / CVC wrong length. |
| wallet-pending | Apple Pay / Google Pay sheet shown. |
| processing | Payment submitted. Submit button disabled. |
| 3ds-pending | 3DS auth required. |
| succeeded | Payment confirmed. After-submit actions fire. |
| failed | Declined. Retry available. |
| requires-action | Further provider action needed. |
| provider-error | MCP payment provider error. |

### 10.3 Amount Formula Engine

```javascript
// Examples
formula: '{{field.quantity}} * {{field.planPrice}} + {{field.shippingCost}}'
formula: '{{field.plan}} === "pro" ? 29.00 : 9.00'

// Repeater field aggregation (see §69 for full specification)
formula: 'sum(field.lineItems, "quantity") * {{field.unitPrice}}'
formula: 'sum(field.orderRows, "subtotal")'
```

Amount is always **re-calculated server-side** on submission. Client amount is display-only.

**Formula failure:** If a referenced field value is missing or non-numeric (e.g. hidden field): HTTP 422 with `{ field: 'payment', code: 'formula_evaluation_failed', detail: '...' }`. Submission not stored.

**Formula evaluation context:** Only values from fields NOT hidden server-side are used. Hidden/skipped fields are treated as `null`.

**Repeater fields in formulas:** Repeater fields serialize as `object[]`. Direct access (`{{field.lineItems}}`) is not valid in a payment formula. Use the built-in `sum(field, subFieldName)` aggregation function. Other arithmetic on Repeater sub-fields must use supported aggregation functions. See §69 for full specification, and §33 for the complete expression grammar.

**Hard block at publish:** If a payment formula references a Repeater field name without using a supported aggregation function (e.g. uses `{{field.lineItems}}` directly), the pre-flight check raises a hard block: "Payment formula directly references a Repeater field. Use sum() or another aggregation function. See §69."

---

## 11. FORM TEMPLATES

### 11.1 Built-in Template Library

| Template | Description |
|----------|-------------|
| Login | Email, Password. CAPTCHA. SSO buttons. Remember Me. |
| Register | Name, Email, Password, Confirm Password, Terms Consent. |
| Reset Password | Email. CAPTCHA. |
| New Password | Password + Confirm Password (strength meter). |
| Contact Us | Name, Email, Subject, Message, File Attachment. |
| Event Registration | Name, Email, Phone, Ticket Type, Guests, Dietary. Payment field. |
| Simple Survey | Title, Rating, Feedback, NPS. |
| Lead Capture | Name, Email, Company, Role, Use Case, How did you hear. |
| Payment / Checkout | Plan selector, Promo Code, Billing Address, Payment field. |
| Profile Update | Avatar Upload, Name, Display Name, Bio, Phone, Website. |
| Feedback Widget | Sentiment, Comments, Page URL (hidden). |
| Job Application | Name, Email, Phone, LinkedIn, CV Upload, Cover Letter. |
| Waitlist | Email, Name, Referral Code (optional). |
| Subscription Cancellation | Reason, Comments (conditional), 'Are you sure?' heading. |

### 11.2 Template States

| State | Description |
|-------|-------------|
| browsing | Gallery. Grid of cards. Category filters. Name search. |
| previewing | Preview modal. 'Use this template' CTA. |
| instantiating | User clicks 'Use Template'. Name input. |
| instantiated | New `custom` FormDefinition created as `draft`. Builder opens. |
| instantiate-failed | Error shown. Retry. |

### 11.3 Custom Templates

1. Admin opens a form in builder.
2. Toolbar > More > 'Save as Template'.
3. Enter template name and description.
4. Template appears in gallery under 'My Templates' (tenant-scoped).

RBAC: form.template.create, form.template.delete.

---

## 12. PREVIEW & TEST RUNNER

### 12.1 Preview Mode

**Access:** Builder toolbar → 'Preview'. Shortcut: Cmd/Ctrl+Shift+P.

| Feature | Description |
|---------|-------------|
| device-preview | Desktop / Tablet / Mobile. Canvas resizes. |
| locale-preview | Language dropdown. Switches to selected locale. Falls back to defaultLocale for missing translations. |
| theme-preview | Light / Dark / High Contrast toggle. |
| user-context | Simulate as: anonymous / authenticated user / specific user. DataBinding fields pre-filled from simulated user context. |
| condition-preview | Manually set field values to trigger conditional logic. |
| interactive | Fields fully interactive. |
| not-submittable | Submit button shows 'Preview Only'. Submission is disabled. |

### 12.2 Test Run

A real end-to-end submission in sandbox mode. No data affects live records.

**Access:** Builder toolbar → 'Test'.

**Sandbox requirement:** Payment fields require a sandbox payment key configured in Settings > Billing > Sandbox. If not configured: warning shown — payment action skipped during test run.

**Execution:**
- Form validates test data client-side.
- Test submission record created with `type: 'test'`.
- After-submit actions execute in sandbox:
  - show-message/modal/popup: renders in preview panel
  - redirect: shows destination URL, does NOT redirect
  - trigger-email: sends to admin test inbox only
  - trigger-webhook: sends to `sandboxWebhookUrl` (§13.6) or logs payload if none configured
  - trigger-api: executes with `X-Form-Test-Run: true` header; response logged, not acted on
  - payment: uses sandbox payment credentials

| State | Description |
|-------|-------------|
| configuring | Test Run modal. Data input. |
| validating | Client-side validation on test data. |
| running | Submission in progress. Step-by-step status shown. |
| completed | All actions completed. Report ready. |
| failed | One or more actions failed. |
| partial | Some succeeded, some failed. |

### 12.3 Test Run Report

- **Submission Record:** ID, timestamp, all field values, validation result
- **Action Results:** Per action: type, config summary, execution status, response data, duration
- **Email Preview:** Rendered HTML email in iframe
- **Webhook Log:** Payload, HTTP response code and body
- **API Call Log:** URL, method, headers (redacted), payload, response code and body
- **Condition Trace:** Which conditions evaluated, their values, which fired
- **Validation Report:** Per field: rules ran, passed/failed
- **Performance:** Form load time, submission time, action execution times
- **Warnings:** A11y, missing translations, orphaned conditions, conflicts, sandbox config gaps

---

## 13. SUBMISSION MANAGEMENT

### 13.1 Submission Entity

```typescript
interface FormSubmission {
  id: string;
  formId: string;
  formVersion: number;
  formSnapshot: FormDefinition; // full deep-copy at time of submission
  tenantId: string;
  submittedBy: string | null;
  submittedAt: Date;
  ipAddress: string | null;
  // IP handling: stored as full address internally. Displayed as truncated
  // (last octet zeroed for IPv4, last 80 bits zeroed for IPv6) to admins without
  // `fb.submissions.export.elevated` permission.
  // Included in JSON exports only for users with `fb.submissions.export.elevated`
  // permission (restricted to `system_owner` by default and delegable to `super_admin`).
  // NOT included in CSV exports.
  // Purged on GDPR delete (same as other PII).
  userAgent: string;
  type: 'real' | 'test';
  data: Record<string, SubmissionFieldValue>; // see §52 for per-type serialisation. Auth/security secrets are excluded from standard submission persistence.
  paymentResult: PaymentResult | null;
  processingStatus: ProcessingStatus;
  afterSubmitResults: AfterSubmitResult[]; // see §51
  notes: AdminNote[];                       // see §51
  flags: SubmissionFlag[];                  // see §51
  paymentChargeGuard: PaymentChargeGuard | null; // see §57
}

type ProcessingStatus =
  | 'pending'     // received; after-submit actions not yet started
  | 'processing'  // actions executing
  | 'processed'   // all actions completed successfully
  | 'failed'      // all retries exhausted; at least one action failed; none succeeded
  | 'partial'     // some actions succeeded, some failed after all retries
  | 'deleted';    // soft-deleted / removed from active admin workflows

// Status transitions:
// pending → processing → processed
//                    ↘ failed (all actions failed, all retries exhausted)
//                    ↘ partial (mixed outcome)
// Re-run triggered by admin: any status → processing (for the re-run scope).
// After successful re-run of a 'failed' submission: status → 'processed'.
// After partial re-run success: may transition from 'failed' to 'partial' or 'processed'.

type SubmissionFieldValue = string | number | boolean | string[] | object | null;
```

> **Operational-form carve-out:** system auth/security flows (for example `auth.login`, `auth.register`, `auth.forgot-password`, `auth.reset-password`, `auth.2fa-*`, `profile.change-email`, and other credential-handling workflows) may execute without creating a standard persisted `FormSubmission` record. Where those flows do emit operational audit or support traces, secret values are always excluded.

### 13.2 Submissions List View

| State | Description |
|-------|-------------|
| loading | Skeleton rows. |
| loaded | Table: ID, submitted-by, date, status, quick preview. |
| empty | 'This form has no submissions.' |
| filtered | Active filter chips shown. |
| searching | Full-text search across indexed (`searchable`) fields. |
| detail-open | Side-panel or full page. Values rendered against `formSnapshot`. |
| exporting | CSV/JSON export generating. See §43. |

### 13.3 Admin Actions on Submissions

- **View:** Values rendered using `formSnapshot`. Labels/types match what user saw.
- **Edit:** RBAC: form.submissions.edit. Creates audit entry with before/after diff. Edited fields show 'Edited' badge. Does NOT re-trigger after-submit actions or write-back.
- **Add Note:** Admin-only internal note. See `AdminNote` in §51.
- **Flag:** Mark for review. See `SubmissionFlag` in §51.
- **Delete:** Soft-delete by default. Hard-delete: restricted to `system_owner` by default; may be delegated to `super_admin`. RBAC: form.submissions.delete. `processingStatus` transitions to `deleted` (not a live status — used for audit).
- **Re-run actions:** Re-fire any or all after-submit actions. Updates `afterSubmitResults`. Transitions `processingStatus` as described in §12.1. API body: `{ actionIndexes?: number[] }` — if omitted, all actions are re-run. **Payment is NEVER re-charged on re-run** (the PaymentChargeGuard in §57 prevents it). Conditions ARE re-evaluated against the stored submission data. DataBinding write-back is NOT re-triggered on re-run.
- **Export single:** JSON or PDF. PDF export renders the submission using the `formSnapshot` layout. See §65 for full single-submission PDF export specification.

### 13.4 Data Retention

- `dataRetentionDays: null` = keep forever.
- GDPR delete always overrides retention policy.
- Anonymous submissions (`submittedBy: null`) are NOT auto-removed by GDPR user-delete. Submissions containing the deleted user's email in field data are flagged for admin review in a 'GDPR Review' queue. See §66 for the full GDPR Review Queue specification.
- Retention expiry: auto-deleted N days after `submittedAt`. Admin notified 7 days before. Associated `StoredFile` records deleted in same job.
- `ipAddress` is treated as PII: purged on GDPR delete.

---

## 14. FORM SETTINGS & ADMIN CONFIGURATION

### 14.1 Access & Visibility Settings

| Setting | Description |
|---------|-------------|
| visibility | public / authenticated / role-gated |
| auth-redirect | redirect to login / show custom message / show login form inline |
| allowed-roles | If role-gated: multi-select roles |
| ip-allowlist | CIDR notation or single IPs. Multiple entries comma-separated. Stored as `FormDefinition.ipAllowlist: string[]`. Null = no restriction (default). Applies to GET render and POST submit. On mismatch: HTTP 403. CDN/proxy headers respected; outermost non-trusted IP used. Example: `['203.0.113.0/24', '198.51.100.5']`. |
| csrf | Always enabled for system forms. For custom forms: enabled by default; can be disabled only for API-only forms explicitly flagged. |
| cors-origins | CorsOrigin[] (see §2.5). Exact origins only; no wildcards. |

### 14.2 Rate Limit Settings

| Setting | Description |
|---------|-------------|
| limit | Max submissions per window. Default: 10/hour/IP (public), 60/hour/user (auth). |
| window | Rolling window in minutes (default: 60). |
| by | Per IP / Per user / Per session / Per tenant. |
| exceeded-action | Block / Queue |
| exceeded-message | LocalisedString shown when limit reached. |

These settings map to `FormDefinition.rateLimitPolicy`. See `RateLimitPolicy` type in §51.

### 14.3 Notifications Settings

| Setting | Description |
|---------|-------------|
| notify-on-submit | Toggle. Who to notify: admin / specific emails / webhook. |
| notification-frequency | Every submission / Digest (hourly/daily) / Only if field matches condition. |
| submission-email-template | Template for admin notification. |
| confirmation-email-template | Template sent to submitter. |

**Digest delivery:** Batch email sent at scheduled intervals (hourly or daily, as configured). Zero submissions in the window = no digest sent. Digest lists: count of submissions in window, 3-field preview per submission (the first 3 non-sensitive, non-hidden fields in the form, in field order), link to submissions panel. Uses `form-digest` built-in email template. The 'window' is the period between this digest delivery and the previous one. Digest scheduling is managed by a background job — see §62 for the full Digest Delivery Engine specification. Digest is a scheduled background operation; it is NOT an `AfterSubmitAction` type and does not appear in `AfterSubmitConfig`.

**Note on email subjects:** `{{ }}` expression syntax (§33) is supported in email subject lines in addition to email bodies. Available variables are the same as those listed in §40.3.

### 14.4 Embedding & Sharing

| Setting | Description |
|---------|-------------|
| direct-url | `/forms/{slug}` |
| embed-code | Iframe embed snippet. See §23. |
| popup-embed | JS snippet. Modal overlay. See §23. |
| short-link | Optional short URL. Generated on demand. Stored as `FormDefinition.shortLink`. See §63 for generation, lifecycle, and expiry rules. |
| qr-code | Auto-generated QR code PNG. Generated on demand from `FormDefinition.qrCodeToken`. Downloadable (PNG/SVG). See §63. |

### 14.5 RBAC Configuration (Per Form)

- `fb.form.view`, `fb.form.edit`, `fb.form.publish`, `fb.form.duplicate`, `fb.form.archive`
- `fb.form.delete` (distinct from `fb.form.archive` — delete is permanent, archive is reversible)
- fb.submissions.view, fb.submissions.edit, fb.submissions.export, fb.submissions.delete
- form.test.run, form.system.edit, form.template.create, form.template.delete, form.field.customCss
- fb.submissions.export.elevated (restricted to `system_owner` by default; may be delegated to `super_admin`; includes IP address in JSON exports)

### 14.6 Sandbox Settings

| Setting | Description |
|---------|-------------|
| sandbox-webhook-url | Webhook endpoint for test runs. If blank: payload logged but not sent. |
| sandbox-payment | Tenant-level sandbox credentials. Link to Settings > Billing > Sandbox. |

### 14.7 Submission Deduplication Settings

| Setting | Description |
|---------|-------------|
| allowMultipleSubmissions | Toggle (default: on). Only available on authenticated/role-gated forms. |
| multipleSubmissionsErrorMessage | LocalisedString. Default: 'You have already submitted this form.' |

See **Section 34** for full deduplication specification.

---

## 15. MULTI-STEP FORMS

### 14.1 Multi-Step Configuration

| Element | States | Interconnects |
|---------|--------|---------------|
| Enable multi-step | Toggle | Converts flat section list into steps. Creates `steps[]` array on FormDefinition. |
| Step titles | Each step gets a `title: LocalisedString` | i18n_BRANCH |
| Progress indicator | steps-bar / progress-bar / step-count / none | None |
| Navigation | Next / Back buttons | Next disabled until current step valid |
| Step conditions | Entire steps can be conditionally skipped | CONDITION_BRANCH |
| Save-and-resume | Toggle | AUTH_BRANCH — requires authenticated user. See §14.4. |

### 14.2 Multi-Step States

| State | Description |
|-------|-------------|
| step-N-active | Current step rendered. Previous steps collapsed. |
| step-N-complete | Step passed validation. Progress indicator updates. |
| step-N-invalid | User clicked Next with errors. Focus on first error. |
| step-skipped | Condition caused step to be skipped. See §14.3. |
| step-back | User clicked Back. Previous step shown. |
| step-1-back | User clicks Back on the first step. Back button hidden by default. Admin can configure: show button (browser back) / hide (default) / custom `onFirstStepBack` URL redirect. |
| final-step | Last non-skipped step. Next replaced with Submit button. |
| submission | Submit pressed. All non-skipped steps' data merged. Full validation run. |
| resumed | User returned to partially-completed form. Last completed step shown. |

### 14.3 Validation Rules for Skipped Steps

1. **Required field waiver:** Required fields in a skipped step have their required validation waived.
2. **Server authority:** Server re-evaluates all step-skip conditions independently. If server determines a step was NOT skipped (client manipulation detected), it validates that step's required fields normally.
3. **Data discarded:** Field values from skipped steps are omitted from submission payload. Client-submitted values for server-confirmed skipped step fields are discarded.
4. **Hidden field override:** If a field has `hidden: true` AND is inside a server-confirmed skipped step, the step-skip rule takes precedence — value is discarded, not submitted.
5. **Formula references:** Skipped-step field values are treated as `null` in formulas. If this causes formula evaluation to fail: HTTP 422.

### 14.4 Save-and-Resume

- Requires `visibility: 'authenticated'` or `'role-gated'`. If enabled on a public form: silently disabled + build warning.
- Draft saved to server on every step completion.
- **Draft expiry:** See §55 for the canonical save-and-resume draft expiry specification. Summary: TTL is 30 days from the last step completion (not from creation, and not reset by mere form visits). Each step completion resets the TTL. TTL is capped by `dataRetentionDays` if that value is less than 30.
- On return visit: toast 'Continue where you left off?' with 'Resume' / 'Start Over'.
- 'Start Over' discards server draft.
- After successful submission: server draft deleted.
- **Visibility change impact:** If a form's visibility is changed from `authenticated`/`role-gated` to `public`, save-and-resume is automatically disabled. All in-progress drafts are immediately invalidated and deleted. Admins are warned before saving the visibility change: "Changing visibility to public will disable Save & Resume and delete N in-progress drafts." See §67 for full draft and deduplication state transition rules on visibility changes.

**File upload in save-and-resume — abandonment race condition:** See **Section 56** for the full specification of what happens when a file uploaded mid-form is at risk of the 48-hour purge when the user resumes a saved draft.

---

## 16. VERSION HISTORY & ROLLBACK

### 15.1 Version Entity

```typescript
interface FormVersion {
  formId: string;
  version: number;
  publishedAt: Date;
  publishedBy: string;
  changesSummary: string;  // auto-generated diff summary (capped 500 chars)
  snapshot: FormDefinition;
  submissionCount: number;
}
```

### 15.2 Version History Panel

| State | Description |
|-------|-------------|
| list | Timeline. Each entry: version number, published by, date, changes summary, submission count. |
| preview | Click version → read-only preview. |
| diff | Select two versions → visual diff: added/removed/modified fields highlighted. |
| restore | Click 'Restore' → creates new draft from that version. Current draft replaced after confirmation. |

**Version history limit:** The version history panel shows the last **50 versions**. Older versions are not displayed in the panel but their snapshots remain embedded in submission records. If an admin tries to view a submission whose `formVersion` is not in the version history panel, the submission detail renders correctly using the embedded `formSnapshot` — and a note reads: "This submission was collected on an older version (v{N}) that is no longer shown in version history." There is no functional loss; only the panel listing is limited.

### 15.3 Changes Summary Auto-Generation

Auto-generated at publish time by diffing new snapshot against previous version. Not admin-authored. Format:
- "Added field 'Company Name' (text input) to section 'Business Details'"
- "Removed field 'Fax Number'"
- "Changed label of 'Email' → 'Email Address'"

Capped at 500 chars. Overflow: "N fields added, M removed, P modified. [View diff]"

### 15.4 Draft vs Live

| State | publishedStatus | draftStatus | End-user access |
|-------|----------------|-------------|----------------|
| Never published | never-published | clean or has-changes | 404 |
| Published, no draft | published | clean | Live version served |
| Published with draft | published | has-changes | Live version served; draft invisible to users |
| Archived | archived | clean or has-changes | 410 Gone |

Builder toolbar badge: if `publishedStatus: 'published'` AND `draftStatus: 'has-changes'` → shows 'Draft' badge. If `draftStatus: 'clean'` → shows 'Up to date'. If `publishedStatus: 'never-published'` → shows 'Not published'.

---

## 17. GLOBAL ERROR PATHS & EDGE CASES

### 16.1 Builder Error Paths

| State | Description |
|-------|-------------|
| Auto-save fails | Toast: 'Auto-save failed'. Manual save button glows amber. |
| Publish fails | See §35. Draft retained. |
| ETag conflict | See §21. |
| Circular condition | Detected on save. Cannot publish. |
| Orphaned field reference | Warning badge. See §38. |
| Template instantiation fails | Error shown. Retry. |
| Field type plugin unavailable | Field renders as disabled placeholder. Admin alerted. |
| Formula evaluation error in builder | Inline warning on payment field. |
| Protected field hidden by condition | Warning. Publishing allowed with acknowledgement. |

### 16.2 Form Render Error Paths

| State | Description |
|-------|-------------|
| Form not found | 404. 'This form is not available.' |
| Form not published | Draft-only. 404. |
| Form archived | 410 Gone. |
| Auth-required, not logged in | Redirect to login or inline login form. |
| Form disabled by feature flag | 404 or 'Not available'. |
| JS fails to load | No-JS fallback for system forms. Non-system: 'This form requires JavaScript.' |
| IP allowlist mismatch | HTTP 403. |

### 16.3 Submission Error Paths

| State | Description |
|-------|-------------|
| Validation fails | HTTP 400. `{ errors: [{ field, code, message }] }` |
| CSRF token invalid | HTTP 403. 'Session expired. Please refresh.' |
| Rate limit | HTTP 429. Custom exceeded-message or default. |
| Duplicate submission | HTTP 409. multipleSubmissionsErrorMessage. See §34. |
| Payment fails | Payment field enters failed state. User corrects and re-submits. |
| After-submit email fails | Submission stored. Failure logged. User sees success. Admin notified. |
| After-submit webhook fails | Stored. Retried. processingStatus → 'failed'/'partial' after retries. Admin notified. |
| After-submit API call fails | Same as webhook. See §28. |
| After-submit redirect blocked | Fallback link: 'Click here to continue.' |
| Server error (5xx) | Not stored. localStorage backup written. On next form load: before showing 'Restore unsaved submission?' prompt, the client performs a silent server check for an existing submission from this user for this form. If one exists (meaning the submission succeeded but the 5xx was a network/response error): prompt reads 'Your previous submission was received successfully.' with no restore option. If no submission exists: prompt reads 'Restore unsaved submission?' |
| Formula evaluation fails | HTTP 422. Per-field error on payment field. Not stored. |
| Step-skip validation conflict | HTTP 422. Server detected client-manipulated skip. |
| CAPTCHA fails | HTTP 400. `{ errors: [{ field: 'captcha', code: 'captcha_failed' }] }` |
| Context entity missing property | Field renders empty. Standard required validation applies. Form load does NOT fail. |
| Payload too large | HTTP 413. See §58 for submission payload size limits. |
| Mid-session publish | See §54 for how the form handles a new version published while user is filling out a form. |
| Form deleted mid-session | POST submission to deleted form URL returns HTTP 404. Client shows: 'This form is no longer available. Your data has not been submitted.' No localStorage backup is written. |

---

## 18. INTERNATIONALISATION (i18n)

### 17.1 Overview

All user-facing text on a rendered form is internationalised. Translations stored inline on the FormDefinition.

### 17.2 LocalisedString Type

```typescript
// CANONICAL DEFINITION — referenced everywhere in this document.
// LocalisedString is used for all user-facing text. Do not redefine elsewhere.
interface LocalisedString {
  default: string;                     // value for the form's defaultLocale. MUST NOT be empty.
  translations: Record<string, string>; // keyed by BCP-47 locale code: 'fr', 'de', 'pt-BR'
}
```

`LocalisedString` is used for:
- Field labels, placeholders, help text
- Section titles and descriptions
- Step titles (multi-step forms)
- Validation error messages
- After-submit messages, modal titles, popup text
- Option labels (select, radio, checkbox)
- CAPTCHA error messages
- Rate limit exceeded messages
- Submit button label (see §49)
- Email template subjects and bodies — `{{ }}` expressions (§33) are supported in both subject and body `LocalisedString` fields of `FormEmailTemplate`. Available variables: same set as §40.3.

### 17.3 Translation Storage

Translations stored on the FormDefinition JSONB record alongside the structure they apply to. No separate translation table.

### 17.4 Fallback Chain

1. Check for exact locale: `pt-BR`.
2. Try language without region: `pt`.
3. Use `default` (the `defaultLocale` value).
4. Never show empty string for a label.

### 17.5 Editing Translations in Builder

1. Toolbar **Locale Switcher** dropdown — all `supportedLocales`.
2. Non-default locale selected → Properties Panel shows translation value for that locale (empty if not yet translated).
3. Admin types translation → saved to `translations[locale]` on the `LocalisedString`.
4. Fields missing a translation for a supported locale: warning badge 'Missing translation for FR'.
5. Build Warnings panel lists all missing translations. Form can be published with missing translations (soft warning).

### 17.6 Locale-Aware Preview

Preview mode Locale dropdown switches all rendered text to selected locale. Missing translations fall back to `default` and are highlighted with a yellow outline.

### 17.7 i18n Scope

i18n applies to the rendered form only. Builder admin UI is not part of this spec.

---

## 19. DATA BINDING

### 18.1 Overview

DataBinding connects a form field to a property of an entity in the application. Serves two purposes:
1. **Pre-fill:** Field is pre-populated on form load.
2. **Write-back (optional):** On successful submission, entity property is updated.

### 18.2 DataBinding Schema

```typescript
interface DataBinding {
  entityType: DataBindingEntityType;
  property: string;           // dot-notation: 'profile.firstName', 'address.city'
  writeBack: boolean;
  writeBackCondition: ConditionGroup | null; // canonical definition: §51.5
}

type DataBindingEntityType = 'user' | 'tenant' | 'context';
```

### 18.3 Pre-fill Behaviour

- Pre-fill is server-side: bound fields have `defaultValue` resolved server-side from the entity.
- Unauthenticated user + `entityType: 'user'` binding: field renders empty. No error.
- Pre-filled values are editable unless `readOnly: true`.

### 18.4 Context Entity

Provided by the embedding page via:
- URL parameter `?formContext=<base64-encoded-JSON>` (HMAC-signed to prevent tampering)
- JavaScript `window.FormContext` object (popup embeds)

Context properties must be declared in `FormDefinition.contextSchema`. Binding to undeclared property: build warning + ignored at runtime. Missing context property bound to a required field: field renders empty, standard required validation applies.

### 18.5 Expression Syntax

| Expression | Resolves To |
|------------|-------------|
| `{{user.email}}` | Authenticated user's email |
| `{{user.firstName}}` | Authenticated user's first name |
| `{{query.ref}}` | URL query parameter `ref` |
| `{{date.today}}` | Current date, ISO format |
| `{{date.now}}` | Current datetime, ISO format |
| `{{tenant.name}}` | Current tenant name |
| `{{context.orderId}}` | Context entity property `orderId` |
| `{{field.fieldName}}` | Value of another field |
| `{{submission.id}}` | Available post-submission only (redirect URLs) |

Resolved server-side. Unknown expressions → empty string + server warning logged.

### 18.6 Write-Back

Occurs after submission stored, before after-submit actions fire (see §48 for full order).

1. Engine calls entity API to update bound property.
2. Write-back failure: submission stored, failure logged, admin notified. After-submit actions still execute.
3. RBAC check: submitting user must have permission to update the bound property. If not: silently skipped, failure logged.

> **Conflict warning — write-back and trigger-api:** DataBinding write-back (Step 8 in §48) fires before after-submit actions (Step 9). A `trigger-api` action in Step 9 could target the same entity property as a write-back binding, potentially overwriting the write-back result (or vice versa, depending on which arrives at the entity API last). Do not use both DataBinding write-back and `trigger-api` to update the same entity property on the same form. Use one mechanism only for any given property.

---

## 20. ACCESSIBILITY (A11Y)

### 19.1 Target Standard

WCAG 2.1 Level AA.

### 19.2 Keyboard Navigation

| Interaction | Behaviour |
|-------------|-----------|
| Tab / Shift+Tab | Move focus between interactive fields. Hidden fields excluded from tab order. |
| Enter | Submit when focused on submit button. For select: open options. |
| Space | Toggle checkboxes, radio buttons, toggles. |
| Arrow keys | Navigate radio groups, button groups, date picker, slider. |
| Escape | Close dropdowns, date pickers, modals. |
| Multi-step Next | Focus moves to first field of new step. |
| Error focus | On validation failure: focus moves to first invalid field. |

### 19.3 Screen Reader Support

| Element | Implementation |
|---------|---------------|
| Form | `<form>` with `aria-label` from form name. |
| Field labels | `<label for="...">` associated with every input. Never placeholder-only. |
| Required fields | `aria-required="true"`. Label suffix: ' (required)' or asterisk with `aria-label`. |
| Error messages | `aria-describedby` links input to error element. Error element: `role="alert"`. |
| Help text | `aria-describedby` links input to help text. |
| Section headings | `<h2>` (or configured level). |
| Progress bar | `<progress>` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`, `aria-label`. |
| Dynamic content | When field becomes visible: focus NOT moved (avoid disorientation). When hidden: if it had focus, focus moves to nearest visible sibling or section. |
| After-submit message | `role="status"` (success) or `role="alert"` (error). `aria-live="polite"`. |
| Modal | `role="dialog"`, `aria-modal="true"`, `aria-labelledby`. Focus trapped. On close: focus returns to trigger. |
| Submit button | `aria-busy="true"` while processing. `aria-disabled="true"` while disabled. |
| Loading states | `aria-busy="true"` on field or form. |

### 19.4 Colour Contrast

- Text on background: minimum 4.5:1 (AA). Builder warns on custom colours below minimum.
- UI components (borders, focus rings): minimum 3:1.
- Focus ring: minimum 2px solid, contrasting. Builder warns on low contrast custom focus colour.
- Error colour: minimum 4.5:1 against background.

### 19.5 Hidden Labels Warning

If label is `hidden` AND field is `required`: builder warning. Field still gets `aria-label` from its `label` value even when visually hidden.

### 19.6 No-JS Fallback

For system forms (login, register, reset-password): server-rendered HTML fallback at same URL. Standard HTML form submission (POST + redirect). CSRF token as hidden input. Server-side validation errors inline.

---

## 21. SUBMISSION SCHEMA VERSIONING

### 20.1 The Problem

Form fields change over time. Submission collected on form v3 may reference fields that no longer exist in form v12.

### 20.2 Solution: Submission Snapshot

Every `FormSubmission` stores a `formSnapshot: FormDefinition` — full deep-copy of the published FormDefinition at submission time. Never modified.

### 20.3 Submission Detail Rendering

1. UI loads submission's `formSnapshot`, not current definition.
2. Fields rendered in snapshot's layout — section order, field order, labels, types.
3. Data key matching snapshot field → shown normally.
4. Data key not in snapshot → shown in 'Unrecognised Fields' section.
5. Snapshot fields with no data → shown as 'No response'.

### 20.4 Schema Version Badge

Submission detail shows: "Collected on form version N." If current version is higher: "The form has changed since this submission was collected."

### 20.5 Export Consistency

Cross-version exports include `formVersion` column. Field columns normalised across versions: submissions missing a field from a later version have an empty cell. `schema_note` column flags inconsistencies.

---

## 22. CONCURRENT EDITING & OPTIMISTIC CONCURRENCY

### 21.1 Edit Lock

**Acquisition:** Admin opens form → `POST /forms/:id/lock`. Granted if no active lock or existing lock is expired (TTL: 5 minutes, renewed by heartbeat every 2 minutes). Lock includes: `lockedBy`, `lockedAt`, `expiresAt`, `sessionId`.

**Release:** `DELETE /forms/:id/lock` on close/navigate.

**Contention:** If another user holds lock: canvas read-only. Banner: "[Name] is editing this form." Button: "Request edit access." Lock holder gets real-time WebSocket notification. Lock holder can grant (transfers) or deny.

**Expiry protection:** 30-second countdown warning to lock holder before TTL expiry.

### 21.2 ETag-Based Save Guard

All save requests (PATCH /forms/:id) require `If-Match: <etag>` header.

- Match: save proceeds, new ETag generated.
- Mismatch: HTTP 412. Canvas: "Your changes conflict with a recent save."
- Diff view shown. Admin chooses: 'Keep my changes' (force-save with confirmation) or 'Discard my changes' (reload server version).

---

## 23. EMBEDDED FORMS — AUTH, CSRF & CROSS-ORIGIN

### 23.1 Overview

Forms can be embedded on external sites via iFrame or popup JS snippet.

### 23.2 CORS Configuration

`corsOrigins: CorsOrigin[]` (see §2.5) specifies allowed embedding origins. Exact-match only; no wildcards. Server includes `Access-Control-Allow-Origin: <matching-origin>` on form GET and submission POST responses.

`Access-Control-Allow-Credentials: true` is included ONLY when `visibility: 'public'` AND the embedding origin is in `corsOrigins`. The purpose is to allow the form to read first-party cookies that the parent site may have set (e.g. for analytics or pre-fill from a session). Because auth-gated forms cannot be embedded on external origins at all (§23.4), this header is never needed for authentication flows. Developers building embeds should use the header-based CSRF token flow (§23.3) rather than relying on cookie-based credentials.

### 23.3 CSRF Token in Cross-Origin Embeds

1. Embed snippet fetches short-lived CSRF token from `/api/forms/:id/csrf-token` (CORS-enabled GET).
2. Token stored in JS memory (not localStorage, not cookies).
3. Submitted as `X-Form-CSRF-Token` header.
4. Server validates header-based token. Cookie-based CSRF only for same-origin.
5. CSRF tokens expire after 30 minutes. Embed snippet auto-refreshes.

### 23.4 Auth-Gated Forms in Embeds

Auth-gated forms cannot be embedded on external origins. Builder blocks setting `corsOrigins` on auth-gated forms: "Auth-gated forms cannot be embedded on external sites."

Same-origin embedding is permitted.

### 23.5 Popup Embed

JS snippet injects a button that opens the form in a shadow DOM modal overlay. Avoids iFrame sandboxing. CSRF flow (§23.3) still applies.

### 23.6 iFrame Embed Limitations

Third-party cookies blocked by Safari/Firefox. Auth-dependent features (save-and-resume, pre-fill from user DataBinding) will not function in iFrame embeds on external origins. Embed snippet shows client-side warning if cookies are blocked.

---

## 24. FILE STORAGE & LIFECYCLE

### 24.1 Canonical Schema Reference

**The canonical schema for file storage is defined in GeneralApp §23.3 (`file_uploads` table).**

Form Builder uses the `form_id`, `submission_id`, and `draft_id` columns defined in GeneralApp §23.3 to link files to form entities:
- `source_type`: `'form_builder'` — indicates the file originates from FormBuilder
- `form_id`: References the `FormDefinition` the file was uploaded through
- `submission_id`: References `FormSubmission` (populated on final submission)
- `draft_id`: References `FormDraft` for save-and-resume scenarios
- `field_name`: The field key that stored this file

### 24.2 TypeScript Projection

```typescript
// Runtime projection of GeneralApp §23.3 for TypeScript convenience
interface StoredFile {
  id: string;
  tenantId: string;
  formId: string;              // Maps to form_id in §23.3
  submissionId: string | null; // Maps to submission_id in §23.3
  draftId: string | null;      // Maps to draft_id in §23.3
  fieldName: string;
  originalName: string;
  mimeType: string;
  sizeBytes: number;
  storageKey: string;          // Maps to storage_key in §23.3
  previewUrlExpiresAt: Date | null;
  scanStatus: 'pending' | 'clean' | 'quarantined' | 'error'; // Maps to scan_status in §23.3
  uploadedAt: Date;
  deletedAt: Date | null;
  saveAndResumeDraftId: string | null;
}
```

**Reference Rule:** This document does not define a durable file schema. All file storage uses GeneralApp §23.3 `file_uploads` table.

### 24.3 Signed URL Lifecycle

- After upload: signed URL valid 24 hours (for immediate preview).
- Submission data stores `StoredFile.id`, not URL.
- Admin views submission → server generates fresh signed URL on demand (15 minutes).
- Signed URLs never stored in database.
- `previewUrlExpiresAt` is a runtime projection describing the most recently issued signed URL expiry for UI/session use only; durable file records still store only file identity and storage location.

### 24.4 Upload Flow

1. `POST /forms/:formId/fields/:fieldId/upload-url` — server validates MIME type, size.
2. Server returns `{ uploadUrl, fileId }` — pre-signed PUT URL (valid 10 minutes).
3. Client uploads directly to object storage.
4. `POST /forms/:formId/fields/:fieldId/upload-complete { fileId }` — server initiates virus scan.
5. `scan_status: 'pending'`. Field shows scanning indicator.
6. Scan complete: `clean` → ready. `quarantined` → threat detected, file quarantined. `error` → scan failed, manual review required.

**Upload security rules:** allow/deny decisions are enforced server-side using extension allowlists, MIME validation, and file-signature / magic-byte inspection. Image-specific fields must successfully decode as images; SVG is rejected by default for image/avatar-style fields in the shared runtime because it can carry active content. Original filenames are metadata only; the trusted storage location is always derived from server-generated identifiers.
7. On form submission: `submissionId` set on all `StoredFile` records for this session.

### 24.5 Deletion Lifecycle

| Trigger | Behaviour |
|---------|-----------|
| Submission deleted | All StoredFile records soft-deleted. Object storage hard-deleted after 24 hours. |
| Data retention expiry | StoredFile records deleted in same retention job. Object storage deleted immediately. |
| GDPR delete | StoredFile records for user's submissions deleted. Object storage deleted immediately. |
| Quarantined file | Deleted/quarantined immediately on scan completion. |
| Abandoned upload | Uploads with `submissionId: null` AND `saveAndResumeDraftId: null` purged after 48 hours by hourly background job. Uploads linked to a save-and-resume draft are NOT purged by the 48-hour rule — they are governed by the draft expiry policy (see §56). |

---

## 25. FORM ANALYTICS & METRICS

### 25.1 Form-Level Metrics

| Metric | Description |
|--------|-------------|
| views | GET requests to form URL |
| starts | Sessions with ≥1 field interaction |
| completions | Successful submissions |
| completion_rate | completions / starts |
| median_time_to_complete | Median seconds from first interaction to submission |

### 25.2 Field-Level Metrics

| Metric | Description |
|--------|-------------|
| interaction_rate | % of starters who interacted with this field |
| drop_off_rate | % of users who focused this field but did not complete |
| validation_error_rate | % of interactions resulting in a validation error |

### 25.3 Multi-Step Funnel

Per-step funnel: entered, completed, drop-off count and rate per step.

**Skipped steps in funnel analytics:** Steps that are server-confirmed skipped (§14.3) are tracked separately with status `'skipped'`. They do NOT count as entered, completed, or dropped-off. The funnel chart renders skipped steps with a distinct visual treatment (grey, strikethrough) and a `skipped: N` count. This prevents funnel gaps that would otherwise confuse admins reviewing step-level completion rates.

### 25.4 FieldAnalyticsConfig

```typescript
interface FieldAnalyticsConfig {
  trackInteraction: boolean;  // default true. Auto-false if field.sensitive=true.
}
```

### 25.5 Analytics Dashboard

Accessible from: Builder toolbar → Analytics tab.

| View | Description |
|------|-------------|
| overview | Form-level metric cards. Date range picker. |
| funnel | Step-by-step funnel chart. |
| field-detail | Per-field interaction, drop-off, error rates. Sortable. |
| trends | Submissions over time chart. |

### 25.6 Analytics Retention

90 days. Independent of `dataRetentionDays`. See §45.

---

## 26. FORM DUPLICATION

### 26.1 How to Duplicate

Toolbar > More > 'Duplicate Form'. RBAC: `fb.form.duplicate`.

### 26.2 What is Copied

| Element | Behaviour |
|---------|-----------|
| FormDefinition structure | Deep copy. All sections, fields, conditions, after-submit config. |
| Theme | Copied. |
| DataBindings | Copied. Warning badge on each bound field: 'DataBinding copied — verify target entity is correct.' |
| systemKey | Set to `null`. Always `type: 'custom'`. |
| slug | Auto-generated: `{original-slug}-copy`. Must be unique within tenant. |
| publishedStatus | `never-published`. |
| draftStatus | `has-changes`. |
| Submissions | NOT copied. Zero submissions. |
| Version history | NOT copied. Starts at version 0. |
| Analytics data | NOT copied. New form starts with zero analytics. `analyticsEnabled` setting is copied from source. |
| Short-link / QR | NOT copied. New form has `shortLink: null`, `qrCodeToken: null`. Must generate new ones if needed. |
| Audit | `fb.form.duplicated` logged: sourceFormId, newFormId, actor. |

### 26.3 Post-Duplication UX

Builder opens with new form in draft. Banner: "This form was duplicated from [Source Form Name]. Review all settings before publishing."

---

## 27. REPEATER FIELD — FULL SPECIFICATION

### 27.1 Overview

Repeater allows users to add multiple instances of a structured sub-form. Examples: multiple addresses, job history entries, line items.

### 27.2 RepeaterFieldConfig

```typescript
interface RepeaterFieldConfig {
  minRows: number;          // default 0
  maxRows: number;          // default 10
  addButtonLabel: LocalisedString;    // default: '+ Add item'
  removeButtonLabel: LocalisedString; // default: 'Remove'
  allowReorder: boolean;    // default true
  subFields: RepeaterSubField[];
}

interface RepeaterSubField {
  id: string;
  type: FieldType;          // any type EXCEPT: Repeater, Payment, Captcha, Signature
  name: string;
  label: LocalisedString;
  placeholder: LocalisedString | null;
  helpText: LocalisedString | null;
  required: boolean;
  disabled: boolean;
  validation: ValidationRule[];
  options: FieldOption[] | null;
  optionSource: OptionSource | null;
  conditions: DisplayCondition[]; // scoped: can only reference other sub-fields in SAME ROW
  style: FieldStyle;
  // DataBinding NOT supported. No nested Repeaters.
}
```

### 27.3 Restrictions

| Restriction | Reason |
|-------------|--------|
| No nested Repeaters | Serialisation ambiguity |
| No Payment sub-field | Payment is form-level |
| No CAPTCHA sub-field | CAPTCHA is session-scoped |
| No Signature sub-field | Base64 per row creates unacceptable payload size |
| Conditions scoped to row siblings only | Cross-row conditions not supported |
| No DataBinding | Write-back semantics undefined for array entries |

### 27.4 Repeater States

| State | Description |
|-------|-------------|
| empty | 0 rows. '+Add' shown. If minRows > 0: error on submit. |
| N-rows | N rows rendered. |
| adding | New empty row appended. Focus moves to first sub-field of new row. |
| removing | Confirmation if minRows would be violated. Row removed with animation. |
| reordering | Row lifts. Drop zones between rows. |
| at-max | maxRows reached. '+Add' disabled. |
| row-invalid | Sub-field error. Row highlighted. |

### 27.5 Submission Serialisation

```json
{
  "data": {
    "addresses": [
      { "line1": "123 Main St", "city": "Boston", "postcode": "02101" },
      { "line1": "456 Elm St", "city": "Cambridge", "postcode": "02139" }
    ]
  }
}
```

### 27.6 Repeater in Submission Detail View

Collapsible table: one column per sub-field, one row per entry. Row count in collapsed header.

### 27.7 Repeater Validation

Sub-field validations applied per-row independently. `minRows`/`maxRows` enforced server-side and client-side.

### 27.8 Repeater in Exports

CSV: one CSV row per Repeater entry, parent fields repeated. `repeater_row_index` column added. JSON: preserves nested array.

---

## 28. CAPTCHA — VERIFICATION LIFECYCLE

### 28.1 Overview

CAPTCHA field integrates with providers via `captcha.verify` MCP capability.

### 28.2 Supported Providers

| Provider | Notes |
|----------|-------|
| hCaptcha | Default. GDPR-compliant. |
| Cloudflare Turnstile | Invisible CAPTCHA option. |

Configured in Settings > Security > CAPTCHA. See §41.

### 28.3 Client-Side Flow

1. Widget rendered at field position.
2. User completes challenge (or invisible auto-completes).
3. `onSuccess(token)` fires. Token stored in form state.
4. Token expires before submission (typically 2 minutes): widget resets to `expired`. User must re-solve.

### 28.4 Server-Side Verification

1. Server extracts token from submission.
2. Server calls provider verification API (server-to-server; secret never exposed to client).
3. Provider returns `{ success: boolean, error-codes?: string[] }`.
4. `success: false`: HTTP 400. Submission not stored.
5. Provider API unreachable: fail-closed by default. `captchaFailOpenOnProviderError: true` switches to fail-open.

### 28.5 Test Run Behaviour

CAPTCHA verification bypassed in test runs. `X-Form-Test-Run: true` header + authenticated admin session.

---

## 29. AFTER-SUBMIT `trigger-api` — FULL SPECIFICATION

### 29.1 Overview

`trigger-api` makes a custom outbound HTTP call on form submission. Distinct from `trigger-webhook` (which targets registered endpoints). `trigger-api` is a fully configurable HTTP call.

### 29.2 Configuration Schema

```typescript
interface TriggerApiConfig {
  url: string;                         // absolute URL. HTTPS only.
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
  headers: Record<string, string>;     // values support {{ }} expressions
  bodyTemplate: string | null;         // JSON template with {{ }} variable injection
  auth: TriggerApiAuth | null;
  timeoutMs: number;                   // default 10000. Max 30000.
  responseMapping: ResponseMapping | null;
  retryPolicy: RetryPolicy;
}

type TriggerApiAuth =
  | { type: 'bearer'; token: string }
  | { type: 'basic'; username: string; password: string }
  | { type: 'header'; headerName: string; value: string }
  | { type: 'oauth2-client-credentials'; tokenUrl: string; clientId: string; clientSecret: string; scope?: string };

interface ResponseMapping {
  mappings: { responsePath: string; submissionKey: string }[];
}

interface RetryPolicy {
  maxAttempts: number;   // default 3
  backoffMs: number;     // default 1000
  retryOn: number[];     // default: [429, 500, 502, 503, 504]
}
```

### 29.3 URL Security

`trigger-api` URLs are subject to the SSRF protection rules defined in **§61 (Security Policy Registry)**. Summary: HTTPS only, no private IPs (RFC 1918), no localhost, no link-local. Validated at save time and at call time. Violation: `form.triggerapi.ssrf_blocked` audit logged, admin notified. See §61 for the full authoritative policy.

### 29.4 Execution & Error Handling

1. Fires after submission stored (see §48).
2. Request made server-side.
3. Success (2xx): response optionally mapped via `responseMapping`. Mapped values are appended to `FormSubmission.data` under the specified `submissionKey` names. These values appear in the submission detail view (labelled as '[API Response] fieldName') and are included in exports. Subsequent after-submit actions in the same sequence CAN reference mapped values via `{{field.submissionKey}}` expressions.
4. Failure: retried per `retryPolicy`. After all retries: `processingStatus` → `'failed'` or `'partial'`. Admin notified.
5. Test run: executes with `X-Form-Test-Run: true`. `responseMapping` not applied to real submission.

### 29.5 Builder UI

| Element | Description |
|---------|-------------|
| URL input | HTTPS required. Validated on blur. |
| Method selector | GET / POST / PUT / PATCH / DELETE |
| Headers editor | Key-value table. Values support {{ }}. |
| Body editor | JSON template with {{ }} autocomplete. |
| Auth section | Type selector + credential inputs. Secret fields masked after save. |
| Response mapping | JSON path → submission data key. |
| Test call button | Fires real call with sample data. SSRF protection applies. |
| Timeout input | Slider: 1–30 seconds. |
| Retry policy | Max attempts (1–5), initial backoff, retry-on status codes. |

---

## 30. FORMS LIST VIEW

### 30.1 Access

**URL:** `/admin/forms`  
**RBAC:** `fb.form.view` required. Forms without `fb.form.view` permission are not shown.

### 30.2 Layout

| Element | Description |
|---------|-------------|
| Page title | "Forms" |
| Search bar | Full-text search on form name and slug. Real-time (debounced). |
| Filter bar | Status (All / Draft / Published / Archived), Type (All / System / Custom / Template), Created by. Multiple filters combinable. |
| Sort controls | Name (A–Z), Last modified, Created date, Submission count. Asc/desc. |
| Create button | '+ New Form'. Requires `fb.form.create`. See §29.5. |
| Form cards / table | Card grid (default) or table row view. |

### 30.3 Form Card / Row

| Field | Description |
|-------|-------------|
| Form name | Clickable → opens in builder. |
| Slug | `/forms/{slug}` |
| Status badge | Draft / Published / Archived |
| Type badge | System / Custom / Template |
| Last modified | Relative time. Absolute on hover. |
| Submission count | Cached total. |
| Published version | Current live version number. |
| Actions menu | Edit, Duplicate, Archive, Delete, View submissions, Copy form URL. |

### 30.4 Bulk Actions

Multi-select (checkbox per row in table view). Bulk actions: Archive selected, Delete selected. Confirmation modal required. Bulk delete blocked if any selected form is a system form.

### 30.5 Creating a New Form

1. Click '+ New Form'.
2. Modal opens:
   - **Form name** (required). Auto-generates slug.
   - **Slug** (editable). URL-safe, unique within tenant. Uniqueness check on blur.
   - **Start from:** Blank form / From template / Duplicate existing.
3. Click 'Create'.
4. Server creates `FormDefinition` with `publishedStatus: 'never-published'`, `draftStatus: 'has-changes'`, `type: 'custom'`, `sections: []`, default `afterSubmit`, default `theme`.
5. Builder opens immediately.

**Defaults for new form:**
- `visibility: 'public'`
- `enableCsrf: true`
- `allowMultipleSubmissions: true`
- `analyticsEnabled: true`
- `defaultLocale: tenant.defaultLocale`
- `theme: tenant.defaultFormTheme`

### 30.6 Empty States

No forms exist: "You haven't created any forms yet." + "Create your first form" button.

No search results: "No forms match your search." + "Clear filters" link.

---

## 31. FORM THEMING SYSTEM

### 31.1 Overview

Every form has a `theme: FormTheme`. Built from design tokens. A form can use the tenant's default theme, override specific tokens, or use a fully custom theme.

### 31.2 FormTheme Type

```typescript
interface FormTheme {
  mode: 'inherit' | 'light' | 'dark';  // 'inherit' uses tenant/system theme
  tokens: Partial<ThemeTokens>;          // base token values (applied in all modes)
  darkTokens: Partial<ThemeTokens> | null; // dark-mode-only overrides. See §70.
  // Effective dark-mode tokens: { ...tokens, ...(darkTokens ?? {}) }
}

interface ThemeTokens {
  // Colours
  colorPrimary: string;
  colorPrimaryText: string;
  colorDanger: string;
  colorDangerText: string;
  colorBackground: string;
  colorSurface: string;
  colorBorder: string;
  colorText: string;
  colorTextMuted: string;

  // Typography
  fontFamily: string;
  fontSizeBase: string;
  fontSizeLabel: string;
  fontWeightLabel: number;
  lineHeight: number;

  // Shape
  borderRadius: string;
  borderWidth: string;
  inputHeight: string;
  inputPaddingX: string;
  inputPaddingY: string;

  // Spacing
  fieldGap: string;
  sectionGap: string;

  // Submit button
  submitButtonColor: string;
  submitButtonTextColor: string;
  submitButtonBorderRadius: string;
}
```

### 31.3 SectionStyle Type

```typescript
interface SectionStyle {
  backgroundColor: string | null;
  borderStyle: 'none' | 'all' | 'left-accent' | 'rounded';
  borderColor: string | null;
  padding: 'xs' | 'sm' | 'md' | 'lg' | 'xl';
}
```

### 31.4 FieldStyle Type

```typescript
interface FieldStyle {
  width: 'full' | 'half' | 'third' | 'quarter' | 'auto' | string;
  labelPosition: 'top' | 'left' | 'hidden';
  labelFontSize: string | null;
  labelFontWeight: number | null;
  labelColor: string | null;
  inputSize: 'sm' | 'md' | 'lg';
  borderStyle: 'default' | 'none' | 'underline' | 'rounded' | 'pill';
  focusColor: string | null;
  errorColor: string | null;
  prefixIcon: string | null;
  suffixIcon: string | null;
  customCss: string | null;  // RBAC: form.field.customCss
}
```

### 31.5 Tenant Default Theme

`tenant.defaultFormTheme: FormTheme` configured in Settings > Appearance > Forms. The `FormTheme` type (§30.2) contains both `tokens: Partial<ThemeTokens>` (base light-mode token values) and `darkTokens: Partial<ThemeTokens> | null` (dark-mode overrides). New forms inherit both `tokens` and `darkTokens` from the tenant default (see §70.5). The `mode` field of the tenant's stored `FormTheme` is treated as a template default — it is applied as the new form's initial `mode` but can be overridden per form. Individual forms override tokens without affecting others.

### 31.6 Theme in Builder

- **Theme Panel:** Toolbar > Theme (paint bucket icon). Token categories as collapsible sections.
- **Live preview:** Token changes reflected on canvas instantly.
- **Reset to tenant default:** Clears all overrides.
- **Light/Dark preview toggle.**
- **A11y contrast checker:** Inline contrast ratio next to colour inputs. Red warning if below minimum.

---

## 32. VALIDATION RULE SCHEMA

### 32.1 ValidationRule Type

```typescript
interface ValidationRule {
  id: string;
  type: ValidationRuleType;
  params: ValidationRuleParams;  // type-specific (see 31.2)
  message: LocalisedString;
  enabled: boolean;              // default true
}

type ValidationRuleType =
  | 'required' | 'min' | 'max' | 'pattern' | 'email' | 'url'
  | 'phone' | 'unique' | 'match' | 'custom' | 'ai-classify';
```

### 32.2 ValidationRuleParams by Type

```typescript
type ValidationRuleParams =
  | { type: 'required' }
  | { type: 'min'; value: number }
  | { type: 'max'; value: number }
  | { type: 'pattern'; regex: string; flags?: string }
  | { type: 'email'; allowedDomains?: string[] }
  | { type: 'url'; allowedProtocols?: string[] }
  | { type: 'phone'; allowedCountries?: string[] }
  | { type: 'unique'; endpoint: string; auth?: { type: 'bearer'; token: string } }
  | { type: 'match'; targetFieldName: string }
  | { type: 'custom'; endpoint: string; method: 'GET' | 'POST'; auth?: { type: 'bearer'; token: string } }
  | { type: 'ai-classify'; allowedCategories: string[]; rejectMessage: LocalisedString };

// auth tokens are stored encrypted server-side and never exposed to clients. See §53.6.
```

---

## 33. FIELDOPTION & DYNAMIC OPTION SOURCES

### 33.1 FieldOption Type

```typescript
interface FieldOption {
  value: string;
  label: LocalisedString;
  disabled: boolean;
  icon: string | null;
  group: string | null;  // optional group label for grouped dropdowns
}
```

### 33.2 Static Options

`FieldOption[]` stored on `FormField.options`. Admin adds/edits/reorders in Properties Panel > Options tab.

### 33.3 OptionSource — API-Backed Dynamic Options

```typescript
interface OptionSource {
  type: 'api';
  url: string;                  // HTTPS only. {{ }} expressions for query params.
  method: 'GET' | 'POST';
  headers: Record<string, string>;
  auth: OptionSourceAuth | null;
  responseMapping: {
    optionsPath: string;        // JSON path to options array. e.g. 'data.items'
    valuePath: string;          // e.g. 'id'
    labelPath: string;          // e.g. 'name'
    disabledPath?: string;
  };
  triggerOn: 'load' | 'focus' | 'dependency';
  dependencyField: string | null;
  cacheSeconds: number;         // 0 = no cache. Default: 60.
  searchParam: string | null;   // user search query appended as this param name
  debounceMs: number;           // default 300
}

type OptionSourceAuth =
  | { type: 'bearer'; token: string }
  | { type: 'header'; headerName: string; value: string };
```

**OptionSource fetch architecture:** OptionSource requests are made **server-side** (proxied through the application server). The client never calls the option source URL directly. This enables:
1. SSRF protection (see §61).
2. Auth credentials kept server-side, never exposed to client.
3. Cache coherence managed server-side.

**Cache:** The server caches option source responses keyed by `tenantId + url + formId` (per-form, not shared across forms). Cache duration: `cacheSeconds`. If `cacheSeconds: 0`: no cache. On form publish where `url` is changed: the cached entry for the previous URL (for this formId) is invalidated immediately. Two different forms sharing the same OptionSource URL maintain independent cache entries and do NOT share a cache.

### 33.4 Dynamic Option Loading States

| State | Description |
|-------|-------------|
| idle | Not yet loaded. |
| loading | Server-side fetch in progress. Spinner shown. |
| loaded | Options available. |
| error | Fetch failed. Show static `options` if any; else 'Could not load options. Try again.' + retry button. |
| empty | Fetch succeeded, 0 options returned. 'No options available.' |

### 33.5 SSRF Protection

OptionSource URLs are subject to the same SSRF protection rules defined in **§61 (Security Policy Registry)**. Summary: HTTPS only, no private IPs (RFC 1918), no localhost, no link-local. Validated at save time (builder warns) and at request time (server blocks). Violation: `form.optionsource.ssrf_blocked` audit logged. Refer to §61 for the authoritative and complete SSRF policy — this section does not restate the full policy.

---

## 34. EXPRESSION ENGINE — GRAMMAR, SANDBOX & ERROR HANDLING

### 34.1 Overview

Expressions are used in: field default values, `set-value` conditions, after-submit redirect URLs, `trigger-api` body templates and headers, `OptionSource` URL parameters, and email template bodies.

### 34.2 Expression Grammar

Expressions use `{{ }}` delimiters.

| Construct | Example | Notes |
|-----------|---------|-------|
| Variable access | `{{user.email}}` | Dot-notation only. |
| Arithmetic | `{{field.quantity * field.price}}` | +, -, *, / |
| Ternary | `{{field.plan == "pro" ? 29 : 9}}` | Single level only. No nesting. |
| String concatenation | `{{"Hello, " + user.firstName}}` | + for strings. |
| Comparison | `{{field.age >= 18}}` | ==, !=, >, <, >=, <= |
| Logical | `{{field.a && field.b}}` | && and \|\| |
| Default / fallback | `{{user.company \|\| "Unknown"}}` | Right side if left is null/empty. |

**Forbidden:** function calls (except built-ins), loops, assignments, `eval`, `Function`, `import`, `require`, prototype access, `window`, `document`, `process`.

**Built-in functions:**

| Function | Description |
|----------|-------------|
| `upper(str)` | Uppercase |
| `lower(str)` | Lowercase |
| `trim(str)` | Trim whitespace |
| `len(val)` | Length of string or array |
| `round(n, places)` | Round to N decimal places |
| `floor(n)` | Floor |
| `ceil(n)` | Ceil |
| `now()` | Current ISO datetime string |
| `today()` | Current ISO date string |
| `formatDate(date, format)` | Format date. Tokens: YYYY, MM, DD, HH, mm. |
| `sum(field, subFieldName)` | Sum a numeric sub-field across all rows of a Repeater field. Example: `sum(field.lineItems, "qty")`. Returns 0 if Repeater is empty. Returns null and logs a warning if the sub-field is non-numeric. See §69. |
| `count(field)` | Count the number of rows in a Repeater field. Returns 0 if empty. |
| `min(field, subFieldName)` | Minimum value of a numeric sub-field across Repeater rows. |
| `max(field, subFieldName)` | Maximum value of a numeric sub-field across Repeater rows. |

### 34.3 Sandbox

- Server-side sandbox (isolated VM, no filesystem/network/global scope).
- Max evaluation time: 50ms. Exceeded → aborted, treated as error.
- Max expression string length: 500 characters. Rejected at save time.
- Builder validates syntax on blur.

### 34.4 Error Handling

| Error | At build time | At runtime |
|-------|---------------|------------|
| Syntax error | Builder inline error. Cannot save. | N/A — blocked at save. |
| Unknown variable | Builder warning. Publish allowed. | Resolves to empty string. Warning logged. |
| Arithmetic error (divide by zero, NaN) | N/A | Resolves to `null`. Warning logged. |
| Timeout (>50ms) | N/A | Resolves to `null`. Warning logged. |
| Expression too long | Builder error. Cannot save. | N/A. |

---

## 35. SUBMIT DEDUPLICATION & ONCE-PER-USER ENFORCEMENT

### 35.1 Overview

Forms with `allowMultipleSubmissions: false` enforce one-submission-per-user. Only available on authenticated/role-gated forms.

### 35.2 Check Flow

1. On form load (authenticated user): server checks for existing `real` submission for `formId + submittedBy`.
2. If prior submission exists:
   - Form rendered in **locked state**: all fields read-only, showing prior submission values.
   - Banner: multipleSubmissionsErrorMessage (or default).
   - Submit button hidden.
   - 'Edit your submission' link shown if user holds form.submissions.edit RBAC.
3. No prior submission: renders normally.

### 35.3 Server Enforcement

Server checks `formId + submittedBy` uniqueness before storing. Duplicate: HTTP 409 `{ code: 'duplicate_submission', message: multipleSubmissionsErrorMessage }`.

### 35.4 Edge Cases

| Case | Behaviour |
|------|-----------|
| User's submission was deleted by admin | Deduplication lifted. User can resubmit. |
| Test submissions | `type: 'test'` excluded from deduplication. |
| Public forms | `allowMultipleSubmissions: false` not available. Builder disables toggle with tooltip. |
| Payment failure | A failed payment (Step 6 in §48) aborts the submission lifecycle BEFORE the submission record is created (Step 7). Therefore, a failed payment does NOT create a submission record and does NOT trigger deduplication. The user is free to correct their payment details and resubmit. This is the correct and expected behavior. |
| Prior submission has `paymentResult.status: 'failed'` | This can only occur if an admin manually edited the submission record post-creation. In this case, the record DOES exist and deduplication WILL block resubmission. Admin must delete the record (or toggle `allowMultipleSubmissions: true`) to allow the user to try again. |

---

## 36. PUBLISH PRE-FLIGHT CHECKS

### 36.1 Overview

Server performs pre-flight checks before publishing. Hard blocks reject publish. Soft warnings require admin acknowledgement.

### 36.2 Hard Blocks

| Check | Description |
|-------|-------------|
| Circular condition dependency | Circular field condition chain. |
| Unsafe redirect target | `redirect` action resolves to a disallowed external host, protocol-relative URL, credential-bearing URL, or `javascript:` / `data:` style target. |
| Duplicate field names | Two fields with same `name` key. |
| Invalid expression syntax | Field default or set-value expression has invalid syntax. |
| Required protected field removed | System-protected field removal attempt (server-side guard). |
| Password field on persisted non-auth form | Password fields are allowed only on auth/security system workflows. Non-auth persisted forms may not collect passwords. |
| Formula references non-existent field | Payment formula references unknown field name. |
| Formula directly references Repeater field | Payment formula uses `{{field.repeaterName}}` directly instead of an aggregation function. |
| OptionSource URL is HTTP | Non-HTTPS option source URL. |
| Missing defaultLocale content | A `LocalisedString.default` is empty. |
| CAPTCHA unconfigured | A CAPTCHA field exists but no CAPTCHA provider is configured in tenant settings. |
| show-next-step on flat form | An AfterSubmitAction of type `show-next-step` exists on a form where `steps` is null. |

### 36.3 Soft Warnings

| Warning | Description |
|---------|-------------|
| Missing translations | Fields have empty translations for a supported locale. |
| Orphaned condition reference | Condition references a deleted field. |
| Conflicting conditions | A field has contradictory show/hide conditions. |
| Hidden required protected field | System-protected field has a hide condition. |
| DataBinding to unverified entity | DataBinding's entity property not found during simulation. |
| No after-submit action | Form has no AfterSubmit actions. User will see blank screen after submitting. |
| No CAPTCHA on high-traffic public form | Advisory only. |
| Visibility change with active drafts | Changing visibility from authenticated/role-gated to public will disable save-and-resume and delete N in-progress drafts. Advisory — admin must acknowledge. |
| Visibility change with deduplication records | Changing visibility to public disables allowMultipleSubmissions:false and existing deduplication records become dormant (not deleted). Advisory — admin must acknowledge. |
| Active sessions on publish | N users are currently filling out this form. They will be notified of the update. (Approximate, based on recent render requests.) |

### 36.4 Publish Flow

1. Admin clicks 'Publish'.
2. Server runs hard-block checks.
3. Hard blocks found: publish rejected. Modal lists issues with links. Admin must fix.
4. No hard blocks, soft warnings exist: confirmation modal. Admin checks: "I understand these warnings and want to publish." Then confirms.
5. No issues: publish proceeds immediately.
6. On success: `publishedVersion` incremented, `draftStatus` → `clean`, `publishedStatus` → `published`, `formSnapshot` stored, `changesSummary` generated, cache invalidated, `fb.form.published` audit logged.

---

## 37. FORM ARCHIVING

### 37.1 What Archiving Does

Sets `publishedStatus: 'archived'`.

| Effect | Description |
|--------|-------------|
| Form URL returns 410 | End users see: 'This form is no longer accepting submissions.' |
| No new submissions | POST rejected with HTTP 410. |
| Builder access preserved | Admins can open form (read-only). |
| Submissions preserved | All existing submissions retained, viewable, exportable. |
| System forms cannot be archived | Error: 'System forms cannot be archived.' |
| Analytics preserved | Historical analytics retained. No new events collected after archiving. |
| Slug reserved | The archived form's slug remains reserved. It cannot be reused by a new form until the archived form is deleted. |

### 37.2 Unarchiving

Admin sets back to `published` (RBAC: `fb.form.publish`) or `draft` (RBAC: `fb.form.edit`). Does not affect submission records.

### 37.3 How to Archive

Toolbar > More > Archive. Confirmation: "Archiving this form will immediately prevent new submissions. Existing submissions are not affected." RBAC: **`fb.form.archive`** (distinct from `fb.form.delete` — see §1.1 Rule on RBAC and §13.5).

### 37.4 Archive vs. Deletion

| | Archive | Delete |
|-|---------|--------|
| Accessible in builder | Yes (read-only) | No |
| Submissions preserved | Yes | No |
| URL | 410 | 404 |
| Slug released | No (reserved until deleted) | Yes |
| Reversible | Yes | No (soft-delete has 30-day recovery) |

---

## 38. UNDO / REDO

### 38.1 Scope

| In Scope | Out of Scope |
|----------|-------------|
| Field additions and deletions | Published versions |
| Field property changes | Settings changes (visibility, CORS, rate limits) |
| Field reordering | Template instantiation |
| Section additions, deletions, reordering | Form name changes |
| Section property changes | |
| Column layout changes | |
| Multi-field bulk moves | |

### 38.2 Stack

- Max depth: **100 operations**. Oldest dropped when exceeded.
- **In-memory only.** Not persisted. Tab close/refresh clears stack.
- Autosave creates a server-side recovery checkpoint but does NOT truncate the undo stack.

### 38.3 Keyboard Shortcuts

- `Cmd/Ctrl + Z` — Undo
- `Cmd/Ctrl + Shift + Z` — Redo
- `Cmd/Ctrl + Y` — Redo (alternate)

### 38.4 UI

- Undo/Redo buttons in toolbar. Greyed out when empty.
- Tooltip shows next operation: "Undo: Add field 'Email'"
- Unsaved changes indicator remains active if undone operations still differ from last saved state.

---

## 39. BUILD WARNINGS PANEL

### 39.1 Location

Toolbar → Build Warnings badge (count). Clicking opens slide-in panel from right edge.

### 39.2 Warning Categories

| Category | Examples |
|----------|---------|
| Missing translations | 'Field "Company Name" is missing a French translation.' |
| Orphaned conditions | 'Field "Tax ID" has a condition referencing deleted field "countryOld".' |
| Conflicting conditions | 'Field "Phone" has both show and hide conditions that may both be satisfied.' |
| Hidden required protected fields | 'Protected field "Email" has a hide condition.' |
| A11y issues | 'Field "File Upload" has a hidden label but no aria-label alternative.' |
| Formula warnings | 'Payment formula references "planPrice" which could be null if the Plan field is hidden.' |
| Unverified DataBindings | 'DataBinding on "First Name" could not resolve property "user.givenName".' |
| No after-submit action | 'This form has no after-submit actions. Users will see a blank screen.' |
| show-next-step on flat form | (Hard block — prevents publish, does not remain as a warning.) |
| Visibility change impact | 'Changing to public will disable save-and-resume and delete N drafts.' |

**Relationship to Publish Pre-Flight (§36):** Build Warnings and Pre-Flight Checks share the same warning computation engine. The Build Warnings panel is a real-time superset of the soft warnings checked at publish time. Hard blocks (§36.2) are not shown as persistent warnings in the panel — they appear only at publish time as blocking errors. This means: if the warnings panel is empty, pre-flight soft checks will also pass; hard block checks run only at publish time.

### 39.3 Warning Behaviour

- Recalculated on every save.
- Each warning has a **'Fix' link** to the relevant field or section.
- Informational — do not prevent saving (unless elevated to hard-block per §36.2).
- Toolbar badge shows count. If 0: badge hidden.
- Persist until resolved. Cannot be manually dismissed (except publish-acknowledgement per §35.4).

---

## 40. TRIGGER-EMAIL TEMPLATE SYSTEM

### 40.1 Overview

`trigger-email` after-submit action sends emails. Rendered from named templates managed in Settings > Email Templates > Forms.

### 40.2 Template Entity

```typescript
interface FormEmailTemplate {
  id: string;
  tenantId: string;
  name: string;
  slug: string;               // referenced in TriggerEmailConfig.templateSlug
  subject: LocalisedString;   // ALWAYS comes from the template. No per-action override.
  bodyHtml: LocalisedString;
  bodyText: LocalisedString;
  createdAt: Date;
  updatedAt: Date;
}
```

### 40.3 Available Template Variables

All `{{ }}` expressions from §33 available, plus:

| Variable | Description |
|----------|-------------|
| `{{submission.id}}` | Submission UUID |
| `{{submission.formName}}` | Form name |
| `{{submission.submittedAt}}` | ISO datetime of submission |
| `{{field.fieldName}}` | Value of any field (by field name) |
| `{{submitter.email}}` | Submitter email (null for anonymous) |
| `{{submitter.firstName}}` | Submitter first name |
| `{{tenant.name}}` | Tenant name |
| `{{tenant.logoUrl}}` | Tenant logo URL |
| `{{form.directUrl}}` | Direct URL to the form |
| `{{submissionsUrl}}` | Admin URL to submission detail |

### 40.4 Built-in Templates

| Slug | Description |
|------|-------------|
| `form-confirmation` | Sent to submitter. Subject: "Thanks for your submission." Subject supports `{{ }}` expressions — e.g. `"Thanks for submitting {{submission.formName}}."` |
| `form-admin-notification` | Sent to admin. Includes all field values. |
| `form-digest` | Scheduled digest. Sent by the Digest Delivery Engine (§62), NOT as an AfterSubmitAction. Lists submissions received in the digest window. This template is used by the background digest job, not by the trigger-email action. |
| `form-refund-confirmation` | Sent to submitter when a refund is issued. Includes refund amount, currency, and reason. See §68 for refund notification actions. |

Admins can edit built-in templates or create custom ones. `{{ }}` expression syntax (§33) is supported in both subject and body fields of all templates.

### 40.5 trigger-email Config Schema

```typescript
interface TriggerEmailConfig {
  to: 'submitter' | 'admin' | 'specific' | 'field';
  toAddress?: string;            // if to='specific'
  toField?: string;              // if to='field': field name whose value is the recipient email
  templateSlug: string;          // references FormEmailTemplate.slug
  additionalVariables?: Record<string, string>; // extra {{ }} variables
  // NOTE: subject always comes from the template. There is no subject override in this config.
}
```

### 40.6 Email Delivery

Sent via EMAIL_BRANCH. Delivery failures logged. Submission stored regardless. Admin notified of failures per SCAN-08.

---

## 41. WEBHOOK REGISTRY

> **Canonical Definition:** GeneralApp §23.7 (shared infrastructure). FormBuilder extends with form-specific delivery logic.

### 41.1 Overview

`trigger-webhook` sends submission payload to a registered endpoint using GeneralApp's shared webhook infrastructure. Endpoints must be registered and verified. Distinct from `trigger-api` (§28), which is an ad-hoc unregistered call.

### 41.2 WebhookEndpoint Entity

> **Base Schema:** GeneralApp §23.7 `webhook_endpoints` table

```typescript
// FormBuilder uses GeneralApp §9 WebhookEndpoint interface
// FormBuilder-specific delivery configuration:
interface FormWebhookConfig {
  webhookId: string;           // References GeneralApp webhook_endpoints.id
  payloadFormat: 'standard' | 'minimal';  // Form-specific: full submission vs minimal
  includePaymentData: boolean; // Whether to include paymentResult in payload
  customHeaders?: Record<string, string>; // Additional headers for this form
}
```

**FormBuilder Event Subscriptions:**
FormBuilder automatically subscribes registered webhooks to form events based on configuration:
- `fb.submission.received` — When form is submitted
- `fb.submission.processed` — After after-submit actions complete
- `fb.payment.succeeded` — When payment is confirmed

### 41.3 Registration Flow

1. Settings > Webhooks → '+ Add Webhook'.
2. Enter name and HTTPS URL.
3. System generates a signing secret. Shown **once** — must be copied. The raw secret is stored encrypted at rest and never returned again. A non-sensitive fingerprint is retained for identification and audit.
4. System sends verification POST: `{ type: 'webhook.verify', challenge: '<random-string>' }` signed with secret.
5. Endpoint must respond HTTP 200: `{ challenge: '<same-string>' }` within 10 seconds.
6. Verified: status → `active`. Failed: stays `unverified`. Admin can re-trigger.

### 41.4 Delivery Payload

```json
{
  "type": "form.submission",
  "timestamp": "2026-03-10T14:00:00Z",
  "formId": "...",
  "formSlug": "contact-us",
  "submissionId": "...",
  "data": { "name": "Jane", "email": "jane@example.com" },
  "submittedBy": "user_123",
  "paymentResult": null
}
```

`X-Form-Signature` header: `sha256=<HMAC-SHA256 of raw body using secret>`.

### 41.5 Delivery Failure & Status

| Status | Trigger |
|--------|---------|
| active | Verified and delivering. |
| failing | 3+ consecutive delivery failures. Admin notified. |
| disabled | Admin disabled, or auto-disabled after 10 consecutive failures. |

**Webhook retry policy:** Non-2xx responses or timeout trigger retry: exponential backoff, 3 attempts over 5 minutes. **Retry-on status codes for webhooks:** 429, 500, 502, 503, 504. 4xx responses (except 429) are treated as permanent failures and are NOT retried — a 4xx indicates a client-side misconfiguration that retrying will not fix. All retries exhausted: `processingStatus` → `'failed'` or `'partial'`. Admin notified.

### 41.6 Delivery Log

Each attempt logged: timestamp, HTTP status, response body (truncated 500 chars), latency, attempt number. Retained 30 days.

---

## 42. CAPTCHA TENANT CONFIGURATION SURFACE

### 42.1 Location

Settings > Security > CAPTCHA.

### 42.2 Configuration Options

| Setting | Description |
|---------|-------------|
| Provider | Default: Cloudflare Turnstile. Alternative: hCaptcha. Integrated through the shared `captcha.verify` capability so the provider remains swappable per tenant. |
| Site key | Public key for client-side rendering. |
| Secret key | Server-side verification key. Encrypted storage. Displayed as '••••••' after save. Rotatable. |
| Fail-open on provider error | Toggle. Default: off (fail-closed). Field: `captchaFailOpenOnProviderError` on tenant CAPTCHA config. See §27.4. |
| Test bypass enabled | Toggle. Default: on. Global (not per-form). |

### 42.3 Unconfigured CAPTCHA Behaviour

CAPTCHA field added but no provider configured: builder shows error banner on field. Publishing is **hard-blocked** until CAPTCHA is configured or the CAPTCHA field is removed. (Elevated from soft warning to hard block per §36.2.)

### 42.4 Provider Change

Changing provider: all CAPTCHA fields on all forms immediately use new provider and site key. No per-form changes required.

---

## 43. FORM RENDER ENGINE — SERVING ARCHITECTURE & CACHING

### 43.1 Architecture

Separate service from Form Builder API, reading from same database. Responsibilities:
- Serving form pages at `/forms/{slug}`
- SSR rendering → React hydration
- Handling form submission POSTs

Builder unavailability has zero impact on Render Engine.

### 43.2 Form Load Flow

1. GET `/forms/{slug}`.
2. CDN edge cache check. Hit: serve cached response.
3. Cache miss: fetch published `FormDefinition` from database.
4. Server-side pre-fill resolution (DataBinding, expression defaults) for requesting user.
5. SSR HTML rendered with pre-filled values.
6. Cache-Control headers set. CDN caches response.
7. Client hydrates React component.

### 43.3 Cache Strategy

| Layer | TTL | Invalidation |
|-------|-----|-------------|
| CDN edge cache | 60s for public forms; no cache for authenticated (user-specific pre-fill) | On publish: immediate CDN purge via CDN purge API for form's slug. |
| Server-side FormDefinition cache (in-process) | 30s | Invalidated on publish by pub/sub event to all Render Engine instances. |
| No-JS fallback pages | Cached separately at CDN. Same invalidation. | |

### 43.4 Form Unavailability

Database unreachable on cache miss:
- Last successfully cached version served (if in local memory), with banner: "This form may be temporarily outdated."
- No local cache: HTTP 503 with Retry-After header.

---

## 44. SUBMISSION EXPORT — FORMAT SPECIFICATION

### 44.1 Export Formats

CSV and JSON.

### 44.2 CSV Export

**Column ordering:**
1. `submission_id`
2. `submitted_by` (user ID or 'anonymous')
3. `submitted_at` (ISO 8601)
4. `form_version`
5. `schema_note` (populated only if cross-version inconsistencies exist)
6. One column per field, ordered by field position in **latest** version.
7. Removed fields at end: `[field_name] (removed in v{N})`.

**Field value serialisation:**

| Field Type | CSV Serialisation |
|------------|------------------|
| Text, Number | Raw value |
| Date Picker | ISO date string |
| DateTime Picker | ISO datetime string |
| Date Range | `{fieldName}_start`, `{fieldName}_end` sub-columns |
| Checkbox (boolean) | `true` or `false` |
| Checkbox Group, Multi-select | Pipe-separated: `option1\|option2` |
| Address | Sub-columns per §46.3 |
| File Upload | Comma-separated filenames. No URLs. |
| Payment | `{fieldName}_amount`, `{fieldName}_currency`, `{fieldName}_status` sub-columns |
| Repeater | One CSV row per entry. Parent fields repeated. `repeater_row_index` column. |
| Signature | `[signature]` placeholder. Not exported. |
| Rating | Number |
| Slider | Number |
| IP address | NOT included in CSV exports. |

**Payment data in exports:** Card number and CVC never included. Only: amount, currency, status, last 4 digits, payment method type.

**Volume limits:** 
- **≤ 1,000 rows:** Synchronous export. Download URL returned immediately in the response.
- **> 1,000 rows and ≤ 50,000 rows:** Asynchronous export. HTTP 202 returned immediately. Admin notified by email with download link when ready (single file).
- **> 50,000 rows:** Asynchronous export. Split into multiple files of up to 50,000 rows each. Admin notified by email with links to all files when ready.
- **At exactly 1,000 rows:** Treated as ≤ 1,000 (synchronous).
- **At exactly 50,000 rows:** Treated as ≤ 50,000 (single async file, not split).

### 44.3 JSON Export

Each submission matches `FormSubmission` entity shape. File upload fields contain `StoredFile.id` (not URLs). `ipAddress` is included only for users with `fb.submissions.export.elevated` permission. In the seeded role model this is restricted to `system_owner` by default and may be delegated to `super_admin`. See §13.5 for the elevated export permission definition.

**Signature fields in JSON export:** Signature fields are exported as a signed URL to the signature image (valid 15 minutes, generated at export time), NOT as the raw base64 data URL. This prevents submissions with Signature fields from inflating the export file by 50–500 KB each (see §58.2). Signed URL format: `{ "type": "signed_url", "url": "https://...", "expiresAt": "ISO-datetime" }`. Consumers must download the image within the URL's validity window.

```json
[
  {
    "id": "sub_123",
    "submittedBy": "user_456",
    "submittedAt": "2026-03-10T14:00:00Z",
    "formVersion": 3,
    "data": { "name": "Jane", "email": "jane@example.com" },
    "paymentResult": { "amount": 2900, "currency": "usd", "status": "succeeded", "last4": "4242" }
  }
]
```

---

## 45. REST API REFERENCE

### 45.1 Authentication

**Admin API (§44.2 - §44.3):** Requires `Authorization: Bearer <token>`. Tenant context derived from authenticated identity. Used by form builders and administrators.

**Public API (§44.4):** No authentication required for endpoints marked **Public**. Used by form respondents. Some endpoints support optional authentication for personalized experiences (e.g., pre-filling known user data).

| Endpoint Category | Auth Required | Token Source | Used By |
|-------------------|---------------|--------------|---------|
| Admin CRUD (§44.2) | Yes | Supabase Auth JWT | Form builders, admins |
| Submissions Admin (§44.3) | Yes | Supabase Auth JWT | Admins, analysts |
| Form Render/Submit (§44.4) | Optional/Public | None or session cookie | Form respondents |
| File Upload (§44.4) | Optional | Pre-signed URL | Form respondents |

### 45.2 Forms Endpoints

| Method | Endpoint | Description | RBAC |
|--------|----------|-------------|------|
| GET | `/forms` | List forms. Query: `status`, `type`, `q`, `page`, `perPage`, `sort`. | `fb.form.view` |
| POST | `/forms` | Create form. Body: `{ name, slug?, type, fromTemplate?, fromFormId? }`. | `fb.form.create` |
| GET | `/forms/:id` | Get form definition (draft). | `fb.form.view` |
| PATCH | `/forms/:id` | Update form. `If-Match: <etag>` required. | `fb.form.edit` |
| DELETE | `/forms/:id` | Soft-delete. | `fb.form.delete` |
| POST | `/forms/:id/publish` | Publish draft. Runs pre-flight (§35). | `fb.form.publish` |
| POST | `/forms/:id/archive` | Archive form. | `fb.form.archive` |
| POST | `/forms/:id/unarchive` | Restore archived form to published or draft state. Body: `{ targetStatus: 'published'\|'draft' }`. | `fb.form.publish` or `fb.form.edit` |
| POST | `/forms/:id/duplicate` | Duplicate. Returns new FormDefinition. | `fb.form.duplicate` |
| GET | `/forms/:id/versions` | List version history. | `fb.form.view` |
| GET | `/forms/:id/versions/:v` | Get specific version snapshot. | `fb.form.view` |
| POST | `/forms/:id/versions/:v/restore` | Restore as new draft. | `fb.form.edit` |
| POST | `/forms/:id/lock` | Acquire edit lock. | `fb.form.edit` |
| PUT | `/forms/:id/lock` | Renew lock heartbeat. | `fb.form.edit` |
| DELETE | `/forms/:id/lock` | Release lock. | `fb.form.edit` |

### 45.3 Submission Endpoints

| Method | Endpoint | Description | RBAC |
|--------|----------|-------------|------|
| GET | `/forms/:id/submissions` | List. Query: `page`, `perPage`, `sort`, `q`, `type`, `status`. | form.submissions.view |
| GET | `/forms/:id/submissions/:subId` | Get one. | form.submissions.view |
| PATCH | `/forms/:id/submissions/:subId` | Edit data. Body: `{ data: { fieldName: value } }`. | form.submissions.edit |
| DELETE | `/forms/:id/submissions/:subId` | Delete. | form.submissions.delete |
| POST | `/forms/:id/submissions/:subId/actions/re-run` | Re-run actions. Body: `{ actionIndexes?: number[] }` — omit to re-run all. Payment is never re-charged. Conditions re-evaluated. Write-back NOT re-triggered. | form.submissions.edit |
| POST | `/forms/:id/submissions/export` | Request export. Body: `{ format: 'csv'\|'json', filters? }`. Returns download URL or 202. | fb.submissions.export |
| POST | `/forms/:id/submissions/:subId/refund` | Issue refund. Body: `{ type: 'full'\|'partial', amount?: number, reason?: string }`. RBAC: form.submissions.edit. See §60. | form.submissions.edit |
| GET | `/forms/:id/submissions/:subId/export/pdf` | Export single submission as PDF. Returns `application/pdf`. Synchronous. See §65. | form.submissions.view |

### 45.4 Public Render / Submission Endpoints

These endpoints are **PUBLIC** (no authentication required) unless marked otherwise. They are used by form respondents.

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/forms/:slug` | **Public** | Render form page (SSR). Returns HTML. |
| POST | `/forms/:slug/submit` | **Public** | Submit form. Rate limited by IP. Returns 200, 400, 409, 413, 422, 429. |
| GET | `/api/forms/:id/csrf-token` | **Public** | CSRF token for embedded forms. CORS-enabled. |
| POST | `/forms/:formId/fields/:fieldId/upload-url` | **Public*** | Pre-signed upload URL. *Requires valid form session. |
| POST | `/forms/:formId/fields/:fieldId/upload-complete` | **Public*** | Notify upload complete; begin virus scan. *Requires valid form session. |
| GET | `/api/forms/:id/qr-code` | **Public** | QR code image. Query: `format=png\|svg`. No auth required. See §63. |
| POST | `/forms/:id/test` | **Admin** | Test submission. Requires `Authorization` header + `X-Form-Test-Run: true`. |

---

## 46. ANALYTICS PII & GDPR HANDLING

### 46.1 What Analytics Captures

Behavioural events only — never field values:
- Form view (page view)
- Form start (first interaction)
- Field interaction (focus, blur, error count)
- Step transition
- Form submission (success/failure)
- Session duration

Analytics events do NOT capture field values, user identity (session-based anonymous ID only), or data from `sensitive: true` fields.

### 46.2 Sensitive Field Handling

Fields with `sensitive: true`: excluded from all analytics. No interaction events, no drop-off events, not shown in field-level analytics reports.

Auto-marked sensitive: Password, Payment card input sub-fields.

### 46.3 GDPR & Analytics

Analytics stored with anonymous session ID — not tied to user account. GDPR user deletion does NOT trigger analytics deletion (events are anonymous). Analytics data retained **90 days** then auto-purged.

### 46.4 Analytics vs. dataRetentionDays

`dataRetentionDays` governs submission records only. Analytics events have their own 90-day cycle, independent of submission retention.

---

## 47. ADDRESS FIELD — ADDRESSOBJECT TYPE

### 46.1 AddressObject Type

```typescript
interface AddressObject {
  line1: string;
  line2: string | null;
  city: string;
  state: string | null;    // State, county, province. Null for countries with no state concept.
  postcode: string | null; // Null for countries without postcodes.
  country: string;         // ISO 3166-1 alpha-2. e.g. 'US', 'GB', 'DE'.
}
```

### 46.2 Address Field Behaviour

- `country` sub-field: always a dropdown. All ISO 3166-1 alpha-2 countries.
- Selecting `country`: `state` sub-field updates to country-specific states/provinces (static built-in dataset).
- `postcode` validated per country (client-side and server-side, built-in library).
- Sub-fields individually configurable (required/optional) in Properties Panel > Sub-Fields tab.
- `line2` optional by default. All others required by default.

### 46.3 Address in Exports

CSV: flattened to `{fieldName}_line1`, `{fieldName}_line2`, `{fieldName}_city`, `{fieldName}_state`, `{fieldName}_postcode`, `{fieldName}_country`.

---

## 48. PAYMENTRESULT TYPE

### 47.1 PaymentResult Type

```typescript
interface PaymentResult {
  chargeId: string;
  status: PaymentStatus;
  amount: number;          // smallest currency unit (e.g. cents)
  currency: string;        // ISO 4217. e.g. 'usd', 'gbp'
  paymentMethod: PaymentMethodType;
  last4: string | null;
  cardBrand: string | null;
  billingName: string | null;
  billingEmail: string | null;
  billingAddress: AddressObject | null;
  receiptUrl: string | null;
  savedMethodId: string | null;
  errorCode: string | null;
  errorMessage: string | null;
  processedAt: Date;
  refunds: PaymentRefund[];  // see §60 for refund handling
}

type PaymentStatus =
  | 'pending'           // Awaiting payment completion
  | 'succeeded'         // Payment completed successfully (may map to a module-specific terminal success state)
  | 'failed'            // Payment attempt failed
  | 'refunded'          // Full refund issued
  | 'partially_refunded'// Partial refund issued
  | 'disputed'          // Chargeback/dispute in progress
  | 'requires_action'   // Additional authentication needed (3DS)
  | 'cancelled';        // Payment cancelled before completion

// Example mapping into a product-module payment table:
// PaymentResult (FormBuilder)    | product payment status
// -------------------------------|--------------------------------
// 'succeeded'                    | 'paid'
// 'failed'                       | 'failed'
// 'refunded'                     | 'refunded'
// 'partially_refunded'           | 'partially_refunded'
// 'pending'                      | 'pending'
// 'cancelled'                    | 'cancelled'

type PaymentMethodType = 'card' | 'bank_transfer' | 'apple_pay' | 'google_pay' | 'link';

interface PaymentRefund {
  refundId: string;            // Provider refund ID
  amount: number;              // Amount refunded (smallest currency unit)
  reason: string | null;       // Admin-entered reason
  refundedBy: string;          // Admin user ID
  refundedAt: Date;
  status: 'pending' | 'succeeded' | 'failed';
}
```

### 47.2 PaymentResult in Submissions

`FormSubmission.paymentResult` is `null` for forms without a Payment field. Populated on successful payment. On payment failure: `status: 'failed'` with `errorCode` and `errorMessage`.

### 47.3 Security Constraints

Raw card numbers, CVCs, full PAN: NEVER stored or returned. Only `last4` and `cardBrand`.

CSV exports: `receiptUrl`, `savedMethodId`, `billingEmail`, `billingAddress` redacted by default. JSON exports include them for users with `fb.submissions.export.elevated` permission. In the seeded role model this is restricted to `system_owner` by default and may be delegated to `super_admin`. See §13.5.

---

## 49. SUBMISSION LIFECYCLE — AUTHORITATIVE EXECUTION ORDER

This section defines the single, canonical execution order for every form submission. All other sections that touch submission processing are subordinate to this ordering.

### 48.1 Execution Sequence

```
STEP 1: Rate limit check
  → HTTP 429 if exceeded. Processing stops.

STEP 2: CSRF token validation
  → HTTP 403 if invalid. Processing stops.

STEP 3: CAPTCHA verification
  → HTTP 400 if CAPTCHA fails or provider unreachable (fail-closed). Processing stops.
  → CAPTCHA is checked HERE — before validation — to reject bots before any server-side
     work is performed. This is the most effective bot gate position.
  → CAPTCHA is bypassed for test runs (§27.5).

STEP 4: Deduplication check (if allowMultipleSubmissions: false)
  → HTTP 409 if duplicate. Processing stops.

STEP 5: Server-side validation
  a. Re-evaluate step-skip conditions. Confirm which steps/fields are active.
  b. Re-evaluate all display conditions. Confirm which fields are hidden/shown.
  c. Re-evaluate set-value rules. Apply server-computed values. Client-submitted values for
     set-value target fields are overwritten.
     ⚑ set-value is applied FIRST within this step, before formula evaluation (see 5d).
  d. Re-evaluate payment formula (if Payment field active). Formula uses the server-applied
     set-value results from step 5c, not client-submitted values.
  e. Run all ValidationRules on all active, non-skipped fields.
  → HTTP 400 (validation errors), HTTP 422 (formula failure), if invalid. Processing stops.

STEP 6: Payment authorization / checkout / confirmation
  → If Payment field active and not already charged this session, the runtime resolves the configured payment mode:
     • embedded mode → `payment.create_intent`, then `payment.confirm_intent`
     • hosted mode → `payment.create_checkout`
  → If the provider confirms payment asynchronously, the confirmation path is validated via
     `payment.verify_webhook` before the submission is treated as paid.
  → Charge guard checked (§57) to prevent duplicate checkout creation / re-charge.
  → HTTP 402 / payment error if checkout creation or confirmation fails. Processing stops
     (submission not stored).

STEP 7: Store submission
  → FormSubmission record created. formSnapshot embedded. processingStatus: 'pending'.

STEP 8: DataBinding write-back
  → Entity API updated for all bound fields with writeBack: true (and writeBackCondition met).
  → Write-back failure: logged, admin notified. Does NOT prevent proceeding to step 9.
  ⚑ Do not use DataBinding write-back AND trigger-api to update the same entity property.
     See §18.6 for the conflict warning.

STEP 9: After-submit actions
  → processingStatus: 'processing'.
  → Actions executed in declared order (with delays).
  → Background actions (trigger-email, trigger-webhook, trigger-api) run asynchronously.
  → Visual actions (show-message, redirect, etc.) return to client immediately.
  → processingStatus updated by background worker after all async actions settle.
     Expected settlement time: within 5 minutes under normal load.
  → Final processingStatus: 'processed' | 'partial' | 'failed'.

STEP 10: HTTP 200 returned to client
  → Client executes synchronous after-submit actions (show-message, confetti, redirect, etc.)
```

### 48.2 Key Ordering Rules

- **CAPTCHA is verified BEFORE validation (Step 3).** This is the most effective bot gate — rejecting bots before any server-side validation work. Prior versions of this spec placed CAPTCHA after validation; that was corrected in v5.0 (see Appendix A, BL-01).
- **set-value rules are applied BEFORE payment formula evaluation** (both within Step 5). The formula uses server-applied set-value results, not client-submitted values. This ensures computed field values feed correctly into price calculations.
- **Payment happens BEFORE store submission.** A failed payment means no submission record is created, regardless of whether the active flow is embedded-intent or hosted-checkout.
- **Write-back happens AFTER store submission.** This ensures the submission exists even if write-back fails.
- **After-submit actions happen AFTER write-back.** They can reference `{{submission.id}}` because the record exists.
- **processingStatus is settled by a background worker** after all async after-submit actions complete or exhaust retries. The HTTP 200 is returned to the client (Step 10) before background actions finish. Expected settlement: within 5 minutes under normal load.

---

## 50. SUBMIT BUTTON — FULL SPECIFICATION

### 49.1 SubmitButtonConfig

```typescript
interface SubmitButtonConfig {
  label: LocalisedString;           // default: { default: 'Submit' }
  loadingLabel: LocalisedString;    // shown while processing. default: { default: 'Submitting…' }
  successLabel: LocalisedString | null; // shown briefly on success before next action fires. null = don't change label.
  position: 'left' | 'center' | 'right' | 'full-width'; // default: 'right'
  size: 'sm' | 'md' | 'lg';        // default: 'md'
  style: 'filled' | 'outlined' | 'ghost'; // default: 'filled'
  // Colour comes from theme.submitButtonColor and theme.submitButtonTextColor (§30.2)
  // unless overridden here:
  colorOverride: string | null;     // hex. null = use theme token.
  textColorOverride: string | null; // hex. null = use theme token.
  icon: string | null;              // optional icon identifier rendered before label
}
```

### 49.2 Submit Button States

| State | Description |
|-------|-------------|
| idle | Enabled. Showing `label`. |
| disabled | `aria-disabled="true"`. Shown when: (a) a payment is in the `processing` state, (b) step validation is still running, (c) form is in deduplication-locked state. |
| loading | Processing. Label switches to `loadingLabel`. `aria-busy="true"`. Spinner shown. Cannot be clicked again (double-submit prevented). |
| success | Briefly shows `successLabel` (if configured) before after-submit action fires. |
| error | Server returned an error. Returns to `idle` state with error displayed on the form. |
| preview-only | In preview mode: shows 'Preview Only'. Clicks are no-ops. |

### 49.3 Double-Submit Prevention

Once the submit button enters `loading` state:
- Button is `aria-disabled="true"` with pointer-events disabled.
- The form does not listen to further submission events.
- If somehow a second POST request arrives server-side within 5 seconds of the first: HTTP 429 with `{ code: 'submission_in_progress' }`.

### 49.4 Builder Configuration

Submit button is configured in:
- Properties Panel when no field/section is selected (form-level properties tab)
- Or Toolbar > Settings > Submit Button tab

Admin can change: label, loading label, success label, position, size, style, colour override.

---

## 51. LOCALE SELECTION — DETECTION & RESOLUTION

### 50.1 Locale Selection Logic

When a user loads a form, the Render Engine determines the locale to use for rendering. The selection follows this ordered priority:

```
1. Explicit locale URL parameter: ?locale=fr
   → Use if 'fr' is in form's supportedLocales.

2. User profile locale (authenticated users only):
   → Use if user has a locale preference set and it is in supportedLocales.

3. Browser Accept-Language header:
   → Iterate through Accept-Language preferences in order.
   → For each: check if it is in supportedLocales (exact match first, then language-only).
   → Use the first match.

4. Form's defaultLocale:
   → Used as final fallback if none of the above produce a match.
```

### 50.2 Locale Not in supportedLocales

If the detected locale is not in `supportedLocales`, the engine does NOT use it — it falls through to the next rule. This prevents serving a partially-translated form for locales the admin has not prepared.

### 50.3 Locale Switcher on Form

If the form has `supportedLocales.length > 1`, a locale switcher is rendered on the form. Position is configured via `FormDefinition.localeSwitcherPosition` (see §2.1 and §73): `'top-left' | 'top-right' | 'bottom-left' | 'bottom-right' | 'hidden'`. Default: `'top-right'`. Setting `'hidden'` suppresses the switcher even when multiple locales are supported (useful when locale is always determined programmatically). Selecting a locale via the switcher takes priority over all other detection rules (equivalent to explicit URL parameter).

### 50.4 Locale in Locale-Aware Preview

In preview mode (§11.1), the locale dropdown overrides all detection. The selected locale is applied without any fallback chain — the admin is explicitly selecting a locale to test.

---

## 52. SUPPORTING TYPES

This section defines all supporting types referenced throughout the document that do not belong to a specific section.

### 51.1 AfterSubmitResult

```typescript
interface AfterSubmitResult {
  actionType: AfterSubmitActionType;
  actionIndex: number;           // position in AfterSubmitConfig.actions[]
  status: 'succeeded' | 'failed' | 'skipped'; // 'skipped' when action condition not met
  executedAt: Date | null;
  duration: number | null;       // milliseconds
  error: string | null;          // error message on failure
  responseStatus: number | null; // HTTP status for webhook/api actions
  responseBody: string | null;   // truncated to 500 chars for webhook/api
  retryCount: number;            // number of retry attempts made
}
```

### 51.2 AdminNote

```typescript
interface AdminNote {
  id: string;
  submissionId: string;
  content: string;
  createdBy: string;             // admin user ID
  createdAt: Date;
  updatedAt: Date;
}
```

### 51.3 SubmissionFlag

```typescript
interface SubmissionFlag {
  id: string;
  submissionId: string;
  reason: string;                // admin-entered reason
  flaggedBy: string;             // admin user ID
  flaggedAt: Date;
  resolvedBy: string | null;
  resolvedAt: Date | null;
}
```

### 51.4 RateLimitPolicy

> **Canonical Definition:** GeneralApp §11.2 (unified base). FormBuilder extends with form-specific fields.

```typescript
// FormBuilder extends GeneralApp RateLimitPolicy
interface RateLimitPolicy extends BaseRateLimitPolicy {
  // Inherited from GeneralApp §11.2:
  // - key: string
  // - dimension: 'ip' | 'user' | 'session' | 'tenant' | 'api-key' | 'global'
  // - endpoint: string (form slug for form submissions)
  // - limit: number
  // - windowSeconds: number
  // - burstLimit: number
  // - planOverrides: { plan: string; limit: number }[]
  // - roleOverrides: { role: string; limit: number }[]
  // - action: 'block' | 'throttle' | 'warn'
  
  // FormBuilder-specific extensions:
  windowMinutes: number;         // Convenience: converted to windowSeconds (×60)
  by: 'ip' | 'user' | 'session' | 'tenant';  // Maps to 'dimension' in base
  exceededAction: 'block' | 'queue';          // Form-specific: queue submissions
  exceededMessage: LocalisedString;           // Custom message shown to form users
}
```

**Conversion Rules:**
- `windowMinutes` → `windowSeconds` × 60
- `by` → `dimension` (direct mapping)
- `exceededAction: 'queue'` → `action: 'queue'` (FormBuilder-specific action)

**Resolution Hierarchy (§13.1):**
```
System Default (GeneralApp §11.3)
      ↓ most restrictive wins
Tenant Policy (GeneralApp §4.5.1)
      ↓ most restrictive wins
Form Policy (FormBuilder §13)
      ↓ most restrictive wins
Field Policy (FormBuilder §4)
```

### 51.5 ConditionGroup

```typescript
// CANONICAL DEFINITION — referenced everywhere. Not re-defined elsewhere.
// A condition group without an 'action' field.
// Used in: AfterSubmitAction.condition, DataBinding.writeBackCondition, FormStep.conditions[]
// Evaluates to true or false against the form's submitted (and set-value processed) data.
interface ConditionGroup {
  logic: 'all' | 'any';
  conditions: ConditionClause[]; // same ConditionClause type as DisplayCondition
}
```

**Multiple ConditionGroups on a FormStep:** A `FormStep` has a `conditions: ConditionGroup[]` array. When multiple ConditionGroups are present, they are combined using **AND logic**: ALL ConditionGroups must evaluate to `true` for the step to be shown. If any ConditionGroup evaluates to `false`, the step is skipped. If `conditions` is empty, the step is always shown. This combining rule is at the FormStep level and is separate from each ConditionGroup's own internal `logic: 'all'|'any'` field.

### 51.6 FormStep

```typescript
interface FormStep {
  id: string;
  title: LocalisedString;
  order: number;
  conditions: ConditionGroup[];  // ALL ConditionGroups must evaluate to true for the step to be shown.
                                 // Empty array = step always shown.
                                 // Each ConditionGroup has its own internal 'all'|'any' logic.
                                 // The FormStep-level combining rule is always AND (all groups must pass).
  // Sections belonging to this step are found via FormSection.stepId === FormStep.id
}
```

### 51.7 RateLimitOverride

A per-submission or per-admin override to the form's rate limit policy. Applied by owners or via the admin API to temporarily bypass or adjust rate-limiting for a specific context.

```typescript
interface RateLimitOverride {
  id: string;
  tenantId: string;
  formId: string;
  targetType: 'ip' | 'user' | 'session'; // which dimension is overridden
  targetValue: string;                    // the IP, userId, or sessionId being overridden
  overrideType: 'exempt' | 'custom';
  customLimit: number | null;             // max submissions. null if overrideType: 'exempt' (unlimited).
  customWindowMinutes: number | null;     // null if exempt.
  reason: string;                         // admin-entered justification
  createdBy: string;                      // admin user ID
  createdAt: Date;
  expiresAt: Date | null;                 // null = permanent until revoked
}
```

RateLimitOverrides are evaluated at Step 1 of the submission lifecycle (§48) before the form's RateLimitPolicy. If an override applies: use override limits instead of policy limits. Overrides are managed in Settings > Rate Limits > Overrides (RBAC: `system_owner` by default; may be delegated to `super_admin`).

---

## 53. FIELD SERIALISATION REFERENCE

The canonical serialisation format for each field type in `FormSubmission.data`.

| Field Type | JSON value type | Example |
|------------|----------------|---------|
| Text Input | `string` | `"Jane Doe"` |
| Textarea | `string` | `"Long text..."` |
| Rich Text Editor | `string` (HTML) | `"<p>Hello <strong>world</strong></p>"` |
| Number | `number` | `42` or `3.14` |
| Password | `null` (operational-only; never persisted in standard `FormSubmission.data`) | `null` |
| Hidden | as configured | `"utm_source_value"` |
| Dropdown (single) | `string` | `"option_a"` |
| Dropdown (multi) | `string[]` | `["option_a", "option_b"]` |
| Radio Group | `string` | `"option_b"` |
| Checkbox | `boolean` | `true` |
| Checkbox Group | `string[]` | `["opt1", "opt3"]` |
| Toggle / Switch | `boolean` | `false` |
| Button Group | `string` | `"option_c"` |
| Combobox | `string` | `"Custom value"` |
| Date Picker | `string` (ISO date) | `"2026-03-15"` |
| Time Picker | `string` (HH:MM) | `"14:30"` |
| DateTime Picker | `string` (ISO 8601) | `"2026-03-15T14:30:00Z"` |
| Date Range | `{ start: string, end: string }` | `{ "start": "2026-03-01", "end": "2026-03-15" }` |
| File Upload | `StoredFile.id[]` | `["file_abc123", "file_def456"]` |
| Image Upload | `StoredFile.id` (single) | `"file_abc123"` |
| Avatar Upload | `StoredFile.id` (single) | `"file_abc123"` |
| Signature | `string` (base64 PNG data URL) | `"data:image/png;base64,..."` |
| OTP / Code Input | `string` | `"483920"` |
| Phone Number | `string` (E.164) | `"+12025550123"` |
| Address | `AddressObject` | `{ "line1": "123 Main St", "city": "Boston", "state": "MA", "postcode": "02101", "country": "US", "line2": null }` |
| Rating | `number` | `4` |
| Slider / Range | `number` | `75` |
| Colour Picker | `string` (hex) | `"#FF5733"` |
| Autocomplete | `string` | `"Boston"` |
| Repeater | `object[]` | `[{ "name": "Item 1", "qty": 2 }]` |
| CAPTCHA | `null` (token not stored) | `null` |
| Payment | `null` (PaymentResult stored separately) | `null` |
| Consent / Terms | `boolean` | `true` — `true` means the user checked the box. A value of `false` cannot be submitted if the field is required (server rejects). See §64 for full specification. |
| Layout fields (Heading, Divider, etc.) | `null` (not collected) | `null` |

---

## 54. ASYNC VALIDATION ENDPOINTS — SECURITY & ARCHITECTURE

### 54.1 Overview

Two validation rule types make outbound network calls: `unique` and `custom`. Both require careful security and architecture specification.

### 54.2 Execution Architecture — Server-Side Proxied

Both `unique` and `custom` validation endpoint calls are made **server-side** (not client-side). The flow:

1. User blurs a field (or submits form).
2. Client sends the field value to the application server: `POST /api/forms/:formId/fields/:fieldId/validate { value: "..." }`.
3. Application server calls the configured `endpoint` URL with the value.
4. Application server returns `{ valid: boolean, message?: string }` to client.

**Why server-side:** Prevents clients from seeing existence-check responses directly (leaks data); allows auth credentials to be kept server-side; enables SSRF protection.

### 54.3 SSRF Protection for Validation Endpoints

Async validation endpoints are subject to the same SSRF protection rules defined in **§61 (Security Policy Registry)**. Summary: HTTPS only, no private IPs (RFC 1918), no localhost, no link-local. Validated at save time (builder warns) and at call time (server blocks). Violations: `form.validation.ssrf_blocked` audit logged. See §61 for the full and authoritative SSRF policy.

### 54.4 unique Endpoint Contract

```
GET {endpoint}?value={urlEncodedValue}
Response: { "exists": boolean }
```

If `exists: true`: field fails unique validation with configured `message`. If endpoint returns non-2xx or times out: validation fails gracefully with message: "Could not verify uniqueness. Please try again."

### 54.5 custom Endpoint Contract

```
GET or POST {endpoint}
Body (POST): { "value": <fieldValue>, "formId": "<formId>", "fieldName": "<fieldName>" }
Query (GET): ?value={urlEncodedValue}&formId={formId}&fieldName={fieldName}
Response: { "valid": boolean, "message"?: string }
```

If `valid: false`: field fails with `message` from response (or configured fallback message). If endpoint unavailable: validation fails gracefully.

### 54.6 Auth for Validation Endpoints

Validation endpoints that require authentication can be configured with a bearer token in the ValidationRule params:

```typescript
| { type: 'unique'; endpoint: string; auth?: { type: 'bearer'; token: string } }
| { type: 'custom'; endpoint: string; method: 'GET' | 'POST'; auth?: { type: 'bearer'; token: string } }
```

Auth tokens are stored encrypted and never exposed to clients.

### 54.7 Rate Limiting for Validation Calls

Async validation calls (on-blur) are rate-limited at 10 calls per field per session to prevent abusive enumeration. On-submit validation is not additionally rate-limited beyond the form-level rate limit.

---

## 55. LIVE-PUBLISH MID-SESSION HANDLING

### 55.1 The Problem

A user begins filling out a form on version 3. While they are mid-fill, an admin publishes version 4 (which may add new required fields, change options, or remove fields the user has already filled in).

### 55.2 Detection

The Render Engine embeds the form's `publishedVersion` number in the initial page HTML as a data attribute. The hydrated React client polls `/api/forms/:id/version` every **60 seconds** (only while the form is visible and the tab is active). If the returned `publishedVersion` is higher than the client's loaded version: a version change event is triggered.

### 55.3 Handling Logic

On detection of a version change:

1. **Banner shown:** "This form has been updated. Refresh to see the latest version." with a 'Refresh' button and a 'Continue with current version' option.
2. **User chooses 'Refresh':** Page reloads. Already-entered field values are preserved in localStorage (keyed by form slug + field name + session ID) and restored after reload into fields that still exist in the new version.
3. **User chooses 'Continue':** They proceed with the old version client-side. On submission, the server accepts the submission against the version the user loaded (the `formVersion` in the submission payload is checked). If the submitted `formVersion` is not the current published version:
   - The server applies the OLD version's validation rules (from the `formSnapshot` for that version).
   - The submission is stored with `formVersion: 3` (the version the user loaded).
   - A `schema_note` is added to the submission: "Collected on v3 while v4 was live."
4. **Newly required fields in v4 not present in v3 session:** If a new required field was added in v4 and the user is continuing with v3, the new field is NOT validated (user never saw it). This is acceptable and expected behaviour.

### 55.4 Builder Warning

Admins are advised (soft warning in §35.3) when publishing a form that has currently-active sessions: "N users are currently filling out this form. They will be notified of the update." (Session count is approximate, based on recent form render requests.)

---

## 56. SAVE-AND-RESUME DRAFT EXPIRY

### 56.1 Draft TTL — Canonical Definition

Save-and-resume drafts are stored as `FormDraft` records. The `FormDraft` schema:

```typescript
interface FormDraft {
  id: string;
  formId: string;
  userId: string;            // the authenticated user who owns this draft
  tenantId: string;
  data: Record<string, SubmissionFieldValue>; // partial submission data; same key-value format as FormSubmission.data
  lastStepCompleted: number; // 0-based index of the last completed step. Used to determine resume position.
  lastActivityAt: Date;      // timestamp of last step completion (TTL anchor — see rule 1 below)
  expiresAt: Date;           // computed: min(lastActivityAt + 30 days, dataRetentionDays deadline)
  createdAt: Date;
}
```

Each draft has a TTL governed by the following rules:

1. **Base TTL:** 30 days from the last **step completion** (not from creation, not from mere form visits or page loads). Each time the user completes a step, the TTL is reset.
2. **Data retention cap:** If `FormDefinition.dataRetentionDays` is less than 30, the draft TTL is capped to `dataRetentionDays`. This prevents a draft outliving the form's configured data retention window.
3. **Result:** Effective TTL = `min(30 days, dataRetentionDays ?? 30 days)` from the last step completion.

This is the single authoritative definition. §14.4 cross-references here.

### 56.2 Expiry Behaviour

On TTL expiry: draft silently deleted by background job (hourly scan). The **user** is NOT notified (they will see a fresh form on next visit). The **admin** is NOT notified (draft expiry is low-priority infrastructure). Next visit: form renders fresh with no restore prompt.

### 56.3 Draft on Form Archiving

If a form is archived while active save-and-resume drafts exist: drafts are immediately invalidated (deleted). On the user's next visit, they receive a 410 form-unavailable page, not a resume prompt.

### 56.4 Draft on Form Deletion

If a form is deleted: all associated save-and-resume drafts are deleted in the same operation.

---

## 57. FILE UPLOAD IN MULTI-STEP — ABANDONMENT & RESUME RACE CONDITION

### 56.1 The Problem

A user uploads a file on step 2 of a 4-step form, then does not complete the form. The `StoredFile` record has `submissionId: null`. The 48-hour abandoned-upload purge (§23.4) will delete the file. If the user resumes within 48 hours, the file reference is still valid. If they resume after 48 hours, the file is gone.

### 56.2 Solution: Draft-Linked File Retention

When a file is uploaded while a save-and-resume draft is active, the `StoredFile.saveAndResumeDraftId` field is set to the draft's ID. Files linked to a draft are **NOT** purged by the 48-hour abandoned-upload rule. They are retained for as long as the draft is alive (governed by the draft TTL in §55).

### 56.3 On Draft Expiry

When a save-and-resume draft expires (or is discarded by the user choosing 'Start Over'), all `StoredFile` records linked to that draft via `saveAndResumeDraftId` are purged in the same operation.

### 56.4 On Resume After File Stale

If (through any edge case) a file referenced in a resumed draft has already been purged:
- The file field renders in an `error-stale` state: "Previously uploaded file is no longer available. Please upload again."
- The field is treated as empty. If the field is required, user must re-upload.
- No data loss in other fields — only the affected file field is reset.

### 56.5 File Upload Before Draft Exists

If a user uploads a file on a multi-step form before the save-and-resume draft record exists (i.e., before they complete the first step and trigger a draft save):
- The file is in the standard 48-hour abandoned-upload window.
- On first step completion (draft created): all `StoredFile` records from this session with `submissionId: null` and `saveAndResumeDraftId: null` uploaded in the last 48 hours are linked to the new draft (`saveAndResumeDraftId` updated).

---

## 58. PAYMENT RE-CHARGE GUARD IN MULTI-STEP

### 57.1 The Problem

A user completes payment on step 3 of a 5-step multi-step form. The payment succeeds (charge captured). The user then navigates back to step 1 and clicks 'Next' again, eventually reaching and re-submitting step 3. Without a guard, the card could be charged again.

### 57.2 PaymentChargeGuard

```typescript
interface PaymentChargeGuard {
  sessionId: string;          // browser session identifier
  chargeId: string;           // provider charge ID from the first successful payment
  chargedAt: Date;
  amount: number;
  currency: string;
  formId: string;
}
```

`PaymentChargeGuard` is stored server-side keyed by `sessionId + formId`. It is also embedded in `FormSubmission.paymentChargeGuard` for audit.

### 57.3 Guard Logic (Step 6 of §48)

Before starting the payment operation in submission step 6:
1. Server checks if a `PaymentChargeGuard` exists for `sessionId + formId`.
2. If guard exists AND `chargeId` matches a succeeded payment: **charge is skipped**. The existing `PaymentResult` from the prior charge is reused in the new submission.
3. If guard exists but the prior charge failed: a new charge attempt is made normally (user is retrying a failed payment).
4. If no guard: proceed with new charge.

### 57.4 Guard TTL

The `PaymentChargeGuard` is valid for **24 hours** from `chargedAt`. After 24 hours, the guard is invalidated (a new payment would be required on re-submission).

### 57.5 Guard After Successful Submission

On successful form submission: the `PaymentChargeGuard` record is deleted. Re-visiting the form URL in the same session will require a new payment if the user submits again (assuming `allowMultipleSubmissions: true`).

---

## 59. SUBMISSION PAYLOAD SIZE LIMITS

### 58.1 Maximum Payload

The maximum submission payload size is **10 MB** per submission. This limit applies to the HTTP POST body of the submission request.

### 58.2 Field Contribution to Payload Size

| Field Type | Typical size | Notes |
|------------|-------------|-------|
| Text/Number/Date fields | < 1 KB each | Negligible |
| Repeater | Scales with row count × sub-field count | Large Repeaters can grow quickly |
| Signature | 50 KB – 500 KB | Base64 PNG; largest contributor for non-file forms |
| File fields | Stored separately as StoredFile; only the ID is in payload | IDs are < 100 bytes |
| Serialised form with 200 fields | ~50 KB typical | Within limits |

### 58.3 Limit Enforcement

- Server enforces 10 MB limit on the raw HTTP body.
- Rejection: HTTP 413 with `{ code: 'payload_too_large', message: 'Submission exceeds the maximum allowed size.' }`.
- Client-side warning: the form render engine estimates payload size before submission. If the estimate exceeds **8 MB** (80% of limit), a warning is shown to the admin in the Build Warnings panel: "This form may produce large submissions. Consider reducing field count or Signature field resolution."

### 58.4 File Upload Exemption

File content itself is NOT part of the submission payload — it is uploaded separately via the pre-signed URL flow (§23.3). The submission payload only contains the `StoredFile.id` references. The 10 MB limit does not apply to file content.

---

## 60. SEARCH INDEX LATENCY

### 59.1 Indexing Architecture

Fields with `searchable: true` are indexed in a full-text search index (Elasticsearch or equivalent). Indexing is **asynchronous** — it happens in a background worker after the submission is stored.

### 59.2 Latency SLA

Indexed submissions are searchable within **5 seconds** of storage under normal load. Under high load, latency may extend to **30 seconds**. The index is never guaranteed to be real-time.

### 59.3 Search-During-Lag Behaviour

If a submission is searched for within the lag window (before indexing completes):
- The submission will NOT appear in search results.
- Direct access (by submission ID, by browsing the submissions list) is available immediately — search is the only affected surface.

### 59.4 Indexing Failure

If a submission fails to index (worker failure, Elasticsearch unavailable):
- The submission remains stored and accessible by direct browse/filter.
- Failed indexing is retried up to 3 times with exponential backoff.
- After 3 failures: logged as `form.search.index_failed`. Admin NOT notified (low-priority infrastructure event). Submission accessible via direct browse.

### 59.5 Re-indexing

Admins can trigger a full re-index of a form's submissions from the form's settings panel (Settings > Advanced > Re-index submissions). This is a background operation. Progress shown in settings panel. Used after recovering from Elasticsearch downtime.

---

## 61. REFUND HANDLING

### 60.1 Overview

Refunds apply to submissions with `paymentResult.status: 'succeeded'`. Refunds can be full or partial. They are initiated by admins from the submission detail view.

### 60.2 Initiating a Refund

1. Admin opens submission detail view.
2. Payment section shows 'Issue Refund' button (visible only if `paymentResult.status: 'succeeded'` or `'partially_refunded'` and user has form.submissions.edit RBAC).
3. Admin selects: Full refund / Partial refund (enters amount).
4. Admin enters optional reason.
5. Confirmation modal.
6. Server calls `payment.refund` MCP capability with `chargeId`, `amount`, `reason`.
7. Provider processes refund.

### 60.3 Refund States

| Outcome | Behaviour |
|---------|-----------|
| Refund succeeded | `PaymentResult.refunds[]` gets a new `PaymentRefund` entry. `paymentResult.status` updated: if fully refunded → `'refunded'`. If partially → `'partially_refunded'`. `form.submission.refunded` audit event logged. |
| Refund failed | Error shown to admin. Submission `paymentResult.status` unchanged. Failure logged. |
| Refund pending | Some providers (bank transfer) have async refunds. Status: `'pending'`. Webhook from provider updates status when settled. |

### 60.4 Refund and After-Submit Actions

Issuing a refund does NOT re-trigger the original after-submit actions. However, configurable **Refund Notification Actions** can be set up per form — see **§68** for full specification. These include: `trigger-email` (using the built-in `form-refund-confirmation` template or a custom one), `trigger-webhook`, and `trigger-api`. If no refund notification actions are configured, the admin must send a refund confirmation to the user manually.

### 60.5 Refund in Exports

CSV exports: payment sub-columns include a `{fieldName}_refunded_amount` column. JSON exports include the full `PaymentResult.refunds[]` array.

---

## 62. SECURITY POLICY REGISTRY — SSRF, CORS & CREDENTIAL RULES

This section is the single authoritative source for all outbound-call security policies. All other sections that make outbound calls (§28 trigger-api, §32 OptionSource, §53 async validation) reference here rather than restating the rules.

### 61.1 SSRF Protection Policy

Applies to: `trigger-api` URLs (§28), OptionSource URLs (§32), async validation endpoints (§53), and any other admin-configurable outbound URL.

| Rule | Detail |
|------|--------|
| HTTPS required | HTTP URLs rejected at save time. Build warning shown. Hard block at publish. |
| No private IPs | RFC 1918 addresses (10.x, 172.16–31.x, 192.168.x) blocked at call time against resolved IP. |
| No localhost | 127.0.0.1, ::1, `localhost` blocked. |
| No link-local | 169.254.x.x, fe80::/10 blocked. |
| DNS rebinding protection | IP resolved at call time (not at save time only). If DNS resolves to a blocked range at call time, the call is aborted even if it passed the save-time check. |
| Violation handling | Call aborted. Audit event logged with section-specific code (e.g. `form.triggerapi.ssrf_blocked`, `form.optionsource.ssrf_blocked`, `form.validation.ssrf_blocked`). Admin notified. |

### 61.2 Credential Storage Policy

All admin-configured secrets (auth tokens for trigger-api, OptionSource, async validation, webhook secrets) are:
- Stored encrypted at rest (AES-256-GCM or equivalent).
- Never returned to clients in API responses.
- Displayed as `••••••` in the builder after initial save.
- Rotatable without changing the associated configuration (regenerate/re-enter).

### 61.3 CORS Policy
See §2.5 (CorsOrigin type) and §23 (Embedded Forms) for the full CORS specification. Summary:
- Exact-match origins only. No wildcards. No path matching.
- `Access-Control-Allow-Credentials: true` only for public forms on explicitly listed origins.
- Auth-gated forms cannot be embedded on external origins.

---

## 63. DIGEST DELIVERY ENGINE

### 62.1 Overview

The Digest Delivery Engine is a background scheduled job, entirely separate from the submission lifecycle and AfterSubmitAction system. It is NOT an `AfterSubmitAction` type and does not appear in `AfterSubmitConfig`.

### 62.2 Configuration

Configured per form in Settings > Notifications (§13.3). Options:
- **Frequency:** `'hourly'` | `'daily-morning'` (08:00 tenant timezone) | `'daily-evening'` (18:00 tenant timezone) | `'none'`.
- **Recipients:** Admin email(s) or a specific email address. Same as notify-on-submit recipient config.
- **Template:** Uses `form-digest` built-in template or a custom template with slug `form-digest-*`.

### 62.3 Digest Job Execution

1. Scheduled job fires at configured frequency.
2. For each form with digest enabled: query submissions received since last digest delivery for this form.
3. Zero submissions in window: no email sent. `lastDigestAt` timestamp still updated.
4. One or more submissions: render `form-digest` template with:
   - `{{digest.count}}` — number of submissions in window.
   - `{{digest.submissions}}` — array of up to 10 submission previews (first 3 non-sensitive, non-hidden fields in field order, plus submitted_at and submittedBy).
   - `{{digest.windowStart}}` / `{{digest.windowEnd}}` — ISO datetimes.
   - `{{submissionsUrl}}` — admin link to submissions list filtered to the window.
5. Email sent via EMAIL_BRANCH.
6. `lastDigestAt` updated on the form's notification config record.

### 62.4 Failure Handling

Digest delivery failure: logged, retried once. If retry fails: admin notified via direct email (not via digest). Submission records not affected.

---

## 64. SHORT-LINK & QR CODE REGISTRY

### 63.1 Short-Link Generation

Short links are optional. Generated on demand from Settings > Sharing > Generate Short Link (RBAC: `fb.form.edit`).

```typescript
interface ShortLinkConfig {
  slug: string;         // e.g. 'abc12'. 5-8 chars, alphanumeric. Tenant-scoped unique.
  createdAt: Date;
  expiresAt: Date | null; // null = permanent.
}
```

- Stored as `FormDefinition.shortLink` (the slug string). Full short URL is `https://{tenant.shortDomain}/{slug}`.
- Short links resolve to `/forms/{formSlug}`. They do not bypass auth, rate-limits, or CORS rules.
- Short links for archived or deleted forms return 410 or 404 respectively.
- Admin can regenerate (creates a new slug; old slug immediately returns 404).
- Admin can set expiry date. Expired short links return 410.

### 63.2 QR Code Generation

QR codes are auto-generated for every form. The QR encodes the form's direct URL (`/forms/{slug}`).

- Generated on demand. Stored internally as `FormDefinition.qrCodeToken` (an opaque token used to construct the QR image endpoint).
- Served at: `GET /api/forms/:id/qr-code?format=png|svg`. Returns the QR image. No auth required.
- If the form is archived or deleted, the QR endpoint returns 410 or 404.
- If the form's slug changes, the QR code auto-updates (it encodes the current direct URL).
- Admin can download QR (PNG or SVG) from Settings > Sharing.

---

## 65. CONSENT / TERMS FIELD — FULL SPECIFICATION

### 64.1 Overview

The Consent / Terms field is a purpose-specific boolean checkbox for collecting explicit user agreement to terms, privacy policies, or consent statements. It is distinct from the generic Checkbox field (§4.2) in that it has a built-in link mechanism and specific required-validation semantics.

### 64.2 ConsentFieldConfig

```typescript
interface ConsentFieldConfig {
  linkText: LocalisedString;    // Text of the link. e.g. 'Terms of Service'
  linkUrl: string;              // Absolute URL to the terms document.
  linkOpensIn: 'same-tab' | 'new-tab'; // default: 'new-tab'
  labelTemplate: LocalisedString;
  // LocalisedString with a placeholder for the link.
  // e.g. { default: 'I agree to the {{link}}.' }
  // {{link}} is replaced at render time with the linked text.
}
```

### 64.3 States

| State | Description |
|-------|-------------|
| unchecked | Default. Value: `false`. |
| checked | User checked. Value: `true`. |
| link-clicked | User clicked the terms link. Tracking event fired (if analyticsEnabled). |
| required-error | User tried to submit without checking. Error: 'You must agree to the [Terms] to continue.' |

### 64.4 Validation

A Consent / Terms field with `required: true` (default) cannot be submitted with value `false`. Server-side: `{ field: 'consentFieldName', code: 'consent_required', message: ... }` if submitted `false`.

### 64.5 Serialisation

Serialised as `boolean` in `FormSubmission.data`. `true` = user consented. `false` values are rejected server-side for required fields and should never appear in stored submissions for required Consent fields.

### 64.6 CSV Export

Exported as `true` or `false` (same as Checkbox).

---

## 66. SINGLE-SUBMISSION PDF EXPORT

### 65.1 Overview

Admins can export individual submissions as formatted PDF documents from the submission detail view (§12.3). RBAC: form.submissions.view.

### 65.2 PDF Layout

The PDF is rendered using the submission's `formSnapshot` layout (not the current form definition). Layout:

- **Header:** Form name, submission ID, submitted_at, submittedBy (name or 'Anonymous').
- **Sections:** One block per FormSection in snapshot order.
- **Fields:** Label (from snapshot) followed by value. Types rendered as:
  - Text/Number/Date: Plain text.
  - Checkbox/Toggle: ✓ or ✗.
  - Checkbox Group / Multi-select: Bullet list of selected values.
  - Address: Formatted multi-line address.
  - File Upload: Filename(s) only. No download links (files require separate access).
  - Signature: Embedded signature image (PNG, from signed URL at export time). If signed URL expired or file purged: `[Signature not available]` placeholder.
  - Payment: Amount, currency, status, last4, payment method type. Refund details if any.
  - Repeater: Table with one row per entry.
  - Layout fields (Heading, Divider): Rendered as structural elements.
- **Footer:** Tenant name, export timestamp, `form.submissions.view` RBAC watermark if requested.

### 65.3 API Endpoint

`GET /forms/:id/submissions/:subId/export/pdf` — returns PDF binary with `Content-Type: application/pdf`. Synchronous for all submission sizes (PDF is single-submission, not a bulk export).

### 65.4 Sensitive Field Handling

Fields with `sensitive: true` are included in the PDF by default. Admin can optionally exclude them: query param `?excludeSensitive=true`.

---

## 67. GDPR REVIEW QUEUE

### 66.1 Overview

When a GDPR user-deletion request is processed, anonymous submissions (those with `submittedBy: null`) that contain the deleted user's email address in field data are flagged for admin review rather than auto-deleted (since the system cannot confirm they belong to the deleted user without identity linkage).

### 66.2 Queue Location

Settings > Privacy > GDPR Review Queue. RBAC: `system_owner` by default; may be delegated to `super_admin`.

### 66.3 Queue Entry

```typescript
interface GdprReviewEntry {
  id: string;
  submissionId: string;
  formId: string;
  formName: string;
  matchedFields: string[];      // field names where the deleted user's email was found
  deletedUserEmail: string;     // the email that triggered this review
  deletedUserId: string;
  flaggedAt: Date;
  status: 'pending' | 'redacted' | 'deleted' | 'dismissed';
  reviewedBy: string | null;
  reviewedAt: Date | null;
  notes: string | null;
}
```

This flag is DISTINCT from `SubmissionFlag` (§51.3), which is a general-purpose admin flag. GdprReviewEntry is privacy-law-specific and restricted to `system_owner` by default, with optional delegation to `super_admin`.

### 66.4 Available Actions

| Action | Description |
|--------|-------------|
| Redact | Replace matched field values with `[REDACTED]`. Submission record retained. |
| Delete | Hard-delete the submission and all associated StoredFiles. |
| Dismiss | Mark as reviewed with no action (e.g. the email match was coincidental). |

All actions require a confirmation modal. Logged as `form.submission.gdpr_reviewed` audit event with action, actor, and rationale.

### 66.5 Automated Scan

GDPR deletion triggers an async scan of all anonymous submissions for the deleted user's email across all field types (including address sub-fields and Repeater sub-fields). Scan runs in background. Admin notified when scan completes with count of flagged entries.

---

## 68. DRAFT & DEDUPLICATION STATE TRANSITIONS ON VISIBILITY CHANGE

### 67.1 Triggered By

Changes to `FormDefinition.visibility` that transition:
- `'authenticated'` or `'role-gated'` → `'public'`
- `'public'` → `'authenticated'` or `'role-gated'`

### 67.2 Authenticated/Role-Gated → Public

| Feature | Behaviour |
|---------|-----------|
| Save-and-resume | Automatically disabled. All in-progress drafts (FormDraft records) immediately invalidated and deleted. |
| allowMultipleSubmissions | If `false`: automatically set to `true` (not available on public forms). Existing deduplication records for prior authenticated submissions are preserved but become dormant — they do not affect new anonymous submissions. |
| Admin warning | Before saving: "Changing visibility to public will disable Save & Resume and permanently delete N in-progress drafts. allowMultipleSubmissions will be reset to true. This cannot be undone." Admin must confirm. |

### 67.3 Public → Authenticated/Role-Gated

| Feature | Behaviour |
|---------|-----------|
| Save-and-resume | Remains disabled until admin explicitly enables it. |
| allowMultipleSubmissions | Remains at its current value (default `true`). Admin can now enable deduplication. |
| Admin warning | None required. Transition is non-destructive. |

### 67.4 API Enforcement

`PATCH /forms/:id` with a visibility change triggers the transition logic server-side before saving. If the transition would delete drafts: the response includes `{ warnings: [{ code: 'drafts_deleted', count: N }] }` in addition to the updated FormDefinition.

---

## 69. REFUND NOTIFICATION ACTIONS

### 68.1 Overview

Configurable actions that fire when an admin issues a refund on a submission. Configured per form in Settings > Payments > Refund Notifications.

### 68.2 RefundNotificationConfig

```typescript
interface RefundNotificationConfig {
  actions: RefundNotificationAction[];
}

interface RefundNotificationAction {
  type: 'trigger-email' | 'trigger-webhook' | 'trigger-api';
  config: TriggerEmailConfig | TriggerWebhookConfig | TriggerApiConfig;
  condition: ConditionGroup | null; // evaluated against PaymentResult + submission data
  fireOn: 'full_refund' | 'partial_refund' | 'any_refund'; // default: 'any_refund'
}
```

Stored on `FormDefinition.refundNotificationConfig`.

### 68.3 Execution

Refund notification actions fire after `payment.refund` MCP returns successfully (§60.2 Step 7). They execute with the same retry and failure handling as regular after-submit actions. Failures: logged, admin notified. Do not affect the refund outcome.

### 68.4 Email Template Variables

When using `trigger-email` in a refund notification action, the following additional variables are available:

| Variable | Description |
|----------|-------------|
| `{{refund.amount}}` | Refunded amount in smallest currency unit |
| `{{refund.amountFormatted}}` | Human-formatted amount (e.g. '$29.00') |
| `{{refund.currency}}` | ISO 4217 currency code |
| `{{refund.reason}}` | Admin-entered reason (null if none) |
| `{{refund.refundedAt}}` | ISO datetime of refund |
| `{{payment.amount}}` | Original charge amount |
| `{{payment.status}}` | Updated payment status after refund |

The built-in `form-refund-confirmation` template (§40.4) uses these variables.

### 68.5 Builder Configuration

Settings > Payments > Refund Notifications. Same action builder UI as AfterSubmitConfig. Only `trigger-email`, `trigger-webhook`, and `trigger-api` are available (not redirect or UI actions).

---

## 70. REPEATER FIELD IN PAYMENT FORMULAS

### 69.1 Overview

Repeater fields serialize as `object[]`. They cannot be referenced directly in payment formulas using `{{field.repeaterName}}` — this will trigger a hard publish block (§36.2). Instead, use the aggregation functions defined in §70.

### 69.2 Supported Aggregation Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `sum` | `sum(field.repeaterName, 'subFieldName')` | Sum all numeric values of the named sub-field across rows. Returns 0 for empty Repeater. Returns null if sub-field is non-numeric. |
| `count` | `count(field.repeaterName)` | Number of rows in the Repeater. Returns 0 if empty. |
| `min` | `min(field.repeaterName, 'subFieldName')` | Minimum numeric sub-field value. Returns null if empty. |
| `max` | `max(field.repeaterName, 'subFieldName')` | Maximum numeric sub-field value. Returns null if empty. |

### 69.3 Examples

```javascript
// Sum line item quantities × unit price
formula: 'sum(field.lineItems, "quantity") * {{field.unitPrice}}'

// Count-based pricing (charge per row)
formula: 'count(field.attendees) * 25.00'

// Dynamic max-based pricing
formula: 'max(field.bids, "amount") * 0.05'
```

### 69.4 Null Handling in Formulas

If a Repeater aggregation returns `null` (empty Repeater with min/max, or non-numeric sub-field): the formula is treated as an evaluation failure. HTTP 422 returned. Submission not stored. Use a ternary guard: `count(field.items) > 0 ? sum(field.items, "price") : 0`.

### 69.5 Server Re-Evaluation

Like all payment formula components, Repeater aggregations are re-evaluated server-side on submission using the submitted Repeater data. Client-displayed amounts are display-only.

---

## 71. DARK-MODE THEME TOKEN SYSTEM

### 70.1 Overview

`FormTheme.mode: 'dark'` enables dark-mode rendering. The base `ThemeTokens` (§30.2) define light-mode values. Dark-mode token overrides allow specifying different token values for dark mode without duplicating the entire token set.

### 70.2 Updated FormTheme Type

```typescript
interface FormTheme {
  mode: 'inherit' | 'light' | 'dark';  // 'inherit' uses tenant/system theme
  tokens: Partial<ThemeTokens>;          // applied in all modes (base values)
  darkTokens: Partial<ThemeTokens> | null; // overrides for dark mode only. null = no dark-specific overrides.
}
```

When `mode: 'dark'`, the effective token set is: `{ ...tokens, ...(darkTokens ?? {}) }`. When `mode: 'inherit'`, the system/tenant theme controls both base and dark tokens.

### 70.3 Dark Mode Rendering

Dark mode is applied at the form container level via a CSS data attribute `data-form-theme="dark"`. All CSS custom properties (design tokens) are scoped to this attribute. This means a dark form can be embedded on a light page without affecting the host page's styles.

### 70.4 Builder Support

- Theme Panel (§30.6) shows a **Light/Dark toggle** at the top.
- When Dark is selected, Properties Panel shows `darkTokens` overrides alongside base token values.
- `darkTokens` that differ from `tokens` are highlighted with a moon icon in the Theme Panel.
- A11y contrast checker (§30.6) evaluates contrast for the active mode (light or dark separately).

### 70.5 Tenant Default Dark Theme

`tenant.defaultFormTheme` includes a `darkTokens` field. New forms inherit both `tokens` and `darkTokens` from the tenant default.

---

## 72. AUTOCOMPLETE FIELD — SOURCE SECURITY & ARCHITECTURE

### 71.1 Overview

The Autocomplete field (§4.6) uses the `OptionSource` schema (§32.3) for its data source, subject to the same server-side proxy architecture and SSRF protection rules (§61) as Dropdown/Select dynamic sources.

### 71.2 Autocomplete-Specific OptionSource Behavior

| Property | Autocomplete Behavior |
|----------|-----------------------|
| `triggerOn` | Always `'focus'` or `'dependency'` for Autocomplete. `'load'` is not supported (Autocomplete lists are fetched on user input, not on page load). |
| `searchParam` | Required for Autocomplete. The user's typed query is sent as this param. |
| `debounceMs` | Default 300ms. Min 100ms for Autocomplete to prevent excessive requests. |
| `cacheSeconds` | Results cached per search query string. Cache key: `tenantId + url + formId + queryString`. |
| `minChars` | Additional Autocomplete-specific config: minimum characters before triggering a fetch. Default: 2. |

### 71.3 Server-Side Proxy

Autocomplete fetches are proxied through the application server (same as OptionSource). The client never calls the source URL directly. Auth credentials are server-side only.

### 71.4 SSRF Protection

Same rules as §61. HTTPS only. No private IPs, no localhost, no link-local. Validated at save time and at fetch time.

### 71.5 Builder Configuration

In the Field Properties Panel, an Autocomplete field shows an 'Options Source' tab identical to the OptionSource tab for Dropdown fields (§32.3), with the addition of a `minChars` input (default 2).

---

## 73. CONCURRENT MULTI-TAB SUBMISSION GUARD

### 72.1 The Problem

A user opening the same form in two browser tabs simultaneously and submitting both could cause a TOCTOU (time-of-check/time-of-use) race condition on deduplication: both tabs pass the deduplication check (§48 Step 4) before either stores a submission (§48 Step 7). Without a guard, two submissions could be stored even when `allowMultipleSubmissions: false`.

### 72.2 Database-Level Deduplication Guard

For forms with `allowMultipleSubmissions: false`, the database enforces a **unique constraint** on `(formId, submittedBy)` for submissions with `type: 'real'` and `processingStatus != 'deleted'`. This is enforced at the database level (not just application level) to handle concurrent inserts.

On concurrent insert conflict: the second insert is rejected with a database unique constraint violation. The application catches this and returns HTTP 409 with `{ code: 'duplicate_submission' }` — same response as the application-level deduplication check.

### 72.3 Session-Level Guard (Client)

As an additional layer, the form client sets a `sessionStorage` flag `formSubmitting_{formId}` when the submit button enters the `loading` state. On the same tab: double-submit prevention (§49.3) handles this. On other tabs of the same browser session: the flag is visible, and the other tab's submit button enters a `duplicate-in-progress` state with banner: "A submission is already in progress from another tab."

### 72.4 Scope

This guard applies only to `allowMultipleSubmissions: false` forms. For `allowMultipleSubmissions: true` forms, concurrent submissions are permitted and no special handling is required.

---

## 74. LOCALE SWITCHER POSITION CONFIGURATION

### 73.1 Overview

The locale switcher renders on forms with `supportedLocales.length > 1`. Its position is controlled by `FormDefinition.localeSwitcherPosition` (see §2.1).

### 73.2 Allowed Values

| Value | Position |
|-------|----------|
| `'top-right'` | Top-right corner of the form container. **Default.** |
| `'top-left'` | Top-left corner. |
| `'bottom-right'` | Bottom-right corner. |
| `'bottom-left'` | Bottom-left corner. |
| `'hidden'` | Switcher not rendered. Locale determined by detection logic only (§50.1). |

### 73.3 Builder Configuration

In Settings > Internationalisation > Locale Switcher Position. Dropdown with the 5 options above. Live preview updates immediately in canvas.

### 73.4 A11y

The locale switcher renders as a `<select>` element with `aria-label="Select language"`. Screen readers announce it as a landmark. When the locale changes, an `aria-live="polite"` region announces: "Language changed to [locale name]." Focus is not moved on locale change — the user's current focus position is preserved.

---

## 75. APPENDIX A — V4.0 AUDIT ISSUE REGISTER

This appendix lists all 35 issues identified in the v4.0 audit and their resolution status (resolved in v5.0, carried forward to v6.0).

| ID | Severity | Category | Title | Resolution |
|----|----------|----------|-------|------------|
| BL-01 | CRITICAL | Broken Logic | CAPTCHA step order | Corrected: CAPTCHA moved to Step 3 (before validation) in §48. |
| BL-02 | CRITICAL | Broken Logic | Failed-payment + deduplication impossible scenario | Corrected: §34.4 now accurately reflects that failed payments create no submission record. |
| BL-03 | CRITICAL | Broken Logic | Rule 12 missing case (required:false + enabled rule) | Corrected: full precedence matrix in §5.2 and §1.1 Rule 12. |
| BL-04 | HIGH | Broken Logic | FormStep.conditions combining logic ambiguous/contradictory | Corrected: §51.6 and §51.5 now define AND-combining rule explicitly. |
| BL-05 | HIGH | Broken Logic | Save-and-resume draft TTL described inconsistently | Corrected: §55.1 is now the canonical definition; §14.4 cross-references it. |
| BL-06 | MEDIUM | Broken Logic | redirect-back in embedded form undefined fallback | Corrected: §8.6 clarifies fallback behavior when parent ignores the message. |
| BL-07 | MEDIUM | Broken Logic | show-next-step silently ignored on flat forms | Corrected: escalated to hard publish block in §9.12 and §36.2. |
| INC-01 | HIGH | Inconsistency | LocalisedString defined twice | Fixed: §2.3 now cross-references §17.2 as canonical. |
| INC-02 | HIGH | Inconsistency | ConditionGroup defined twice | Fixed: §6.1 now cross-references §51.5 as canonical. |
| INC-03 | HIGH | Inconsistency | ValidationRuleParams missing auth fields | Fixed: §31.2 updated with optional auth on unique and custom types. |
| INC-04 | MEDIUM | Inconsistency | CORS credentials flag rationale misleading | Fixed: §23.2 now explains the purpose clearly. |
| INC-05 | MEDIUM | Inconsistency | Export volume threshold boundary conditions missing | Fixed: §43.2 now defines exact boundary behavior at 1,000 and 50,000 rows. |
| INC-06 | MEDIUM | Inconsistency | IP address export handling contradictory | Fixed: §13.5 defines `fb.submissions.export.elevated`; §43.3 references it consistently. |
| INC-07 | LOW | Inconsistency | Webhook retry retryOn codes unspecified | Fixed: §41.5 now specifies retry-on status codes for webhooks. |
| ORP-01 | HIGH | Orphaned | form-digest template delivery mechanism unspecified | Fixed: §62 Digest Delivery Engine fully specifies the mechanism. |
| ORP-02 | HIGH | Orphaned | short-link and QR code have no generation/storage spec | Fixed: §63 Short-Link & QR Code Registry fully specified. FormDefinition schema updated. |
| ORP-03 | HIGH | Orphaned | Consent/Terms field in §52 but absent from Field Type Catalogue | Fixed: §4.6 updated; §64 Consent/Terms Field full specification added. |
| ORP-04 | MEDIUM | Orphaned | Single-submission PDF export never specified | Fixed: §65 Single-Submission PDF Export fully specified. |
| ORP-05 | MEDIUM | Orphaned | IP allowlist has no field in FormDefinition schema | Fixed: `ipAllowlist` added to FormDefinition schema in §2.1. |
| ORP-06 | MEDIUM | Orphaned | GDPR Review Queue mentioned but never defined | Fixed: §66 GDPR Review Queue fully specified. |
| ORP-07 | LOW | Orphaned | RateLimitOverride type listed in header but never defined | Fixed: Added to §51 Supporting Types and TOC. |
| ORP-08 | LOW | Orphaned | `fb.form.delete` used for archive — asymmetric naming | Fixed: `fb.form.archive` added as distinct RBAC permission in §1.2, §13.5, §36.3, §44.2. |
| NF-01 | CRITICAL | Not Factored In | Repeater fields in payment formulas unspecified | Fixed: §9.3 updated; §33 aggregation functions added; §69 full specification added. |
| NF-02 | CRITICAL | Not Factored In | No refund notification mechanism | Fixed: §69 Refund Notification Actions fully specified. `form-refund-confirmation` template added in §40.4. |
| NF-03 | HIGH | Not Factored In | Expression support in email subjects unspecified | Fixed: §17.2 usage list and §13.3 note both specify subject expression support. |
| NF-04 | HIGH | Not Factored In | Analytics handling on duplication unspecified | Fixed: §25.2 duplication table now explicitly states analytics data is not copied. |
| NF-05 | HIGH | Not Factored In | Visibility change impact on drafts/deduplication unspecified | Fixed: §14.4 updated; §67 Draft & Deduplication State Transitions on Visibility Change added. |
| NF-06 | HIGH | Not Factored In | Autocomplete field SSRF protection unspecified | Fixed: §4.6 updated with OptionSource reference; §71 Autocomplete Source Security & Architecture added. |
| NF-07 | HIGH | Not Factored In | Password field handling on shared forms unspecified | Fixed: §4.1, §12.1, and §52 now specify that password values are operational-only for auth/security workflows and are never persisted in standard `FormSubmission.data`. |
| NF-08 | MEDIUM | Not Factored In | Concurrent multi-tab submission race condition unspecified | Fixed: §72 Concurrent Multi-Tab Submission Guard added. |
| NF-09 | MEDIUM | Not Factored In | Locale switcher position setting has no schema field | Fixed: `localeSwitcherPosition` added to FormDefinition schema in §2.1; §73 added. |
| NF-10 | MEDIUM | Not Factored In | Submission to deleted form mid-session unspecified | Fixed: §16.3 error paths table now specifies HTTP 404 behavior for deleted forms. |
| NF-11 | MEDIUM | Not Factored In | Dark-mode has no token variants | Fixed: §70 Dark-Mode Theme Token System added; `darkTokens` added to FormTheme type in §30.2. |
| NF-12 | LOW | Not Factored In | Signature field in JSON export size concern unspecified | Fixed: §43.3 now specifies signed URL export (not raw base64) for Signature fields. |
| DUP-01 | MEDIUM | Duplication | Save-and-resume TTL stated in both §14.4 and §55.2 | Fixed: §55.1 is canonical; §14.4 cross-references. |
| DUP-02 | MEDIUM | Duplication | SSRF rules duplicated across §28.3, §32.5, §53.3 | Fixed: §61 is canonical; all three sections now reference §61. |
| DUP-03 | LOW | Duplication | Build warnings count described in §3.1 and §38 | Fixed: §3.1 callout removed; §38 is the single source. |
| INC-08 | HIGH | Incomplete | trigger-api responseMapping destination unspecified | Fixed: §28.4 now specifies values stored in FormSubmission.data. |
| INC-09 | HIGH | Incomplete | Re-run actions body schema unspecified | Fixed: §12.3 and §44.3 now specify body schema and constraints. |
| INC-10 | HIGH | Incomplete | processingStatus settlement timing unspecified | Fixed: §48.1 Step 9 and §48.2 now specify background worker settlement and ~5-minute SLA. |
| INC-11 | MEDIUM | Incomplete | OptionSource cache key scope unspecified | Fixed: §32.3 now specifies per-form cache key: `tenantId + url + formId`. |
| INC-12 | MEDIUM | Incomplete | Digest format and preview field selection unspecified | Fixed: §62 fully specifies digest window, field selection, and template variables. |
| INT-01 | HIGH | Interconnection Gap | set-value before formula evaluation order implicit only | Fixed: §48.1 Step 5c/5d and §48.2 now explicitly state set-value fires before formula. |
| INT-02 | HIGH | Interconnection Gap | DataBinding write-back and trigger-api can conflict | Fixed: §18.6 conflict warning added. |
| INT-03 | MEDIUM | Interconnection Gap | Analytics doesn't specify skipped step behavior | Fixed: §24.3 now specifies skipped steps shown as distinct 'skipped: N' status. |
| INT-04 | MEDIUM | Interconnection Gap | Build Warnings and Pre-Flight relationship undefined | Fixed: §38.2 now explicitly states shared computation engine and superset relationship. |
| INT-05 | MEDIUM | Interconnection Gap | localStorage restore not reconciled with deduplication | Fixed: §16.3 server 5xx error path now specifies silent deduplication check before restore prompt. |

---

## 76. APPENDIX B — V5.0 AUDIT ISSUE REGISTER

This appendix lists all 12 issues identified in the v5.0 audit and their resolution status in v6.0.

| ID | Severity | Category | Section(s) | Title | Resolution |
|----|----------|----------|-----------|-------|------------|
| STR-01 | HIGH | Structural | §51 (×2) | Duplicate §51 section heading | Fixed: orphaned `## 51. SUPPORTING TYPES (UPDATED)` heading removed from after Appendix A. `§51.7 RateLimitOverride` merged into the main §51 body. |
| SCH-01 | HIGH | Schema Omission | §2.1, §68 | `refundNotificationConfig` missing from FormDefinition schema | Fixed: `refundNotificationConfig: RefundNotificationConfig \| null` added to §2.1 FormDefinition interface under a `// Payments` comment. |
| SCH-02 | MEDIUM | Schema Inconsistency | §30.5, §70.5 | `tenant.defaultFormTheme` typed as `ThemeTokens` instead of `FormTheme` | Fixed: §30.5 updated to declare `tenant.defaultFormTheme: FormTheme`. Explanation of `tokens` + `darkTokens` inheritance added. |
| REF-01 | MEDIUM | Stale Cross-Reference | §8.1, §18.2 | ConditionGroup comments still referenced `§6.1, §51` instead of `§51.5` | Fixed: Both comments updated to reference `§51.5` as the canonical definition. |
| REF-02 | MEDIUM | Missing Cross-Reference | §12.4 | GDPR Review Queue mention had no §66 citation | Fixed: `See §66 for the full GDPR Review Queue specification.` appended to the relevant bullet in §12.4. |
| REF-03 | MEDIUM | Missing API Entry | §44.3 | PDF export endpoint absent from REST API reference table | Fixed: `GET /forms/:id/submissions/:subId/export/pdf` row added to §44.3. |
| REF-04 | MEDIUM | Missing API Entry | §44.4 | QR code endpoint absent from REST API reference table | Fixed: `GET /api/forms/:id/qr-code` row added to §44.4. |
| REF-05 | MEDIUM | Missing RBAC Entry | §1.2 SCAN-04 | `fb.submissions.export.elevated` missing from RBAC registry | Fixed: Permission added to SCAN-04 with description `(restricted to system_owner by default; delegable to super_admin; includes IP address in JSON exports — see §13.5)`. |
| DUP-04 | LOW | Incomplete Fix | §3.1, §74 | DUP-03 resolution falsely claimed; §3.1 Build Warnings callout still present | Fixed: The `> **Bottom bar vs. toolbar badge:**` callout block removed from §3.1. §38 remains the single source for Build Warnings behaviour. |
| SCH-03 | LOW | Schema Incompleteness | §55 | `FormDraft` entity referenced but never defined | Fixed: `FormDraft` TypeScript interface added to §55.1 with all fields: `id`, `formId`, `userId`, `tenantId`, `data`, `lastStepCompleted`, `lastActivityAt`, `expiresAt`, `createdAt`. |
| INC-13 | LOW | Incomplete Specification | §27.4, §41.2 | `captchaFailOpenOnProviderError` field name not cross-referenced in §41.2 | Fixed: §41.2 table row updated to include `Field: captchaFailOpenOnProviderError on tenant CAPTCHA config. See §27.4.` |
| INC-14 | LOW | Missing Cross-Reference | §47.3 | "Elevated export permission" unnamed in §47.3 | Fixed: §47.3 updated to read `fb.submissions.export.elevated permission (restricted to system_owner by default; delegable to super_admin). See §13.5.` |

---

## 77. APPENDIX C — CROSS-SYSTEM INTEGRATION REFERENCE

This appendix provides quick reference for Form Builder's integration points with consumer product modules and the General Application foundation.

> **Canonical Bridge Spec:** GeneralApp §23.9

### C.1 Example Product-Module Integration Points

Form Builder integrates with product modules via the **Form-to-Entity Bridge** defined in GeneralApp §23.9. The Form Builder engine provides the generic mechanism; product modules register handlers at runtime via `registerBridgeHandler()`.

| Feature | Form Builder (Generic) | Example Product Module | Integration Spec |
|---------|----------------------|--------------------------------------|------------------|
| **Entity Bridge** | `bridgeConfig` | Handler key: `'product-module-bridge'` | GeneralApp §23.9 |
| **Field Mapping** | `bridgeConfig.fieldMappings` | Target: module-owned entity fields | GeneralApp §23.9 |
| **Payment Result** | `PaymentResult` | Maps to module payment table | Consuming product spec |
| **File Storage** | `file_uploads` (GeneralApp §23.3) | optional consumer-record linkage | GeneralApp §23.3 |
| **Email Delivery** | `trigger-email` action | Uses `connectors` registry | GeneralApp §23.5 |
| **Custom Field Types** | `extensionHooks.customFieldTypes` | `'custom_record_key'`, `'catalog_selector'` | Consuming product spec |
| **Payment Adapter** | `extensionHooks.paymentAdapter` | `'custom-billing-split'` | Consuming product spec |

**Bridge Handler Registration Pattern (GeneralApp §23.9):**
```typescript
// Product module registers handlers at system initialization
registerBridgeHandler('product-module-bridge', {
  async execute(submission, config, context) {
    // Product-specific logic: create entity, field values, payments
    const entityId = await createProductEntityFromSubmission(submission, config);
    return { success: true, entityId, entityType: 'product_record' };
  }
});

// Additional extension hooks (product-specific)
registerFieldType({
  key: 'ext.product_module.custom_record_key',
  component: CustomRecordKeyFieldPlugin
});
registerPaymentAdapter({
  key: 'ext.product_module.custom_billing_split',
  adapter: CustomBillingSplitAdapter
});
```

### C.2 Payment Status Mapping

| FormBuilder (PaymentResult) | Example Product Module | Notes |
|-----------------------------|--------------------------|-------|
| `pending` | `pending` | Awaiting completion |
| `succeeded` | `paid` | Successful payment (mapped term) |
| `failed` | `failed` | Failed attempt |
| `refunded` | `refunded` | Full refund |
| `partially_refunded` | `partially_refunded` | Partial refund |
| `disputed` | `disputed` | Chargeback |
| `requires_action` | — | 3DS pending |
| `cancelled` | `cancelled` | User cancelled |

### C.3 General Application Integration (Shared Primitives)

Form Builder inherits all shared platform primitives from GeneralApp:

| Primitive | GeneralApp (Canonical) | Form Builder Usage |
|-----------|----------------------|-------------------|
| **Users & Auth** | §23.1 `users` table | Form Builder references `submittedBy` user IDs; delegates auth to Supabase Auth |
| **Sessions** | §23.2 `sessions` table | Builder edit locks, save-and-resume drafts reference session context |
| **File Storage** | §23.3 `file_uploads` table | `StoredFile` projection; uses `source_type='form_builder'` |
| **Audit Log** | §23.4 `audit_log` table | All form lifecycle events logged with `fb.*` action prefix |
| **MCP Connectors** | §23.5 `connectors` + `capability_bindings` | Payment, email, storage, CAPTCHA capabilities |
| **RBAC** | §23.6 `rbac_roles`, `role_permissions` | Form permissions (`fb.form.create`, `fb.form.edit`, etc.) |
| **Webhooks** | §23.7 `webhook_endpoints` | Form events delivered via shared infrastructure |

### C.4 Unified File Storage Schema

FormBuilder's `StoredFile` (§24) maps to GeneralApp §23.3 `file_uploads`:

```typescript
// FormBuilder StoredFile projection
{
  id,                    // → file_uploads.id
  tenantId,              // → file_uploads.tenant_id
  formId,                // → file_uploads.form_id
  submissionId,          // → file_uploads.submission_id (when linked)
  fieldName,             // → file_uploads.field_name
  originalName,          // → file_uploads.original_filename
  mimeType,              // → file_uploads.mime_type
  sizeBytes,             // → file_uploads.file_size_bytes
  storageKey,            // → file_uploads.storage_key
  scanStatus,            // → file_uploads.scan_status
  uploadedAt,            // → file_uploads.created_at
  deletedAt,             // → file_uploads.deleted_at
  saveAndResumeDraftId,  // → file_uploads.draft_id
}
```

### C.5 Webhook Event Namespacing

FormBuilder uses GeneralApp §23.7 webhook infrastructure. Event types are namespaced:

**FormBuilder events** (prefixed `fb.`):
| Event | Description |
|-------|-------------|
| `fb.submission.received` | Form submitted |
| `fb.submission.processed` | After-submit actions complete |
| `fb.form.published` | Form published |
| `fb.form.archived` | Form archived |
| `fb.payment.succeeded` | Payment processed |
| `fb.payment.failed` | Payment failed |
| `fb.refund.issued` | Refund processed |

**Consumer-specific bridge events** (example product-module prefix `pm.`):
| Event | Description |
|-------|-------------|
| `pm.record.created_from_form` | Product record created via FormBuilder bridge |
| `pm.payment.received_from_form` | Payment linked to product record |

### C.6 Settings Hierarchy Resolution

Form-level settings are resolved against tenant and system settings using GeneralApp §4.5.1 rules:

| Setting Type | Resolution Function | Rationale |
|--------------|---------------------|-----------|
| **Security settings** (rate limits, password policy) | `Math.min(system, tenant, form)` | Most restrictive wins |
| **Retention settings** (data retention, audit logs) | `Math.max(system, tenant, form)` | Longest that satisfies legal floor |

```
Form Setting (FormBuilder)
    ↓
Tenant Default (GeneralApp tenant_config)
    ↓
System Minimum (GeneralApp System Settings)
```

See GeneralApp §4.5.1 for complete resolution rules and API reference.

### C.7 Cross-Document Quick Reference

| Primitive | FormBuilder | Product Module | GeneralApp (Canonical) |
|-----------|-------------|---------------|------------------------|
| **Users** | `submittedBy` FK | `user_id` refs | §23.1 `users` |
| **Sessions** | Edit locks, drafts | API auth context | §23.2 `sessions` |
| **Files** | §24 `StoredFile` | `file_uploads` refs | §23.3 `file_uploads` |
| **Audit** | `fb.*` actions | `pm.*` actions | §23.4 `audit_log` |
| **Connectors** | Payment adapters | Product-specific integrations | §23.5 `connectors` |
| **RBAC** | `form.*` perms | product-module perms | §23.6 `rbac_roles` |
| **Webhooks** | §41 delivery | Consumer handlers | §23.7 `webhook_endpoints` |
| **Payments** | §47, §60 adapters | §17 business logic | — |
| **Notifications** | §40 templates | §10 delivery prefs | — |
| **Settings** | §13 form config | Tenant overrides | §4.5.1 resolution |

---

*End of Document — Version 6.1 (Clean Architecture Refactor)*

**Change Notes:**
- §2.1: `bridgeConfig` now uses generic `targetSchemaId`, `targetFieldKey`, `productConfig` (removed hardcoded product-module concepts)
- §2.1: Bridge field mappings use generic patterns (product-specific examples in consuming modules)
- §24: `virusScanStatus` updated to use `quarantined` state (unified contract with GeneralApp §23.3)
- Rule 16 (NEW): Auth/security system forms excluded from standard submission persistence
- Appendix C.3: Added GeneralApp §23 integration reference
- Appendix C.4-C.7: Updated cross-document references to use GeneralApp as canonical source for shared primitives

*Audit issue registers are tracked in Appendix A (§74) and Appendix B (§75). Cross-system integration references are summarized in Appendix C.*
