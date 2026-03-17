# Comprehensive MCP & Integrations System Configuration Flow

> **Foundation wiring note:** Treat this as shared capability/integration guidance for `GeneralApp`. Provider choices must stay behind foundation adapters and capability bindings rather than leaking into module contracts.

## Table of Contents
1. [System Overview](#system-overview)
2. [Universal Integration Configuration Pattern](#universal-integration-configuration-pattern)
3. [MCP (Model Context Protocol) System](#mcp-model-context-protocol-system)
4. [AI Provider Configuration](#ai-provider-configuration)
5. [AI Capabilities Registry](#ai-capabilities-registry)
6. [MCP Permissions & Access Control](#mcp-permissions--access-control)
7. [AI Usage Tracking & Limits](#ai-usage-tracking--limits)
8. [Integration Categories](#integration-categories)
9. [Third-Party Integrations](#third-party-integrations)
10. [API Access & Webhooks](#api-access--webhooks)
11. [SSO & Identity Providers](#sso--identity-providers)
12. [Email & Communication Integrations](#email--communication-integrations)
13. [Storage & Data Integrations](#storage--data-integrations)
14. [Analytics & Monitoring Integrations](#analytics--monitoring-integrations)
15. [Custom Integration Builder](#custom-integration-builder)
16. [Integration Marketplace](#integration-marketplace)
17. [Admin Integration Dashboard](#admin-integration-dashboard)
18. [User Integration Experience](#user-integration-experience)
19. [Integration API Architecture](#integration-api-architecture)
20. [Database Schema](#database-schema)
21. [Security & Encryption](#security--encryption)
22. [Monitoring & Health Checks](#monitoring--health-checks)
23. [Audit & Compliance](#audit--compliance)
24. [Edge Cases & Conflict Resolution](#edge-cases--conflict-resolution)
25. [Performance & Rate Limiting](#performance--rate-limiting)
26. [Migration & Versioning](#migration--versioning)

---

## System Overview

### Core Principles
1. **Unified Integration Hub**: All integrations (AI, third-party, webhooks, SSO) managed from one place
2. **Provider Abstraction**: Consistent API across different AI providers and integrations
3. **Granular Access Control**: Per-capability, per-role, per-user MCP permissions
4. **Usage Transparency**: Full tracking of all API calls, costs, and usage
5. **Security-First**: All sensitive data encrypted, audit logged, access controlled
6. **Self-Service**: Users can configure personal AI providers; admins configure org-wide
7. **Fallback & Resilience**: Automatic failover, retry logic, circuit breakers
8. **Real-Time Sync**: Integration status, health, and usage updated in real-time

### Integration Architecture
```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Integration Gateway                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │   AI/MCP     │  │  Third-Party │  │   Webhooks  │  │    SSO    │ │
│  │   Providers  │  │  Integrations│  │   Events    │  │ Providers │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘ │
│         │                 │                  │                │          │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐  ┌─┴─────────┐ │
│  │    MCP       │  │  Integration │  │   Event      │  │   Auth    │ │
│  │  Registry    │  │   Manager    │  │   Bus        │  │  Provider │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘  └───────────┘ │
│         │                 │                  │                         │
│  ┌──────┴─────────────────┴──────────────────┴────────────────────────┐ │
│  │                     Access Control Layer                             │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────┐  │ │
│  │  │   Role     │  │  Capability  │  │   Rate      │  │  Quota   │  │ │
│  │  │  Checks    │  │   Checks    │  │  Limiting   │  │  Limits  │  │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └──────────┘  │ │
│  └────────────────────────────────────────────────────────────────────┘ │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐  │
│  │                    Monitoring & Audit Layer                         │  │
│  │  Usage Tracking • Cost Analytics • Health Checks • Audit Logs     │  │
│  └────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Universal Integration Configuration Pattern

Every integration follows this universal structure:

```typescript
interface IntegrationConfiguration {
  // ============================================
  // CORE IDENTITY
  // ============================================
  id: string                              // "integration-ai-providers"
  name: string                            // "AI & MCP Providers"
  category: IntegrationCategory            // ai | third_party | webhook | sso | email | storage | analytics
  type: IntegrationType                   // provider | connector | gateway | marketplace

  // Enable/Disable
  enabled: boolean                        // Master switch
  maintenanceMode: boolean               // Lock during maintenance

  // ============================================
  // AI/MCP CONFIGURATION
  // ============================================
  aiConfig: {
    // Supported providers
    providers: {
      openai: ProviderConfig
      anthropic: ProviderConfig
      google: ProviderConfig
      cohere: ProviderConfig
      mistral: ProviderConfig
      custom: ProviderConfig
    }

    // Default provider
    defaultProvider: string
    fallbackProviders: string[]          // Failover order

    // Model configuration
    models: {
      provider: string
      modelId: string
      displayName: string
      maxTokens: number
      supportsStreaming: boolean
      supportsFunctionCalling: boolean
      inputCostPer1K: number            // dollars
      outputCostPer1K: number
    }[]

    // MCP servers
    mcpServers: {
      id: string
      name: string
      type: MCPType
      url: string
      auth: MCPAuthConfig
      capabilities: string[]
      enabled: boolean
    }[]

    // Usage limits
    usageLimits: {
      maxRequestsPerMinute: number
      maxTokensPerRequest: number
      maxConcurrentRequests: number
      dailyCreditLimit: number
      monthlyCreditLimit: number
      perUserLimits: {
        maxRequestsPerDay: number
        maxCreditsPerDay: number
      }
    }

    // Security
    security: {
      encryptApiKeys: boolean
      allowUserApiKeys: boolean          // Users can add personal keys
      allowPersonalProviders: boolean
      auditAllRequests: boolean
      logPrompts: boolean
      logResponses: boolean
      retainLogsDays: number
      piiDetection: boolean
      contentFiltering: boolean
    }
  }

  // ============================================
  // THIRD-PARTY INTEGRATIONS CONFIGURATION
  // ============================================
  thirdPartyConfig: {
    // Available integrations
    integrations: {
      slack: IntegrationConfig
      github: IntegrationConfig
      jira: IntegrationConfig
      figma: IntegrationConfig
      miro: IntegrationConfig
      notion: IntegrationConfig
      zapier: IntegrationConfig
      custom: IntegrationConfig
    }

    // OAuth settings
    oauth: {
      enabled: boolean
      clientId: string
      clientSecret: string              // Encrypted
      redirectUris: string[]
      scopes: string[]
      autoProvision: boolean            // Auto-create user on OAuth
      enforceDomains: string[]          // Restrict to email domains
    }

    // API key settings
    apiKeys: {
      enabled: boolean
      maxKeysPerOrg: number
      maxKeysPerUser: number
      requireDescription: boolean
      expiryDays: number
      rotationReminderDays: number
    }

    // Webhook settings
    webhooks: {
      enabled: boolean
      maxWebhooksPerOrg: number
      secretSigning: boolean
      retryPolicy: {
        maxRetries: number
        retryDelays: number[]           // [1, 5, 30, 300] seconds
        timeoutSeconds: number
      }
      deliveryLogging: boolean
      retentionDays: number
    }
  }

  // ============================================
  // SSO CONFIGURATION
  // ============================================
  ssoConfig: {
    // SSO providers
    providers: {
      saml: SSOProviderConfig
      oidc: SSOProviderConfig
      ldap: SSOProviderConfig
    }

    // Global SSO settings
    global: {
      enforceForDomains: string[]       // Force SSO for these domains
      allowPasswordFallback: boolean   // Allow password login alongside SSO
      defaultRole: string              // Role for new SSO users
      autoProvision: boolean
      autoDeprovision: boolean        // Remove from org when removed from IdP
      jitProvisioning: boolean         // Just-in-time user creation
      sessionTimeout: number           // SSO session duration
      mfaIntegration: boolean         // Use IdP MFA
    }
  }

  // ============================================
  // EMAIL INTEGRATIONS
  // ============================================
  emailConfig: {
    // Email providers
    providers: {
      smtp: EmailProviderConfig
      sendgrid: EmailProviderConfig
      resend: EmailProviderConfig
      ses: EmailProviderConfig
      postmark: EmailProviderConfig
      mailgun: EmailProviderConfig
    }

    // Email settings
    settings: {
      defaultFromName: string
      defaultFromEmail: string
      replyToEmail: string
      bccAddress: string               // BCC all outgoing emails
      maxRecipients: number
      maxEmailsPerHour: number
      maxEmailsPerDay: number
    }

    // Tracking
    tracking: {
      openTracking: boolean
      clickTracking: boolean
      bounceHandling: boolean
      complaintHandling: boolean
    }
  }

  // ============================================
  // STORAGE INTEGRATIONS
  // ============================================
  storageConfig: {
    // Storage providers
    providers: {
      local: StorageProviderConfig
      s3: StorageProviderConfig
      gcs: StorageProviderConfig
      azureBlob: StorageProviderConfig
      cloudflareR2: StorageProviderConfig
    }

    // Storage settings
    settings: {
      defaultProvider: string
      maxUploadSize: number            // MB
      allowedFileTypes: string[]
      virusScanEnabled: boolean
      encryptAtRest: boolean
      compressUploads: boolean
      cdnEnabled: boolean
      cdnUrl: string
    }
  }

  // ============================================
  // ANALYTICS & MONITORING
  // ============================================
  analyticsConfig: {
    // Analytics providers
    providers: {
      googleAnalytics: AnalyticsProviderConfig
      mixpanel: AnalyticsProviderConfig
      amplitude: AnalyticsProviderConfig
      segment: AnalyticsProviderConfig
      custom: AnalyticsProviderConfig
    }

    // Monitoring providers
    monitoring: {
      datadog: MonitoringProviderConfig
      newrelic: MonitoringProviderConfig
      sentry: MonitoringProviderConfig
      prometheus: MonitoringProviderConfig
    }

    // Event tracking
    events: {
      trackPageViews: boolean
      trackEvents: boolean
      trackErrors: boolean
      trackPerformance: boolean
      sampleRate: number               // 0-1
    }
  }

  // ============================================
  // RATE LIMITING & QUOTAS
  // ============================================
  rateLimiting: {
    enabled: boolean
    global: {
      requestsPerMinute: number
      requestsPerHour: number
      requestsPerDay: number
      burstLimit: number
    }
    perProvider: {
      [provider: string]: {
        requestsPerMinute: number
        requestsPerHour: number
        concurrentLimit: number
      }
    }
    perUser: {
      requestsPerMinute: number
      requestsPerHour: number
      dailyLimit: number
    }
    backoffStrategy: {
      initialDelay: number             // ms
      maxDelay: number                 // ms
      multiplier: number
    }
  }

  // ============================================
  // PLAN-BASED RESTRICTIONS
  // ============================================
  planRestrictions: {
    enabled: boolean
    plans: {
      FREE: {
        aiProviders: {
          openai: false
          anthropic: false
          google: false
          custom: false
        }
        mcpServers: 0
        thirdPartyIntegrations: 0
        webhooks: 0
        customDomain: false
        sso: false
        prioritySupport: false
      }
      STARTER: {
        aiProviders: {
          openai: true
          anthropic: false
          google: false
          custom: false
        }
        mcpServers: 1
        thirdPartyIntegrations: 1
        webhooks: 2
        customDomain: false
        sso: false
        prioritySupport: false
      }
      PRO: {
        aiProviders: {
          openai: true
          anthropic: true
          google: true
          custom: true
        }
        mcpServers: 5
        thirdPartyIntegrations: 5
        webhooks: 10
        customDomain: true
        sso: false
        prioritySupport: false
      }
      ENTERPRISE: {
        aiProviders: {
          openai: true
          anthropic: true
          google: true
          custom: true
        }
        mcpServers: -1                    // Unlimited
        thirdPartyIntegrations: -1
        webhooks: -1
        customDomain: true
        sso: true
        prioritySupport: true
      }
    }
  }

  // ============================================
  // VISIBILITY & ACCESS
  // ============================================
  visibility: {
    visibleToRoles: string[]
    configurableByRoles: string[]
    adminOnlyCategories: string[]       // ["sso", "system"]
    userConfigurableCategories: string[] // ["personal_ai", "personal_integrations"]
    requireApprovalFor: string[]        // Categories requiring admin approval
  }
}
```

---

## MCP (Model Context Protocol) System

### MCP Server Configuration

```typescript
interface MCPConfiguration {
  // ============================================
  // MCP SERVER IDENTITY
  // ============================================
  id: string                              // "mcp-clx-xxx"
  name: string                            // "Production AI Server"
  type: MCPType                            // "openai" | "anthropic" | "google" | "custom_mcp" | "sse" | "stdio"

  // ============================================
  // CONNECTION CONFIGURATION
  // ============================================
  connection: {
    // For HTTP/SSE endpoints
    url: string                          // "https://api.openai.com/v1"
    baseUrl?: string                    // Override base URL
    
    // For stdio processes
    command: string                     // "node"
    args: string[]                      // ["server.js"]
    env: Record<string, string>         // Environment variables
    
    // Authentication
    auth: {
      type: "none" | "api_key" | "bearer" | "basic" | "oauth2"
      apiKey?: string                   // Encrypted
      bearerToken?: string              // Encrypted
      basic?: {
        username: string
        password: string                // Encrypted
      }
      oauth2?: {
        clientId: string
        clientSecret: string           // Encrypted
        tokenUrl: string
        scopes: string[]
      }
    }

    // Headers
    headers: Record<string, string>     // Custom headers
    timeout: number                      // milliseconds
    retries: {
      enabled: boolean
      maxAttempts: number
      backoffMs: number
    }
  }

  // ============================================
  // CAPABILITIES
  // ============================================
  capabilities: {
    // AI Models available
    models: {
      id: string
      name: string
      maxTokens: number
      supportsStreaming: boolean
      supportsFunctionCalling: boolean
      contextWindow: number
    }[]

    // Tools available
    tools: {
      id: string
      name: string
      description: string
      inputSchema: object
      category: string
    }[]

    // Resources available
    resources: {
      id: string
      name: string
      mimeType: string
      uri: string
    }[]

    // Prompts available
    prompts: {
      id: string
      name: string
      description: string
      arguments: object
    }[]
  }

  // ============================================
  // HEALTH & MONITORING
  // ============================================
  health: {
    status: "connected" | "disconnected" | "error" | "connecting"
    lastConnectedAt: Date
    lastError: string
    lastErrorAt: Date
    healthCheckInterval: number          // seconds
    autoReconnect: boolean
    reconnectDelay: number               // seconds
  }

  // ============================================
  // USAGE TRACKING
  // ============================================
  usage: {
    connectCount: number
    lastUsedAt: Date
    totalRequests: number
    totalTokens: number
    totalCost: number
    averageLatency: number               // ms
  }

  // ============================================
  // ACCESS CONTROL
  // ============================================
  access: {
    isDefault: boolean                   // Default MCP for this type
    isActive: boolean
    allowedRoles: string[]              // Roles that can use this MCP
    allowedUsers: string[]               // Specific users
    blockedRoles: string[]
    rateLimit: number                    // requests per minute
    dailyBudget: number                  // max cost per day
  }

  // ============================================
  // SCOPING
  // ============================================
  scope: {
    organizationId: string              // Org-level MCP
    userId?: string                    // Personal MCP (user-level)
    teamId?: string                    // Team-level MCP
  }
}
```

### MCP Server Scenarios

```typescript
// ============================================
// SCENARIO 1: Admin Adds OpenAI MCP Server
// ============================================
// Route: POST /api/admin/mcp/servers
{
  name: "Production OpenAI",
  type: "openai",
  connection: {
    url: "https://api.openai.com/v1",
    auth: {
      type: "bearer",
      apiKey: "sk-xxx..."              // Will be encrypted
    },
    timeout: 60000,
    retries: {
      enabled: true,
      maxAttempts: 3,
      backoffMs: 1000
    }
  },
  capabilities: {
    models: [
      { id: "gpt-4-turbo", name: "GPT-4 Turbo", maxTokens: 128000, supportsStreaming: true },
      { id: "gpt-3.5-turbo", name: "GPT-3.5 Turbo", maxTokens: 16385, supportsStreaming: true }
    ],
    tools: [
      { id: "code_execution", name: "Code Execution", category: "development" },
      { id: "web_search", name: "Web Search", category: "research" }
    ]
  },
  access: {
    isDefault: true,
    allowedRoles: ["OWNER", "ADMIN", "MEMBER"],
    rateLimit: 60
  }
}

// Response:
{
  server: {
    id: "mcp-clx-xxx",
    name: "Production OpenAI",
    status: "connected",
    lastConnectedAt: "2026-02-15T15:00:00Z",
    isDefault: true
  }
}

// ============================================
// SCENARIO 2: User Adds Personal Anthropic MCP
// ============================================
// Route: POST /api/user/mcp/servers
{
  name: "My Claude API",
  type: "anthropic",
  connection: {
    url: "https://api.anthropic.com",
    auth: {
      type: "api_key",
      apiKey: "sk-ant-xxx..."          // User's personal key
    }
  },
  isDefault: false
}

// Response:
{
  server: {
    id: "mcp-clx-user-xxx",
    name: "My Claude API",
    status: "connected",
    scope: "personal"
  }
}

// ============================================
// SCENARIO 3: Admin Configures Custom MCP Server (SSE)
// ============================================
// Route: POST /api/admin/mcp/servers
{
  name: "Internal Code Assistant",
  type: "sse",
  connection: {
    url: "https://code-assistant.internal.company.com/mcp",
    auth: {
      type: "bearer",
      bearerToken: "internal-token"
    },
    headers: {
      "X-Company-ID": "acme-corp"
    },
    timeout: 30000
  },
  capabilities: {
    models: [
      { id: "codeassist-7b", name: "CodeAssist 7B", maxTokens: 8192, supportsStreaming: true }
    ],
    tools: [
      { id: "file_read", name: "Read File", category: "filesystem" },
      { id: "file_write", name: "Write File", category: "filesystem" },
      { id: "git_command", name: "Git Command", category: "vcs" },
      { id: "bash_command", name: "Bash Command", category: "shell" }
    ]
  },
  access: {
    allowedRoles: ["OWNER", "ADMIN"],
    rateLimit: 10
  }
}

// ============================================
// SCENARIO 4: Admin Connects to Stdio-Based MCP
// ============================================
// Route: POST /api/admin/mcp/servers
{
  name: "Local Code Analysis",
  type: "stdio",
  connection: {
    command: "/usr/local/bin/code-analyzer",
    args: ["--port", "8080", "--verbose"],
    env: {
      "LOG_LEVEL": "info",
      "CACHE_DIR": "/tmp/analyzer-cache"
    }
  },
  capabilities: {
    tools: [
      { id: "analyze_code", name: "Analyze Code", category: "analysis" },
      { id: "find_bugs", name: "Find Bugs", category: "analysis" },
      { id: "suggest_refactor", name: "Suggest Refactoring", category: "improvement" }
    ]
  },
  health: {
    healthCheckInterval: 60,
    autoReconnect: true,
    reconnectDelay: 5000
  }
}

// ============================================
// SCENARIO 5: Test MCP Connection
// ============================================
// Route: POST /api/admin/mcp/servers/:id/test
{
  testPrompt: "What is 2 + 2?",
  model: "gpt-4-turbo"
}

// Response:
{
  success: true,
  response: "2 + 2 equals 4.",
  latency: 1250,
  tokensUsed: 25,
  cost: 0.0001
}

// Or if failed:
{
  success: false,
  error: "Authentication failed",
  errorCode: "INVALID_API_KEY",
  suggestion: "Check your API key and ensure it has the correct permissions"
}

// ============================================
// SCENARIO 6: Set MCP as Default
// ============================================
// Route: POST /api/admin/mcp/servers/:id/set-default
{
  // Previous default (if any) is no longer default
}

// ============================================
// SCENARIO 7: Delete MCP Server
// ============================================
// Route: DELETE /api/admin/mcp/servers/:id
// Validation: Cannot delete if it's the only server of its type
{
  success: true,
  reassignUsageTo: "mcp-other-server-id",  // Usage migrated to this server
  affectedUsers: 15
}

// ============================================
// SCENARIO 8: View MCP Usage Analytics
// ============================================
// Route: GET /api/admin/mcp/servers/:id/usage
// Query: ?period=7d&granularity=daily

// Response:
{
  server: { id: "mcp-xxx", name: "Production OpenAI" },
  period: "7d",
  metrics: {
    totalRequests: 15420,
    totalTokens: {
      input: 12500000,
      output: 8500000
    },
    totalCost: 145.50,
    averageLatency: 890,
    successRate: 99.7,
    errorRate: 0.3,
    topModels: [
      { model: "gpt-4-turbo", requests: 12000, cost: 120.00 },
      { model: "gpt-3.5-turbo", requests: 3420, cost: 25.50 }
    ],
    topUsers: [
      { userId: "user-1", requests: 5000, cost: 50.00 },
      { userId: "user-2", requests: 3200, cost: 35.00 }
    ],
    dailyBreakdown: [
      { date: "2026-02-15", requests: 2200, cost: 20.50 },
      { date: "2026-02-14", requests: 2100, cost: 19.00 }
    ]
  }
}
```

---

## AI Provider Configuration

### Provider Configuration Interface

```typescript
interface AIProviderConfiguration {
  // ============================================
  // PROVIDER IDENTITY
  // ============================================
  provider: "openai" | "anthropic" | "google" | "cohere" | "mistral" | "custom"

  // ============================================
  // CONNECTION SETTINGS
  // ============================================
  connection: {
    // API Configuration
    apiKey: string                      // Encrypted at rest
    organizationId?: string              // For OpenAI org-level access

    // Endpoint Override (for custom proxies)
    baseUrl?: string                    // "https://api.openai.com/v1"
    proxyUrl?: string                   // Custom proxy URL

    // Model Defaults
    defaultModel: string
    defaultMaxTokens: number
    defaultTemperature: number
    defaultTopP: number

    // Advanced
    apiVersion?: string                  // For providers with versioning
    extraHeaders?: Record<string, string>
  }

  // ============================================
  // MODEL CONFIGURATION
  // ============================================
  models: {
    // Available models for this provider
    available: {
      id: string
      name: string
      enabled: boolean
      maxTokens: number
      contextWindow: number
      supportsStreaming: boolean
      supportsFunctionCalling: boolean
      supportsVision: boolean
      inputCostPer1K: number
      outputCostPer1K: number
      cacheCostPer1K?: number           // Cached input cost
    }[]

    // Model-specific overrides
    overrides: {
      [modelId: string]: {
        maxTokens?: number
        temperature?: number
        topP?: number
        enabled?: boolean
      }
    }
  }

  // ============================================
  // SAFETY & CONTENT FILTERING
  // ============================================
  safety: {
    enabled: boolean
    contentFilters: {
      blockPII: boolean
      blockProfanity: boolean
      blockHarmfulContent: boolean
      customBlockPatterns: string[]      // Regex patterns
    }
    moderation: {
      enabled: boolean
      provider: "openai" | "custom"
      blockOnFlag: boolean
      flagThreshold: number             // 0-1 threshold
    }
    outputFiltering: {
      removePII: boolean
      removePersonalInfo: boolean
      redactPhoneNumbers: boolean
      redactEmails: boolean
      redactSSN: boolean
    }
  }

  // ============================================
  // LOGGING & AUDIT
  // ============================================
  logging: {
    logRequests: boolean
    logResponses: boolean
    logTokens: boolean
    logCosts: boolean
    logLatency: boolean
    retainDays: number
    includeSystemPrompts: boolean
    includeFunctionCalls: boolean
    includeToolResults: boolean
    sampleRate: number                   // 0-1, for sampling
  }

  // ============================================
  // RATE LIMITS
  // ============================================
  rateLimits: {
    requestsPerMinute: number
    requestsPerHour: number
    tokensPerMinute: number
    tokensPerDay: number
    concurrentRequests: number
    burstLimit: number
  }
}
```

### Provider Scenarios

```typescript
// ============================================
// SCENARIO 1: Admin Configures OpenAI Provider
// ============================================
// Route: POST /api/admin/ai/providers
{
  provider: "openai",
  connection: {
    apiKey: "sk-xxx...",
    organizationId: "org-xxx",           // Optional
    baseUrl: "https://api.openai.com/v1",
    defaultModel: "gpt-4-turbo",
    defaultMaxTokens: 4000,
    defaultTemperature: 0.7,
    defaultTopP: 1.0
  },
  models: {
    available: [
      { id: "gpt-4-turbo", enabled: true, maxTokens: 128000, inputCostPer1K: 0.01, outputCostPer1K: 0.03 },
      { id: "gpt-4", enabled: true, maxTokens: 8192, inputCostPer1K: 0.03, outputCostPer1K: 0.06 },
      { id: "gpt-3.5-turbo", enabled: true, maxTokens: 16385, inputCostPer1K: 0.0005, outputCostPer1K: 0.0015 }
    ]
  },
  safety: {
    enabled: true,
    contentFilters: {
      blockHarmfulContent: true,
      customBlockPatterns: ["regex-pattern-1"]
    }
  },
  logging: {
    logRequests: true,
    logResponses: false,               // Save costs, still log metadata
    logTokens: true,
    logCosts: true,
    retainDays: 90
  },
  rateLimits: {
    requestsPerMinute: 60,
    tokensPerMinute: 90000
  }
}

// ============================================
// SCENARIO 2: User Adds Personal OpenAI Key
// ============================================
// Route: POST /api/user/ai/providers
{
  provider: "openai",
  connection: {
    apiKey: "sk-personal-xxx...",
    defaultModel: "gpt-3.5-turbo"
  },
  name: "My OpenAI Account",
  isDefault: true
}
// Note: Personal providers bypass org rate limits but count toward user's quota

// ============================================
// SCENARIO 3: Admin Configures Anthropic Provider
// ============================================
// Route: POST /api/admin/ai/providers
{
  provider: "anthropic",
  connection: {
    apiKey: "sk-ant-xxx...",
    defaultModel: "claude-3-5-sonnet-20241022",
    defaultMaxTokens: 4096,
    defaultTemperature: 0.7
  },
  models: {
    available: [
      { id: "claude-3-5-sonnet-20241022", enabled: true, maxTokens: 200000, inputCostPer1K: 0.003, outputCostPer1K: 0.015 },
      { id: "claude-3-opus-20240229", enabled: false, maxTokens: 200000, inputCostPer1K: 0.015, outputCostPer1K: 0.075 },
      { id: "claude-3-haiku-20240307", enabled: true, maxTokens: 200000, inputCostPer1K: 0.00025, outputCostPer1K: 0.00125 }
    ]
  },
  safety: {
    enabled: true,
    moderation: {
      enabled: true,
      provider: "custom",
      blockOnFlag: true
    }
  }
}

// ============================================
// SCENARIO 4: Admin Configures Custom AI Provider (Proxy)
// ============================================
// Route: POST /api/admin/ai/providers
{
  provider: "custom",
  name: "Azure OpenAI",
  connection: {
    baseUrl: "https://my-company.openai.azure.com/openai",
    apiKey: "azure-xxx...",
    apiVersion: "2024-02-15-preview",
    extraHeaders: {
      "api-key": "azure-xxx..."
    }
  },
  models: {
    available: [
      { id: "gpt-4-turbo", name: "Azure GPT-4 Turbo", enabled: true, maxTokens: 128000, inputCostPer1K: 0.01, outputCostPer1K: 0.03 }
    ]
  }
}

// ============================================
// SCENARIO 5: Toggle Model Availability
// ============================================
// Route: PATCH /api/admin/ai/providers/openai/models/gpt-4
{
  enabled: false,
  reason: "GPT-4 being deprecated, migrate to GPT-4 Turbo"
}
// Users attempting to use GPT-4 will get:
// {
//   error: "This model has been disabled by your administrator",
//   suggestion: "Use gpt-4-turbo instead"
// }

// ============================================
// SCENARIO 6: View Provider Health & Status
// ============================================
// Route: GET /api/admin/ai/providers/status

// Response:
{
  providers: [
    {
      provider: "openai",
      status: "healthy",
      latency: {
        p50: 450,
        p95: 1200,
        p99: 2500
      },
      rateLimitStatus: {
        remaining: 45,
        resetsAt: "2026-02-15T15:01:00Z"
      },
      quotaStatus: {
        used: 450.00,
        limit: 1000.00,
        percentUsed: 45
      },
      uptime: 99.95,
      errorsLast24h: 12
    },
    {
      provider: "anthropic",
      status: "degraded",
      latency: {
        p50: 800,
        p95: 3500,
        p99: 8000
      },
      error: "High latency detected",
      suggestion: "Consider enabling fallback to OpenAI"
    }
  ]
}

// ============================================
// SCENARIO 7: Configure Fallback
// ============================================
// Route: POST /api/admin/ai/fallback
{
  primary: "anthropic",
  fallbacks: [
    { provider: "openai", trigger: "error", delayMs: 5000 },
    { provider: "google", trigger: "error", delayMs: 10000 }
  ],
  fallbackBehavior: {
    retryOriginalPrompt: true,
    preserveConversation: true,
    notifyOnFallback: true
  }
}
```

---

## AI Capabilities Registry

### Capability Definition

```typescript
interface AICapability {
  // ============================================
  // CAPABILITY IDENTITY
  // ============================================
  id: string                              // "cap-clx-xxx"
  slug: string                            // "code-generator"
  name: string                            // "Code Generator"
  description: string                     // "Generate code from natural language descriptions"

  // ============================================
  // CAPABILITY DEFINITION
  // ============================================
  definition: {
    // Category
    category: AICapabilityCategory
    // CODE_GENERATION | TEXT_GENERATION | IMAGE_GENERATION | ANALYSIS | 
    // TRANSFORMATION | CHAT | RESEARCH | CUSTOM

    // Tags
    tags: string[]

    // Provider requirement
    provider: string                       // "openai" | "anthropic" | "any"

    // Model requirements
    models: {
      required: string[]                  // Models that must support these features
      recommended: string[]
      minContextWindow: number
      supportsVision?: boolean
      supportsFunctionCalling?: boolean
    }

    // Input/Output schema
    inputSchema: object                   // JSON Schema
    outputSchema: object

    // System prompt
    systemPrompt?: string
    userPromptTemplate?: string

    // Parameters
    parameters: {
      temperature?: { min: number, max: number, default: number }
      maxTokens?: { min: number, max: number, default: number }
      topP?: { min: number, max: number, default: number }
    }
  }

  // ============================================
  // CAPABILITY CONFIGURATION
  // ============================================
  config: {
    // Cost
    creditCost: number                    // Credits per use
    costOverride?: number                // Override provider cost

    // Limits
    maxCallsPerDay: number
    maxTokensPerCall: number

    // Features
    streaming: boolean
    functionCalling: boolean
    vision: boolean

    // Custom tools
    tools?: {
      id: string
      name: string
      description: string
      schema: object
    }[]
  }

  // ============================================
  // ACCESS CONTROL
  // ============================================
  access: {
    // Visibility
    isPublic: boolean                    // Available to all orgs (system capability)

    // Scope
    organizationId?: string              // Org-specific capability

    // Who can use
    allowedRoles: UserRole[]
    allowedUsers: string[]
    requirePurchase: boolean             // Requires credit purchase
    requiredPlan?: string                // Minimum plan

    // Approval
    requireApproval: boolean
    approvedBy?: string
  }

  // ============================================
  // USAGE TRACKING
  // ============================================
  usage: {
    usageCount: number
    lastUsedAt: Date
    averageLatency: number
    successRate: number
  }

  // ============================================
  // STATUS
  // ============================================
  status: "active" | "beta" | "deprecated" | "disabled"
  deprecationNotice?: string
  replacementCapabilityId?: string
}

// ============================================
// SYSTEM CAPABILITIES (Built-in)
// ============================================
const SYSTEM_AI_CAPABILITIES: AICapability[] = [
  {
    id: "cap-code-generator",
    slug: "code-generator",
    name: "Code Generator",
    description: "Generate code from natural language descriptions",
    category: "CODE_GENERATION",
    tags: ["code", "development", "programming"],
    provider: "any",
    models: {
      required: [],
      minContextWindow: 4096
    },
    creditCost: 2,
    streaming: true,
    functionCalling: true,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER", "VIEWER"]
    }
  },
  {
    id: "cap-code-explainer",
    slug: "code-explainer",
    name: "Code Explainer",
    description: "Explain what a piece of code does",
    category: "ANALYSIS",
    tags: ["code", "analysis", "documentation"],
    provider: "any",
    creditCost: 1,
    streaming: true,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER", "VIEWER"]
    }
  },
  {
    id: "cap-bug-detector",
    slug: "bug-detector",
    name: "Bug Detector",
    description: "Find bugs and security vulnerabilities in code",
    category: "ANALYSIS",
    tags: ["code", "security", "bugs", "quality"],
    provider: "any",
    models: {
      required: [],
      minContextWindow: 8192
    },
    creditCost: 3,
    streaming: true,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER"]
    }
  },
  {
    id: "cap-code-refactor",
    slug: "code-refactor",
    name: "Code Refactor",
    description: "Refactor and improve existing code",
    category: "TRANSFORMATION",
    tags: ["code", "refactoring", "improvement"],
    provider: "any",
    creditCost: 3,
    streaming: true,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER"]
    }
  },
  {
    id: "cap-text-summarizer",
    slug: "text-summarizer",
    name: "Text Summarizer",
    description: "Summarize long text into concise summaries",
    category: "TEXT_GENERATION",
    tags: ["text", "summarization", "writing"],
    provider: "any",
    creditCost: 1,
    streaming: true,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER", "VIEWER"]
    }
  },
  {
    id: "cap-blueprint-generator",
    slug: "blueprint-generator",
    name: "Blueprint Generator",
    description: "Generate architectural blueprints from descriptions",
    category: "CODE_GENERATION",
    tags: ["blueprint", "architecture", "diagrams"],
    provider: "any",
    models: {
      required: [],
      minContextWindow: 16000,
      supportsVision: true
    },
    creditCost: 5,
    streaming: false,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER"]
    }
  },
  {
    id: "cap-mindmap-generator",
    slug: "mindmap-generator",
    name: "Mind Map Generator",
    description: "Generate mind maps from topic descriptions",
    category: "CODE_GENERATION",
    tags: ["mindmap", "visualization", "brainstorming"],
    provider: "any",
    creditCost: 3,
    streaming: false,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER"]
    }
  },
  {
    id: "cap-code-translator",
    slug: "code-translator",
    name: "Code Translator",
    description: "Translate code from one language to another",
    category: "TRANSFORMATION",
    tags: ["code", "translation", "programming"],
    provider: "any",
    creditCost: 2,
    streaming: true,
    access: {
      isPublic: true,
      allowedRoles: ["OWNER", "ADMIN", "MEMBER"]
    }
  }
]
```

### Capability Scenarios

```typescript
// ============================================
// SCENARIO 1: Admin Enables/Disables Capability
// ============================================
// Route: PATCH /api/admin/ai/capabilities/:id
{
  enabled: false,
  reason: "Being replaced by newer version"
}
// Users trying to use this capability will see:
// "This feature is currently unavailable. Contact your administrator."

// ============================================
// SCENARIO 2: Admin Creates Custom Capability
// ============================================
// Route: POST /api/admin/ai/capabilities
{
  name: "Internal Code Review",
  slug: "internal-code-review",
  description: "Custom code review using internal guidelines",
  category: "ANALYSIS",
  tags: ["code", "internal", "compliance"],
  provider: "custom",
  systemPrompt: "You are a code reviewer following ACME Corp's internal coding standards...",
  inputSchema: {
    type: "object",
    properties: {
      code: { type: "string" },
      language: { type: "string" },
      guidelines: { type: "string" }
    },
    required: ["code"]
  },
  creditCost: 5,
  maxTokensPerCall: 8000,
  allowedRoles: ["OWNER", "ADMIN"],
  requireApproval: false
}

// ============================================
// SCENARIO 3: Admin Sets Role-Based Access
// ============================================
// Route: PATCH /api/admin/ai/capabilities/:id/access
{
  allowedRoles: ["OWNER", "ADMIN", "MEMBER"],
  allowedUsers: ["user-specific-id"],      // Exception users
  requiredPlan: "PRO",
  maxCallsPerDay: 100
}

// ============================================
// SCENARIO 4: User Views Available Capabilities
// ============================================
// Route: GET /api/user/ai/capabilities
// Response:
{
  capabilities: [
    {
      id: "cap-code-generator",
      name: "Code Generator",
      description: "Generate code from natural language",
      category: "CODE_GENERATION",
      creditCost: 2,
      access: "allowed",
      available: true,
      myUsageToday: 5,
      myDailyLimit: 50
    },
    {
      id: "cap-bug-detector",
      name: "Bug Detector",
      description: "Find bugs in code",
      category: "ANALYSIS",
      creditCost: 3,
      access: "allowed",
      available: true,
      myUsageToday: 2,
      myDailyLimit: 20
    },
    {
      id: "cap-internal-code-review",
      name: "Internal Code Review",
      category: "ANALYSIS",
      access: "denied",
      reason: "Requires ADMIN role",
      upgradeToAccess: "PRO"
    }
  ]
}

// ============================================
// SCENARIO 5: Capability Usage Analytics
// ============================================
// Route: GET /api/admin/ai/capabilities/:id/usage
// Query: ?period=30d&groupBy=user

// Response:
{
  capability: {
    id: "cap-code-generator",
    name: "Code Generator"
  },
  period: "30d",
  metrics: {
    totalCalls: 15420,
    successfulCalls: 15300,
    failedCalls: 120,
    totalCredits: 30840,
    averageLatency: 2500,
    successRate: 99.2
  },
  byUser: [
    { userId: "user-1", calls: 5000, credits: 10000 },
    { userId: "user-2", calls: 3200, credits: 6400 }
  ],
  byModel: [
    { model: "gpt-4-turbo", calls: 12000, credits: 24000 },
    { model: "claude-3-5-sonnet", calls: 3420, credits: 6840 }
  ],
  dailyTrend: [...]
}
```

---

## MCP Permissions & Access Control

### Permission Model

```typescript
interface MCPPermission {
  // ============================================
  // PERMISSION IDENTITY
  // ============================================
  id: string
  organizationId: string
  capabilityId: string                    // Which AI capability

  // ============================================
  // SCOPE (Who gets access)
  // ============================================
  scope: "ORGANIZATION" | "ROLE" | "USER" | "TEAM"

  // Scope-specific
  role?: UserRole                        // If scope = ROLE
  userId?: string                        // If scope = USER
  teamId?: string                        // If scope = TEAM

  // ============================================
  // PERMISSION LEVEL
  // ============================================
  allowed: boolean                       // true = grant, false = deny

  // ============================================
  // LIMITS (Optional restrictions)
  // ============================================
  limits?: {
    maxCallsPerDay: number
    maxTokensPerDay: number
    maxCreditsPerDay: number
    maxConcurrentCalls: number
    allowedModels?: string[]            // Only these models
    blockedModels?: string[]
    allowedTimes?: {                     // Time-based restrictions
      start: string                      // "09:00"
      end: string                        // "17:00"
      timezone: string                   // "America/New_York"
      daysOfWeek: number[]               // [1,2,3,4,5] Mon-Fri
    }
  }

  // ============================================
  // CONDITIONS (ABAC-style)
  // ============================================
  conditions?: {
    ipWhitelist?: string[]              // Only from these IPs
    ipBlacklist?: string[]              // Blocked IPs
    requireMFA?: boolean               // Must have MFA enabled
    requireVerifiedEmail?: boolean
    organizationPlan?: string[]          // Only for these plans
  }

  // ============================================
  // METADATA
  // ============================================
  description?: string
  createdById: string

  // ============================================
  // STATUS
  // ============================================
  isActive: boolean
  expiresAt?: Date
}
```

### Permission Scenarios

```typescript
// ============================================
// SCENARIO 1: Grant Org-Wide Access to Capability
// ============================================
// Route: POST /api/admin/mcp/permissions
{
  capabilityId: "cap-code-generator",
  scope: "ORGANIZATION",
  allowed: true,
  description: "All org members can use code generator"
}

// ============================================
// SCENARIO 2: Grant Role-Based Access
// ============================================
// Route: POST /api/admin/mcp/permissions
{
  capabilityId: "cap-bug-detector",
  scope: "ROLE",
  role: "MEMBER",
  allowed: true,
  limits: {
    maxCallsPerDay: 20,
    maxCreditsPerDay: 60
  }
}

// ============================================
// SCENARIO 3: Grant Specific User Access
// ============================================
// Route: POST /api/admin/mcp/permissions
{
  capabilityId: "cap-internal-code-review",
  scope: "USER",
  userId: "user-clx-xxx",
  allowed: true,
  description: "Grant to senior developer for internal reviews"
}

// ============================================
// SCENARIO 4: Deny Access to Specific Role
// ============================================
// Route: POST /api/admin/mcp/permissions
{
  capabilityId: "cap-code-generator",
  scope: "ROLE",
  role: "VIEWER",
  allowed: false,
  description: "Viewers cannot generate code"
}

// ============================================
// SCENARIO 5: Grant Access with Time Restrictions
// ============================================
// Route: POST /api/admin/mcp/permissions
{
  capabilityId: "cap-ai-generation",
  scope: "ROLE",
  role: "MEMBER",
  allowed: true,
  limits: {
    maxCallsPerDay: 50,
    allowedTimes: {
      start: "09:00",
      end: "18:00",
      timezone: "America/New_York",
      daysOfWeek: [1, 2, 3, 4, 5]       // Mon-Fri only
    }
  }
}

// User trying to use outside hours gets:
// "Code generation is only available 9:00 AM - 6:00 PM Eastern Time, Monday-Friday"

// ============================================
// SCENARIO 6: Grant Temporary Access
// ============================================
// Route: POST /api/admin/mcp/permissions
{
  capabilityId: "cap-premium-generator",
  scope: "USER",
  userId: "user-clx-xxx",
  allowed: true,
  expiresAt: "2026-03-01T00:00:00Z",
  description: "Trial access for one week"
}

// ============================================
// SCENARIO 7: View User's Effective MCP Permissions
// ============================================
// Route: GET /api/admin/mcp/users/:userId/effective-permissions

// Response:
{
  user: { id: "user-clx-xxx", role: "MEMBER" },
  effectivePermissions: [
    {
      capability: "cap-code-generator",
      allowed: true,
      source: "ORGANIZATION",
      limits: {
        maxCallsPerDay: 50,
        maxCreditsPerDay: 100
      }
    },
    {
      capability: "cap-bug-detector",
      allowed: true,
      source: "ROLE:MEMBER",
      limits: {
        maxCallsPerDay: 20,
        maxCreditsPerDay: 60
      }
    },
    {
      capability: "cap-premium-generator",
      allowed: true,
      source: "USER:SPECIFIC",
      expiresAt: "2026-03-01T00:00:00Z"
    },
    {
      capability: "cap-admin-only",
      allowed: false,
      source: "ROLE:MEMBER",
      reason: "Requires ADMIN role"
    }
  ],
  quotaUsage: {
    today: {
      calls: 12,
      credits: 36,
      remaining: {
        calls: 38,
        credits: 64
      }
    }
  }
}

// ============================================
// SCENARIO 8: Check Permission Before API Call
// ============================================
// Internal: Called before every AI capability invocation
// Function: checkMCPPermission(userId, capabilityId, modelId)

function checkMCPPermission(
  userId: string,
  capabilityId: string,
  modelId?: string
): PermissionCheckResult {
  // 1. Get all permissions for user (role, user-specific, org-wide)
  const permissions = getPermissionsForUser(userId, capabilityId)

  // 2. Check for explicit deny (always wins)
  const explicitDeny = permissions.find(p => !p.allowed && p.scope === 'USER')
  if (explicitDeny) {
    return { allowed: false, reason: explicitDeny.description }
  }

  // 3. Check time-based restrictions
  if (permissions.some(p => p.limits?.allowedTimes)) {
    if (!isWithinAllowedTime(permissions)) {
      return { allowed: false, reason: "Outside allowed time window" }
    }
  }

  // 4. Check quota limits
  const quotaCheck = checkQuotaLimits(userId, capabilityId)
  if (!quotaCheck.allowed) {
    return { allowed: false, reason: quotaCheck.reason }
  }

  // 5. Check model restrictions
  if (modelId && permissions.some(p => p.limits?.allowedModels)) {
    const allowedModels = permissions.flatMap(p => p.limits?.allowedModels || [])
    if (!allowedModels.includes(modelId)) {
      return { allowed: false, reason: `Model ${modelId} not allowed for this capability` }
    }
  }

  // 6. Check for any allow
  const allowPermission = permissions.find(p => p.allowed)
  if (allowPermission) {
    return {
      allowed: true,
      source: allowPermission.source,
      limits: allowPermission.limits
    }
  }

  // 7. Default deny
  return { allowed: false, reason: "No permission to use this capability" }
}
```

---

## AI Usage Tracking & Limits

### Usage Tracking Model

```typescript
interface AIUsageTracking {
  // ============================================
  // REAL-TIME TRACKING
  // ============================================

  // Per-request tracking
  trackRequest: {
    userId: string
    organizationId: string
    capabilityId: string
    provider: string
    model: string
    inputTokens: number
    outputTokens: number
    totalTokens: number
    cost: number
    latencyMs: number
    status: "success" | "error" | "partial"
    errorCode?: string
    cached?: boolean                      // Used cached input
  }

  // Aggregated tracking
  aggregations: {
    // Real-time counters
    realTime: {
      requestsThisMinute: number
      tokensThisMinute: number
      creditsThisMinute: number
    }

    // Daily aggregates
    daily: {
      totalRequests: number
      totalInputTokens: number
      totalOutputTokens: number
      totalCost: number
      uniqueUsers: number
      byCapability: { [capabilityId]: number }
      byModel: { [modelId]: number }
      byUser: { [userId]: UsageSummary }
    }

    // Monthly aggregates
    monthly: {
      totalCost: number
      totalRequests: number
      averageLatency: number
      budgetUsed: number
      budgetRemaining: number
    }
  }
}

// ============================================
// LIMIT TYPES
// ============================================

interface UsageLimits {
  // Organization limits
  organization: {
    maxCreditsPerMonth: number
    maxRequestsPerMinute: number
    maxConcurrentRequests: number
    warnAtPercent: number               // Alert at X% of limit
  }

  // User limits
  user: {
    maxCreditsPerDay: number
    maxCreditsPerWeek: number
    maxCreditsPerMonth: number
    maxRequestsPerDay: number
    maxRequestsPerHour: number
  }

  // Capability-specific limits
  capability: {
    [capabilityId: string]: {
      maxCallsPerDay: number
      maxTokensPerCall: number
    }
  }

  // Rate limiting
  rateLimit: {
    requestsPerSecond: number
    tokensPerMinute: number
    burstLimit: number
  }
}

// ============================================
// USAGE SCENARIOS
// ============================================

// Scenario 1: User Approaches Daily Limit
// When user reaches 80% of daily limit:
{
  notification: {
    type: "usage_warning",
    channel: "in_app",
    message: "You've used 80% of your daily AI credits (40/50). 10 credits remaining.",
    action: {
      type: "upgrade",
      label: "Get More Credits",
      url: "/settings/billing"
    }
  }
}

// Scenario 2: User Exceeds Limit
// When user tries to use AI at limit:
{
  error: {
    code: "DAILY_LIMIT_EXCEEDED",
    message: "You've reached your daily AI credit limit",
    details: {
      limit: 50,
      used: 50,
      resetsAt: "2026-02-16T00:00:00Z"
    },
    options: [
      { action: "wait", label: "Wait until tomorrow" },
      { action: "upgrade", label: "Upgrade to PRO for higher limits" },
      { action: "purchase", label: "Purchase additional credits" }
    ]
  }
}

// Scenario 3: Admin Views Org Usage Dashboard
// Route: GET /api/admin/ai/usage
// Query: ?period=30d&groupBy=day

{
  organization: { id: "org-xxx", name: "Acme Corp", plan: "PRO" },
  period: "30d",
  summary: {
    totalCost: 1250.00,
    totalRequests: 45000,
    totalInputTokens: 15000000,
    totalOutputTokens: 8500000,
    averageLatency: 1200,
    successRate: 99.5
  },
  limits: {
    monthlyBudget: 2000.00,
    used: 1250.00,
    remaining: 750.00,
    percentUsed: 62.5
  },
  dailyTrend: [
    { date: "2026-02-15", cost: 45.00, requests: 1500 },
    { date: "2026-02-14", cost: 42.00, requests: 1400 }
  ],
  byUser: [
    { userId: "user-1", name: "John Doe", cost: 350.00, requests: 12000, percent: 28 },
    { userId: "user-2", name: "Jane Smith", cost: 280.00, requests: 9500, percent: 22 }
  ],
  byCapability: [
    { capability: "Code Generator", cost: 600.00, requests: 20000 },
    { capability: "Bug Detector", cost: 350.00, requests: 15000 }
  ],
  byProvider: [
    { provider: "OpenAI", cost: 750.00, requests: 28000 },
    { provider: "Anthropic", cost: 500.00, requests: 17000 }
  ]
}

// Scenario 4: Admin Sets Usage Alerts
// Route: POST /api/admin/ai/usage/alerts
{
  alerts: [
    {
      type: "budget_threshold",
      threshold: 80,                    // percent
      channel: "email",
      recipients: ["admin@acme.com"]
    },
    {
      type: "user_threshold",
      threshold: 100,                   // percent of user's personal limit
      userId: "user-1",
      channel: "in_app",
      notifyUser: true
    },
    {
      type: "anomaly",
      condition: "requests_spike_200%",  // 2x normal
      channel: "slack",
      webhookUrl: "https://hooks.slack.com/..."
    }
  ]
}

// Scenario 5: User Views Their Usage
// Route: GET /api/user/ai/usage

{
  user: { id: "user-xxx", role: "MEMBER" },
  period: "30d",
  summary: {
    totalRequests: 450,
    totalCredits: 900,
    averageLatency: 1800
  },
  limits: {
    daily: { limit: 50, used: 35, remaining: 15 },
    monthly: { limit: 1000, used: 900, remaining: 100 }
  },
  byCapability: [
    { capability: "Code Generator", calls: 250, credits: 500 },
    { capability: "Bug Detector", calls: 200, credits: 400 }
  ],
  recentActivity: [
    { timestamp: "2026-02-15T14:30:00Z", capability: "Code Generator", tokens: 500, cost: 2 },
    { timestamp: "2026-02-15T14:25:00Z", capability: "Bug Detector", tokens: 800, cost: 3 }
  ]
}
```

---

## Third-Party Integrations

### Integration Configuration

```typescript
interface ThirdPartyIntegration {
  // ============================================
  // INTEGRATION IDENTITY
  // ============================================
  id: string
  name: string                            // "Slack"
  slug: string                            // "slack"
  description: string
  category: IntegrationCategory           // "communication" | "development" | "productivity" | "storage" | "analytics"

  // ============================================
  // CONNECTION TYPE
  // ============================================
  connection: {
    type: "oauth2" | "api_key" | "webhook" | "custom"

    // OAuth2 config
    oauth?: {
      clientId: string
      clientSecret: string              // Encrypted
      authorizationUrl: string
      tokenUrl: string
      scopes: string[]
      redirectUri: string
    }

    // API Key config
    apiKey?: {
      location: "header" | "query"       // Where to send the key
      keyName: string                    // "X-API-Key"
      prefix?: string                    // e.g., "sk_live_"
    }

    // Webhook config
    webhook?: {
      url: string
      secret: string                     // For signature verification
    }

    // Custom config
    custom?: {
      endpoint: string
      authType: string
      config: Record<string, string>
    }
  }

  // ============================================
  // FEATURES & CAPABILITIES
  // ============================================
  features: {
    // What can be synced
    sync: {
      users?: boolean                   // Sync user accounts
      projects?: boolean
      notifications?: boolean
      files?: boolean
      comments?: boolean
    }

    // Events supported
    events: {
      onProjectCreated: boolean
      onProjectUpdated: boolean
      onProjectDeleted: boolean
      onUserInvited: boolean
      onCommentAdded: boolean
    }

    // Actions supported
    actions: {
      createProject: boolean
      updateProject: boolean
      inviteUser: boolean
      sendNotification: boolean
    }
  }

  // ============================================
  // STATUS
  // ============================================
  status: {
    connected: boolean
    lastSyncAt?: Date
    lastError?: string
    lastErrorAt?: Date
  }

  // ============================================
  // SCOPING
  // ============================================
  scope: {
    organizationId: string
    userId?: string                      // Personal integration
    teamId?: string
  }
}
```

### Integration Scenarios

```typescript
// ============================================
// SCENARIO 1: Connect Slack Integration
// ============================================
// Step 1: Initiate OAuth
// Route: GET /api/integrations/slack/connect
// Response: Redirect to Slack OAuth

// Step 2: Handle callback
// Route: GET /api/integrations/slack/callback
// Query: ?code=xxx

// On success:
{
  integration: {
    id: "int-slack-xxx",
    name: "Slack",
    status: "connected",
    connectedAt: "2026-02-15T15:00:00Z",
    scopes: ["chat:write", "channels:read", "users:read"]
  },
  settings: {
    defaultChannel: "#general",
    notifyOn: ["project_created", "project_completed", "user_invited"]
  }
}

// ============================================
// SCENARIO 2: Connect GitHub Integration
// ============================================
// Route: POST /api/integrations/github/connect
{
  oauthCode: "xxx",
  settings: {
    repository: "acme/blueprint-forge",
    autoSync: true,
    syncOptions: {
      issues: true,
      pullRequests: true,
      deployments: false
    },
    webhookEvents: ["push", "pull_request", "issues"]
  }
}

// Response:
{
  integration: {
    id: "int-github-xxx",
    name: "GitHub",
    status: "connected",
    connectedAt: "2026-02-15T15:00:00Z",
    repository: "acme/blueprint-forge"
  }
}

// ============================================
// SCENARIO 3: Configure Webhook Integration
// ============================================
// Route: POST /api/integrations/webhooks
{
  name: "Zapier",
  url: "https://hooks.zapier.com/hooks/catch/xxx/yyy/",
  secret: "whsec_xxx...",               // Webhook signature secret
  events: [
    "project.created",
    "project.updated",
    "user.invited",
    "blueprint.generated"
  ],
  active: true
}

// Response:
{
  webhook: {
    id: "wh-xxx",
    name: "Zapier",
    url: "https://hooks.zapier.com/...",
    status: "active",
    events: ["project.created", ...],
    testPayload: { sent: true, response: 200 }
  }
}

// Test webhook:
// Route: POST /api/integrations/webhooks/:id/test
{
  event: "project.created"
}
// Response: { success: true, deliveryTime: 150 }

// ============================================
// SCENARIO 4: Connect Custom REST API
// ============================================
// Route: POST /api/integrations/custom
{
  name: "Internal API",
  type: "rest",
  endpoint: "https://api.internal.company.com/v1",
  auth: {
    type: "bearer",
    token: "internal-token-xxx"
  },
  headers: {
    "X-Company-ID": "acme-corp"
  },
  features: {
    sync: { users: true },
    events: { onUserInvited: true },
    actions: { inviteUser: true }
  }
}

// Test connection:
// Route: POST /api/integrations/custom/:id/test
{
  action: "getUser",
  params: { userId: "test-user" }
}
// Response: { success: true, responseTime: 200 }

// ============================================
// SCENARIO 5: Sync Integration Data
// ============================================
// Route: POST /api/integrations/:id/sync
// For manual sync

// Or automatic sync (if enabled):
// Background job syncs based on configured interval

// Response:
{
  sync: {
    status: "completed",
    startedAt: "2026-02-15T15:00:00Z",
    completedAt: "2026-02-15T15:01:30Z",
    itemsProcessed: 150,
    itemsCreated: 5,
    itemsUpdated: 20,
    itemsDeleted: 0,
    errors: []
  }
}

// ============================================
// SCENARIO 6: Disconnect Integration
// ============================================
// Route: DELETE /api/integrations/:id
{
  confirmation: "I understand this will stop all syncs and notifications"
}
// Response: { success: true, deletedData: { lastSyncedAt: "2026-02-15T14:00:00Z" } }

// ============================================
// SCENARIO 7: View Integration Health
// ============================================
// Route: GET /api/integrations/:id/health

{
  integration: { id: "int-slack-xxx", name: "Slack" },
  status: "healthy",
  checks: {
    oauth: { status: "valid", expiresAt: "2026-03-15T15:00:00Z" },
    api: { status: "ok", latency: 150 },
    webhook: { status: "ok", lastDelivery: "2026-02-15T14:55:00Z" }
  },
  metrics: {
    requestsToday: 45,
    successRate: 99.8,
    averageLatency: 200,
    errorsLast24h: 2
  }
}
```

---

## Integration API Architecture

### Complete API Routes

```typescript
interface IntegrationAPIRoutes {
  // ============================================
  // MCP SERVERS
  // ============================================
  mcpServers: {
    "GET /api/admin/mcp/servers": {
      auth: "OWNER | ADMIN"
      response: { servers: MCPConfiguration[] }
    }
    "POST /api/admin/mcp/servers": {
      auth: "OWNER | ADMIN"
      body: MCPConfiguration
      response: { server: MCPConfiguration }
    }
    "GET /api/admin/mcp/servers/:id": {
      auth: "OWNER | ADMIN"
      response: { server: MCPConfiguration }
    }
    "PATCH /api/admin/mcp/servers/:id": {
      auth: "OWNER | ADMIN"
      body: Partial<MCPConfiguration>
      response: { server: MCPConfiguration }
    }
    "DELETE /api/admin/mcp/servers/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
    "POST /api/admin/mcp/servers/:id/test": {
      auth: "OWNER | ADMIN"
      body: { testPrompt: string, model?: string }
      response: { success: boolean, response?: string, latency?: number }
    }
    "POST /api/admin/mcp/servers/:id/set-default": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
    "GET /api/admin/mcp/servers/:id/usage": {
      auth: "OWNER | ADMIN"
      query: { period: string }
      response: { usage: UsageMetrics }
    }
    "GET /api/admin/mcp/servers/:id/health": {
      auth: "OWNER | ADMIN"
      response: { health: ServerHealth }
    }

    // User personal MCP
    "GET /api/user/mcp/servers": {
      auth: "any user"
      response: { servers: MCPConfiguration[] }
    }
    "POST /api/user/mcp/servers": {
      auth: "any user"
      body: MCPConfiguration
      response: { server: MCPConfiguration }
    }
    "DELETE /api/user/mcp/servers/:id": {
      auth: "owner only"
      response: { success: boolean }
    }
  }

  // ============================================
  // AI PROVIDERS
  // ============================================
  aiProviders: {
    "GET /api/admin/ai/providers": {
      auth: "OWNER | ADMIN"
      response: { providers: AIProviderConfiguration[] }
    }
    "POST /api/admin/ai/providers": {
      auth: "OWNER | ADMIN"
      body: AIProviderConfiguration
      response: { provider: AIProviderConfiguration }
    }
    "PATCH /api/admin/ai/providers/:provider": {
      auth: "OWNER | ADMIN"
      body: Partial<AIProviderConfiguration>
      response: { provider: AIProviderConfiguration }
    }
    "DELETE /api/admin/ai/providers/:provider": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
    "POST /api/admin/ai/providers/:provider/test": {
      auth: "OWNER | ADMIN"
      body: { model: string, prompt: string }
      response: { success: boolean }
    }
    "GET /api/admin/ai/providers/status": {
      auth: "OWNER | ADMIN"
      response: { providers: ProviderStatus[] }
    }
    "POST /api/admin/ai/fallback": {
      auth: "OWNER | ADMIN"
      body: FallbackConfiguration
      response: { fallback: FallbackConfiguration }
    }

    // User personal providers
    "GET /api/user/ai/providers": {
      auth: "any user"
      response: { providers: AIProviderConfiguration[] }
    }
    "POST /api/user/ai/providers": {
      auth: "any user"
      body: AIProviderConfiguration
      response: { provider: AIProviderConfiguration }
    }
  }

  // ============================================
  // AI CAPABILITIES
  // ============================================
  aiCapabilities: {
    "GET /api/admin/ai/capabilities": {
      auth: "OWNER | ADMIN"
      response: { capabilities: AICapability[] }
    }
    "POST /api/admin/ai/capabilities": {
      auth: "OWNER | ADMIN"
      body: AICapability
      response: { capability: AICapability }
    }
    "PATCH /api/admin/ai/capabilities/:id": {
      auth: "OWNER | ADMIN"
      body: Partial<AICapability>
      response: { capability: AICapability }
    }
    "DELETE /api/admin/ai/capabilities/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
    "PATCH /api/admin/ai/capabilities/:id/access": {
      auth: "OWNER | ADMIN"
      body: AccessConfiguration
      response: { access: AccessConfiguration }
    }
    "GET /api/admin/ai/capabilities/:id/usage": {
      auth: "OWNER | ADMIN"
      response: { usage: CapabilityUsage }
    }

    // User capabilities
    "GET /api/user/ai/capabilities": {
      auth: "any user"
      response: { capabilities: AICapability[] }
    }
  }

  // ============================================
  // MCP PERMISSIONS
  // ============================================
  mcpPermissions: {
    "GET /api/admin/mcp/permissions": {
      auth: "OWNER | ADMIN"
      query: { capabilityId?: string, scope?: string }
      response: { permissions: MCPPermission[] }
    }
    "POST /api/admin/mcp/permissions": {
      auth: "OWNER | ADMIN"
      body: MCPPermission
      response: { permission: MCPPermission }
    }
    "DELETE /api/admin/mcp/permissions/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
    "GET /api/admin/mcp/users/:userId/effective-permissions": {
      auth: "OWNER | ADMIN"
      response: { effectivePermissions: EffectivePermission[] }
    }
  }

  // ============================================
  // AI USAGE
  // ============================================
  aiUsage: {
    "GET /api/admin/ai/usage": {
      auth: "OWNER | ADMIN"
      query: { period: string, groupBy?: string }
      response: { usage: AIUsageSummary }
    }
    "POST /api/admin/ai/usage/alerts": {
      auth: "OWNER | ADMIN"
      body: UsageAlertConfiguration
      response: { alerts: UsageAlert[] }
    }
    "GET /api/user/ai/usage": {
      auth: "any user"
      response: { usage: UserAIUsage }
    }
  }

  // ============================================
  // THIRD-PARTY INTEGRATIONS
  // ============================================
  integrations: {
    "GET /api/integrations": {
      auth: "OWNER | ADMIN"
      response: { integrations: ThirdPartyIntegration[] }
    }
    "GET /api/integrations/marketplace": {
      auth: "any user"
      response: { available: IntegrationDefinition[] }
    }
    "POST /api/integrations/:provider/connect": {
      auth: "OWNER | ADMIN"
      body: ConnectionConfig
      response: { authUrl?: string, integration?: ThirdPartyIntegration }
    }
    "GET /api/integrations/:provider/callback": {
      auth: "automatic"
      query: { code?: string, state?: string }
      response: { integration: ThirdPartyIntegration }
    }
    "GET /api/integrations/:id": {
      auth: "OWNER | ADMIN"
      response: { integration: ThirdPartyIntegration }
    }
    "PATCH /api/integrations/:id": {
      auth: "OWNER | ADMIN"
      body: Partial<ThirdPartyIntegration>
      response: { integration: ThirdPartyIntegration }
    }
    "DELETE /api/integrations/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
    "POST /api/integrations/:id/sync": {
      auth: "OWNER | ADMIN"
      response: { sync: SyncResult }
    }
    "GET /api/integrations/:id/health": {
      auth: "OWNER | ADMIN"
      response: { health: IntegrationHealth }
    }

    // Webhooks
    "GET /api/integrations/webhooks": {
      auth: "OWNER | ADMIN"
      response: { webhooks: WebhookConfig[] }
    }
    "POST /api/integrations/webhooks": {
      auth: "OWNER | ADMIN"
      body: WebhookConfig
      response: { webhook: WebhookConfig }
    }
    "POST /api/integrations/webhooks/:id/test": {
      auth: "OWNER | ADMIN"
      body: { event: string }
      response: { success: boolean }
    }
    "DELETE /api/integrations/webhooks/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
  }

  // ============================================
  // API KEYS (FOR INTEGRATIONS)
  // ============================================
  apiKeys: {
    "GET /api/admin/integrations/keys": {
      auth: "OWNER | ADMIN"
      response: { keys: IntegrationAPIKey[] }
    }
    "POST /api/admin/integrations/keys": {
      auth: "OWNER | ADMIN"
      body: { name: string, scopes: string[], expiresAt?: Date }
      response: { key: IntegrationAPIKey }
    }
    "DELETE /api/admin/integrations/keys/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }

    "GET /api/user/integrations/keys": {
      auth: "any user"
      response: { keys: IntegrationAPIKey[] }
    }
    "POST /api/user/integrations/keys": {
      auth: "any user"
      body: { name: string, scopes: string[] }
      response: { key: IntegrationAPIKey }
    }
    "DELETE /api/user/integrations/keys/:id": {
      auth: "owner only"
      response: { success: boolean }
    }
  }
}
```

---

## Summary

This comprehensive MCP & Integrations system provides:

1. **Unified Integration Gateway** — Central hub for all AI providers, third-party integrations, webhooks, and SSO
2. **MCP Server Management** — Full CRUD for MCP servers (OpenAI, Anthropic, Google, custom SSE/stdio), health monitoring, connection testing, usage analytics
3. **AI Provider Configuration** — Per-provider settings, model management, safety/content filtering, logging, rate limiting
4. **AI Capabilities Registry** — Built-in capabilities (code generator, bug detector, etc.), custom capability creation, role-based access
5. **MCP Permissions & Access Control** — Organization/role/user/team-scoped permissions, time-based restrictions, quota limits, ABAC-style conditions
6. **AI Usage Tracking** — Real-time tracking, daily/monthly aggregations, budget alerts, user and org usage dashboards
7. **Third-Party Integrations** — OAuth2 flow, API key auth, webhook configuration, custom REST connectors for Slack, GitHub, Jira, Figma, etc.
8. **API Access & Webhooks** — Programmatic access, webhook delivery with retry logic, event subscription system
9. **SSO & Identity Providers** — SAML, OIDC, LDAP configuration with just-in-time provisioning
10. **Email & Communication** — SMTP, SendGrid, Resend, SES integration with tracking
11. **Storage & Data** — S3, GCS, Azure Blob, Cloudflare R2 integration
12. **Analytics & Monitoring** — Google Analytics, Mixpanel, Datadog, Sentry integration
13. **Admin Dashboard** — Integration management, health monitoring, usage analytics, alert configuration
14. **Security & Encryption** — All sensitive data encrypted, API key management, audit logging
15. **Plan-Based Restrictions** — Feature availability tied to subscription plan
16. **Edge Cases** — Connection failures, rate limiting, quota exceeded, permission conflicts, sync errors
17. **Complete API Architecture** — 40+ API routes for all integration operations

Every aspect is **admin-configurable without code changes**, **RBAC-gated**, **plan-aware**, and **fully auditable**.
