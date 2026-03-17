
# Comprehensive Element Configuration Flow

> **Foundation wiring note:** This file feeds the shared owner-configurable UI model in `GeneralApp`. Every meaningful element must trace through foundation-owned config, persistence, and runtime enforcement rather than decorative settings.
## Universal Pattern for Admin Configuration & RBAC Management

---

## Table of Contents
1. [Philosophy & Principles](#philosophy--principles)
2. [Configuration Architecture](#configuration-architecture)
3. [Element Categories](#element-categories)
4. [Universal Configuration Pattern](#universal-configuration-pattern)
5. [Admin Control Panel Structure](#admin-control-panel-structure)
6. [User Experience Flow](#user-experience-flow)
7. [RBAC Implementation](#rbac-implementation)
8. [State Management](#state-management)
9. [Examples by Category](#examples-by-category)

---

## Philosophy & Principles

### Core Principles
1. **Conditional Rendering, Not Hiding**: Elements that users don't have access to are NOT rendered at all (not just hidden with CSS/display:none)
2. **Granular Control**: Every aspect of every element must be configurable by admin
3. **Complete Flow Mapping**: Each element must have defined states, branches, and lifecycle
4. **Role-Based Everything**: Access, visibility, and functionality tied to user roles
5. **Default vs Override**: System defaults with role-specific overrides
6. **Audit Trail**: Every configuration change is logged with timestamp and admin user

### Design Philosophy
- Admin has god-mode control over every element
- Users see only what their role permits
- Every element follows the same configuration pattern
- Every element has enable/disable toggle
- Every element has role-based visibility
- Every element has customizable templates/content
- Every element has state management

---

## Configuration Architecture

### Three-Layer System

```
┌─────────────────────────────────────────────────────┐
│                   Layer 1: GLOBAL                    │
│              System-Wide Configurations              │
│  - SMTP Settings                                     │
│  - Default Role Permissions                          │
│  - Global Feature Toggles                            │
│  - Branding & Theme                                  │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│                   Layer 2: CATEGORY                  │
│           Element Category Configurations            │
│  - Email Templates (Authentication, Notifications)   │
│  - UI Components (Forms, Modals, Dashboards)        │
│  - Features (Exports, Imports, Analytics)           │
│  - Integrations (APIs, Webhooks, Services)          │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│                  Layer 3: ELEMENT                    │
│            Individual Element Configurations         │
│  - Specific Email Template                           │
│  - Specific UI Component                             │
│  - Specific Feature Module                           │
│  - Specific Integration                              │
└─────────────────────────────────────────────────────┘
```

---

## Element Categories

### 1. Communication Elements
- **Email Templates** (Authentication, Notifications, Marketing, Transactional)
- **SMS Templates**
- **Push Notifications**
- **In-App Notifications**
- **System Alerts**

### 2. UI Components
- **Forms** (Login, Registration, Profile, Settings)
- **Modals** (Confirmations, Alerts, Wizards)
- **Dashboards** (Admin, User, Analytics)
- **Navigation** (Menus, Sidebars, Breadcrumbs)
- **Data Tables** (Lists, Grids, Reports)

### 3. Feature Modules
- **Data Export** (CSV, Excel, PDF, JSON)
- **Data Import** (Bulk Upload, CSV Import)
- **Search & Filters**
- **Analytics & Reporting**
- **User Management**
- **Content Management**

### 4. Integration Elements
- **Third-party APIs**
- **Webhooks**
- **OAuth Providers**
- **Payment Gateways**
- **Storage Providers**

### 5. Security Elements
- **Authentication Methods** (Password, MFA, SSO, OAuth)
- **Authorization Rules**
- **Rate Limiting**
- **IP Whitelisting/Blacklisting**
- **Session Management**

### 6. Business Logic Elements
- **Workflows** (Approval Chains, Automation)
- **Scheduled Jobs** (Cron, Background Tasks)
- **Business Rules**
- **Validation Rules**

---

## Universal Configuration Pattern

### Every Element Must Have:

```javascript
{
  // === IDENTIFICATION ===
  id: "unique-element-id",
  category: "email|ui|feature|integration|security|business",
  type: "specific-type",
  name: "Display Name",
  description: "What this element does",

  // === CORE CONFIGURATION ===
  enabled: true|false,                    // Master on/off switch
  required: true|false,                   // Is this element mandatory?
  environment: ["dev", "staging", "prod"], // Which environments?

  // === ROLE-BASED ACCESS CONTROL ===
  rbac: {
    allowedRoles: ["admin", "manager", "user"], // Which roles can access
    deniedRoles: ["guest", "suspended"],        // Explicitly denied roles
    defaultRole: "user",                        // Default fallback
    inheritFromParent: true|false,              // Inherit category RBAC
    customPermissions: {
      view: ["admin", "manager", "user"],
      create: ["admin", "manager"],
      edit: ["admin", "manager"],
      delete: ["admin"],
      export: ["admin", "manager"]
    }
  },

  // === CONDITIONAL RENDERING ===
  renderConditions: {
    requiresAuth: true|false,
    requiresSubscription: "free|pro|enterprise",
    requiresFeatureFlag: "feature-name",
    customConditions: [
      {
        type: "userProperty",
        property: "emailVerified",
        value: true
      },
      {
        type: "dateRange",
        startDate: "2025-01-01",
        endDate: "2025-12-31"
      }
    ]
  },

  // === STATES & LIFECYCLE ===
  states: {
    draft: {
      enabled: true,
      allowedRoles: ["admin"],
      actions: ["edit", "preview", "publish"]
    },
    active: {
      enabled: true,
      allowedRoles: ["admin", "manager", "user"],
      actions: ["view", "use"]
    },
    inactive: {
      enabled: false,
      allowedRoles: ["admin"],
      actions: ["view", "activate"]
    },
    archived: {
      enabled: false,
      allowedRoles: ["admin"],
      actions: ["view", "restore", "delete"]
    }
  },
  currentState: "active",

  // === CONTENT/TEMPLATE ===
  content: {
    template: "template-id|raw-content",
    variables: {
      // Dynamic variables available
      user: ["name", "email", "role"],
      system: ["siteURL", "siteName", "date"],
      custom: ["key": "value"]
    },
    localization: {
      enabled: true,
      defaultLanguage: "en",
      availableLanguages: ["en", "es", "fr", "de"],
      fallbackToDefault: true
    }
  },

  // === SCHEDULING ===
  schedule: {
    enabled: true|false,
    timezone: "UTC",
    activeHours: {
      start: "09:00",
      end: "17:00"
    },
    activeDays: ["monday", "tuesday", "wednesday", "thursday", "friday"],
    blackoutDates: ["2025-12-25", "2025-01-01"]
  },

  // === RATE LIMITING & THROTTLING ===
  limits: {
    enabled: true|false,
    maxUsesPerUser: 100,
    maxUsesPerRole: {
      free: 10,
      pro: 100,
      enterprise: -1 // unlimited
    },
    resetPeriod: "daily|weekly|monthly",
    throttle: {
      enabled: true,
      requestsPerMinute: 60
    }
  },

  // === DEPENDENCIES ===
  dependencies: {
    requires: ["element-id-1", "element-id-2"], // Must be enabled
    conflicts: ["element-id-3"],                // Cannot coexist
    enhancedBy: ["element-id-4"]                // Optional enhancements
  },

  // === VALIDATION & RULES ===
  validation: {
    enabled: true,
    rules: [
      {
        type: "required",
        message: "This field is required"
      },
      {
        type: "custom",
        function: "validateFunction",
        message: "Custom validation failed"
      }
    ]
  },

  // === LOGGING & ANALYTICS ===
  logging: {
    enabled: true,
    logLevel: "info|debug|warn|error",
    trackUsage: true,
    trackPerformance: true,
    retentionDays: 90
  },

  // === NOTIFICATIONS ===
  notifications: {
    onSuccess: {
      enabled: true,
      recipients: ["user", "admin"],
      template: "notification-template-id"
    },
    onFailure: {
      enabled: true,
      recipients: ["admin"],
      template: "error-notification-template-id"
    },
    onStateChange: {
      enabled: true,
      recipients: ["admin"],
      template: "state-change-template-id"
    }
  },

  // === CUSTOMIZATION ===
  customization: {
    allowUserOverride: true|false,
    cssClass: "custom-class",
    styles: {},
    metadata: {}
  },

  // === AUDIT TRAIL ===
  audit: {
    createdAt: "2025-01-01T00:00:00Z",
    createdBy: "admin-user-id",
    updatedAt: "2025-02-15T00:00:00Z",
    updatedBy: "admin-user-id",
    version: 2,
    changeLog: [
      {
        timestamp: "2025-02-15T00:00:00Z",
        user: "admin-user-id",
        action: "updated",
        changes: {
          enabled: { from: false, to: true }
        }
      }
    ]
  }
}
```

---

## Admin Control Panel Structure

### Navigation Hierarchy

```
Admin Dashboard
│
├── Configuration
│   ├── Global Settings
│   │   ├── SMTP Settings
│   │   ├── System Defaults
│   │   ├── Feature Flags
│   │   └── Branding
│   │
│   ├── Communication
│   │   ├── Email Templates ⭐
│   │   │   ├── Authentication
│   │   │   │   ├── Confirm Sign Up
│   │   │   │   ├── Invite User
│   │   │   │   ├── Magic Link
│   │   │   │   ├── Change Email Address
│   │   │   │   ├── Reset Password
│   │   │   │   └── Reauthentication
│   │   │   └── Security Notifications
│   │   │       ├── Password Changed
│   │   │       ├── Email Address Changed
│   │   │       ├── Phone Number Changed
│   │   │       ├── Identity Linked
│   │   │       ├── Identity Unlinked
│   │   │       ├── MFA Method Added
│   │   │       └── MFA Method Removed
│   │   ├── SMS Templates
│   │   ├── Push Notifications
│   │   └── In-App Notifications
│   │
│   ├── UI Components
│   │   ├── Forms
│   │   ├── Modals
│   │   ├── Dashboards
│   │   └── Navigation
│   │
│   ├── Features
│   │   ├── Data Export
│   │   ├── Data Import
│   │   ├── Search & Filters
│   │   └── Analytics
│   │
│   ├── Integrations
│   │   ├── APIs
│   │   ├── Webhooks
│   │   ├── OAuth Providers
│   │   └── Payment Gateways
│   │
│   ├── Security
│   │   ├── Authentication Methods
│   │   ├── Authorization Rules
│   │   ├── Rate Limiting
│   │   └── Session Management
│   │
│   └── Business Logic
│       ├── Workflows
│       ├── Scheduled Jobs
│       └── Validation Rules
│
└── Access Control
    ├── Roles Management
    ├── Permissions Matrix
    └── User Assignments
```

---

## Email Template Configuration (Example from Images)

### Template Structure

Each email template has:

#### 1. Identification
- Name: "Confirm Sign Up"
- Category: "Authentication"
- Description: "Ask users to confirm their email address after signing up"

#### 2. Content Configuration
```
Subject: Confirm Your Signup

Body:
<h2>Confirm your signup</h2>
<p>Follow this link to confirm your user:</p>
<p><a href="{{ .ConfirmationURL }}">Confirm your mail</a></p>

Available Variables:
- {{ .ConfirmationURL }}
- {{ .Token }}
- {{ .TokenHash }}
- {{ .SiteURL }}
- {{ .Email }}
- {{ .Data }}
- {{ .RedirectTo }}
```

#### 3. Toggle Controls
- Source / Preview tabs
- Save changes button
- Docs link

---

## User Experience Flow

### Rendering Decision Tree

```
User Request → Element
                │
                ▼
        Is Element Enabled? ────NO───→ Return NULL (No Render)
                │
               YES
                ▼
        Check User Role
                │
                ▼
    Is User Role in allowedRoles? ────NO───→ Return NULL (No Render)
                │
               YES
                ▼
    Is User Role in deniedRoles? ────YES──→ Return NULL (No Render)
                │
                NO
                ▼
    Check Custom Permissions
                │
                ▼
    Does User Have Required Permission? ────NO───→ Return NULL
                │
               YES
                ▼
    Check Conditional Rendering
                │
                ▼
    Are All Conditions Met? ────NO───→ Return NULL (No Render)
                │
               YES
                ▼
    Check Dependencies
                │
                ▼
    Are Required Elements Enabled? ────NO───→ Return NULL
                │
               YES
                ▼
    Check Schedule
                │
                ▼
    Is Current Time/Date Valid? ────NO───→ Return NULL
                │
               YES
                ▼
    Check Rate Limits
                │
                ▼
    Has User Exceeded Limits? ────YES──→ Return Error/Message
                │
                NO
                ▼
    Check Element State
                │
                ▼
    Is State = "active"? ────NO───→ Return NULL
                │
               YES
                ▼
        RENDER ELEMENT ✓
                │
                ▼
        Log Usage Event
```

---

## RBAC Implementation

### Role Hierarchy

```
System Roles (Built-in):
├── super_admin (God mode - all access)
├── admin (Full application management)
├── manager (Department/team management)
├── user (Standard user access)
├── guest (Limited/read-only access)
└── suspended (No access)

Custom Roles (Admin-defined):
├── content_editor
├── analyst
├── support_agent
├── billing_manager
└── [custom roles...]
```

### Permission Types

```javascript
{
  view: "Can see the element",
  create: "Can create new instances",
  edit: "Can modify existing instances",
  delete: "Can remove instances",
  export: "Can export data",
  import: "Can import data",
  publish: "Can make live/public",
  configure: "Can change settings",
  assign: "Can assign to others",
  approve: "Can approve workflows"
}
```

### Role-Element Matrix (Example)

| Element | Super Admin | Admin | Manager | User | Guest |
|---------|-------------|-------|---------|------|-------|
| Email: Confirm Signup | Configure | Configure | View | - | - |
| Email: Password Reset | Configure | Configure | View | - | - |
| Dashboard: Analytics | View/Configure | View/Configure | View | View | - |
| Feature: Data Export | All | All | Export Own | Export Own | - |
| Feature: User Management | All | All | View Team | View Own | - |

---

## State Management

### Universal States

Every element can be in one of these states:

```javascript
{
  draft: {
    description: "Being created/configured",
    visibleTo: ["admin"],
    actions: ["edit", "preview", "publish", "discard"],
    canRender: false
  },

  active: {
    description: "Live and operational",
    visibleTo: ["admin", "manager", "user"],
    actions: ["view", "use", "edit", "deactivate"],
    canRender: true
  },

  inactive: {
    description: "Temporarily disabled",
    visibleTo: ["admin", "manager"],
    actions: ["view", "activate", "archive"],
    canRender: false
  },

  scheduled: {
    description: "Scheduled to activate",
    visibleTo: ["admin", "manager"],
    actions: ["view", "edit", "activate", "cancel"],
    canRender: false, // until schedule triggers
    activateAt: "2025-03-01T00:00:00Z"
  },

  archived: {
    description: "Kept for records, not in use",
    visibleTo: ["admin"],
    actions: ["view", "restore", "delete"],
    canRender: false
  },

  deprecated: {
    description: "Old version, being phased out",
    visibleTo: ["admin"],
    actions: ["view", "migrate", "delete"],
    canRender: false,
    replacedBy: "new-element-id"
  }
}
```

### State Transitions

```
draft ──publish──→ active
  │                  │
  └──discard──→  [deleted]
                     │
              deactivate
                     │
                     ▼
                  inactive
                     │
              ┌──────┴──────┐
           archive        activate
              │              │
              ▼              ▼
          archived        active
              │
           restore
              │
              ▼
           active
```

---

## Examples by Category

### 1. Email Template: "Password Reset"

```javascript
{
  id: "email-password-reset",
  category: "email",
  type: "authentication",
  name: "Reset Password",
  description: "Allow users to reset their password if they forget it",

  enabled: true,
  required: true,
  environment: ["dev", "staging", "prod"],

  rbac: {
    allowedRoles: ["*"], // All users can trigger
    deniedRoles: ["suspended"],
    customPermissions: {
      configure: ["admin", "super_admin"],
      preview: ["admin", "super_admin", "manager"]
    }
  },

  content: {
    subject: "Reset Your Password",
    body: `
      <h2>Reset Password</h2>
      <p>Follow this link to reset the password for your user:</p>
      <p><a href="{{ .ConfirmationURL }}">Reset Password</a></p>
    `,
    variables: [
      ".ConfirmationURL",
      ".Token",
      ".TokenHash",
      ".SiteURL",
      ".Email",
      ".Data",
      ".RedirectTo"
    ],
    localization: {
      enabled: true,
      translations: {
        en: { subject: "Reset Your Password" },
        es: { subject: "Restablecer su contraseña" },
        fr: { subject: "Réinitialiser votre mot de passe" }
      }
    }
  },

  states: {
    active: { enabled: true }
  },

  limits: {
    enabled: true,
    maxUsesPerUser: 5,
    resetPeriod: "daily",
    throttle: {
      enabled: true,
      requestsPerMinute: 3
    }
  },

  notifications: {
    onSuccess: {
      enabled: false
    },
    onFailure: {
      enabled: true,
      recipients: ["admin"],
      template: "admin-alert-failed-password-reset"
    }
  },

  logging: {
    enabled: true,
    logLevel: "info",
    trackUsage: true
  }
}
```

---

### 2. UI Component: "User Profile Form"

```javascript
{
  id: "ui-user-profile-form",
  category: "ui",
  type: "form",
  name: "User Profile Form",
  description: "Form for users to edit their profile information",

  enabled: true,
  required: false,
  environment: ["dev", "staging", "prod"],

  rbac: {
    allowedRoles: ["admin", "manager", "user"],
    deniedRoles: ["guest", "suspended"],
    customPermissions: {
      view: ["admin", "manager", "user"],
      edit: ["admin", "manager", "user"], // users can edit own
      configure: ["admin"]
    }
  },

  renderConditions: {
    requiresAuth: true,
    customConditions: [
      {
        type: "userProperty",
        property: "emailVerified",
        value: true
      }
    ]
  },

  fields: [
    {
      id: "firstName",
      type: "text",
      label: "First Name",
      required: true,
      enabled: true,
      rbac: {
        allowedRoles: ["*"]
      }
    },
    {
      id: "lastName",
      type: "text",
      label: "Last Name",
      required: true,
      enabled: true,
      rbac: {
        allowedRoles: ["*"]
      }
    },
    {
      id: "phoneNumber",
      type: "tel",
      label: "Phone Number",
      required: false,
      enabled: true,
      rbac: {
        allowedRoles: ["admin", "manager", "user"]
      }
    },
    {
      id: "department",
      type: "select",
      label: "Department",
      required: false,
      enabled: true,
      rbac: {
        allowedRoles: ["admin", "manager"],
        deniedRoles: ["user"] // users cannot change department
      }
    },
    {
      id: "internalNotes",
      type: "textarea",
      label: "Internal Notes",
      required: false,
      enabled: true,
      rbac: {
        allowedRoles: ["admin"], // only admins see this
      }
    }
  ],

  validation: {
    enabled: true,
    rules: [
      {
        field: "email",
        type: "email",
        message: "Invalid email format"
      },
      {
        field: "phoneNumber",
        type: "phone",
        message: "Invalid phone number"
      }
    ]
  },

  states: {
    active: { enabled: true }
  },

  notifications: {
    onSuccess: {
      enabled: true,
      recipients: ["user"],
      template: "profile-updated-confirmation"
    }
  }
}
```

---

### 3. Feature Module: "Data Export"

```javascript
{
  id: "feature-data-export",
  category: "feature",
  type: "data-operation",
  name: "Data Export",
  description: "Allow users to export data in various formats",

  enabled: true,
  required: false,
  environment: ["staging", "prod"], // not in dev

  rbac: {
    allowedRoles: ["admin", "manager", "analyst", "user"],
    deniedRoles: ["guest", "suspended"],
    customPermissions: {
      exportAll: ["admin", "manager"],
      exportOwn: ["admin", "manager", "analyst", "user"],
      exportTeam: ["admin", "manager"],
      configure: ["admin"]
    }
  },

  renderConditions: {
    requiresAuth: true,
    requiresSubscription: "pro", // free users don't get export
    customConditions: [
      {
        type: "featureFlag",
        flag: "advanced-exports",
        value: true
      }
    ]
  },

  formats: [
    {
      id: "csv",
      name: "CSV",
      enabled: true,
      rbac: {
        allowedRoles: ["*"]
      }
    },
    {
      id: "excel",
      name: "Excel (XLSX)",
      enabled: true,
      rbac: {
        allowedRoles: ["admin", "manager", "analyst"]
      }
    },
    {
      id: "pdf",
      name: "PDF",
      enabled: true,
      rbac: {
        allowedRoles: ["admin", "manager"]
      }
    },
    {
      id: "json",
      name: "JSON",
      enabled: true,
      rbac: {
        allowedRoles: ["admin"]
      }
    }
  ],

  limits: {
    enabled: true,
    maxUsesPerRole: {
      admin: -1, // unlimited
      manager: 100,
      analyst: 50,
      user: 10
    },
    resetPeriod: "monthly",
    maxRowsPerExport: {
      admin: -1,
      manager: 10000,
      analyst: 5000,
      user: 1000
    }
  },

  states: {
    active: { enabled: true }
  },

  notifications: {
    onSuccess: {
      enabled: true,
      recipients: ["user"],
      template: "export-ready-notification"
    },
    onFailure: {
      enabled: true,
      recipients: ["user", "admin"],
      template: "export-failed-notification"
    }
  },

  logging: {
    enabled: true,
    logLevel: "info",
    trackUsage: true,
    retentionDays: 365 // keep export logs for 1 year
  }
}
```

---

### 4. Integration: "Stripe Payment Gateway"

```javascript
{
  id: "integration-stripe",
  category: "integration",
  type: "payment-gateway",
  name: "Stripe Payment Gateway",
  description: "Process payments through Stripe",

  enabled: true,
  required: false,
  environment: ["staging", "prod"],

  rbac: {
    allowedRoles: ["admin", "billing_manager"],
    deniedRoles: ["guest", "suspended"],
    customPermissions: {
      configure: ["admin"],
      viewTransactions: ["admin", "billing_manager"],
      processPayment: ["admin", "billing_manager"],
      refund: ["admin"]
    }
  },

  configuration: {
    apiKey: "sk_live_...",
    webhookSecret: "whsec_...",
    testMode: false,
    currency: "USD",
    supportedCurrencies: ["USD", "EUR", "GBP"],
    features: {
      subscriptions: true,
      oneTimePayments: true,
      refunds: true,
      webhooks: true
    }
  },

  dependencies: {
    requires: ["ssl-certificate", "secure-endpoints"],
    conflicts: ["integration-paypal"] // can't have both active
  },

  states: {
    active: { enabled: true },
    inactive: { enabled: false }
  },

  webhookHandlers: [
    {
      event: "payment_intent.succeeded",
      handler: "handlePaymentSuccess",
      notification: {
        enabled: true,
        recipients: ["user", "billing_manager"],
        template: "payment-success"
      }
    },
    {
      event: "payment_intent.payment_failed",
      handler: "handlePaymentFailure",
      notification: {
        enabled: true,
        recipients: ["user", "admin"],
        template: "payment-failed"
      }
    }
  ],

  limits: {
    enabled: true,
    maxTransactionsPerDay: 1000,
    throttle: {
      enabled: true,
      requestsPerMinute: 100
    }
  },

  logging: {
    enabled: true,
    logLevel: "debug",
    trackUsage: true,
    trackPerformance: true,
    retentionDays: 2555 // 7 years for financial records
  }
}
```

---

### 5. Security Element: "Multi-Factor Authentication"

```javascript
{
  id: "security-mfa",
  category: "security",
  type: "authentication-method",
  name: "Multi-Factor Authentication",
  description: "Require users to verify identity with second factor",

  enabled: true,
  required: false, // can be made mandatory per role
  environment: ["dev", "staging", "prod"],

  rbac: {
    allowedRoles: ["*"], // all users can enable MFA
    deniedRoles: ["suspended"],
    requiredForRoles: ["admin", "billing_manager"], // mandatory for these
    customPermissions: {
      configure: ["admin"],
      enforce: ["admin"],
      bypass: [] // no one can bypass (not even admin)
    }
  },

  methods: [
    {
      id: "totp",
      name: "Authenticator App (TOTP)",
      enabled: true,
      rbac: {
        allowedRoles: ["*"]
      },
      priority: 1
    },
    {
      id: "sms",
      name: "SMS Code",
      enabled: true,
      rbac: {
        allowedRoles: ["admin", "manager", "user"]
      },
      priority: 2,
      limits: {
        maxSMSPerDay: 10
      }
    },
    {
      id: "email",
      name: "Email Code",
      enabled: true,
      rbac: {
        allowedRoles: ["*"]
      },
      priority: 3
    },
    {
      id: "backup-codes",
      name: "Backup Codes",
      enabled: true,
      rbac: {
        allowedRoles: ["*"]
      },
      count: 10
    }
  ],

  enforcement: {
    gracePeriodDays: 7, // days to enable MFA after account creation
    reminderFrequency: "daily",
    allowSkip: false, // for required roles
    lockAccountAfter: 30 // days without MFA if required
  },

  states: {
    active: { enabled: true }
  },

  notifications: {
    onMethodAdded: {
      enabled: true,
      recipients: ["user"],
      template: "mfa-method-added"
    },
    onMethodRemoved: {
      enabled: true,
      recipients: ["user"],
      template: "mfa-method-removed"
    },
    onEnforcement: {
      enabled: true,
      recipients: ["user"],
      template: "mfa-enforcement-reminder"
    }
  },

  logging: {
    enabled: true,
    logLevel: "info",
    trackUsage: true,
    retentionDays: 365
  }
}
```

---

## Implementation Checklist

When implementing ANY new element, ensure:

### Phase 1: Definition
- [ ] Element ID defined
- [ ] Category assigned
- [ ] Type specified
- [ ] Name and description written
- [ ] Default state determined

### Phase 2: RBAC Configuration
- [ ] Allowed roles defined
- [ ] Denied roles defined
- [ ] Custom permissions mapped
- [ ] Default role set
- [ ] Role inheritance configured

### Phase 3: Rendering Logic
- [ ] Enabled/disabled toggle implemented
- [ ] Required flag set
- [ ] Conditional rendering rules defined
- [ ] Environment filters set
- [ ] Dependency checks configured

### Phase 4: State Management
- [ ] All states defined
- [ ] State transitions mapped
- [ ] Actions per state defined
- [ ] Initial state set
- [ ] State change handlers implemented

### Phase 5: Content/Template
- [ ] Template created
- [ ] Variables defined
- [ ] Localization configured
- [ ] Preview functionality added
- [ ] Source/visual editors implemented

### Phase 6: Limits & Throttling
- [ ] Rate limits defined per role
- [ ] Usage limits set
- [ ] Reset periods configured
- [ ] Throttling implemented
- [ ] Quota tracking added

### Phase 7: Dependencies
- [ ] Required elements listed
- [ ] Conflicting elements listed
- [ ] Enhancement dependencies noted
- [ ] Dependency checks implemented

### Phase 8: Validation
- [ ] Validation rules defined
- [ ] Error messages written
- [ ] Custom validators implemented
- [ ] Client-side validation added
- [ ] Server-side validation added

### Phase 9: Notifications
- [ ] Success notifications configured
- [ ] Failure notifications configured
- [ ] State change notifications configured
- [ ] Recipients defined
- [ ] Templates created

### Phase 10: Logging & Analytics
- [ ] Logging enabled
- [ ] Log level set
- [ ] Usage tracking implemented
- [ ] Performance tracking added
- [ ] Retention policy set

### Phase 11: Audit Trail
- [ ] Created timestamp
- [ ] Created by user
- [ ] Version tracking
- [ ] Change log implementation
- [ ] History view added

### Phase 12: Testing
- [ ] Unit tests for all states
- [ ] RBAC tests for all roles
- [ ] Integration tests
- [ ] Edge case tests
- [ ] Performance tests

---

## Database Schema Pattern

### Elements Table
```sql
CREATE TABLE elements (
  id VARCHAR(255) PRIMARY KEY,
  category ENUM('email', 'ui', 'feature', 'integration', 'security', 'business'),
  type VARCHAR(100),
  name VARCHAR(255),
  description TEXT,
  enabled BOOLEAN DEFAULT true,
  required BOOLEAN DEFAULT false,
  current_state VARCHAR(50) DEFAULT 'draft',
  configuration JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by VARCHAR(255),
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  updated_by VARCHAR(255),
  version INT DEFAULT 1
);
```

### Element Roles Table
```sql
CREATE TABLE element_roles (
  id INT PRIMARY KEY AUTO_INCREMENT,
  element_id VARCHAR(255),
  role_id VARCHAR(255),
  permission_type ENUM('allowed', 'denied', 'view', 'create', 'edit', 'delete', 'configure'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (element_id) REFERENCES elements(id),
  FOREIGN KEY (role_id) REFERENCES roles(id)
);
```

### Element States Table
```sql
CREATE TABLE element_states (
  id INT PRIMARY KEY AUTO_INCREMENT,
  element_id VARCHAR(255),
  state VARCHAR(50),
  enabled BOOLEAN,
  configuration JSON,
  FOREIGN KEY (element_id) REFERENCES elements(id)
);
```

### Element Change Log Table
```sql
CREATE TABLE element_changelog (
  id INT PRIMARY KEY AUTO_INCREMENT,
  element_id VARCHAR(255),
  user_id VARCHAR(255),
  action VARCHAR(50),
  changes JSON,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (element_id) REFERENCES elements(id)
);
```

### Element Usage Log Table
```sql
CREATE TABLE element_usage_log (
  id INT PRIMARY KEY AUTO_INCREMENT,
  element_id VARCHAR(255),
  user_id VARCHAR(255),
  role_id VARCHAR(255),
  action VARCHAR(100),
  success BOOLEAN,
  metadata JSON,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (element_id) REFERENCES elements(id)
);
```

---

## API Endpoints Pattern

### Get Element Configuration
```
GET /api/admin/elements/{element_id}
Authorization: Admin

Response:
{
  "element": { /* full configuration */ },
  "effectiveConfig": { /* after inheritance */ },
  "dependencies": { /* dependency status */ },
  "usage": { /* usage statistics */ }
}
```

### Update Element Configuration
```
POST /api/admin/elements/{element_id}
Authorization: Admin

Body:
{
  "enabled": true,
  "rbac": { /* updated RBAC */ },
  "content": { /* updated content */ }
}

Response:
{
  "success": true,
  "element": { /* updated configuration */ },
  "version": 3
}
```

### Check Element Access (User-facing)
```
GET /api/elements/{element_id}/check-access
Authorization: User

Response:
{
  "canAccess": true,
  "permissions": ["view", "use"],
  "element": { /* sanitized config for user */ }
}
```

### Render Element (User-facing)
```
GET /api/elements/{element_id}/render
Authorization: User

Response:
{
  "shouldRender": true,
  "content": { /* rendered content */ },
  "metadata": { /* additional metadata */ }
}
```

---

## Frontend Implementation Pattern

### React Component Example

```typescript
import { useElementConfig } from '@/hooks/useElementConfig';
import { useUserRole } from '@/hooks/useUserRole';

interface ElementWrapperProps {
  elementId: string;
  children: React.ReactNode;
}

const ElementWrapper: React.FC<ElementWrapperProps> = ({
  elementId,
  children
}) => {
  const { role, permissions } = useUserRole();
  const {
    element,
    shouldRender,
    loading
  } = useElementConfig(elementId, role);

  // Don't render anything while loading
  if (loading) return null;

  // Don't render if conditions not met (NOT HIDDEN - NOT RENDERED)
  if (!shouldRender) return null;

  // Don't render if not enabled
  if (!element.enabled) return null;

  // Don't render if not in active state
  if (element.currentState !== 'active') return null;

  // Don't render if user role not allowed
  if (!element.rbac.allowedRoles.includes(role) &&
      !element.rbac.allowedRoles.includes('*')) {
    return null;
  }

  // Don't render if user role explicitly denied
  if (element.rbac.deniedRoles.includes(role)) {
    return null;
  }

  // Check custom permissions
  const hasPermission = permissions.some(p =>
    element.rbac.customPermissions.view?.includes(role)
  );

  if (!hasPermission) return null;

  // All checks passed - RENDER
  return (
    <div
      className={element.customization.cssClass}
      data-element-id={elementId}
    >
      {children}
    </div>
  );
};

export default ElementWrapper;
```

### Usage Example

```tsx
<ElementWrapper elementId="email-password-reset">
  <PasswordResetForm />
</ElementWrapper>

<ElementWrapper elementId="feature-data-export">
  <DataExportButton />
</ElementWrapper>

<ElementWrapper elementId="ui-admin-dashboard">
  <AdminDashboard />
</ElementWrapper>
```

---

## Admin UI Component Example

```tsx
import { useState } from 'react';
import { updateElement } from '@/api/elements';

interface ElementConfigPanelProps {
  elementId: string;
}

const ElementConfigPanel: React.FC<ElementConfigPanelProps> = ({
  elementId
}) => {
  const [element, setElement] = useState(null);
  const [activeTab, setActiveTab] = useState<'general' | 'rbac' | 'content'>('general');

  const handleToggleEnabled = async () => {
    const updated = await updateElement(elementId, {
      enabled: !element.enabled
    });
    setElement(updated);
  };

  return (
    <div className="element-config-panel">
      <header>
        <h2>{element.name}</h2>
        <p>{element.description}</p>
        <button onClick={handleToggleEnabled}>
          {element.enabled ? 'Disable' : 'Enable'}
        </button>
      </header>

      <nav className="tabs">
        <button
          className={activeTab === 'general' ? 'active' : ''}
          onClick={() => setActiveTab('general')}
        >
          General
        </button>
        <button
          className={activeTab === 'rbac' ? 'active' : ''}
          onClick={() => setActiveTab('rbac')}
        >
          Access Control
        </button>
        <button
          className={activeTab === 'content' ? 'active' : ''}
          onClick={() => setActiveTab('content')}
        >
          Content
        </button>
      </nav>

      <section className="tab-content">
        {activeTab === 'general' && (
          <GeneralSettings element={element} />
        )}

        {activeTab === 'rbac' && (
          <RBACSettings element={element} />
        )}

        {activeTab === 'content' && (
          <ContentEditor element={element} />
        )}
      </section>

      <footer>
        <button>Save Changes</button>
        <button>Preview</button>
        <a href="/docs" target="_blank">Docs</a>
      </footer>
    </div>
  );
};
```

---

## Summary

This comprehensive flow ensures:

1. **Complete Admin Control**: Every aspect of every element is configurable
2. **Granular RBAC**: Role-based access at every level
3. **Conditional Rendering**: Elements don't render unless all conditions met
4. **State Management**: Clear lifecycle and transitions
5. **Audit Trail**: Complete history of changes
6. **Scalability**: Pattern applies to ANY element type
7. **Security**: No hidden elements, only non-rendered ones
8. **Flexibility**: Admin can enable/disable, configure, and control everything

Every element follows this pattern, from email templates to UI components to entire feature modules. The system is designed to give administrators god-mode control while ensuring users only see what they're authorized to access.
