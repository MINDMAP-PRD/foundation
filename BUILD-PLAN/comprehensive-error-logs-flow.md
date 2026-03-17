# Comprehensive Error Logs System Configuration Flow

> **Foundation wiring note:** Treat this as shared platform observability guidance for `GeneralApp` and its admin shell. Error logs, incidents, alerts, and auditability should be implemented as shared foundation services and operator surfaces.

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Universal Error Configuration Pattern](#2-universal-error-configuration-pattern)
3. [Error Categories & Classification](#3-error-categories--classification)
4. [Error Collection & Capture](#4-error-collection--capture)
5. [Error Storage & Retention](#5-error-storage--retention)
6. [Error Processing & Analysis](#6-error-processing--analysis)
7. [Alerting & Notifications](#7-alerting--notifications)
8. [Error API Routes](#8-error-api-routes)
9. [Error Database Schema](#9-error-database-schema)
10. [Edge Cases](#10-edge-cases)
11. [Performance & Caching](#11-performance--caching)
12. [Integration Examples](#12-integration-examples)

---

## 1. System Overview

### 1.1 Core Principles

The Error Logs System provides comprehensive error tracking, analysis, and alerting capabilities across the entire platform. It serves as the central repository for all application errors, system failures, and unexpected behaviors.

**Core Design Principles:**

- **Real-time Capture**: Errors are captured immediately as they occur with full context
- **Intelligent Grouping**: Similar errors are automatically grouped to reduce noise
- **Full Context Preservation**: Every error includes stack traces, request data, user context, and system state
- **Alert Flexibility**: Configurable alerting rules with multiple notification channels
- **Performance Isolation**: Error logging never impacts application performance
- **Compliance Ready**: Full audit trail with configurable retention policies

### 1.2 System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        ERROR LOGS SYSTEM ARCHITECTURE                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │
│  │  Frontend   │    │  Backend    │    │   Workers   │                    │
│  │  Error      │    │  Error      │    │   Error     │                    │
│  │  Capture    │    │  Capture    │    │   Capture   │                    │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                    │
│         │                  │                  │                            │
│         └──────────────────┼──────────────────┘                            │
│                            │                                                │
│                    ┌───────▼───────┐                                        │
│                    │  Error        │                                        │
│                    │  Ingestion    │                                        │
│                    │  Service      │                                        │
│                    └───────┬───────┘                                        │
│                            │                                                │
│         ┌──────────────────┼──────────────────┐                            │
│         │                  │                  │                            │
│  ┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐                   │
│  │  Error      │    │  Alert      │    │  Analytics  │                   │
│  │  Storage    │    │  Engine     │    │  Engine     │                   │
│  │  (Postgres) │    │             │    │             │                   │
│  └─────────────┘    └─────────────┘    └─────────────┘                   │
│                            │                                                │
│         ┌──────────────────┼──────────────────┐                            │
│         │                  │                  │                            │
│  ┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐                   │
│  │  Webhooks   │    │  Email      │    │  Slack/     │                   │
│  │  & API      │    │  Notifs     │    │  Teams      │                   │
│  └─────────────┘    └─────────────┘    └─────────────┘                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Error Flow Pipeline

```
Error Occurs → Capture Context → Validate & Enrich → Store in Database 
→ Process & Group → Check Alert Rules → Send Notifications → Update Dashboard
```

---

## 2. Universal Error Configuration Pattern

### 2.1 ErrorConfiguration Interface

```typescript
interface ErrorConfiguration {
  // Error Capture Settings
  capture: ErrorCaptureConfig;
  
  // Storage & Retention
  storage: ErrorStorageConfig;
  
  // Processing & Grouping
  processing: ErrorProcessingConfig;
  
  // Alerting Rules
  alerting: ErrorAlertingConfig;
  
  // Notification Channels
  notifications: NotificationChannelConfig[];
  
  // Access Control
  access: ErrorAccessConfig;
  
  // Performance Settings
  performance: ErrorPerformanceConfig;
  
  // Integration Settings
  integrations: ErrorIntegrationConfig;
}

interface ErrorCaptureConfig {
  // What types of errors to capture
  enabledTypes: ErrorType[];
  
  // Include/exclude patterns
  includePatterns: string[];
  excludePatterns: string[];
  
  // Context collection
  collectUserContext: boolean;
  collectRequestData: boolean;
  collectSystemState: boolean;
  collectEnvironmentVars: boolean;
  
  // Data limits
  maxStackTraceLength: number;
  maxContextDataSize: number;
  maxBreadcrumbs: number;
  
  // Sampling
  samplingRate: number; // 0-100
  sampleTypes: ErrorType[];
}

interface ErrorStorageConfig {
  // Database settings
  database: {
    provider: 'postgresql' | 'mysql' | 'mongodb';
    retention: RetentionPolicy;
    partitioning: PartitioningStrategy;
  };
  
  // Archive settings
  archive: {
    enabled: boolean;
    destination: 's3' | 'gcs' | 'azure' | 'local';
    retention: number; // days
    compression: 'gzip' | 'zstd' | 'none';
  };
  
  // Encryption
  encryption: {
    atRest: boolean;
    inTransit: boolean;
    keyId: string;
  };
}

interface ErrorProcessingConfig {
  // Grouping settings
  grouping: {
    enabled: boolean;
    fingerprintFields: string[];
    groupingTimeout: number; // ms
    mergeSimilar: boolean;
  };
  
  // Deduplication
  deduplication: {
    enabled: boolean;
    window: number; // seconds
    threshold: number;
  };
  
  // Enrichment
  enrichment: {
    addGeolocation: boolean;
    addDeviceInfo: boolean;
    addReleaseVersion: boolean;
    customEnrichers: string[];
  };
}

interface ErrorAlertingConfig {
  // Global alerting
  enabled: boolean;
  
  // Alert rules
  rules: AlertRule[];
  
  // Rate limiting for alerts
  rateLimit: {
    maxAlertsPerHour: number;
    maxAlertsPerDay: number;
    cooldownPeriod: number; // seconds
  };
  
  // Escalation
  escalation: {
    enabled: boolean;
    escalationMatrix: EscalationLevel[];
  };
}

interface AlertRule {
  id: string;
  name: string;
  enabled: boolean;
  
  // Conditions
  conditions: AlertCondition[];
  
  // Actions
  actions: AlertAction[];
  
  // Schedule
  schedule?: ScheduleConfig;
  
  // Thresholds
  thresholds: {
    errorCount: number;
    errorRate: number;
    uniqueUsers: number;
    timeWindow: number; // minutes
  };
}

interface AlertCondition {
  field: string;
  operator: 'equals' | 'not_equals' | 'contains' | 'greater_than' | 'less_than' | 'regex';
  value: any;
}

interface AlertAction {
  type: 'email' | 'slack' | 'webhook' | 'pagerduty' | 'opsgenie' | 'custom';
  config: Record<string, any>;
  recipients?: string[];
}

interface NotificationChannelConfig {
  id: string;
  type: 'email' | 'slack' | 'webhook' | 'pagerduty' | 'opsgenie' | 'sms' | 'discord' | 'teams';
  name: string;
  enabled: boolean;
  config: Record<string, any>;
  
  // Rate limiting per channel
  rateLimit?: {
    maxPerHour: number;
    maxPerDay: number;
  };
}

interface ErrorAccessConfig {
  // Who can view errors
  viewRoles: Role[];
  
  // Who can configure error settings
  configRoles: Role[];
  
  // Who can delete/resolve errors
  manageRoles: Role[];
  
  // Field-level access
  fieldRestrictions: {
    field: string;
    roles: Role[];
    mask: boolean;
  }[];
  
  // Data masking
  maskSensitiveData: boolean;
  sensitiveFields: string[];
}

interface ErrorPerformanceConfig {
  // Async processing
  asyncProcessing: boolean;
  queueSize: number;
  processingConcurrency: number;
  
  // Caching
  caching: {
    enabled: boolean;
    ttl: number; // seconds
    maxSize: number; // entries
  };
  
  // Batch operations
  batchSize: number;
  flushInterval: number; // ms
}

interface ErrorIntegrationConfig {
  // External integrations
  slack?: SlackIntegrationConfig;
  pagerduty?: PagerDutyIntegrationConfig;
  opsgenie?: OpsGenieIntegrationConfig;
  jira?: JiraIntegrationConfig;
  github?: GitHubIntegrationConfig;
  
  // API access
  apiAccess: {
    enabled: boolean;
    rateLimit: number; // per minute
    apiKeys: ApiKeyConfig[];
  };
  
  // Webhook for errors
  webhooks: WebhookConfig[];
}
```

### 2.2 Default Error Configuration

```typescript
const defaultErrorConfiguration: ErrorConfiguration = {
  capture: {
    enabledTypes: [
      'javascript_error',
      'server_error',
      'database_error',
      'api_error',
      'authentication_error',
      'validation_error',
      'rate_limit_error',
      'timeout_error',
      'network_error',
      'custom_error'
    ],
    includePatterns: ['*'],
    excludePatterns: ['/health', '/ping', '/metrics'],
    collectUserContext: true,
    collectRequestData: true,
    collectSystemState: true,
    collectEnvironmentVars: false,
    maxStackTraceLength: 10000,
    maxContextDataSize: 50000,
    maxBreadcrumbs: 50,
    samplingRate: 100,
    sampleTypes: []
  },
  storage: {
    database: {
      provider: 'postgresql',
      retention: {
        hotStorage: 30,    // days in primary DB
        warmStorage: 90,  // days in slower storage
        coldStorage: 365  // days in archive
      },
      partitioning: {
        strategy: 'daily',
        indexFields: ['organizationId', 'errorType', 'severity', 'timestamp']
      }
    },
    archive: {
      enabled: true,
      destination: 's3',
      retention: 365,
      compression: 'gzip'
    },
    encryption: {
      atRest: true,
      inTransit: true,
      keyId: 'aws/kms/default'
    }
  },
  processing: {
    grouping: {
      enabled: true,
      fingerprintFields: ['message', 'stackTrace', 'location'],
      groupingTimeout: 5000,
      mergeSimilar: true
    },
    deduplication: {
      enabled: true,
      window: 300,
      threshold: 100
    },
    enrichment: {
      addGeolocation: true,
      addDeviceInfo: true,
      addReleaseVersion: true,
      customEnrichers: []
    }
  },
  alerting: {
    enabled: true,
    rules: [
      {
        id: 'critical-error',
        name: 'Critical Error Alert',
        enabled: true,
        conditions: [
          { field: 'severity', operator: 'equals', value: 'critical' }
        ],
        actions: [
          { type: 'slack', config: { channel: '#errors-critical' } },
          { type: 'email', config: {}, recipients: ['ops-team@company.com'] }
        ],
        thresholds: { errorCount: 1, errorRate: 0, uniqueUsers: 0, timeWindow: 1 }
      },
      {
        id: 'high-error-rate',
        name: 'High Error Rate Alert',
        enabled: true,
        conditions: [],
        actions: [
          { type: 'slack', config: { channel: '#errors' } }
        ],
        thresholds: { errorCount: 100, errorRate: 5, uniqueUsers: 10, timeWindow: 5 }
      }
    ],
    rateLimit: {
      maxAlertsPerHour: 10,
      maxAlertsPerDay: 50,
      cooldownPeriod: 300
    },
    escalation: {
      enabled: true,
      escalationMatrix: [
        { afterMinutes: 5, notifyRoles: ['OWNER'] },
        { afterMinutes: 15, notifyRoles: ['ADMIN'] },
        { afterMinutes: 30, notifyExternal: ['on-call@company.com'] }
      ]
    }
  },
  notifications: [],
  access: {
    viewRoles: ['OWNER', 'ADMIN', 'MEMBER'],
    configRoles: ['OWNER', 'ADMIN'],
    manageRoles: ['OWNER', 'ADMIN'],
    fieldRestrictions: [],
    maskSensitiveData: true,
    sensitiveFields: ['password', 'token', 'secret', 'apiKey', 'creditCard']
  },
  performance: {
    asyncProcessing: true,
    queueSize: 10000,
    processingConcurrency: 5,
    caching: {
      enabled: true,
      ttl: 300,
      maxSize: 1000
    },
    batchSize: 100,
    flushInterval: 5000
  },
  integrations: {
    apiAccess: {
      enabled: true,
      rateLimit: 100,
      apiKeys: []
    },
    webhooks: []
  }
};
```

---

## 3. Error Categories & Classification

### 3.1 Error Type Taxonomy

```typescript
enum ErrorType {
  // JavaScript/Frontend Errors
  JAVASCRIPT_ERROR = 'javascript_error',
  UNHANDLED_REJECTION = 'unhandled_rejection',
  RESOURCE_ERROR = 'resource_error',
  
  // Server Errors
  SERVER_ERROR = 'server_error',
  INTERNAL_ERROR = 'internal_error',
  SERVICE_UNAVAILABLE = 'service_unavailable',
  
  // Database Errors
  DATABASE_ERROR = 'database_error',
  CONNECTION_ERROR = 'connection_error',
  QUERY_ERROR = 'query_error',
  TRANSACTION_ERROR = 'transaction_error',
  
  // API Errors
  API_ERROR = 'api_error',
  BAD_REQUEST = 'bad_request',
  NOT_FOUND = 'not_found',
  CONFLICT = 'conflict',
  
  // Authentication & Authorization
  AUTHENTICATION_ERROR = 'authentication_error',
  AUTHORIZATION_ERROR = 'authorization_error',
  PERMISSION_DENIED = 'permission_denied',
  
  // Validation Errors
  VALIDATION_ERROR = 'validation_error',
  SCHEMA_ERROR = 'schema_error',
  
  // Rate Limiting
  RATE_LIMIT_ERROR = 'rate_limit_error',
  QUOTA_EXCEEDED = 'quota_exceeded',
  
  // Network Errors
  NETWORK_ERROR = 'network_error',
  TIMEOUT_ERROR = 'timeout_error',
  CONNECTION_REFUSED = 'connection_refused',
  DNS_ERROR = 'dns_error',
  
  // Custom Errors
  CUSTOM_ERROR = 'custom_error',
  BUSINESS_LOGIC_ERROR = 'business_logic_error',
  INTEGRATION_ERROR = 'integration_error',
  
  // System Errors
  SYSTEM_ERROR = 'system_error',
  OUT_OF_MEMORY = 'out_of_memory',
  STACK_OVERFLOW = 'stack_overflow',
  DISK_FULL = 'disk_full'
}

enum ErrorSeverity {
  CRITICAL = 'critical',
  HIGH = 'high',
  MEDIUM = 'medium',
  LOW = 'low',
  INFO = 'info'
}

enum ErrorStatus {
  NEW = 'new',
  REOPENED = 'reopened',
  IN_PROGRESS = 'in_progress',
  RESOLVED = 'resolved',
  IGNORED = 'ignored',
  ARCHIVED = 'archived'
}

enum ErrorSource {
  FRONTEND = 'frontend',
  BACKEND = 'backend',
  WORKER = 'worker',
  CRON = 'cron',
  WEBHOOK = 'webhook',
  API = 'api',
  DATABASE = 'database',
  EXTERNAL = 'external'
}
```

### 3.2 Error Classification Rules

```typescript
interface ErrorClassificationRule {
  id: string;
  name: string;
  enabled: boolean;
  
  // Matching criteria
  match: {
    type?: ErrorType[];
    severity?: ErrorSeverity[];
    source?: ErrorSource[];
    messagePattern?: string;
    stackTracePattern?: string;
    urlPattern?: string;
    statusCode?: number[];
  };
  
  // Classification
  classify: {
    severity: ErrorSeverity;
    category?: string;
    tags?: string[];
    assignTeam?: string;
    autoResolve?: boolean;
    resolutionTimeSLA?: number; // hours
  };
}

// Default classification rules
const defaultClassificationRules: ErrorClassificationRule[] = [
  {
    id: 'critical-server-errors',
    name: 'Critical Server Errors',
    enabled: true,
    match: {
      type: ['server_error', 'internal_error', 'service_unavailable'],
      statusCode: [500, 502, 503, 504]
    },
    classify: {
      severity: 'critical',
      category: 'server',
      tags: ['auto-classified', 'critical'],
      autoResolve: false,
      resolutionTimeSLA: 1
    }
  },
  {
    id: 'auth-errors',
    name: 'Authentication Errors',
    enabled: true,
    match: {
      type: ['authentication_error', 'authorization_error', 'permission_denied']
    },
    classify: {
      severity: 'high',
      category: 'security',
      tags: ['auth', 'security'],
      autoResolve: false,
      resolutionTimeSLA: 4
    }
  },
  {
    id: 'validation-errors',
    name: 'Validation Errors',
    enabled: true,
    match: {
      type: ['validation_error', 'schema_error', 'bad_request']
    },
    classify: {
      severity: 'low',
      category: 'validation',
      tags: ['client-error', 'validation'],
      autoResolve: true,
      resolutionTimeSLA: 24
    }
  },
  {
    id: 'rate-limit-errors',
    name: 'Rate Limit Errors',
    enabled: true,
    match: {
      type: ['rate_limit_error', 'quota_exceeded']
    },
    classify: {
      severity: 'medium',
      category: 'rate-limiting',
      tags: ['rate-limit', 'throttling'],
      autoResolve: true,
      resolutionTimeSLA: 8
    }
  },
  {
    id: 'database-errors',
    name: 'Database Errors',
    enabled: true,
    match: {
      type: ['database_error', 'connection_error', 'query_error', 'transaction_error']
    },
    classify: {
      severity: 'high',
      category: 'database',
      tags: ['database', 'infrastructure'],
      autoResolve: false,
      resolutionTimeSLA: 2
    }
  },
  {
    id: 'timeout-errors',
    name: 'Timeout Errors',
    enabled: true,
    match: {
      type: ['timeout_error', 'network_error']
    },
    classify: {
      severity: 'medium',
      category: 'network',
      tags: ['timeout', 'network'],
      autoResolve: true,
      resolutionTimeSLA: 8
    }
  }
];
```

---

## 4. Error Collection & Capture

### 4.1 Frontend Error Capture

```typescript
interface FrontendErrorCollector {
  // Initialize with configuration
  initialize(config: FrontendErrorConfig): void;
  
  // Capture methods
  captureException(error: Error, context?: ErrorContext): string;
  captureMessage(message: string, level: ErrorSeverity, context?: ErrorContext): string;
  captureUnhandledPromiseRejection(reason: any, promise: Promise<any>): string;
  
  // Breadcrumb tracking
  addBreadcrumb(breadcrumb: Breadcrumb): void;
  
  // Context management
  setUser(user: User | null): void;
  setTags(tags: Record<string, string>): void;
  setExtra(extra: Record<string, any>): void;
  clearContext(): void;
}

interface FrontendErrorConfig {
  // DSN (Data Source Name) - your Sentry-compatible endpoint
  dsn: string;
  
  // Environment
  environment: string;
  release: string;
  
  // Sampling
  sampleRate: number;
  
  // Traces
  tracesSampleRate: number;
  
  // Filters
  ignoreErrors: RegExp[];
  allowUrls: RegExp[];
  blockUrls: RegExp[];
  
  // Attachments
  maxBreadcrumbs: number;
  maxValueLength: number;
  
  // Before send hook
  beforeSend?: (event: ErrorEvent) => ErrorEvent | null;
  
  // Integration options
  integrations?: Integration[];
}

interface ErrorContext {
  // Request data
  request?: {
    url: string;
    method: string;
    headers: Record<string, string>;
    body?: any;
    query?: Record<string, string>;
  };
  
  // User data
  user?: {
    id: string;
    email: string;
    role: string;
    organizationId?: string;
  };
  
  // Environment
  tags?: Record<string, string>;
  extra?: Record<string, any>;
  
  // Level override
  level?: ErrorSeverity;
  
  // Fingerprint for grouping
  fingerprint?: string[];
}

interface Breadcrumb {
  type: 'navigation' | 'http' | 'console' | 'click' | 'custom';
  category: string;
  message: string;
  level: 'debug' | 'info' | 'warning' | 'error';
  data?: Record<string, any>;
  timestamp?: number;
}
```

### 4.2 Backend Error Capture

```typescript
interface BackendErrorCollector {
  // Capture methods
  captureException(error: Error, context: ErrorContext): Promise<string>;
  captureMessage(message: string, level: ErrorSeverity, context?: ErrorContext): Promise<string>;
  
  // Structured logging
  logStructured(level: 'debug' | 'info' | 'warn' | 'error', message: string, meta?: Record<string, any>): void;
  
  // Middleware factory
  errorMiddleware(): (err: Error, req: Request, res: Response, next: NextFunction) => void;
}

interface BackendErrorContext extends ErrorContext {
  // Server-specific
  server?: {
    hostname: string;
    processId: number;
    version: string;
    uptime: number;
  };
  
  // Request-specific
  request?: {
    id: string;
    method: string;
    path: string;
    query: Record<string, string>;
    headers: Record<string, string>;
    body: any;
    ip: string;
    userAgent: string;
    duration: number;
  };
  
  // Database
  database?: {
    query?: string;
    duration?: number;
    connection?: string;
  };
  
  // External services
  external?: {
    service: string;
    duration?: number;
    statusCode?: number;
  };
}

// Express/Next.js middleware example
const errorCollectorMiddleware = (collector: BackendErrorCollector) => {
  return (err: Error, req: Request, res: Response, next: NextFunction) => {
    const context: ErrorContext = {
      request: {
        id: req.headers['x-request-id'] as string || uuid(),
        method: req.method,
        path: req.path,
        query: req.query as Record<string, string>,
        headers: sanitizeHeaders(req.headers),
        body: req.body,
        ip: req.ip || req.socket.remoteAddress,
        userAgent: req.headers['user-agent']
      },
      user: req.user ? {
        id: req.user.id,
        email: req.user.email,
        role: req.user.role,
        organizationId: req.user.organizationId
      } : undefined,
      extra: {
        stack: err.stack,
        ...res.locals
      }
    };
    
    collector.captureException(err, context);
    next(err);
  };
};
```

### 4.3 Worker & Background Task Error Capture

```typescript
interface WorkerErrorCollector {
  // Capture job errors
  captureJobError(job: Job, error: Error): Promise<void>;
  
  // Capture scheduler errors
  captureSchedulerError(task: ScheduledTask, error: Error): Promise<void>;
  
  // Capture webhook processing errors
  captureWebhookError(webhook: WebhookPayload, error: Error): Promise<void>;
}

interface JobErrorContext extends ErrorContext {
  job?: {
    id: string;
    name: string;
    queue: string;
    attempts: number;
    maxAttempts: number;
    payload: any;
    result?: any;
  };
}
```

---

## 5. Error Storage & Retention

### 5.1 Storage Configuration

```typescript
interface ErrorStorageConfiguration {
  // Primary storage (hot)
  hot: HotStorageConfig;
  
  // Warm storage (archive queries)
  warm: WarmStorageConfig;
  
  // Cold storage (long-term retention)
  cold: ColdStorageConfig;
  
  // Indexing strategy
  indexing: IndexingConfig;
}

interface HotStorageConfig {
  enabled: boolean;
  type: 'postgresql' | 'mongodb';
  
  // Connection
  connection: {
    host: string;
    port: number;
    database: string;
    schema?: string;
    pool: {
      min: number;
      max: number;
    };
  };
  
  // Retention in hot storage
  retention: {
    maxAge: number; // days
    maxSize: number; // GB
    cleanupInterval: number; // hours
  };
  
  // Compression
  compression: {
    enabled: boolean;
    algorithm: 'pglz' | 'lz4' | 'zstd';
  };
}

interface WarmStorageConfig {
  enabled: boolean;
  type: 'postgresql' | 'timeseries';
  
  // Partitioning
  partitioning: {
    strategy: 'daily' | 'weekly' | 'monthly';
    column: string;
  };
  
  // Retention
  retention: {
    maxAge: number; // days
  };
  
  // Query optimization
  optimization: {
    materializedViews: boolean;
    aggregates: boolean;
  };
}

interface ColdStorageConfig {
  enabled: boolean;
  provider: 'aws_s3' | 'google_gcs' | 'azure_blob' | 'local';
  
  // Connection
  connection: {
    bucket: string;
    region: string;
    accessKeyId?: string;
    secretAccessKey?: string;
    endpoint?: string;
  };
  
  // Archive format
  format: {
    type: 'json' | 'parquet' | 'csv';
    compression: 'gzip' | 'zstd' | 'none';
    includeAttachments: boolean;
  };
  
  // Lifecycle
  lifecycle: {
    moveToColdAfter: number; // days
    deleteAfter: number; // days
  };
}

interface IndexingConfig {
  // Primary indexes
  primary: {
    fields: string[];
    unique: boolean;
  }[];
  
  // Secondary indexes
  secondary: {
    fields: string[];
    name: string;
    type: 'btree' | 'hash' | 'gin' | 'gist';
  }[];
  
  // Full-text search
  fullText: {
    enabled: boolean;
    fields: string[];
    language: string;
  };
  
  // Time-series optimization
  timeSeries: {
    enabled: boolean;
    timestampField: string;
    orderBy: string[];
  };
}
```

### 5.2 Retention Policies

```typescript
interface RetentionPolicy {
  // Hot storage (fast queries)
  hot: {
    days: number;
    maxSizeGB: number;
  };
  
  // Warm storage (slower queries)
  warm: {
    days: number;
    archiveFormat: 'json' | 'parquet';
  };
  
  // Cold storage (long-term)
  cold: {
    days: number;
    destination: 's3' | 'gcs' | 'azure' | 'none';
    deleteAfter: boolean;
  };
  
  // Per-error-type retention overrides
  overrides?: {
    errorTypes: ErrorType[];
    retention: Partial<RetentionPolicy>;
  }[];
  
  // Compliance settings
  compliance?: {
    immutable: boolean;
    legalHold: boolean;
    retentionReason?: string;
  };
}

// Default retention policy
const defaultRetentionPolicy: RetentionPolicy = {
  hot: {
    days: 30,
    maxSizeGB: 100
  },
  warm: {
    days: 90,
    archiveFormat: 'json'
  },
  cold: {
    days: 365,
    destination: 's3',
    deleteAfter: false
  },
  overrides: [
    {
      errorTypes: ['authentication_error', 'authorization_error', 'permission_denied'],
      retention: {
        hot: { days: 90, maxSizeGB: 50 },
        warm: { days: 180, archiveFormat: 'json' },
        cold: { days: 730, destination: 's3', deleteAfter: false }
      }
    },
    {
      errorTypes: ['validation_error'],
      retention: {
        hot: { days: 7, maxSizeGB: 10 },
        warm: { days: 14, archiveFormat: 'json' },
        cold: { days: 30, destination: 'none', deleteAfter: true }
      }
    }
  ]
};
```

---

## 6. Error Processing & Analysis

### 6.1 Error Grouping & Fingerprinting

```typescript
interface ErrorGrouper {
  // Generate fingerprint for error grouping
  generateFingerprint(error: ErrorLog): string;
  
  // Group similar errors
  groupErrors(errors: ErrorLog[]): ErrorGroup[];
  
  // Calculate similarity score
  calculateSimilarity(error1: ErrorLog, error2: ErrorLog): number;
}

interface ErrorGroup {
  id: string;
  fingerprint: string;
  title: string;
  
  // Statistics
  stats: {
    totalCount: number;
    uniqueUsers: number;
    firstSeen: Date;
    lastSeen: Date;
    avgFrequency: number; // per hour
  };
  
  // Errors in this group
  errors: ErrorLog[];
  
  // Status
  status: ErrorStatus;
  assignedTo?: string;
  
  // Timeline
  timeline: {
    timestamp: Date;
    count: number;
    uniqueUsers: number;
  }[];
}

// Fingerprint generation
const generateFingerprint = (error: ErrorLog): string => {
  const components = [
    // Main error type
    error.type,
    
    // Normalized message (remove numbers, UUIDs, etc.)
    normalizeMessage(error.message),
    
    // Stack trace frame (top 3 frames)
    getStackFingerprint(error.stackTrace, 3),
    
    // Location
    error.source,
    error.location?.file,
    error.location?.function
  ].filter(Boolean);
  
  return hashComponents(components);
};

const normalizeMessage = (message: string): string => {
  return message
    .replace(/\d{4,}/g, 'N')        // Replace long numbers
    .replace(/[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}/gi, 'UUID') // Replace UUIDs
    .replace(/https?:\/\/[^\s]+/g, 'URL') // Replace URLs
    .replace(/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g, 'EMAIL') // Replace emails
    .trim()
    .substring(0, 200);
};

const getStackFingerprint = (stackTrace: string, frames: number): string => {
  const lines = stackTrace.split('\n').slice(0, frames);
  return lines
    .map(line => {
      const match = line.match(/at\s+(.+?)\s+\((.+?):(\d+):(\d+)\)/);
      return match ? `${match[1]}@${match[2]}:${match[3]}` : '';
    })
    .filter(Boolean)
    .join('|');
};
```

### 6.2 Error Analytics

```typescript
interface ErrorAnalytics {
  // Real-time metrics
  getRealTimeMetrics(timeRange: TimeRange): Promise<MetricsSnapshot>;
  
  // Trend analysis
  getTrends(errorType?: ErrorType, timeRange?: TimeRange): Promise<TrendData[]>;
  
  // Impact analysis
  getImpactAnalysis(errorGroupId: string): Promise<ImpactAnalysis>;
  
  // Root cause analysis
  getRootCauseAnalysis(errorGroupId: string): Promise<RootCauseAnalysis>;
}

interface MetricsSnapshot {
  timestamp: Date;
  
  // Counts
  total: number;
  bySeverity: Record<ErrorSeverity, number>;
  byType: Record<ErrorType, number>;
  bySource: Record<ErrorSource, number>;
  
  // Rates
  errorRate: number; // errors per minute
  uniqueErrorRate: number;
  
  // Users impacted
  uniqueUsers: number;
  userImpactRate: number;
  
  // Resolution metrics
  avgResolutionTime: number;
  resolutionRate: number;
}

interface TrendData {
  timestamp: Date;
  count: number;
  previousCount: number;
  percentChange: number;
  trend: 'increasing' | 'decreasing' | 'stable';
}

interface ImpactAnalysis {
  errorGroup: ErrorGroup;
  
  // User impact
  impactedUsers: {
    total: number;
    bySeverity: Record<ErrorSeverity, number>;
    topUsers: User[];
  };
  
  // Business impact
  businessImpact: {
    failedActions: number;
    revenueImpact?: number;
    conversionImpact?: number;
  };
  
  // System impact
  systemImpact: {
    affectedEndpoints: number;
    affectedServices: number;
    degradedFeatures: string[];
  };
}

interface RootCauseAnalysis {
  errorGroup: ErrorGroup;
  
  // Most common patterns
  patterns: {
    pattern: string;
    frequency: number;
    examples: string[];
  }[];
  
  // Correlations
  correlations: {
    factor: string;
    correlation: number;
    evidence: string[];
  }[];
  
  // Suggested fix
  suggestedFix?: {
    description: string;
    confidence: number;
    codeChanges?: CodeChange[];
  };
}
```

---

## 7. Alerting & Notifications

### 7.1 Alert Rule Configuration

```typescript
interface AlertRule {
  id: string;
  name: string;
  description?: string;
  enabled: boolean;
  
  // Trigger conditions
  conditions: AlertCondition[];
  
  // Time window
  timeWindow: {
    type: 'sliding' | 'fixed';
    minutes: number;
  };
  
  // Thresholds
  thresholds: {
    errorCount?: number;
    errorRate?: number; // percentage
    uniqueUsers?: number;
    errorPercentage?: number; // errors / total requests * 100
  };
  
  // Actions
  actions: AlertAction[];
  
  // Scheduling
  schedule?: {
    enabled: boolean;
    timezone: string;
    schedule: RecurrenceRule;
  };
  
  // Escalation
  escalation?: {
    enabled: boolean;
    levels: EscalationLevel[];
  };
  
  // Filters
  filters?: {
    errorTypes?: ErrorType[];
    severity?: ErrorSeverity[];
    sources?: ErrorSource[];
    environments?: string[];
    tags?: Record<string, string>;
  };
}

interface AlertCondition {
  // Field to evaluate
  field: 'error_count' | 'error_rate' | 'unique_users' | 'severity' | 'type' | 'source' | 'custom';
  
  // Operator
  operator: 'equals' | 'not_equals' | 'greater_than' | 'less_than' | 'contains' | 'in' | 'not_in';
  
  // Value
  value: any;
  
  // Combining
  combinator?: 'and' | 'or';
}

interface AlertAction {
  // Action type
  type: 'email' | 'slack' | 'webhook' | 'pagerduty' | 'opsgenie' | 'sms' | 'jira' | 'custom';
  
  // Configuration
  config: {
    // Email
    recipients?: string[];
    subject?: string;
    template?: string;
    
    // Slack
    channel?: string;
    thread?: boolean;
    mention?: string;
    
    // Webhook
    url?: string;
    method?: 'POST' | 'PUT';
    headers?: Record<string, string>;
    body?: string;
    
    // PagerDuty
    serviceId?: string;
    escalationPolicyId?: string;
    urgency?: 'high' | 'low';
    
    // OpsGenie
    teamId?: string;
    priority?: string;
    
    // Jira
    projectKey?: string;
    issueType?: string;
    assignee?: string;
  };
  
  // Rate limiting
  rateLimit?: {
    maxPerHour: number;
    cooldown: number;
  };
}

interface EscalationLevel {
  level: number;
  afterMinutes: number;
  
  // Notify
  notifyRoles?: Role[];
  notifyUsers?: string[];
  notifyChannels?: string[];
  notifyExternal?: string[];
  
  // Actions
  actions?: AlertAction[];
}

// Example alert rules
const exampleAlertRules: AlertRule[] = [
  {
    id: 'critical-immediate',
    name: 'Critical Error - Immediate Alert',
    description: 'Alert on any critical error immediately',
    enabled: true,
    conditions: [
      { field: 'severity', operator: 'equals', value: 'critical' }
    ],
    timeWindow: { type: 'sliding', minutes: 5 },
    thresholds: { errorCount: 1 },
    actions: [
      {
        type: 'slack',
        config: { channel: '#critical-alerts', mention: '@here' }
      },
      {
        type: 'pagerduty',
        config: { urgency: 'high', escalationPolicyId: 'esc_123' }
      },
      {
        type: 'email',
        config: { recipients: ['on-call@company.com', 'engineering-lead@company.com'] }
      }
    ],
    escalation: {
      enabled: true,
      levels: [
        { level: 1, afterMinutes: 5, notifyRoles: ['OWNER'] },
        { level: 2, afterMinutes: 15, notifyRoles: ['ADMIN'] },
        { level: 3, afterMinutes: 30, notifyExternal: ['cto@company.com'] }
      ]
    }
  },
  {
    id: 'high-error-rate',
    name: 'High Error Rate',
    description: 'Alert when error rate exceeds 5%',
    enabled: true,
    conditions: [
      { field: 'error_rate', operator: 'greater_than', value: 5 }
    ],
    timeWindow: { type: 'sliding', minutes: 5 },
    thresholds: { errorRate: 5 },
    actions: [
      {
        type: 'slack',
        config: { channel: '#alerts' }
      },
      {
        type: 'email',
        config: { recipients: ['engineering@company.com'] }
      }
    ],
    filters: {
      severity: ['high', 'critical', 'medium']
    }
  },
  {
    id: 'new-error-pattern',
    name: 'New Error Pattern Detected',
    description: 'Alert when a new error fingerprint appears',
    enabled: true,
    conditions: [
      { field: 'is_new_pattern', operator: 'equals', value: true }
    ],
    timeWindow: { type: 'fixed', minutes: 60 },
    thresholds: { errorCount: 1 },
    actions: [
      {
        type: 'slack',
        config: { channel: '#new-errors' }
      }
    ]
  },
  {
    id: 'database-errors',
    name: 'Database Error Spike',
    description: 'Alert on database errors',
    enabled: true,
    conditions: [
      { field: 'type', operator: 'in', value: ['database_error', 'connection_error', 'query_error'] }
    ],
    timeWindow: { type: 'sliding', minutes: 10 },
    thresholds: { errorCount: 10 },
    actions: [
      {
        type: 'slack',
        config: { channel: '#database-alerts' }
      },
      {
        type: 'webhook',
        config: {
          url: 'https://internal.company.com/api/database-alert',
          body: JSON.stringify({ alert: 'database_error_spike', errors: '{{errors}}' })
        }
      }
    ]
  },
  {
    id: 'auth-errors',
    name: 'Authentication Error Spike',
    description: 'Potential security issue with auth errors',
    enabled: true,
    conditions: [
      { field: 'type', operator: 'in', value: ['authentication_error', 'authorization_error'] }
    ],
    timeWindow: { type: 'sliding', minutes: 15 },
    thresholds: { errorCount: 20, uniqueUsers: 10 },
    actions: [
      {
        type: 'slack',
        config: { channel: '#security-alerts', mention: '@security-team' }
      },
      {
        type: 'email',
        config: { recipients: ['security@company.com'] }
      },
      {
        type: 'pagerduty',
        config: { urgency: 'high' }
      }
    ]
  }
];
```

### 7.2 Notification Channels

```typescript
interface NotificationChannel {
  id: string;
  name: string;
  type: NotificationChannelType;
  enabled: boolean;
  
  // Configuration
  config: NotificationChannelConfig;
  
  // Rate limiting
  rateLimit?: {
    maxPerHour: number;
    maxPerDay: number;
  };
  
  // Testing
  testConfig?: {
    enabled: boolean;
    testRecipient?: string;
  };
}

type NotificationChannelType = 
  | 'email'
  | 'slack'
  | 'teams'
  | 'discord'
  | 'webhook'
  | 'pagerduty'
  | 'opsgenie'
  | 'sms'
  | 'jira'
  | 'github';

interface EmailNotificationConfig {
  provider: 'smtp' | 'sendgrid' | 'mailgun' | 'ses';
  
  // SMTP
  smtp?: {
    host: string;
    port: number;
    secure: boolean;
    auth: {
      user: string;
      pass: string;
    };
  };
  
  // API
  apiKey?: string;
  
  // Defaults
  from: string;
  fromName?: string;
  replyTo?: string;
  
  // Templates
  templates?: {
    alert: string;
    summary: string;
    digest: string;
  };
}

interface SlackNotificationConfig {
  // Authentication
  auth: {
    botToken: string;
    signingSecret: string;
  };
  
  // Defaults
  defaultChannel?: string;
  
  // Message formatting
  formatting?: {
    useBlocks: boolean;
    showCode: boolean;
    maxFields: number;
    includeStackTrace: boolean;
  };
  
  // Actions
  actions?: {
    buttons: boolean;
    links: boolean;
  };
}

interface WebhookNotificationConfig {
  // URL
  url: string;
  method: 'POST' | 'PUT' | 'PATCH';
  
  // Authentication
  auth?: {
    type: 'basic' | 'bearer' | 'api_key' | 'hmac';
    credentials: Record<string, string>;
  };
  
  // Headers
  headers?: Record<string, string>;
  
  // Body template
  body?: string;
  
  // Retry
  retry?: {
    enabled: boolean;
    maxRetries: number;
    backoff: 'exponential' | 'linear';
  };
  
  // SSL
  ssl?: {
    verify: boolean;
    caCert?: string;
  };
}
```

---

## 8. Error API Routes

### 8.1 Complete API Architecture

```
/api/admin/errors
├── GET /                          # List errors (paginated)
├── GET /stats                     # Get error statistics
├── GET /trends                    # Get error trends
├── GET /groups                    # Get error groups
├── GET /export                    # Export errors
├── GET /configure                 # Get error configuration
├── PUT /configure                 # Update error configuration
├── GET /classification-rules      # Get classification rules
├── PUT /classification-rules      # Update classification rules
├── GET /alert-rules               # Get alert rules
├── POST /alert-rules              # Create alert rule
├── PUT /alert-rules/[id]         # Update alert rule
├── DELETE /alert-rules/[id]      # Delete alert rule
├── GET /alert-rules/[id]/test    # Test alert rule
├── GET /notification-channels    # Get notification channels
├── POST /notification-channels   # Create notification channel
├── PUT /notification-channels/[id] # Update notification channel
├── DELETE /notification-channels/[id] # Delete notification channel
├── GET /notification-channels/[id]/test # Test notification channel
├── GET /retention                # Get retention policies
├── PUT /retention                # Update retention policies
├── GET /storage                  # Get storage configuration
├── PUT /storage                  # Update storage configuration
├── GET /[id]                     # Get error details
├── PUT /[id]                     # Update error
├── DELETE /[id]                  # Delete error
├── POST /[id]/resolve            # Resolve error
├── POST /[id]/reopen             # Reopen error
├── POST /[id]/assign             # Assign error
├── GET /[id]/similar             # Get similar errors
├── GET /[id]/occurrences         # Get error occurrences
├── GET /[id]/affected-users      # Get affected users
├── GET /[id]/timeline            # Get error timeline
├── POST /bulk/resolve            # Bulk resolve errors
├── POST /bulk/delete             # Bulk delete errors
├── POST /bulk/assign             # Bulk assign errors
└── POST /capture                 # Capture error (manual)

/api/admin/errors/realtime
├── GET /stream                   # Real-time error stream (SSE)
└── WS /ws                        # WebSocket for real-time updates
```

### 8.2 API Route Implementations

```typescript
// app/api/admin/errors/route.ts - List errors
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { z } from 'zod';
import { prisma } from '@/lib/db';
import { authOptions } from '@/lib/auth';
import { ErrorType, ErrorSeverity, ErrorStatus } from '@/lib/errors/types';

const listErrorsSchema = z.object({
  // Pagination
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(25),
  
  // Filters
  type: z.nativeEnum(ErrorType).optional(),
  severity: z.nativeEnum(ErrorSeverity).optional(),
  status: z.nativeEnum(ErrorStatus).optional(),
  source: z.string().optional(),
  
  // Date range
  startDate: z.coerce.date().optional(),
  endDate: z.coerce.date().optional(),
  
  // Search
  search: z.string().optional(),
  
  // Sorting
  sortBy: z.enum(['timestamp', 'severity', 'count']).default('timestamp'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
  
  // Grouping
  groupId: z.string().optional(),
  
  // Organization
  organizationId: z.string().optional()
});

export async function GET(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    if (!session) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // Check role permissions
    const allowedRoles = ['OWNER', 'ADMIN', 'MEMBER'];
    if (!allowedRoles.includes(session.user.role)) {
      return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
    }

    const searchParams = request.nextUrl.searchParams;
    const validatedParams = listErrorsSchema.parse(Object.fromEntries(searchParams));
    
    const { 
      page, 
      limit, 
      type, 
      severity, 
      status, 
      source,
      startDate, 
      endDate, 
      search,
      sortBy,
      sortOrder,
      groupId,
      organizationId 
    } = validatedParams;

    // Build where clause
    const where: any = {
      organizationId: organizationId || session.user.organizationId
    };

    if (type) where.type = type;
    if (severity) where.severity = severity;
    if (status) where.status = status;
    if (source) where.source = source;
    if (groupId) where.groupId = groupId;
    
    if (startDate || endDate) {
      where.timestamp = {};
      if (startDate) where.timestamp.gte = startDate;
      if (endDate) where.timestamp.lte = endDate;
    }

    if (search) {
      where.OR = [
        { message: { contains: search, mode: 'insensitive' } },
        { stackTrace: { contains: search, mode: 'insensitive' } },
        { context: { path: 'extra', string_contains: search } }
      ];
    }

    // Get total count
    const total = await prisma.errorLog.count({ where });

    // Get paginated errors
    const errors = await prisma.errorLog.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { [sortBy]: sortOrder },
      include: {
        user: {
          select: { id: true, name: true, email: true }
        },
        organization: {
          select: { id: true, name: true }
        },
        group: {
          select: { id: true, title: true, status: true }
        }
      }
    });

    return NextResponse.json({
      data: errors,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
        hasMore: page * limit < total
      }
    });
  } catch (error) {
    console.error('Error fetching errors:', error);
    
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation error', details: error.errors },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: 'Failed to fetch errors' },
      { status: 500 }
    );
  }
}

// app/api/admin/errors/stats/route.ts - Get error statistics
export async function GET_STATS(request: NextRequest) {
  // Implementation for error statistics
  // Returns counts by severity, type, source, status
  // Includes time-series data for trends
}

// app/api/admin/errors/groups/route.ts - Get error groups
export async function GET_GROUPS(request: NextRequest) {
  // Implementation for error grouping
  // Returns grouped errors with statistics
}

// app/api/admin/errors/configure/route.ts - Configure error settings
export async function GET_CONFIGURE() {
  // Return current error configuration
}

export async function PUT_CONFIGURE(request: NextRequest) {
  // Update error configuration
  // Validate and save configuration
  // Trigger reconfiguration if needed
}

// app/api/admin/errors/alert-rules/route.ts - Manage alert rules
export async function GET_ALERT_RULES() {
  // Return all alert rules
}

export async function POST_ALERT_RULES(request: NextRequest) {
  // Create new alert rule
  // Validate rule configuration
  // Test rule (optional)
  // Save and activate
}

// app/api/admin/errors/alert-rules/[id]/route.ts
export async function GET_ALERT_RULE(request: NextRequest, { params }: { params: { id: string } }) {
  // Get specific alert rule
}

export async function PUT_ALERT_RULE(request: NextRequest, { params }: { params: { id: string } }) {
  // Update alert rule
}

export async function DELETE_ALERT_RULE(request: NextRequest, { params }: { params: { id: string } }) {
  // Delete alert rule
}

// app/api/admin/errors/[id]/route.ts - Single error operations
export async function GET_ERROR(request: NextRequest, { params }: { params: { id: string } }) {
  // Get error details with full context
}

export async function PUT_ERROR(request: NextRequest, { params }: { params: { id: string } }) {
  // Update error (status, assignment, notes)
}

export async function DELETE_ERROR(request: NextRequest, { params }: { params: { id: string } }) {
  // Delete error (with proper authorization)
}

// app/api/admin/errors/capture/route.ts - Manual error capture
export async function POST_CAPTURE(request: NextRequest) {
  // Manually capture an error
  // Used for testing or manual error reporting
}
```

---

## 9. Error Database Schema

### 9.1 Prisma Schema Models

```prisma
// Core Error Log Model
model ErrorLog {
  // Identification
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Error classification
  type           ErrorType
  severity       ErrorSeverity
  status         ErrorStatus @default(NEW)
  source         ErrorSource
  
  // Error details
  message        String   @db.Text
  stackTrace     String?  @db.Text
  fingerprint    String   @index
  
  // Location
  location       Json?
  // {
  //   file: string,
  //   line: number,
  //   column: number,
  //   function: string,
  //   url?: string
  // }
  
  // Context
  context        Json?
  // {
  //   request: { ... },
  //   user: { ... },
  //   extra: { ... },
  //   environment: { ... }
  // }
  
  // User information
  userId         String?
  user           User?    @relation(fields: [userId], references: [id])
  
  // Grouping
  groupId        String?
  group          ErrorGroup? @relation(fields: [groupId], references: [id])
  
  // Metadata
  tags           String[] @default([])
  environment    String?
  release        String?
  
  // Timing
  timestamp      DateTime @default(now()) @index
  resolvedAt     DateTime?
  
  // Assignment
  assignedTo     String?
  assignedUser   User?    @relation("ErrorAssignments", fields: [assignedTo], references: [id])
  
  // Notes
  notes          ErrorNote[]
  
  // Alert status
  alertFired     Boolean  @default(false)
  alertId        String?
  
  // Retention
  archivedAt     DateTime?
  
  // Relations
  occurrences    ErrorOccurrence[]
  breadcrumbs    ErrorBreadcrumb[]
  
  // Indexes
  @@index([organizationId, timestamp])
  @@index([organizationId, severity])
  @@index([organizationId, type])
  @@index([fingerprint, timestamp])
  @@index([groupId])
  @@index([status])
  @@index([userId])
  
  @@map("error_logs")
}

// Error Group - Groups similar errors
model ErrorGroup {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Group identification
  fingerprint     String   @unique
  title           String
  
  // Statistics
  totalCount      Int      @default(1)
  uniqueUsers     Int      @default(1)
  firstSeen       DateTime @default(now())
  lastSeen        DateTime @default(now())
  
  // Status
  status          ErrorStatus @default(NEW)
  
  // Assignment
  assignedTo      String?
  
  // Severity (highest in group)
  severity        ErrorSeverity
  
  // Type
  type            ErrorType
  source          ErrorSource
  
  // Error logs in this group
  errors          ErrorLog[]
  
  // Timeline (daily aggregates)
  timeline        ErrorGroupTimeline[]
  
  // Resolution
  resolvedAt      DateTime?
  resolutionNotes String?  @db.Text
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  // Indexes
  @@index([organizationId])
  @@index([status])
  @@index([lastSeen])
  
  @@map("error_groups")
}

// Error Occurrence - Individual occurrence in a group
model ErrorOccurrence {
  id              String   @id @default(cuid())
  errorLogId      String
  errorLog        ErrorLog @relation(fields: [errorLogId], references: [id])
  groupId         String
  group           ErrorGroup @relation(fields: [groupId], references: [id])
  
  // Timing
  timestamp       DateTime @default(now())
  
  // Context at time of occurrence
  context         Json?
  
  // User at time of occurrence
  userId          String?
  
  @@unique([errorLogId, groupId])
  @@index([groupId, timestamp])
  
  @@map("error_occurrences")
}

// Error Note - User notes on errors
model ErrorNote {
  id          String   @id @default(cuid())
  errorLogId  String
  errorLog    ErrorLog @relation(fields: [errorLogId], references: [id])
  
  // Content
  content     String   @db.Text
  
  // Author
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  
  // Timestamps
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([errorLogId])
  @@map("error_notes")
}

// Error Breadcrumb - User action trail
model ErrorBreadcrumb {
  id          String   @id @default(cuid())
  errorLogId  String
  errorLog    ErrorLog @relation(fields: [errorLogId], references: [id])
  
  // Breadcrumb data
  type        String
  category    String
  message     String
  level       String
  data        Json?
  timestamp   DateTime @default(now())
  
  // Order
  order       Int
  
  @@index([errorLogId])
  @@map("error_breadcrumbs")
}

// Error Group Timeline - Daily aggregates
model ErrorGroupTimeline {
  id          String   @id @default(cuid())
  groupId     String
  group       ErrorGroup @relation(fields: [groupId], references: [id])
  
  // Date
  date        DateTime @db.Date
  
  // Counts
  count       Int      @default(0)
  uniqueUsers Int      @default(0)
  
  // Severity breakdown
  critical    Int      @default(0)
  high        Int      @default(0)
  medium      Int      @default(0)
  low         Int      @default(0)
  
  @@unique([groupId, date])
  @@index([groupId])
  
  @@map("error_group_timeline")
}

// Alert Rule
model AlertRule {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Rule definition
  name            String
  description     String?
  enabled         Boolean  @default(true)
  
  // Conditions
  conditions      Json
  // [{
  //   field: string,
  //   operator: string,
  //   value: any,
  //   combinator?: 'and' | 'or'
  // }]
  
  // Thresholds
  thresholds      Json
  // {
  //   errorCount?: number,
  //   errorRate?: number,
  //   uniqueUsers?: number,
  //   timeWindow: number
  // }
  
  // Time window
  timeWindow      Json
  // {
  //   type: 'sliding' | 'fixed',
  //   minutes: number
  // }
  
  // Filters
  filters         Json?
  // {
  //   errorTypes?: string[],
  //   severity?: string[],
  //   sources?: string[]
  // }
  
  // Actions
  actions         Json
  // [{
  //   type: string,
  //   config: object,
  //   rateLimit?: object
  // }]
  
  // Escalation
  escalation      Json?
  
  // Scheduling
  schedule        Json?
  
  // Statistics
  lastTriggered   DateTime?
  triggerCount    Int      @default(0)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  createdBy       String
  
  // Indexes
  @@index([organizationId, enabled])
  
  @@map("alert_rules")
}

// Notification Channel
model NotificationChannel {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Configuration
  name            String
  type            String
  enabled         Boolean  @default(true)
  
  // Channel config
  config          Json
  // {
  //   // Email
  //   recipients?: string[],
  //   from?: string,
  //   
  //   // Slack
  //   channel?: string,
  //   botToken?: string,
  //   
  //   // Webhook
  //   url?: string,
  //   method?: string,
  //   headers?: object,
  //   body?: string
  // }
  
  // Rate limiting
  rateLimit       Json?
  
  // Testing
  testConfig      Json?
  
  // Status
  lastTested      DateTime?
  testStatus      String?
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  // Indexes
  @@index([organizationId, enabled])
  
  @@map("notification_channels")
}

// Error Configuration - System-wide settings
model ErrorConfiguration {
  id              String   @id @default(cuid())
  organizationId String   @unique
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Capture settings
  capture         Json
  
  // Storage settings
  storage         Json
  
  // Processing settings
  processing      Json
  
  // Alerting settings
  alerting        Json
  
  // Access control
  access          Json
  
  // Performance
  performance     Json
  
  // Integrations
  integrations    Json?
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  updatedBy       String
  
  @@map("error_configurations")
}

// Error Classification Rule
model ErrorClassificationRule {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Rule definition
  name            String
  enabled         Boolean  @default(true)
  
  // Matching
  match           Json
  
  // Classification
  classify        Json
  
  // Priority
  priority        Int      @default(0)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  // Indexes
  @@index([organizationId, enabled, priority])
  
  @@map("error_classification_rules")
}

// Alert History
model AlertHistory {
  id              String   @id @default(cuid())
  alertRuleId     String
  alertRule       AlertRule @relation(fields: [alertRuleId], references: [id])
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Trigger details
  triggeredAt    DateTime @default(now())
  errors         Json
  // [{
  //   id: string,
  //   message: string,
  //   severity: string
  // }]
  
  // Actions taken
  actions         Json
  // [{
  //   type: string,
  //   status: 'sent' | 'failed' | 'rate_limited',
  //   details?: object
  // }]
  
  // Resolution
  resolvedAt      DateTime?
  resolvedBy      String?
  
  // Indexes
  @@index([alertRuleId, triggeredAt])
  @@index([organizationId, triggeredAt])
  
  @@map("alert_history")
}

// Enum definitions
enum ErrorType {
  JAVASCRIPT_ERROR
  SERVER_ERROR
  DATABASE_ERROR
  API_ERROR
  AUTHENTICATION_ERROR
  AUTHORIZATION_ERROR
  VALIDATION_ERROR
  RATE_LIMIT_ERROR
  TIMEOUT_ERROR
  NETWORK_ERROR
  CUSTOM_ERROR
  // ... more types
}

enum ErrorSeverity {
  CRITICAL
  HIGH
  MEDIUM
  LOW
  INFO
}

enum ErrorStatus {
  NEW
  REOPENED
  IN_PROGRESS
  RESOLVED
  IGNORED
  ARCHIVED
}

enum ErrorSource {
  FRONTEND
  BACKEND
  WORKER
  CRON
  WEBHOOK
  API
  DATABASE
  EXTERNAL
}
```

---

## 10. Edge Cases

### 10.1 Error Collection Edge Cases

```typescript
const errorCollectionEdgeCases = [
  {
    scenario: 'Error in error handler',
    description: 'Error occurs while processing another error',
    handling: 'Use isolated error boundary, log to secondary destination'
  },
  {
    scenario: 'Error message too large',
    description: 'Error message exceeds maximum size limit',
    handling: 'Truncate with indicator, store full in separate storage'
  },
  {
    scenario: 'Circular reference in context',
    description: 'Error context contains circular references',
    handling: 'Use safe stringify with cycle detection'
  },
  {
    scenario: 'Sensitive data in error',
    description: 'PII or secrets accidentally in error message',
    handling: 'Apply regex patterns to mask sensitive fields'
  },
  {
    scenario: 'Stack trace unavailable',
    description: 'Error occurs without stack trace (older browsers, minified code)',
    handling: 'Generate synthetic stack trace, request context only'
  },
  {
    scenario: 'Network failure during capture',
    description: 'Cannot send error to backend',
    handling: 'Queue locally, retry with exponential backoff'
  },
  {
    scenario: 'Browser storage full',
    description: 'LocalStorage/IndexedDB is full',
    handling: 'Clear old errors, implement sampling'
  },
  {
    scenario: 'CORS blocked request',
    description: 'Error capture request blocked by CORS',
    handling: 'Use JSONP fallback, report via beacon API'
  },
  {
    scenario: 'Concurrent error flood',
    description: 'Large number of errors in short time',
    handling: 'Implement queuing, sampling, aggregation'
  },
  {
    scenario: 'Invalid error object',
    description: 'Non-Error object thrown',
    handling: 'Convert to Error, extract message from toString()'
  }
];
```

### 10.2 Storage & Retrieval Edge Cases

```typescript
const storageEdgeCases = [
  {
    scenario: 'Database connection failure',
    description: 'Cannot connect to error database',
    handling: 'Switch to fallback storage, buffer in memory, alert on prolonged failure'
  },
  {
    scenario: 'Storage quota exceeded',
    description: 'Hot storage reaches capacity',
    handling: 'Trigger archive process, apply retention policy, reject new errors with notification'
  },
  {
    scenario: 'Partition query failure',
    description: 'Query across partitions fails',
    handling: 'Query each partition separately, merge results'
  },
  {
    scenario: 'Archive retrieval timeout',
    description: 'Requesting archived error times out',
    handling: 'Provide async retrieval, cache recent archive metadata'
  },
  {
    scenario: 'Concurrent update conflict',
    description: 'Two users update same error simultaneously',
    handling: 'Use optimistic locking, last-write-wins with notification'
  },
  {
    scenario: 'Bulk delete in progress',
    description: 'Delete operation running while querying',
    handling: 'Use consistent reads, avoid scanning deleted partitions'
  },
  {
    scenario: 'Index corruption',
    description: 'Error log index is corrupted',
    handling: 'Rebuild indexes, use full table scan as fallback'
  }
];
```

### 10.3 Alerting & Notification Edge Cases

```typescript
const alertingEdgeCases = [
  {
    scenario: 'Notification channel failure',
    description: 'Slack/Email/PagerDuty API unavailable',
    handling: 'Retry with backoff, fallback to alternate channel, queue notifications'
  },
  {
    scenario: 'Alert rate limit exceeded',
    description: 'Too many alerts triggered',
    handling: 'Aggregate similar alerts, use cooldown, prioritize by severity'
  },
  {
    scenario: 'Escalation loop',
    description: 'Alert keeps re-escalating without resolution',
    handling: 'Max escalation limit, suppress after hours, require manual acknowledgment'
  },
  {
    scenario: 'Webhook retry storm',
    description: 'Webhook endpoint keeps failing causing retries',
    handling: 'Exponential backoff, dead letter queue, disable after max retries'
  },
  {
    scenario: 'Alert rule conflicts',
    description: 'Multiple rules trigger for same error',
    handling: 'Deduplicate at notification level, combine actions'
  },
  {
    scenario: 'Timezone handling',
    description: 'Alert schedule in different timezone',
    handling: 'Store timezone with schedule, convert on evaluation'
  },
  {
    scenario: 'Alert action timeout',
    description: 'Alert action takes too long',
    handling: 'Async actions, timeout with status tracking'
  }
];
```

### 10.4 Performance Edge Cases

```typescript
const performanceEdgeCases = [
  {
    scenario: 'High error volume during incident',
    description: 'Errors spike during major outage',
    handling: 'Adaptive sampling, queue-based processing, scale read replicas'
  },
  {
    scenario: 'Complex aggregation query',
    description: 'Query across millions of errors times out',
    handling: 'Pre-aggregate data, use materialized views, pagination'
  },
  {
    scenario: 'Real-time connection overload',
    description: 'Too many clients connected for real-time stream',
    handling: 'Connection limits, load balance across instances, message batching'
  },
  {
    scenario: 'Memory pressure from caching',
    description: 'Error caching consumes too much memory',
    handling: 'LRU eviction, memory limits, offload to Redis'
  },
  {
    scenario: 'Background job backlog',
    description: 'Error processing jobs accumulate',
    handling: 'Scale workers, prioritize recent errors, skip less critical processing'
  }
];
```

---

## 11. Performance & Caching

### 11.1 Performance Optimization Strategies

```typescript
interface ErrorPerformanceConfig {
  // Async processing queue
  queue: {
    type: 'memory' | 'redis' | 'kafka';
    maxSize: number;
    batchSize: number;
    flushInterval: number;
    concurrency: number;
  };
  
  // Caching layers
  caching: {
    // In-memory cache
    memory: {
      enabled: boolean;
      maxSize: number;
      ttl: number;
      evictionPolicy: 'lru' | 'lfu' | 'fifo';
    };
    
    // Redis cache
    redis: {
      enabled: boolean;
      ttl: number;
      keyPrefix: string;
    };
    
    // CDN cache for static error pages
    cdn: {
      enabled: boolean;
      cacheControl: string;
    };
  };
  
  // Query optimization
  queryOptimization: {
    // Pagination
    maxPageSize: number;
    defaultPageSize: number;
    
    // Field selection
    defaultFields: string[];
    maxFields: number;
    
    // Date range limits
    maxDateRangeDays: number;
    
    // Result limits
    maxExportRows: number;
    
    // Query timeouts
    queryTimeout: number;
  };
  
  // Database optimization
  database: {
    // Connection pool
    pool: {
      min: number;
      max: number;
    };
    
    // Read replicas
    readReplicas: {
      enabled: boolean;
      hosts: string[];
      lagThreshold: number;
    };
    
    // Partitioning
    partitioning: {
      strategy: 'daily' | 'weekly' | 'monthly';
      column: string;
      retention: number;
    };
    
    // Indexing
    indexes: {
      automatic: boolean;
      analyzeOnQuery: boolean;
    };
  };
}

// Performance monitoring
interface ErrorPerformanceMetrics {
  // Capture metrics
  capture: {
    errorsPerSecond: number;
    avgCaptureLatency: number;
    captureErrors: number;
    queueDepth: number;
  };
  
  // Storage metrics
  storage: {
    writesPerSecond: number;
    avgWriteLatency: number;
    readLatency: P95Latency;
    storageUsedGB: number;
  };
  
  // Query metrics
  query: {
    queriesPerSecond: number;
    avgQueryLatency: number;
    slowQueries: number;
    cacheHitRate: number;
  };
  
  // Alerting metrics
  alerting: {
    alertsTriggered: number;
    alertsDelivered: number;
    alertFailures: number;
    avgDeliveryLatency: number;
  };
}
```

### 11.2 Caching Strategy

```typescript
const errorCachingStrategy = {
  // Real-time error feed
  'errors:realtime': {
    ttl: 0, // No cache, always fresh
    invalidation: 'event-based'
  },
  
  // Error groups (change infrequently)
  'errors:groups': {
    ttl: 300, // 5 minutes
    invalidation: 'on-new-error-in-group'
  },
  
  // Error statistics (aggregated hourly)
  'errors:stats:hourly': {
    ttl: 3600, // 1 hour
    invalidation: 'on-hour-change'
  },
  
  // Error trends (daily aggregates)
  'errors:trends:daily': {
    ttl: 86400, // 24 hours
    invalidation: 'on-day-change'
  },
  
  // User error history
  'errors:user:{userId}': {
    ttl: 300, // 5 minutes
    invalidation: 'on-new-error'
  },
  
  // Organization summary
  'errors:org:{orgId}:summary': {
    ttl: 60, // 1 minute
    invalidation: 'on-error-change'
  },
  
  // Configuration
  'errors:config:{orgId}': {
    ttl: 3600, // 1 hour
    invalidation: 'on-config-change'
  }
};
```

---

## 12. Integration Examples

### 12.1 Frontend Integration

```typescript
// errors/client.ts - Frontend Error Client
import { BrowserClient, Scope } from '@sentry/nextjs';

class ErrorCollector {
  private client: BrowserClient;
  private scope: Scope;
  
  constructor(config: ErrorCollectorConfig) {
    this.client = new BrowserClient({
      dsn: config.dsn,
      environment: config.environment,
      release: config.release,
      sampleRate: config.sampleRate,
      integrations: [
        new CaptureConsole(),
        new GlobalHandlers(),
        new LinkedErrors(),
        new RewriteFrames()
      ],
      beforeSend: (event) => this.beforeSend(event)
    });
    
    this.scope = new Scope();
    this.client.init();
  }
  
  // Set user context
  setUser(user: User | null) {
    this.scope.setUser(user ? {
      id: user.id,
      email: user.email,
      role: user.role
    } : null);
  }
  
  // Set organization context
  setOrganization(organization: Organization) {
    this.scope.setTag('organizationId', organization.id);
    this.scope.setTag('organizationName', organization.name);
  }
  
  // Add breadcrumb
  addBreadcrumb(breadcrumb: Breadcrumb) {
    this.scope.addBreadcrumb(breadcrumb);
  }
  
  // Capture error
  captureException(error: Error, context?: ErrorContext) {
    this.scope.setExtra('context', context);
    this.client.captureException(error);
  }
  
  // Capture message
  captureMessage(message: string, level: SeverityLevel = 'error') {
    this.client.captureMessage(message, level);
  }
  
  // Before send hook
  private beforeSend(event: Event): Event | null {
    // Remove sensitive data
    if (event.request?.headers) {
      delete event.request.headers['authorization'];
      delete event.request.headers['cookie'];
    }
    
    // Add custom data
    event.tags = {
      ...event.tags,
      appVersion: process.env.NEXT_PUBLIC_APP_VERSION
    };
    
    return event;
  }
}

// Usage in Next.js
const errorCollector = new ErrorCollector({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NEXT_PUBLIC_APP_ENV,
  release: process.env.NEXT_PUBLIC_APP_VERSION,
  sampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0
});

// Error boundary component
class ErrorBoundary extends React.Component<Props, State> {
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    errorCollector.captureException(error, {
      componentStack: errorInfo.componentStack
    });
  }
  
  render() {
    return this.props.children;
  }
}
```

### 12.2 Backend Integration

```typescript
// errors/server.ts - Backend Error Collector
import { NodeClient } from '@sentry/node';
import { v4 as uuid } from 'uuid';

class BackendErrorCollector {
  private client: NodeClient;
  
  constructor(config: BackendErrorConfig) {
    this.client = new NodeClient({
      dsn: config.dsn,
      environment: config.environment,
      release: config.release,
      tracesSampleRate: config.tracesSampleRate,
      integrations: [
        new ExpressIntegration(),
        new PostgresIntegration(),
        new RedisIntegration()
      ]
    });
  }
  
  // Express middleware
  errorMiddleware() {
    return (err: Error, req: Request, res: Response, next: NextFunction) => {
      const errorId = this.captureException(err, {
        request: {
          id: req.headers['x-request-id'] || uuid(),
          method: req.method,
          path: req.path,
          query: req.query,
          headers: this.sanitizeHeaders(req.headers),
          body: req.body,
          ip: req.ip
        },
        user: req.user ? {
          id: req.user.id,
          email: req.user.email,
          role: req.user.role
        } : undefined
      });
      
      // Attach error ID to response
      res.setHeader('X-Error-ID', errorId);
      next(err);
    };
  }
  
  // Capture with full context
  captureException(error: Error, context?: ErrorContext): string {
    const eventId = this.client.captureException(error, {
      extra: {
        ...context,
        timestamp: new Date().toISOString()
      },
      tags: {
        source: 'backend',
        environment: process.env.NODE_ENV
      }
    });
    
    return eventId;
  }
  
  // Sanitize headers
  private sanitizeHeaders(headers: IncomingHttpHeaders): Record<string, string> {
    const sanitized = { ...headers };
    const sensitive = ['authorization', 'cookie', 'x-api-key', 'x-secret'];
    
    sensitive.forEach(key => {
      if (sanitized[key]) {
        sanitized[key] = '[REDACTED]';
      }
    });
    
    return sanitized;
  }
}

// Usage in Express/Next.js API routes
const errorCollector = new BackendErrorCollector({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.APP_VERSION,
  tracesSampleRate: 0.1
});

// Apply middleware
app.use(errorCollector.errorMiddleware());
```

### 12.3 Custom Error Capture API

```typescript
// Custom API for capturing errors from any source
app.post('/api/errors/capture', async (req, res) => {
  const { 
    type,
    severity,
    message,
    stackTrace,
    location,
    context,
    userId,
    organizationId,
    timestamp,
    fingerprint
  } = req.body;
  
  // Validate
  if (!type || !message) {
    return res.status(400).json({ error: 'Missing required fields' });
  }
  
  // Create error log
  const errorLog = await prisma.errorLog.create({
    data: {
      type,
      severity: severity || 'medium',
      status: 'new',
      source: determineSource(req),
      message: truncate(message, 10000),
      stackTrace: truncate(stackTrace, 20000),
      location,
      context: sanitizeContext(context),
      userId,
      organizationId,
      timestamp: timestamp ? new Date(timestamp) : new Date(),
      fingerprint: fingerprint || generateFingerprint(message, stackTrace)
    }
  });
  
  // Check alert rules
  await checkAlertRules(errorLog);
  
  return res.status(201).json({ id: errorLog.id });
});
```

### 12.4 Slack Integration Example

```typescript
// Slack notification for errors
async function sendSlackAlert(alert: Alert, channel: string, webhookUrl: string) {
  const error = alert.errors[0];
  
  const message = {
    channel,
    username: 'Error Bot',
    icon_emoji: getSeverityEmoji(alert.severity),
    blocks: [
      {
        type: 'header',
        text: {
          type: 'plain_text',
          text: `:rotating_light: ${alert.title}`,
          emoji: true
        }
      },
      {
        type: 'section',
        fields: [
          {
            type: 'mrkdwn',
            text: `*Severity:*\n${alert.severity.toUpperCase()}`
          },
          {
            type: 'mrkdwn',
            text: `*Errors:*\n${alert.errorCount}`
          },
          {
            type: 'mrkdwn',
            text: `*First Seen:*\n<${alert.firstSeen}|${formatDate(alert.firstSeen)}>`
          },
          {
            type: 'mrkdwn',
            text: `*Last Seen:*\n<${alert.lastSeen}|${formatDate(alert.lastSeen)}>`
          }
        ]
      },
      {
        type: 'section',
        text: {
          type: 'mrkdwn',
          text: `\`\`\`${error.message}\n${error.stackTrace?.substring(0, 500)}\`\`\``
        }
      },
      {
        type: 'actions',
        elements: [
          {
            type: 'button',
            text: {
              type: 'plain_text',
              text: 'View in Dashboard'
            },
            url: `${process.env.APP_URL}/admin/errors/${alert.id}`,
            style: 'primary'
          },
          {
            type: 'button',
            text: {
              type: 'plain_text',
              text: 'Resolve'
            },
            action_id: `resolve_${alert.id}`
          }
        ]
      }
    ]
  };
  
  await fetch(webhookUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(message)
  });
}
```

---

## Summary

This comprehensive Error Logs System configuration flow covers every aspect of error tracking, from collection to alerting:

### Key Components

1. **Error Capture** - Frontend, backend, and worker error collection with full context
2. **Classification** - Automatic categorization by type, severity, and source
3. **Storage** - Multi-tier storage with hot/warm/cold retention policies
4. **Processing** - Intelligent grouping, deduplication, and analytics
5. **Alerting** - Flexible rules with multiple notification channels
6. **API** - Complete REST API for error management
7. **Database** - Comprehensive Prisma schema with all necessary models

### Admin Capabilities

- Configure error capture settings and filters
- Set up retention policies and storage
- Create and manage alert rules
- Configure notification channels
- Define classification rules
- View analytics and trends
- Bulk manage errors

### User Capabilities

- View errors affecting their organization
- See personal error history
- Receive alerts (based on role)
- Add notes to errors
- Resolve/manage assigned errors
- Access error dashboards

### Integration Points

- Frontend SDK (Sentry-compatible)
- Backend SDK (Node.js)
- Custom capture API
- Slack/Teams notifications
- PagerDuty/OpsGenie integration
- Webhook support
- Jira/GitHub integration
