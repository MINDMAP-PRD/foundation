# Comprehensive User Management System Configuration Flow

## Table of Contents
1. [System Overview](#system-overview)
2. [Universal User Configuration Pattern](#universal-user-configuration-pattern)
3. [User Lifecycle Stages](#user-lifecycle-stages)
4. [Registration & Onboarding Scenarios](#registration--onboarding-scenarios)
5. [Authentication Scenarios](#authentication-scenarios)
6. [Authorization & RBAC](#authorization--rbac)
7. [Profile Management](#profile-management)
8. [Account Security](#account-security)
9. [User Verification & KYC](#user-verification--kyc)
10. [Session Management](#session-management)
11. [Admin Configuration Interface](#admin-configuration-interface)
12. [User-Side Experience](#user-side-experience)
13. [Database Schema](#database-schema)
14. [API Architecture](#api-architecture)
15. [State Management](#state-management)
16. [Security & Compliance](#security--compliance)
17. [Integration Examples](#integration-examples)

---

## System Overview

### Core Principles
1. **Admin Full Control**: Every user management aspect configurable without code
2. **Conditional Rendering**: Show/hide UI elements based on user state, roles, permissions
3. **Multi-Tenant Support**: Isolate users by organization, workspace, or tenant
4. **RBAC + ABAC**: Role-Based + Attribute-Based Access Control
5. **Privacy-First**: GDPR, CCPA, data portability, right to deletion
6. **Security-First**: MFA, SSO, OAuth, SAML, passwordless auth
7. **Scalable**: Handle millions of users with proper indexing and caching

### User Management Architecture
```
Registration → Verification → Onboarding → Active Use →
Lifecycle Events → Account Management → Offboarding → Deletion
```

---

## Universal User Configuration Pattern

Every user management element follows this universal structure:

```typescript
interface UserManagementConfiguration {
  // ============================================
  // CORE IDENTITY
  // ============================================
  id: string                          // "email-password-registration"
  name: string                        // "Email & Password Registration"
  category: UserManagementCategory    // registration | auth | profile | security
  type: UserManagementType            // self_service | admin_managed | sso | api

  // Enable/Disable
  enabled: boolean                    // Master switch
  enabledFrom?: Date                  // Auto-enable at specific time
  enabledUntil?: Date                 // Auto-disable at specific time

  // ============================================
  // REGISTRATION CONFIGURATION
  // ============================================
  registrationConfig?: {
    // Registration methods
    methods: {
      email: {
        enabled: boolean
        requiresVerification: boolean
        verificationMethod: "email_link" | "email_code" | "both"
        verificationCodeLength: number
        verificationCodeExpiry: number  // Minutes
        allowPlusAddressing: boolean    // user+tag@domain.com
        allowDisposableEmails: boolean
        blockedEmailDomains: string[]
        allowedEmailDomains?: string[]  // Whitelist mode
      }

      phone: {
        enabled: boolean
        requiresVerification: boolean
        verificationMethod: "sms_code" | "voice_call" | "whatsapp"
        verificationCodeLength: number
        verificationCodeExpiry: number
        allowedCountryCodes: string[]   // ["+1", "+44", "+91"]
        blockedCountryCodes: string[]
        requirePhoneVerification: boolean
      }

      username: {
        enabled: boolean
        minLength: number
        maxLength: number
        allowedCharacters: string       // Regex pattern
        reservedUsernames: string[]
        caseSensitive: boolean
        uniqueAcrossSystem: boolean
      }

      social: {
        enabled: boolean
        providers: {
          google: {
            enabled: boolean
            clientId: string
            clientSecret: string        // Encrypted
            scopes: string[]
            autoLinkAccount: boolean    // Link to existing email
            requireEmailVerification: boolean
          }
          facebook: {
            enabled: boolean
            appId: string
            appSecret: string           // Encrypted
            scopes: string[]
            autoLinkAccount: boolean
          }
          github: {
            enabled: boolean
            clientId: string
            clientSecret: string
            scopes: string[]
            autoLinkAccount: boolean
          }
          linkedin: {
            enabled: boolean
            clientId: string
            clientSecret: string
            scopes: string[]
          }
          apple: {
            enabled: boolean
            teamId: string
            keyId: string
            privateKey: string          // Encrypted
            autoLinkAccount: boolean
          }
          microsoft: {
            enabled: boolean
            tenantId: string
            clientId: string
            clientSecret: string
          }
          twitter: {
            enabled: boolean
            apiKey: string
            apiSecret: string
          }
        }
      }

      sso: {
        enabled: boolean
        providers: {
          saml: {
            enabled: boolean
            entityId: string
            ssoUrl: string
            x509Certificate: string
            allowedDomains: string[]
          }
          oidc: {
            enabled: boolean
            issuer: string
            clientId: string
            clientSecret: string
            scopes: string[]
          }
          ldap: {
            enabled: boolean
            server: string
            port: number
            baseDn: string
            bindDn: string
            bindPassword: string
            searchFilter: string
          }
        }
      }

      passwordless: {
        enabled: boolean
        methods: {
          magicLink: {
            enabled: boolean
            linkExpiry: number          // Minutes
            allowMultipleLinks: boolean
            ipRestriction: boolean
          }
          otp: {
            enabled: boolean
            codeLength: number
            codeExpiry: number
            deliveryMethod: ("email" | "sms")[]
          }
          webauthn: {
            enabled: boolean
            requireResidentKey: boolean
            userVerification: "required" | "preferred" | "discouraged"
          }
        }
      }

      invite: {
        enabled: boolean
        requiresInvite: boolean         // Invite-only registration
        inviteExpiry: number            // Days
        maxInvitesPerUser: number
        allowInviteRoleSelection: boolean
        inviteCodeLength: number
        customInviteMessage: boolean
      }
    }

    // Required fields
    requiredFields: {
      email: boolean
      phone: boolean
      username: boolean
      firstName: boolean
      lastName: boolean
      fullName: boolean
      dateOfBirth: boolean
      gender: boolean
      address: boolean
      company: boolean
      jobTitle: boolean
      website: boolean
      bio: boolean
      avatar: boolean
      customFields: {
        fieldId: string
        fieldName: string
        fieldType: "text" | "number" | "select" | "multiselect" | "date" | "boolean"
        required: boolean
        options?: string[]
        validation?: string             // Regex pattern
      }[]
    }

    // Password policy
    passwordPolicy: {
      minLength: number
      maxLength: number
      requireUppercase: boolean
      requireLowercase: boolean
      requireNumbers: boolean
      requireSpecialChars: boolean
      specialCharsSet: string
      preventCommonPasswords: boolean
      preventUserInfoInPassword: boolean  // Name, email, etc.
      preventPreviousPasswords: number    // Last N passwords
      passwordStrengthMeter: boolean
      minStrengthScore: number            // 0-4 (zxcvbn)
      expiryDays?: number                 // Force password change
    }

    // Terms & Conditions
    termsAndConditions: {
      required: boolean
      version: string
      url: string
      requireExplicitConsent: boolean
      trackConsentVersion: boolean
    }

    privacyPolicy: {
      required: boolean
      version: string
      url: string
      requireExplicitConsent: boolean
    }

    marketingConsent: {
      enabled: boolean
      required: boolean
      defaultValue: boolean
      granular: {
        email: boolean
        sms: boolean
        push: boolean
        thirdParty: boolean
      }
    }

    // CAPTCHA
    captcha: {
      enabled: boolean
      provider: "recaptcha_v2" | "recaptcha_v3" | "hcaptcha" | "turnstile"
      siteKey: string
      secretKey: string
      scoreThreshold?: number             // For invisible captchas
      showOnAttempt: number               // Show after N failed attempts
    }

    // Rate limiting
    rateLimiting: {
      enabled: boolean
      maxAttemptsPerMinute: number
      maxAttemptsPerHour: number
      maxAttemptsPerDay: number
      lockoutDuration: number             // Minutes
      progressiveLockout: boolean
    }

    // Geographic restrictions
    geoRestrictions: {
      enabled: boolean
      allowedCountries?: string[]
      blockedCountries?: string[]
      blockVpn: boolean
      blockProxy: boolean
      blockTor: boolean
      requireCountryMatch: boolean        // IP matches selected country
    }

    // Device restrictions
    deviceRestrictions: {
      enabled: boolean
      allowedPlatforms: ("web" | "ios" | "android" | "desktop")[]
      deviceFingerprinting: boolean
      maxDevicesPerUser?: number
    }

    // Age restrictions
    ageRestrictions: {
      enabled: boolean
      minimumAge: number
      requireDateOfBirth: boolean
      verifyAge: boolean
      verificationMethod: "document" | "credit_card" | "third_party"
    }

    // Email domain validation
    emailDomainValidation: {
      enabled: boolean
      checkMxRecords: boolean
      checkSmtpValidity: boolean
      blockDisposable: boolean
      blockFreeProviders: boolean
      allowedDomains?: string[]           // Whitelist
      blockedDomains: string[]            // Blacklist
    }

    // Duplicate prevention
    duplicatePrevention: {
      enabled: boolean
      checkFields: ("email" | "phone" | "username" | "device_fingerprint")[]
      allowMultipleAccounts: boolean
      maxAccountsPerEmail: number
      maxAccountsPerPhone: number
      maxAccountsPerDevice: number
    }

    // Auto-assignment
    autoAssignment: {
      defaultRole: string
      defaultTeam?: string
      defaultOrganization?: string
      defaultPermissions: string[]
      defaultSubscription?: string
    }

    // Referral tracking
    referralTracking: {
      enabled: boolean
      requireReferralCode: boolean
      trackUtmParams: boolean
      rewardReferrer: boolean
      rewardReferee: boolean
    }

    // Custom validation
    customValidation: {
      enabled: boolean
      validationRules: {
        ruleId: string
        field: string
        condition: string               // Expression
        errorMessage: string
      }[]
    }
  }

  // ============================================
  // AUTHENTICATION CONFIGURATION
  // ============================================
  authenticationConfig: {
    // Primary authentication
    primaryMethod: "password" | "passwordless" | "sso" | "biometric"

    // Multi-factor authentication
    mfa: {
      enabled: boolean
      required: boolean                 // Force MFA for all users
      requiredForRoles: string[]        // Force MFA for specific roles
      requiredAfterDays: number         // Force MFA after N days
      methods: {
        totp: {
          enabled: boolean
          issuer: string
          algorithm: "SHA1" | "SHA256" | "SHA512"
          digits: number
          period: number                // Seconds
          allowBackupCodes: boolean
          backupCodesCount: number
        }
        sms: {
          enabled: boolean
          provider: "twilio" | "sns" | "custom"
          codeLength: number
          codeExpiry: number
        }
        email: {
          enabled: boolean
          codeLength: number
          codeExpiry: number
        }
        authenticatorApp: {
          enabled: boolean
          allowedApps: ("google_authenticator" | "authy" | "microsoft_authenticator")[]
        }
        webauthn: {
          enabled: boolean
          allowPlatformAuthenticators: boolean  // Touch ID, Face ID
          allowCrossPlatformAuthenticators: boolean  // YubiKey, etc.
          userVerification: "required" | "preferred" | "discouraged"
          attestation: "none" | "indirect" | "direct"
        }
        push: {
          enabled: boolean
          provider: "firebase" | "onesignal" | "custom"
          expirySeconds: number
        }
      }
      gracePeriod: number               // Days to set up MFA
      rememberDevice: {
        enabled: boolean
        duration: number                // Days
      }
      trustedDeviceLimit: number
    }

    // Session management
    session: {
      strategy: "jwt" | "stateful" | "hybrid"

      jwt: {
        algorithm: "HS256" | "RS256"
        secret?: string
        privateKey?: string
        publicKey?: string
        issuer: string
        audience: string
        expiresIn: number               // Seconds
        refreshTokenEnabled: boolean
        refreshTokenExpiry: number
        refreshTokenRotation: boolean
      }

      stateful: {
        store: "redis" | "database" | "memory"
        prefix: string
        expiresIn: number
        slidingExpiration: boolean
        absoluteExpiration: number
      }

      cookie: {
        name: string
        httpOnly: boolean
        secure: boolean
        sameSite: "strict" | "lax" | "none"
        domain?: string
        path: string
        maxAge: number
      }

      concurrency: {
        enabled: boolean
        maxActiveSessions: number
        strategy: "block_new" | "invalidate_oldest" | "allow_all"
      }

      inactivityTimeout: {
        enabled: boolean
        timeout: number                 // Minutes
        warningBefore: number           // Minutes
        extendOnActivity: boolean
      }

      absoluteTimeout: {
        enabled: boolean
        timeout: number                 // Hours
        forceReauth: boolean
      }

      deviceTracking: {
        enabled: boolean
        trackUserAgent: boolean
        trackIpAddress: boolean
        trackLocation: boolean
        flagSuspiciousLogin: boolean
        notifyOnNewDevice: boolean
      }
    }

    // IP restrictions
    ipRestrictions: {
      enabled: boolean
      whitelist: string[]               // CIDR notation
      blacklist: string[]
      requireWhitelistForRoles: string[]
      allowDynamic: boolean
    }

    // Time-based restrictions
    timeRestrictions: {
      enabled: boolean
      allowedHours: [number, number]    // [9, 17]
      allowedDays: number[]             // [1,2,3,4,5]
      timezone: string
      exemptRoles: string[]
    }

    // Failed login handling
    failedLoginHandling: {
      enabled: boolean
      maxAttempts: number
      lockoutDuration: number           // Minutes
      progressiveLockout: {
        enabled: boolean
        multiplier: number
      }
      notifyUser: boolean
      notifyAdmin: boolean
      requireCaptchaAfter: number
      requireMfaAfter: number
      resetPasswordAfter: number
    }

    // Account recovery
    accountRecovery: {
      enabled: boolean
      methods: {
        email: {
          enabled: boolean
          tokenExpiry: number           // Minutes
          maxAttemptsPerHour: number
        }
        sms: {
          enabled: boolean
          codeExpiry: number
          maxAttemptsPerHour: number
        }
        securityQuestions: {
          enabled: boolean
          requiredQuestions: number
          availableQuestions: string[]
        }
        adminApproval: {
          enabled: boolean
          approvers: string[]           // Roles
        }
      }
      requireIdentityVerification: boolean
      cooldownPeriod: number            // Hours between recovery attempts
    }
  }

  // ============================================
  // AUTHORIZATION & RBAC
  // ============================================
  authorizationConfig: {
    // Role-Based Access Control
    rbac: {
      enabled: boolean

      roles: {
        roleId: string
        name: string
        description: string
        level: number                   // Hierarchy level
        inheritsFrom?: string[]         // Parent roles
        permissions: string[]
        isDefault: boolean
        isSystem: boolean               // Cannot be deleted
        maxUsers?: number

        conditions: {
          requiresVerification: boolean
          requiresMfa: boolean
          requiresApproval: boolean
          autoExpires: boolean
          expiryDays?: number

          assignmentRules: {
            ruleId: string
            condition: string           // Expression
            priority: number
          }[]
        }

        metadata: {
          color: string
          icon: string
          badge: string
        }
      }[]

      permissions: {
        permissionId: string
        name: string
        description: string
        resource: string                // users, posts, payments
        actions: ("create" | "read" | "update" | "delete" | "execute")[]
        scope: "own" | "team" | "organization" | "global"

        conditions: {
          timeRestricted: boolean
          ipRestricted: boolean
          resourceStateRestricted: boolean
          customConditions: string    // Expression
        }
      }[]

      // Dynamic role assignment
      dynamicAssignment: {
        enabled: boolean
        rules: {
          ruleId: string
          name: string
          condition: string             // "user.email.endsWith('@company.com')"
          assignRole: string
          priority: number
          active: boolean
        }[]
      }

      // Role hierarchies
      hierarchies: {
        enabled: boolean
        inheritPermissions: boolean
        cascadeActions: boolean
      }
    }

    // Attribute-Based Access Control
    abac: {
      enabled: boolean

      attributes: {
        user: string[]                  // age, department, location
        resource: string[]              // status, owner, sensitivity
        environment: string[]           // time, ip, device
        action: string[]                // read, write, approve
      }

      policies: {
        policyId: string
        name: string
        description: string
        effect: "allow" | "deny"
        priority: number

        rules: {
          subject: string               // User attributes
          resource: string              // Resource attributes
          action: string[]
          environment: string           // Environmental conditions
          condition: string             // Complex expression
        }[]
      }[]
    }

    // Teams & Organizations
    multiTenancy: {
      enabled: boolean
      model: "single" | "multi_tenant" | "hybrid"

      organizations: {
        enabled: boolean
        userCanCreateOrg: boolean
        requiresApproval: boolean
        maxOrgsPerUser: number
        isolation: "strict" | "relaxed"

        roles: {
          owner: string[]
          admin: string[]
          member: string[]
          guest: string[]
        }

        billing: {
          perOrganization: boolean
          perUser: boolean
        }
      }

      teams: {
        enabled: boolean
        userCanCreateTeam: boolean
        maxTeamsPerUser: number
        maxMembersPerTeam: number

        roles: {
          lead: string[]
          member: string[]
        }
      }

      workspaces: {
        enabled: boolean
        maxWorkspacesPerOrg: number
        isolation: boolean
      }
    }

    // Resource ownership
    ownership: {
      enabled: boolean
      transferOwnership: boolean
      requireApprovalForTransfer: boolean
      allowSharedOwnership: boolean
      maxCoOwners: number
    }
  }

  // ============================================
  // PROFILE MANAGEMENT
  // ============================================
  profileConfig: {
    // Profile visibility
    visibility: {
      default: "public" | "private" | "team_only" | "org_only"
      allowUserControl: boolean

      fields: {
        email: "public" | "private" | "user_choice"
        phone: "public" | "private" | "user_choice"
        fullName: "public" | "private" | "user_choice"
        avatar: "public" | "private" | "user_choice"
        bio: "public" | "private" | "user_choice"
        location: "public" | "private" | "user_choice"
        socialLinks: "public" | "private" | "user_choice"
        customFields: Record<string, "public" | "private" | "user_choice">
      }
    }

    // Profile fields
    fields: {
      // Basic fields
      firstName: {
        enabled: boolean
        required: boolean
        editable: boolean
        minLength?: number
        maxLength?: number
        validation?: string
      }

      lastName: {
        enabled: boolean
        required: boolean
        editable: boolean
        minLength?: number
        maxLength?: number
      }

      displayName: {
        enabled: boolean
        required: boolean
        editable: boolean
        uniqueRequired: boolean
      }

      email: {
        editable: boolean
        requireVerification: boolean
        allowMultiple: boolean
        maxEmails: number
      }

      phone: {
        editable: boolean
        requireVerification: boolean
        allowMultiple: boolean
        maxPhones: number
      }

      avatar: {
        enabled: boolean
        required: boolean
        uploadEnabled: boolean
        maxFileSize: number           // Bytes
        allowedFormats: string[]
        defaultAvatarStyle: "initials" | "identicon" | "gravatar" | "custom"
      }

      bio: {
        enabled: boolean
        required: boolean
        maxLength: number
        allowHtml: boolean
        allowMarkdown: boolean
      }

      dateOfBirth: {
        enabled: boolean
        required: boolean
        editable: boolean
        minimumAge: number
        displayAge: boolean
        hideYear: boolean
      }

      gender: {
        enabled: boolean
        required: boolean
        options: string[]
        allowCustom: boolean
      }

      pronouns: {
        enabled: boolean
        required: boolean
        options: string[]
        allowCustom: boolean
      }

      location: {
        enabled: boolean
        required: boolean
        type: "country" | "city" | "full_address" | "coordinates"
        autoDetect: boolean
        verifyAddress: boolean
      }

      timezone: {
        enabled: boolean
        required: boolean
        autoDetect: boolean
      }

      language: {
        enabled: boolean
        required: boolean
        supportedLanguages: string[]
        autoDetect: boolean
      }

      company: {
        enabled: boolean
        required: boolean
        autocomplete: boolean
      }

      jobTitle: {
        enabled: boolean
        required: boolean
      }

      department: {
        enabled: boolean
        required: boolean
        options?: string[]
      }

      website: {
        enabled: boolean
        required: boolean
        validate: boolean
      }

      socialLinks: {
        enabled: boolean
        platforms: {
          twitter: boolean
          linkedin: boolean
          github: boolean
          facebook: boolean
          instagram: boolean
          youtube: boolean
          custom: boolean
        }
        verifyOwnership: boolean
      }

      // Custom fields
      customFields: {
        fieldId: string
        name: string
        type: "text" | "number" | "email" | "url" | "date" | "select" | "multiselect" | "checkbox" | "textarea" | "file"
        required: boolean
        editable: boolean
        validation?: string
        options?: string[]
        defaultValue?: any
        helpText?: string
        placeholder?: string
        group?: string                // Field grouping
      }[]
    }

    // Profile completion
    completionTracking: {
      enabled: boolean
      requiredFields: string[]
      showProgressBar: boolean
      incentivizeCompletion: boolean
      completionRewards: {
        badgeId?: string
        points?: number
        unlockFeatures?: string[]
      }
    }

    // Profile updates
    updateSettings: {
      requireReauth: boolean          // Require password for sensitive changes
      requireEmailVerification: boolean
      auditLog: boolean
      moderationRequired: boolean
      moderationRoles: string[]
      cooldownPeriod: number          // Hours between updates
    }

    // Profile deletion
    deletionSettings: {
      allowSelfDelete: boolean
      requireReason: boolean
      cooldownPeriod: number          // Days before permanent deletion
      dataRetention: number           // Days to retain deleted data
      anonymizeData: boolean
      transferOwnership: boolean
      notifyConnections: boolean
    }
  }

  // ============================================
  // VERIFICATION & KYC
  // ============================================
  verificationConfig: {
    // Email verification
    emailVerification: {
      enabled: boolean
      required: boolean
      method: "link" | "code" | "both"
      linkExpiry: number              // Minutes
      codeLength: number
      codeExpiry: number
      maxResendAttempts: number
      resendCooldown: number          // Seconds
      template: string
      customDomain: boolean
      trackClicks: boolean
    }

    // Phone verification
    phoneVerification: {
      enabled: boolean
      required: boolean
      method: "sms" | "call" | "whatsapp"
      provider: "twilio" | "sns" | "vonage" | "custom"
      codeLength: number
      codeExpiry: number
      maxAttempts: number
      cooldownPeriod: number
    }

    // Identity verification (KYC)
    identityVerification: {
      enabled: boolean
      required: boolean
      requiredForRoles: string[]
      requiredForActions: string[]

      levels: {
        basic: {
          enabled: boolean
          requirements: ("email" | "phone" | "name" | "dob")[]
        }

        standard: {
          enabled: boolean
          requirements: ("email" | "phone" | "name" | "dob" | "address" | "id_document")[]
          documentTypes: ("passport" | "drivers_license" | "national_id" | "residence_permit")[]
          allowedCountries: string[]
        }

        enhanced: {
          enabled: boolean
          requirements: ("email" | "phone" | "name" | "dob" | "address" | "id_document" | "selfie" | "proof_of_address" | "biometric")[]
          liveness Detection: boolean
          facialRecognition: boolean
          backgroundCheck: boolean
        }
      }

      provider: {
        name: "stripe_identity" | "onfido" | "jumio" | "veriff" | "sumsub" | "custom"
        apiKey: string
        webhookUrl: string
        autoApprove: boolean
        manualReviewThreshold: number
      }

      retryPolicy: {
        maxAttempts: number
        cooldownDays: number
      }

      expiryPolicy: {
        enabled: boolean
        expiryDays: number
        renewalReminderDays: number
      }
    }

    // Address verification
    addressVerification: {
      enabled: boolean
      required: boolean
      method: "document" | "postcard" | "sms" | "api"
      provider?: "google_maps" | "smarty_streets" | "loqate"
      acceptedDocuments: string[]
      documentExpiry: number
    }

    // Age verification
    ageVerification: {
      enabled: boolean
      required: boolean
      minimumAge: number
      method: "dob" | "document" | "credit_card" | "third_party"
      strictMode: boolean
    }

    // Business verification
    businessVerification: {
      enabled: boolean
      required: boolean
      requirements: ("business_name" | "registration_number" | "tax_id" | "incorporation_doc" | "proof_of_address")[]
      verifyWithRegistry: boolean
      registryProvider?: string
    }

    // Social proof verification
    socialProofVerification: {
      enabled: boolean
      platforms: ("twitter" | "linkedin" | "github" | "facebook")[]
      minimumFollowers?: number
      verificationBadge: boolean
    }
  }

  // ============================================
  // ACCOUNT SECURITY
  // ============================================
  securityConfig: {
    // Password management
    passwordManagement: {
      allowChange: boolean
      requireCurrentPassword: boolean
      changeFrequency: number         // Days (0 = never expires)
      expireWarningDays: number
      preventReuse: number            // Last N passwords
      strengthRequirements: {
        minLength: number
        maxLength: number
        requireUppercase: boolean
        requireLowercase: boolean
        requireNumbers: boolean
        requireSpecialChars: boolean
        minStrengthScore: number
      }
      compromisedPasswordCheck: boolean
      provider: "haveibeenpwned" | "custom"
    }

    // Account lockout
    accountLockout: {
      enabled: boolean
      maxFailedAttempts: number
      lockoutDuration: number         // Minutes
      permanentLockAfter: number      // Failed attempts
      notifyUser: boolean
      notifyAdmin: boolean
      unlockMethods: ("time" | "admin" | "email" | "support")[]
    }

    // Suspicious activity detection
    suspiciousActivity: {
      enabled: boolean

      monitors: {
        unusualLocation: boolean
        unusualDevice: boolean
        unusualTime: boolean
        rapidPasswordChanges: boolean
        multipleFailedLogins: boolean
        dataDownload: boolean
        permissionChanges: boolean
        apiAbusePattern: boolean
      }

      actions: {
        flagAccount: boolean
        requireMfa: boolean
        lockAccount: boolean
        notifyUser: boolean
        notifyAdmin: boolean
        requireReauth: boolean
      }

      mlDetection: {
        enabled: boolean
        provider: "aws_fraud_detector" | "sift" | "custom"
        scoreThreshold: number
      }
    }

    // Login notifications
    loginNotifications: {
      enabled: boolean
      notifyOn: {
        newDevice: boolean
        newLocation: boolean
        unusualTime: boolean
        afterFailedAttempts: boolean
      }
      channels: ("email" | "sms" | "push")[]
      includeDetails: boolean
    }

    // Security questions
    securityQuestions: {
      enabled: boolean
      required: boolean
      minimumQuestions: number
      customQuestionsAllowed: boolean
      predefinedQuestions: string[]
      useForRecovery: boolean
      useForSensitiveActions: boolean
    }

    // Device management
    deviceManagement: {
      enabled: boolean
      trackDevices: boolean
      maxDevices: number
      allowRemoteLogout: boolean
      requireApprovalForNew: boolean
      notifyOnNew: boolean
      deviceFingerprinting: boolean
    }

    // API keys & tokens
    apiKeyManagement: {
      enabled: boolean
      allowUserGeneration: boolean
      maxKeysPerUser: number
      keyExpiry: number
      rateLimiting: boolean
      ipRestriction: boolean
      scopeRestriction: boolean
      auditLog: boolean
    }

    // Encryption
    encryption: {
      atRest: {
        enabled: boolean
        algorithm: "AES-256" | "ChaCha20"
        keyRotation: boolean
        rotationDays: number
      }
      inTransit: {
        enforceHttps: boolean
        tlsVersion: "1.2" | "1.3"
        hsts: boolean
        certificatePinning: boolean
      }
    }
  }

  // ============================================
  // USER LIFECYCLE
  // ============================================
  lifecycleConfig: {
    // Onboarding
    onboarding: {
      enabled: boolean
      steps: {
        stepId: string
        name: string
        description: string
        type: "profile_completion" | "tutorial" | "verification" | "preferences" | "custom"
        required: boolean
        skippable: boolean
        order: number
        conditions?: string           // When to show
      }[]
      showProgress: boolean
      incentiveCompletion: boolean
      allowLater: boolean
      timeout: number                 // Days to complete
    }

    // Account states
    accountStates: {
      states: {
        active: {
          allowLogin: boolean
          allowActions: string[]
        }
        inactive: {
          inactiveDays: number        // Days before considered inactive
          autoNotify: boolean
          notifyDaysBefore: number[]
          autoAction: "none" | "suspend" | "delete"
        }
        suspended: {
          allowLogin: boolean
          showReason: boolean
          appealProcess: boolean
          autoUnsuspendDays?: number
        }
        pending: {
          allowLogin: boolean
          expiryDays: number
          autoDelete: boolean
        }
        banned: {
          permanent: boolean
          appealProcess: boolean
          showReason: boolean
        }
        deleted: {
          softDelete: boolean
          retentionDays: number
          allowReactivation: boolean
          reactivationDays: number
        }
      }

      transitions: {
        fromState: string
        toState: string
        allowedBy: string[]           // Roles
        requiresReason: boolean
        requiresApproval: boolean
        notifyUser: boolean
        reversible: boolean
      }[]
    }

    // Inactivity handling
    inactivityHandling: {
      enabled: boolean
      inactiveDays: number
      warningDays: number[]
      actions: {
        sendReminder: boolean
        suspendAccount: boolean
        deleteAccount: boolean
        anonymizeData: boolean
      }
      exemptRoles: string[]
    }

    // Account expiry
    accountExpiry: {
      enabled: boolean
      expiryDays: number
      expiryDate?: Date
      renewalProcess: boolean
      autoRenewal: boolean
      reminderDays: number[]
      actions: {
        suspend: boolean
        delete: boolean
        downgrade: boolean
      }
    }

    // Reactivation
    reactivation: {
      allowReactivation: boolean
      withinDays: number
      requiresVerification: boolean
      requiresApproval: boolean
      restoreData: boolean
      notifyAdmin: boolean
    }

    // Account merging
    accountMerging: {
      enabled: boolean
      allowUserInitiated: boolean
      requiresApproval: boolean
      mergeStrategy: "keep_primary" | "merge_all" | "custom"
      transferData: boolean
      transferSubscriptions: boolean
    }
  }

  // ============================================
  // PRIVACY & COMPLIANCE
  // ============================================
  privacyConfig: {
    // GDPR compliance
    gdpr: {
      enabled: boolean
      applicableRegions: string[]

      rights: {
        rightToAccess: boolean
        rightToRectification: boolean
        rightToErasure: boolean
        rightToRestriction: boolean
        rightToPortability: boolean
        rightToObject: boolean
      }

      dataPortability: {
        enabled: boolean
        formats: ("json" | "csv" | "xml")[]
        includeFiles: boolean
        processingTime: number        // Hours
      }

      consent: {
        explicit: boolean
        granular: boolean
        withdrawable: boolean
        trackVersion: boolean
        reconfirmDays?: number
      }

      dataRetention: {
        defaultDays: number
        customRetention: Record<string, number>
        autoDelete: boolean
      }

      dpo: {
        name: string
        email: string
        phone: string
      }
    }

    // CCPA compliance
    ccpa: {
      enabled: boolean
      applicableStates: string[]

      rights: {
        rightToKnow: boolean
        rightToDelete: boolean
        rightToOptOut: boolean
      }

      doNotSell: {
        enabled: boolean
        defaultValue: boolean
        showLink: boolean
      }
    }

    // Data minimization
    dataMinimization: {
      enabled: boolean
      collectOnlyNecessary: boolean
      autoDeleteUnused: boolean
      unusedDataDays: number
    }

    // Audit logging
    auditLogging: {
      enabled: boolean
      logActions: ("login" | "logout" | "profile_update" | "password_change" | "permission_change" | "data_export" | "data_delete")[]
      retentionDays: number
      immutable: boolean
      userAccess: boolean
    }

    // Cookie policy
    cookiePolicy: {
      enabled: boolean
      categories: {
        necessary: boolean
        functional: boolean
        analytics: boolean
        marketing: boolean
      }
      requireConsent: boolean
      granularControl: boolean
      cookieBanner: boolean
    }
  }

  // ============================================
  // NOTIFICATIONS & COMMUNICATIONS
  // ============================================
  notificationConfig: {
    // Channel preferences
    channels: {
      email: {
        enabled: boolean
        provider: "sendgrid" | "ses" | "mailgun" | "postmark" | "custom"
        fromAddress: string
        fromName: string
        replyTo: string
        templates: Record<string, string>
      }

      sms: {
        enabled: boolean
        provider: "twilio" | "sns" | "vonage"
        senderId: string
      }

      push: {
        enabled: boolean
        provider: "firebase" | "onesignal" | "pusher"
        webPush: boolean
        mobilePush: boolean
      }

      inApp: {
        enabled: boolean
        realtime: boolean
        retention: number             // Days
      }

      webhook: {
        enabled: boolean
        url: string
        events: string[]
        retryPolicy: {
          maxRetries: number
          backoff: "linear" | "exponential"
        }
      }
    }

    // Notification types
    types: {
      welcome: {
        enabled: boolean
        channels: ("email" | "sms" | "push")[]
        delay: number                 // Minutes
        template: string
      }

      emailVerification: {
        enabled: boolean
        channels: ["email"]
        template: string
      }

      passwordReset: {
        enabled: boolean
        channels: ("email" | "sms")[]
        template: string
      }

      loginAlert: {
        enabled: boolean
        channels: ("email" | "sms" | "push")[]
        template: string
      }

      securityAlert: {
        enabled: boolean
        channels: ("email" | "sms" | "push")[]
        template: string
      }

      accountSuspended: {
        enabled: boolean
        channels: ("email" | "sms")[]
        template: string
      }

      inactivityReminder: {
        enabled: boolean
        channels: ("email" | "push")[]
        template: string
      }

      profileUpdated: {
        enabled: boolean
        channels: ("email" | "push")[]
        template: string
      }

      roleChanged: {
        enabled: boolean
        channels: ("email" | "push")[]
        template: string
      }

      teamInvite: {
        enabled: boolean
        channels: ("email" | "push")[]
        template: string
      }
    }

    // User preferences
    userPreferences: {
      allowOptOut: boolean
      granularControl: boolean
      defaultSettings: Record<string, boolean>
      unsubscribeLink: boolean
      frequencyCapping: {
        enabled: boolean
        maxPerDay: number
        maxPerWeek: number
      }
    }

    // Digest notifications
    digest: {
      enabled: boolean
      frequency: "daily" | "weekly" | "monthly"
      time: string                    // HH:MM
      timezone: string
      includeTypes: string[]
    }
  }

  // ============================================
  // INTEGRATIONS
  // ============================================
  integrationConfig: {
    // CRM integration
    crm: {
      enabled: boolean
      provider: "salesforce" | "hubspot" | "pipedrive" | "custom"
      syncDirection: "one_way" | "two_way"
      syncFields: string[]
      syncFrequency: "realtime" | "hourly" | "daily"
      createContactOn: ("registration" | "verification" | "purchase")[]
    }

    // Analytics
    analytics: {
      enabled: boolean
      providers: {
        googleAnalytics: {
          enabled: boolean
          trackingId: string
          anonymizeIp: boolean
        }
        mixpanel: {
          enabled: boolean
          projectToken: string
        }
        segment: {
          enabled: boolean
          writeKey: string
        }
        amplitude: {
          enabled: boolean
          apiKey: string
        }
      }
      trackEvents: string[]
      trackUserProperties: boolean
    }

    // Customer support
    customerSupport: {
      enabled: boolean
      provider: "intercom" | "zendesk" | "freshdesk" | "custom"
      syncUserData: boolean
      showChat: boolean
      autoCreateTicket: boolean
    }

    // Marketing automation
    marketingAutomation: {
      enabled: boolean
      provider: "mailchimp" | "klaviyo" | "sendgrid" | "custom"
      syncLists: boolean
      automations: {
        automationId: string
        trigger: string
        listId: string
      }[]
    }

    // Webhooks
    webhooks: {
      enabled: boolean
      endpoints: {
        url: string
        events: string[]
        secret: string
        headers: Record<string, string>
        retryPolicy: {
          maxRetries: number
          timeout: number
        }
      }[]
    }
  }

  // ============================================
  // METADATA & CUSTOMIZATION
  // ============================================
  metadata: {
    displayName: string
    description: string
    icon: string
    color: string
    category: string
    tags: string[]
    customFields: Record<string, any>
    translations: Record<string, Record<string, string>>
  }

  // ============================================
  // AUDIT & VERSIONING
  // ============================================
  audit: {
    createdAt: Date
    createdBy: string
    updatedAt: Date
    updatedBy: string
    version: number
    changelog: {
      timestamp: Date
      userId: string
      action: string
      changes: Record<string, { old: any; new: any }>
    }[]
  }

  // ============================================
  // STATE & TESTING
  // ============================================
  state: "draft" | "active" | "inactive" | "deprecated"
  testMode: {
    enabled: boolean
    testUsers: string[]
    mockProviders: boolean
  }
}
```

---

## User Lifecycle Stages

### Stage 1: Anonymous Visitor
```typescript
{
  stage: "anonymous",
  availableActions: [
    "view_public_content",
    "register",
    "login"
  ],
  tracking: {
    sessionId: "generated",
    analytics: "anonymous",
    cookies: "essential_only"
  }
}
```

### Stage 2: Registration
```typescript
{
  stage: "registration",
  substages: [
    "form_entry",
    "validation",
    "captcha",
    "terms_acceptance",
    "account_creation"
  ],
  dataCollected: [
    "email",
    "password",
    "name",
    "consent"
  ]
}
```

### Stage 3: Pending Verification
```typescript
{
  stage: "pending_verification",
  status: "registered_unverified",
  allowedActions: [
    "verify_email",
    "resend_verification",
    "limited_access"
  ],
  restrictions: {
    maxActions: 5,
    limitedFeatures: true,
    expiryDays: 7
  }
}
```

### Stage 4: Verified User
```typescript
{
  stage: "verified",
  status: "active",
  allowedActions: [
    "full_access",
    "profile_edit",
    "content_creation"
  ],
  unlocked: [
    "all_features",
    "api_access",
    "team_creation"
  ]
}
```

### Stage 5: Onboarding
```typescript
{
  stage: "onboarding",
  steps: [
    { id: "1", name: "Profile Setup", completed: false },
    { id: "2", name: "Preferences", completed: false },
    { id: "3", name: "Tutorial", completed: false }
  ],
  progress: 0,
  incentives: {
    badgeOnCompletion: true,
    featureUnlock: ["advanced_search"]
  }
}
```

### Stage 6: Active User
```typescript
{
  stage: "active",
  status: "engaged",
  metrics: {
    lastLoginAt: Date,
    loginCount: number,
    actionsPerformed: number,
    engagementScore: number
  },
  benefits: [
    "full_features",
    "priority_support",
    "beta_access"
  ]
}
```

### Stage 7: Inactive User
```typescript
{
  stage: "inactive",
  inactiveDays: 30,
  reengagement: {
    emailsSent: 2,
    nextReminderAt: Date,
    incentives: ["discount", "new_features"]
  },
  risk: "medium"
}
```

### Stage 8: Suspended User
```typescript
{
  stage: "suspended",
  reason: "policy_violation",
  suspendedAt: Date,
  suspendedBy: "admin_user_id",
  duration: 7,                    // Days
  appeal: {
    allowed: true,
    submitted: false,
    deadline: Date
  }
}
```

### Stage 9: Offboarding
```typescript
{
  stage: "offboarding",
  deletionRequested: true,
  deletionDate: Date,             // 30 days from request
  dataExport: {
    requested: true,
    generated: false,
    downloadUrl: null
  },
  retentionPeriod: 30
}
```

### Stage 10: Deleted User
```typescript
{
  stage: "deleted",
  deletedAt: Date,
  method: "soft_delete",
  dataRetention: {
    anonymized: true,
    retainedUntil: Date,
    reactivatable: true,
    reactivationDeadline: Date
  }
}
```

---

## Registration & Onboarding Scenarios

### Scenario 1: Simple Email/Password Registration
```typescript
{
  id: "email-password-registration",
  name: "Email & Password Registration",
  category: "registration",
  type: "self_service",
  enabled: true,

  registrationConfig: {
    methods: {
      email: {
        enabled: true,
        requiresVerification: true,
        verificationMethod: "email_link",
        verificationCodeExpiry: 60,
        allowDisposableEmails: false,
        blockedEmailDomains: ["tempmail.com", "throwaway.email"]
      },

      username: {
        enabled: false
      },

      social: {
        enabled: false
      }
    },

    requiredFields: {
      email: true,
      firstName: true,
      lastName: true,
      dateOfBirth: false,
      phone: false
    },

    passwordPolicy: {
      minLength: 8,
      maxLength: 128,
      requireUppercase: true,
      requireLowercase: true,
      requireNumbers: true,
      requireSpecialChars: true,
      preventCommonPasswords: true,
      minStrengthScore: 3
    },

    termsAndConditions: {
      required: true,
      version: "1.0.0",
      url: "/terms",
      requireExplicitConsent: true
    },

    captcha: {
      enabled: true,
      provider: "recaptcha_v3",
      scoreThreshold: 0.5
    },

    rateLimiting: {
      enabled: true,
      maxAttemptsPerHour: 5,
      lockoutDuration: 60
    },

    autoAssignment: {
      defaultRole: "user",
      defaultPermissions: ["read_content", "create_post"]
    }
  }
}
```

### Scenario 2: Social Authentication (OAuth)
```typescript
{
  id: "social-registration",
  name: "Social Login Registration",
  category: "registration",
  type: "self_service",
  enabled: true,

  registrationConfig: {
    methods: {
      email: {
        enabled: false
      },

      social: {
        enabled: true,
        providers: {
          google: {
            enabled: true,
            clientId: process.env.GOOGLE_CLIENT_ID,
            clientSecret: process.env.GOOGLE_CLIENT_SECRET,
            scopes: ["email", "profile"],
            autoLinkAccount: true,
            requireEmailVerification: false
          },

          github: {
            enabled: true,
            clientId: process.env.GITHUB_CLIENT_ID,
            clientSecret: process.env.GITHUB_CLIENT_SECRET,
            scopes: ["user:email"],
            autoLinkAccount: true
          },

          apple: {
            enabled: true,
            teamId: process.env.APPLE_TEAM_ID,
            keyId: process.env.APPLE_KEY_ID,
            privateKey: process.env.APPLE_PRIVATE_KEY,
            autoLinkAccount: false      // Apple users prefer privacy
          }
        }
      }
    },

    requiredFields: {
      email: true,                      // From OAuth provider
      firstName: true,                  // From OAuth provider
      lastName: true                    // From OAuth provider
    },

    captcha: {
      enabled: false                    // OAuth providers handle bot detection
    },

    autoAssignment: {
      defaultRole: "user",
      defaultPermissions: ["read_content", "create_post"]
    }
  },

  lifecycleConfig: {
    onboarding: {
      enabled: true,
      steps: [
        {
          stepId: "preferences",
          name: "Set Your Preferences",
          type: "preferences",
          required: false,
          skippable: true,
          order: 1
        }
      ]
    }
  }
}
```

### Scenario 3: Enterprise SSO (SAML)
```typescript
{
  id: "enterprise-sso-saml",
  name: "Enterprise SSO via SAML",
  category: "registration",
  type: "sso",
  enabled: true,

  registrationConfig: {
    methods: {
      sso: {
        enabled: true,
        providers: {
          saml: {
            enabled: true,
            entityId: "https://app.company.com",
            ssoUrl: "https://idp.company.com/sso",
            x509Certificate: process.env.SAML_CERT,
            allowedDomains: ["company.com", "subsidiary.com"]
          }
        }
      }
    },

    requiredFields: {
      email: true,
      firstName: true,
      lastName: true,
      department: true,
      jobTitle: true
    },

    autoAssignment: {
      defaultRole: "employee",
      defaultTeam: "auto_detect_from_department",
      defaultOrganization: "company_org"
    }
  },

  authorizationConfig: {
    rbac: {
      enabled: true,
      dynamicAssignment: {
        enabled: true,
        rules: [
          {
            ruleId: "assign_admin_role",
            condition: "user.jobTitle === 'Engineering Manager'",
            assignRole: "admin",
            priority: 1,
            active: true
          },
          {
            ruleId: "assign_team_lead",
            condition: "user.jobTitle.includes('Lead')",
            assignRole: "team_lead",
            priority: 2,
            active: true
          }
        ]
      }
    }
  }
}
```

### Scenario 4: Invite-Only Registration
```typescript
{
  id: "invite-only-registration",
  name: "Invite-Only Registration",
  category: "registration",
  type: "invite",
  enabled: true,

  registrationConfig: {
    methods: {
      email: {
        enabled: true,
        requiresVerification: true,
        verificationMethod: "email_link"
      },

      invite: {
        enabled: true,
        requiresInvite: true,
        inviteExpiry: 7,
        maxInvitesPerUser: 5,
        allowInviteRoleSelection: true,
        inviteCodeLength: 8,
        customInviteMessage: true
      }
    },

    requiredFields: {
      email: true,
      firstName: true,
      lastName: true,
      inviteCode: true
    },

    passwordPolicy: {
      minLength: 10,
      requireUppercase: true,
      requireLowercase: true,
      requireNumbers: true,
      requireSpecialChars: true,
      minStrengthScore: 4
    },

    autoAssignment: {
      defaultRole: "from_invite",     // Role from invite
      defaultTeam: "from_invite",     // Team from invite
      defaultPermissions: ["read_content"]
    }
  },

  lifecycleConfig: {
    onboarding: {
      enabled: true,
      steps: [
        {
          stepId: "profile_setup",
          name: "Complete Your Profile",
          type: "profile_completion",
          required: true,
          skippable: false,
          order: 1
        },
        {
          stepId: "team_intro",
          name: "Meet Your Team",
          type: "tutorial",
          required: false,
          skippable: true,
          order: 2
        }
      ],
      timeout: 14
    }
  }
}
```

### Scenario 5: Phone Number Registration (Passwordless)
```typescript
{
  id: "phone-passwordless-registration",
  name: "Phone Number Passwordless Registration",
  category: "registration",
  type: "self_service",
  enabled: true,

  registrationConfig: {
    methods: {
      phone: {
        enabled: true,
        requiresVerification: true,
        verificationMethod: "sms_code",
        verificationCodeLength: 6,
        verificationCodeExpiry: 10,
        allowedCountryCodes: ["+1", "+44", "+91", "+86"],
        blockedCountryCodes: []
      },

      passwordless: {
        enabled: true,
        methods: {
          otp: {
            enabled: true,
            codeLength: 6,
            codeExpiry: 10,
            deliveryMethod: ["sms"]
          }
        }
      }
    },

    requiredFields: {
      phone: true,
      firstName: true,
      lastName: false
    },

    captcha: {
      enabled: true,
      provider: "recaptcha_v3"
    },

    rateLimiting: {
      enabled: true,
      maxAttemptsPerHour: 3,
      lockoutDuration: 120
    },

    autoAssignment: {
      defaultRole: "user"
    }
  },

  authenticationConfig: {
    primaryMethod: "passwordless",
    session: {
      strategy: "jwt",
      jwt: {
        expiresIn: 86400             // 24 hours
      }
    }
  }
}
```

### Scenario 6: Multi-Step B2B Registration
```typescript
{
  id: "b2b-multi-step-registration",
  name: "B2B Multi-Step Registration",
  category: "registration",
  type: "self_service",
  enabled: true,

  registrationConfig: {
    methods: {
      email: {
        enabled: true,
        requiresVerification: true,
        verificationMethod: "email_link",
        allowedEmailDomains: [],      // Corporate emails only
        blockedEmailDomains: ["gmail.com", "yahoo.com", "hotmail.com"]
      }
    },

    requiredFields: {
      email: true,
      firstName: true,
      lastName: true,
      company: true,
      jobTitle: true,
      phone: true,
      customFields: [
        {
          fieldId: "company_size",
          fieldName: "Company Size",
          fieldType: "select",
          required: true,
          options: ["1-10", "11-50", "51-200", "201-500", "500+"]
        },
        {
          fieldId: "industry",
          fieldName: "Industry",
          fieldType: "select",
          required: true,
          options: ["Technology", "Finance", "Healthcare", "Retail", "Other"]
        },
        {
          fieldId: "use_case",
          fieldName: "Primary Use Case",
          fieldType: "textarea",
          required: false
        }
      ]
    },

    termsAndConditions: {
      required: true,
      version: "2.0.0",
      url: "/terms-business"
    },

    autoAssignment: {
      defaultRole: "business_user",
      defaultPermissions: ["read_content", "create_workspace"]
    }
  },

  verificationConfig: {
    businessVerification: {
      enabled: true,
      required: false,
      requirements: ["business_name", "registration_number"],
      verifyWithRegistry: false
    }
  },

  lifecycleConfig: {
    onboarding: {
      enabled: true,
      steps: [
        {
          stepId: "workspace_setup",
          name: "Create Your Workspace",
          type: "custom",
          required: true,
          skippable: false,
          order: 1
        },
        {
          stepId: "invite_team",
          name: "Invite Team Members",
          type: "custom",
          required: false,
          skippable: true,
          order: 2
        },
        {
          stepId: "integration_setup",
          name: "Connect Your Tools",
          type: "custom",
          required: false,
          skippable: true,
          order: 3
        }
      ],
      showProgress: true,
      incentiveCompletion: true,
      timeout: 30
    }
  }
}
```

---

## Authentication Scenarios

### Scenario 1: Standard Username/Password Auth
```typescript
{
  id: "standard-password-auth",
  name: "Username/Password Authentication",
  category: "authentication",
  type: "password",
  enabled: true,

  authenticationConfig: {
    primaryMethod: "password",

    session: {
      strategy: "jwt",
      jwt: {
        algorithm: "RS256",
        privateKey: process.env.JWT_PRIVATE_KEY,
        publicKey: process.env.JWT_PUBLIC_KEY,
        issuer: "app.company.com",
        audience: "company_users",
        expiresIn: 3600,              // 1 hour
        refreshTokenEnabled: true,
        refreshTokenExpiry: 604800,   // 7 days
        refreshTokenRotation: true
      },

      cookie: {
        name: "auth_token",
        httpOnly: true,
        secure: true,
        sameSite: "strict",
        maxAge: 3600
      },

      concurrency: {
        enabled: true,
        maxActiveSessions: 3,
        strategy: "invalidate_oldest"
      },

      inactivityTimeout: {
        enabled: true,
        timeout: 30,
        warningBefore: 5,
        extendOnActivity: true
      }
    },

    failedLoginHandling: {
      enabled: true,
      maxAttempts: 5,
      lockoutDuration: 15,
      progressiveLockout: {
        enabled: true,
        multiplier: 2
      },
      notifyUser: true,
      requireCaptchaAfter: 3
    },

    accountRecovery: {
      enabled: true,
      methods: {
        email: {
          enabled: true,
          tokenExpiry: 60,
          maxAttemptsPerHour: 3
        },
        securityQuestions: {
          enabled: true,
          requiredQuestions: 2
        }
      }
    }
  },

  securityConfig: {
    loginNotifications: {
      enabled: true,
      notifyOn: {
        newDevice: true,
        newLocation: true
      },
      channels: ["email"]
    }
  }
}
```

### Scenario 2: MFA with TOTP
```typescript
{
  id: "mfa-totp-auth",
  name: "Multi-Factor Authentication (TOTP)",
  category: "authentication",
  type: "password",
  enabled: true,

  authenticationConfig: {
    primaryMethod: "password",

    mfa: {
      enabled: true,
      required: true,
      requiredForRoles: ["admin", "finance", "developer"],

      methods: {
        totp: {
          enabled: true,
          issuer: "CompanyApp",
          algorithm: "SHA256",
          digits: 6,
          period: 30,
          allowBackupCodes: true,
          backupCodesCount: 10
        },

        authenticatorApp: {
          enabled: true,
          allowedApps: ["google_authenticator", "authy", "microsoft_authenticator"]
        }
      },

      gracePeriod: 7,

      rememberDevice: {
        enabled: true,
        duration: 30
      },

      trustedDeviceLimit: 5
    },

    session: {
      strategy: "jwt",
      jwt: {
        expiresIn: 3600
      }
    }
  }
}
```

### Scenario 3: Passwordless Magic Link
```typescript
{
  id: "magic-link-auth",
  name: "Passwordless Magic Link Authentication",
  category: "authentication",
  type: "passwordless",
  enabled: true,

  authenticationConfig: {
    primaryMethod: "passwordless",

    mfa: {
      enabled: false
    },

    session: {
      strategy: "jwt",
      jwt: {
        expiresIn: 86400            // 24 hours
      }
    }
  },

  registrationConfig: {
    methods: {
      passwordless: {
        enabled: true,
        methods: {
          magicLink: {
            enabled: true,
            linkExpiry: 15,
            allowMultipleLinks: false,
            ipRestriction: true
          }
        }
      }
    }
  }
}
```

### Scenario 4: WebAuthn/Biometric
```typescript
{
  id: "webauthn-biometric-auth",
  name: "WebAuthn Biometric Authentication",
  category: "authentication",
  type: "biometric",
  enabled: true,

  authenticationConfig: {
    primaryMethod: "biometric",

    mfa: {
      enabled: false,               // Biometric IS the MFA
      methods: {
        webauthn: {
          enabled: true,
          allowPlatformAuthenticators: true,   // Touch ID, Face ID
          allowCrossPlatformAuthenticators: true,  // YubiKey
          userVerification: "required",
          attestation: "direct"
        }
      }
    },

    session: {
      strategy: "jwt",
      jwt: {
        expiresIn: 3600
      }
    }
  },

  registrationConfig: {
    methods: {
      passwordless: {
        enabled: true,
        methods: {
          webauthn: {
            enabled: true,
            requireResidentKey: true,
            userVerification: "required"
          }
        }
      }
    }
  }
}
```

---

## Authorization & RBAC

### Complete RBAC Example
```typescript
{
  id: "enterprise-rbac",
  name: "Enterprise Role-Based Access Control",
  category: "authorization",
  type: "rbac",
  enabled: true,

  authorizationConfig: {
    rbac: {
      enabled: true,

      roles: [
        {
          roleId: "super_admin",
          name: "Super Administrator",
          description: "Full system access",
          level: 1,
          inheritsFrom: [],
          permissions: ["*"],         // All permissions
          isDefault: false,
          isSystem: true,
          maxUsers: 3,

          conditions: {
            requiresVerification: true,
            requiresMfa: true,
            requiresApproval: true,
            autoExpires: false
          },

          metadata: {
            color: "#FF0000",
            icon: "crown",
            badge: "SUPER_ADMIN"
          }
        },

        {
          roleId: "admin",
          name: "Administrator",
          description: "Administrative access",
          level: 2,
          inheritsFrom: ["moderator"],
          permissions: [
            "users.create",
            "users.read",
            "users.update",
            "users.delete",
            "roles.create",
            "roles.read",
            "roles.update",
            "settings.update"
          ],
          isDefault: false,
          isSystem: false,

          conditions: {
            requiresVerification: true,
            requiresMfa: true,
            requiresApproval: true,
            autoExpires: false
          },

          metadata: {
            color: "#FFA500",
            icon: "shield",
            badge: "ADMIN"
          }
        },

        {
          roleId: "moderator",
          name: "Moderator",
          description: "Content moderation access",
          level: 3,
          inheritsFrom: ["user"],
          permissions: [
            "content.moderate",
            "users.read",
            "users.suspend",
            "reports.read",
            "reports.resolve"
          ],
          isDefault: false,
          isSystem: false,

          conditions: {
            requiresVerification: true,
            requiresMfa: false,
            autoExpires: false
          }
        },

        {
          roleId: "premium_user",
          name: "Premium User",
          description: "Premium features access",
          level: 4,
          inheritsFrom: ["user"],
          permissions: [
            "content.create",
            "content.read",
            "content.update_own",
            "content.delete_own",
            "features.premium",
            "api.access",
            "export.data"
          ],
          isDefault: false,
          isSystem: false,

          conditions: {
            autoExpires: true,
            expiryDays: 365,

            assignmentRules: [
              {
                ruleId: "has_premium_subscription",
                condition: "user.subscription.tier === 'premium'",
                priority: 1
              }
            ]
          }
        },

        {
          roleId: "user",
          name: "User",
          description: "Standard user access",
          level: 5,
          inheritsFrom: [],
          permissions: [
            "content.create",
            "content.read",
            "content.update_own",
            "content.delete_own",
            "profile.update_own"
          ],
          isDefault: true,
          isSystem: true,

          conditions: {
            requiresVerification: false,
            requiresMfa: false
          }
        },

        {
          roleId: "guest",
          name: "Guest",
          description: "Limited guest access",
          level: 6,
          inheritsFrom: [],
          permissions: [
            "content.read_public"
          ],
          isDefault: false,
          isSystem: true
        }
      ],

      permissions: [
        {
          permissionId: "users.create",
          name: "Create Users",
          description: "Create new user accounts",
          resource: "users",
          actions: ["create"],
          scope: "global"
        },
        {
          permissionId: "users.read",
          name: "Read Users",
          description: "View user information",
          resource: "users",
          actions: ["read"],
          scope: "global"
        },
        {
          permissionId: "users.update",
          name: "Update Users",
          description: "Edit user information",
          resource: "users",
          actions: ["update"],
          scope: "global"
        },
        {
          permissionId: "users.delete",
          name: "Delete Users",
          description: "Delete user accounts",
          resource: "users",
          actions: ["delete"],
          scope: "global",

          conditions: {
            customConditions: "user.role.level < target.role.level"
          }
        },
        {
          permissionId: "content.moderate",
          name: "Moderate Content",
          description: "Moderate user-generated content",
          resource: "content",
          actions: ["update", "delete"],
          scope: "global"
        }
      ],

      dynamicAssignment: {
        enabled: true,
        rules: [
          {
            ruleId: "auto_premium_role",
            name: "Auto-assign Premium Role",
            condition: "user.subscription.tier === 'premium' && user.subscription.status === 'active'",
            assignRole: "premium_user",
            priority: 1,
            active: true
          },
          {
            ruleId: "auto_remove_premium_role",
            name: "Auto-remove Premium Role on Cancellation",
            condition: "user.subscription.status === 'cancelled'",
            assignRole: "user",
            priority: 2,
            active: true
          }
        ]
      },

      hierarchies: {
        enabled: true,
        inheritPermissions: true,
        cascadeActions: true
      }
    },

    multiTenancy: {
      enabled: true,
      model: "multi_tenant",

      organizations: {
        enabled: true,
        userCanCreateOrg: true,
        requiresApproval: false,
        maxOrgsPerUser: 3,
        isolation: "strict",

        roles: {
          owner: ["org.delete", "org.update", "members.manage", "billing.manage"],
          admin: ["members.manage", "settings.update"],
          member: ["content.create", "content.read"],
          guest: ["content.read"]
        },

        billing: {
          perOrganization: true,
          perUser: false
        }
      },

      teams: {
        enabled: true,
        userCanCreateTeam: true,
        maxTeamsPerUser: 10,
        maxMembersPerTeam: 50,

        roles: {
          lead: ["team.manage", "members.add", "content.moderate"],
          member: ["content.create", "content.read"]
        }
      }
    }
  }
}
```

---

## Profile Management

### Complete Profile Configuration
```typescript
{
  id: "comprehensive-profile-management",
  name: "Comprehensive Profile Management",
  category: "profile",
  type: "self_service",
  enabled: true,

  profileConfig: {
    visibility: {
      default: "public",
      allowUserControl: true,

      fields: {
        email: "user_choice",
        phone: "private",
        fullName: "public",
        avatar: "public",
        bio: "public",
        location: "user_choice",
        socialLinks: "public"
      }
    },

    fields: {
      firstName: {
        enabled: true,
        required: true,
        editable: true,
        minLength: 1,
        maxLength: 50
      },

      lastName: {
        enabled: true,
        required: true,
        editable: true,
        minLength: 1,
        maxLength: 50
      },

      displayName: {
        enabled: true,
        required: true,
        editable: true,
        uniqueRequired: true
      },

      email: {
        editable: true,
        requireVerification: true,
        allowMultiple: true,
        maxEmails: 3
      },

      phone: {
        editable: true,
        requireVerification: true,
        allowMultiple: true,
        maxPhones: 2
      },

      avatar: {
        enabled: true,
        required: false,
        uploadEnabled: true,
        maxFileSize: 5242880,         // 5MB
        allowedFormats: ["image/jpeg", "image/png", "image/webp"],
        defaultAvatarStyle: "initials"
      },

      bio: {
        enabled: true,
        required: false,
        maxLength: 500,
        allowMarkdown: true
      },

      dateOfBirth: {
        enabled: true,
        required: false,
        editable: true,
        minimumAge: 13,
        displayAge: true,
        hideYear: true
      },

      gender: {
        enabled: true,
        required: false,
        options: ["Male", "Female", "Non-binary", "Prefer not to say"],
        allowCustom: true
      },

      pronouns: {
        enabled: true,
        required: false,
        options: ["he/him", "she/her", "they/them", "ze/zir"],
        allowCustom: true
      },

      location: {
        enabled: true,
        required: false,
        type: "city",
        autoDetect: true
      },

      timezone: {
        enabled: true,
        required: true,
        autoDetect: true
      },

      language: {
        enabled: true,
        required: true,
        supportedLanguages: ["en", "es", "fr", "de", "ja", "zh"],
        autoDetect: true
      },

      website: {
        enabled: true,
        required: false,
        validate: true
      },

      socialLinks: {
        enabled: true,
        platforms: {
          twitter: true,
          linkedin: true,
          github: true,
          facebook: false,
          instagram: true
        },
        verifyOwnership: false
      },

      customFields: [
        {
          fieldId: "job_title",
          name: "Job Title",
          type: "text",
          required: false,
          editable: true
        },
        {
          fieldId: "skills",
          name: "Skills",
          type: "multiselect",
          required: false,
          editable: true,
          options: ["JavaScript", "Python", "React", "Node.js", "AWS"]
        },
        {
          fieldId: "interests",
          name: "Interests",
          type: "textarea",
          required: false,
          editable: true,
          helpText: "Tell us about your interests"
        }
      ]
    },

    completionTracking: {
      enabled: true,
      requiredFields: ["firstName", "lastName", "avatar", "bio", "location"],
      showProgressBar: true,
      incentivizeCompletion: true,
      completionRewards: {
        badgeId: "profile_complete",
        points: 100,
        unlockFeatures: ["advanced_search", "recommendations"]
      }
    },

    updateSettings: {
      requireReauth: true,
      requireEmailVerification: true,
      auditLog: true,
      moderationRequired: false,
      cooldownPeriod: 1
    },

    deletionSettings: {
      allowSelfDelete: true,
      requireReason: true,
      cooldownPeriod: 30,
      dataRetention: 90,
      anonymizeData: true,
      transferOwnership: true,
      notifyConnections: false
    }
  }
}
```

---

## Database Schema

```sql
-- ============================================
-- USERS TABLE (Core)
-- ============================================
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Basic Info
  email VARCHAR(255) UNIQUE NOT NULL,
  email_verified BOOLEAN DEFAULT false,
  email_verified_at TIMESTAMP,

  phone VARCHAR(20) UNIQUE,
  phone_verified BOOLEAN DEFAULT false,
  phone_verified_at TIMESTAMP,

  username VARCHAR(50) UNIQUE,

  -- Name
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  display_name VARCHAR(100),

  -- Password (hashed)
  password_hash TEXT,
  password_changed_at TIMESTAMP,
  password_expires_at TIMESTAMP,

  -- Profile
  avatar_url TEXT,
  bio TEXT,
  date_of_birth DATE,
  gender VARCHAR(50),
  pronouns VARCHAR(50),

  -- Location & Locale
  country VARCHAR(2),
  city VARCHAR(100),
  timezone VARCHAR(50),
  language VARCHAR(10) DEFAULT 'en',

  -- Account Status
  status VARCHAR(20) DEFAULT 'pending',
  account_state VARCHAR(20) DEFAULT 'pending_verification',

  -- Verification Levels
  email_verification_level VARCHAR(20),
  identity_verification_level VARCHAR(20),
  identity_verified_at TIMESTAMP,

  -- Security
  mfa_enabled BOOLEAN DEFAULT false,
  mfa_secret TEXT,
  backup_codes TEXT[],
  trusted_devices JSONB DEFAULT '[]',

  -- Tracking
  last_login_at TIMESTAMP,
  last_login_ip INET,
  last_login_device TEXT,
  login_count INTEGER DEFAULT 0,
  failed_login_attempts INTEGER DEFAULT 0,
  locked_until TIMESTAMP,

  -- Privacy
  profile_visibility VARCHAR(20) DEFAULT 'public',
  email_visibility VARCHAR(20) DEFAULT 'private',

  -- Metadata
  metadata JSONB DEFAULT '{}',
  preferences JSONB DEFAULT '{}',
  custom_fields JSONB DEFAULT '{}',

  -- Referral
  referred_by UUID REFERENCES users(id),
  referral_code VARCHAR(20) UNIQUE,

  -- Compliance
  gdpr_consent BOOLEAN DEFAULT false,
  gdpr_consent_at TIMESTAMP,
  marketing_consent BOOLEAN DEFAULT false,
  terms_accepted_version VARCHAR(10),
  terms_accepted_at TIMESTAMP,

  -- Lifecycle
  inactive_since TIMESTAMP,
  suspended_at TIMESTAMP,
  suspended_by UUID REFERENCES users(id),
  suspended_reason TEXT,
  deletion_requested_at TIMESTAMP,
  deleted_at TIMESTAMP,
  anonymized_at TIMESTAMP,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),

  CONSTRAINT valid_status CHECK (status IN ('pending', 'active', 'inactive', 'suspended', 'banned', 'deleted')),
  CONSTRAINT valid_account_state CHECK (account_state IN ('pending_verification', 'active', 'inactive', 'suspended', 'deleted'))
);

-- Indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_created_at ON users(created_at);
CREATE INDEX idx_users_last_login_at ON users(last_login_at);
CREATE INDEX idx_users_referral_code ON users(referral_code);

-- ============================================
-- USER EMAILS (Multiple emails per user)
-- ============================================
CREATE TABLE user_emails (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  email VARCHAR(255) NOT NULL,
  is_primary BOOLEAN DEFAULT false,
  verified BOOLEAN DEFAULT false,
  verified_at TIMESTAMP,

  verification_token VARCHAR(255),
  verification_token_expires_at TIMESTAMP,

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(user_id, email),
  CONSTRAINT one_primary_email_per_user UNIQUE (user_id, is_primary) WHERE is_primary = true
);

CREATE INDEX idx_user_emails_user_id ON user_emails(user_id);
CREATE INDEX idx_user_emails_email ON user_emails(email);

-- ============================================
-- USER PHONES (Multiple phones per user)
-- ============================================
CREATE TABLE user_phones (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  phone VARCHAR(20) NOT NULL,
  country_code VARCHAR(5),
  is_primary BOOLEAN DEFAULT false,
  verified BOOLEAN DEFAULT false,
  verified_at TIMESTAMP,

  verification_code VARCHAR(10),
  verification_code_expires_at TIMESTAMP,

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(user_id, phone),
  CONSTRAINT one_primary_phone_per_user UNIQUE (user_id, is_primary) WHERE is_primary = true
);

CREATE INDEX idx_user_phones_user_id ON user_phones(user_id);
CREATE INDEX idx_user_phones_phone ON user_phones(phone);

-- ============================================
-- ROLES
-- ============================================
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  name VARCHAR(50) UNIQUE NOT NULL,
  slug VARCHAR(50) UNIQUE NOT NULL,
  description TEXT,
  level INTEGER NOT NULL,

  inherits_from UUID[] DEFAULT '{}',
  permissions TEXT[] DEFAULT '{}',

  is_default BOOLEAN DEFAULT false,
  is_system BOOLEAN DEFAULT false,
  max_users INTEGER,

  conditions JSONB DEFAULT '{}',
  metadata JSONB DEFAULT '{}',

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_roles_slug ON roles(slug);
CREATE INDEX idx_roles_is_default ON roles(is_default);

-- ============================================
-- USER ROLES (Many-to-Many)
-- ============================================
CREATE TABLE user_roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,

  assigned_at TIMESTAMP DEFAULT NOW(),
  assigned_by UUID REFERENCES users(id),
  expires_at TIMESTAMP,

  metadata JSONB DEFAULT '{}',

  UNIQUE(user_id, role_id)
);

CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_user_roles_role_id ON user_roles(role_id);

-- ============================================
-- PERMISSIONS
-- ============================================
CREATE TABLE permissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  name VARCHAR(100) UNIQUE NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,

  resource VARCHAR(50) NOT NULL,
  actions TEXT[] NOT NULL,
  scope VARCHAR(20) DEFAULT 'own',

  conditions JSONB DEFAULT '{}',

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_scope CHECK (scope IN ('own', 'team', 'organization', 'global'))
);

CREATE INDEX idx_permissions_slug ON permissions(slug);
CREATE INDEX idx_permissions_resource ON permissions(resource);

-- ============================================
-- ROLE PERMISSIONS (Many-to-Many)
-- ============================================
CREATE TABLE role_permissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  permission_id UUID REFERENCES permissions(id) ON DELETE CASCADE,

  granted_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(role_id, permission_id)
);

CREATE INDEX idx_role_permissions_role_id ON role_permissions(role_id);
CREATE INDEX idx_role_permissions_permission_id ON role_permissions(permission_id);

-- ============================================
-- USER PERMISSIONS (Direct assignment)
-- ============================================
CREATE TABLE user_permissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  permission_id UUID REFERENCES permissions(id) ON DELETE CASCADE,

  granted_at TIMESTAMP DEFAULT NOW(),
  granted_by UUID REFERENCES users(id),
  expires_at TIMESTAMP,

  UNIQUE(user_id, permission_id)
);

CREATE INDEX idx_user_permissions_user_id ON user_permissions(user_id);
CREATE INDEX idx_user_permissions_permission_id ON user_permissions(permission_id);

-- ============================================
-- ORGANIZATIONS
-- ============================================
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  description TEXT,

  owner_id UUID REFERENCES users(id),

  logo_url TEXT,
  website TEXT,

  business_verification_status VARCHAR(20) DEFAULT 'unverified',
  business_registration_number VARCHAR(100),
  tax_id VARCHAR(100),

  settings JSONB DEFAULT '{}',
  metadata JSONB DEFAULT '{}',

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE INDEX idx_organizations_slug ON organizations(slug);
CREATE INDEX idx_organizations_owner_id ON organizations(owner_id);

-- ============================================
-- ORGANIZATION MEMBERS
-- ============================================
CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  role VARCHAR(50) DEFAULT 'member',
  joined_at TIMESTAMP DEFAULT NOW(),
  invited_by UUID REFERENCES users(id),

  UNIQUE(organization_id, user_id),
  CONSTRAINT valid_org_role CHECK (role IN ('owner', 'admin', 'member', 'guest'))
);

CREATE INDEX idx_org_members_org_id ON organization_members(organization_id);
CREATE INDEX idx_org_members_user_id ON organization_members(user_id);

-- ============================================
-- TEAMS
-- ============================================
CREATE TABLE teams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,

  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) NOT NULL,
  description TEXT,

  lead_id UUID REFERENCES users(id),

  settings JSONB DEFAULT '{}',
  metadata JSONB DEFAULT '{}',

  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(organization_id, slug)
);

CREATE INDEX idx_teams_org_id ON teams(organization_id);
CREATE INDEX idx_teams_lead_id ON teams(lead_id);

-- ============================================
-- TEAM MEMBERS
-- ============================================
CREATE TABLE team_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  role VARCHAR(50) DEFAULT 'member',
  joined_at TIMESTAMP DEFAULT NOW(),

  UNIQUE(team_id, user_id),
  CONSTRAINT valid_team_role CHECK (role IN ('lead', 'member'))
);

CREATE INDEX idx_team_members_team_id ON team_members(team_id);
CREATE INDEX idx_team_members_user_id ON team_members(user_id);

-- ============================================
-- SESSIONS
-- ============================================
CREATE TABLE sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  token VARCHAR(500) UNIQUE NOT NULL,
  refresh_token VARCHAR(500) UNIQUE,

  ip_address INET,
  user_agent TEXT,
  device_fingerprint TEXT,
  device_name TEXT,

  location JSONB,

  is_active BOOLEAN DEFAULT true,
  last_activity_at TIMESTAMP DEFAULT NOW(),

  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP NOT NULL,
  refresh_token_expires_at TIMESTAMP
);

CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_token ON sessions(token);
CREATE INDEX idx_sessions_is_active ON sessions(is_active);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);

-- ============================================
-- LOGIN HISTORY
-- ============================================
CREATE TABLE login_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  success BOOLEAN NOT NULL,
  ip_address INET,
  user_agent TEXT,
  device_fingerprint TEXT,
  location JSONB,

  failure_reason VARCHAR(255),

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_login_history_user_id ON login_history(user_id);
CREATE INDEX idx_login_history_created_at ON login_history(created_at);
CREATE INDEX idx_login_history_success ON login_history(success);

-- ============================================
-- PASSWORD RESET TOKENS
-- ============================================
CREATE TABLE password_reset_tokens (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  token VARCHAR(255) UNIQUE NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  used BOOLEAN DEFAULT false,
  used_at TIMESTAMP,

  ip_address INET,

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_password_reset_tokens_token ON password_reset_tokens(token);
CREATE INDEX idx_password_reset_tokens_user_id ON password_reset_tokens(user_id);
CREATE INDEX idx_password_reset_tokens_expires_at ON password_reset_tokens(expires_at);

-- ============================================
-- INVITATIONS
-- ============================================
CREATE TABLE invitations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  email VARCHAR(255) NOT NULL,
  code VARCHAR(20) UNIQUE NOT NULL,

  invited_by UUID REFERENCES users(id),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  team_id UUID REFERENCES teams(id) ON DELETE CASCADE,

  role_id UUID REFERENCES roles(id),
  permissions TEXT[] DEFAULT '{}',

  status VARCHAR(20) DEFAULT 'pending',
  expires_at TIMESTAMP NOT NULL,
  accepted_at TIMESTAMP,
  accepted_by UUID REFERENCES users(id),

  metadata JSONB DEFAULT '{}',

  created_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_invite_status CHECK (status IN ('pending', 'accepted', 'expired', 'cancelled'))
);

CREATE INDEX idx_invitations_code ON invitations(code);
CREATE INDEX idx_invitations_email ON invitations(email);
CREATE INDEX idx_invitations_invited_by ON invitations(invited_by);

-- ============================================
-- USER AUDIT LOG
-- ============================================
CREATE TABLE user_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  action VARCHAR(100) NOT NULL,
  resource VARCHAR(100),
  resource_id UUID,

  changes JSONB,
  metadata JSONB,

  ip_address INET,
  user_agent TEXT,

  performed_by UUID REFERENCES users(id),

  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_audit_log_user_id ON user_audit_log(user_id);
CREATE INDEX idx_user_audit_log_action ON user_audit_log(action);
CREATE INDEX idx_user_audit_log_created_at ON user_audit_log(created_at);

-- ============================================
-- USER PREFERENCES
-- ============================================
CREATE TABLE user_preferences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE UNIQUE,

  theme VARCHAR(20) DEFAULT 'system',
  language VARCHAR(10) DEFAULT 'en',
  timezone VARCHAR(50),

  email_notifications JSONB DEFAULT '{}',
  push_notifications JSONB DEFAULT '{}',
  sms_notifications JSONB DEFAULT '{}',

  privacy_settings JSONB DEFAULT '{}',
  accessibility_settings JSONB DEFAULT '{}',

  custom_preferences JSONB DEFAULT '{}',

  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_preferences_user_id ON user_preferences(user_id);

-- ============================================
-- DATA EXPORT REQUESTS
-- ============================================
CREATE TABLE data_export_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,

  format VARCHAR(10) DEFAULT 'json',
  status VARCHAR(20) DEFAULT 'pending',

  file_url TEXT,
  file_size BIGINT,
  expires_at TIMESTAMP,

  requested_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,

  CONSTRAINT valid_format CHECK (format IN ('json', 'csv', 'xml')),
  CONSTRAINT valid_status CHECK (status IN ('pending', 'processing', 'completed', 'failed', 'expired'))
);

CREATE INDEX idx_data_export_requests_user_id ON data_export_requests(user_id);
CREATE INDEX idx_data_export_requests_status ON data_export_requests(status);
```

---

*Due to output token limits, this document contains the first 50% of the comprehensive user management flow. The document covers:*

1. ✅ System Overview
2. ✅ Universal Configuration Pattern (complete TypeScript interface)
3. ✅ User Lifecycle Stages (10 stages)
4. ✅ Registration & Onboarding Scenarios (6 complete scenarios)
5. ✅ Authentication Scenarios (4 complete scenarios)
6. ✅ Authorization & RBAC (complete enterprise example)
7. ✅ Profile Management (complete configuration)
8. ✅ Database Schema (complete SQL with 20+ tables)

**The document continues with:**
- API Architecture (40+ endpoints)
- State Management (React/Zustand)
- Security & Compliance (GDPR, CCPA, SOC 2)
- Integration Examples (Auth0, Okta, Firebase)
- Admin Dashboard UI
- User Dashboard UI
- Complete working examples

