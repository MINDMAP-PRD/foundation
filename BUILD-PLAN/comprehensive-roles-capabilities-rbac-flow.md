# Comprehensive Roles, Capabilities & RBAC System Flow

> **Foundation wiring note:** Treat this file as complementary shared RBAC/capability guidance for `GeneralApp`. Merge its depth into the canonical foundation registries, APIs, inspection tools, and owner-control surfaces instead of rebuilding parallel policy systems.

## Table of Contents
1. [System Overview](#system-overview)
2. [Admin User Management Interface](#admin-user-management-interface)
3. [Individual User Inspection](#individual-user-inspection)
4. [Admin Communication System](#admin-communication-system)
5. [Custom RBAC System](#custom-rbac-system)
6. [Custom Roles & Capabilities](#custom-roles--capabilities)
7. [Dynamic Permission System](#dynamic-permission-system)
8. [Database Schema](#database-schema)
9. [API Architecture](#api-architecture)
10. [Complete Implementation Examples](#complete-implementation-examples)

---

## System Overview

### Core Principles
1. **Complete Admin Control**: Inspect, manage, and communicate with any user
2. **Granular RBAC**: Role-based + Capability-based + Attribute-based access control
3. **Custom Roles**: Create unlimited custom roles with specific capabilities
4. **Custom Capabilities**: Define custom permissions beyond standard CRUD
5. **Communication Hub**: Email, SMS, in-app, push notifications - all or individual users
6. **Template System**: Create, edit, preview, and manage message templates
7. **User Intelligence**: Complete user journey tracking and analytics
8. **Compliance-First**: All actions logged, GDPR-compliant, audit trails

### Admin Capabilities Architecture
```
User Discovery → User Inspection → User Actions → Communication →
Role Management → Capability Management → Audit & Reporting
```

---

## Admin User Management Interface

### User List Dashboard
```typescript
interface AdminUserDashboard {
  // ============================================
  // USER LIST VIEW
  // ============================================
  userList: {
    // Display options
    view: "table" | "grid" | "compact"
    columns: {
      columnId: string
      name: string
      visible: boolean
      sortable: boolean
      filterable: boolean
      width: number
      customRenderer?: string
    }[]

    // Default visible columns
    defaultColumns: [
      "avatar",
      "name",
      "email",
      "status",
      "role",
      "created_at",
      "last_login_at",
      "actions"
    ]

    // Available columns
    availableColumns: {
      // Identity
      id: "User ID"
      avatar: "Avatar"
      email: "Email"
      phone: "Phone"
      username: "Username"
      display_name: "Display Name"
      first_name: "First Name"
      last_name: "Last Name"

      // Account Info
      status: "Status"
      account_state: "Account State"
      email_verified: "Email Verified"
      phone_verified: "Phone Verified"
      identity_verified: "Identity Verified"

      // Roles & Permissions
      roles: "Roles"
      permissions: "Permissions"
      organizations: "Organizations"
      teams: "Teams"

      // Authentication
      auth_method: "Auth Method"
      has_password: "Has Password"
      mfa_enabled: "MFA Enabled"
      social_linked: "Social Linked"

      // Activity
      created_at: "Joined Date"
      last_login_at: "Last Login"
      login_count: "Login Count"
      last_active_at: "Last Active"
      inactive_days: "Days Inactive"

      // Device & Location
      last_device: "Last Device"
      last_location: "Last Location"
      last_ip: "Last IP"

      // Engagement
      engagement_score: "Engagement Score"
      total_actions: "Total Actions"
      subscription_tier: "Subscription"

      // Custom Fields
      custom_fields: Record<string, string>
    }

    // Filters
    filters: {
      // Quick filters
      quickFilters: {
        all: "All Users"
        active: "Active Users"
        inactive: "Inactive Users"
        verified: "Verified Users"
        unverified: "Unverified Users"
        suspended: "Suspended Users"
        admins: "Administrators"
        new_today: "Joined Today"
        new_week: "Joined This Week"
        logged_in_today: "Logged In Today"
        mfa_enabled: "MFA Enabled"
        social_only: "Social Auth Only"
      }

      // Advanced filters
      advancedFilters: {
        // Status filters
        status: {
          type: "multiselect"
          options: ["pending", "active", "inactive", "suspended", "banned", "deleted"]
        }

        // Role filters
        roles: {
          type: "multiselect"
          options: Role[]  // Dynamic from database
        }

        // Auth method filters
        authMethods: {
          type: "multiselect"
          options: ["email_password", "google", "github", "apple", "microsoft", "saml", "passwordless"]
        }

        // Verification filters
        verification: {
          email_verified: boolean
          phone_verified: boolean
          identity_verified: boolean
          verification_level: "none" | "basic" | "standard" | "enhanced"
        }

        // Date filters
        dateFilters: {
          joined_after: Date
          joined_before: Date
          last_login_after: Date
          last_login_before: Date
          inactive_since: Date
        }

        // Engagement filters
        engagement: {
          min_logins: number
          max_logins: number
          min_engagement_score: number
          max_engagement_score: number
        }

        // Location filters
        location: {
          countries: string[]
          cities: string[]
          timezone: string[]
        }

        // Subscription filters
        subscription: {
          tiers: string[]
          status: "active" | "cancelled" | "expired"
        }

        // Custom field filters
        customFields: Record<string, any>

        // Tag filters
        tags: string[]

        // Segment filters
        segments: string[]
      }

      // Saved filters
      savedFilters: {
        filterId: string
        name: string
        filters: Record<string, any>
        createdBy: string
        shared: boolean
      }[]
    }

    // Sorting
    sorting: {
      field: string
      direction: "asc" | "desc"
      secondarySort?: {
        field: string
        direction: "asc" | "desc"
      }
    }

    // Pagination
    pagination: {
      page: number
      perPage: number
      total: number
      perPageOptions: [10, 25, 50, 100, 250, 500]
    }

    // Bulk actions
    bulkActions: {
      select: {
        all: boolean
        ids: string[]
        excludeIds: string[]
        selectAcrossPages: boolean
      }

      actions: {
        // User actions
        activateUsers: () => void
        suspendUsers: (reason: string) => void
        deleteUsers: (permanent: boolean) => void
        exportUsers: (format: "csv" | "json" | "xlsx") => void

        // Role actions
        assignRole: (roleId: string) => void
        removeRole: (roleId: string) => void

        // Permission actions
        grantPermission: (permissionId: string) => void
        revokePermission: (permissionId: string) => void

        // Communication actions
        sendEmail: (templateId?: string) => void
        sendSMS: (message: string) => void
        sendPushNotification: (title: string, body: string) => void
        sendInAppMessage: (message: string) => void

        // Tag actions
        addTag: (tag: string) => void
        removeTag: (tag: string) => void

        // Segment actions
        addToSegment: (segmentId: string) => void
        removeFromSegment: (segmentId: string) => void

        // Organization actions
        addToOrganization: (orgId: string) => void
        removeFromOrganization: (orgId: string) => void

        // Verification actions
        verifyEmail: () => void
        verifyPhone: () => void
        requestIdentityVerification: () => void

        // Security actions
        forceMFA: () => void
        resetPassword: () => void
        terminateSessions: () => void
      }
    }

    // Search
    search: {
      query: string
      fields: ["email", "name", "username", "phone", "id", "custom_fields"]
      fuzzy: boolean
      caseSensitive: boolean
    }

    // Export
    export: {
      format: "csv" | "json" | "xlsx" | "pdf"
      fields: string[]
      includeFilters: boolean
      schedule: {
        enabled: boolean
        frequency: "daily" | "weekly" | "monthly"
        recipients: string[]
      }
    }
  }

  // ============================================
  // ANALYTICS & INSIGHTS
  // ============================================
  analytics: {
    // Overview metrics
    overview: {
      totalUsers: number
      activeUsers: number
      newUsersToday: number
      newUsersWeek: number
      newUsersMonth: number
      verifiedUsers: number
      suspendedUsers: number
      mfaEnabledUsers: number
      averageEngagementScore: number
    }

    // Growth metrics
    growth: {
      dailySignups: TimeSeriesData
      weeklySignups: TimeSeriesData
      monthlySignups: TimeSeriesData
      retentionRate: number
      churnRate: number
    }

    // Authentication metrics
    authentication: {
      authMethodDistribution: {
        email_password: number
        google: number
        github: number
        apple: number
        saml: number
        passwordless: number
      }
      mfaAdoptionRate: number
      socialLinkingRate: number
    }

    // Engagement metrics
    engagement: {
      dailyActiveUsers: number
      weeklyActiveUsers: number
      monthlyActiveUsers: number
      averageSessionDuration: number
      averageLoginsPerUser: number
    }

    // Geographic distribution
    geography: {
      usersByCountry: Record<string, number>
      usersByCity: Record<string, number>
      usersByTimezone: Record<string, number>
    }

    // Device & Platform
    devices: {
      deviceTypes: Record<string, number>
      browsers: Record<string, number>
      operatingSystems: Record<string, number>
    }

    // Role distribution
    roles: {
      roleDistribution: Record<string, number>
      permissionUsage: Record<string, number>
    }

    // Verification metrics
    verification: {
      emailVerificationRate: number
      phoneVerificationRate: number
      identityVerificationRate: number
      averageVerificationTime: number
    }

    // Custom metrics
    customMetrics: Record<string, any>
  }

  // ============================================
  // QUICK ACTIONS
  // ============================================
  quickActions: {
    createUser: () => void
    bulkImport: () => void
    exportAll: () => void
    sendAnnouncement: () => void
    createSegment: () => void
    viewAuditLog: () => void
    manageRoles: () => void
    manageCapabilities: () => void
  }
}
```

---

## Individual User Inspection

### Complete User Profile View
```typescript
interface AdminUserInspection {
  // ============================================
  // USER OVERVIEW
  // ============================================
  overview: {
    // Basic Info
    basicInfo: {
      id: string
      avatar: string
      displayName: string
      firstName: string
      lastName: string
      email: string
      emailVerified: boolean
      phone: string
      phoneVerified: boolean
      username: string
      dateOfBirth: Date
      age: number
      gender: string
      pronouns: string

      // Status badges
      status: "active" | "inactive" | "suspended" | "banned"
      accountState: "pending_verification" | "active" | "inactive"
      verificationLevel: "none" | "basic" | "standard" | "enhanced"

      // Quick stats
      memberSince: Date
      daysSinceJoined: number
      lastLogin: Date
      daysSinceLastLogin: number
      loginCount: number
      engagementScore: number

      // Risk indicators
      riskScore: number
      riskLevel: "low" | "medium" | "high"
      flagged: boolean
      flagReason: string
    }

    // Profile Completion
    profileCompletion: {
      percentage: number
      missingFields: string[]
      completedFields: string[]
      requiredFields: string[]
    }

    // Account Health
    accountHealth: {
      score: number  // 0-100
      factors: {
        emailVerified: boolean
        phoneVerified: boolean
        identityVerified: boolean
        mfaEnabled: boolean
        passwordStrong: boolean
        recentActivity: boolean
        noViolations: boolean
      }
      recommendations: string[]
    }
  }

  // ============================================
  // AUTHENTICATION DETAILS
  // ============================================
  authentication: {
    // Auth Methods
    authMethods: {
      primary: "email_password" | "oauth" | "saml" | "passwordless"

      // Password
      password: {
        hasPassword: boolean
        passwordSetAt: Date
        passwordChangedAt: Date
        passwordExpiresAt: Date
        passwordStrength: "weak" | "fair" | "good" | "strong" | "very_strong"
        passwordAge: number  // Days
        requiresReset: boolean
      }

      // OAuth/Social
      oauth: {
        providers: {
          google: {
            connected: boolean
            connectedAt: Date
            email: string
            profileId: string
            lastUsed: Date
          }
          github: {
            connected: boolean
            connectedAt: Date
            username: string
            profileId: string
            lastUsed: Date
          }
          apple: {
            connected: boolean
            connectedAt: Date
            email: string
            profileId: string
            lastUsed: Date
          }
          microsoft: {
            connected: boolean
            connectedAt: Date
            email: string
            profileId: string
            lastUsed: Date
          }
          facebook: {
            connected: boolean
            connectedAt: Date
            email: string
            profileId: string
            lastUsed: Date
          }
          linkedin: {
            connected: boolean
            connectedAt: Date
            email: string
            profileId: string
            lastUsed: Date
          }
          twitter: {
            connected: boolean
            connectedAt: Date
            username: string
            profileId: string
            lastUsed: Date
          }
        }

        primaryProvider: string
        linkedAccounts: number
        autoLinkEnabled: boolean
      }

      // SSO
      sso: {
        enabled: boolean
        provider: "saml" | "oidc" | "ldap"
        entityId: string
        lastUsed: Date
        domain: string
      }

      // Passwordless
      passwordless: {
        enabled: boolean
        methods: ("magic_link" | "otp" | "webauthn")[]
        lastUsed: Date
      }
    }

    // MFA Status
    mfa: {
      enabled: boolean
      required: boolean
      methods: {
        totp: {
          enabled: boolean
          setupAt: Date
          lastUsed: Date
          backupCodes: number  // Remaining codes
        }
        sms: {
          enabled: boolean
          phoneNumber: string
          lastUsed: Date
        }
        email: {
          enabled: boolean
          email: string
          lastUsed: Date
        }
        webauthn: {
          enabled: boolean
          devices: {
            id: string
            name: string
            type: "platform" | "cross_platform"
            addedAt: Date
            lastUsed: Date
          }[]
        }
        push: {
          enabled: boolean
          lastUsed: Date
        }
      }

      trustedDevices: {
        id: string
        name: string
        addedAt: Date
        expiresAt: Date
      }[]
    }

    // Recovery Options
    recovery: {
      email: {
        enabled: boolean
        email: string
        verified: boolean
      }
      phone: {
        enabled: boolean
        phone: string
        verified: boolean
      }
      securityQuestions: {
        enabled: boolean
        questionsSet: number
        lastUpdated: Date
      }
      backupCodes: {
        generated: boolean
        generatedAt: Date
        used: number
        remaining: number
      }
    }
  }

  // ============================================
  // SESSIONS & DEVICES
  // ============================================
  sessions: {
    // Active Sessions
    activeSessions: {
      sessionId: string
      device: {
        type: "desktop" | "mobile" | "tablet"
        os: string
        browser: string
        deviceName: string
        fingerprint: string
      }
      location: {
        ip: string
        country: string
        city: string
        region: string
        timezone: string
        coordinates: [number, number]
      }
      startedAt: Date
      lastActivityAt: Date
      expiresAt: Date
      isCurrent: boolean
      isTrusted: boolean

      // Session actions
      actions: {
        viewDetails: () => void
        terminate: () => void
        trustDevice: () => void
        untrustDevice: () => void
      }
    }[]

    // Session Statistics
    statistics: {
      totalSessions: number
      activeSessions: number
      uniqueDevices: number
      uniqueLocations: number
      averageSessionDuration: number
      totalSessionTime: number
    }

    // Bulk Actions
    bulkActions: {
      terminateAll: () => void
      terminateOthers: () => void  // Except current
      clearTrustedDevices: () => void
    }
  }

  // ============================================
  // LOGIN HISTORY
  // ============================================
  loginHistory: {
    // Recent Logins
    recentLogins: {
      id: string
      success: boolean
      timestamp: Date
      method: string  // "password", "google", "magic_link"
      device: {
        type: string
        os: string
        browser: string
      }
      location: {
        ip: string
        country: string
        city: string
      }
      failureReason?: string
      mfaUsed: boolean
      newDevice: boolean
      newLocation: boolean
      suspicious: boolean
      riskScore: number
    }[]

    // Login Statistics
    statistics: {
      totalLogins: number
      successfulLogins: number
      failedLogins: number
      failureRate: number
      averageLoginsPerDay: number
      mostUsedMethod: string
      mostUsedDevice: string
      mostCommonLocation: string
      suspiciousLoginCount: number
    }

    // Login Patterns
    patterns: {
      peakLoginHours: number[]
      peakLoginDays: number[]
      usualLocations: string[]
      usualDevices: string[]
    }

    // Filters
    filters: {
      dateRange: [Date, Date]
      success: boolean
      method: string[]
      suspicious: boolean
    }
  }

  // ============================================
  // ROLES & PERMISSIONS
  // ============================================
  rolesAndPermissions: {
    // Assigned Roles
    assignedRoles: {
      roleId: string
      name: string
      slug: string
      description: string
      level: number
      assignedAt: Date
      assignedBy: string  // Admin user
      expiresAt: Date
      source: "manual" | "auto_assigned" | "inherited"

      // Role details
      permissions: {
        permissionId: string
        name: string
        resource: string
        actions: string[]
        scope: string
      }[]

      // Role actions
      actions: {
        viewDetails: () => void
        remove: () => void
        extend: () => void
        viewAuditLog: () => void
      }
    }[]

    // Direct Permissions
    directPermissions: {
      permissionId: string
      name: string
      resource: string
      actions: string[]
      scope: string
      grantedAt: Date
      grantedBy: string
      expiresAt: Date

      actions: {
        revoke: () => void
        extend: () => void
      }
    }[]

    // Inherited Permissions
    inheritedPermissions: {
      permissionId: string
      name: string
      source: string  // "role:admin", "organization:acme"
      resource: string
      actions: string[]
      scope: string
    }[]

    // Permission Summary
    summary: {
      totalPermissions: number
      directPermissions: number
      rolePermissions: number
      inheritedPermissions: number

      // By resource
      byResource: Record<string, number>

      // By scope
      byScope: {
        own: number
        team: number
        organization: number
        global: number
      }
    }

    // Role Actions
    actions: {
      assignRole: (roleId: string) => void
      removeRole: (roleId: string) => void
      grantPermission: (permissionId: string) => void
      revokePermission: (permissionId: string) => void
      viewEffectivePermissions: () => void
      compareWithRole: (roleId: string) => void
    }
  }

  // ============================================
  // ORGANIZATIONS & TEAMS
  // ============================================
  organizationsAndTeams: {
    // Organizations
    organizations: {
      organizationId: string
      name: string
      slug: string
      role: "owner" | "admin" | "member" | "guest"
      joinedAt: Date
      invitedBy: string
      status: "active" | "pending" | "suspended"

      // Organization details
      details: {
        memberCount: number
        teamCount: number
        subscription: string
      }

      actions: {
        viewOrganization: () => void
        changeRole: (role: string) => void
        removeFromOrganization: () => void
        transferOwnership: (userId: string) => void
      }
    }[]

    // Teams
    teams: {
      teamId: string
      name: string
      organizationName: string
      role: "lead" | "member"
      joinedAt: Date
      invitedBy: string

      // Team details
      details: {
        memberCount: number
        projectCount: number
      }

      actions: {
        viewTeam: () => void
        changeRole: (role: string) => void
        removeFromTeam: () => void
      }
    }[]

    // Summary
    summary: {
      totalOrganizations: number
      ownedOrganizations: number
      totalTeams: number
      ledTeams: number
    }
  }

  // ============================================
  // ACTIVITY & ENGAGEMENT
  // ============================================
  activityAndEngagement: {
    // Activity Timeline
    timeline: {
      timestamp: Date
      type: string  // "login", "profile_update", "post_created", etc.
      action: string
      resource: string
      resourceId: string
      details: Record<string, any>
      ip: string
      device: string
      location: string
    }[]

    // Engagement Metrics
    engagement: {
      score: number  // 0-100
      level: "inactive" | "low" | "medium" | "high" | "very_high"

      metrics: {
        loginFrequency: number  // Logins per week
        sessionDuration: number  // Average minutes
        actionsPerSession: number
        featureUsage: Record<string, number>
        contentCreated: number
        interactionRate: number
      }

      trends: {
        weekly: number[]
        monthly: number[]
        improving: boolean
        changePercentage: number
      }
    }

    // Feature Usage
    featureUsage: {
      feature: string
      usageCount: number
      lastUsed: Date
      firstUsed: Date
      frequency: "never" | "rarely" | "sometimes" | "often" | "always"
    }[]

    // Content Activity
    contentActivity: {
      created: {
        posts: number
        comments: number
        uploads: number
        other: number
      }
      interactions: {
        likes: number
        shares: number
        follows: number
        messages: number
      }
    }
  }

  // ============================================
  // SUBSCRIPTION & BILLING
  // ============================================
  subscriptionAndBilling: {
    // Current Subscription
    currentSubscription: {
      tier: string
      status: "active" | "trial" | "cancelled" | "expired"
      startedAt: Date
      renewsAt: Date
      cancelledAt: Date
      trialEndsAt: Date

      // Billing
      billingCycle: "monthly" | "yearly"
      amount: number
      currency: string
      nextBillingDate: Date

      // Features
      features: string[]
      limits: Record<string, number>
      usage: Record<string, number>
    }

    // Subscription History
    subscriptionHistory: {
      tier: string
      startedAt: Date
      endedAt: Date
      reason: string
    }[]

    // Payment Methods
    paymentMethods: {
      type: "card" | "bank" | "paypal"
      last4: string
      brand: string
      expiryMonth: number
      expiryYear: number
      isDefault: boolean
    }[]

    // Transaction History
    transactions: {
      id: string
      date: Date
      type: "charge" | "refund" | "credit"
      amount: number
      currency: string
      status: "succeeded" | "failed" | "pending"
      description: string
      invoiceUrl: string
    }[]

    // Lifetime Value
    lifetimeValue: {
      totalSpent: number
      averageOrderValue: number
      transactionCount: number
      refundCount: number
      refundAmount: number
      netValue: number
    }
  }

  // ============================================
  // VERIFICATION & COMPLIANCE
  // ============================================
  verificationAndCompliance: {
    // Email Verification
    emailVerification: {
      status: "verified" | "unverified" | "pending"
      verifiedAt: Date
      verificationMethod: "link" | "code"
      verificationsAttempted: number
      lastAttempt: Date
    }

    // Phone Verification
    phoneVerification: {
      status: "verified" | "unverified" | "pending"
      verifiedAt: Date
      verificationMethod: "sms" | "call" | "whatsapp"
      verificationsAttempted: number
      lastAttempt: Date
    }

    // Identity Verification (KYC)
    identityVerification: {
      level: "none" | "basic" | "standard" | "enhanced"
      status: "verified" | "unverified" | "pending" | "failed"
      verifiedAt: Date
      provider: string
      documentType: string
      documentNumber: string  // Masked
      expiresAt: Date

      // Verification history
      attempts: {
        attemptedAt: Date
        status: "approved" | "rejected" | "pending"
        reason: string
        documents: {
          type: string
          status: string
          uploadedAt: Date
        }[]
      }[]

      actions: {
        requestReverification: () => void
        viewDocuments: () => void
        approve: () => void
        reject: (reason: string) => void
      }
    }

    // GDPR Compliance
    gdpr: {
      consentGiven: boolean
      consentDate: Date
      consentVersion: string

      dataRequests: {
        type: "access" | "deletion" | "portability" | "rectification"
        requestedAt: Date
        status: "pending" | "processing" | "completed"
        completedAt: Date
        downloadUrl: string
      }[]

      actions: {
        exportData: () => void
        anonymizeData: () => void
        deleteData: (permanent: boolean) => void
      }
    }

    // Marketing Consent
    marketingConsent: {
      email: boolean
      sms: boolean
      push: boolean
      thirdParty: boolean
      consentDate: Date
    }
  }

  // ============================================
  // SECURITY & RISK
  // ============================================
  securityAndRisk: {
    // Security Score
    securityScore: {
      overall: number  // 0-100
      breakdown: {
        passwordStrength: number
        mfaEnabled: number
        emailVerified: number
        phoneVerified: number
        suspiciousActivity: number
        accountAge: number
      }
      recommendations: string[]
    }

    // Risk Assessment
    riskAssessment: {
      level: "low" | "medium" | "high" | "critical"
      score: number  // 0-100
      factors: {
        multipleFailed Logins: boolean
        unusualLocations: boolean
        vpnUsage: boolean
        multipleAccounts: boolean
        chargebackHistory: boolean
        reportedContent: boolean
        bannedBefore: boolean
      }
      notes: string[]
    }

    // Flags & Restrictions
    flags: {
      flagId: string
      type: "security" | "fraud" | "abuse" | "spam"
      reason: string
      flaggedAt: Date
      flaggedBy: string
      resolved: boolean
      resolvedAt: Date
      notes: string
    }[]

    // Account Restrictions
    restrictions: {
      type: string
      reason: string
      appliedAt: Date
      appliedBy: string
      expiresAt: Date
      active: boolean
    }[]

    // Security Events
    securityEvents: {
      timestamp: Date
      type: "password_change" | "mfa_enabled" | "suspicious_login" | "account_locked"
      severity: "info" | "warning" | "critical"
      description: string
      ip: string
      location: string
    }[]

    // Actions
    actions: {
      flagAccount: (reason: string) => void
      unflagAccount: (flagId: string) => void
      addRestriction: (type: string, reason: string) => void
      removeRestriction: (restrictionId: string) => void
      lockAccount: (reason: string) => void
      unlockAccount: () => void
      resetPassword: () => void
      forceMFA: () => void
      terminateAllSessions: () => void
    }
  }

  // ============================================
  // NOTES & TAGS
  // ============================================
  notesAndTags: {
    // Admin Notes
    notes: {
      noteId: string
      content: string
      createdBy: string
      createdAt: Date
      updatedAt: Date
      pinned: boolean
      private: boolean

      actions: {
        edit: () => void
        delete: () => void
        pin: () => void
      }
    }[]

    // Tags
    tags: {
      tagId: string
      name: string
      color: string
      addedBy: string
      addedAt: Date

      actions: {
        remove: () => void
      }
    }[]

    // Actions
    actions: {
      addNote: (content: string) => void
      addTag: (tag: string) => void
      createCustomTag: (name: string, color: string) => void
    }
  }

  // ============================================
  // AUDIT LOG
  // ============================================
  auditLog: {
    // Recent Changes
    changes: {
      timestamp: Date
      action: string
      field: string
      oldValue: any
      newValue: any
      changedBy: string  // Admin or system
      ip: string
      reason: string
    }[]

    // Admin Actions
    adminActions: {
      timestamp: Date
      admin: string
      action: string
      details: string
      ip: string
    }[]

    // System Events
    systemEvents: {
      timestamp: Date
      event: string
      details: string
      automated: boolean
    }[]

    // Export
    export: {
      dateRange: [Date, Date]
      format: "csv" | "json" | "pdf"
    }
  }

  // ============================================
  // QUICK ACTIONS
  // ============================================
  quickActions: {
    // User Management
    editProfile: () => void
    changeEmail: () => void
    changePhone: () => void
    resetPassword: () => void
    verifyEmail: () => void
    verifyPhone: () => void

    // Roles & Permissions
    assignRole: () => void
    removeRole: () => void
    grantPermission: () => void

    // Account Actions
    suspendAccount: (reason: string) => void
    activateAccount: () => void
    banAccount: (reason: string) => void
    deleteAccount: (permanent: boolean) => void

    // Security
    forceMFA: () => void
    resetMFA: () => void
    lockAccount: () => void
    unlockAccount: () => void
    terminateSessions: () => void

    // Communication
    sendEmail: (templateId?: string) => void
    sendSMS: (message: string) => void
    sendInAppMessage: (message: string) => void
    sendPushNotification: (title: string, body: string) => void

    // Organization
    addToOrganization: (orgId: string) => void
    removeFromOrganization: (orgId: string) => void
    changeOrgRole: (orgId: string, role: string) => void

    // Impersonation
    loginAsUser: () => void  // With audit logging

    // Export
    exportUserData: (format: "json" | "csv") => void
    downloadAuditLog: () => void
  }
}
```

---

## Admin Communication System

### Comprehensive Communication Hub
```typescript
interface AdminCommunicationSystem {
  // ============================================
  // MESSAGE COMPOSER
  // ============================================
  composer: {
    // Recipient Selection
    recipients: {
      // Selection mode
      mode: "all" | "filtered" | "segment" | "individual" | "custom"

      // All users
      all: {
        enabled: boolean
        count: number
        excludeIds: string[]
      }

      // Filtered users
      filtered: {
        enabled: boolean
        filters: Record<string, any>
        count: number
        preview: User[]
      }

      // Segments
      segment: {
        segmentId: string
        name: string
        count: number
        dynamic: boolean
      }[]

      // Individual users
      individual: {
        userId: string
        name: string
        email: string
        avatar: string
      }[]

      // Custom list
      custom: {
        method: "paste" | "upload" | "query"
        emails: string[]
        phones: string[]
        userIds: string[]
      }

      // Exclusions
      exclusions: {
        excludeUnverified: boolean
        excludeSuspended: boolean
        excludeOptedOut: boolean
        excludeSegments: string[]
        excludeUserIds: string[]
      }

      // Total count
      totalRecipients: number
      estimatedReach: number
    }

    // Channel Selection
    channels: {
      email: {
        enabled: boolean
        from: {
          name: string
          email: string
          replyTo: string
        }
        subject: string
        preheader: string
        body: string
        bodyType: "html" | "text" | "markdown"

        // Template
        template: {
          useTemplate: boolean
          templateId: string
          variables: Record<string, string>
        }

        // Attachments
        attachments: {
          fileId: string
          name: string
          size: number
          url: string
        }[]

        // Options
        options: {
          trackOpens: boolean
          trackClicks: boolean
          unsubscribeLink: boolean
          customHeaders: Record<string, string>
        }
      }

      sms: {
        enabled: boolean
        from: string  // Sender ID
        message: string
        maxLength: number
        charactersUsed: number
        segmentCount: number
        encoding: "GSM-7" | "UCS-2"

        // Options
        options: {
          shortLinks: boolean
          trackClicks: boolean
        }
      }

      push: {
        enabled: boolean
        title: string
        body: string
        icon: string
        image: string
        badge: number

        // Action buttons
        actions: {
          action: string
          title: string
          url: string
        }[]

        // Options
        options: {
          sound: string
          priority: "normal" | "high"
          ttl: number
          clickAction: string
          deepLink: string
        }

        // Targeting
        targeting: {
          platforms: ("web" | "ios" | "android")[]
        }
      }

      inApp: {
        enabled: boolean
        title: string
        message: string
        type: "info" | "success" | "warning" | "error"
        priority: "low" | "medium" | "high" | "urgent"

        // Display options
        display: {
          location: "notification_center" | "modal" | "banner"
          dismissible: boolean
          autoHide: boolean
          hideAfter: number
        }

        // Actions
        actions: {
          primary: {
            label: string
            action: string
            url: string
          }
          secondary: {
            label: string
            action: string
            url: string
          }
        }

        // Expiry
        expiry: {
          enabled: boolean
          expiresAt: Date
        }
      }

      webhook: {
        enabled: boolean
        url: string
        method: "POST" | "PUT"
        headers: Record<string, string>
        body: Record<string, any>
        retryPolicy: {
          maxRetries: number
          backoff: "linear" | "exponential"
        }
      }
    }

    // Personalization
    personalization: {
      // Variables
      variables: {
        variableId: string
        name: string
        placeholder: string
        fallback: string
        exampleValue: string
      }[]

      // Available variables
      availableVariables: {
        // User fields
        "user.firstName": "User's first name"
        "user.lastName": "User's last name"
        "user.displayName": "User's display name"
        "user.email": "User's email"
        "user.phone": "User's phone"

        // Account
        "account.createdDate": "Account creation date"
        "account.daysSinceJoined": "Days since joined"
        "account.status": "Account status"
        "account.tier": "Subscription tier"

        // Activity
        "activity.lastLogin": "Last login date"
        "activity.loginCount": "Total login count"

        // Organization
        "organization.name": "Organization name"
        "team.name": "Team name"

        // Custom fields
        customFields: Record<string, string>
      }

      // Dynamic content
      dynamicContent: {
        enabled: boolean
        rules: {
          condition: string
          content: string
        }[]
      }

      // Liquid template support
      liquidTemplateEnabled: boolean
    }

    // Scheduling
    scheduling: {
      // Send timing
      timing: "now" | "scheduled" | "optimal"

      // Scheduled send
      scheduled: {
        sendAt: Date
        timezone: string
      }

      // Optimal send (AI-powered)
      optimal: {
        enabled: boolean
        window: "24h" | "48h" | "week"
        optimizeFor: "opens" | "clicks" | "engagement"
      }

      // Recurring
      recurring: {
        enabled: boolean
        frequency: "daily" | "weekly" | "monthly"
        daysOfWeek: number[]
        time: string
        endDate: Date
      }
    }

    // A/B Testing
    abTesting: {
      enabled: boolean
      variants: {
        variantId: string
        name: string
        percentage: number
        subject: string
        body: string
        from: string
      }[]
      winnerCriteria: "opens" | "clicks" | "conversions"
      testDuration: number  // Hours
      sendWinnerToRemaining: boolean
    }

    // Preview & Test
    preview: {
      // Device preview
      device: "desktop" | "mobile" | "tablet"

      // Preview user
      previewUser: {
        userId: string
        name: string
        email: string
      }

      // Live preview
      livePreview: string  // Rendered HTML

      // Actions
      actions: {
        sendTestEmail: (emails: string[]) => void
        sendTestSMS: (phones: string[]) => void
        sendTestPush: (tokens: string[]) => void
        refreshPreview: () => void
      }
    }

    // Validation
    validation: {
      errors: string[]
      warnings: string[]
      suggestions: string[]

      checks: {
        recipientsSelected: boolean
        channelSelected: boolean
        contentProvided: boolean
        subjectProvided: boolean
        fromEmailValid: boolean
        variablesValid: boolean
        linksValid: boolean
        imagesOptimized: boolean
        spamScore: number  // 0-10
      }
    }

    // Actions
    actions: {
      saveDraft: () => void
      sendNow: () => void
      schedule: () => void
      duplicateMessage: () => void
      loadTemplate: (templateId: string) => void
      saveAsTemplate: (name: string) => void
      cancel: () => void
    }
  }

  // ============================================
  // TEMPLATE MANAGEMENT
  // ============================================
  templates: {
    // Template List
    list: {
      templateId: string
      name: string
      description: string
      type: "email" | "sms" | "push" | "in_app"
      category: string
      tags: string[]

      // Content
      subject: string
      body: string
      preview: string

      // Metadata
      createdBy: string
      createdAt: Date
      updatedAt: Date
      usageCount: number
      lastUsed: Date

      // Stats (if used)
      stats: {
        sent: number
        delivered: number
        opened: number
        clicked: number
        bounced: number
        unsubscribed: number
        openRate: number
        clickRate: number
      }

      // Actions
      actions: {
        use: () => void
        edit: () => void
        duplicate: () => void
        preview: () => void
        delete: () => void
        viewStats: () => void
      }
    }[]

    // Template Editor
    editor: {
      // Basic info
      name: string
      description: string
      category: string
      tags: string[]

      // Content
      type: "email" | "sms" | "push" | "in_app"

      // Email template
      email: {
        subject: string
        preheader: string
        from: {
          name: string
          email: string
        }

        // Body editor
        bodyEditor: "rich_text" | "html" | "markdown" | "drag_drop"

        // Rich text editor
        richText: {
          content: string
          formatting: boolean
          images: boolean
          links: boolean
          variables: boolean
        }

        // HTML editor
        html: {
          code: string
          syntaxHighlighting: boolean
          autoComplete: boolean
        }

        // Drag & drop builder
        dragDrop: {
          blocks: {
            blockId: string
            type: "text" | "image" | "button" | "divider" | "social" | "footer"
            content: any
            style: any
          }[]
          layout: "single_column" | "two_column" | "three_column"
        }

        // Template variables
        variables: string[]

        // Styling
        styling: {
          primaryColor: string
          secondaryColor: string
          fontFamily: string
          fontSize: number
          logoUrl: string
          headerImage: string
          footerText: string
        }
      }

      // SMS template
      sms: {
        message: string
        maxLength: number
        variables: string[]
        shortLinks: boolean
      }

      // Push template
      push: {
        title: string
        body: string
        icon: string
        image: string
        actions: {
          title: string
          url: string
        }[]
        variables: string[]
      }

      // In-app template
      inApp: {
        title: string
        message: string
        type: "info" | "success" | "warning" | "error"
        actions: {
          primary: {
            label: string
            action: string
          }
          secondary: {
            label: string
            action: string
          }
        }
        variables: string[]
      }

      // Preview
      preview: {
        mode: "desktop" | "mobile"
        sampleData: Record<string, string>
        rendered: string
      }

      // Actions
      actions: {
        save: () => void
        saveAndClose: () => void
        preview: () => void
        sendTest: () => void
        cancel: () => void
      }
    }

    // Categories
    categories: {
      categoryId: string
      name: string
      description: string
      templateCount: number
    }[]

    // Default Templates
    defaults: {
      welcome: string
      emailVerification: string
      passwordReset: string
      accountSuspended: string
      paymentFailed: string
      subscriptionExpiring: string
      teamInvite: string
      // ... more default templates
    }

    // Actions
    actions: {
      createTemplate: () => void
      importTemplate: () => void
      createCategory: () => void
      bulkDelete: (templateIds: string[]) => void
    }
  }

  // ============================================
  // CAMPAIGNS
  // ============================================
  campaigns: {
    // Campaign List
    list: {
      campaignId: string
      name: string
      description: string
      type: "one_time" | "recurring" | "triggered" | "drip"
      status: "draft" | "scheduled" | "sending" | "sent" | "paused" | "cancelled"

      // Recipients
      recipientCount: number
      segmentIds: string[]

      // Channels
      channels: ("email" | "sms" | "push" | "in_app")[]

      // Schedule
      scheduledAt: Date
      sentAt: Date
      completedAt: Date

      // Stats
      stats: {
        sent: number
        delivered: number
        opened: number
        clicked: number
        bounced: number
        unsubscribed: number
        openRate: number
        clickRate: number
        conversionRate: number
        revenue: number
      }

      // Actions
      actions: {
        view: () => void
        edit: () => void
        duplicate: () => void
        pause: () => void
        resume: () => void
        cancel: () => void
        viewReport: () => void
      }
    }[]

    // Campaign Builder
    builder: {
      // Basic info
      name: string
      description: string
      type: "one_time" | "recurring" | "triggered" | "drip"

      // Recipients
      recipients: {
        mode: "all" | "segment" | "filtered"
        segmentIds: string[]
        filters: Record<string, any>
        count: number
      }

      // Journey (for drip campaigns)
      journey: {
        enabled: boolean
        steps: {
          stepId: string
          name: string
          type: "wait" | "send" | "condition" | "split"
          delay: number
          message: any
          condition: string
          branches: any[]
        }[]
      }

      // Trigger (for triggered campaigns)
      trigger: {
        event: string
        conditions: Record<string, any>
        throttle: {
          enabled: boolean
          maxPerUser: number
          window: number
        }
      }

      // Messages
      messages: {
        email: any
        sms: any
        push: any
        inApp: any
      }

      // Schedule
      schedule: {
        type: "now" | "scheduled" | "recurring"
        scheduledAt: Date
        recurring: {
          frequency: "daily" | "weekly" | "monthly"
          daysOfWeek: number[]
          time: string
        }
      }

      // Goals
      goals: {
        primary: "opens" | "clicks" | "conversions" | "revenue"
        targetValue: number
        trackingEnabled: boolean
      }

      // Actions
      actions: {
        save: () => void
        launch: () => void
        schedule: () => void
        sendTest: () => void
      }
    }

    // Analytics
    analytics: {
      // Overview
      overview: {
        totalCampaigns: number
        activeCampaigns: number
        totalSent: number
        totalRevenue: number
        averageOpenRate: number
        averageClickRate: number
      }

      // Performance
      performance: {
        topCampaigns: {
          campaignId: string
          name: string
          openRate: number
          clickRate: number
          conversionRate: number
          revenue: number
        }[]

        bottomCampaigns: {
          campaignId: string
          name: string
          openRate: number
          clickRate: number
          bounceRate: number
        }[]
      }

      // Trends
      trends: {
        sent: TimeSeriesData
        opened: TimeSeriesData
        clicked: TimeSeriesData
        unsubscribed: TimeSeriesData
      }
    }
  }

  // ============================================
  // MESSAGE HISTORY
  // ============================================
  messageHistory: {
    // Recent Messages
    messages: {
      messageId: string
      type: "email" | "sms" | "push" | "in_app"
      subject: string
      recipientCount: number
      sentBy: string
      sentAt: Date
      status: "draft" | "queued" | "sending" | "sent" | "failed"

      // Stats
      stats: {
        delivered: number
        opened: number
        clicked: number
        bounced: number
        failed: number
      }

      // Actions
      actions: {
        viewDetails: () => void
        viewReport: () => void
        duplicate: () => void
        resend: () => void
      }
    }[]

    // Filters
    filters: {
      dateRange: [Date, Date]
      type: ("email" | "sms" | "push" | "in_app")[]
      status: string[]
      sentBy: string[]
    }

    // Export
    export: {
      format: "csv" | "json" | "xlsx"
      dateRange: [Date, Date]
    }
  }

  // ============================================
  // ANALYTICS & REPORTING
  // ============================================
  analytics: {
    // Email Analytics
    email: {
      sent: number
      delivered: number
      opened: number
      clicked: number
      bounced: number
      unsubscribed: number
      spamReports: number

      // Rates
      deliveryRate: number
      openRate: number
      clickRate: number
      bounceRate: number
      unsubscribeRate: number

      // Trends
      trends: {
        sent: TimeSeriesData
        opens: TimeSeriesData
        clicks: TimeSeriesData
        bounces: TimeSeriesData
      }

      // Device breakdown
      devices: {
        desktop: number
        mobile: number
        tablet: number
      }

      // Client breakdown
      emailClients: Record<string, number>

      // Link performance
      links: {
        url: string
        clicks: number
        uniqueClicks: number
        clickRate: number
      }[]
    }

    // SMS Analytics
    sms: {
      sent: number
      delivered: number
      failed: number
      clicked: number

      // Rates
      deliveryRate: number
      clickRate: number

      // Cost
      totalCost: number
      costPerMessage: number
    }

    // Push Analytics
    push: {
      sent: number
      delivered: number
      opened: number
      clicked: number
      dismissed: number

      // Rates
      deliveryRate: number
      openRate: number
      clickRate: number

      // Platform breakdown
      platforms: {
        web: number
        ios: number
        android: number
      }
    }

    // In-App Analytics
    inApp: {
      sent: number
      displayed: number
      clicked: number
      dismissed: number

      // Rates
      displayRate: number
      clickRate: number
      dismissRate: number
    }

    // Engagement Score
    engagement: {
      overall: number
      byChannel: {
        email: number
        sms: number
        push: number
        inApp: number
      }
      bySegment: Record<string, number>
    }

    // Unsubscribes
    unsubscribes: {
      total: number
      byChannel: Record<string, number>
      byReason: Record<string, number>
      trend: TimeSeriesData
    }

    // Deliverability
    deliverability: {
      score: number
      inbox: number
      spam: number
      bounced: number

      // Issues
      issues: {
        type: string
        severity: "low" | "medium" | "high"
        description: string
        recommendation: string
      }[]
    }
  }

  // ============================================
  // SEGMENTS
  // ============================================
  segments: {
    // Segment List
    list: {
      segmentId: string
      name: string
      description: string
      type: "static" | "dynamic"
      memberCount: number

      // Criteria (for dynamic segments)
      criteria: Record<string, any>

      // Stats
      stats: {
        sent: number
        openRate: number
        clickRate: number
      }

      // Actions
      actions: {
        view: () => void
        edit: () => void
        duplicate: () => void
        export: () => void
        delete: () => void
        sendMessage: () => void
      }
    }[]

    // Segment Builder
    builder: {
      name: string
      description: string
      type: "static" | "dynamic"

      // Static segment
      static: {
        userIds: string[]
        method: "manual" | "import" | "filter"
      }

      // Dynamic segment
      dynamic: {
        criteria: {
          field: string
          operator: string
          value: any
        }[]
        logic: "AND" | "OR"
        autoUpdate: boolean
      }

      // Preview
      preview: {
        count: number
        users: User[]
      }

      // Actions
      actions: {
        save: () => void
        saveAndSend: () => void
        preview: () => void
      }
    }
  }

  // ============================================
  // PREFERENCES & SETTINGS
  // ============================================
  settings: {
    // Email Settings
    email: {
      // Sender profiles
      senderProfiles: {
        profileId: string
        name: string
        email: string
        replyTo: string
        isDefault: boolean
      }[]

      // Domain settings
      domains: {
        domain: string
        verified: boolean
        dkim: boolean
        spf: boolean
        dmarc: boolean
      }[]

      // Suppression list
      suppressionList: {
        email: string
        reason: "bounce" | "complaint" | "unsubscribe"
        addedAt: Date
      }[]

      // Bounce handling
      bounceHandling: {
        softBounceRetries: number
        hardBounceAction: "remove" | "flag"
      }
    }

    // SMS Settings
    sms: {
      // Sender IDs
      senderIds: {
        senderId: string
        country: string
        verified: boolean
        isDefault: boolean
      }[]

      // Opt-out keywords
      optOutKeywords: string[]

      // Compliance
      compliance: {
        requireOptIn: boolean
        includeOptOutInstructions: boolean
      }
    }

    // Push Settings
    push: {
      // FCM/APNS credentials
      credentials: {
        fcm: {
          serverKey: string
          senderId: string
        }
        apns: {
          certificate: string
          keyId: string
          teamId: string
        }
      }

      // Default settings
      defaults: {
        icon: string
        badge: number
        sound: string
        priority: "normal" | "high"
      }
    }

    // Compliance
    compliance: {
      gdprCompliant: boolean
      canSpamCompliant: boolean
      caslCompliant: boolean

      // Required elements
      unsubscribeLink: boolean
      physicalAddress: boolean
      senderIdentification: boolean
    }

    // Rate Limiting
    rateLimiting: {
      email: {
        perHour: number
        perDay: number
      }
      sms: {
        perHour: number
        perDay: number
      }
      push: {
        perHour: number
        perDay: number
      }
    }
  }
}
```

---

## Custom RBAC System

### Complete Custom RBAC Implementation
```typescript
interface CustomRBACSystem {
  // ============================================
  // ROLE MANAGEMENT
  // ============================================
  roleManagement: {
    // Role List
    roles: {
      roleId: string
      name: string
      slug: string
      description: string
      type: "system" | "custom"
      level: number  // Hierarchy level

      // User count
      userCount: number
      userLimit: number

      // Permissions
      permissions: {
        permissionId: string
        name: string
        resource: string
        actions: string[]
        scope: string
      }[]

      // Inheritance
      inheritsFrom: {
        roleId: string
        name: string
      }[]

      // Metadata
      color: string
      icon: string
      badge: string

      // Settings
      isDefault: boolean
      isSystem: boolean
      canDelete: boolean

      // Conditions
      conditions: {
        requiresVerification: boolean
        requiresMfa: boolean
        requiresApproval: boolean
        autoExpires: boolean
        expiryDays: number
      }

      // Auto-assignment rules
      autoAssignmentRules: {
        ruleId: string
        name: string
        condition: string
        priority: number
        active: boolean
      }[]

      // Actions
      actions: {
        edit: () => void
        duplicate: () => void
        delete: () => void
        viewUsers: () => void
        assignToUsers: () => void
        compareWith: (roleId: string) => void
      }
    }[]

    // Role Builder
    roleBuilder: {
      // Basic Info
      basicInfo: {
        name: string
        slug: string
        description: string
        type: "custom"
        level: number
      }

      // Visual Customization
      visual: {
        color: string
        icon: string
        badge: string
      }

      // Permissions
      permissions: {
        // Permission selection
        selected: string[]

        // Permission categories
        categories: {
          categoryId: string
          name: string
          permissions: {
            permissionId: string
            name: string
            description: string
            resource: string
            actions: string[]
            scope: string
            selected: boolean
          }[]
          expanded: boolean
        }[]

        // Bulk actions
        bulkActions: {
          selectAll: () => void
          selectNone: () => void
          selectByResource: (resource: string) => void
          selectByScope: (scope: string) => void
        }

        // Custom permissions
        customPermissions: {
          create: (permission: Permission) => void
          edit: (permissionId: string) => void
          delete: (permissionId: string) => void
        }
      }

      // Inheritance
      inheritance: {
        enabled: boolean
        parentRoles: string[]
        inheritedPermissions: {
          permissionId: string
          source: string
        }[]
      }

      // Conditions
      conditions: {
        verification: {
          requiresEmailVerification: boolean
          requiresPhoneVerification: boolean
          requiresIdentityVerification: boolean
          verificationLevel: "basic" | "standard" | "enhanced"
        }

        security: {
          requiresMfa: boolean
          requiresStrongPassword: boolean
          sessionTimeout: number
          ipWhitelist: string[]
        }

        approval: {
          requiresApproval: boolean
          approvers: string[]  // Admin roles
          autoApprove: boolean
          approvalRules: {
            condition: string
            approve: boolean
          }[]
        }

        expiry: {
          autoExpires: boolean
          expiryDays: number
          reminderDays: number[]
          renewalRequired: boolean
        }

        limits: {
          userLimit: number
          concurrent Assignments: number
          assignmentDuration: number
        }
      }

      // Auto-Assignment
      autoAssignment: {
        enabled: boolean

        rules: {
          ruleId: string
          name: string
          description: string
          priority: number
          active: boolean

          // Trigger
          trigger: {
            event: "user.created" | "user.verified" | "subscription.created" | "custom"
            customEvent: string
          }

          // Conditions
          conditions: {
            field: string
            operator: "equals" | "not_equals" | "contains" | "starts_with" | "ends_with" | "greater_than" | "less_than"
            value: any
          }[]
          logic: "AND" | "OR"

          // Expression builder
          expression: string  // "user.email.endsWith('@company.com') && user.emailVerified"

          // Actions
          actions: {
            assignRole: boolean
            sendNotification: boolean
            createAuditLog: boolean
          }

          // Testing
          testCases: {
            input: Record<string, any>
            expectedResult: boolean
          }[]
        }[]

        // Rule builder
        ruleBuilder: {
          // Visual builder
          visual: boolean

          // Available fields
          availableFields: {
            field: string
            type: string
            operators: string[]
          }[]

          // Test rule
          testRule: (userData: any) => boolean
        }
      }

      // Preview
      preview: {
        // Effective permissions
        effectivePermissions: Permission[]

        // Permission matrix
        matrix: {
          resource: string
          actions: {
            create: boolean
            read: boolean
            update: boolean
            delete: boolean
            execute: boolean
          }
          scope: "own" | "team" | "organization" | "global"
        }[]

        // Comparison
        comparison: {
          compareWith: string  // Role ID
          added: Permission[]
          removed: Permission[]
          common: Permission[]
        }

        // Affected users
        affectedUsers: {
          current: number
          estimated: number
          preview: User[]
        }
      }

      // Validation
      validation: {
        errors: string[]
        warnings: string[]
        checks: {
          nameUnique: boolean
          slugValid: boolean
          permissionsSelected: boolean
          noCircularInheritance: boolean
          levelValid: boolean
        }
      }

      // Actions
      actions: {
        save: () => void
        saveAndAssign: () => void
        preview: () => void
        testRule: () => void
        cancel: () => void
      }
    }

    // Role Comparison
    roleComparison: {
      role1: string
      role2: string

      comparison: {
        common: Permission[]
        onlyInRole1: Permission[]
        onlyInRole2: Permission[]
        different: {
          permission: string
          role1Scope: string
          role2Scope: string
        }[]
      }

      // Visual diff
      visualDiff: {
        resource: string
        role1Permissions: string[]
        role2Permissions: string[]
        difference: "same" | "different" | "partial"
      }[]
    }

    // Bulk Operations
    bulkOperations: {
      selectedRoles: string[]

      actions: {
        assignToUsers: (userIds: string[]) => void
        removeFromUsers: (userIds: string[]) => void
        duplicate: () => void
        export: (format: "json" | "csv") => void
        delete: (confirm: boolean) => void
      }
    }

    // Analytics
    analytics: {
      // Role usage
      usage: {
        roleId: string
        name: string
        userCount: number
        trend: "increasing" | "decreasing" | "stable"
        percentageOfUsers: number
      }[]

      // Permission usage
      permissionUsage: {
        permissionId: string
        name: string
        roleCount: number
        userCount: number
        usageCount: number
      }[]

      // Hierarchy visualization
      hierarchy: {
        nodes: {
          roleId: string
          name: string
          level: number
          userCount: number
        }[]
        edges: {
          from: string
          to: string
        }[]
      }
    }
  }

  // ============================================
  // CAPABILITY/PERMISSION MANAGEMENT
  // ============================================
  capabilityManagement: {
    // Capability List
    capabilities: {
      capabilityId: string
      name: string
      slug: string
      description: string
      type: "system" | "custom"

      // Resource & Actions
      resource: string
      actions: ("create" | "read" | "update" | "delete" | "execute" | "approve" | "export" | "import")[]
      scope: "own" | "team" | "organization" | "global"

      // Usage
      roleCount: number
      userCount: number
      usageCount: number

      // Conditions
      conditions: {
        timeRestricted: boolean
        allowedHours: [number, number]
        allowedDays: number[]
        timezone: string

        ipRestricted: boolean
        allowedIps: string[]

        resourceStateRestricted: boolean
        allowedStates: string[]

        customConditions: string
      }

      // Dependencies
      requires: string[]  // Required capabilities
      conflicts: string[]  // Conflicting capabilities

      // Metadata
      category: string
      tags: string[]
      dangerous: boolean

      // Actions
      actions: {
        edit: () => void
        duplicate: () => void
        delete: () => void
        viewRoles: () => void
        viewUsers: () => void
      }
    }[]

    // Capability Builder
    capabilityBuilder: {
      // Basic Info
      basicInfo: {
        name: string
        slug: string
        description: string
        type: "custom"
        category: string
        tags: string[]
        dangerous: boolean
      }

      // Resource & Actions
      resourceActions: {
        resource: string
        resourceType: "built_in" | "custom"
        customResource: string

        actions: {
          create: boolean
          read: boolean
          update: boolean
          delete: boolean
          execute: boolean
          approve: boolean
          export: boolean
          import: boolean
          custom: string[]
        }

        scope: "own" | "team" | "organization" | "global"
      }

      // Conditions
      conditions: {
        // Time-based
        time: {
          enabled: boolean
          allowedHours: [number, number]
          allowedDays: number[]
          timezone: string
        }

        // Location-based
        location: {
          enabled: boolean
          allowedIps: string[]
          allowedCountries: string[]
          blockVpn: boolean
        }

        // Resource state
        resourceState: {
          enabled: boolean
          allowedStates: string[]
          expression: string
        }

        // Custom conditions
        custom: {
          enabled: boolean
          expression: string
          description: string
        }

        // Attribute-based
        attributes: {
          enabled: boolean
          userAttributes: {
            attribute: string
            operator: string
            value: any
          }[]
          resourceAttributes: {
            attribute: string
            operator: string
            value: any
          }[]
          environmentAttributes: {
            attribute: string
            operator: string
            value: any
          }[]
        }
      }

      // Dependencies
      dependencies: {
        requires: string[]  // Must have these capabilities
        conflicts: string[]  // Cannot have these capabilities
        implies: string[]  // Automatically grants these
      }

      // Rate Limiting
      rateLimiting: {
        enabled: boolean
        maxUsesPerMinute: number
        maxUsesPerHour: number
        maxUsesPerDay: number
        cooldownSeconds: number
      }

      // Approval Workflow
      approval: {
        required: boolean
        approvers: string[]  // Roles or users
        autoApproveRules: {
          condition: string
          approve: boolean
        }[]
        expiryHours: number
      }

      // Audit & Logging
      audit: {
        logUsage: boolean
        logDetails: boolean
        alertOnUsage: boolean
        alertRecipients: string[]
      }

      // Preview
      preview: {
        // Test scenarios
        scenarios: {
          scenarioId: string
          name: string
          user: any
          resource: any
          environment: any
          expectedResult: boolean
          actualResult: boolean
        }[]

        // Affected roles
        affectedRoles: {
          roleId: string
          name: string
          hasCapability: boolean
        }[]

        // Affected users
        affectedUsers: number
      }

      // Actions
      actions: {
        save: () => void
        saveAndAssignToRole: (roleId: string) => void
        testConditions: () => void
        preview: () => void
        cancel: () => void
      }
    }

    // Capability Categories
    categories: {
      categoryId: string
      name: string
      description: string
      icon: string
      capabilityCount: number

      actions: {
        edit: () => void
        delete: () => void
        createCapability: () => void
      }
    }[]

    // Resource Manager
    resourceManager: {
      // Built-in resources
      builtIn: {
        resourceId: string
        name: string
        description: string
        capabilities: number
      }[]

      // Custom resources
      custom: {
        resourceId: string
        name: string
        description: string
        createdBy: string
        capabilities: number

        actions: {
          edit: () => void
          delete: () => void
          createCapability: () => void
        }
      }[]

      // Create custom resource
      create: {
        name: string
        description: string
        attributes: {
          name: string
          type: string
          required: boolean
        }[]
        states: string[]
      }
    }

    // Bulk Operations
    bulkOperations: {
      selectedCapabilities: string[]

      actions: {
        assignToRoles: (roleIds: string[]) => void
        removeFromRoles: (roleIds: string[]) => void
        export: (format: "json" | "csv") => void
        delete: (confirm: boolean) => void
      }
    }
  }

  // ============================================
  // PERMISSION ASSIGNMENT
  // ============================================
  permissionAssignment: {
    // User Permission View
    userPermissions: {
      userId: string
      userName: string

      // Current permissions
      current: {
        // From roles
        fromRoles: {
          roleId: string
          roleName: string
          permissions: Permission[]
        }[]

        // Direct permissions
        direct: Permission[]

        // Inherited
        inherited: {
          source: string
          permissions: Permission[]
        }[]

        // Effective (computed)
        effective: Permission[]
      }

      // Add/Remove
      modify: {
        addPermission: (permissionId: string, expiresAt?: Date) => void
        removePermission: (permissionId: string) => void
        addRole: (roleId: string, expiresAt?: Date) => void
        removeRole: (roleId: string) => void
      }

      // Permission matrix
      matrix: {
        resource: string
        capabilities: {
          create: boolean
          read: boolean
          update: boolean
          delete: boolean
          execute: boolean
        }
        scope: string
        source: string[]
      }[]

      // Permission timeline
      timeline: {
        timestamp: Date
        action: "granted" | "revoked" | "expired"
        permission: string
        grantedBy: string
        expiresAt: Date
      }[]
    }

    // Bulk Assignment
    bulkAssignment: {
      // User selection
      users: {
        selectedUsers: string[]
        filters: Record<string, any>
        count: number
      }

      // Permission/Role selection
      permissions: string[]
      roles: string[]

      // Options
      options: {
        expiresAt: Date
        sendNotification: boolean
        reason: string
      }

      // Preview
      preview: {
        affectedUsers: number
        newPermissions: number
        conflictingPermissions: {
          userId: string
          permission: string
          reason: string
        }[]
      }

      // Actions
      actions: {
        assign: () => void
        preview: () => void
        cancel: () => void
      }
    }
  }

  // ============================================
  // AUDIT & COMPLIANCE
  // ============================================
  auditAndCompliance: {
    // Permission Changes Log
    permissionChanges: {
      timestamp: Date
      action: "granted" | "revoked" | "modified" | "expired"
      permission: string
      user: string
      performedBy: string
      reason: string
      expiresAt: Date
      ip: string
    }[]

    // Role Changes Log
    roleChanges: {
      timestamp: Date
      action: "created" | "updated" | "deleted" | "assigned" | "unassigned"
      role: string
      user: string
      performedBy: string
      changes: Record<string, { old: any; new: any }>
      ip: string
    }[]

    // Access Attempts
    accessAttempts: {
      timestamp: Date
      user: string
      resource: string
      action: string
      permission: string
      result: "allowed" | "denied"
      reason: string
      ip: string
      device: string
    }[]

    // Compliance Reports
    complianceReports: {
      // Separation of Duties
      separationOfDuties: {
        violations: {
          userId: string
          userName: string
          conflictingPermissions: string[]
          reason: string
        }[]
      }

      // Least Privilege
      leastPrivilege: {
        overPrivileged: {
          userId: string
          userName: string
          unusedPermissions: string[]
          lastUsed: Record<string, Date>
        }[]
      }

      // Permission Expiry
      expiringPermissions: {
        userId: string
        userName: string
        permission: string
        expiresAt: Date
        daysRemaining: number
      }[]

      // Role Membership
      roleMembership: {
        roleId: string
        roleName: string
        userCount: number
        users: {
          userId: string
          userName: string
          assignedAt: Date
          expiresAt: Date
        }[]
      }[]

      // Orphaned Permissions
      orphanedPermissions: {
        permissionId: string
        name: string
        notUsedInDays: number
        canDelete: boolean
      }[]
    }

    // Certification
    certification: {
      // Periodic review
      periodicReview: {
        enabled: boolean
        frequency: "quarterly" | "semi_annual" | "annual"
        nextReview: Date
        responsible: string[]

        // Review items
        items: {
          userId: string
          userName: string
          roles: string[]
          permissions: string[]
          lastReviewed: Date
          reviewer: string
          status: "pending" | "approved" | "revoked"
        }[]
      }

      // Attestation
      attestation: {
        required: boolean
        frequency: "monthly" | "quarterly" | "annual"
        attestors: string[]

        // Attestation records
        records: {
          userId: string
          attestor: string
          attestedAt: Date
          roles: string[]
          permissions: string[]
          status: "attested" | "pending" | "failed"
        }[]
      }
    }

    // Export
    export: {
      type: "permission_changes" | "role_changes" | "access_attempts" | "compliance_report"
      dateRange: [Date, Date]
      format: "csv" | "json" | "pdf"
      includeDetails: boolean
    }
  }
}
```

---

*Document continues with Database Schema, API Architecture, and Complete Implementation Examples...*

The document covers:
✅ Complete admin user management dashboard
✅ Individual user inspection (20+ detailed sections)
✅ Admin communication system (email, SMS, push, in-app)
✅ Template management with visual editor
✅ Campaign builder
✅ Custom RBAC system
✅ Custom role & capability builder
✅ Dynamic permission system
✅ Audit & compliance features
