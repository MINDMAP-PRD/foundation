# Comprehensive Settings Management System Configuration Flow

> **Foundation wiring note:** This file feeds the shared settings hierarchy in `GeneralApp`. Settings must be authoritative and runtime-driving, not decorative forms, and must connect to owner UI, persistence, resolution, and enforcement.

## Table of Contents
1. [System Overview](#system-overview)
2. [Universal Settings Configuration Pattern](#universal-settings-configuration-pattern)
3. [Settings Hierarchy & Scope](#settings-hierarchy--scope)
4. [User Profile Settings](#user-profile-settings)
5. [Account Security Settings](#account-security-settings)
6. [Notification & Communication Settings](#notification--communication-settings)
7. [Organization Settings](#organization-settings)
8. [Branding & Appearance Settings](#branding--appearance-settings)
9. [Billing & Subscription Settings](#billing--subscription-settings)
10. [AI & MCP Provider Settings](#ai--mcp-provider-settings)
11. [Email System Settings](#email-system-settings)
12. [Integration & Webhook Settings](#integration--webhook-settings)
13. [System Configuration (Admin)](#system-configuration-admin)
14. [UI Configuration (Admin)](#ui-configuration-admin)
15. [Privacy & Data Settings](#privacy--data-settings)
16. [Localization & Internationalization](#localization--internationalization)
17. [Admin Settings Dashboard](#admin-settings-dashboard)
18. [User Settings Experience](#user-settings-experience)
19. [Settings Validation & Constraints](#settings-validation--constraints)
20. [Settings API Architecture](#settings-api-architecture)
21. [Database Schema](#database-schema)
22. [Settings Migration & Versioning](#settings-migration--versioning)
23. [Settings Audit & Change Tracking](#settings-audit--change-tracking)
24. [Edge Cases & Conflict Resolution](#edge-cases--conflict-resolution)
25. [Performance & Caching](#performance--caching)
26. [Integration Examples](#integration-examples)

---

## System Overview

### Core Principles
1. **Admin Full Control**: Every setting configurable without code changes
2. **Layered Overrides**: System defaults → Organization → Team → User (most specific wins)
3. **RBAC-Gated**: Settings visibility and editability controlled by roles and permissions
4. **Real-Time Propagation**: Setting changes take effect immediately across all sessions
5. **Audit Trail**: Every setting change is logged with who, what, when, and why
6. **Validation-First**: All settings validated server-side before persistence
7. **Rollback Support**: Any setting can be reverted to previous value or default
8. **Plan-Aware**: Settings availability tied to subscription plan
9. **Multi-Tenant Isolation**: Organization settings never leak across tenants

### Settings Architecture
```
System Defaults (code-level, immutable baseline)
    ↓ overridden by
Platform Settings (OWNER-configured, applies to all orgs)
    ↓ overridden by
Organization Settings (ADMIN-configured, applies to org members)
    ↓ overridden by
Team/Department Settings (Team Lead-configured, optional layer)
    ↓ overridden by
User Settings (self-service, personal preferences)
    ↓ enforced by
RBAC + Plan Restrictions (cannot exceed what role/plan allows)
```

### Settings Resolution Flow
```
User Request → Identify Setting Key
    → Load User-Level Value (if exists)
    → Load Team-Level Value (if exists, user didn't override)
    → Load Org-Level Value (if exists, team didn't override)
    → Load Platform-Level Value (if exists, org didn't override)
    → Load System Default (always exists)
    → Apply RBAC Filter (hide/disable settings user can't access)
    → Apply Plan Filter (hide/disable settings not in plan)
    → Return Resolved Value + Metadata (source, editable, locked)
```

---

## Universal Settings Configuration Pattern

Every settings element follows this universal structure:

```typescript
interface SettingsConfiguration {
  // ============================================
  // CORE IDENTITY
  // ============================================
  id: string                              // "settings-main"
  name: string                            // "Settings Management System"
  category: SettingsCategory              // profile | security | organization | billing | system
  type: SettingsType                      // user | organization | platform | system

  // Enable/Disable
  enabled: boolean                        // Master switch for settings system
  maintenanceMode: boolean                // Lock all settings during maintenance

  // ============================================
  // SETTINGS SCOPE CONFIGURATION
  // ============================================
  scopeConfig: {
    // Available scopes
    scopes: {
      system: {
        enabled: true                     // Always enabled
        editableBy: ["PLATFORM_ADMIN"]    // Who can edit system defaults
        description: "Code-level defaults, baseline for all settings"
      }
      platform: {
        enabled: boolean
        editableBy: ["OWNER"]
        description: "Platform-wide settings, overrides system defaults"
      }
      organization: {
        enabled: boolean
        editableBy: ["OWNER", "ADMIN"]
        description: "Organization-specific settings"
      }
      team: {
        enabled: boolean                  // Optional layer
        editableBy: ["OWNER", "ADMIN", "TEAM_LEAD"]
        description: "Team/department-specific settings"
      }
      user: {
        enabled: boolean
        editableBy: ["self"]              // User edits their own
        description: "Personal user preferences"
      }
    }

    // Override rules
    overrideRules: {
      allowUserOverride: boolean          // Can users override org settings
      allowTeamOverride: boolean          // Can teams override org settings
      lockableByAdmin: boolean            // Admin can lock settings to prevent override
      inheritByDefault: boolean           // New settings inherit from parent scope
    }

    // Scope resolution
    resolution: {
      strategy: "most_specific_wins" | "most_restrictive_wins" | "explicit_only"
      fallbackToDefault: boolean          // Fall back to system default if no value
      cacheResolved: boolean              // Cache resolved values
      cacheTTL: number                    // Seconds
    }
  }

  // ============================================
  // PROFILE SETTINGS CONFIGURATION
  // ============================================
  profileSettings: {
    // Personal information
    personalInfo: {
      fields: {
        fullName: SettingField
        displayName: SettingField
        email: SettingField
        phone: SettingField
        avatar: SettingField
        bio: SettingField
        position: SettingField
        department: SettingField
        location: SettingField
        website: SettingField
        socialLinks: SettingField
        dateOfBirth: SettingField
        pronouns: SettingField
        customFields: SettingField[]
      }

      // Admin controls
      adminConfig: {
        requiredFields: string[]          // Which fields are mandatory
        hiddenFields: string[]            // Which fields are hidden from users
        readOnlyFields: string[]          // Which fields users can't edit
        validationRules: Record<string, ValidationRule>
        maxAvatarSize: number             // MB
        allowedAvatarFormats: string[]    // ["jpg", "png", "webp"]
        avatarCropRequired: boolean
        avatarMinDimensions: { width: number, height: number }
      }
    }

    // Preferences
    preferences: {
      theme: {
        enabled: boolean
        options: ("light" | "dark" | "system" | "custom")[]
        allowCustomTheme: boolean
        defaultTheme: string
        syncAcrossDevices: boolean
      }
      language: {
        enabled: boolean
        availableLanguages: string[]      // ["en", "es", "fr", "de", "pt", "ja", "zh"]
        defaultLanguage: string
        autoDetect: boolean               // Detect from browser
        allowUserOverride: boolean
      }
      timezone: {
        enabled: boolean
        autoDetect: boolean
        defaultTimezone: string
        allowUserOverride: boolean
      }
      dateFormat: {
        enabled: boolean
        options: string[]                 // ["MM/DD/YYYY", "DD/MM/YYYY", "YYYY-MM-DD"]
        defaultFormat: string
      }
      timeFormat: {
        enabled: boolean
        options: ("12h" | "24h")[]
        defaultFormat: string
      }
      numberFormat: {
        enabled: boolean
        options: string[]                 // ["1,000.00", "1.000,00", "1 000,00"]
        defaultFormat: string
      }
      currency: {
        enabled: boolean
        defaultCurrency: string           // "USD"
        allowUserOverride: boolean
      }
      accessibility: {
        enabled: boolean
        reducedMotion: boolean
        highContrast: boolean
        fontSize: "small" | "medium" | "large" | "xlarge"
        screenReaderOptimized: boolean
        keyboardNavigation: boolean
        colorBlindMode: "none" | "protanopia" | "deuteranopia" | "tritanopia"
      }
      dashboard: {
        defaultView: "grid" | "list" | "kanban"
        itemsPerPage: number
        showWelcomeMessage: boolean
        pinnedItems: string[]
        widgetLayout: Record<string, any>
        sidebarCollapsed: boolean
        sidebarWidth: number
      }
    }
  }

  // ============================================
  // SECURITY SETTINGS CONFIGURATION
  // ============================================
  securitySettings: {
    // Password management
    password: {
      changeEnabled: boolean              // Can users change password
      requireCurrentPassword: boolean     // Must enter current password to change
      policy: PasswordPolicy              // Defined in user management flow
      expiryEnabled: boolean
      expiryDays: number
      preventReuse: number                // Last N passwords
      forceChangeOnFirstLogin: boolean
      forceChangeOnReset: boolean
    }

    // Two-factor authentication
    twoFactor: {
      enabled: boolean
      required: boolean                   // Force for all users
      requiredForRoles: string[]          // Force for specific roles
      methods: {
        totp: {
          enabled: boolean
          issuer: string
          allowBackupCodes: boolean
          backupCodesCount: number
        }
        sms: {
          enabled: boolean
          provider: string
        }
        email: {
          enabled: boolean
        }
        webauthn: {
          enabled: boolean
          allowPlatformAuthenticators: boolean
          allowCrossPlatformAuthenticators: boolean
        }
      }
      gracePeriodDays: number             // Days to set up 2FA
      rememberDeviceDays: number          // Trust device for N days
    }

    // Session management
    sessions: {
      viewActiveSessions: boolean         // Can users see active sessions
      revokeOtherSessions: boolean        // Can users revoke other sessions
      showDeviceInfo: boolean             // Show device/browser/IP
      showLocation: boolean               // Show geo location
      maxConcurrentSessions: number
      sessionTimeout: number              // Minutes
      idleTimeout: number                 // Minutes
      rememberMe: {
        enabled: boolean
        duration: number                  // Days
      }
    }

    // Login history
    loginHistory: {
      enabled: boolean
      retentionDays: number
      showToUser: boolean                 // Can users see their login history
      alertOnNewDevice: boolean
      alertOnNewLocation: boolean
      alertOnFailedAttempts: boolean
      alertThreshold: number              // Alert after N failed attempts
    }

    // API keys
    apiKeys: {
      enabled: boolean
      maxKeysPerUser: number
      keyExpiry: boolean
      defaultExpiryDays: number
      allowedScopes: string[]
      requireDescription: boolean
      showLastUsed: boolean
      rotationReminder: boolean
      rotationReminderDays: number
    }

    // Connected accounts (OAuth)
    connectedAccounts: {
      enabled: boolean
      providers: string[]                 // ["google", "github", "microsoft"]
      allowLink: boolean                  // Can link additional providers
      allowUnlink: boolean                // Can unlink providers
      requireAtLeastOne: boolean          // Must have at least one auth method
    }

    // Account deletion
    accountDeletion: {
      enabled: boolean
      requireConfirmation: boolean
      confirmationMethod: "password" | "email_code" | "both"
      gracePeriodDays: number             // Days before permanent deletion
      allowReactivation: boolean          // Can reactivate during grace period
      dataExportBeforeDelete: boolean     // Offer data export before deletion
      notifyAdmin: boolean                // Notify admin on deletion request
    }

    // Account deactivation
    accountDeactivation: {
      enabled: boolean
      selfService: boolean                // Can users deactivate themselves
      preserveData: boolean               // Keep data when deactivated
      autoReactivateOnLogin: boolean
    }
  }

  // ============================================
  // NOTIFICATION SETTINGS CONFIGURATION
  // ============================================
  notificationSettings: {
    // Channels
    channels: {
      email: {
        enabled: boolean
        allowUserToggle: boolean          // Can users turn off email notifications
        digestMode: boolean               // Batch notifications into digest
        digestFrequency: "realtime" | "hourly" | "daily" | "weekly"
        unsubscribeLink: boolean          // Include unsubscribe in emails
      }
      inApp: {
        enabled: boolean
        allowUserToggle: boolean
        showBadge: boolean                // Show notification count badge
        playSound: boolean
        desktopNotifications: boolean     // Browser push notifications
        maxVisible: number                // Max notifications in dropdown
        autoMarkRead: boolean             // Mark as read when viewed
        retentionDays: number
      }
      push: {
        enabled: boolean
        provider: "firebase" | "onesignal" | "custom"
        allowUserToggle: boolean
      }
      sms: {
        enabled: boolean
        provider: string
        allowUserToggle: boolean
        onlyUrgent: boolean               // Only send SMS for urgent notifications
      }
      slack: {
        enabled: boolean
        webhookUrl: string
        allowUserToggle: boolean
        channelMapping: Record<string, string>  // notification type → channel
      }
      webhook: {
        enabled: boolean
        allowUserWebhooks: boolean        // Can users set their own webhooks
        maxWebhooksPerUser: number
      }
    }

    // Notification categories
    categories: {
      security: {
        label: "Security Alerts"
        description: "Login attempts, password changes, 2FA events"
        defaultEnabled: boolean
        userCanDisable: boolean           // Some security alerts can't be disabled
        channels: string[]                // Which channels this category uses
        priority: "low" | "medium" | "high" | "urgent"
      }
      account: {
        label: "Account Updates"
        description: "Profile changes, role changes, plan changes"
        defaultEnabled: boolean
        userCanDisable: boolean
        channels: string[]
        priority: string
      }
      collaboration: {
        label: "Collaboration"
        description: "Comments, mentions, shares, invitations"
        defaultEnabled: boolean
        userCanDisable: boolean
        channels: string[]
        priority: string
      }
      projects: {
        label: "Project Updates"
        description: "Project created, updated, deleted, status changes"
        defaultEnabled: boolean
        userCanDisable: boolean
        channels: string[]
        priority: string
      }
      billing: {
        label: "Billing & Payments"
        description: "Invoice, payment, subscription changes"
        defaultEnabled: boolean
        userCanDisable: boolean
        channels: string[]
        priority: string
      }
      system: {
        label: "System Notifications"
        description: "Maintenance, updates, announcements"
        defaultEnabled: boolean
        userCanDisable: boolean
        channels: string[]
        priority: string
      }
      marketing: {
        label: "Marketing & Tips"
        description: "Product updates, tips, newsletters"
        defaultEnabled: boolean
        userCanDisable: boolean
        channels: string[]
        priority: string
        requireConsent: boolean           // GDPR: require explicit consent
      }
    }

    // Quiet hours
    quietHours: {
      enabled: boolean
      allowUserConfig: boolean
      defaultStart: string                // "22:00"
      defaultEnd: string                  // "08:00"
      timezone: string
      exceptUrgent: boolean               // Still send urgent notifications
      exceptChannels: string[]            // Channels exempt from quiet hours
    }

    // Notification grouping
    grouping: {
      enabled: boolean
      groupByProject: boolean
      groupByType: boolean
      groupingWindow: number              // Minutes to group notifications
      maxGroupSize: number
    }
  }

  // ============================================
  // ORGANIZATION SETTINGS CONFIGURATION
  // ============================================
  organizationSettings: {
    // General
    general: {
      name: SettingField
      slug: SettingField
      description: SettingField
      website: SettingField
      industry: SettingField
      size: SettingField                  // "1-10", "11-50", "51-200", "201-500", "500+"
      foundedYear: SettingField
      taxId: SettingField
      address: {
        street: SettingField
        city: SettingField
        state: SettingField
        postalCode: SettingField
        country: SettingField
      }
      contactEmail: SettingField
      contactPhone: SettingField
    }

    // Branding
    branding: {
      logo: {
        enabled: boolean
        lightModeLogo: string             // URL
        darkModeLogo: string              // URL
        favicon: string                   // URL
        maxSize: number                   // MB
        allowedFormats: string[]
        minDimensions: { width: number, height: number }
      }
      colors: {
        enabled: boolean
        primaryColor: string
        secondaryColor: string
        accentColor: string
        backgroundColor: string
        textColor: string
        customCSS: boolean                // Allow custom CSS injection
      }
      customDomain: {
        enabled: boolean                  // Plan-gated
        domain: string
        sslCertificate: "auto" | "custom"
        verificationMethod: "dns_cname" | "dns_txt" | "file"
        verified: boolean
      }
      emailBranding: {
        enabled: boolean
        fromName: string
        fromEmail: string
        replyToEmail: string
        headerLogo: string
        footerText: string
        customTemplate: boolean
      }
    }

    // Member management
    memberManagement: {
      invitationSettings: {
        enabled: boolean
        whoCanInvite: string[]            // Roles that can invite
        requireApproval: boolean          // Admin must approve invitations
        inviteExpiry: number              // Days
        maxPendingInvites: number
        allowBulkInvite: boolean
        customInviteMessage: boolean
        defaultRole: string               // Default role for invitees
        allowRoleSelection: boolean       // Can inviter choose role
        maxRoleForInviter: string         // Maximum role inviter can assign
        domainRestriction: {
          enabled: boolean
          allowedDomains: string[]        // Only allow @company.com
          blockedDomains: string[]
        }
      }
      memberLimits: {
        maxMembers: number                // Plan-based
        maxAdmins: number
        maxViewers: number
        warnAtPercentage: number          // Warn when 80% of limit reached
      }
      offboarding: {
        enabled: boolean
        transferDataOnRemoval: boolean    // Transfer user's data to another user
        gracePeriodDays: number           // Days before data deletion
        notifyUser: boolean
        notifyAdmin: boolean
        revokeAccessImmediately: boolean
      }
    }

    // Feature toggles (org-level)
    featureToggles: {
      projects: boolean
      blueprints: boolean
      mindmaps: boolean
      aiGeneration: boolean
      collaboration: boolean
      export: boolean
      import: boolean
      apiAccess: boolean
      webhooks: boolean
      customDomain: boolean
      sso: boolean
      auditLogs: boolean
      advancedRBAC: boolean
      customBranding: boolean
    }

    // Resource limits
    resourceLimits: {
      maxProjects: number
      maxBlueprints: number
      maxMindMaps: number
      maxAICreditsPerMonth: number
      maxStorageGB: number
      maxFileUploadMB: number
      maxAPIRequestsPerDay: number
      maxWebhooks: number
      maxCustomRoles: number
      maxIntegrations: number
    }

    // Data retention
    dataRetention: {
      enabled: boolean
      defaultRetentionDays: number
      deletedItemRetention: number        // Days to keep soft-deleted items
      auditLogRetention: number           // Days
      activityLogRetention: number        // Days
      sessionLogRetention: number         // Days
      autoArchive: boolean
      archiveAfterDays: number
    }
  }

  // ============================================
  // BILLING SETTINGS CONFIGURATION
  // ============================================
  billingSettings: {
    // Subscription
    subscription: {
      currentPlan: string
      billingCycle: "monthly" | "annual"
      autoRenew: boolean
      cancelAtPeriodEnd: boolean
      trialDaysRemaining: number
      nextBillingDate: Date
      paymentMethod: PaymentMethod
    }

    // Payment methods
    paymentMethods: {
      enabled: boolean
      allowMultiple: boolean
      defaultMethod: string
      supportedTypes: ("card" | "bank_transfer" | "paypal" | "invoice")[]
      requireBillingAddress: boolean
      taxIdCollection: boolean
    }

    // Invoices
    invoices: {
      enabled: boolean
      autoGenerate: boolean
      format: "pdf" | "html"
      includeLineItems: boolean
      customFields: { label: string, value: string }[]
      sendToEmail: string
      ccEmails: string[]
      retentionYears: number
    }

    // Usage & quotas
    usage: {
      showUsageDashboard: boolean
      alertOnQuotaThreshold: boolean
      quotaThresholdPercent: number       // Alert at 80%
      autoUpgradeOnLimit: boolean         // Auto-upgrade plan when limit hit
      hardLimitBehavior: "block" | "warn" | "allow_overage"
      overageCharging: boolean
      overageRate: Record<string, number> // Per-unit overage rates
    }

    // Tax settings
    tax: {
      enabled: boolean
      autoCalculate: boolean
      taxProvider: "stripe_tax" | "avalara" | "manual"
      defaultTaxRate: number
      taxExempt: boolean
      taxId: string
      taxIdType: string                   // "eu_vat", "us_ein", etc.
    }

    // Admin billing controls
    adminControls: {
      canChangePlan: boolean              // Who can change plan
      canAddPaymentMethod: boolean
      canViewInvoices: boolean
      canDownloadInvoices: boolean
      canManageTax: boolean
      canViewUsage: boolean
      canSetBudgetAlerts: boolean
      budgetLimit: number                 // Monthly budget limit
      budgetAlertThreshold: number        // Alert at percentage
    }
  }

  // ============================================
  // AI & MCP SETTINGS CONFIGURATION
  // ============================================
  aiSettings: {
    // AI providers
    providers: {
      openai: {
        enabled: boolean
        apiKey: string                    // Encrypted
        organization: string
        defaultModel: string              // "gpt-4", "gpt-3.5-turbo"
        maxTokens: number
        temperature: number
        topP: number
        frequencyPenalty: number
        presencePenalty: number
      }
      anthropic: {
        enabled: boolean
        apiKey: string
        defaultModel: string              // "claude-3-opus", "claude-3-sonnet"
        maxTokens: number
        temperature: number
      }
      google: {
        enabled: boolean
        apiKey: string
        defaultModel: string              // "gemini-pro"
        maxTokens: number
      }
      custom: {
        enabled: boolean
        endpoint: string
        apiKey: string
        headers: Record<string, string>
        model: string
      }
    }

    // MCP (Model Context Protocol)
    mcp: {
      enabled: boolean
      servers: {
        id: string
        name: string
        url: string
        apiKey: string                    // Encrypted
        enabled: boolean
        timeout: number                   // Seconds
        retryAttempts: number
        healthCheckInterval: number       // Seconds
        capabilities: string[]
      }[]
      maxConcurrentConnections: number
      connectionTimeout: number
      defaultServer: string
    }

    // AI usage controls
    usageControls: {
      enabled: boolean
      maxCreditsPerUser: number           // Per month
      maxCreditsPerOrg: number            // Per month
      maxRequestsPerMinute: number        // Rate limiting
      maxTokensPerRequest: number
      allowedModels: string[]             // Restrict which models can be used
      contentFilter: boolean              // Filter inappropriate content
      logPrompts: boolean                 // Log AI prompts for audit
      logResponses: boolean               // Log AI responses for audit
      retentionDays: number               // How long to keep AI logs
    }

    // AI feature toggles
    features: {
      codeGeneration: boolean
      textGeneration: boolean
      imageGeneration: boolean
      chatAssistant: boolean
      autoComplete: boolean
      summarization: boolean
      translation: boolean
      blueprintGeneration: boolean
      mindmapGeneration: boolean
    }

    // Admin AI controls
    adminControls: {
      canConfigureProviders: boolean
      canViewUsage: boolean
      canSetLimits: boolean
      canViewPromptLogs: boolean
      canManageMCP: boolean
      canToggleFeatures: boolean
    }
  }

  // ============================================
  // EMAIL SYSTEM SETTINGS CONFIGURATION
  // ============================================
  emailSettings: {
    // Provider configuration
    provider: {
      type: "smtp" | "sendgrid" | "resend" | "ses" | "postmark" | "mailgun"
      config: {
        // SMTP
        smtp?: {
          host: string
          port: number
          secure: boolean
          username: string
          password: string                // Encrypted
          tlsVersion: string
        }
        // SendGrid
        sendgrid?: {
          apiKey: string
        }
        // Resend
        resend?: {
          apiKey: string
        }
        // AWS SES
        ses?: {
          accessKeyId: string
          secretAccessKey: string
          region: string
        }
      }
      fallbackProvider: string            // Fallback if primary fails
      testMode: boolean                   // Send to test inbox only
    }

    // Sending defaults
    defaults: {
      fromName: string
      fromEmail: string
      replyToEmail: string
      bccEmail: string                    // BCC all emails (compliance)
      maxRecipientsPerEmail: number
      maxEmailsPerHour: number
      maxEmailsPerDay: number
    }

    // Templates
    templates: {
      enabled: boolean
      customTemplatesAllowed: boolean
      defaultTemplateEngine: "handlebars" | "mjml" | "react_email"
      previewEnabled: boolean
      testSendEnabled: boolean
      versionHistory: boolean
      maxTemplates: number
    }

    // Queue & delivery
    queue: {
      enabled: boolean
      maxRetries: number
      retryDelay: number                  // Seconds
      priorityLevels: number
      batchSize: number
      processingInterval: number          // Seconds
    }

    // Tracking
    tracking: {
      openTracking: boolean
      clickTracking: boolean
      unsubscribeTracking: boolean
      bounceHandling: boolean
      complaintHandling: boolean
      deliveryWebhook: string
    }

    // Suppression
    suppression: {
      enabled: boolean
      autoSuppressBounces: boolean
      autoSuppressComplaints: boolean
      globalSuppressionList: string[]
      maxBounceRate: number               // Percentage before alert
      maxComplaintRate: number
    }
  }

  // ============================================
  // INTEGRATION SETTINGS CONFIGURATION
  // ============================================
  integrationSettings: {
    // Webhooks
    webhooks: {
      enabled: boolean
      maxWebhooksPerOrg: number
      allowedEvents: string[]
      secretSigning: boolean              // Sign webhook payloads
      retryPolicy: {
        maxRetries: number
        retryDelay: number[]              // [1, 5, 30, 60] seconds
        timeoutSeconds: number
      }
      deliveryLog: boolean
      deliveryLogRetention: number        // Days
    }

    // API access
    apiAccess: {
      enabled: boolean
      rateLimiting: {
        enabled: boolean
        requestsPerMinute: number
        requestsPerHour: number
        requestsPerDay: number
        burstLimit: number
      }
      authentication: {
        methods: ("api_key" | "oauth2" | "jwt")[]
        apiKeyPrefix: string              // "bf_" for BlueprintForge
        tokenExpiry: number               // Hours
      }
      cors: {
        enabled: boolean
        allowedOrigins: string[]
        allowedMethods: string[]
        allowedHeaders: string[]
        maxAge: number
      }
      versioning: {
        currentVersion: string            // "v1"
        supportedVersions: string[]
        deprecatedVersions: string[]
        sunsetDate: Record<string, Date>
      }
    }

    // Third-party integrations
    thirdParty: {
      slack: {
        enabled: boolean
        clientId: string
        clientSecret: string
        webhookUrl: string
        botToken: string
        channels: Record<string, string>
      }
      github: {
        enabled: boolean
        appId: string
        privateKey: string
        webhookSecret: string
        installationId: string
      }
      jira: {
        enabled: boolean
        baseUrl: string
        email: string
        apiToken: string
        projectKey: string
      }
      figma: {
        enabled: boolean
        accessToken: string
        teamId: string
      }
      zapier: {
        enabled: boolean
        webhookUrl: string
      }
      custom: {
        enabled: boolean
        integrations: {
          id: string
          name: string
          type: "rest" | "graphql" | "grpc" | "websocket"
          baseUrl: string
          authentication: Record<string, string>
          headers: Record<string, string>
          enabled: boolean
        }[]
      }
    }

    // SSO / SAML
    sso: {
      enabled: boolean                    // Plan-gated (Enterprise)
      providers: {
        saml: {
          enabled: boolean
          entityId: string
          ssoUrl: string
          sloUrl: string
          x509Certificate: string
          signatureAlgorithm: string
          nameIdFormat: string
          attributeMapping: Record<string, string>
          allowedDomains: string[]
          autoProvision: boolean          // Auto-create users on first SSO login
          autoDeprovision: boolean        // Deactivate when removed from IdP
          defaultRole: string
          jitProvisioning: boolean        // Just-in-time provisioning
        }
        oidc: {
          enabled: boolean
          issuer: string
          clientId: string
          clientSecret: string
          scopes: string[]
          callbackUrl: string
          autoProvision: boolean
          defaultRole: string
        }
        ldap: {
          enabled: boolean
          server: string
          port: number
          baseDn: string
          bindDn: string
          bindPassword: string
          searchFilter: string
          attributeMapping: Record<string, string>
          syncInterval: number            // Minutes
          groupMapping: Record<string, string>
        }
      }
      enforceSSOForDomains: string[]      // Force SSO for these email domains
      allowPasswordFallback: boolean      // Allow password login when SSO is enabled
    }
  }

  // ============================================
  // SYSTEM CONFIGURATION (ADMIN ONLY)
  // ============================================
  systemConfig: {
    // Application settings
    application: {
      appName: string
      appUrl: string
      appDescription: string
      maintenanceMode: boolean
      maintenanceMessage: string
      maintenanceAllowedIPs: string[]     // IPs that can access during maintenance
      debugMode: boolean
      logLevel: "error" | "warn" | "info" | "debug" | "trace"
      environment: "production" | "staging" | "development"
    }

    // Performance
    performance: {
      cacheEnabled: boolean
      cacheProvider: "redis" | "memory" | "none"
      cacheDefaultTTL: number             // Seconds
      compressionEnabled: boolean
      minifyAssets: boolean
      cdnEnabled: boolean
      cdnUrl: string
      imageOptimization: boolean
      lazyLoading: boolean
      prefetching: boolean
    }

    // Security
    security: {
      csrfProtection: boolean
      xssProtection: boolean
      contentSecurityPolicy: string
      strictTransportSecurity: boolean
      referrerPolicy: string
      permissionsPolicy: string
      rateLimiting: {
        enabled: boolean
        globalLimit: number               // Requests per minute
        perUserLimit: number
        perIPLimit: number
        burstLimit: number
      }
      ipBlacklist: string[]
      ipWhitelist: string[]
      geoBlocking: {
        enabled: boolean
        blockedCountries: string[]
        allowedCountries: string[]
      }
      bruteForceProtection: {
        enabled: boolean
        maxAttempts: number
        lockoutDuration: number           // Minutes
        progressiveLockout: boolean
      }
    }

    // Database
    database: {
      connectionPoolSize: number
      queryTimeout: number                // Seconds
      slowQueryThreshold: number          // Milliseconds
      logSlowQueries: boolean
      autoVacuum: boolean
      backupSchedule: string              // Cron expression
      backupRetention: number             // Days
    }

    // Storage
    storage: {
      provider: "local" | "s3" | "gcs" | "azure_blob"
      config: Record<string, string>
      maxUploadSize: number               // MB
      allowedFileTypes: string[]
      virusScanEnabled: boolean
      encryptAtRest: boolean
    }

    // Monitoring
    monitoring: {
      enabled: boolean
      provider: "datadog" | "newrelic" | "sentry" | "custom"
      errorTracking: boolean
      performanceMonitoring: boolean
      uptimeMonitoring: boolean
      alerting: {
        enabled: boolean
        channels: ("email" | "slack" | "pagerduty" | "webhook")[]
        thresholds: {
          errorRate: number               // Percentage
          responseTime: number            // Milliseconds
          cpuUsage: number                // Percentage
          memoryUsage: number             // Percentage
          diskUsage: number               // Percentage
        }
      }
    }

    // Jobs & scheduling
    jobs: {
      enabled: boolean
      scheduler: "cron" | "bull" | "custom"
      jobs: {
        id: string
        name: string
        schedule: string                  // Cron expression
        enabled: boolean
        timeout: number                   // Seconds
        retries: number
        lastRun: Date
        nextRun: Date
        status: "idle" | "running" | "failed" | "disabled"
      }[]
    }
  }

  // ============================================
  // UI CONFIGURATION (ADMIN)
  // ============================================
  uiConfig: {
    // Global UI settings
    global: {
      theme: {
        mode: "light" | "dark" | "system"
        primaryColor: string
        secondaryColor: string
        accentColor: string
        borderRadius: number              // px
        fontFamily: string
        fontSize: number                  // px base
        spacing: number                   // px base
      }
      layout: {
        sidebarPosition: "left" | "right"
        sidebarWidth: number
        sidebarCollapsible: boolean
        headerHeight: number
        footerEnabled: boolean
        maxContentWidth: number
        responsiveBreakpoints: {
          mobile: number
          tablet: number
          desktop: number
          widescreen: number
        }
      }
      navigation: {
        style: "sidebar" | "topbar" | "both"
        showBreadcrumbs: boolean
        showSearch: boolean
        showNotifications: boolean
        showUserMenu: boolean
        showOrgSwitcher: boolean
        menuItems: {
          id: string
          label: string
          icon: string
          href: string
          visibleToRoles: string[]
          enabled: boolean
          order: number
          badge: string | null
          children: any[]
        }[]
      }
      branding: {
        showLogo: boolean
        showAppName: boolean
        showPoweredBy: boolean
        customFooterText: string
        customHeaderHTML: string
        customCSSEnabled: boolean
        customCSS: string
        customJSEnabled: boolean
        customJS: string
      }
    }

    // Per-page configuration
    pages: {
      dashboard: {
        widgets: {
          id: string
          type: string
          title: string
          enabled: boolean
          position: { x: number, y: number, w: number, h: number }
          visibleToRoles: string[]
          config: Record<string, any>
        }[]
        defaultLayout: string
        allowUserCustomization: boolean
      }
      projects: {
        defaultView: "grid" | "list" | "kanban"
        showFilters: boolean
        showSearch: boolean
        showCreateButton: boolean
        itemsPerPage: number
        sortOptions: string[]
        filterOptions: string[]
      }
      // ... similar for each page
    }

    // Component-level configuration
    components: {
      buttons: {
        defaultVariant: "default" | "outline" | "ghost"
        defaultSize: "sm" | "md" | "lg"
        showIcons: boolean
        roundedCorners: boolean
      }
      cards: {
        showShadow: boolean
        borderStyle: "solid" | "dashed" | "none"
        hoverEffect: boolean
      }
      tables: {
        striped: boolean
        hoverable: boolean
        bordered: boolean
        compact: boolean
        stickyHeader: boolean
        showPagination: boolean
        defaultPageSize: number
      }
      forms: {
        labelPosition: "top" | "left" | "floating"
        showRequiredIndicator: boolean
        showHelperText: boolean
        showCharacterCount: boolean
        autoSave: boolean
        autoSaveDelay: number             // Seconds
      }
      modals: {
        closeOnOverlayClick: boolean
        closeOnEscape: boolean
        showCloseButton: boolean
        animation: "fade" | "slide" | "scale" | "none"
      }
    }
  }

  // ============================================
  // PRIVACY & DATA SETTINGS
  // ============================================
  privacySettings: {
    // GDPR compliance
    gdpr: {
      enabled: boolean
      cookieConsent: {
        enabled: boolean
        style: "banner" | "modal" | "floating"
        position: "top" | "bottom"
        categories: {
          necessary: { enabled: true, userCanDisable: false }
          analytics: { enabled: boolean, userCanDisable: boolean, defaultEnabled: boolean }
          marketing: { enabled: boolean, userCanDisable: boolean, defaultEnabled: boolean }
          functional: { enabled: boolean, userCanDisable: boolean, defaultEnabled: boolean }
        }
        consentExpiry: number             // Days
        showOnEveryVisit: boolean
        blockScriptsUntilConsent: boolean
      }
      dataPortability: {
        enabled: boolean
        exportFormats: ("json" | "csv" | "xml")[]
        includeAllData: boolean
        processingTime: number            // Hours
        notifyOnComplete: boolean
      }
      rightToErasure: {
        enabled: boolean
        processingTime: number            // Days
        requireVerification: boolean
        notifyAdmin: boolean
        excludeFromErasure: string[]      // Data types that can't be erased (legal)
      }
      dataProcessingAgreement: {
        enabled: boolean
        version: string
        url: string
        requireAcceptance: boolean
      }
    }

    // CCPA compliance
    ccpa: {
      enabled: boolean
      doNotSellLink: boolean
      optOutMechanism: boolean
      privacyNotice: string
    }

    // Data classification
    dataClassification: {
      enabled: boolean
      levels: {
        public: { label: string, color: string, description: string }
        internal: { label: string, color: string, description: string }
        confidential: { label: string, color: string, description: string }
        restricted: { label: string, color: string, description: string }
      }
      defaultLevel: string
      requireClassification: boolean      // Must classify all data
    }

    // Data encryption
    encryption: {
      atRest: boolean
      inTransit: boolean
      algorithm: string                   // "AES-256-GCM"
      keyRotation: boolean
      keyRotationInterval: number         // Days
      fieldLevelEncryption: {
        enabled: boolean
        encryptedFields: string[]         // ["apiKey", "password", "ssn"]
      }
    }

    // Anonymization
    anonymization: {
      enabled: boolean
      anonymizeOnExport: boolean
      anonymizeInLogs: boolean
      piiFields: string[]                 // Fields considered PII
      anonymizationMethod: "hash" | "mask" | "redact" | "pseudonymize"
    }
  }

  // ============================================
  // LOCALIZATION SETTINGS
  // ============================================
  localizationSettings: {
    // Languages
    languages: {
      defaultLanguage: string             // "en"
      availableLanguages: {
        code: string                      // "en", "es", "fr"
        name: string                      // "English", "Español", "Français"
        nativeName: string                // "English", "Español", "Français"
        direction: "ltr" | "rtl"
        enabled: boolean
        completionPercentage: number      // Translation completion
      }[]
      fallbackLanguage: string
      autoDetect: boolean
      allowUserOverride: boolean
      showLanguageSwitcher: boolean
      translationProvider: "i18next" | "formatjs" | "custom"
    }

    // Regional settings
    regional: {
      defaultTimezone: string
      defaultDateFormat: string
      defaultTimeFormat: string
      defaultNumberFormat: string
      defaultCurrency: string
      firstDayOfWeek: number              // 0=Sun, 1=Mon
      measurementSystem: "metric" | "imperial"
    }

    // Content localization
    content: {
      localizeEmails: boolean
      localizeNotifications: boolean
      localizeErrorMessages: boolean
      localizeUILabels: boolean
      localizeHelpDocs: boolean
      userGeneratedContentTranslation: boolean
      autoTranslateEnabled: boolean
      autoTranslateProvider: string
    }
  }

  // ============================================
  // PLAN-BASED RESTRICTIONS
  // ============================================
  planRestrictions: {
    enabled: boolean
    plans: {
      FREE: {
        settingsAccess: {
          profile: "full"
          security: "basic"               // No API keys, no SSO
          notifications: "basic"          // Email only
          organization: "basic"           // Name, slug only
          billing: "view_only"
          ai: "none"
          email: "none"
          integrations: "none"
          system: "none"
          ui: "none"
          privacy: "basic"
        }
        restrictions: {
          noCustomBranding: true
          noCustomDomain: true
          noSSO: true
          noAPIAccess: true
          noWebhooks: true
          noAuditLogs: true
          noAdvancedSecurity: true
          noCustomRoles: true
        }
      }
      STARTER: {
        settingsAccess: {
          profile: "full"
          security: "standard"            // API keys, no SSO
          notifications: "full"
          organization: "standard"        // + branding basics
          billing: "full"
          ai: "basic"                     // Limited models
          email: "basic"                  // Default provider
          integrations: "basic"           // Webhooks only
          system: "none"
          ui: "basic"
          privacy: "standard"
        }
      }
      PRO: {
        settingsAccess: {
          profile: "full"
          security: "full"                // + SSO
          notifications: "full"
          organization: "full"            // + custom domain
          billing: "full"
          ai: "full"
          email: "full"
          integrations: "full"
          system: "limited"               // View only
          ui: "full"
          privacy: "full"
        }
      }
      ENTERPRISE: {
        settingsAccess: {
          profile: "full"
          security: "full"
          notifications: "full"
          organization: "full"
          billing: "full"
          ai: "full"
          email: "full"
          integrations: "full"
          system: "full"
          ui: "full"
          privacy: "full"
        }
      }
    }
  }

  // ============================================
  // VISIBILITY & CONDITIONAL RENDERING
  // ============================================
  visibility: {
    // Settings navigation visibility
    navigation: {
      profile: { visibleToRoles: ["*"], visibleToPlans: ["*"] }
      security: { visibleToRoles: ["*"], visibleToPlans: ["*"] }
      notifications: { visibleToRoles: ["*"], visibleToPlans: ["*"] }
      organization: { visibleToRoles: ["OWNER", "ADMIN"], visibleToPlans: ["*"] }
      billing: { visibleToRoles: ["OWNER", "ADMIN"], visibleToPlans: ["*"] }
      aiProviders: { visibleToRoles: ["OWNER", "ADMIN"], visibleToPlans: ["STARTER", "PRO", "ENTERPRISE"] }
      email: { visibleToRoles: ["OWNER", "ADMIN"], visibleToPlans: ["STARTER", "PRO", "ENTERPRISE"] }
      integrations: { visibleToRoles: ["OWNER", "ADMIN"], visibleToPlans: ["STARTER", "PRO", "ENTERPRISE"] }
      system: { visibleToRoles: ["OWNER"], visibleToPlans: ["ENTERPRISE"] }
      uiConfig: { visibleToRoles: ["OWNER", "ADMIN"], visibleToPlans: ["PRO", "ENTERPRISE"] }
      privacy: { visibleToRoles: ["OWNER", "ADMIN"], visibleToPlans: ["*"] }
    }

    // Individual setting visibility
    settingVisibility: {
      // Each setting can have its own visibility rules
      pattern: {
        settingKey: string
        visibleToRoles: string[]
        visibleToPlans: string[]
        editableByRoles: string[]
        editableByPlans: string[]
        lockedByAdmin: boolean
        lockedReason: string
      }
    }

    // Conditional rendering
    conditionalRendering: {
      showLockedSettings: boolean         // Show locked settings as disabled
      showPlanGatedSettings: boolean      // Show plan-gated settings with upgrade CTA
      showUpgradePrompt: boolean
      upgradePromptStyle: "inline" | "modal" | "banner"
      customLockedMessage: string
      customUpgradeMessage: string
    }
  }
}
```

---

## Settings Hierarchy & Scope

### Scope Resolution Model

```typescript
interface SettingsScopeResolution {
  // ============================================
  // SCOPE LEVELS
  // ============================================
  levels: {
    // Level 0: System defaults (hardcoded, always present)
    system: {
      priority: 0
      source: "code"
      editable: false                     // Cannot be changed at runtime
      description: "Hardcoded defaults in application code"
      example: {
        "theme.default": "system",
        "session.timeout": 60,
        "password.minLength": 12,
      }
    }

    // Level 1: Platform settings (OWNER configures for entire platform)
    platform: {
      priority: 1
      source: "database"
      editable: true
      editableBy: ["PLATFORM_ADMIN", "OWNER"]
      description: "Platform-wide overrides, applies to all organizations"
      example: {
        "registration.enabled": true,
        "maintenance.mode": false,
        "security.bruteForce.maxAttempts": 5,
      }
    }

    // Level 2: Organization settings (ADMIN configures for their org)
    organization: {
      priority: 2
      source: "database"
      editable: true
      editableBy: ["OWNER", "ADMIN"]
      description: "Organization-specific settings"
      example: {
        "branding.primaryColor": "#3b82f6",
        "features.aiGeneration": true,
        "members.maxUsers": 50,
      }
    }

    // Level 3: Team settings (optional, Team Lead configures)
    team: {
      priority: 3
      source: "database"
      editable: true
      editableBy: ["OWNER", "ADMIN", "TEAM_LEAD"]
      description: "Team/department-specific overrides"
      example: {
        "notifications.digest": "daily",
        "dashboard.defaultView": "kanban",
      }
    }

    // Level 4: User settings (user configures for themselves)
    user: {
      priority: 4                         // Highest priority
      source: "database"
      editable: true
      editableBy: ["self"]
      description: "Personal user preferences"
      example: {
        "theme.mode": "dark",
        "language": "es",
        "timezone": "America/New_York",
        "notifications.email": false,
      }
    }
  }

  // ============================================
  // RESOLUTION ALGORITHM
  // ============================================
  resolve: (settingKey: string, context: SettingsContext) => ResolvedSetting
}

// Resolution implementation
function resolveSetting(
  settingKey: string,
  context: { userId: string, orgId: string, teamId?: string }
): ResolvedSetting {
  // Step 1: Check if setting is locked by admin
  const lock = getSettingLock(settingKey, context.orgId)
  if (lock) {
    return {
      value: lock.value,
      source: "admin_locked",
      editable: false,
      lockedBy: lock.lockedBy,
      lockedReason: lock.reason,
    }
  }

  // Step 2: Check user-level value
  const userValue = getUserSetting(settingKey, context.userId)
  if (userValue !== undefined) {
    return {
      value: userValue,
      source: "user",
      editable: true,
      overriddenFrom: getNextLevelValue(settingKey, context),
    }
  }

  // Step 3: Check team-level value
  if (context.teamId) {
    const teamValue = getTeamSetting(settingKey, context.teamId)
    if (teamValue !== undefined) {
      return {
        value: teamValue,
        source: "team",
        editable: isUserOverrideAllowed(settingKey),
      }
    }
  }

  // Step 4: Check organization-level value
  const orgValue = getOrgSetting(settingKey, context.orgId)
  if (orgValue !== undefined) {
    return {
      value: orgValue,
      source: "organization",
      editable: isUserOverrideAllowed(settingKey),
    }
  }

  // Step 5: Check platform-level value
  const platformValue = getPlatformSetting(settingKey)
  if (platformValue !== undefined) {
    return {
      value: platformValue,
      source: "platform",
      editable: isUserOverrideAllowed(settingKey),
    }
  }

  // Step 6: Return system default
  const systemDefault = getSystemDefault(settingKey)
  return {
    value: systemDefault,
    source: "system_default",
    editable: isUserOverrideAllowed(settingKey),
  }
}
```

### Setting Lock Model

```typescript
interface SettingLock {
  // Admin can lock settings to prevent user/team override
  settingKey: string
  organizationId: string
  lockedValue: any                        // The enforced value
  lockedBy: string                        // Admin who locked it
  lockedAt: Date
  reason: string                          // "Company policy requires dark mode"
  scope: "organization" | "team"          // Lock at org or team level
  affectedUsers: number                   // How many users are affected
}

// ============================================
// LOCK SCENARIOS
// ============================================

// Scenario 1: Admin locks theme to dark mode for entire org
// Route: POST /api/admin/settings/lock
{
  settingKey: "theme.mode",
  value: "dark",
  scope: "organization",
  reason: "Brand consistency requirement"
}
// Result: All users in org see dark mode, cannot change it
// UI shows: "This setting is locked by your admin" with lock icon

// Scenario 2: Admin locks notification settings
{
  settingKey: "notifications.security.enabled",
  value: true,
  scope: "organization",
  reason: "Security notifications cannot be disabled per company policy"
}
// Result: Users cannot disable security notifications

// Scenario 3: Admin unlocks a previously locked setting
// Route: DELETE /api/admin/settings/lock
{
  settingKey: "theme.mode",
  reason: "Allowing user preference"
}
// Result: Users can now change their theme
```

---

## User Profile Settings

### Profile Settings Scenarios

```typescript
// ============================================
// SCENARIO 1: User Updates Profile Information
// ============================================
// Route: PATCH /api/user/profile
{
  fullName: "John Doe",
  displayName: "John",
  phone: "+1 (555) 123-4567",
  bio: "Product Manager passionate about building great software.",
  position: "Product Manager",
  department: "Engineering",
  timezone: "America/New_York",
  language: "en"
}
// Validation:
// - fullName: required, 2-100 chars, no special chars
// - displayName: optional, 2-50 chars
// - phone: valid phone format (E.164)
// - bio: optional, max 500 chars
// - position: optional, max 100 chars
// - department: optional, from allowed list or free text
// - timezone: valid IANA timezone
// - language: from available languages list

// ============================================
// SCENARIO 2: User Updates Avatar
// ============================================
// Route: POST /api/user/avatar
// Content-Type: multipart/form-data
// Body: file (image)
// Validation:
// - Max size: 5MB (configurable by admin)
// - Formats: jpg, png, webp (configurable)
// - Min dimensions: 100x100 (configurable)
// - Auto-crop to square
// - Auto-resize to multiple sizes (32, 64, 128, 256)
// - Store in configured storage provider
// - Delete previous avatar

// ============================================
// SCENARIO 3: User Changes Email
// ============================================
// Route: POST /api/user/email/change
{
  newEmail: "newemail@example.com",
  password: "current-password"            // Verify identity
}
// Flow:
// 1. Verify current password
// 2. Check new email is not taken
// 3. Send verification email to NEW address
// 4. User clicks verification link
// 5. Email is updated
// 6. Send notification to OLD email ("Your email was changed")
// 7. Audit log entry

// ============================================
// SCENARIO 4: User Updates Preferences
// ============================================
// Route: PATCH /api/user/preferences
{
  theme: "dark",
  language: "es",
  timezone: "Europe/Madrid",
  dateFormat: "DD/MM/YYYY",
  timeFormat: "24h",
  dashboard: {
    defaultView: "kanban",
    itemsPerPage: 25,
    sidebarCollapsed: false
  },
  accessibility: {
    reducedMotion: true,
    fontSize: "large"
  }
}

// ============================================
// SCENARIO 5: Admin Configures Required Profile Fields
// ============================================
// Route: PATCH /api/admin/settings/profile
{
  requiredFields: ["fullName", "department", "position"],
  hiddenFields: ["dateOfBirth"],
  readOnlyFields: ["email"],              // Email change has separate flow
  customFields: [
    {
      id: "employee_id",
      label: "Employee ID",
      type: "text",
      required: true,
      validation: "^EMP-[0-9]{6}$",
      placeholder: "EMP-000000"
    },
    {
      id: "team",
      label: "Team",
      type: "select",
      required: true,
      options: ["Frontend", "Backend", "DevOps", "Design", "Product"]
    }
  ]
}

// ============================================
// SCENARIO 6: User Tries to Edit Locked Field
// ============================================
// User tries to change department, but admin locked it
// Response: 403
{
  error: "This field is locked by your administrator",
  field: "department",
  lockedBy: "admin@company.com",
  reason: "Department assignments are managed by HR"
}
```

---

## Account Security Settings

### Security Settings Scenarios

```typescript
// ============================================
// SCENARIO 1: User Changes Password
// ============================================
// Route: PUT /api/user/password
{
  currentPassword: "old-password",
  newPassword: "new-secure-password-123!"
}
// Validation:
// - Current password must be correct
// - New password must meet policy (min 12 chars, uppercase, lowercase, number, special)
// - New password cannot be same as current
// - New password cannot be in last N passwords (configurable)
// - New password cannot contain user's name or email
// - Password strength score >= 3 (zxcvbn)
// Response on success:
{
  success: true,
  message: "Password updated successfully",
  nextExpiryDate: "2026-05-15T00:00:00Z"  // If password expiry is enabled
}

// ============================================
// SCENARIO 2: User Enables 2FA (TOTP)
// ============================================
// Step 1: Generate secret
// Route: POST /api/user/2fa/setup
{
  method: "totp"
}
// Response:
{
  secret: "JBSWY3DPEHPK3PXP",
  qrCodeUrl: "otpauth://totp/BlueprintForge:john@example.com?secret=...",
  qrCodeImage: "data:image/png;base64,...",
  backupCodes: ["12345678", "23456789", "34567890", ...]  // 10 backup codes
}

// Step 2: Verify setup
// Route: POST /api/user/2fa/verify
{
  code: "123456",                         // From authenticator app
  method: "totp"
}
// Response:
{
  success: true,
  message: "Two-factor authentication enabled",
  backupCodesRemaining: 10
}

// ============================================
// SCENARIO 3: User Disables 2FA
// ============================================
// Route: DELETE /api/user/2fa
{
  password: "current-password",           // Verify identity
  code: "123456"                          // Current 2FA code
}
// Validation:
// - If admin requires 2FA for user's role, cannot disable
// Response if required:
{
  error: "Two-factor authentication is required for your role (ADMIN)",
  requiredBy: "organization_policy"
}

// ============================================
// SCENARIO 4: User Views Active Sessions
// ============================================
// Route: GET /api/user/sessions
// Response:
{
  sessions: [
    {
      id: "sess-1",
      device: "Chrome on Windows",
      ip: "203.0.113.1",
      location: "New York, US",
      lastActive: "2026-02-15T15:00:00Z",
      isCurrent: true,
      createdAt: "2026-02-15T10:00:00Z"
    },
    {
      id: "sess-2",
      device: "Safari on iPhone",
      ip: "203.0.113.2",
      location: "New York, US",
      lastActive: "2026-02-14T20:00:00Z",
      isCurrent: false,
      createdAt: "2026-02-14T08:00:00Z"
    }
  ],
  maxSessions: 5
}

// ============================================
// SCENARIO 5: User Revokes a Session
// ============================================
// Route: DELETE /api/user/sessions/sess-2
// Response:
{
  success: true,
  message: "Session revoked. The device will be signed out."
}

// ============================================
// SCENARIO 6: User Revokes All Other Sessions
// ============================================
// Route: POST /api/user/sessions/revoke-all
{
  password: "current-password"            // Verify identity
}
// Response:
{
  success: true,
  revokedCount: 3,
  message: "All other sessions have been revoked"
}

// ============================================
// SCENARIO 7: User Manages API Keys
// ============================================
// Create API key
// Route: POST /api/user/api-keys
{
  name: "CI/CD Pipeline",
  scopes: ["projects:read", "blueprints:read", "blueprints:write"],
  expiresAt: "2026-06-15T00:00:00Z"
}
// Response:
{
  apiKey: {
    id: "key-clx...",
    name: "CI/CD Pipeline",
    key: "bf_live_abc123...",             // Only shown once!
    prefix: "bf_live_abc1",              // Shown in list for identification
    scopes: ["projects:read", "blueprints:read", "blueprints:write"],
    expiresAt: "2026-06-15T00:00:00Z",
    createdAt: "2026-02-15T15:00:00Z",
    lastUsedAt: null
  },
  warning: "Save this API key now. You won't be able to see it again."
}

// List API keys
// Route: GET /api/user/api-keys
{
  apiKeys: [
    {
      id: "key-clx...",
      name: "CI/CD Pipeline",
      prefix: "bf_live_abc1",
      scopes: ["projects:read", "blueprints:read", "blueprints:write"],
      expiresAt: "2026-06-15T00:00:00Z",
      createdAt: "2026-02-15T15:00:00Z",
      lastUsedAt: "2026-02-15T16:30:00Z"
    }
  ]
}

// Revoke API key
// Route: DELETE /api/user/api-keys/key-clx...
{
  success: true,
  message: "API key revoked"
}

// ============================================
// SCENARIO 8: User Manages Connected Accounts
// ============================================
// Link Google account
// Route: POST /api/user/connected-accounts/google
// Redirects to Google OAuth flow
// On success: account linked, can use Google to sign in

// Unlink GitHub account
// Route: DELETE /api/user/connected-accounts/github
// Validation: Must have at least one auth method (password or another provider)
// Response if last method:
{
  error: "Cannot unlink GitHub. It's your only authentication method. Set a password first.",
  code: "LAST_AUTH_METHOD"
}

// ============================================
// SCENARIO 9: User Requests Account Deletion
// ============================================
// Route: POST /api/user/account/delete
{
  password: "current-password",
  reason: "No longer using the service",
  feedback: "The product was great but I'm switching to a competitor"
}
// Response:
{
  success: true,
  deletionScheduledAt: "2026-03-17T15:00:00Z",  // 30-day grace period
  message: "Your account will be permanently deleted on March 17, 2026. You can cancel this by logging in before that date.",
  dataExportUrl: "/api/user/data-export"  // Offer data export
}

// ============================================
// SCENARIO 10: Admin Configures Security Settings for Org
// ============================================
// Route: PATCH /api/admin/settings/security
{
  password: {
    minLength: 14,
    requireUppercase: true,
    requireLowercase: true,
    requireNumbers: true,
    requireSpecialChars: true,
    expiryDays: 90,
    preventReuse: 12
  },
  twoFactor: {
    required: true,
    requiredForRoles: ["OWNER", "ADMIN"],
    gracePeriodDays: 7,
    methods: {
      totp: { enabled: true },
      sms: { enabled: false },            // Disabled for security
      webauthn: { enabled: true }
    }
  },
  sessions: {
    maxConcurrentSessions: 3,
    sessionTimeout: 30,                   // 30 minutes for admins
    idleTimeout: 15
  },
  loginHistory: {
    alertOnNewDevice: true,
    alertOnNewLocation: true,
    alertThreshold: 3
  }
}
```

---

## Organization Settings

### Organization Settings Scenarios

```typescript
// ============================================
// SCENARIO 1: Admin Updates Organization Profile
// ============================================
// Route: PATCH /api/organization
{
  name: "Acme Corporation",
  description: "Building the future of software development",
  website: "https://acme.com",
  industry: "Technology",
  size: "51-200",
  contactEmail: "admin@acme.com"
}

// ============================================
// SCENARIO 2: Admin Configures Branding
// ============================================
// Route: PATCH /api/admin/settings/branding
{
  logo: {
    lightMode: "https://storage.example.com/logos/acme-light.png",
    darkMode: "https://storage.example.com/logos/acme-dark.png",
    favicon: "https://storage.example.com/logos/acme-favicon.ico"
  },
  colors: {
    primary: "#3b82f6",
    secondary: "#10b981",
    accent: "#f59e0b"
  },
  emailBranding: {
    fromName: "Acme Corporation",
    fromEmail: "noreply@acme.com",
    headerLogo: "https://storage.example.com/logos/acme-email.png",
    footerText: "© 2026 Acme Corporation. All rights reserved."
  }
}

// ============================================
// SCENARIO 3: Admin Configures Custom Domain
// ============================================
// Route: POST /api/admin/settings/custom-domain
{
  domain: "app.acme.com"
}
// Response:
{
  domain: "app.acme.com",
  status: "pending_verification",
  verificationMethod: "dns_cname",
  verificationRecord: {
    type: "CNAME",
    name: "app",
    value: "custom.blueprintforge.com",
    ttl: 3600
  },
  instructions: "Add this CNAME record to your DNS provider. Verification may take up to 48 hours."
}

// Verify domain
// Route: POST /api/admin/settings/custom-domain/verify
{
  domain: "app.acme.com"
}
// Response:
{
  verified: true,
  sslStatus: "provisioning",             // Auto-provision SSL
  estimatedReady: "2026-02-15T16:00:00Z"
}

// ============================================
// SCENARIO 4: Admin Configures Member Invitation Settings
// ============================================
// Route: PATCH /api/admin/settings/invitations
{
  whoCanInvite: ["OWNER", "ADMIN", "MEMBER"],
  requireApproval: false,
  inviteExpiry: 7,                        // Days
  maxPendingInvites: 50,
  allowBulkInvite: true,
  defaultRole: "MEMBER",
  allowRoleSelection: true,
  maxRoleForInviter: "MEMBER",            // Members can only invite as MEMBER
  domainRestriction: {
    enabled: true,
    allowedDomains: ["acme.com", "acme.io"]
  }
}

// ============================================
// SCENARIO 5: Admin Configures Feature Toggles
// ============================================
// Route: PATCH /api/admin/settings/features
{
  projects: true,
  blueprints: true,
  mindmaps: true,
  aiGeneration: true,
  collaboration: true,
  export: true,
  import: true,
  apiAccess: true,
  webhooks: false,                        // Disable webhooks for this org
  customDomain: true,
  sso: false,                             // Not on current plan
  auditLogs: true,
  advancedRBAC: true,
  customBranding: true
}

// ============================================
// SCENARIO 6: Admin Configures Resource Limits
// ============================================
// Route: PATCH /api/admin/settings/limits
{
  maxProjects: 100,
  maxBlueprints: 200,
  maxMindMaps: 50,
  maxAICreditsPerMonth: 5000,
  maxStorageGB: 50,
  maxFileUploadMB: 100,
  maxAPIRequestsPerDay: 10000,
  maxWebhooks: 20,
  maxCustomRoles: 25,
  maxIntegrations: 10
}
// Validation:
// - Cannot exceed plan limits
// - Cannot reduce below current usage
// Response if exceeds plan:
{
  error: "Cannot set maxProjects to 100. Your PRO plan allows up to 50.",
  currentPlan: "PRO",
  planLimit: 50,
  upgradeUrl: "/settings/billing"
}

// ============================================
// SCENARIO 7: Admin Configures Data Retention
// ============================================
// Route: PATCH /api/admin/settings/data-retention
{
  defaultRetentionDays: 365,
  deletedItemRetention: 30,               // Keep deleted items for 30 days
  auditLogRetention: 730,                 // 2 years
  activityLogRetention: 90,
  sessionLogRetention: 30,
  autoArchive: true,
  archiveAfterDays: 180
}

// ============================================
// SCENARIO 8: Admin Transfers Organization Ownership
// ============================================
// Route: POST /api/admin/organization/transfer
{
  newOwnerId: "user-clx...",
  password: "current-owner-password",     // Verify identity
  reason: "Leaving the company"
}
// Flow:
// 1. Verify current owner's password
// 2. Verify new owner is ADMIN in the org
// 3. Send confirmation email to new owner
// 4. New owner accepts transfer
// 5. Roles are swapped (old OWNER → ADMIN, new user → OWNER)
// 6. Audit log entry
// 7. Notify all admins
```

---

## Admin Settings Dashboard

### Admin Settings Interface

```typescript
interface AdminSettingsDashboard {
  // ============================================
  // ADMIN SETTINGS PAGES
  // ============================================

  // Route: /admin/system (System Configuration)
  systemPage: {
    tabs: [
      {
        id: "general",
        label: "General",
        icon: "Settings",
        sections: [
          "Application Name & URL",
          "Maintenance Mode",
          "Debug Mode & Log Level",
          "Environment Configuration"
        ]
      },
      {
        id: "security",
        label: "Security",
        icon: "Shield",
        sections: [
          "CSRF & XSS Protection",
          "Content Security Policy",
          "Rate Limiting",
          "IP Blacklist/Whitelist",
          "Geo Blocking",
          "Brute Force Protection"
        ]
      },
      {
        id: "performance",
        label: "Performance",
        icon: "Zap",
        sections: [
          "Cache Configuration",
          "CDN Settings",
          "Compression",
          "Image Optimization",
          "Lazy Loading & Prefetching"
        ]
      },
      {
        id: "database",
        label: "Database",
        icon: "Database",
        sections: [
          "Connection Pool",
          "Query Timeout",
          "Slow Query Logging",
          "Backup Schedule",
          "Backup Retention"
        ]
      },
      {
        id: "storage",
        label: "Storage",
        icon: "HardDrive",
        sections: [
          "Storage Provider",
          "Upload Limits",
          "Allowed File Types",
          "Virus Scanning",
          "Encryption at Rest"
        ]
      },
      {
        id: "monitoring",
        label: "Monitoring",
        icon: "Activity",
        sections: [
          "Monitoring Provider",
          "Error Tracking",
          "Performance Monitoring",
          "Uptime Monitoring",
          "Alert Configuration"
        ]
      },
      {
        id: "jobs",
        label: "Background Jobs",
        icon: "Clock",
        sections: [
          "Job Scheduler",
          "Active Jobs",
          "Job History",
          "Failed Jobs",
          "Manual Triggers"
        ]
      },
      {
        id: "health",
        label: "System Health",
        icon: "HeartPulse",
        sections: [
          "Service Status",
          "Database Health",
          "Cache Health",
          "Storage Health",
          "External Service Health",
          "Recent Errors"
        ]
      }
    ]
  }

  // Route: /admin/ui-config (UI Configuration)
  uiConfigPage: {
    tabs: [
      {
        id: "theme",
        label: "Theme",
        sections: ["Colors", "Typography", "Spacing", "Border Radius", "Shadows"]
      },
      {
        id: "layout",
        label: "Layout",
        sections: ["Sidebar", "Header", "Footer", "Content Width", "Responsive"]
      },
      {
        id: "navigation",
        label: "Navigation",
        sections: ["Menu Items", "Breadcrumbs", "Search", "User Menu"]
      },
      {
        id: "components",
        label: "Components",
        sections: ["Buttons", "Cards", "Tables", "Forms", "Modals"]
      },
      {
        id: "pages",
        label: "Pages",
        sections: ["Dashboard Widgets", "Project List", "Blueprint Editor"]
      },
      {
        id: "preview",
        label: "Preview",
        sections: ["Live Preview", "Role Preview", "Device Preview"]
      }
    ]
  }

  // Route: /settings (User Settings)
  userSettingsPage: {
    tabs: [
      { id: "profile", label: "Profile", icon: "User" },
      { id: "organization", label: "Organization", icon: "Building2" },
      { id: "security", label: "Security", icon: "Shield" },
      { id: "billing", label: "Billing", icon: "CreditCard" },
      { id: "notifications", label: "Notifications", icon: "Bell" },
      { id: "mcp", label: "AI Providers", icon: "Plug" },
      { id: "email", label: "Email", icon: "Mail" },
    ]
  }
}
```

---

## Settings API Architecture

### Complete Settings API Routes

```typescript
interface SettingsAPIRoutes {
  // ============================================
  // USER PROFILE
  // ============================================
  userProfile: {
    "GET /api/user/profile": {
      auth: "any authenticated user"
      response: { profile: UserProfile }
    }
    "PATCH /api/user/profile": {
      auth: "any authenticated user"
      body: Partial<UserProfile>
      response: { profile: UserProfile }
    }
    "POST /api/user/avatar": {
      auth: "any authenticated user"
      body: FormData                      // file upload
      response: { avatarUrl: string }
    }
    "DELETE /api/user/avatar": {
      auth: "any authenticated user"
      response: { success: boolean }
    }
    "POST /api/user/email/change": {
      auth: "any authenticated user"
      body: { newEmail: string, password: string }
      response: { verificationSent: boolean }
    }
    "POST /api/user/email/verify": {
      auth: "any authenticated user"
      body: { token: string }
      response: { success: boolean }
    }
  }

  // ============================================
  // USER PREFERENCES
  // ============================================
  userPreferences: {
    "GET /api/user/preferences": {
      auth: "any authenticated user"
      response: { preferences: UserPreferences }
    }
    "PATCH /api/user/preferences": {
      auth: "any authenticated user"
      body: Partial<UserPreferences>
      response: { preferences: UserPreferences }
    }
    "POST /api/user/preferences/reset": {
      auth: "any authenticated user"
      body: { keys?: string[] }           // Reset specific keys or all
      response: { preferences: UserPreferences }
    }
  }

  // ============================================
  // USER SECURITY
  // ============================================
  userSecurity: {
    "PUT /api/user/password": {
      auth: "any authenticated user"
      body: { currentPassword: string, newPassword: string }
      response: { success: boolean }
    }
    "POST /api/user/2fa/setup": {
      auth: "any authenticated user"
      body: { method: "totp" | "sms" | "webauthn" }
      response: { secret?: string, qrCode?: string, backupCodes?: string[] }
    }
    "POST /api/user/2fa/verify": {
      auth: "any authenticated user"
      body: { code: string, method: string }
      response: { success: boolean }
    }
    "DELETE /api/user/2fa": {
      auth: "any authenticated user"
      body: { password: string, code: string }
      response: { success: boolean }
    }
    "GET /api/user/sessions": {
      auth: "any authenticated user"
      response: { sessions: Session[] }
    }
    "DELETE /api/user/sessions/:id": {
      auth: "any authenticated user"
      response: { success: boolean }
    }
    "POST /api/user/sessions/revoke-all": {
      auth: "any authenticated user"
      body: { password: string }
      response: { revokedCount: number }
    }
    "GET /api/user/api-keys": {
      auth: "any authenticated user"
      response: { apiKeys: APIKey[] }
    }
    "POST /api/user/api-keys": {
      auth: "any authenticated user"
      body: { name: string, scopes: string[], expiresAt?: Date }
      response: { apiKey: APIKey }
    }
    "DELETE /api/user/api-keys/:id": {
      auth: "any authenticated user"
      response: { success: boolean }
    }
    "GET /api/user/connected-accounts": {
      auth: "any authenticated user"
      response: { accounts: ConnectedAccount[] }
    }
    "DELETE /api/user/connected-accounts/:provider": {
      auth: "any authenticated user"
      response: { success: boolean }
    }
    "GET /api/user/login-history": {
      auth: "any authenticated user"
      query: { page?: number, limit?: number }
      response: { history: LoginEvent[], total: number }
    }
  }

  // ============================================
  // USER NOTIFICATIONS
  // ============================================
  userNotifications: {
    "GET /api/user/notification-preferences": {
      auth: "any authenticated user"
      response: { preferences: NotificationPreferences }
    }
    "PATCH /api/user/notification-preferences": {
      auth: "any authenticated user"
      body: Partial<NotificationPreferences>
      response: { preferences: NotificationPreferences }
    }
  }

  // ============================================
  // USER ACCOUNT
  // ============================================
  userAccount: {
    "POST /api/user/account/delete": {
      auth: "any authenticated user"
      body: { password: string, reason?: string, feedback?: string }
      response: { deletionScheduledAt: Date }
    }
    "POST /api/user/account/cancel-deletion": {
      auth: "any authenticated user"
      response: { success: boolean }
    }
    "POST /api/user/account/deactivate": {
      auth: "any authenticated user"
      body: { password: string }
      response: { success: boolean }
    }
    "GET /api/user/data-export": {
      auth: "any authenticated user"
      query: { format: "json" | "csv" }
      response: File
    }
  }

  // ============================================
  // ORGANIZATION SETTINGS
  // ============================================
  organization: {
    "GET /api/organization": {
      auth: "any org member"
      response: { organization: Organization }
    }
    "PATCH /api/organization": {
      auth: "OWNER | ADMIN"
      body: Partial<Organization>
      response: { organization: Organization }
    }
    "POST /api/admin/organization/transfer": {
      auth: "OWNER"
      body: { newOwnerId: string, password: string }
      response: { success: boolean }
    }
    "DELETE /api/organization": {
      auth: "OWNER"
      body: { password: string, confirmation: string }
      response: { deletionScheduledAt: Date }
    }
  }

  // ============================================
  // ADMIN SETTINGS
  // ============================================
  adminSettings: {
    // Get all admin settings
    "GET /api/admin/settings": {
      auth: "OWNER | ADMIN"
      response: { settings: AdminSettings }
    }

    // Update specific setting category
    "PATCH /api/admin/settings/:category": {
      auth: "OWNER | ADMIN"
      body: Partial<CategorySettings>
      response: { settings: CategorySettings }
    }

    // Lock a setting
    "POST /api/admin/settings/lock": {
      auth: "OWNER | ADMIN"
      body: { settingKey: string, value: any, reason: string }
      response: { lock: SettingLock }
    }

    // Unlock a setting
    "DELETE /api/admin/settings/lock": {
      auth: "OWNER | ADMIN"
      body: { settingKey: string }
      response: { success: boolean }
    }

    // Get locked settings
    "GET /api/admin/settings/locks": {
      auth: "OWNER | ADMIN"
      response: { locks: SettingLock[] }
    }

    // Reset settings to defaults
    "POST /api/admin/settings/reset": {
      auth: "OWNER"
      body: { category?: string, keys?: string[] }
      response: { resetCount: number }
    }

    // Export settings
    "GET /api/admin/settings/export": {
      auth: "OWNER"
      query: { format: "json" }
      response: { config: SettingsExport }
    }

    // Import settings
    "POST /api/admin/settings/import": {
      auth: "OWNER"
      body: { config: SettingsExport, mode: "merge" | "replace" }
      response: { imported: number, conflicts: number }
    }

    // Settings audit log
    "GET /api/admin/settings/audit": {
      auth: "OWNER | ADMIN"
      query: { category?: string, performedBy?: string, dateRange?: string }
      response: { logs: SettingsAuditLog[] }
    }

    // Branding
    "PATCH /api/admin/settings/branding": {
      auth: "OWNER | ADMIN"
      body: BrandingSettings
      response: { branding: BrandingSettings }
    }

    // Custom domain
    "POST /api/admin/settings/custom-domain": {
      auth: "OWNER"
      body: { domain: string }
      response: { domain: CustomDomain }
    }
    "POST /api/admin/settings/custom-domain/verify": {
      auth: "OWNER"
      response: { verified: boolean }
    }
    "DELETE /api/admin/settings/custom-domain": {
      auth: "OWNER"
      response: { success: boolean }
    }

    // Feature toggles
    "PATCH /api/admin/settings/features": {
      auth: "OWNER | ADMIN"
      body: Partial<FeatureToggles>
      response: { features: FeatureToggles }
    }

    // Resource limits
    "PATCH /api/admin/settings/limits": {
      auth: "OWNER | ADMIN"
      body: Partial<ResourceLimits>
      response: { limits: ResourceLimits }
    }

    // Invitation settings
    "PATCH /api/admin/settings/invitations": {
      auth: "OWNER | ADMIN"
      body: InvitationSettings
      response: { settings: InvitationSettings }
    }

    // Security settings
    "PATCH /api/admin/settings/security": {
      auth: "OWNER | ADMIN"
      body: SecuritySettings
      response: { settings: SecuritySettings }
    }

    // Data retention
    "PATCH /api/admin/settings/data-retention": {
      auth: "OWNER"
      body: DataRetentionSettings
      response: { settings: DataRetentionSettings }
    }
  }

  // ============================================
  // SYSTEM CONFIGURATION (OWNER ONLY)
  // ============================================
  systemConfig: {
    "GET /api/admin/system/config": {
      auth: "OWNER"
      response: { config: SystemConfig }
    }
    "PATCH /api/admin/system/config": {
      auth: "OWNER"
      body: Partial<SystemConfig>
      response: { config: SystemConfig }
    }
    "GET /api/admin/system/health": {
      auth: "OWNER | ADMIN"
      response: { health: SystemHealth }
    }
    "GET /api/admin/system/jobs": {
      auth: "OWNER"
      response: { jobs: BackgroundJob[] }
    }
    "POST /api/admin/system/jobs/:id/run": {
      auth: "OWNER"
      response: { success: boolean }
    }
    "POST /api/admin/system/maintenance": {
      auth: "OWNER"
      body: { enabled: boolean, message?: string, allowedIPs?: string[] }
      response: { success: boolean }
    }
    "POST /api/admin/system/cache/clear": {
      auth: "OWNER"
      body: { scope?: "all" | "permissions" | "settings" | "sessions" }
      response: { clearedKeys: number }
    }
  }

  // ============================================
  // UI CONFIGURATION
  // ============================================
  uiConfig: {
    "GET /api/admin/ui-config": {
      auth: "OWNER | ADMIN"
      response: { config: UIConfig }
    }
    "PATCH /api/admin/ui-config": {
      auth: "OWNER | ADMIN"
      body: Partial<UIConfig>
      response: { config: UIConfig }
    }
    "POST /api/admin/ui-config/preview": {
      auth: "OWNER | ADMIN"
      body: { config: Partial<UIConfig>, role?: string }
      response: { previewUrl: string }
    }
    "POST /api/admin/ui-config/publish": {
      auth: "OWNER | ADMIN"
      body: { config: UIConfig }
      response: { success: boolean, version: number }
    }
    "POST /api/admin/ui-config/rollback": {
      auth: "OWNER"
      body: { version: number }
      response: { success: boolean }
    }
  }

  // ============================================
  // RESOLVED SETTINGS (for frontend)
  // ============================================
  resolvedSettings: {
    // Get all resolved settings for current user
    "GET /api/settings/resolved": {
      auth: "any authenticated user"
      response: {
        settings: Record<string, {
          value: any
          source: "user" | "team" | "organization" | "platform" | "system_default"
          editable: boolean
          locked: boolean
          lockedReason?: string
        }>
      }
    }

    // Get specific resolved setting
    "GET /api/settings/resolved/:key": {
      auth: "any authenticated user"
      response: {
        key: string
        value: any
        source: string
        editable: boolean
        locked: boolean
        history: { value: any, changedAt: Date, changedBy: string }[]
      }
    }
  }
}
```

---

## Database Schema

### Complete Settings Schema

```prisma
// ============================================
// SETTINGS STORAGE
// ============================================

model Setting {
  id    String @id @default(cuid())
  key   String                            // "theme.mode", "notifications.email.enabled"
  value Json                              // The setting value

  // Scope
  scope          String                   // "system", "platform", "organization", "team", "user"
  organizationId String?
  teamId         String?
  userId         String?

  // Metadata
  category    String?                     // "profile", "security", "notifications"
  description String?
  dataType    String                      // "string", "number", "boolean", "json", "array"

  // Lock
  isLocked     Boolean  @default(false)
  lockedBy     String?
  lockedAt     DateTime?
  lockedReason String?

  // Timestamps
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  version   Int       @default(1)

  @@unique([key, scope, organizationId, teamId, userId])
  @@index([scope])
  @@index([organizationId])
  @@index([userId])
  @@index([category])
  @@index([key])
}

// ============================================
// SETTINGS AUDIT LOG
// ============================================

model SettingsAuditLog {
  id             String @id @default(cuid())
  organizationId String?

  settingKey    String
  previousValue Json?
  newValue      Json?
  scope         String

  performedById String
  performedAt   DateTime @default(now())
  reason        String?
  ipAddress     String?

  @@index([organizationId])
  @@index([settingKey])
  @@index([performedById])
  @@index([performedAt])
}

// ============================================
// API KEYS
// ============================================

model APIKey {
  id             String @id @default(cuid())
  userId         String
  organizationId String

  name      String
  keyHash   String                        // Hashed key (never store plaintext)
  prefix    String                        // First 8 chars for identification
  scopes    String[]
  expiresAt DateTime?

  lastUsedAt DateTime?
  lastUsedIP String?
  usageCount Int       @default(0)

  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  revokedAt DateTime?
  revokedBy String?

  @@index([userId])
  @@index([organizationId])
  @@index([keyHash])
  @@index([prefix])
  @@index([isActive])
}

// ============================================
// CONNECTED ACCOUNTS
// ============================================

// (Uses existing Account model from NextAuth)

// ============================================
// LOGIN HISTORY
// ============================================

model LoginHistory {
  id     String @id @default(cuid())
  userId String

  method    String                        // "password", "google", "github", "2fa"
  success   Boolean
  ip        String
  userAgent String
  device    String?                       // Parsed device info
  location  Json?                         // { country, region, city }

  failureReason String?                   // "invalid_password", "account_locked"

  createdAt DateTime @default(now())

  @@index([userId])
  @@index([createdAt])
  @@index([success])
}

// ============================================
// CUSTOM DOMAIN
// ============================================

model CustomDomain {
  id             String @id @default(cuid())
  organizationId String @unique

  domain             String @unique
  verificationMethod String               // "dns_cname", "dns_txt"
  verificationRecord Json                 // DNS record details
  verified           Boolean @default(false)
  verifiedAt         DateTime?

  sslStatus    String  @default("pending") // "pending", "provisioning", "active", "failed"
  sslExpiresAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([domain])
  @@index([verified])
}

// ============================================
// NOTIFICATION PREFERENCES
// ============================================

model NotificationPreference {
  id     String @id @default(cuid())
  userId String @unique

  // Channel preferences
  emailEnabled   Boolean @default(true)
  inAppEnabled   Boolean @default(true)
  pushEnabled    Boolean @default(false)
  smsEnabled     Boolean @default(false)

  // Category preferences (JSON for flexibility)
  categoryPreferences Json @default("{}")

  // Digest settings
  digestMode      String  @default("realtime") // "realtime", "hourly", "daily", "weekly"
  digestTime      String? // "09:00" for daily digest

  // Quiet hours
  quietHoursEnabled Boolean @default(false)
  quietHoursStart   String? // "22:00"
  quietHoursEnd     String? // "08:00"
  quietHoursTimezone String?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
}

// ============================================
// FEATURE FLAGS
// ============================================

model FeatureFlag {
  id             String @id @default(cuid())
  key            String                   // "ai_generation", "custom_domain"
  name           String
  description    String?

  // Scope
  scope          String                   // "global", "organization", "user"
  organizationId String?
  userId         String?

  // State
  enabled     Boolean @default(false)
  percentage  Int?                        // Gradual rollout percentage
  conditions  Json?                       // Targeting conditions

  // Plan restriction
  requiredPlan String?                    // Minimum plan required

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([key, scope, organizationId, userId])
  @@index([key])
  @@index([scope])
  @@index([organizationId])
}
```

---

## Edge Cases & Conflict Resolution

### Edge Case Handling

```typescript
interface SettingsEdgeCases {
  // ============================================
  // EDGE CASE 1: User Overrides Org Setting That Gets Locked
  // ============================================
  userOverrideLocked: {
    scenario: "User set theme to 'light', then admin locks theme to 'dark'"
    handling: "Admin lock takes precedence immediately"
    steps: [
      "1. User's preference is preserved in database (not deleted)",
      "2. Resolved value returns admin's locked value",
      "3. UI shows lock icon with explanation",
      "4. If admin unlocks later, user's preference is restored"
    ]
  }

  // ============================================
  // EDGE CASE 2: Plan Downgrade Removes Settings Access
  // ============================================
  planDowngrade: {
    scenario: "Org downgrades from PRO to FREE, losing custom branding"
    handling: "Graceful degradation"
    steps: [
      "1. Settings values are preserved (not deleted)",
      "2. Settings become read-only with upgrade prompt",
      "3. Features using those settings revert to defaults",
      "4. If org upgrades again, settings are restored",
      "5. Notify admin about affected settings"
    ]
  }

  // ============================================
  // EDGE CASE 3: Concurrent Settings Updates
  // ============================================
  concurrentUpdate: {
    scenario: "Two admins update the same setting simultaneously"
    handling: "Last write wins with conflict notification"
    steps: [
      "1. Each setting has a version number",
      "2. Update includes expected version",
      "3. If version mismatch: return 409 Conflict",
      "4. Client refreshes and shows diff",
      "5. Both changes are in audit log"
    ]
  }

  // ============================================
  // EDGE CASE 4: Setting Validation Fails After Schema Change
  // ============================================
  schemaChange: {
    scenario: "New validation rule added; existing values don't comply"
    handling: "Grandfather existing values"
    steps: [
      "1. New validation only applies to new writes",
      "2. Existing non-compliant values are flagged",
      "3. Admin notified of non-compliant settings",
      "4. Grace period to update values",
      "5. After grace period, force default values"
    ]
  }

  // ============================================
  // EDGE CASE 5: Custom Domain SSL Certificate Expires
  // ============================================
  sslExpiry: {
    scenario: "Auto-provisioned SSL certificate is about to expire"
    handling: "Auto-renewal with fallback"
    steps: [
      "1. Background job checks SSL expiry daily",
      "2. Auto-renew 30 days before expiry",
      "3. If renewal fails, notify admin",
      "4. If renewal fails 7 days before expiry, send urgent alert",
      "5. If expired, fall back to default domain"
    ]
  }

  // ============================================
  // EDGE CASE 6: User Deletes Account During Grace Period
  // ============================================
  deletionGracePeriod: {
    scenario: "User requests deletion, then logs in during 30-day grace period"
    handling: "Cancel deletion on login"
    steps: [
      "1. User logs in during grace period",
      "2. Show banner: 'Your account is scheduled for deletion on [date]'",
      "3. Offer to cancel deletion",
      "4. If user cancels, account is fully restored",
      "5. If user doesn't cancel, deletion proceeds"
    ]
  }

  // ============================================
  // EDGE CASE 7: Email Provider Fails
  // ============================================
  emailProviderFailure: {
    scenario: "Primary email provider (SendGrid) is down"
    handling: "Automatic failover"
    steps: [
      "1. Detect failure (timeout, 5xx response)",
      "2. Switch to fallback provider (SMTP)",
      "3. Queue failed emails for retry",
      "4. Notify admin of provider failure",
      "5. Auto-switch back when primary recovers",
      "6. Process queued emails"
    ]
  }

  // ============================================
  // EDGE CASE 8: API Key Used After User Deactivation
  // ============================================
  deactivatedUserAPIKey: {
    scenario: "User is deactivated but their API key is still being used by CI/CD"
    handling: "Immediate key invalidation"
    steps: [
      "1. When user is deactivated, all their API keys are revoked",
      "2. API requests with revoked keys return 401",
      "3. Audit log captures the revocation",
      "4. Admin can transfer API keys to another user before deactivation"
    ]
  }

  // ============================================
  // EDGE CASE 9: Circular Setting Dependencies
  // ============================================
  circularDependency: {
    scenario: "Setting A depends on Setting B, which depends on Setting A"
    handling: "Dependency graph validation"
    steps: [
      "1. Settings can declare dependencies",
      "2. Before saving, validate no circular dependencies",
      "3. If circular detected, reject with clear error",
      "4. Show dependency graph in admin UI"
    ]
  }

  // ============================================
  // EDGE CASE 10: Bulk Settings Import Conflicts
  // ============================================
  importConflicts: {
    scenario: "Importing settings from another org with conflicting values"
    handling: "Conflict resolution wizard"
    options: [
      "keep_existing: Keep current values for conflicts",
      "use_imported: Use imported values for conflicts",
      "manual: Show diff and let admin choose per-setting",
      "merge: Deep merge objects, imported wins for primitives"
    ]
  }

  // ============================================
  // EDGE CASE 11: Setting Value Exceeds Storage Limit
  // ============================================
  storageLimitExceeded: {
    scenario: "Custom CSS setting exceeds maximum allowed size"
    handling: "Validation with clear limits"
    response: {
      error: "Custom CSS exceeds maximum size of 50KB",
      currentSize: "75KB",
      maxSize: "50KB",
      suggestion: "Consider using a CDN for large stylesheets"
    }
  }

  // ============================================
  // EDGE CASE 12: Timezone Change Affects Scheduled Jobs
  // ============================================
  timezoneChange: {
    scenario: "Admin changes org timezone; scheduled jobs use old timezone"
    handling: "Recalculate all scheduled times"
    steps: [
      "1. Detect timezone change",
      "2. Recalculate all scheduled job times",
      "3. Recalculate notification digest times",
      "4. Recalculate quiet hours for all users",
      "5. Notify affected users of schedule changes"
    ]
  }
}
```

---

## Performance & Caching

### Settings Caching Strategy

```typescript
interface SettingsCaching {
  // ============================================
  // CACHING LAYERS
  // ============================================
  layers: {
    // Layer 1: Request-level cache
    requestCache: {
      scope: "single request"
      purpose: "Avoid multiple DB queries for same setting in one request"
      implementation: "Map in request context"
    }

    // Layer 2: Resolved settings cache (Redis)
    resolvedCache: {
      scope: "user's complete resolved settings"
      ttl: 300                            // 5 minutes
      key: "settings:resolved:{orgId}:{userId}"
      purpose: "Serve complete settings to frontend without DB queries"
      invalidation: "On any setting change affecting user"
    }

    // Layer 3: Organization settings cache
    orgCache: {
      scope: "organization settings"
      ttl: 600                            // 10 minutes
      key: "settings:org:{orgId}"
      purpose: "Cache org-level settings shared by all members"
      invalidation: "On org setting change"
    }

    // Layer 4: System/platform settings cache
    systemCache: {
      scope: "platform-wide settings"
      ttl: 3600                           // 1 hour
      key: "settings:platform"
      purpose: "Cache rarely-changing platform settings"
      invalidation: "On platform setting change"
    }
  }

  // ============================================
  // CACHE INVALIDATION
  // ============================================
  invalidation: {
    triggers: {
      userSettingChanged: ["user_resolved_cache"]
      teamSettingChanged: ["all_team_members_resolved_cache"]
      orgSettingChanged: ["org_cache", "all_org_members_resolved_cache"]
      platformSettingChanged: ["system_cache", "all_resolved_caches"]
      settingLocked: ["all_org_members_resolved_cache"]
      settingUnlocked: ["all_org_members_resolved_cache"]
      planChanged: ["org_cache", "all_org_members_resolved_cache"]
    }
    strategy: "targeted"
    broadcast: "redis_pubsub"
  }

  // ============================================
  // FRONTEND SETTINGS LOADING
  // ============================================
  frontendLoading: {
    // Load all resolved settings on login
    onLogin: {
      endpoint: "GET /api/settings/resolved"
      cacheInMemory: true
      cacheInLocalStorage: false          // Security: don't cache sensitive settings
      refreshInterval: 300                // Seconds (poll for changes)
      useWebSocket: true                  // Real-time updates via WebSocket
    }

    // Selective loading for specific pages
    onPageLoad: {
      // Only load settings relevant to current page
      endpoint: "GET /api/settings/resolved?category=profile"
      lazyLoad: true
    }
  }
}
```

---

## Settings Audit & Change Tracking

### Audit Configuration

```typescript
interface SettingsAudit {
  // ============================================
  // WHAT TO AUDIT
  // ============================================
  auditEvents: {
    settingCreated: boolean
    settingUpdated: boolean
    settingDeleted: boolean
    settingLocked: boolean
    settingUnlocked: boolean
    settingReset: boolean
    settingsImported: boolean
    settingsExported: boolean
    passwordChanged: boolean
    twoFactorEnabled: boolean
    twoFactorDisabled: boolean
    apiKeyCreated: boolean
    apiKeyRevoked: boolean
    sessionRevoked: boolean
    accountDeletionRequested: boolean
    accountDeletionCancelled: boolean
    accountDeactivated: boolean
    accountReactivated: boolean
    organizationUpdated: boolean
    brandingUpdated: boolean
    customDomainAdded: boolean
    customDomainRemoved: boolean
    featureToggled: boolean
    planChanged: boolean
  }

  // ============================================
  // AUDIT LOG ENTRY
  // ============================================
  logEntry: {
    id: string
    timestamp: Date
    organizationId: string
    performedById: string
    performedByName: string
    performedByRole: string
    action: string                        // "setting_updated"
    category: string                      // "security"
    settingKey: string                    // "password.minLength"
    previousValue: any
    newValue: any
    scope: string                         // "organization"
    ipAddress: string
    userAgent: string
    reason: string                        // Optional reason for change
  }

  // ============================================
  // AUDIT QUERIES
  // ============================================
  queries: {
    // "What changed in the last 24 hours?"
    recentChanges: {
      endpoint: "GET /api/admin/settings/audit?last=24h"
    }
    // "Who changed the password policy?"
    specificSetting: {
      endpoint: "GET /api/admin/settings/audit?key=password.minLength"
    }
    // "What did admin X change?"
    byPerformer: {
      endpoint: "GET /api/admin/settings/audit?performedBy=user-id"
    }
    // "Show all security setting changes"
    byCategory: {
      endpoint: "GET /api/admin/settings/audit?category=security"
    }
  }
}
```

---

## Integration Examples

### Complete Integration: Settings-Aware React Component

```tsx
// ============================================
// EXAMPLE: Settings-Aware Dashboard
// ============================================

'use client'

import { useSettings, useSetting, useSettingEditable } from '@/lib/settings/client'
import { useRole } from '@/lib/rbac/client'

export default function DashboardPage() {
  // Load resolved settings
  const { settings, isLoading } = useSettings()

  // Individual setting hooks
  const theme = useSetting('theme.mode', 'system')
  const dashboardView = useSetting('dashboard.defaultView', 'grid')
  const sidebarCollapsed = useSetting('dashboard.sidebarCollapsed', false)
  const showWelcome = useSetting('dashboard.showWelcomeMessage', true)

  // Check if user can edit settings
  const canEditDashboard = useSettingEditable('dashboard.defaultView')

  const { isOwner, isAdmin } = useRole()

  if (isLoading) return <DashboardSkeleton />

  return (
    <div className={cn(
      "dashboard",
      theme === 'dark' && 'dark',
      sidebarCollapsed && 'sidebar-collapsed'
    )}>
      {/* Welcome message (can be disabled by user) */}
      {showWelcome && (
        <WelcomeBanner
          onDismiss={() => updateSetting('dashboard.showWelcomeMessage', false)}
        />
      )}

      {/* View switcher (only if user can edit) */}
      {canEditDashboard && (
        <ViewSwitcher
          current={dashboardView}
          onChange={(view) => updateSetting('dashboard.defaultView', view)}
          options={['grid', 'list', 'kanban']}
        />
      )}

      {/* Dashboard content based on view setting */}
      {dashboardView === 'grid' && <GridView />}
      {dashboardView === 'list' && <ListView />}
      {dashboardView === 'kanban' && <KanbanView />}

      {/* Admin-only settings shortcut */}
      {(isOwner || isAdmin) && (
        <Card className="mt-4">
          <CardHeader>
            <CardTitle>Quick Settings</CardTitle>
          </CardHeader>
          <CardContent>
            <SettingToggle
              settingKey="features.aiGeneration"
              label="AI Generation"
              description="Enable AI-powered code generation"
            />
            <SettingToggle
              settingKey="features.collaboration"
              label="Real-time Collaboration"
              description="Enable real-time editing and comments"
            />
          </CardContent>
        </Card>
      )}
    </div>
  )
}

// ============================================
// REUSABLE: Setting Toggle Component
// ============================================
function SettingToggle({
  settingKey,
  label,
  description,
}: {
  settingKey: string
  label: string
  description: string
}) {
  const value = useSetting(settingKey, false)
  const editable = useSettingEditable(settingKey)
  const { source, locked, lockedReason } = useSettingMetadata(settingKey)

  return (
    <div className="flex items-center justify-between py-3">
      <div>
        <p className="font-medium">{label}</p>
        <p className="text-sm text-muted-foreground">{description}</p>
        {locked && (
          <p className="text-xs text-amber-500 flex items-center gap-1 mt-1">
            <LockIcon className="h-3 w-3" />
            {lockedReason || "Locked by admin"}
          </p>
        )}
        {source !== 'user' && !locked && (
          <p className="text-xs text-muted-foreground mt-1">
            Inherited from {source}
          </p>
        )}
      </div>
      <Switch
        checked={value}
        onCheckedChange={(checked) => updateSetting(settingKey, checked)}
        disabled={!editable || locked}
      />
    </div>
  )
}
```

### Complete Integration: Settings API Route

```typescript
// ============================================
// EXAMPLE: Settings API with Full Validation
// ============================================
// File: app/api/admin/settings/[category]/route.ts

import { NextRequest, NextResponse } from 'next/server'
import { auth } from '@/lib/auth'
import { prisma } from '@/lib/db'
import { checkPermission } from '@/lib/rbac'
import { validateSettings, getSettingsSchema } from '@/lib/settings/validation'
import { invalidateSettingsCache } from '@/lib/settings/cache'
import { logSettingsChange } from '@/lib/settings/audit'

export async function PATCH(
  request: NextRequest,
  { params }: { params: Promise<{ category: string }> }
) {
  // Step 1: Authenticate
  const session = await auth()
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const user = session.user as any
  const { category } = await params

  // Step 2: Check permission
  const permission = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType: 'SETTINGS',
    action: 'CONFIGURE',
  })

  if (!permission.allowed) {
    return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
  }

  // Step 3: Parse and validate body
  const body = await request.json()
  const schema = getSettingsSchema(category)
  const validation = validateSettings(body, schema)

  if (!validation.valid) {
    return NextResponse.json({
      error: 'Validation failed',
      errors: validation.errors
    }, { status: 400 })
  }

  // Step 4: Check plan restrictions
  const org = await prisma.organization.findUnique({
    where: { id: user.organizationId },
    select: { plan: true }
  })

  const planRestrictions = getPlanRestrictions(org!.plan, category)
  if (planRestrictions.restricted) {
    return NextResponse.json({
      error: `This setting requires ${planRestrictions.requiredPlan} plan`,
      currentPlan: org!.plan,
      requiredPlan: planRestrictions.requiredPlan,
    }, { status: 403 })
  }

  // Step 5: Save settings
  const previousValues: Record<string, any> = {}

  await prisma.$transaction(async (tx) => {
    for (const [key, value] of Object.entries(body)) {
      const settingKey = `${category}.${key}`

      // Get previous value for audit
      const existing = await tx.setting.findFirst({
        where: {
          key: settingKey,
          scope: 'organization',
          organizationId: user.organizationId,
        }
      })
      previousValues[key] = existing?.value

      // Upsert setting
      await tx.setting.upsert({
        where: {
          key_scope_organizationId_teamId_userId: {
            key: settingKey,
            scope: 'organization',
            organizationId: user.organizationId,
            teamId: null,
            userId: null,
          }
        },
        create: {
          key: settingKey,
          value: value as any,
          scope: 'organization',
          organizationId: user.organizationId,
          category,
          dataType: typeof value,
          createdAt: new Date(),
        },
        update: {
          value: value as any,
          updatedAt: new Date(),
          version: { increment: 1 },
        }
      })
    }
  })

  // Step 6: Invalidate cache
  await invalidateSettingsCache(user.organizationId, category)

  // Step 7: Audit log
  for (const [key, value] of Object.entries(body)) {
    await logSettingsChange({
      organizationId: user.organizationId,
      performedById: user.id,
      settingKey: `${category}.${key}`,
      previousValue: previousValues[key],
      newValue: value,
      scope: 'organization',
      ipAddress: request.headers.get('x-forwarded-for') || 'unknown',
    })
  }

  // Step 8: Return updated settings
  const updatedSettings = await getOrgSettings(user.organizationId, category)
  return NextResponse.json({ settings: updatedSettings })
}
```

---

## Summary

This comprehensive Settings Management system provides:

1. **Layered Settings Hierarchy** — System → Platform → Organization → Team → User with most-specific-wins resolution
2. **User Profile Settings** — Personal info, avatar, email change, preferences (theme, language, timezone, accessibility)
3. **Account Security Settings** — Password management, 2FA (TOTP/SMS/WebAuthn), session management, API keys, connected accounts, account deletion/deactivation
4. **Notification Settings** — Multi-channel (email, in-app, push, SMS, Slack, webhook), per-category preferences, quiet hours, digest mode
5. **Organization Settings** — General info, branding, custom domain, member management, feature toggles, resource limits, data retention
6. **Billing Settings** — Subscription management, payment methods, invoices, usage tracking, tax configuration
7. **AI & MCP Settings** — Provider configuration (OpenAI, Anthropic, Google), MCP servers, usage controls, feature toggles
8. **Email System Settings** — Provider config, templates, queue management, tracking, suppression lists
9. **Integration Settings** — Webhooks, API access, third-party integrations (Slack, GitHub, Jira), SSO/SAML/OIDC/LDAP
10. **System Configuration** — Application settings, performance, security, database, storage, monitoring, background jobs
11. **UI Configuration** — Theme, layout, navigation, component-level config, per-page config, live preview
12. **Privacy & Data Settings** — GDPR/CCPA compliance, cookie consent, data portability, right to erasure, encryption, anonymization
13. **Localization** — Multi-language, regional settings, content localization
14. **Setting Locks** — Admin can lock settings to prevent user override
15. **Plan-Based Restrictions** — Settings availability tied to subscription plan
16. **Full Audit Trail** — Every setting change logged with who, what, when, previous/new value
17. **Settings Import/Export** — Portable configuration with conflict resolution
18. **Real-Time Propagation** — Changes take effect immediately via cache invalidation + WebSocket
19. **16 Edge Cases** — Covering concurrent updates, plan downgrades, SSL expiry, timezone changes, circular dependencies, and more
20. **Complete API Architecture** — 50+ API routes covering every settings operation

Every aspect is **admin-configurable without code changes**, **RBAC-gated**, **plan-aware**, and **fully auditable**.
