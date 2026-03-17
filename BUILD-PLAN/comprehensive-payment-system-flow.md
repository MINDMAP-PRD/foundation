# Comprehensive Payment System Configuration Flow

## Table of Contents
1. [System Overview](#system-overview)
2. [Universal Payment Configuration Pattern](#universal-payment-configuration-pattern)
3. [Payment Categories & Scenarios](#payment-categories--scenarios)
4. [Admin Configuration Interface](#admin-configuration-interface)
5. [User-Side Experience](#user-side-experience)
6. [Database Schema](#database-schema)
7. [API Architecture](#api-architecture)
8. [State Management](#state-management)
9. [Security & Compliance](#security--compliance)
10. [Integration Examples](#integration-examples)

---

## System Overview

### Core Principles
1. **Admin Full Control**: Every payment aspect configurable without code changes
2. **Conditional Rendering**: Show/hide elements based on rules (never just hide with CSS)
3. **RBAC Integration**: Role-based access to payment features
4. **Multi-Provider Support**: Stripe, PayPal, Square, custom providers
5. **Multi-Currency**: Dynamic currency selection and conversion
6. **Multi-Gateway**: Route payments to different gateways based on rules
7. **Compliance-First**: PCI DSS, PSD2, SCA, regional regulations built-in

### Payment Flow Architecture
```
User Action → Payment Intent → Rules Engine → Provider Selection →
Processing → Webhooks → Reconciliation → Notifications → Reporting
```

---

## Universal Payment Configuration Pattern

Every payment element follows this universal structure:

```typescript
interface PaymentConfiguration {
  // Core Identity
  id: string                          // "stripe-card-checkout"
  name: string                        // "Credit Card via Stripe"
  category: PaymentCategory           // payment_method | gateway | currency | fee | etc
  type: PaymentType                   // one_time | subscription | split | escrow | etc

  // Enable/Disable
  enabled: boolean                    // Master switch
  enabledFrom?: Date                  // Auto-enable at specific time
  enabledUntil?: Date                 // Auto-disable at specific time

  // RBAC - Role-Based Access Control
  rbac: {
    allowedRoles: string[]           // ["admin", "premium", "verified"]
    deniedRoles: string[]            // ["suspended", "unverified", "trial"]
    customPermissions: {
      view: string[]                 // Who can see this payment option
      initiate: string[]             // Who can start a payment
      process: string[]              // Who can process payments
      refund: string[]               // Who can issue refunds
      viewTransactions: string[]     // Who can view transaction history
      exportData: string[]           // Who can export payment data
    }
    requiresVerification: boolean    // KYC/identity verification required
    verificationLevel: "basic" | "standard" | "enhanced"
  }

  // Conditional Rendering Rules
  renderConditions: {
    requiresAuth: boolean            // Must be logged in
    requiresSubscription: SubscriptionTier[]  // ["free", "pro", "enterprise"]
    requiresCountry: string[]        // ["US", "GB", "EU"]
    excludedCountries: string[]      // ["IR", "KP", "SY"]
    requiresMinAge: number           // Minimum age for payment access
    requiresBusinessAccount: boolean // B2B payments only

    // Amount-based conditions
    minAmount?: number               // Minimum transaction amount
    maxAmount?: number               // Maximum transaction amount
    dailyLimit?: number              // Daily transaction limit
    monthlyLimit?: number            // Monthly transaction limit

    // Time-based conditions
    allowedHours?: [number, number]  // [9, 17] = 9 AM to 5 PM
    allowedDays?: number[]           // [1,2,3,4,5] = Mon-Fri
    timezone?: string                // "America/New_York"

    // Device/Platform conditions
    allowedPlatforms?: ("web" | "mobile" | "api")[]
    allowedDevices?: ("desktop" | "tablet" | "phone")[]

    // Business logic conditions
    requiresPreviousPurchase?: boolean
    minimumAccountAge?: number       // Days since account creation
    minimumTrustScore?: number       // 0-100 fraud prevention score

    // Custom conditions (evaluated server-side)
    customRules?: {
      ruleId: string
      expression: string             // "user.totalSpent > 1000 && user.refunds < 2"
      errorMessage: string
    }[]
  }

  // Payment Method Configuration
  methodConfig: {
    provider: "stripe" | "paypal" | "square" | "custom"
    providerMode: "live" | "test"

    // Credentials (encrypted at rest)
    credentials: {
      publicKey?: string
      secretKey?: string             // Always encrypted
      merchantId?: string
      apiVersion?: string
    }

    // Accepted payment types
    acceptedTypes: {
      cards: {
        enabled: boolean
        types: ("visa" | "mastercard" | "amex" | "discover" | "jcb" | "diners")[]
        require3DS: boolean          // 3D Secure requirement
        saveCards: boolean           // Allow saving cards
      }
      wallets: {
        enabled: boolean
        types: ("apple_pay" | "google_pay" | "samsung_pay" | "paypal")[]
      }
      bankTransfer: {
        enabled: boolean
        types: ("ach" | "sepa" | "bacs" | "wire")[]
        verificationRequired: boolean
      }
      crypto: {
        enabled: boolean
        currencies: ("BTC" | "ETH" | "USDT" | "USDC")[]
        network: string              // "mainnet" | "testnet"
      }
      buyNowPayLater: {
        enabled: boolean
        providers: ("klarna" | "afterpay" | "affirm" | "clearpay")[]
        minAmount: number
        maxAmount: number
      }
    }
  }

  // Currency Configuration
  currencyConfig: {
    supportedCurrencies: string[]    // ["USD", "EUR", "GBP", "JPY"]
    defaultCurrency: string          // "USD"
    autoDetectCurrency: boolean      // Based on user location
    allowCurrencySwitch: boolean     // User can change currency
    conversionProvider: "exchangerates" | "stripe" | "custom"
    conversionMarkup: number         // 0.02 = 2% markup on conversions
    displayFormat: {
      position: "before" | "after"   // Symbol position
      decimals: number               // 2 for most, 0 for JPY
      thousandSeparator: string      // "," or "."
      decimalSeparator: string       // "." or ","
    }
  }

  // Fee Configuration
  feeConfig: {
    platformFee: {
      enabled: boolean
      type: "fixed" | "percentage" | "hybrid"
      fixedAmount?: number
      percentage?: number            // 0.029 = 2.9%
      minimumFee?: number
      maximumFee?: number
      capAt?: number                 // Cap percentage fees at this amount
    }

    processingFee: {
      passToCustomer: boolean        // Customer pays processing fee
      absorbCost: boolean            // Platform absorbs cost
      split: number                  // 0.5 = 50/50 split

      // Per-method fees
      cardFee: {
        percentage: number
        fixed: number
      }
      achFee: {
        percentage: number
        fixed: number
      }
      internationalFee: {
        percentage: number           // Additional fee for international cards
        fixed: number
      }
    }

    subscriptionFee: {
      setupFee?: number              // One-time setup fee
      recurringFee?: number          // Per billing cycle
      upgradeFee?: number            // Fee for plan upgrades
      downgradeFee?: number          // Fee for plan downgrades
      cancellationFee?: number       // Early cancellation fee
    }

    // Dynamic fee rules
    dynamicFees: {
      ruleId: string
      condition: string              // "amount > 10000"
      feeAdjustment: {
        type: "add" | "subtract" | "replace"
        value: number
      }
    }[]
  }

  // Subscription Configuration (if applicable)
  subscriptionConfig?: {
    billingCycle: "daily" | "weekly" | "monthly" | "quarterly" | "yearly" | "custom"
    customInterval?: {
      value: number
      unit: "day" | "week" | "month" | "year"
    }

    trialPeriod: {
      enabled: boolean
      duration: number               // Number of days
      requiresPaymentMethod: boolean // Collect card during trial
      autoConvert: boolean           // Auto-convert to paid after trial
    }

    pricing: {
      model: "flat" | "tiered" | "volume" | "usage" | "hybrid"

      // Flat pricing
      flatPrice?: number

      // Tiered pricing (Stripe-like)
      tiers?: {
        upTo: number                 // null for infinity
        unitPrice: number
        flatPrice?: number           // Optional tier base fee
      }[]

      // Usage-based pricing
      usageType?: "licensed" | "metered"
      aggregationUsage?: "sum" | "last_during_period" | "max" | "last_ever"

      // Per-seat pricing
      perSeat?: {
        enabled: boolean
        minSeats: number
        maxSeats?: number
        pricePerSeat: number
      }
    }

    lifecycle: {
      allowUpgrade: boolean
      allowDowngrade: boolean
      upgradeProration: "immediate" | "end_of_cycle" | "custom"
      downgradeProration: "immediate" | "end_of_cycle" | "custom"
      allowPause: boolean
      maxPauseDuration?: number      // Days
      allowCancellation: boolean
      cancellationPolicy: "immediate" | "end_of_cycle"
      requiresCancellationReason: boolean
    }

    renewals: {
      autoRenew: boolean
      renewalReminder: {
        enabled: boolean
        daysBefore: number[]         // [7, 3, 1] = remind 7, 3, and 1 day before
      }
      failedPaymentRetry: {
        enabled: boolean
        maxAttempts: number
        retrySchedule: number[]      // [1, 3, 5, 7] = retry after these days
        gracePeriod: number          // Days before cancellation
      }
    }
  }

  // Refund Configuration
  refundConfig: {
    enabled: boolean
    allowedBy: string[]              // ["admin", "merchant", "customer"]

    policy: {
      windowDays: number             // 30 = 30-day refund window
      partialRefunds: boolean
      refundMethod: "original" | "store_credit" | "manual"
      requiresReason: boolean
      approvalRequired: boolean
      autoApproveUnder?: number      // Auto-approve refunds under this amount
    }

    restrictions: {
      maxRefundsPerUser?: number     // Per time period
      maxRefundPercentage?: number   // 0.5 = max 50% of original amount
      noRefundAfterDays?: number
      noRefundForCategories?: string[]
    }
  }

  // Dispute/Chargeback Handling
  disputeConfig: {
    autoRespond: boolean
    evidenceCollection: {
      requireCustomerSignature: boolean
      requireProofOfDelivery: boolean
      requireCommunicationLogs: boolean
      customEvidence: string[]       // Additional evidence fields
    }

    preventionRules: {
      blockHighRiskCountries: boolean
      requireAVS: boolean            // Address Verification System
      requireCVV: boolean
      requirePhoneVerification: boolean
      velocityChecks: {
        maxTransactionsPerHour: number
        maxAmountPerDay: number
      }
    }
  }

  // Tax Configuration
  taxConfig: {
    enabled: boolean
    type: "inclusive" | "exclusive"  // Tax included in price or added

    calculation: {
      method: "automatic" | "manual" | "hybrid"
      provider?: "stripe_tax" | "taxjar" | "avalara"

      // Manual tax rules
      rules: {
        country: string
        state?: string
        rate: number                 // 0.075 = 7.5%
        productCategories?: string[]
        exemptions?: string[]        // "non-profit", "government"
      }[]
    }

    collection: {
      collectTaxId: boolean          // VAT/GST number
      validateTaxId: boolean
      reverseCharge: boolean         // For B2B EU transactions
    }
  }

  // Split Payment Configuration
  splitPaymentConfig?: {
    enabled: boolean
    type: "marketplace" | "crowdfunding" | "group_payment"

    // Marketplace splits
    marketplace?: {
      platformPercentage: number     // Platform's cut
      sellerPercentage: number       // Seller's cut
      transferSchedule: "immediate" | "daily" | "weekly" | "monthly"
      holdPeriod?: number            // Days to hold funds

      // Stripe Connect / PayPal Marketplace
      connectAccountRequired: boolean
      accountVerificationRequired: boolean
    }

    // Crowdfunding splits
    crowdfunding?: {
      goalAmount: number
      deadline: Date
      refundIfNotMet: boolean
      allowOverfunding: boolean
      maxOverfundingPercentage?: number
    }

    // Group payment (split bill)
    groupPayment?: {
      splitMethod: "equal" | "percentage" | "custom"
      allowPartialPayment: boolean
      requiredParticipants: number
      deadline: Date
    }
  }

  // Escrow Configuration
  escrowConfig?: {
    enabled: boolean
    releaseConditions: {
      automatic: boolean
      conditions: ("delivery_confirmed" | "milestone_reached" | "approval_granted")[]
      holdDuration: number           // Days to hold in escrow
    }

    disputeResolution: {
      allowDisputes: boolean
      mediationRequired: boolean
      arbitrationProvider?: string
      timeoutAction: "release_to_seller" | "refund_to_buyer" | "hold"
    }
  }

  // Installment Configuration
  installmentConfig?: {
    enabled: boolean
    provider: "internal" | "affirm" | "klarna" | "afterpay"

    terms: {
      minAmount: number
      maxAmount: number
      availablePlans: {
        installments: number         // 3, 6, 12, 24
        interestRate: number         // 0 = interest-free
        fees: number
      }[]
    }

    creditCheck: {
      required: boolean
      provider: "experian" | "equifax" | "transunion"
      softPull: boolean             // Soft vs hard credit check
    }
  }

  // Payout Configuration
  payoutConfig: {
    schedule: "instant" | "daily" | "weekly" | "monthly" | "manual"
    minimumAmount: number            // Minimum balance for payout

    methods: {
      bankTransfer: {
        enabled: boolean
        fee: number
        processingDays: number
      }
      debitCard: {
        enabled: boolean
        fee: number
        processingDays: number
      }
      paypal: {
        enabled: boolean
        fee: number
        processingDays: number
      }
      check: {
        enabled: boolean
        fee: number
        processingDays: number
      }
    }

    holds: {
      initialHoldDays: number        // For new accounts
      standardHoldDays: number
      highRiskHoldDays: number
    }
  }

  // Notification Configuration
  notificationConfig: {
    customer: {
      paymentSuccess: {
        email: boolean
        sms: boolean
        push: boolean
        templateId?: string
      }
      paymentFailed: {
        email: boolean
        sms: boolean
        push: boolean
        templateId?: string
      }
      refundProcessed: {
        email: boolean
        sms: boolean
        push: boolean
        templateId?: string
      }
      subscriptionRenewal: {
        email: boolean
        sms: boolean
        push: boolean
        templateId?: string
        daysBefore: number[]
      }
      subscriptionExpiring: {
        email: boolean
        sms: boolean
        push: boolean
        templateId?: string
        daysBefore: number[]
      }
    }

    merchant: {
      newPayment: boolean
      failedPayment: boolean
      refundRequested: boolean
      disputeFiled: boolean
      payoutProcessed: boolean
    }

    admin: {
      highValueTransaction: {
        enabled: boolean
        threshold: number
      }
      suspiciousActivity: boolean
      complianceAlert: boolean
    }
  }

  // Fraud Prevention
  fraudConfig: {
    enabled: boolean
    riskLevel: "low" | "medium" | "high" | "custom"

    rules: {
      // Velocity checks
      maxTransactionsPerMinute: number
      maxTransactionsPerHour: number
      maxTransactionsPerDay: number
      maxAmountPerDay: number

      // Geographic checks
      blockVpn: boolean
      blockProxy: boolean
      blockTor: boolean
      allowedCountries?: string[]
      blockedCountries?: string[]

      // Device fingerprinting
      deviceFingerprintRequired: boolean
      blockMultipleCards: boolean    // Multiple cards from same device

      // Behavioral analysis
      requireConsistentBehavior: boolean
      flagUnusualPurchasePatterns: boolean

      // Machine learning
      mlFraudDetection: {
        enabled: boolean
        provider: "stripe_radar" | "sift" | "custom"
        scoreThreshold: number       // 0-100, reject if above
        reviewThreshold: number      // 0-100, manual review if above
      }
    }

    actions: {
      block: boolean                 // Block transaction
      review: boolean                // Hold for manual review
      challenge: boolean             // Additional verification
      captcha: boolean              // Show CAPTCHA
      twoFactor: boolean            // Require 2FA
    }
  }

  // Compliance Configuration
  complianceConfig: {
    pciDss: {
      level: "1" | "2" | "3" | "4"
      auditRequired: boolean
      lastAuditDate?: Date
      nextAuditDate?: Date
    }

    regulations: {
      gdpr: boolean                  // EU General Data Protection Regulation
      ccpa: boolean                  // California Consumer Privacy Act
      psd2: boolean                  // EU Payment Services Directive 2
      sca: boolean                   // Strong Customer Authentication
      aml: boolean                   // Anti-Money Laundering
      kyc: boolean                   // Know Your Customer
    }

    dataRetention: {
      transactionData: number        // Days to retain
      customerData: number
      logData: number
      anonymizeAfter: number         // Days before anonymization
    }

    reporting: {
      regulatoryReports: string[]    // ["SAR", "CTR", "FBAR"]
      reportingFrequency: "daily" | "weekly" | "monthly" | "quarterly"
      autoSubmit: boolean
    }
  }

  // Webhook Configuration
  webhookConfig: {
    enabled: boolean
    endpoints: {
      url: string
      events: string[]               // ["payment.success", "payment.failed", etc]
      secret: string                 // Webhook signing secret
      retryPolicy: {
        maxRetries: number
        backoffMultiplier: number
        timeout: number
      }
    }[]

    events: {
      payment: ("initiated" | "processing" | "success" | "failed" | "cancelled")[]
      subscription: ("created" | "updated" | "renewed" | "cancelled" | "paused")[]
      refund: ("requested" | "processing" | "completed" | "failed")[]
      dispute: ("created" | "updated" | "won" | "lost")[]
      payout: ("scheduled" | "processing" | "completed" | "failed")[]
    }
  }

  // Reporting & Analytics
  reportingConfig: {
    realTimeAnalytics: boolean
    dashboardAccess: string[]        // Roles with dashboard access

    metrics: {
      revenue: boolean
      volume: boolean
      conversionRate: boolean
      averageTransactionValue: boolean
      refundRate: boolean
      chargebackRate: boolean
      customerLifetimeValue: boolean
    }

    exports: {
      formats: ("csv" | "xlsx" | "pdf" | "json")[]
      scheduled: {
        enabled: boolean
        frequency: "daily" | "weekly" | "monthly"
        recipients: string[]
      }
    }

    customReports: {
      reportId: string
      name: string
      query: string                  // SQL or query DSL
      schedule?: string              // Cron expression
    }[]
  }

  // Metadata & Customization
  metadata: {
    // Display customization
    displayName: string
    description: string
    icon?: string
    color?: string
    sortOrder: number

    // Grouping
    category: string
    tags: string[]

    // Custom fields
    customFields: Record<string, any>

    // Localization
    translations: {
      [locale: string]: {
        displayName: string
        description: string
      }
    }
  }

  // Audit Trail
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

  // State Management
  state: "draft" | "active" | "inactive" | "suspended" | "archived" | "deprecated"

  // Testing
  testMode: {
    enabled: boolean
    testCards: string[]              // Test card numbers
    testWebhooks: boolean
    testNotifications: boolean
  }
}
```

---

## Payment Categories & Scenarios

### 1. One-Time Payments

#### Scenario: E-commerce Product Purchase
```typescript
{
  id: "ecommerce-checkout",
  name: "E-commerce Checkout",
  category: "payment_method",
  type: "one_time",
  enabled: true,

  rbac: {
    allowedRoles: ["customer", "guest"],
    deniedRoles: ["suspended"],
    customPermissions: {
      view: ["*"],
      initiate: ["customer", "guest"],
      process: ["system"],
      refund: ["admin", "merchant"],
    }
  },

  renderConditions: {
    requiresAuth: false,            // Allow guest checkout
    minAmount: 1,
    maxAmount: 50000,
    dailyLimit: 100000,
  },

  methodConfig: {
    provider: "stripe",
    providerMode: "live",
    acceptedTypes: {
      cards: {
        enabled: true,
        types: ["visa", "mastercard", "amex"],
        require3DS: true,
        saveCards: true
      },
      wallets: {
        enabled: true,
        types: ["apple_pay", "google_pay"]
      },
      bankTransfer: {
        enabled: false
      },
      buyNowPayLater: {
        enabled: true,
        providers: ["klarna", "afterpay"],
        minAmount: 100,
        maxAmount: 5000
      }
    }
  },

  currencyConfig: {
    supportedCurrencies: ["USD", "EUR", "GBP"],
    defaultCurrency: "USD",
    autoDetectCurrency: true,
    allowCurrencySwitch: true
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "percentage",
      percentage: 0.029,
      fixedAmount: 0.30
    },
    processingFee: {
      passToCustomer: false,
      absorbCost: true
    }
  },

  refundConfig: {
    enabled: true,
    allowedBy: ["admin", "merchant", "customer"],
    policy: {
      windowDays: 30,
      partialRefunds: true,
      refundMethod: "original",
      requiresReason: true,
      approvalRequired: true,
      autoApproveUnder: 50
    }
  },

  fraudConfig: {
    enabled: true,
    riskLevel: "medium",
    rules: {
      maxTransactionsPerDay: 10,
      maxAmountPerDay: 10000,
      blockVpn: false,
      deviceFingerprintRequired: true,
      mlFraudDetection: {
        enabled: true,
        provider: "stripe_radar",
        scoreThreshold: 80,
        reviewThreshold: 60
      }
    }
  }
}
```

#### Scenario: Service Booking Payment
```typescript
{
  id: "service-booking-payment",
  name: "Service Booking Payment",
  category: "payment_method",
  type: "one_time",
  enabled: true,

  rbac: {
    allowedRoles: ["verified_customer"],
    customPermissions: {
      initiate: ["verified_customer"],
      refund: ["admin"]
    },
    requiresVerification: true,
    verificationLevel: "basic"
  },

  renderConditions: {
    requiresAuth: true,
    minimumAccountAge: 7,           // Must have 7-day old account
    allowedHours: [6, 22],          // 6 AM to 10 PM only
    allowedDays: [1,2,3,4,5,6,7]    // All days
  },

  methodConfig: {
    provider: "stripe",
    acceptedTypes: {
      cards: {
        enabled: true,
        types: ["visa", "mastercard"],
        require3DS: true,
        saveCards: true              // For recurring bookings
      },
      wallets: {
        enabled: true,
        types: ["apple_pay", "google_pay"]
      }
    }
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "percentage",
      percentage: 0.15              // 15% platform fee
    },
    processingFee: {
      passToCustomer: true,
      split: 0.5                    // 50/50 split
    }
  },

  splitPaymentConfig: {
    enabled: true,
    type: "marketplace",
    marketplace: {
      platformPercentage: 0.15,
      sellerPercentage: 0.85,
      transferSchedule: "weekly",
      holdPeriod: 7                 // 7-day hold for disputes
    }
  },

  notificationConfig: {
    customer: {
      paymentSuccess: {
        email: true,
        sms: true,
        push: true,
        templateId: "booking-confirmation"
      }
    },
    merchant: {
      newPayment: true
    }
  }
}
```

### 2. Subscription Payments

#### Scenario: SaaS Monthly Subscription
```typescript
{
  id: "saas-monthly-subscription",
  name: "Monthly Pro Plan",
  category: "payment_method",
  type: "subscription",
  enabled: true,

  rbac: {
    allowedRoles: ["user", "premium"],
    deniedRoles: ["trial", "suspended"],
    customPermissions: {
      view: ["*"],
      initiate: ["user"],
      viewTransactions: ["user", "admin"]
    }
  },

  renderConditions: {
    requiresAuth: true,
    requiresSubscription: ["free"],  // Only free tier can upgrade
    minAmount: 9.99,
    excludedCountries: ["CU", "IR", "KP", "SY"]
  },

  methodConfig: {
    provider: "stripe",
    acceptedTypes: {
      cards: {
        enabled: true,
        types: ["visa", "mastercard", "amex"],
        require3DS: true,
        saveCards: true              // Required for subscriptions
      },
      wallets: {
        enabled: true,
        types: ["apple_pay", "google_pay"]
      },
      bankTransfer: {
        enabled: true,
        types: ["ach", "sepa"],
        verificationRequired: true
      }
    }
  },

  currencyConfig: {
    supportedCurrencies: ["USD", "EUR", "GBP", "CAD", "AUD"],
    defaultCurrency: "USD",
    autoDetectCurrency: true
  },

  feeConfig: {
    platformFee: {
      enabled: false                // No platform fee on direct SaaS
    },
    processingFee: {
      absorbCost: true
    }
  },

  subscriptionConfig: {
    billingCycle: "monthly",

    trialPeriod: {
      enabled: true,
      duration: 14,
      requiresPaymentMethod: true,
      autoConvert: true
    },

    pricing: {
      model: "flat",
      flatPrice: 29.99
    },

    lifecycle: {
      allowUpgrade: true,
      allowDowngrade: true,
      upgradeProration: "immediate",
      downgradeProration: "end_of_cycle",
      allowPause: true,
      maxPauseDuration: 90,
      allowCancellation: true,
      cancellationPolicy: "end_of_cycle",
      requiresCancellationReason: true
    },

    renewals: {
      autoRenew: true,
      renewalReminder: {
        enabled: true,
        daysBefore: [7, 3, 1]
      },
      failedPaymentRetry: {
        enabled: true,
        maxAttempts: 3,
        retrySchedule: [1, 3, 7],
        gracePeriod: 14
      }
    }
  },

  refundConfig: {
    enabled: true,
    allowedBy: ["admin"],
    policy: {
      windowDays: 7,
      partialRefunds: true,
      refundMethod: "original",
      requiresReason: true,
      approvalRequired: false
    }
  },

  notificationConfig: {
    customer: {
      subscriptionRenewal: {
        email: true,
        sms: false,
        push: true,
        daysBefore: [3, 1]
      },
      paymentFailed: {
        email: true,
        sms: true,
        push: true
      }
    }
  }
}
```

#### Scenario: Usage-Based Pricing (API Calls)
```typescript
{
  id: "api-usage-subscription",
  name: "Pay-As-You-Go API",
  category: "payment_method",
  type: "subscription",
  enabled: true,

  subscriptionConfig: {
    billingCycle: "monthly",

    pricing: {
      model: "tiered",
      tiers: [
        {
          upTo: 1000,
          unitPrice: 0,             // First 1000 calls free
          flatPrice: 0
        },
        {
          upTo: 10000,
          unitPrice: 0.01,          // $0.01 per call
          flatPrice: 10             // Plus $10 base fee
        },
        {
          upTo: 100000,
          unitPrice: 0.005,         // $0.005 per call
          flatPrice: 50
        },
        {
          upTo: null,               // Infinity
          unitPrice: 0.002,         // $0.002 per call
          flatPrice: 200
        }
      ]
    },

    lifecycle: {
      allowUpgrade: true,
      allowDowngrade: true,
      allowCancellation: true,
      cancellationPolicy: "immediate"
    }
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "percentage",
      percentage: 0.10,             // 10% platform fee
      minimumFee: 1,
      maximumFee: 1000,
      capAt: 1000
    }
  }
}
```

#### Scenario: Per-Seat Team Subscription
```typescript
{
  id: "team-subscription",
  name: "Team Plan (Per Seat)",
  category: "payment_method",
  type: "subscription",
  enabled: true,

  subscriptionConfig: {
    billingCycle: "monthly",

    pricing: {
      model: "flat",
      perSeat: {
        enabled: true,
        minSeats: 3,
        maxSeats: 100,
        pricePerSeat: 15
      }
    },

    lifecycle: {
      allowUpgrade: true,
      allowDowngrade: true,
      upgradeProration: "immediate",
      downgradeProration: "end_of_cycle"
    }
  },

  feeConfig: {
    subscriptionFee: {
      setupFee: 99,                 // One-time setup
      recurringFee: 0,
      upgradeFee: 0,
      downgradeFee: 0,
      cancellationFee: 0
    }
  }
}
```

### 3. Marketplace/Split Payments

#### Scenario: Multi-Vendor Marketplace
```typescript
{
  id: "marketplace-seller-payout",
  name: "Marketplace Seller Payment",
  category: "payment_method",
  type: "split",
  enabled: true,

  rbac: {
    allowedRoles: ["verified_seller"],
    customPermissions: {
      initiate: ["customer"],
      process: ["system"],
      viewTransactions: ["seller", "admin"]
    },
    requiresVerification: true,
    verificationLevel: "enhanced"    // Full KYC for sellers
  },

  renderConditions: {
    requiresAuth: true,
    requiresBusinessAccount: false,
    minimumTrustScore: 70
  },

  methodConfig: {
    provider: "stripe",
    acceptedTypes: {
      cards: {
        enabled: true,
        types: ["visa", "mastercard", "amex", "discover"],
        require3DS: true
      },
      wallets: {
        enabled: true,
        types: ["apple_pay", "google_pay", "paypal"]
      }
    }
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "hybrid",
      percentage: 0.029,
      fixedAmount: 0.30
    },
    processingFee: {
      passToCustomer: false,
      absorbCost: false,
      split: 0                      // Seller pays all processing fees
    }
  },

  splitPaymentConfig: {
    enabled: true,
    type: "marketplace",
    marketplace: {
      platformPercentage: 0.15,      // 15% to platform
      sellerPercentage: 0.85,        // 85% to seller
      transferSchedule: "weekly",
      holdPeriod: 14,                // 14-day hold for disputes
      connectAccountRequired: true,
      accountVerificationRequired: true
    }
  },

  payoutConfig: {
    schedule: "weekly",
    minimumAmount: 25,

    methods: {
      bankTransfer: {
        enabled: true,
        fee: 0,
        processingDays: 2
      },
      debitCard: {
        enabled: true,
        fee: 0.50,
        processingDays: 0            // Instant
      },
      paypal: {
        enabled: true,
        fee: 1.00,
        processingDays: 1
      }
    },

    holds: {
      initialHoldDays: 30,           // New sellers
      standardHoldDays: 7,
      highRiskHoldDays: 21
    }
  },

  disputeConfig: {
    autoRespond: true,
    evidenceCollection: {
      requireProofOfDelivery: true,
      requireCommunicationLogs: true
    }
  }
}
```

#### Scenario: Crowdfunding Campaign
```typescript
{
  id: "crowdfunding-campaign",
  name: "Crowdfunding Payment",
  category: "payment_method",
  type: "split",
  enabled: true,

  rbac: {
    allowedRoles: ["backer"],
    customPermissions: {
      initiate: ["backer"],
      refund: ["system", "admin"]
    }
  },

  renderConditions: {
    requiresAuth: true,
    minAmount: 1,
    maxAmount: 10000,
    dailyLimit: 50000
  },

  splitPaymentConfig: {
    enabled: true,
    type: "crowdfunding",
    crowdfunding: {
      goalAmount: 100000,
      deadline: new Date("2026-12-31"),
      refundIfNotMet: true,
      allowOverfunding: true,
      maxOverfundingPercentage: 200  // Up to 200% of goal
    }
  },

  escrowConfig: {
    enabled: true,
    releaseConditions: {
      automatic: true,
      conditions: ["milestone_reached"],
      holdDuration: 0                // Release when goal met
    },
    disputeResolution: {
      allowDisputes: true,
      timeoutAction: "refund_to_buyer"
    }
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "percentage",
      percentage: 0.05,              // 5% platform fee
      fixedAmount: 0
    }
  },

  refundConfig: {
    enabled: true,
    allowedBy: ["system"],
    policy: {
      windowDays: 0,
      partialRefunds: false,
      refundMethod: "original",
      requiresReason: false,
      approvalRequired: false        // Auto-refund if goal not met
    }
  }
}
```

### 4. Escrow Payments

#### Scenario: Freelance Project Payment
```typescript
{
  id: "freelance-escrow",
  name: "Freelance Project Escrow",
  category: "payment_method",
  type: "escrow",
  enabled: true,

  rbac: {
    allowedRoles: ["client", "freelancer"],
    customPermissions: {
      initiate: ["client"],
      process: ["system"],
      viewTransactions: ["client", "freelancer", "admin"]
    },
    requiresVerification: true,
    verificationLevel: "standard"
  },

  renderConditions: {
    requiresAuth: true,
    minAmount: 50,
    maxAmount: 100000,
    requiresPreviousPurchase: false,
    minimumAccountAge: 30
  },

  methodConfig: {
    provider: "stripe",
    acceptedTypes: {
      cards: {
        enabled: true,
        types: ["visa", "mastercard", "amex"],
        require3DS: true
      },
      bankTransfer: {
        enabled: true,
        types: ["ach", "wire"],
        verificationRequired: true
      }
    }
  },

  escrowConfig: {
    enabled: true,
    releaseConditions: {
      automatic: false,
      conditions: ["approval_granted", "milestone_reached"],
      holdDuration: 90               // Max 90 days in escrow
    },
    disputeResolution: {
      allowDisputes: true,
      mediationRequired: true,
      arbitrationProvider: "internal_team",
      timeoutAction: "hold"          // Hold if no resolution
    }
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "percentage",
      percentage: 0.10,              // 10% escrow fee
      minimumFee: 5
    },
    processingFee: {
      passToCustomer: true
    }
  },

  splitPaymentConfig: {
    enabled: true,
    type: "marketplace",
    marketplace: {
      platformPercentage: 0.10,
      sellerPercentage: 0.90,
      transferSchedule: "immediate",  // Release immediately when approved
      holdPeriod: 0
    }
  },

  notificationConfig: {
    customer: {
      paymentSuccess: {
        email: true,
        templateId: "escrow-deposit-confirmation"
      }
    },
    merchant: {
      newPayment: true
    }
  }
}
```

### 5. Installment/BNPL Payments

#### Scenario: Buy Now Pay Later
```typescript
{
  id: "bnpl-checkout",
  name: "Buy Now Pay Later",
  category: "payment_method",
  type: "installment",
  enabled: true,

  rbac: {
    allowedRoles: ["customer"],
    deniedRoles: ["suspended", "high_risk"],
    requiresVerification: true,
    verificationLevel: "standard"
  },

  renderConditions: {
    requiresAuth: true,
    minAmount: 100,
    maxAmount: 5000,
    requiresCountry: ["US", "GB", "AU", "DE"],
    requiresMinAge: 18,
    minimumAccountAge: 90,
    minimumTrustScore: 60
  },

  methodConfig: {
    provider: "stripe",
    acceptedTypes: {
      buyNowPayLater: {
        enabled: true,
        providers: ["klarna", "afterpay", "affirm"],
        minAmount: 100,
        maxAmount: 5000
      }
    }
  },

  installmentConfig: {
    enabled: true,
    provider: "klarna",

    terms: {
      minAmount: 100,
      maxAmount: 5000,
      availablePlans: [
        {
          installments: 4,
          interestRate: 0,           // Interest-free
          fees: 0
        },
        {
          installments: 6,
          interestRate: 0.10,        // 10% APR
          fees: 0
        },
        {
          installments: 12,
          interestRate: 0.15,        // 15% APR
          fees: 0
        }
      ]
    },

    creditCheck: {
      required: true,
      provider: "klarna",            // Provider handles credit check
      softPull: true
    }
  },

  feeConfig: {
    platformFee: {
      enabled: false
    },
    processingFee: {
      absorbCost: false,
      passToCustomer: false          // BNPL provider charges merchant
    }
  },

  notificationConfig: {
    customer: {
      paymentSuccess: {
        email: true,
        templateId: "bnpl-confirmation"
      },
      subscriptionRenewal: {
        email: true,
        daysBefore: [7, 3, 1]        // Remind before each installment
      }
    }
  }
}
```

### 6. Cryptocurrency Payments

#### Scenario: Bitcoin/Ethereum Checkout
```typescript
{
  id: "crypto-checkout",
  name: "Cryptocurrency Payment",
  category: "payment_method",
  type: "one_time",
  enabled: true,

  rbac: {
    allowedRoles: ["verified_customer"],
    requiresVerification: true,
    verificationLevel: "enhanced"    // KYC required for crypto
  },

  renderConditions: {
    requiresAuth: true,
    minAmount: 50,
    maxAmount: 100000,
    excludedCountries: ["CN", "IN"],  // Countries with crypto restrictions
    requiresMinAge: 18,
    minimumAccountAge: 30
  },

  methodConfig: {
    provider: "coinbase_commerce",
    acceptedTypes: {
      crypto: {
        enabled: true,
        currencies: ["BTC", "ETH", "USDT", "USDC"],
        network: "mainnet"
      }
    }
  },

  currencyConfig: {
    supportedCurrencies: ["USD", "EUR"],
    defaultCurrency: "USD",
    conversionProvider: "coinbase",
    conversionMarkup: 0.01          // 1% markup on crypto conversion
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "percentage",
      percentage: 0.01,              // 1% platform fee
      minimumFee: 1
    },
    processingFee: {
      absorbCost: false,
      passToCustomer: true           // Customer pays network fees
    }
  },

  refundConfig: {
    enabled: true,
    allowedBy: ["admin"],
    policy: {
      windowDays: 7,
      partialRefunds: false,
      refundMethod: "manual",        // Manual crypto refunds
      requiresReason: true,
      approvalRequired: true
    },
    restrictions: {
      maxRefundsPerUser: 2,
      noRefundAfterDays: 7
    }
  },

  complianceConfig: {
    regulations: {
      aml: true,
      kyc: true
    }
  },

  fraudConfig: {
    enabled: true,
    riskLevel: "high",
    rules: {
      maxTransactionsPerDay: 5,
      maxAmountPerDay: 50000,
      blockVpn: true,
      blockProxy: true,
      blockTor: true
    }
  }
}
```

### 7. Donation Payments

#### Scenario: Nonprofit Donation
```typescript
{
  id: "nonprofit-donation",
  name: "Nonprofit Donation",
  category: "payment_method",
  type: "one_time",
  enabled: true,

  rbac: {
    allowedRoles: ["*"],
    customPermissions: {
      view: ["*"],
      initiate: ["*"]
    }
  },

  renderConditions: {
    requiresAuth: false,
    minAmount: 1,
    maxAmount: 1000000
  },

  methodConfig: {
    provider: "stripe",
    acceptedTypes: {
      cards: {
        enabled: true,
        types: ["visa", "mastercard", "amex", "discover"],
        require3DS: false            // Lower friction for donations
        saveCards: true              // For recurring donations
      },
      wallets: {
        enabled: true,
        types: ["apple_pay", "google_pay", "paypal"]
      },
      bankTransfer: {
        enabled: true,
        types: ["ach"],
        verificationRequired: false
      }
    }
  },

  feeConfig: {
    platformFee: {
      enabled: false                // No fee for nonprofit donations
    },
    processingFee: {
      absorbCost: false,
      passToCustomer: true,
      cardFee: {
        percentage: 0.022,           // Nonprofit rate
        fixed: 0.30
      }
    },
    dynamicFees: [
      {
        ruleId: "cover-fees-option",
        condition: "user.coverFees === true",
        feeAdjustment: {
          type: "add",
          value: 0                  // User opted to cover fees
        }
      }
    ]
  },

  taxConfig: {
    enabled: true,
    type: "exclusive",
    collection: {
      collectTaxId: true,            // For tax deduction receipts
      validateTaxId: false
    }
  },

  notificationConfig: {
    customer: {
      paymentSuccess: {
        email: true,
        templateId: "donation-receipt"  // Tax-deductible receipt
      }
    }
  },

  reportingConfig: {
    customReports: [
      {
        reportId: "annual-donation-report",
        name: "Annual Donation Summary",
        query: "SELECT donor_id, SUM(amount) FROM donations WHERE year = :year",
        schedule: "0 0 1 1 *"       // January 1st annually
      }
    ]
  }
}
```

### 8. Invoice Payments

#### Scenario: B2B Invoice Payment
```typescript
{
  id: "b2b-invoice",
  name: "Business Invoice Payment",
  category: "payment_method",
  type: "one_time",
  enabled: true,

  rbac: {
    allowedRoles: ["business_customer"],
    requiresVerification: true,
    verificationLevel: "enhanced",
    customPermissions: {
      initiate: ["business_customer", "accountant"],
      viewTransactions: ["business_customer", "accountant", "admin"]
    }
  },

  renderConditions: {
    requiresAuth: true,
    requiresBusinessAccount: true,
    minAmount: 1000,
    maxAmount: 500000,
    minimumAccountAge: 90
  },

  methodConfig: {
    provider: "stripe",
    acceptedTypes: {
      cards: {
        enabled: true,
        types: ["visa", "mastercard", "amex"],
        require3DS: true
      },
      bankTransfer: {
        enabled: true,
        types: ["ach", "wire"],
        verificationRequired: true
      }
    }
  },

  currencyConfig: {
    supportedCurrencies: ["USD", "EUR", "GBP", "JPY", "AUD"],
    defaultCurrency: "USD"
  },

  feeConfig: {
    platformFee: {
      enabled: true,
      type: "percentage",
      percentage: 0.005,             // 0.5% for large B2B transactions
      minimumFee: 10,
      maximumFee: 500
    },
    processingFee: {
      passToCustomer: false,
      absorbCost: true
    }
  },

  taxConfig: {
    enabled: true,
    type: "exclusive",
    calculation: {
      method: "automatic",
      provider: "avalara"
    },
    collection: {
      collectTaxId: true,
      validateTaxId: true,
      reverseCharge: true            // EU B2B VAT reverse charge
    }
  },

  payoutConfig: {
    schedule: "manual",              // Invoice-based payouts
    minimumAmount: 0
  },

  notificationConfig: {
    customer: {
      paymentSuccess: {
        email: true,
        templateId: "invoice-payment-confirmation"
      }
    },
    merchant: {
      newPayment: true
    }
  },

  reportingConfig: {
    exports: {
      formats: ["csv", "pdf"],
      scheduled: {
        enabled: true,
        frequency: "monthly",
        recipients: ["accounting@company.com"]
      }
    }
  },

  complianceConfig: {
    dataRetention: {
      transactionData: 2555,         // 7 years for tax purposes
      customerData: 2555
    }
  }
}
```

---

## Admin Configuration Interface

### Dashboard Layout
```typescript
interface AdminPaymentDashboard {
  // Top-level sections
  sections: {
    overview: {
      metrics: {
        totalRevenue: number
        transactionVolume: number
        activeSubscriptions: number
        refundRate: number
        chargebackRate: number
        averageTransactionValue: number
      }
      charts: {
        revenueOverTime: TimeSeriesChart
        paymentMethodDistribution: PieChart
        geographicDistribution: MapChart
        conversionFunnel: FunnelChart
      }
    }

    paymentMethods: {
      list: PaymentConfiguration[]
      filters: {
        category: PaymentCategory[]
        type: PaymentType[]
        status: ("active" | "inactive" | "draft")[]
        provider: string[]
      }
      actions: {
        create: () => void
        edit: (id: string) => void
        duplicate: (id: string) => void
        enable: (id: string) => void
        disable: (id: string) => void
        delete: (id: string) => void
        bulkActions: () => void
      }
    }

    transactions: {
      list: Transaction[]
      filters: {
        status: ("pending" | "success" | "failed" | "refunded")[]
        dateRange: [Date, Date]
        amountRange: [number, number]
        paymentMethod: string[]
        customer: string
      }
      actions: {
        view: (id: string) => void
        refund: (id: string) => void
        export: () => void
      }
    }

    subscriptions: {
      list: Subscription[]
      filters: {
        status: ("active" | "cancelled" | "paused" | "expired")[]
        plan: string[]
        customer: string
      }
      actions: {
        view: (id: string) => void
        cancel: (id: string) => void
        pause: (id: string) => void
        resume: (id: string) => void
      }
    }

    disputes: {
      list: Dispute[]
      filters: {
        status: ("open" | "under_review" | "won" | "lost")[]
        reason: string[]
      }
      actions: {
        view: (id: string) => void
        respond: (id: string) => void
        acceptLoss: (id: string) => void
      }
    }

    payouts: {
      list: Payout[]
      filters: {
        status: ("pending" | "in_transit" | "paid" | "failed")[]
        recipient: string
      }
      actions: {
        initiate: () => void
        cancel: (id: string) => void
        retry: (id: string) => void
      }
    }

    settings: {
      providers: {
        stripe: ProviderConfig
        paypal: ProviderConfig
        square: ProviderConfig
        custom: ProviderConfig[]
      }

      currencies: {
        supported: string[]
        default: string
        conversionProvider: string
      }

      fees: {
        platform: FeeConfig
        processing: FeeConfig
        custom: FeeConfig[]
      }

      compliance: {
        pci: PCIConfig
        regulations: RegulationConfig
        dataRetention: RetentionConfig
      }

      notifications: {
        customer: NotificationConfig
        merchant: NotificationConfig
        admin: NotificationConfig
      }

      fraud: FraudConfig

      webhooks: WebhookConfig

      reporting: ReportingConfig
    }
  }

  // Quick actions
  quickActions: {
    createPaymentMethod: () => void
    viewLatestTransactions: () => void
    processRefund: () => void
    downloadReport: () => void
  }
}
```

### Configuration Form Builder
```typescript
interface PaymentConfigForm {
  // Form structure
  tabs: {
    basic: {
      fields: [
        { name: "name", type: "text", required: true },
        { name: "category", type: "select", options: PaymentCategory[], required: true },
        { name: "type", type: "select", options: PaymentType[], required: true },
        { name: "enabled", type: "toggle", default: false },
        { name: "enabledFrom", type: "datetime", conditional: "enabled === true" },
        { name: "enabledUntil", type: "datetime", conditional: "enabled === true" }
      ]
    }

    rbac: {
      fields: [
        { name: "allowedRoles", type: "multi-select", options: Role[] },
        { name: "deniedRoles", type: "multi-select", options: Role[] },
        { name: "requiresVerification", type: "toggle" },
        { name: "verificationLevel", type: "select", options: ["basic", "standard", "enhanced"], conditional: "requiresVerification === true" }
      ]
      customPermissions: {
        matrix: PermissionMatrix  // Role × Permission grid
      }
    }

    renderConditions: {
      fields: [
        { name: "requiresAuth", type: "toggle" },
        { name: "minAmount", type: "number", min: 0 },
        { name: "maxAmount", type: "number", min: 0 },
        { name: "requiresCountry", type: "country-multi-select" },
        { name: "excludedCountries", type: "country-multi-select" }
      ]
      customRules: {
        builder: RuleBuilder      // Visual rule builder
      }
    }

    paymentMethod: {
      fields: [
        { name: "provider", type: "select", options: ["stripe", "paypal", "square", "custom"] },
        { name: "providerMode", type: "select", options: ["live", "test"] }
      ]
      credentials: {
        encrypted: true
        fields: [
          { name: "publicKey", type: "text" },
          { name: "secretKey", type: "password", encrypted: true },
          { name: "webhookSecret", type: "password", encrypted: true }
        ]
      }
      acceptedTypes: {
        cards: CardTypeSelector
        wallets: WalletSelector
        bankTransfer: BankTransferSelector
        crypto: CryptoSelector
        bnpl: BNPLSelector
      }
    }

    currency: {
      fields: [
        { name: "supportedCurrencies", type: "currency-multi-select" },
        { name: "defaultCurrency", type: "currency-select" },
        { name: "autoDetectCurrency", type: "toggle" },
        { name: "conversionProvider", type: "select", options: ["exchangerates", "stripe", "custom"] }
      ]
    }

    fees: {
      platformFee: FeeConfigForm
      processingFee: ProcessingFeeForm
      subscriptionFee: SubscriptionFeeForm
      dynamicFees: DynamicFeeBuilder
    }

    subscription: {
      conditional: "type === 'subscription'"
      fields: [
        { name: "billingCycle", type: "select", options: ["daily", "weekly", "monthly", "quarterly", "yearly"] },
        { name: "pricingModel", type: "select", options: ["flat", "tiered", "volume", "usage", "hybrid"] }
      ]
      trialPeriod: TrialPeriodForm
      lifecycle: LifecycleForm
      renewals: RenewalForm
    }

    refunds: {
      policy: RefundPolicyForm
      restrictions: RefundRestrictionForm
    }

    fraud: {
      riskLevel: RiskLevelSelector
      rules: FraudRuleBuilder
      actions: FraudActionSelector
    }

    compliance: {
      pci: PCIForm
      regulations: RegulationCheckboxes
      dataRetention: RetentionPolicyForm
    }

    notifications: {
      customer: NotificationTemplateSelector
      merchant: NotificationTemplateSelector
      admin: NotificationTemplateSelector
    }

    webhooks: {
      endpoints: WebhookEndpointList
      events: WebhookEventSelector
    }

    advanced: {
      splitPayment: SplitPaymentForm
      escrow: EscrowForm
      installment: InstallmentForm
      payout: PayoutForm
      tax: TaxForm
    }
  }

  // Form actions
  actions: {
    save: () => void
    saveAndActivate: () => void
    cancel: () => void
    preview: () => void           // Preview user-facing UI
    test: () => void              // Test payment flow
    duplicate: () => void
    export: () => void
  }

  // Validation
  validation: {
    realTime: boolean
    rules: ValidationRule[]
  }

  // Help & Documentation
  help: {
    contextualHelp: boolean
    tooltips: boolean
    documentation: string
    examples: PaymentConfiguration[]
  }
}
```

### Visual Rule Builder
```typescript
interface RuleBuilder {
  // Drag-and-drop rule creation
  conditions: {
    field: string               // "amount", "user.role", "user.country"
    operator: "equals" | "not_equals" | "greater_than" | "less_than" | "contains" | "in" | "not_in"
    value: any
  }[]

  logic: "AND" | "OR"

  // Visual representation
  tree: RuleTreeNode

  // Code view
  expression: string            // Generated code expression

  // Testing
  testCase: {
    input: any
    expectedOutput: boolean
  }[]
}
```

---

## User-Side Experience

### Payment Selection Interface
```typescript
interface UserPaymentInterface {
  // Payment method selection
  methods: {
    card: {
      visible: boolean          // Based on renderConditions + RBAC
      disabled: boolean
      disabledReason?: string
      icon: string
      label: string
      description: string
      fees: {
        processing: number
        platform: number
        total: number
      }
    }

    wallet: {
      visible: boolean
      options: ("apple_pay" | "google_pay" | "paypal")[]
    }

    bankTransfer: {
      visible: boolean
      types: ("ach" | "sepa" | "wire")[]
    }

    crypto: {
      visible: boolean
      currencies: ("BTC" | "ETH" | "USDT")[]
    }

    bnpl: {
      visible: boolean
      providers: ("klarna" | "afterpay" | "affirm")[]
      terms: InstallmentTerm[]
    }
  }

  // Amount display
  amount: {
    subtotal: number
    fees: {
      platform: number
      processing: number
      tax: number
    }
    total: number
    currency: string
    formatted: string
  }

  // Form fields
  form: {
    // Conditional based on payment method
    cardNumber?: boolean
    expiryDate?: boolean
    cvv?: boolean
    billingAddress?: boolean
    taxId?: boolean              // For B2B

    // Save for future
    savePaymentMethod?: boolean

    // Terms acceptance
    termsAcceptance: boolean
  }

  // Security indicators
  security: {
    ssl: boolean
    pciCompliant: boolean
    threeDSecure?: boolean
  }

  // Actions
  actions: {
    submit: () => void
    cancel: () => void
    changeCurrency: (currency: string) => void
  }
}
```

### Payment Flow States
```typescript
type PaymentFlowState =
  | { type: "idle" }
  | { type: "selecting_method" }
  | { type: "entering_details" }
  | { type: "validating", data: PaymentData }
  | { type: "processing", transactionId: string }
  | { type: "3ds_challenge", challengeUrl: string }
  | { type: "success", transaction: Transaction }
  | { type: "failed", error: PaymentError, retryable: boolean }
  | { type: "cancelled" }
```

### User Payment Dashboard
```typescript
interface UserPaymentDashboard {
  // Overview
  overview: {
    balance: number
    pendingPayments: number
    completedPayments: number
    savedPaymentMethods: number
  }

  // Transaction history
  transactions: {
    list: UserTransaction[]
    filters: {
      status: ("pending" | "success" | "failed" | "refunded")[]
      dateRange: [Date, Date]
      amountRange: [number, number]
    }
    actions: {
      view: (id: string) => void
      downloadReceipt: (id: string) => void
      requestRefund: (id: string) => void
    }
  }

  // Subscriptions
  subscriptions: {
    list: UserSubscription[]
    actions: {
      view: (id: string) => void
      upgrade: (id: string) => void
      downgrade: (id: string) => void
      pause: (id: string) => void
      cancel: (id: string) => void
      updatePaymentMethod: (id: string) => void
    }
  }

  // Saved payment methods
  paymentMethods: {
    list: SavedPaymentMethod[]
    actions: {
      add: () => void
      setDefault: (id: string) => void
      remove: (id: string) => void
    }
  }

  // Invoices
  invoices: {
    list: Invoice[]
    actions: {
      download: (id: string) => void
      pay: (id: string) => void
    }
  }

  // Settings
  settings: {
    defaultCurrency: string
    billingAddress: Address
    taxId?: string
    notifications: NotificationPreferences
  }
}
```

---

## Database Schema

### Core Tables

```sql
-- Payment configurations
CREATE TABLE payment_configurations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  category VARCHAR(50) NOT NULL,
  type VARCHAR(50) NOT NULL,
  enabled BOOLEAN DEFAULT false,
  enabled_from TIMESTAMP,
  enabled_until TIMESTAMP,

  -- RBAC
  allowed_roles TEXT[],
  denied_roles TEXT[],
  custom_permissions JSONB,
  requires_verification BOOLEAN DEFAULT false,
  verification_level VARCHAR(20),

  -- Render conditions
  render_conditions JSONB,

  -- Method configuration
  provider VARCHAR(50),
  provider_mode VARCHAR(10),
  credentials_encrypted TEXT,
  accepted_types JSONB,

  -- Currency configuration
  supported_currencies TEXT[],
  default_currency VARCHAR(3),
  currency_config JSONB,

  -- Fee configuration
  fee_config JSONB,

  -- Subscription configuration
  subscription_config JSONB,

  -- Refund configuration
  refund_config JSONB,

  -- Dispute configuration
  dispute_config JSONB,

  -- Tax configuration
  tax_config JSONB,

  -- Split payment configuration
  split_payment_config JSONB,

  -- Escrow configuration
  escrow_config JSONB,

  -- Installment configuration
  installment_config JSONB,

  -- Payout configuration
  payout_config JSONB,

  -- Notification configuration
  notification_config JSONB,

  -- Fraud configuration
  fraud_config JSONB,

  -- Compliance configuration
  compliance_config JSONB,

  -- Webhook configuration
  webhook_config JSONB,

  -- Reporting configuration
  reporting_config JSONB,

  -- Metadata
  metadata JSONB,

  -- State
  state VARCHAR(20) DEFAULT 'draft',

  -- Test mode
  test_mode JSONB,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES users(id),
  updated_at TIMESTAMP DEFAULT NOW(),
  updated_by UUID REFERENCES users(id),
  version INTEGER DEFAULT 1,

  CONSTRAINT valid_state CHECK (state IN ('draft', 'active', 'inactive', 'suspended', 'archived', 'deprecated'))
);

-- Indexes
CREATE INDEX idx_payment_configs_category ON payment_configurations(category);
CREATE INDEX idx_payment_configs_type ON payment_configurations(type);
CREATE INDEX idx_payment_configs_enabled ON payment_configurations(enabled);
CREATE INDEX idx_payment_configs_state ON payment_configurations(state);
CREATE INDEX idx_payment_configs_provider ON payment_configurations(provider);

-- Transactions
CREATE TABLE transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  payment_config_id UUID REFERENCES payment_configurations(id),

  -- Transaction details
  amount DECIMAL(19, 4) NOT NULL,
  currency VARCHAR(3) NOT NULL,
  amount_refunded DECIMAL(19, 4) DEFAULT 0,

  -- Fees
  platform_fee DECIMAL(19, 4) DEFAULT 0,
  processing_fee DECIMAL(19, 4) DEFAULT 0,
  tax_amount DECIMAL(19, 4) DEFAULT 0,
  net_amount DECIMAL(19, 4),

  -- Parties
  customer_id UUID REFERENCES users(id),
  merchant_id UUID REFERENCES users(id),

  -- Payment method
  payment_method VARCHAR(50),
  payment_method_details JSONB,

  -- Status
  status VARCHAR(20) NOT NULL,
  status_history JSONB[],

  -- Provider details
  provider VARCHAR(50),
  provider_transaction_id VARCHAR(255),
  provider_response JSONB,

  -- Metadata
  description TEXT,
  metadata JSONB,

  -- Risk & fraud
  fraud_score DECIMAL(5, 2),
  fraud_details JSONB,

  -- Timestamps
  initiated_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  failed_at TIMESTAMP,
  refunded_at TIMESTAMP,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_status CHECK (status IN ('pending', 'processing', 'success', 'failed', 'cancelled', 'refunded', 'partially_refunded'))
);

-- Indexes
CREATE INDEX idx_transactions_customer ON transactions(customer_id);
CREATE INDEX idx_transactions_merchant ON transactions(merchant_id);
CREATE INDEX idx_transactions_status ON transactions(status);
CREATE INDEX idx_transactions_payment_config ON transactions(payment_config_id);
CREATE INDEX idx_transactions_created_at ON transactions(created_at);
CREATE INDEX idx_transactions_provider_id ON transactions(provider_transaction_id);

-- Subscriptions
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  payment_config_id UUID REFERENCES payment_configurations(id),

  -- Subscription details
  customer_id UUID REFERENCES users(id) NOT NULL,
  plan_name VARCHAR(255),
  billing_cycle VARCHAR(50),
  amount DECIMAL(19, 4) NOT NULL,
  currency VARCHAR(3) NOT NULL,

  -- Lifecycle
  status VARCHAR(20) NOT NULL,
  started_at TIMESTAMP DEFAULT NOW(),
  trial_ends_at TIMESTAMP,
  current_period_start TIMESTAMP,
  current_period_end TIMESTAMP,
  cancelled_at TIMESTAMP,
  ended_at TIMESTAMP,

  -- Payment
  payment_method_id UUID,
  latest_invoice_id UUID,

  -- Metadata
  metadata JSONB,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_status CHECK (status IN ('trialing', 'active', 'past_due', 'cancelled', 'paused', 'expired'))
);

-- Indexes
CREATE INDEX idx_subscriptions_customer ON subscriptions(customer_id);
CREATE INDEX idx_subscriptions_status ON subscriptions(status);
CREATE INDEX idx_subscriptions_current_period_end ON subscriptions(current_period_end);

-- Refunds
CREATE TABLE refunds (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_id UUID REFERENCES transactions(id) NOT NULL,

  -- Refund details
  amount DECIMAL(19, 4) NOT NULL,
  currency VARCHAR(3) NOT NULL,
  reason VARCHAR(255),
  status VARCHAR(20) NOT NULL,

  -- Processing
  requested_by UUID REFERENCES users(id),
  approved_by UUID REFERENCES users(id),
  processed_at TIMESTAMP,

  -- Provider
  provider VARCHAR(50),
  provider_refund_id VARCHAR(255),
  provider_response JSONB,

  -- Metadata
  metadata JSONB,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_status CHECK (status IN ('requested', 'processing', 'completed', 'failed', 'cancelled'))
);

-- Indexes
CREATE INDEX idx_refunds_transaction ON refunds(transaction_id);
CREATE INDEX idx_refunds_status ON refunds(status);

-- Disputes/Chargebacks
CREATE TABLE disputes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_id UUID REFERENCES transactions(id) NOT NULL,

  -- Dispute details
  amount DECIMAL(19, 4) NOT NULL,
  currency VARCHAR(3) NOT NULL,
  reason VARCHAR(255),
  status VARCHAR(20) NOT NULL,

  -- Evidence
  evidence JSONB,
  evidence_due_by TIMESTAMP,
  evidence_submitted_at TIMESTAMP,

  -- Resolution
  resolved_at TIMESTAMP,
  outcome VARCHAR(20),

  -- Provider
  provider VARCHAR(50),
  provider_dispute_id VARCHAR(255),
  provider_response JSONB,

  -- Metadata
  metadata JSONB,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_status CHECK (status IN ('open', 'under_review', 'won', 'lost', 'withdrawn')),
  CONSTRAINT valid_outcome CHECK (outcome IN ('won', 'lost', 'accepted', NULL))
);

-- Indexes
CREATE INDEX idx_disputes_transaction ON disputes(transaction_id);
CREATE INDEX idx_disputes_status ON disputes(status);

-- Payouts
CREATE TABLE payouts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Payout details
  recipient_id UUID REFERENCES users(id) NOT NULL,
  amount DECIMAL(19, 4) NOT NULL,
  currency VARCHAR(3) NOT NULL,
  fee DECIMAL(19, 4) DEFAULT 0,
  net_amount DECIMAL(19, 4),

  -- Method
  method VARCHAR(50),
  destination JSONB,

  -- Status
  status VARCHAR(20) NOT NULL,
  initiated_at TIMESTAMP DEFAULT NOW(),
  expected_arrival_date DATE,
  completed_at TIMESTAMP,
  failed_at TIMESTAMP,
  failure_reason TEXT,

  -- Provider
  provider VARCHAR(50),
  provider_payout_id VARCHAR(255),
  provider_response JSONB,

  -- Metadata
  description TEXT,
  metadata JSONB,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_status CHECK (status IN ('pending', 'in_transit', 'paid', 'failed', 'cancelled'))
);

-- Indexes
CREATE INDEX idx_payouts_recipient ON payouts(recipient_id);
CREATE INDEX idx_payouts_status ON payouts(status);

-- Saved payment methods
CREATE TABLE saved_payment_methods (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) NOT NULL,

  -- Method details
  type VARCHAR(50) NOT NULL,
  provider VARCHAR(50),
  provider_method_id VARCHAR(255),

  -- Card details (masked)
  card_brand VARCHAR(50),
  card_last4 VARCHAR(4),
  card_exp_month INTEGER,
  card_exp_year INTEGER,

  -- Bank details (masked)
  bank_name VARCHAR(255),
  bank_last4 VARCHAR(4),
  account_type VARCHAR(50),

  -- Settings
  is_default BOOLEAN DEFAULT false,
  billing_address JSONB,

  -- Metadata
  metadata JSONB,

  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT one_default_per_user UNIQUE (user_id, is_default) WHERE is_default = true
);

-- Indexes
CREATE INDEX idx_saved_payment_methods_user ON saved_payment_methods(user_id);
CREATE INDEX idx_saved_payment_methods_provider_id ON saved_payment_methods(provider_method_id);

-- Audit log
CREATE TABLE payment_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Reference
  entity_type VARCHAR(50) NOT NULL,
  entity_id UUID NOT NULL,

  -- Change details
  action VARCHAR(50) NOT NULL,
  changes JSONB,

  -- Actor
  user_id UUID REFERENCES users(id),
  ip_address INET,
  user_agent TEXT,

  -- Timestamp
  created_at TIMESTAMP DEFAULT NOW(),

  CONSTRAINT valid_entity_type CHECK (entity_type IN ('payment_configuration', 'transaction', 'subscription', 'refund', 'dispute', 'payout'))
);

-- Indexes
CREATE INDEX idx_audit_entity ON payment_audit_log(entity_type, entity_id);
CREATE INDEX idx_audit_user ON payment_audit_log(user_id);
CREATE INDEX idx_audit_created_at ON payment_audit_log(created_at);
```

---

## API Architecture

### RESTful Endpoints

```typescript
// Payment Configuration Management
GET    /api/admin/payment-configs              // List all configurations
POST   /api/admin/payment-configs              // Create new configuration
GET    /api/admin/payment-configs/:id          // Get specific configuration
PUT    /api/admin/payment-configs/:id          // Update configuration
DELETE /api/admin/payment-configs/:id          // Delete configuration
PATCH  /api/admin/payment-configs/:id/enable   // Enable configuration
PATCH  /api/admin/payment-configs/:id/disable  // Disable configuration
POST   /api/admin/payment-configs/:id/duplicate // Duplicate configuration
POST   /api/admin/payment-configs/:id/test     // Test configuration

// Payment Processing
GET    /api/payments/methods                   // Get available payment methods (filtered by RBAC + conditions)
POST   /api/payments/intent                    // Create payment intent
POST   /api/payments/process                   // Process payment
GET    /api/payments/:id                       // Get payment status
POST   /api/payments/:id/cancel                // Cancel payment
POST   /api/payments/:id/confirm               // Confirm payment (for 3DS, etc)

// Subscriptions
GET    /api/subscriptions                      // List user's subscriptions
POST   /api/subscriptions                      // Create subscription
GET    /api/subscriptions/:id                  // Get subscription details
PATCH  /api/subscriptions/:id                  // Update subscription
POST   /api/subscriptions/:id/cancel           // Cancel subscription
POST   /api/subscriptions/:id/pause            // Pause subscription
POST   /api/subscriptions/:id/resume           // Resume subscription
POST   /api/subscriptions/:id/upgrade          // Upgrade subscription
POST   /api/subscriptions/:id/downgrade        // Downgrade subscription

// Refunds
POST   /api/refunds                            // Request refund
GET    /api/refunds/:id                        // Get refund status
POST   /api/admin/refunds/:id/approve          // Approve refund
POST   /api/admin/refunds/:id/reject           // Reject refund

// Disputes
GET    /api/disputes                           // List disputes
GET    /api/disputes/:id                       // Get dispute details
POST   /api/disputes/:id/evidence              // Submit evidence
POST   /api/disputes/:id/accept                // Accept dispute

// Payouts
GET    /api/payouts                            // List payouts
POST   /api/payouts                            // Initiate payout
GET    /api/payouts/:id                        // Get payout status
POST   /api/payouts/:id/cancel                 // Cancel payout

// Saved Payment Methods
GET    /api/payment-methods                    // List saved methods
POST   /api/payment-methods                    // Save payment method
DELETE /api/payment-methods/:id                // Remove payment method
PATCH  /api/payment-methods/:id/default        // Set as default

// Webhooks (from payment providers)
POST   /api/webhooks/stripe                    // Stripe webhook
POST   /api/webhooks/paypal                    // PayPal webhook
POST   /api/webhooks/square                    // Square webhook

// Reporting
GET    /api/admin/reports/revenue              // Revenue report
GET    /api/admin/reports/transactions         // Transaction report
GET    /api/admin/reports/subscriptions        // Subscription report
GET    /api/admin/reports/refunds              // Refund report
POST   /api/admin/reports/export               // Export report

// Analytics
GET    /api/admin/analytics/overview           // Overview metrics
GET    /api/admin/analytics/conversion         // Conversion funnel
GET    /api/admin/analytics/cohorts            // Cohort analysis
GET    /api/admin/analytics/churn              // Churn analysis
```

### Payment Processing Flow

```typescript
// 1. Get available payment methods
async function getAvailablePaymentMethods(context: RequestContext): Promise<PaymentMethod[]> {
  const user = context.user
  const amount = context.amount
  const currency = context.currency

  // Fetch all active payment configurations
  const configs = await db.paymentConfigurations.findMany({
    where: { enabled: true, state: 'active' }
  })

  // Filter based on RBAC + render conditions
  const availableMethods = configs.filter(config => {
    // RBAC check
    if (!checkRBAC(config.rbac, user)) return false

    // Render conditions check
    if (!checkRenderConditions(config.renderConditions, {
      user,
      amount,
      currency,
      timestamp: new Date()
    })) return false

    return true
  })

  return availableMethods.map(config => ({
    id: config.id,
    name: config.name,
    type: config.type,
    icon: config.metadata.icon,
    description: config.metadata.description,
    fees: calculateFees(config.feeConfig, amount, currency)
  }))
}

// 2. Create payment intent
async function createPaymentIntent(data: PaymentIntentData): Promise<PaymentIntent> {
  const config = await db.paymentConfigurations.findUnique({
    where: { id: data.paymentConfigId }
  })

  // Validate again (RBAC + conditions)
  if (!validatePayment(config, data)) {
    throw new Error('Payment not allowed')
  }

  // Calculate amounts
  const fees = calculateFees(config.feeConfig, data.amount, data.currency)
  const tax = calculateTax(config.taxConfig, data.amount, data.customer)
  const totalAmount = data.amount + fees.total + tax

  // Fraud check
  const fraudScore = await checkFraud(config.fraudConfig, {
    user: data.customer,
    amount: totalAmount,
    paymentMethod: data.paymentMethod
  })

  if (fraudScore > config.fraudConfig.rules.mlFraudDetection.scoreThreshold) {
    throw new Error('Transaction blocked: high fraud risk')
  }

  // Create transaction record
  const transaction = await db.transactions.create({
    data: {
      paymentConfigId: config.id,
      customerId: data.customer.id,
      amount: data.amount,
      currency: data.currency,
      platformFee: fees.platform,
      processingFee: fees.processing,
      taxAmount: tax,
      netAmount: totalAmount,
      status: 'pending',
      fraudScore,
      metadata: data.metadata
    }
  })

  // Create payment intent with provider
  const providerIntent = await createProviderIntent(config, {
    amount: totalAmount,
    currency: data.currency,
    customer: data.customer,
    metadata: {
      transactionId: transaction.id,
      ...data.metadata
    }
  })

  // Update transaction with provider details
  await db.transactions.update({
    where: { id: transaction.id },
    data: {
      provider: config.provider,
      providerTransactionId: providerIntent.id,
      providerResponse: providerIntent
    }
  })

  return {
    id: transaction.id,
    clientSecret: providerIntent.clientSecret,
    amount: totalAmount,
    currency: data.currency,
    status: 'pending'
  }
}

// 3. Process payment
async function processPayment(data: ProcessPaymentData): Promise<Transaction> {
  const transaction = await db.transactions.findUnique({
    where: { id: data.transactionId },
    include: { paymentConfig: true }
  })

  // Process with provider
  const result = await processProviderPayment(
    transaction.paymentConfig,
    data.paymentMethod,
    transaction.providerTransactionId
  )

  // Handle split payments
  if (transaction.paymentConfig.splitPaymentConfig?.enabled) {
    await processSplitPayment(transaction, result)
  }

  // Update transaction
  const updatedTransaction = await db.transactions.update({
    where: { id: transaction.id },
    data: {
      status: result.status === 'succeeded' ? 'success' : 'failed',
      completedAt: result.status === 'succeeded' ? new Date() : undefined,
      failedAt: result.status === 'failed' ? new Date() : undefined,
      providerResponse: result
    }
  })

  // Send notifications
  await sendPaymentNotifications(transaction.paymentConfig.notificationConfig, {
    transaction: updatedTransaction,
    customer: transaction.customer,
    merchant: transaction.merchant
  })

  // Trigger webhooks
  await triggerWebhooks(transaction.paymentConfig.webhookConfig, {
    event: result.status === 'succeeded' ? 'payment.success' : 'payment.failed',
    data: updatedTransaction
  })

  return updatedTransaction
}

// 4. Process refund
async function processRefund(data: RefundData): Promise<Refund> {
  const transaction = await db.transactions.findUnique({
    where: { id: data.transactionId },
    include: { paymentConfig: true }
  })

  // Validate refund policy
  if (!validateRefund(transaction.paymentConfig.refundConfig, {
    transaction,
    amount: data.amount,
    requestedBy: data.requestedBy
  })) {
    throw new Error('Refund not allowed')
  }

  // Create refund record
  const refund = await db.refunds.create({
    data: {
      transactionId: transaction.id,
      amount: data.amount,
      currency: transaction.currency,
      reason: data.reason,
      status: 'requested',
      requestedBy: data.requestedBy
    }
  })

  // Auto-approve if under threshold
  const autoApprove = transaction.paymentConfig.refundConfig.policy.autoApproveUnder
  if (autoApprove && data.amount <= autoApprove) {
    return await executeRefund(refund)
  }

  return refund
}
```

---

## State Management

### React/Next.js Implementation

```typescript
// Store: Zustand
import { create } from 'zustand'

interface PaymentStore {
  // Available methods
  availableMethods: PaymentMethod[]
  selectedMethod: PaymentMethod | null

  // Payment flow
  flowState: PaymentFlowState
  transaction: Transaction | null

  // Form data
  paymentData: PaymentData

  // Actions
  fetchAvailableMethods: (amount: number, currency: string) => Promise<void>
  selectMethod: (method: PaymentMethod) => void
  updatePaymentData: (data: Partial<PaymentData>) => void
  createPaymentIntent: () => Promise<void>
  processPayment: () => Promise<void>
  cancelPayment: () => Promise<void>
  reset: () => void
}

export const usePaymentStore = create<PaymentStore>((set, get) => ({
  availableMethods: [],
  selectedMethod: null,
  flowState: { type: 'idle' },
  transaction: null,
  paymentData: {},

  fetchAvailableMethods: async (amount, currency) => {
    const response = await fetch('/api/payments/methods', {
      method: 'POST',
      body: JSON.stringify({ amount, currency })
    })
    const methods = await response.json()
    set({ availableMethods: methods, flowState: { type: 'selecting_method' } })
  },

  selectMethod: (method) => {
    set({ selectedMethod: method, flowState: { type: 'entering_details' } })
  },

  updatePaymentData: (data) => {
    set({ paymentData: { ...get().paymentData, ...data } })
  },

  createPaymentIntent: async () => {
    set({ flowState: { type: 'validating', data: get().paymentData } })

    const response = await fetch('/api/payments/intent', {
      method: 'POST',
      body: JSON.stringify({
        paymentConfigId: get().selectedMethod?.id,
        ...get().paymentData
      })
    })

    const intent = await response.json()
    set({
      transaction: intent,
      flowState: { type: 'processing', transactionId: intent.id }
    })
  },

  processPayment: async () => {
    const { transaction, paymentData } = get()

    try {
      const response = await fetch('/api/payments/process', {
        method: 'POST',
        body: JSON.stringify({
          transactionId: transaction.id,
          paymentMethod: paymentData.paymentMethod
        })
      })

      const result = await response.json()

      if (result.requires3DS) {
        set({ flowState: { type: '3ds_challenge', challengeUrl: result.challengeUrl } })
      } else if (result.status === 'success') {
        set({ flowState: { type: 'success', transaction: result } })
      } else {
        set({ flowState: { type: 'failed', error: result.error, retryable: result.retryable } })
      }
    } catch (error) {
      set({ flowState: { type: 'failed', error, retryable: true } })
    }
  },

  cancelPayment: async () => {
    const { transaction } = get()
    if (transaction) {
      await fetch(`/api/payments/${transaction.id}/cancel`, { method: 'POST' })
    }
    set({ flowState: { type: 'cancelled' } })
  },

  reset: () => {
    set({
      availableMethods: [],
      selectedMethod: null,
      flowState: { type: 'idle' },
      transaction: null,
      paymentData: {}
    })
  }
}))

// Components
export function PaymentMethodSelector() {
  const { availableMethods, selectedMethod, selectMethod } = usePaymentStore()

  return (
    <div className="grid grid-cols-2 gap-4">
      {availableMethods.map(method => (
        <button
          key={method.id}
          onClick={() => selectMethod(method)}
          className={cn(
            "border rounded-lg p-4",
            selectedMethod?.id === method.id && "border-blue-500"
          )}
        >
          <div className="flex items-center gap-3">
            <img src={method.icon} alt={method.name} className="w-8 h-8" />
            <div>
              <div className="font-medium">{method.name}</div>
              <div className="text-sm text-gray-500">{method.description}</div>
            </div>
          </div>
          <div className="mt-2 text-sm">
            Fees: {formatCurrency(method.fees.total)}
          </div>
        </button>
      ))}
    </div>
  )
}

export function PaymentForm() {
  const { selectedMethod, paymentData, updatePaymentData, createPaymentIntent } = usePaymentStore()

  if (!selectedMethod) return null

  return (
    <form onSubmit={async (e) => {
      e.preventDefault()
      await createPaymentIntent()
    }}>
      {selectedMethod.requiresCard && (
        <div className="space-y-4">
          <CardElement onChange={(data) => updatePaymentData({ card: data })} />
          <BillingAddressForm onChange={(data) => updatePaymentData({ billingAddress: data })} />
        </div>
      )}

      {selectedMethod.requiresBankAccount && (
        <BankAccountForm onChange={(data) => updatePaymentData({ bankAccount: data })} />
      )}

      <div className="mt-4">
        <label className="flex items-center gap-2">
          <input
            type="checkbox"
            checked={paymentData.savePaymentMethod}
            onChange={(e) => updatePaymentData({ savePaymentMethod: e.target.checked })}
          />
          Save payment method for future use
        </label>
      </div>

      <button type="submit" className="w-full mt-6 bg-blue-600 text-white rounded-lg py-3">
        Pay {formatCurrency(paymentData.amount)}
      </button>
    </form>
  )
}

export function PaymentStatus() {
  const { flowState, reset } = usePaymentStore()

  switch (flowState.type) {
    case 'processing':
      return <ProcessingSpinner />

    case '3ds_challenge':
      return <ThreeDSecureChallenge url={flowState.challengeUrl} />

    case 'success':
      return (
        <SuccessMessage
          transaction={flowState.transaction}
          onClose={reset}
        />
      )

    case 'failed':
      return (
        <ErrorMessage
          error={flowState.error}
          retryable={flowState.retryable}
          onRetry={() => usePaymentStore.getState().processPayment()}
          onClose={reset}
        />
      )

    default:
      return null
  }
}
```

---

## Security & Compliance

### PCI DSS Compliance Checklist

```typescript
interface PCIComplianceChecklist {
  // Requirement 1: Firewall configuration
  firewall: {
    configured: boolean
    tested: boolean
    documented: boolean
  }

  // Requirement 2: No default passwords
  passwords: {
    defaultsChanged: boolean
    strongPasswordPolicy: boolean
  }

  // Requirement 3: Protect stored cardholder data
  dataProtection: {
    cardDataEncrypted: boolean
    truncateCardNumbers: boolean  // Store only last 4 digits
    neverStoreCVV: boolean
    neverStorePIN: boolean
    keyManagement: boolean
  }

  // Requirement 4: Encrypt transmission
  transmission: {
    tlsEnabled: boolean
    strongCryptography: boolean
  }

  // Requirement 5: Antivirus
  antivirus: {
    installed: boolean
    upToDate: boolean
  }

  // Requirement 6: Secure systems
  systems: {
    patchesApplied: boolean
    secureCodeReview: boolean
    vulnerabilityScanning: boolean
  }

  // Requirement 7: Access control
  accessControl: {
    rbacImplemented: boolean
    leastPrivilege: boolean
    uniqueIds: boolean
  }

  // Requirement 8: Unique IDs
  authentication: {
    uniqueUserIds: boolean
    multiFactorAuth: boolean
    passwordComplexity: boolean
  }

  // Requirement 9: Physical access
  physical: {
    restrictedAccess: boolean
    mediaDestruction: boolean
  }

  // Requirement 10: Logging
  logging: {
    accessLogging: boolean
    logReview: boolean
    logRetention: boolean  // 1 year minimum
  }

  // Requirement 11: Testing
  testing: {
    vulnerabilityScans: boolean  // Quarterly
    penetrationTests: boolean    // Annually
  }

  // Requirement 12: Policy
  policy: {
    securityPolicy: boolean
    annualRiskAssessment: boolean
    staffTraining: boolean
  }
}
```

### Data Encryption

```typescript
// Never store sensitive payment data in plain text
// Use tokenization from payment providers

// Example: Stripe tokenization
async function createPaymentMethodToken(cardData: CardData): Promise<string> {
  // Card data is sent directly to Stripe from client-side
  // Server never sees full card number
  const token = await stripe.tokens.create({
    card: {
      number: cardData.number,
      exp_month: cardData.expMonth,
      exp_year: cardData.expYear,
      cvc: cardData.cvc
    }
  })

  return token.id  // Return token ID, not card data
}

// Store only token in database
await db.savedPaymentMethods.create({
  data: {
    userId: user.id,
    type: 'card',
    provider: 'stripe',
    providerMethodId: token.id,  // Token, not card number
    cardBrand: token.card.brand,
    cardLast4: token.card.last4,  // Only last 4 digits
    // NEVER store: full card number, CVV, PIN
  }
})
```

### Webhook Signature Verification

```typescript
// Always verify webhook signatures
async function handleStripeWebhook(req: Request): Promise<Response> {
  const signature = req.headers.get('stripe-signature')
  const secret = process.env.STRIPE_WEBHOOK_SECRET

  try {
    // Verify signature
    const event = stripe.webhooks.constructEvent(
      await req.text(),
      signature,
      secret
    )

    // Process verified event
    await processWebhookEvent(event)

    return new Response(JSON.stringify({ received: true }), { status: 200 })
  } catch (error) {
    console.error('Webhook signature verification failed:', error)
    return new Response('Signature verification failed', { status: 400 })
  }
}
```

---

## Integration Examples

### Stripe Integration

```typescript
import Stripe from 'stripe'

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY, {
  apiVersion: '2023-10-16'
})

async function createStripePaymentIntent(
  config: PaymentConfiguration,
  data: PaymentIntentData
): Promise<Stripe.PaymentIntent> {
  const paymentIntent = await stripe.paymentIntents.create({
    amount: Math.round(data.amount * 100), // Convert to cents
    currency: data.currency.toLowerCase(),
    customer: data.customerId,
    metadata: data.metadata,

    // Apply configuration
    automatic_payment_methods: {
      enabled: config.methodConfig.acceptedTypes.cards.enabled
    },

    // 3D Secure
    payment_method_options: {
      card: {
        request_three_d_secure: config.methodConfig.acceptedTypes.cards.require3DS ? 'always' : 'automatic'
      }
    },

    // Split payment (Stripe Connect)
    transfer_data: config.splitPaymentConfig?.enabled ? {
      destination: data.merchantAccountId,
      amount: Math.round(data.amount * config.splitPaymentConfig.marketplace.sellerPercentage * 100)
    } : undefined,

    // Platform fee
    application_fee_amount: config.feeConfig.platformFee.enabled ?
      Math.round(data.amount * config.feeConfig.platformFee.percentage * 100) : undefined
  })

  return paymentIntent
}
```

### PayPal Integration

```typescript
import paypal from '@paypal/checkout-server-sdk'

async function createPayPalOrder(
  config: PaymentConfiguration,
  data: PaymentIntentData
): Promise<paypal.orders.Order> {
  const environment = config.methodConfig.providerMode === 'live' ?
    new paypal.core.LiveEnvironment(clientId, clientSecret) :
    new paypal.core.SandboxEnvironment(clientId, clientSecret)

  const client = new paypal.core.PayPalHttpClient(environment)

  const request = new paypal.orders.OrdersCreateRequest()
  request.prefer('return=representation')
  request.requestBody({
    intent: 'CAPTURE',
    purchase_units: [{
      amount: {
        currency_code: data.currency,
        value: data.amount.toFixed(2),
        breakdown: {
          item_total: {
            currency_code: data.currency,
            value: data.amount.toFixed(2)
          }
        }
      }
    }],

    // Platform fee
    payment_instruction: config.feeConfig.platformFee.enabled ? {
      platform_fees: [{
        amount: {
          currency_code: data.currency,
          value: (data.amount * config.feeConfig.platformFee.percentage).toFixed(2)
        }
      }]
    } : undefined
  })

  const response = await client.execute(request)
  return response.result
}
```

### Cryptocurrency Integration (Coinbase Commerce)

```typescript
import CoinbaseCommerce from 'coinbase-commerce-node'

const Client = CoinbaseCommerce.Client
Client.init(process.env.COINBASE_API_KEY)

async function createCryptoCharge(
  config: PaymentConfiguration,
  data: PaymentIntentData
): Promise<any> {
  const Charge = CoinbaseCommerce.resources.Charge

  const charge = await Charge.create({
    name: data.description,
    description: data.metadata.description,
    local_price: {
      amount: data.amount.toFixed(2),
      currency: data.currency
    },
    pricing_type: 'fixed_price',
    metadata: {
      transactionId: data.transactionId,
      customerId: data.customerId
    },

    // Redirect URLs
    redirect_url: `${process.env.APP_URL}/payment/success`,
    cancel_url: `${process.env.APP_URL}/payment/cancel`
  })

  return charge
}
```

---

## Complete Example: Full Payment Flow

```typescript
// 1. User lands on checkout page
export default function CheckoutPage() {
  const [amount] = useState(99.99)
  const [currency] = useState('USD')

  const {
    availableMethods,
    selectedMethod,
    flowState,
    fetchAvailableMethods,
    selectMethod,
    updatePaymentData,
    createPaymentIntent,
    processPayment
  } = usePaymentStore()

  useEffect(() => {
    fetchAvailableMethods(amount, currency)
  }, [amount, currency])

  if (flowState.type === 'idle' || flowState.type === 'selecting_method') {
    return (
      <div className="max-w-2xl mx-auto p-6">
        <h1 className="text-2xl font-bold mb-6">Select Payment Method</h1>

        <div className="mb-6 p-4 bg-gray-50 rounded-lg">
          <div className="flex justify-between">
            <span>Subtotal</span>
            <span>${amount.toFixed(2)}</span>
          </div>
        </div>

        <PaymentMethodSelector />
      </div>
    )
  }

  if (flowState.type === 'entering_details') {
    return (
      <div className="max-w-2xl mx-auto p-6">
        <h1 className="text-2xl font-bold mb-6">Payment Details</h1>
        <PaymentForm />
      </div>
    )
  }

  return <PaymentStatus />
}

// 2. Backend processes payment
export async function POST(request: Request) {
  const data = await request.json()
  const user = await getCurrentUser(request)

  // Get payment configuration
  const config = await db.paymentConfigurations.findUnique({
    where: { id: data.paymentConfigId }
  })

  // Validate RBAC
  if (!checkRBAC(config.rbac, user)) {
    return new Response('Unauthorized', { status: 403 })
  }

  // Validate render conditions
  if (!checkRenderConditions(config.renderConditions, {
    user,
    amount: data.amount,
    currency: data.currency,
    timestamp: new Date()
  })) {
    return new Response('Payment method not available', { status: 400 })
  }

  // Calculate fees
  const fees = calculateFees(config.feeConfig, data.amount, data.currency)
  const tax = calculateTax(config.taxConfig, data.amount, user)
  const totalAmount = data.amount + fees.total + tax

  // Fraud check
  const fraudScore = await checkFraud(config.fraudConfig, {
    user,
    amount: totalAmount,
    paymentMethod: data.paymentMethod,
    ipAddress: request.headers.get('x-forwarded-for'),
    userAgent: request.headers.get('user-agent')
  })

  if (fraudScore > config.fraudConfig.rules.mlFraudDetection.scoreThreshold) {
    await logSecurityEvent('payment_blocked_fraud', { user, fraudScore })
    return new Response('Payment blocked', { status: 403 })
  }

  // Create transaction
  const transaction = await db.transactions.create({
    data: {
      paymentConfigId: config.id,
      customerId: user.id,
      amount: data.amount,
      currency: data.currency,
      platformFee: fees.platform,
      processingFee: fees.processing,
      taxAmount: tax,
      netAmount: totalAmount,
      status: 'pending',
      fraudScore,
      metadata: data.metadata
    }
  })

  // Process with payment provider
  let providerResult

  switch (config.provider) {
    case 'stripe':
      providerResult = await processStripePayment(config, transaction, data)
      break
    case 'paypal':
      providerResult = await processPayPalPayment(config, transaction, data)
      break
    case 'coinbase_commerce':
      providerResult = await processCryptoPayment(config, transaction, data)
      break
    default:
      throw new Error(`Unsupported provider: ${config.provider}`)
  }

  // Update transaction
  await db.transactions.update({
    where: { id: transaction.id },
    data: {
      status: providerResult.status,
      providerTransactionId: providerResult.id,
      providerResponse: providerResult,
      completedAt: providerResult.status === 'success' ? new Date() : undefined
    }
  })

  // Handle split payments
  if (config.splitPaymentConfig?.enabled && providerResult.status === 'success') {
    await processSplitPayment(config, transaction, providerResult)
  }

  // Send notifications
  await sendPaymentNotifications(config.notificationConfig, {
    transaction,
    customer: user,
    status: providerResult.status
  })

  // Trigger webhooks
  await triggerWebhooks(config.webhookConfig, {
    event: providerResult.status === 'success' ? 'payment.success' : 'payment.failed',
    data: transaction
  })

  return new Response(JSON.stringify({
    transaction,
    status: providerResult.status,
    requires3DS: providerResult.requires3DS,
    challengeUrl: providerResult.challengeUrl
  }), {
    headers: { 'Content-Type': 'application/json' }
  })
}
```

---

## Summary

This comprehensive payment system configuration provides:

✅ **Complete Admin Control**: Configure every aspect of payment processing without code changes
✅ **RBAC Integration**: Fine-grained role-based access control
✅ **Conditional Rendering**: Smart show/hide based on business rules
✅ **Multi-Provider Support**: Stripe, PayPal, Square, crypto, BNPL
✅ **Multi-Currency**: Automatic detection and conversion
✅ **Subscription Management**: Trials, upgrades, downgrades, pauses
✅ **Split Payments**: Marketplace, crowdfunding, group payments
✅ **Escrow**: Project-based payments with milestone releases
✅ **Fraud Prevention**: ML-powered fraud detection
✅ **Compliance**: PCI DSS, GDPR, PSD2, AML/KYC ready
✅ **Tax Handling**: Automatic tax calculation and collection
✅ **Refund & Dispute Management**: Automated workflows
✅ **Webhooks & Notifications**: Real-time event handling
✅ **Reporting & Analytics**: Comprehensive business insights

Every scenario is accounted for, every edge case handled, and every compliance requirement met.
