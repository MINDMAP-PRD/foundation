# Comprehensive Notification System Configuration Flow

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Universal Notification Configuration Pattern](#2-universal-notification-configuration-pattern)
3. [Notification Categories & Types](#3-notification-categories--types)
4. [In-App Notifications](#4-in-app-notifications)
5. [External Notifications (Email/SMS/Push)](#5-external-notifications-emailsmspush)
6. [Nudging System](#6-nudging-system)
7. [Gamification System](#7-gamification-system)
8. [Notification API Routes](#8-notification-api-routes)
9. [Notification Database Schema](#9-notification-database-schema)
10. [Edge Cases](#10-edge-cases)
11. [Performance & Caching](#11-performance--caching)
12. [Integration Examples](#12-integration-examples)

---

## 1. System Overview

### 1.1 Core Principles

The Notification System provides comprehensive, multi-channel notification capabilities with intelligent nudging, in-app messaging, external alerts, and gamification elements. It serves as the central communication hub for user engagement, action reminders, and achievement tracking.

**Core Design Principles:**

- **Multi-Channel Delivery** - Notifications reach users through in-app, email, SMS, push, and external integrations
- **Intelligent Routing** - Smart delivery based on user preferences, urgency, and context
- **Personalization** - Customizable templates, timing, and content per user/role
- **Nudging Engine** - Automated reminders with configurable frequency, escalation, and smart delays
- **Gamification** - Points, badges, achievements, streaks, and leaderboards to drive engagement
- **Privacy-First** - User control over notification preferences with consent management
- **Performance Isolation** - Notification delivery never impacts application performance

### 1.2 System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    NOTIFICATION SYSTEM ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │
│  │   Event     │    │  Trigger    │    │  Scheduled  │                    │
│  │   Sources   │    │  Processing │    │  Jobs       │                    │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                    │
│         │                  │                  │                            │
│         └──────────────────┼──────────────────┘                            │
│                            │                                                │
│                    ┌───────▼───────┐                                        │
│                    │  Notification │                                        │
│                    │  Engine       │                                        │
│                    └───────┬───────┘                                        │
│                            │                                                │
│  ┌─────────────────────────┼─────────────────────────────────────────┐    │
│  │                         │                                         │    │
│  ▼                         ▼                                         ▼    │
│ ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│ │  In-App    │    │  Nudging    │    │ Gamification│    │   External  │ │
│ │  Channel   │    │  Engine     │    │   Engine    │    │   Channels  │ │
│ └──────┬──────┘    └──────┬──────┘    └──────┬──────┘    └──────┬──────┘ │
│        │                  │                  │                  │         │
│  ┌─────▼─────┐      ┌─────▼─────┐      ┌─────▼─────┐      ┌─────▼─────┐  │
│  │ Real-time │      │  Reminder │      │   Points  │      │  Email    │  │
│  │  Feed     │      │  Queue    │      │  & Badges │      │  SMS      │  │
│  └───────────┘      └───────────┘      └───────────┘      │  Push     │  │
│                                                          │  Slack    │  │
│                                                          └───────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Notification Flow Pipeline

```
Event/Trigger → Queue → Process → Route → Deliver → Track → Update User State
                    ↓
              ┌─────┴─────┐
              │  Nudge    │
              │  Check    │
              └─────┬─────┘
                    ↓
              ┌─────┴─────┐
              │ Gamify    │
              │ Check     │
              └───────────┘
```

---

## 2. Universal Notification Configuration Pattern

### 2.1 NotificationConfiguration Interface

```typescript
interface NotificationConfiguration {
  // In-App Settings
  inApp: InAppConfig;
  
  // External Channels
  external: ExternalConfig;
  
  // Nudging System
  nudging: NudgingConfig;
  
  // Gamification
  gamification: GamificationConfig;
  
  // Delivery Settings
  delivery: DeliveryConfig;
  
  // User Preferences
  preferences: NotificationPreferences;
  
  // Access Control
  access: NotificationAccessConfig;
  
  // Performance Settings
  performance: NotificationPerformanceConfig;
}

interface InAppConfig {
  // Real-time notifications
  realTime: {
    enabled: boolean;
    provider: 'websocket' | 'sse' | 'polling';
    connectionTimeout: number;
    reconnectAttempts: number;
  };
  
  // Notification center
  center: {
    enabled: boolean;
    maxDisplay: number;
    retention: number; // days
    showUnreadCount: boolean;
  };
  
  // Types enabled
  types: InAppNotificationType[];
  
  // Styling
  styling: {
    position: 'top-right' | 'bottom-right' | 'bottom-left';
    duration: number; // ms
    animation: 'slide' | 'fade' | 'none';
  };
}

interface ExternalConfig {
  // Email notifications
  email: {
    enabled: boolean;
    provider: 'smtp' | 'sendgrid' | 'mailgun' | 'ses' | 'postmark';
    config: EmailProviderConfig;
    templates: EmailTemplateConfig;
    digestSettings: EmailDigestSettings;
  };
  
  // SMS notifications
  sms: {
    enabled: boolean;
    provider: 'twilio' | 'aws-sns' | 'nexmo';
    config: SMSProviderConfig;
    templates: SMSTemplateConfig;
    maxLength: number;
  };
  
  // Push notifications
  push: {
    enabled: boolean;
    provider: 'fcm' | 'apns' | 'web-push';
    config: PushProviderConfig;
    templates: PushTemplateConfig;
    icon: string;
  };
  
  // External integrations
  integrations: {
    slack: SlackNotificationConfig;
    teams: TeamsNotificationConfig;
    discord: DiscordNotificationConfig;
    webhook: WebhookNotificationConfig;
  };
}

interface NudgingConfig {
  // Enable/disable nudging
  enabled: boolean;
  
  // Global nudging settings
  global: {
    maxNudgesPerDay: number;
    maxNudgesPerAction: number;
    quietHours: QuietHoursConfig;
    snoozeDuration: number; // hours
  };
  
  // Nudge types
  types: NudgeTypeConfig[];
  
  // Escalation
  escalation: {
    enabled: boolean;
    escalationRules: EscalationRule[];
    maxEscalations: number;
  };
  
  // Smart nudging
  smartNudging: {
    enabled: boolean;
    userActivityBased: boolean;
    contextAware: boolean;
    optimalTimeDetection: boolean;
  };
  
  // Action required notifications
  actionRequired: {
    enabled: boolean;
    deadlineWarning: number; // hours before deadline
    overdueWarning: number; // hours after deadline
    escalationOnIgnore: boolean;
  };
}

interface GamificationConfig {
  // Enable gamification
  enabled: boolean;
  
  // Points system
  points: {
    enabled: boolean;
    categories: PointCategoryConfig[];
    earningRules: PointEarningRule[];
    redemptionOptions: PointRedemption[];
  };
  
  // Badges & Achievements
  badges: {
    enabled: boolean;
    badgeDefinitions: BadgeDefinition[];
    unlockCriteria: BadgeUnlockCriteria[];
    displayOptions: BadgeDisplayOptions;
  };
  
  // Streaks
  streaks: {
    enabled: boolean;
    streakTypes: StreakConfig[];
    freezeOptions: StreakFreezeConfig;
    rewards: StreakReward[];
  };
  
  // Leaderboards
  leaderboards: {
    enabled: boolean;
    types: LeaderboardConfig[];
    refreshInterval: number; // minutes
    historyDays: number;
  };
  
  // Levels & Progression
  levels: {
    enabled: boolean;
    levelThresholds: LevelThreshold[];
    levelBenefits: LevelBenefit[];
  };
  
  // Social features
  social: {
    enabled: boolean;
    sharingEnabled: boolean;
    compareEnabled: boolean;
    collaborationRewards: boolean;
  };
}

interface DeliveryConfig {
  // Priority handling
  priority: {
    high: DeliveryPriority;
    normal: DeliveryPriority;
    low: DeliveryPriority;
  };
  
  // Retry logic
  retry: {
    maxAttempts: number;
    backoffStrategy: 'exponential' | 'linear' | 'fixed';
    initialDelay: number; // ms
    maxDelay: number; // ms
  };
  
  // Rate limiting
  rateLimit: {
    perUser: {
      email: number;
      sms: number;
      push: number;
      inApp: number;
    };
    global: {
      email: number;
      sms: number;
      push: number;
    };
  };
  
  // Deduplication
  deduplication: {
    enabled: boolean;
    window: number; // seconds
    groupingKeyFields: string[];
  };
  
  // Scheduling
  scheduling: {
    enabled: boolean;
    timezoneAware: boolean;
    defaultTimezone: string;
    businessHoursOnly: boolean;
  };
}

interface NotificationPreferences {
  // Default preferences
  defaults: ChannelPreferences;
  
  // Per-notification-type overrides
  typePreferences: Record<NotificationType, ChannelPreferences>;
  
  // Quiet hours per channel
  quietHours: ChannelQuietHours[];
  
  // Do not disturb
  dnd: {
    enabled: boolean;
    schedule?: RecurrenceSchedule;
  };
  
  // Frequency caps
  frequencyCaps: FrequencyCap[];
  
  // Consent management
  consent: {
    required: boolean;
    granular: boolean;
    categories: ConsentCategory[];
  };
}

interface NotificationAccessConfig {
  // Who can view notifications
  viewRoles: Role[];
  
  // Who can configure notifications
  configRoles: Role[];
  
  // Who can send notifications
  sendRoles: Role[];
  
  // Who can access analytics
  analyticsRoles: Role[];
  
  // Gamification access
  gamificationRoles: Role[];
  
  // Field-level restrictions
  fieldRestrictions: FieldRestriction[];
}
```

### 2.2 Default Notification Configuration

```typescript
const defaultNotificationConfiguration: NotificationConfiguration = {
  inApp: {
    realTime: {
      enabled: true,
      provider: 'websocket',
      connectionTimeout: 30000,
      reconnectAttempts: 5
    },
    center: {
      enabled: true,
      maxDisplay: 50,
      retention: 30,
      showUnreadCount: true
    },
    types: [
      'info',
      'success',
      'warning',
      'error',
      'achievement',
      'reminder',
      'social',
      'system'
    ],
    styling: {
      position: 'bottom-right',
      duration: 5000,
      animation: 'slide'
    }
  },
  external: {
    email: {
      enabled: true,
      provider: 'sendgrid',
      config: {
        apiKey: process.env.SENDGRID_API_KEY,
        fromEmail: 'noreply@company.com',
        fromName: 'Platform'
      },
      templates: {},
      digestSettings: {
        enabled: true,
        frequency: 'daily',
        time: '09:00',
        types: ['summary', 'alerts', 'updates']
      }
    },
    sms: {
      enabled: false,
      provider: 'twilio',
      config: {},
      templates: {},
      maxLength: 160
    },
    push: {
      enabled: true,
      provider: 'fcm',
      config: {},
      templates: {},
      icon: '/icons/push-icon.png'
    },
    integrations: {}
  },
  nudging: {
    enabled: true,
    global: {
      maxNudgesPerDay: 10,
      maxNudgesPerAction: 3,
      quietHours: {
        enabled: true,
        start: '22:00',
        end: '08:00'
      },
      snoozeDuration: 24
    },
    types: [],
    escalation: {
      enabled: true,
      escalationRules: [],
      maxEscalations: 2
    },
    smartNudging: {
      enabled: true,
      userActivityBased: true,
      contextAware: true,
      optimalTimeDetection: true
    },
    actionRequired: {
      enabled: true,
      deadlineWarning: 24,
      overdueWarning: 48,
      escalationOnIgnore: true
    }
  },
  gamification: {
    enabled: true,
    points: {
      enabled: true,
      categories: [
        { id: 'activity', name: 'Activity', multiplier: 1.0 },
        { id: 'engagement', name: 'Engagement', multiplier: 1.5 },
        { id: 'contribution', name: 'Contribution', multiplier: 2.0 },
        { id: 'achievement', name: 'Achievement', multiplier: 3.0 }
      ],
      earningRules: [],
      redemptionOptions: []
    },
    badges: {
      enabled: true,
      badgeDefinitions: [],
      unlockCriteria: [],
      displayOptions: {
        showOnProfile: true,
        showOnFeed: true,
        animationEnabled: true
      }
    },
    streaks: {
      enabled: true,
      streakTypes: [],
      freezeOptions: {
        enabled: true,
        maxFreezes: 3,
        freezeCost: 100
      },
      rewards: []
    },
    leaderboards: {
      enabled: true,
      types: [],
      refreshInterval: 60,
      historyDays: 30
    },
    levels: {
      enabled: true,
      levelThresholds: [],
      levelBenefits: []
    },
    social: {
      enabled: true,
      sharingEnabled: true,
      compareEnabled: true,
      collaborationRewards: true
    }
  },
  delivery: {
    priority: {
      high: { immediate: true, channels: ['inApp', 'push', 'email'] },
      normal: { immediate: false, channels: ['inApp'], delayed: ['email'] },
      low: { immediate: false, channels: ['inApp'], delayed: ['email', 'digest'] }
    },
    retry: {
      maxAttempts: 3,
      backoffStrategy: 'exponential',
      initialDelay: 1000,
      maxDelay: 60000
    },
    rateLimit: {
      perUser: {
        email: 10,
        sms: 5,
        push: 20,
        inApp: 50
      },
      global: {
        email: 1000,
        sms: 100,
        push: 500
      }
    },
    deduplication: {
      enabled: true,
      window: 300,
      groupingKeyFields: ['type', 'userId', 'actionId']
    },
    scheduling: {
      enabled: true,
      timezoneAware: true,
      defaultTimezone: 'UTC',
      businessHoursOnly: false
    }
  },
  preferences: {
    defaults: {
      email: 'instant',
      sms: 'never',
      push: 'instant',
      inApp: 'instant'
    },
    typePreferences: {},
    quietHours: [],
    dnd: {
      enabled: false
    },
    frequencyCaps: [],
    consent: {
      required: true,
      granular: true,
      categories: ['marketing', 'product', 'security', 'social']
    }
  },
  access: {
    viewRoles: ['OWNER', 'ADMIN', 'MEMBER', 'VIEWER'],
    configRoles: ['OWNER', 'ADMIN'],
    sendRoles: ['OWNER', 'ADMIN', 'SYSTEM'],
    analyticsRoles: ['OWNER', 'ADMIN'],
    gamificationRoles: ['OWNER', 'ADMIN', 'MEMBER', 'VIEWER'],
    fieldRestrictions: []
  },
  performance: {
    asyncProcessing: true,
    queueSize: 10000,
    batchSize: 100,
    flushInterval: 5000
  }
};
```

---

## 3. Notification Categories & Types

### 3.1 Notification Type Taxonomy

```typescript
enum NotificationType {
  // System notifications
  SYSTEM = 'system',
  SECURITY = 'security',
  MAINTENANCE = 'maintenance',
  BILLING = 'billing',
  
  // User activity
  USER_ACTION = 'user_action',
  USER_MENTION = 'user_mention',
  USER_INVITE = 'user_invite',
  USER_FOLLOW = 'user_follow',
  
  // Team & Collaboration
  TEAM_INVITE = 'team_invite',
  TEAM_UPDATE = 'team_update',
  PROJECT_INVITE = 'project_invite',
  PROJECT_UPDATE = 'project_update',
  COMMENT = 'comment',
  REVIEW = 'review',
  APPROVAL = 'approval',
  
  // Actions required
  ACTION_REQUIRED = 'action_required',
  DEADLINE = 'deadline',
  TASK_ASSIGNED = 'task_assigned',
  
  // Gamification
  ACHIEVEMENT_UNLOCKED = 'achievement_unlocked',
  LEVEL_UP = 'level_up',
  POINTS_EARNED = 'points_earned',
  STREAK_MILESTONE = 'streak_milestone',
  BADGE_EARNED = 'badge_earned',
  LEADERBOARD_RANK = 'leaderboard_rank',
  
  // Alerts
  ERROR_ALERT = 'error_alert',
  WARNING_ALERT = 'warning_alert',
  INFO_ALERT = 'info_alert',
  
  // Nudges
  REMINDER = 'reminder',
  FOLLOW_UP = 'follow_up',
  ENCOURAGEMENT = 'encouragement',
  
  // Marketing
  PROMOTION = 'promotion',
  NEWSLETTER = 'newsletter',
  FEATURE_ANNOUNCEMENT = 'feature_announcement',
  
  // Digest
  DAILY_DIGEST = 'daily_digest',
  WEEKLY_DIGEST = 'weekly_digest',
  MONTHLY_DIGEST = 'monthly_digest'
}

enum NotificationPriority {
  CRITICAL = 'critical',
  HIGH = 'high',
  NORMAL = 'normal',
  LOW = 'low'
}

enum NotificationStatus {
  PENDING = 'pending',
  SENT = 'sent',
  DELIVERED = 'delivered',
  READ = 'read',
  ARCHIVED = 'archived',
  FAILED = 'failed',
  CANCELLED = 'cancelled'
}

enum NotificationChannel {
  IN_APP = 'inApp',
  EMAIL = 'email',
  SMS = 'sms',
  PUSH = 'push',
  SLACK = 'slack',
  TEAMS = 'teams',
  DISCORD = 'discord',
  WEBHOOK = 'webhook'
}

enum NudgeType {
  REMINDER = 'reminder',
  FOLLOW_UP = 'follow_up',
  URGENT = 'urgent',
  ENCOURAGEMENT = 'encouragement',
  SOCIAL = 'social',
  ACTION_REQUIRED = 'action_required',
  DEADLINE = 'deadline',
  CHECK_IN = 'check_in'
}

enum GamificationEventType {
  ACTION_COMPLETED = 'action_completed',
  STREAK_CONTINUED = 'streak_continued',
  STREAK_BROKEN = 'streak_broken',
  ACHIEVEMENT_UNLOCKED = 'achievement_unlocked',
  LEVEL_UP = 'level_up',
  POINTS_EARNED = 'points_earned',
  BADGE_EARNED = 'badge_earned',
  MILESTONE_REACHED = 'milestone_reached',
  LEADERBOARD_CHANGED = 'leaderboard_changed'
}
```

### 3.2 Notification Category Configuration

```typescript
interface NotificationCategory {
  id: string;
  name: string;
  description: string;
  
  // Channels
  channels: NotificationChannel[];
  
  // Default settings
  defaultSettings: {
    enabled: boolean;
    priority: NotificationPriority;
    channels: ChannelPreference[];
    frequency?: string;
    quietHoursRespected: boolean;
  };
  
  // Templates
  templates: Record<NotificationChannel, NotificationTemplate>;
  
  // Nudging
  nudging?: {
    enabled: boolean;
    nudgeType: NudgeType;
    initialDelay: number; // hours
    maxNudges: number;
    escalationDelay: number; // hours
  };
  
  // Gamification
  gamification?: {
    points: number;
    badgeIds?: string[];
    streakType?: string;
  };
  
  // Access
  roles: Role[];
  
  // Filters
  filters?: {
    userTypes?: string[];
    plans?: string[];
    environments?: string[];
  };
}

// Example categories
const notificationCategories: NotificationCategory[] = [
  {
    id: 'security',
    name: 'Security Alerts',
    description: 'Important security notifications and alerts',
    channels: ['inApp', 'email', 'push'],
    defaultSettings: {
      enabled: true,
      priority: 'critical',
      channels: ['inApp', 'email', 'push'],
      quietHoursRespected: false
    },
    templates: {},
    roles: ['OWNER', 'ADMIN', 'MEMBER']
  },
  {
    id: 'achievements',
    name: 'Achievements & Rewards',
    description: 'Badges, points, and achievement notifications',
    channels: ['inApp', 'email'],
    defaultSettings: {
      enabled: true,
      priority: 'normal',
      channels: ['inApp', 'email']
    },
    templates: {},
    nudging: {
      enabled: false,
      nudgeType: 'encouragement',
      initialDelay: 24,
      maxNudges: 2,
      escalationDelay: 48
    },
    gamification: {
      points: 50,
      badgeIds: ['achievement_1', 'achievement_5', 'achievement_10']
    },
    roles: ['OWNER', 'ADMIN', 'MEMBER', 'VIEWER']
  },
  {
    id: 'action_required',
    name: 'Action Required',
    description: 'Tasks and actions that require user attention',
    channels: ['inApp', 'push', 'email'],
    defaultSettings: {
      enabled: true,
      priority: 'high',
      channels: ['inApp', 'push', 'email']
    },
    templates: {},
    nudging: {
      enabled: true,
      nudgeType: 'action_required',
      initialDelay: 1,
      maxNudges: 5,
      escalationDelay: 24
    },
    roles: ['OWNER', 'ADMIN', 'MEMBER']
  },
  {
    id: 'team_activity',
    name: 'Team Activity',
    description: 'Updates about team members and collaborations',
    channels: ['inApp'],
    defaultSettings: {
      enabled: true,
      priority: 'normal',
      channels: ['inApp']
    },
    templates: {},
    roles: ['OWNER', 'ADMIN', 'MEMBER']
  }
];
```

---

## 4. In-App Notifications

### 4.1 In-App Notification Interface

```typescript
interface InAppNotification {
  id: string;
  organizationId: string;
  
  // Content
  type: NotificationType;
  category: string;
  priority: NotificationPriority;
  title: string;
  message: string;
  
  // Action
  action?: {
    type: 'link' | 'action' | 'modal';
    url?: string;
    actionId?: string;
    data?: Record<string, any>;
  };
  
  // Sender
  sender?: {
    type: 'user' | 'system' | 'bot';
    id?: string;
    name?: string;
    avatar?: string;
  };
  
  // Media
  media?: {
    type: 'image' | 'video' | 'icon';
    url: string;
    alt?: string;
  };
  
  // Styling
  styling?: {
    variant: 'info' | 'success' | 'warning' | 'error';
    icon?: string;
    color?: string;
  };
  
  // State
  status: NotificationStatus;
  readAt?: Date;
  dismissedAt?: Date;
  
  // User
  userId: string;
  
  // Gamification
  gamification?: {
    pointsEarned?: number;
    badgeIds?: string[];
    levelGained?: number;
    streakMilestone?: number;
  };
  
  // Context
  context?: Record<string, any>;
  
  // Timing
  createdAt: Date;
  deliveredAt?: Date;
  expiresAt?: Date;
}
```

### 4.2 Real-Time Notification System

```typescript
interface RealTimeNotificationService {
  // Connection management
  connect(userId: string): Promise<Connection>;
  disconnect(userId: string): void;
  
  // Send notification
  send(notification: InAppNotification): Promise<void>;
  sendBatch(notifications: InAppNotification[]): Promise<void>;
  
  // Subscribe to channels
  subscribe(userId: string, channels: string[]): void;
  unsubscribe(userId: string, channels: string[]): void;
  
  // Presence
  getOnlineUsers(): string[];
  isUserOnline(userId: string): boolean;
}

interface NotificationCenter {
  // Get notifications
  getNotifications(userId: string, options: GetNotificationsOptions): Promise<NotificationList>;
  getUnreadCount(userId: string): Promise<number>;
  getUnreadNotifications(userId: string, limit: number): Promise<InAppNotification[]>;
  
  // Mark as read
  markAsRead(notificationId: string): Promise<void>;
  markAllAsRead(userId: string): Promise<void>;
  
  // Dismiss
  dismiss(notificationId: string): Promise<void>;
  dismissAll(userId: string): Promise<void>;
  
  // Archive
  archive(notificationId: string): Promise<void>;
  
  // Delete
  delete(notificationId: string): Promise<void>;
  
  // Bulk operations
  bulkMarkAsRead(notificationIds: string[]): Promise<void>;
  bulkDelete(notificationIds: string[]): Promise<void>;
}

interface GetNotificationsOptions {
  // Pagination
  page?: number;
  limit?: number;
  
  // Filters
  type?: NotificationType[];
  category?: string[];
  priority?: NotificationPriority[];
  status?: NotificationStatus[];
  read?: boolean;
  
  // Date range
  startDate?: Date;
  endDate?: Date;
  
  // Sorting
  sortBy?: 'createdAt' | 'priority' | 'type';
  sortOrder?: 'asc' | 'desc';
}
```

### 4.3 In-App Notification UI Components

```typescript
// Toast notification component
interface ToastNotification {
  id: string;
  type: 'info' | 'success' | 'warning' | 'error' | 'achievement';
  title: string;
  message?: string;
  icon?: string;
  action?: {
    label: string;
    onClick: () => void;
  };
  duration?: number;
  dismissible?: boolean;
  position?: 'top-right' | 'bottom-right' | 'bottom-left';
}

// Notification bell dropdown
interface NotificationBellProps {
  userId: string;
  showUnreadCount: boolean;
  maxDisplay?: number;
  onNotificationClick?: (notification: InAppNotification) => void;
  onMarkAllRead?: () => void;
}

// Notification center page
interface NotificationCenterPageProps {
  userId: string;
  filters?: NotificationFilter[];
  sortBy?: string;
  onAction?: (action: NotificationAction) => void;
}
```

---

## 5. External Notifications (Email/SMS/Push)

### 5.1 Email Notification System

```typescript
interface EmailNotification {
  // Recipients
  to: string[];
  cc?: string[];
  bcc?: string[];
  replyTo?: string;
  
  // Content
  subject: string;
  body: string;
  html?: string;
  text?: string;
  
  // Template
  template?: {
    id: string;
    variables: Record<string, any>;
  };
  
  // Attachments
  attachments?: Attachment[];
  
  // Headers
  headers?: Record<string, string>;
  
  // Priority
  priority?: 'high' | 'normal' | 'low';
  
  // Scheduled
  scheduledAt?: Date;
  
  // Metadata
  metadata?: Record<string, any>;
}

interface EmailTemplate {
  id: string;
  organizationId?: string;
  name: string;
  description?: string;
  
  // Content
  subject: string;
  body: string;
  html?: string;
  text?: string;
  
  // Variables
  variables: TemplateVariable[];
  
  // Design
  design?: {
    layout: string;
    theme: string;
    logo?: string;
    colors: Record<string, string>;
  };
  
  // Categories
  categories: string[];
  
  // Active
  active: boolean;
  
  // Versions
  versions: EmailTemplateVersion[];
  
  // Testing
  testConfig?: {
    testEmails: string[];
    testVariables: Record<string, any>;
  };
}

interface EmailDigestSettings {
  enabled: boolean;
  
  // Frequency
  frequency: 'realtime' | 'hourly' | 'daily' | 'weekly' | 'monthly';
  time?: string;
  dayOfWeek?: number;
  dayOfMonth?: number;
  
  // Types included
  types: string[];
  
  // Layout
  layout: 'single' | 'combined';
  maxItems: number;
  
  // Filters
  filters?: {
    minPriority?: NotificationPriority;
    excludeRead?: boolean;
    categories?: string[];
  };
  
  // Customization
  includeHeader: boolean;
  includeSummary: boolean;
  includeActionButtons: boolean;
}
```

### 5.2 SMS Notification System

```typescript
interface SMSNotification {
  // Recipients
  to: string[];
  
  // Content
  body: string;
  
  // Template
  template?: {
    id: string;
    variables: Record<string, any>;
  };
  
  // Media (MMS)
  media?: string[];
  
  // Scheduling
  scheduledAt?: Date;
  
  // Options
  options?: {
    maxPrice?: number;
    validateOnly?: boolean;
  };
}

interface SMSTemplate {
  id: string;
  organizationId?: string;
  name: string;
  
  // Content
  body: string;
  
  // Variables
  variables: TemplateVariable[];
  
  // Character limit handling
  characterLimit: number;
  splitStrategy: 'truncate' | 'multiple' | 'concat';
  
  // Active
  active: boolean;
}
```

### 5.3 Push Notification System

```typescript
interface PushNotification {
  // Recipients
  tokens: string[];
  
  // Content
  title: string;
  body: string;
  subtitle?: string;
  
  // Media
  image?: string;
  icon?: string;
  badge?: string;
  sound?: string;
  
  // Action
  actions?: PushAction[];
  
  // Data
  data?: Record<string, any>;
  
  // Options
  options?: {
    priority?: 'high' | 'normal';
    ttl?: number;
    collapseKey?: string;
    mutableContent?: boolean;
    contentAvailable?: boolean;
  };
  
  // Scheduling
  scheduledAt?: Date;
}

interface PushAction {
  id: string;
  title: string;
  icon?: string;
  foreground?: boolean;
}

interface PushSubscription {
  id: string;
  userId: string;
  
  // Token
  token: string;
  provider: 'fcm' | 'apns' | 'web-push';
  
  // Device info
  device?: {
    type: 'ios' | 'android' | 'web';
    model?: string;
    osVersion?: string;
    appVersion?: string;
  };
  
  // Preferences
  preferences: {
    enabled: boolean;
    quietHours?: QuietHoursConfig;
    types: string[];
  };
  
  // Status
  active: boolean;
  lastUsed: Date;
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}
```

---

## 6. Nudging System

### 6.1 Nudge Configuration

```typescript
interface NudgeConfiguration {
  // Enable nudging
  enabled: boolean;
  
  // Global settings
  global: NudgeGlobalSettings;
  
  // Nudge definitions
  nudges: NudgeDefinition[];
  
  // Escalation rules
  escalation: NudgeEscalation[];
  
  // Smart nudging
  smartNudging: SmartNudgingConfig;
}

interface NudgeGlobalSettings {
  // Limits
  maxNudgesPerDay: number;
  maxNudgesPerAction: number;
  maxTotalNudges: number;
  
  // Timing
  initialDelay: number; // hours after trigger
  minimumDelay: number; // minimum hours between nudges
  maximumDelay: number; // maximum hours before forcing nudge
  
  // Quiet hours
  quietHours: {
    enabled: boolean;
    start: string; // HH:mm
    end: string;
    timezone: string;
    respectForced?: boolean;
  };
  
  // Snooze
  snooze: {
    enabled: boolean;
    defaultDuration: number; // hours
    maxDuration: number;
  };
  
  // User control
  userCanDisable: boolean;
  userCanSnooze: boolean;
}

interface NudgeDefinition {
  id: string;
  name: string;
  description?: string;
  
  // Trigger
  trigger: NudgeTrigger;
  
  // Content
  content: {
    title: string;
    message: string;
    actionLabel?: string;
    actionUrl?: string;
  };
  
  // Channels
  channels: NudgeChannel[];
  
  // Timing
  timing: {
    initialDelay: number; // hours
    interval: number; // hours between nudges
    maxCount: number;
    expiresAfter?: number; // hours after trigger
  };
  
  // Conditions
  conditions?: NudgeCondition[];
  
  // Priority
  priority: 'low' | 'medium' | 'high' | 'urgent';
  
  // Gamification
  gamification?: {
    pointsIfCompleted: number;
    penaltyIfIgnored: number;
  };
  
  // Targeting
  targeting?: {
    roles?: string[];
    plans?: string[];
    segments?: string[];
  };
  
  // Active
  active: boolean;
}

interface NudgeTrigger {
  type: 'event' | 'schedule' | 'inactivity' | 'deadline' | 'custom';
  
  // Event trigger
  event?: {
    eventType: string;
    conditions?: Record<string, any>;
  };
  
  // Schedule trigger
  schedule?: {
    cron?: string;
    time?: string;
    timezone?: string;
    daysOfWeek?: number[];
  };
  
  // Inactivity trigger
  inactivity?: {
    afterDays: number;
    action?: string;
  };
  
  // Deadline trigger
  deadline?: {
    field: string;
    offset: number; // hours before/after
    recalculateOnView?: boolean;
  };
}

interface NudgeChannel {
  type: 'inApp' | 'email' | 'push' | 'sms';
  
  // Template override
  template?: {
    subject?: string;
    title?: string;
    message?: string;
  };
  
  // Delivery
  delivery: {
    immediate: boolean;
    fallback?: string;
  };
}

interface NudgeCondition {
  field: string;
  operator: 'equals' | 'notEquals' | 'contains' | 'greaterThan' | 'lessThan' | 'in';
  value: any;
  then?: {
    modifyTiming?: {
      delay?: number;
      interval?: number;
    };
    changePriority?: string;
  };
}

interface NudgeEscalation {
  id: string;
  nudgeId: string;
  
  // Trigger
  afterCount: number;
  afterHours: number;
  
  // Escalated content
  content: {
    title: string;
    message: string;
    urgencyLevel: number; // 1-10
  };
  
  // Channels to escalate to
  channels: string[];
  
  // Notify
  notifyRoles?: string[];
  notifyUsers?: string[];
}

interface SmartNudgingConfig {
  enabled: boolean;
  
  // Learning
  learning: {
    enabled: boolean;
    modelType: 'simple' | 'ml';
    features: string[];
    minDataPoints: number;
  };
  
  // User activity based
  activityBased: {
    enabled: boolean;
    considerLoginTime: boolean;
    considerActiveHours: boolean;
    considerDeviceType: boolean;
  };
  
  // Context aware
  contextAware: {
    enabled: boolean;
    factors: string[];
  };
  
  // Optimal time detection
  optimalTime: {
    enabled: boolean;
    algorithm: 'historical' | 'predictive';
    lookbackDays: number;
  };
  
  // Fatigue management
  fatigueManagement: {
    enabled: boolean;
    maxConsecutive: number;
    cooldownPeriod: number;
  };
}
```

### 6.2 Nudge Examples

```typescript
const exampleNudges: NudgeDefinition[] = [
  {
    id: 'complete-profile',
    name: 'Complete Your Profile',
    description: 'Nudge users to complete their profile',
    trigger: {
      type: 'event',
      event: {
        eventType: 'user_registered',
        conditions: { profileCompletePercent: { lessThan: 50 } }
      }
    },
    content: {
      title: 'Complete your profile',
      message: 'Adding a photo and bio helps your team know you better!',
      actionLabel: 'Complete Profile',
      actionUrl: '/settings/profile'
    },
    channels: [
      { type: 'inApp', delivery: { immediate: false, fallback: 'email' } }
    ],
    timing: {
      initialDelay: 24,
      interval: 48,
      maxCount: 5,
      expiresAfter: 168 // 7 days
    },
    priority: 'low',
    gamification: {
      pointsIfCompleted: 25
    },
    active: true
  },
  {
    id: 'task-deadline',
    name: 'Task Deadline Warning',
    description: 'Remind users of upcoming task deadlines',
    trigger: {
      type: 'deadline',
      deadline: {
        field: 'task.dueDate',
        offset: -24 // 24 hours before
      }
    },
    content: {
      title: '⏰ Task due soon',
      message: 'Your task "{{task.title}}" is due in {{hoursRemaining}} hours',
      actionLabel: 'View Task',
      actionUrl: '/tasks/{{task.id}}'
    },
    channels: [
      { type: 'inApp', delivery: { immediate: true } },
      { type: 'push', delivery: { immediate: true } },
      { type: 'email', delivery: { immediate: false } }
    ],
    timing: {
      initialDelay: 0,
      interval: 12,
      maxCount: 3
    },
    conditions: [
      { field: 'task.priority', operator: 'equals', value: 'high' }
    ],
    priority: 'high',
    active: true
  },
  {
    id: 'weekly-checkin',
    name: 'Weekly Engagement Check-in',
    description: 'Encourage users to engage weekly',
    trigger: {
      type: 'schedule',
      schedule: {
        time: '10:00',
        timezone: 'user',
        daysOfWeek: [1] // Monday
      }
    },
    content: {
      title: '👋 How was your week?',
      message: "Let's see what you accomplished this week!",
      actionLabel: 'View Dashboard',
      actionUrl: '/dashboard'
    },
    channels: [
      { type: 'inApp', delivery: { immediate: true } }
    ],
    timing: {
      initialDelay: 0,
      interval: 168,
      maxCount: 1,
      expiresAfter: 24
    },
    priority: 'low',
    gamification: {
      pointsIfCompleted: 10
    },
    targeting: {
      segments: ['active_users']
    },
    active: true
  },
  {
    id: 'incomplete-action',
    name: 'Incomplete Action Follow-up',
    description: 'Follow up on incomplete user actions',
    trigger: {
      type: 'inactivity',
      inactivity: {
        afterDays: 3,
        action: 'action_started'
      }
    },
    content: {
      title: '👀 You left something unfinished',
      message: 'Your action "{{action.name}}" is waiting for you',
      actionLabel: 'Continue',
      actionUrl: '/actions/{{action.id}}'
    },
    channels: [
      { type: 'inApp', delivery: { immediate: true } },
      { type: 'email', delivery: { immediate: false, fallback: 'inApp' } }
    ],
    timing: {
      initialDelay: 72,
      interval: 48,
      maxCount: 3
    },
    priority: 'medium',
    active: true
  }
];
```

---

## 7. Gamification System

### 7.1 Points & Earning Rules

```typescript
interface PointsSystem {
  // Enable points
  enabled: boolean;
  
  // Point categories
  categories: PointCategory[];
  
  // Earning rules
  earningRules: PointEarningRule[];
  
  // Redemption options
  redemption: PointRedemption[];
  
  // Limits
  limits: {
    maxPointsPerDay: number;
    maxPointsPerAction: number;
    maxTotalPoints: number;
    pointExpiry: number; // days, 0 = never
  };
}

interface PointCategory {
  id: string;
  name: string;
  description?: string;
  multiplier: number;
  icon?: string;
  color?: string;
}

interface PointEarningRule {
  id: string;
  name: string;
  description?: string;
  
  // Trigger
  trigger: {
    type: 'action' | 'event' | 'achievement' | 'streak' | 'custom';
    action?: string;
    event?: string;
  };
  
  // Points
  points: number;
  categoryId?: string;
  
  // Limits
  limits?: {
    maxTimes: number;
    period?: 'daily' | 'weekly' | 'monthly' | 'forever';
  };
  
  // Conditions
  conditions?: Record<string, any>;
  
  // Bonus
  bonus?: {
    enabled: boolean;
    multiplier: number;
    condition: string;
  };
  
  // Active
  active: boolean;
}

interface PointRedemption {
  id: string;
  name: string;
  description?: string;
  
  // Cost
  cost: number;
  
  // Type
  type: 'feature' | 'reward' | 'discount' | 'custom';
  
  // Value
  value: {
    type: 'string' | 'number' | 'feature';
    data: any;
  };
  
  // Limits
  limits?: {
    maxRedemptions: number;
    perUserLimit?: number;
    startDate?: Date;
    endDate?: Date;
  };
  
  // Active
  active: boolean;
}

// Example earning rules
const exampleEarningRules: PointEarningRule[] = [
  {
    id: 'daily-login',
    name: 'Daily Login',
    description: 'Earn points for logging in each day',
    trigger: { type: 'action', action: 'user.login' },
    points: 10,
    categoryId: 'activity',
    limits: { maxTimes: 1, period: 'daily' },
    active: true
  },
  {
    id: 'complete-task',
    name: 'Complete Task',
    description: 'Earn points for completing tasks',
    trigger: { type: 'action', action: 'task.complete' },
    points: 50,
    categoryId: 'contribution',
    active: true
  },
  {
    id: 'invite-member',
    name: 'Invite Team Member',
    description: 'Earn points for inviting new members',
    trigger: { type: 'action', action: 'team.invite' },
    points: 100,
    categoryId: 'engagement',
    limits: { maxTimes: 5, period: 'monthly' },
    bonus: {
      enabled: true,
      multiplier: 2,
      condition: 'member.completedOnboarding'
    },
    active: true
  },
  {
    id: 'streak-7',
    name: '7-Day Streak',
    description: 'Bonus for 7-day activity streak',
    trigger: { type: 'streak', event: 'streak.7' },
    points: 200,
    categoryId: 'achievement',
    active: true
  }
];
```

### 7.2 Badges & Achievements

```typescript
interface BadgeSystem {
  // Enable badges
  enabled: boolean;
  
  // Badge definitions
  badges: BadgeDefinition[];
  
  // Display options
  display: BadgeDisplayConfig;
}

interface BadgeDefinition {
  id: string;
  name: string;
  description: string;
  
  // Tier
  tier: 'bronze' | 'silver' | 'gold' | 'platinum' | 'diamond';
  
  // Icon
  icon: string;
  image?: string;
  
  // Rarity
  rarity: number; // 0-100, lower = rarer
  
  // Criteria
  criteria: BadgeCriteria[];
  
  // Rewards
  rewards?: {
    points: number;
    exclusiveFeatures?: string[];
  };
  
  // Display
  display: {
    showName: boolean;
    showDescription: boolean;
    showProgress: boolean;
    animation?: string;
  };
  
  // Limits
  limits?: {
    maxCount: number;
    releaseDate?: Date;
    expirationDate?: Date;
  };
  
  // Active
  active: boolean;
}

interface BadgeCriteria {
  // Type
  type: 'action' | 'count' | 'streak' | 'achievement' | 'custom';
  
  // Action criteria
  action?: {
    action: string;
    count?: number;
    timeframe?: 'day' | 'week' | 'month' | 'all';
  };
  
  // Count criteria
  count?: {
    metric: string;
    operator: 'gte' | 'lte' | 'eq';
    value: number;
    timeframe?: string;
  };
  
  // Streak criteria
  streak?: {
    type: string;
    days: number;
  };
  
  // Achievement criteria
  achievement?: {
    achievementId: string;
  };
  
  // Progress tracking
  progressable: boolean;
  progressFormula?: string;
}

// Example badges
const exampleBadges: BadgeDefinition[] = [
  {
    id: 'first-login',
    name: 'First Steps',
    description: 'Complete your first login',
    tier: 'bronze',
    icon: '🚀',
    rarity: 90,
    criteria: [
      { type: 'action', action: 'user.login', count: 1, progressable: false }
    ],
    rewards: { points: 25 },
    display: { showName: true, showDescription: true, showProgress: false },
    active: true
  },
  {
    id: 'task-master',
    name: 'Task Master',
    description: 'Complete 100 tasks',
    tier: 'gold',
    icon: '📋',
    rarity: 20,
    criteria: [
      { type: 'count', metric: 'tasks.completed', operator: 'gte', value: 100, progressable: true }
    ],
    rewards: { points: 500, exclusiveFeatures: ['golden-badge-frame'] },
    display: { showName: true, showDescription: true, showProgress: true, animation: 'sparkle' },
    active: true
  },
  {
    id: 'week-warrior',
    name: 'Week Warrior',
    description: 'Maintain a 7-day activity streak',
    tier: 'silver',
    icon: '🔥',
    rarity: 40,
    criteria: [
      { type: 'streak', type: 'activity', days: 7, progressable: true }
    ],
    rewards: { points: 200 },
    display: { showName: true, showDescription: true, showProgress: true },
    active: true
  },
  {
    id: 'team-builder',
    name: 'Team Builder',
    description: 'Invite 10 team members who complete onboarding',
    tier: 'platinum',
    icon: '👥',
    rarity: 10,
    criteria: [
      { 
        type: 'count', 
        metric: 'team.invites.completed', 
        operator: 'gte', 
        value: 10,
        timeframe: 'all',
        progressable: true 
      }
    ],
    rewards: { points: 1000, exclusiveFeatures: ['platinum-badge-frame', 'priority-support'] },
    display: { showName: true, showDescription: true, showProgress: true, animation: 'glow' },
    limits: { maxCount: 100 },
    active: true
  }
];
```

### 7.3 Streaks & Leaderboards

```typescript
interface StreakSystem {
  // Enable streaks
  enabled: boolean;
  
  // Streak types
  types: StreakType[];
  
  // Freeze options
  freezes: StreakFreezeConfig;
  
  // Rewards
  rewards: StreakReward[];
}

interface StreakType {
  id: string;
  name: string;
  description: string;
  
  // Activity tracking
  activity: {
    type: string;
    action: string;
    frequency: 'daily' | 'weekly';
  };
  
  // Milestones
  milestones: number[]; // days
  
  // Display
  icon: string;
  color?: string;
  
  // Active
  active: boolean;
}

interface LeaderboardSystem {
  // Enable leaderboards
  enabled: boolean;
  
  // Types
  types: LeaderboardType[];
  
  // Display
  display: {
    showTop: number;
    showUserRank: boolean;
    showPoints: boolean;
    showTrend: boolean;
  };
  
  // History
  history: {
    enabled: boolean;
    days: number;
  };
}

interface LeaderboardType {
  id: string;
  name: string;
  description: string;
  
  // Scope
  scope: 'global' | 'organization' | 'team' | 'friends';
  
  // Ranking by
  rankingBy: 'points' | 'badges' | 'streak' | 'actions';
  
  // Time period
  period: 'all' | 'daily' | 'weekly' | 'monthly' | 'yearly';
  
  // Refresh
  refreshInterval: number; // minutes
  
  // Active
  active: boolean;
}
```

### 7.4 Levels & Progression

```typescript
interface LevelSystem {
  // Enable levels
  enabled: boolean;
  
  // Level thresholds
  levels: LevelDefinition[];
  
  // Benefits per level
  benefits: LevelBenefit[];
  
  // Display
  display: {
    showLevel: boolean;
    showProgress: boolean;
    showNextLevel: boolean;
    animation?: string;
  };
}

interface LevelDefinition {
  level: number;
  name: string;
  title: string;
  
  // Points required
  pointsRequired: number;
  
  // Icon
  icon?: string;
  color?: string;
  
  // Badge
  badge?: string;
}

interface LevelBenefit {
  level: number;
  benefits: {
    type: string;
    name: string;
    description: string;
    value: any;
  }[];
}

// Default level progression
const defaultLevels: LevelDefinition[] = [
  { level: 1, name: 'Newcomer', title: 'Newcomer', pointsRequired: 0, icon: '🌱' },
  { level: 2, name: 'Explorer', title: 'Explorer', pointsRequired: 100, icon: '🌿' },
  { level: 3, name: 'Contributor', title: 'Contributor', pointsRequired: 500, icon: '🌳' },
  { level: 4, name: 'Specialist', title: 'Specialist', pointsRequired: 1500, icon: '⭐' },
  { level: 5, name: 'Expert', title: 'Expert', pointsRequired: 5000, icon: '🌟' },
  { level: 6, name: 'Master', title: 'Master', pointsRequired: 15000, icon: '💎' },
  { level: 7, name: 'Legend', title: 'Legend', pointsRequired: 50000, icon: '👑' },
  { level: 8, name: 'Visionary', title: 'Visionary', pointsRequired: 150000, icon: '🏆' }
];
```

---

## 8. Notification API Routes

### 8.1 Complete API Architecture

```
/api/notifications
├── GET /                          # List notifications
├── GET /unread-count              # Get unread count
├── GET /preferences               # Get notification preferences
├── PUT /preferences               # Update notification preferences
├── GET /channels                  # Get notification channels
├── POST /channels                 # Register notification channel
├── PUT /channels/[id]             # Update channel
├── DELETE /channels/[id]          # Delete channel
├── GET /templates                 # Get email/push templates
├── POST /templates                # Create template
├── PUT /templates/[id]            # Update template
├── DELETE /templates/[id]         # Delete template
├── POST /templates/[id]/test      # Test template
├── GET /digests                   # Get digest settings
├── PUT /digests                   # Update digest settings
├── POST /send                     # Send notification (manual)
├── POST /send-batch               # Send batch notifications
├── GET /[id]                     # Get notification details
├── PUT /[id]/read                # Mark as read
├── PUT /[id]/dismiss             # Dismiss notification
├── DELETE /[id]                   # Delete notification
├── POST /mark-all-read           # Mark all as read
├── POST /bulk-delete             # Bulk delete
└── GET /real-time/connect        # Real-time connection (SSE/WS)

/api/admin/notifications
├── GET /configure                 # Get notification configuration
├── PUT /configure                 # Update notification configuration
├── GET /categories                # Get notification categories
├── PUT /categories                # Update categories
├── GET /nudges                    # Get nudge definitions
├── POST /nudges                   # Create nudge
├── PUT /nudges/[id]              # Update nudge
├── DELETE /nudges/[id]           # Delete nudge
├── POST /nudges/[id]/test        # Test nudge
├── GET /gamification              # Get gamification config
├── PUT /gamification              # Update gamification config
├── GET /points-rules              # Get points rules
├── POST /points-rules             # Create points rule
├── PUT /points-rules/[id]        # Update points rule
├── DELETE /points-rules/[id]     # Delete points rule
├── GET /badges                    # Get badge definitions
├── POST /badges                   # Create badge
├── PUT /badges/[id]              # Update badge
├── DELETE /badges/[id]           # Delete badge
├── GET /leaderboards              # Get leaderboard config
├── PUT /leaderboards              # Update leaderboard config
├── GET /levels                    # Get level config
├── PUT /levels                    # Update level config
├── GET /analytics                 # Get notification analytics
├── GET /analytics/gamification   # Get gamification analytics
├── GET /analytics/engagement     # Get engagement analytics
└── POST /trigger-event           # Trigger notification event
```

### 8.2 API Route Implementations

```typescript
// app/api/notifications/route.ts - List notifications
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { z } from 'zod';
import { prisma } from '@/lib/db';
import { authOptions } from '@/lib/auth';

const listNotificationsSchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(25),
  type: z.string().optional(),
  category: z.string().optional(),
  priority: z.enum(['critical', 'high', 'normal', 'low']).optional(),
  status: z.enum(['pending', 'sent', 'delivered', 'read', 'archived']).optional(),
  read: z.boolean().optional(),
  startDate: z.coerce.date().optional(),
  endDate: z.coerce.date().optional(),
  sortBy: z.enum(['createdAt', 'priority', 'type']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc')
});

export async function GET(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    if (!session) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const searchParams = request.nextUrl.searchParams;
    const validatedParams = listNotificationsSchema.parse(Object.fromEntries(searchParams));
    
    const { page, limit, type, category, priority, status, read, startDate, endDate, sortBy, sortOrder } = validatedParams;

    // Build where clause
    const where: any = {
      userId: session.user.id,
      organizationId: session.user.organizationId
    };

    if (type) where.type = type;
    if (category) where.category = category;
    if (priority) where.priority = priority;
    if (status) where.status = status;
    if (read !== undefined) {
      where.readAt = read ? { not: null } : null;
    }
    
    if (startDate || endDate) {
      where.createdAt = {};
      if (startDate) where.createdAt.gte = startDate;
      if (endDate) where.createdAt.lte = endDate;
    }

    // Get total count
    const total = await prisma.notification.count({ where });

    // Get notifications
    const notifications = await prisma.notification.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { [sortBy]: sortOrder },
      include: {
        sender: {
          select: { id: true, name: true, email: true, avatar: true }
        }
      }
    });

    // Get unread count
    const unreadCount = await prisma.notification.count({
      where: {
        userId: session.user.id,
        readAt: null
      }
    });

    return NextResponse.json({
      data: notifications,
      unreadCount,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
        hasMore: page * limit < total
      }
    });
  } catch (error) {
    console.error('Error fetching notifications:', error);
    
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation error', details: error.errors },
        { status: 400 }
      );
    }
    
    return NextResponse.json(
      { error: 'Failed to fetch notifications' },
      { status: 500 }
    );
  }
}

// app/api/notifications/preferences/route.ts - Manage preferences
export async function GET_PREFERENCES(request: NextRequest) {
  // Return user notification preferences
}

export async function PUT_PREFERENCES(request: NextRequest) {
  // Update notification preferences
  // Handle channel preferences, quiet hours, frequency caps
}

// app/api/notifications/send/route.ts - Send notification
export async function POST_SEND(request: NextRequest) {
  const body = await request.json();
  const { userId, type, title, message, channels, priority, scheduledAt } = body;
  
  // Validate
  if (!userId || !type || !title) {
    return NextResponse.json(
      { error: 'Missing required fields' },
      { status: 400 }
    );
  }
  
  // Create notification
  const notification = await prisma.notification.create({
    data: {
      userId,
      organizationId: body.organizationId,
      type,
      title,
      message,
      priority: priority || 'normal',
      status: 'pending',
      channels: channels || ['inApp'],
      scheduledAt,
      context: body.context
    }
  });
  
  // Queue for delivery
  await queueNotificationDelivery(notification);
  
  return NextResponse.json({ id: notification.id }, { status: 201 });
}

// app/api/admin/notifications/configure/route.ts - Admin configuration
export async function GET_CONFIGURE(request: NextRequest) {
  // Return organization notification configuration
}

export async function PUT_CONFIGURE(request: NextRequest) {
  // Update notification configuration
  // Validate and save configuration
}

// app/api/admin/notifications/nudges/route.ts - Manage nudges
export async function GET_NUDGES(request: NextRequest) {
  // Return nudge definitions
}

export async function POST_NUDGES(request: NextRequest) {
  // Create new nudge definition
}

// app/api/admin/notifications/gamification/route.ts - Gamification config
export async function GET_GAMIFICATION(request: NextRequest) {
  // Return gamification configuration
}

export async function PUT_GAMIFICATION(request: NextRequest) {
  // Update gamification config
}
```

---

## 9. Notification Database Schema

### 9.1 Prisma Schema Models

```prisma
// Notification model
model Notification {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Content
  type           NotificationType
  category       String
  priority       NotificationPriority @default(NORMAL)
  title          String
  message        String   @db.Text
  
  // Action
  action         Json?
  // {
  //   type: 'link' | 'action' | 'modal',
  //   url?: string,
  //   actionId?: string,
  //   data?: object
  // }
  
  // Sender
  senderId       String?
  sender         User?    @relation("NotificationSender", fields: [senderId], references: [id])
  
  // Media
  media          Json?
  
  // Styling
  styling        Json?
  
  // User
  userId         String
  user           User     @relation("UserNotifications", fields: [userId], references: [id])
  
  // State
  status         NotificationStatus @default(PENDING)
  readAt         DateTime?
  dismissedAt    DateTime?
  archivedAt     DateTime?
  
  // Delivery tracking
  channels       Json?
  deliveredAt    DateTime?
  deliveryError  String?
  
  // Gamification
  gamification   Json?
  
  // Context
  context        Json?
  
  // Expiration
  expiresAt      DateTime?
  
  // Timestamps
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
  
  // Indexes
  @@index([organizationId, userId])
  @@index([userId, status])
  @@index([userId, createdAt])
  @@index([type, category])
  @@index([priority])
  @@index([expiresAt])
  
  @@map("notifications")
}

// User Notification Preferences
model NotificationPreference {
  id              String   @id @default(cuid())
  userId          String   @unique
  user            User     @relation(fields: [userId], references: [id])
  
  // Channel preferences
  channels        Json
  // {
  //   email: { enabled: boolean, frequency: string },
  //   sms: { enabled: boolean },
  //   push: { enabled: boolean },
  //   inApp: { enabled: boolean }
  // }
  
  // Type preferences
  typePreferences Json?
  
  // Quiet hours
  quietHours      Json?
  
  // Do not disturb
  dnd             Json?
  
  // Frequency caps
  frequencyCaps   Json?
  
  // Consent
  consent         Json?
  
  // Updated
  updatedAt       DateTime @updatedAt
  updatedBy       String?
  
  @@map("notification_preferences")
}

// Push Subscription
model PushSubscription {
  id              String   @id @default(cuid())
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  
  // Token
  token           String   @unique
  provider        String   // fcm, apns, web-push
  
  // Device
  device          Json?
  
  // Preferences
  preferences     Json
  active          Boolean  @default(true)
  
  // Usage
  lastUsed        DateTime @default(now())
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([userId])
  @@map("push_subscriptions")
}

// Email Template
model EmailTemplate {
  id              String   @id @default(cuid())
  organizationId String?
  organization   Organization? @relation(fields: [organizationId], references: [id])
  
  name            String
  description     String?
  
  // Content
  subject         String
  body            String   @db.Text
  html            String?  @db.Text
  text            String?  @db.Text
  
  // Variables
  variables       Json?
  
  // Design
  design          Json?
  
  // Categories
  categories      String[]
  
  // Active
  active          Boolean  @default(true)
  
  // Versioning
  version         Int      @default(1)
  versions        Json?
  
  // Usage
  usageCount      Int      @default(0)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  createdBy       String
  
  // Indexes
  @@index([organizationId])
  @@index([categories])
  
  @@map("email_templates")
}

// Notification Channel (Admin configured)
model NotificationChannel {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  name            String
  type            String // email, sms, push, slack, webhook
  
  // Config
  config          Json
  
  // Status
  enabled         Boolean  @default(true)
  testStatus      String?
  lastTested      DateTime?
  
  // Usage
  lastUsed        DateTime?
  usageCount      Int      @default(0)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([organizationId, type])
  
  @@map("notification_channels")
}

// Nudge Definition
model Nudge {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  name            String
  description     String?
  
  // Trigger
  trigger         Json
  
  // Content
  content         Json
  
  // Channels
  channels        Json
  
  // Timing
  timing          Json
  
  // Conditions
  conditions      Json?
  
  // Priority
  priority        String   @default('normal')
  
  // Gamification
  gamification    Json?
  
  // Targeting
  targeting       Json?
  
  // Stats
  triggerCount    Int      @default(0)
  nudgeCount      Int      @default(0)
  completeCount   Int      @default(0)
  
  // Active
  active          Boolean  @default(true)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  createdBy       String
  
  // Indexes
  @@index([organizationId, active])
  
  @@map("nudges")
}

// Nudge Instance (individual nudge sent)
model NudgeInstance {
  id              String   @id @default(cuid())
  nudgeId         String
  nudge           Nudge    @relation(fields: [nudgeId], references: [id])
  
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  
  // Context
  context         Json?
  
  // State
  status          String   @default('pending') // pending, sent, completed, ignored, expired
  sentAt          DateTime?
  completedAt     DateTime?
  ignoredAt       DateTime?
  
  // Nudge number in sequence
  sequenceNumber  Int
  
  // Escalation level
  escalationLevel Int      @default(0)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([nudgeId, userId])
  @@index([userId, status])
  
  @@map("nudge_instances")
}

// Gamification: User Points
model UserPoints {
  id              String   @id @default(cuid())
  userId          String   @unique
  user            User     @relation(fields: [userId], references: [id])
  organizationId String
  
  // Points
  totalPoints     Int      @default(0)
  currentPoints   Int      @default(0)
  lifetimePoints  Int      @default(0)
  
  // Level
  level           Int      @default(1)
  
  // Streaks
  currentStreak   Int      @default(0)
  longestStreak   Int      @default(0)
  streakFreezes   Int      @default(0)
  lastActivityAt  DateTime?
  
  // History
  pointsHistory   Json?
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([organizationId, totalPoints])
  @@index([organizationId, level])
  
  @@map("user_points")
}

// Gamification: Points Transaction
model PointsTransaction {
  id              String   @id @default(cuid())
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  organizationId String
  
  // Points
  points          Int
  type            String   // earned, redeemed, expired, adjustment
  
  // Rule
  ruleId          String?
  ruleName        String?
  
  // Description
  description     String?
  
  // Balance
  balanceAfter    Int
  
  // Timestamps
  createdAt       DateTime @default(now())
  
  @@index([userId, createdAt])
  @@index([organizationId, createdAt])
  
  @@map("points_transactions")
}

// Gamification: User Badge
model UserBadge {
  id              String   @id @default(cuid())
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  organizationId String
  
  badgeId         String
  badge           Badge    @relation(fields: [badgeId], references: [id])
  
  // Progress (if applicable)
  progress        Json?
  progressPercent Float    @default(0)
  
  // Status
  unlocked        Boolean  @default(false)
  unlockedAt      DateTime?
  
  // Display
  displayed       Boolean  @default(true)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@unique([userId, badgeId])
  @@index([organizationId, unlocked])
  
  @@map("user_badges")
}

// Badge Definition
model Badge {
  id              String   @id @default(cuid())
  organizationId String?
  organization   Organization? @relation(fields: [organizationId], references: [id])
  
  name            String
  description     String
  
  // Tier
  tier            String   // bronze, silver, gold, platinum, diamond
  
  // Icon
  icon            String
  image           String?
  
  // Rarity
  rarity          Int      @default(50)
  
  // Criteria
  criteria        Json
  
  // Rewards
  rewards         Json?
  
  // Display
  display         Json?
  
  // Limits
  limits          Json?
  
  // Unlocks
  unlockCount     Int      @default(0)
  
  // Active
  active          Boolean  @default(true)
  
  // Users
  users           UserBadge[]
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([organizationId])
  @@index([tier])
  
  @@map("badges")
}

// Gamification: Leaderboard Entry
model LeaderboardEntry {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  
  // Leaderboard
  leaderboardId   String
  
  // Period
  period          String   // daily, weekly, monthly, all
  
  // Date
  date            DateTime @db.Date
  
  // Points
  points          Int      @default(0)
  rank            Int
  
  // Change
  rankChange      Int      @default(0)
  
  // Timestamps
  updatedAt       DateTime @updatedAt
  
  @@unique([leaderboardId, userId, period, date])
  @@index([leaderboardId, period, date, rank])
  @@index([organizationId, period, date])
  
  @@map("leaderboard_entries")
}

// Gamification: Achievement
model Achievement {
  id              String   @id @default(cuid())
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  organizationId String
  
  // Achievement data
  type            String
  name            String
  description     String?
  
  // Progress
  progress        Json?
  completed       Boolean  @default(false)
  completedAt     DateTime?
  
  // Rewards
  rewards         Json?
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([userId, type])
  @@index([organizationId, completed])
  
  @@map("achievements")
}

// Notification Delivery Log
model NotificationDeliveryLog {
  id              String   @id @default(cuid())
  notificationId String
  notification    Notification @relation(fields: [notificationId], references: [id])
  
  channel         String
  
  // Status
  status          String   // pending, sent, delivered, failed
  sentAt          DateTime?
  deliveredAt     DateTime?
  readAt          DateTime?
  failedAt        DateTime?
  
  // Error
  error           String?
  errorCode       String?
  
  // Provider response
  providerResponse Json?
  
  // Timestamps
  createdAt       DateTime @default(now())
  
  @@index([notificationId, channel])
  @@index([status])
  
  @@map("notification_delivery_logs")
}

// Enum definitions
enum NotificationType {
  SYSTEM
  SECURITY
  MAINTENANCE
  BILLING
  USER_ACTION
  USER_MENTION
  USER_INVITE
  USER_FOLLOW
  TEAM_INVITE
  TEAM_UPDATE
  PROJECT_INVITE
  PROJECT_UPDATE
  COMMENT
  REVIEW
  APPROVAL
  ACTION_REQUIRED
  DEADLINE
  TASK_ASSIGNED
  ACHIEVEMENT_UNLOCKED
  LEVEL_UP
  POINTS_EARNED
  STREAK_MILESTONE
  BADGE_EARNED
  LEADERBOARD_RANK
  ERROR_ALERT
  WARNING_ALERT
  INFO_ALERT
  REMINDER
  FOLLOW_UP
  ENCOURAGEMENT
  PROMOTION
  NEWSLETTER
  FEATURE_ANNOUNCEMENT
  DAILY_DIGEST
  WEEKLY_DIGEST
  MONTHLY_DIGEST
}

enum NotificationPriority {
  CRITICAL
  HIGH
  NORMAL
  LOW
}

enum NotificationStatus {
  PENDING
  SENT
  DELIVERED
  READ
  ARCHIVED
  FAILED
  CANCELLED
}
```

---

## 10. Edge Cases

### 10.1 Notification Delivery Edge Cases

```typescript
const notificationDeliveryEdgeCases = [
  {
    scenario: 'User has multiple devices',
    description: 'Same user logged in on multiple devices',
    handling: 'Send to all active devices, deduplicate by notification ID'
  },
  {
    scenario: 'Channel temporarily unavailable',
    description: 'Email service or push service is down',
    handling: 'Queue locally, retry with exponential backoff, fallback to alternate channel'
  },
  {
    scenario: 'Rate limit exceeded',
    description: 'User or global rate limit reached',
    handling: 'Queue notification, send in next available window, notify sender'
  },
  {
    scenario: 'User preferences changed mid-delivery',
    description: 'User disables channel while notification queued',
    handling: 'Re-evaluate preferences at delivery time, skip if disabled'
  },
  {
    scenario: 'User deleted during send',
    description: 'User account deleted while notification in queue',
    handling: 'Cancel notification, clean up queue entries'
  },
  {
    scenario: 'Template not found',
    description: 'Template ID in notification does not exist',
    handling: 'Fallback to default template, log error for admin'
  },
  {
    scenario: 'Large notification payload',
    description: 'Notification data exceeds channel limits',
    handling: 'Truncate with link to full content, compress if possible'
  },
  {
    scenario: 'Timezone confusion',
    description: 'Scheduled notification in wrong timezone',
    handling: 'Store in UTC, convert on display, respect user timezone'
  },
  {
    scenario: 'Duplicate notification',
    description: 'Same notification triggered multiple times',
    handling: 'Deduplicate by grouping key, configurable window'
  }
];
```

### 10.2 Nudging Edge Cases

```typescript
const nudgingEdgeCases = [
  {
    scenario: 'User snoozes nudge',
    description: 'User snoozes the reminder',
    handling: 'Respect snooze duration, allow multiple snoozes up to limit'
  },
  {
    scenario: 'User completes action before nudge',
    description: 'Action completed before nudge scheduled',
    handling: 'Cancel pending nudge, credit points if applicable'
  },
  {
    scenario: 'Nudge triggers during quiet hours',
    description: 'Scheduled nudge falls in quiet hours',
    handling: 'Delay until quiet hours end, or skip based on config'
  },
  {
    scenario: 'Nudge chain too long',
    description: 'Max nudges reached without action',
    handling: 'Stop nudging, escalate if configured, log for analysis'
  },
  {
    scenario: 'Action deadline passes',
    description: 'Deadline passes while nudges pending',
    handling: 'Cancel pending nudges, send final notification, log as ignored'
  },
  {
    scenario: 'Nudge for deleted item',
    description: 'Nudge references deleted task/project',
    handling: 'Cancel nudge, mark as obsolete'
  },
  {
    scenario: 'Nudge fatigue',
    description: 'User receiving too many nudges',
    handling: 'Adaptive frequency, reduce priority, respect user feedback'
  }
];
```

### 10.3 Gamification Edge Cases

```typescript
const gamificationEdgeCases = [
  {
    scenario: 'Points overflow',
    description: 'User accumulates more than max integer',
    handling: 'Cap at max value, notify user, adjust thresholds'
  },
  {
    scenario: 'Badge criteria changed',
    description: 'Badge criteria modified after user earned it',
    handling: 'Lock earned badges, only apply new criteria to new earners'
  },
  {
    scenario: 'Streak freeze abuse',
    description: 'User exploits streak freeze',
    handling: 'Limit freezes, audit usage, disable if abuse detected'
  },
  {
    scenario: 'Leaderboard manipulation',
    description: 'User artificially inflates points',
    handling: 'Flag suspicious activity, manual review, potential reversal'
  },
  {
    scenario: 'Points earned for deleted action',
    description: 'Points credited for action that was later undone',
    handling: 'Reverse points, reverse badge if below threshold'
  },
  {
    scenario: 'Level down after point adjustment',
    description: 'Point reversal causes level decrease',
    handling: 'Maintain current level, slowly adjust, show warning'
  },
  {
    scenario: 'Multiple devices, syncing points',
    description: 'Points not syncing across devices',
    handling: 'Server-authoritative, eventual consistency, conflict resolution'
  }
];
```

---

## 11. Performance & Caching

### 11.1 Performance Optimization

```typescript
interface NotificationPerformanceConfig {
  // Async processing
  asyncProcessing: boolean;
  queueType: 'memory' | 'redis' | 'kafka';
  
  // Queue settings
  queue: {
    maxSize: number;
    batchSize: number;
    flushInterval: number;
    workers: number;
  };
  
  // Caching
  caching: {
    enabled: boolean;
    provider: 'memory' | 'redis';
    ttl: number;
  };
  
  // Database
  database: {
    connectionPool: {
      min: number;
      max: number;
    };
    readReplicas: boolean;
    partitioning: boolean;
  };
  
  // Real-time
  realTime: {
    provider: 'websocket' | 'sse';
    maxConnections: number;
    heartbeat: number;
  };
  
  // Rate limiting
  rateLimiting: {
    enabled: boolean;
    redisEnabled: boolean;
    windowSize: number;
    maxRequests: number;
  };
}

// Performance metrics
interface NotificationMetrics {
  delivery: {
    totalNotifications: number;
    deliveredCount: number;
    failedCount: number;
    avgDeliveryTime: number;
  };
  channels: {
    email: { sent: number; failed: number; openRate: number };
    push: { sent: number; failed: number; clickRate: number };
    inApp: { sent: number; readRate: number };
  };
  nudging: {
    triggered: number;
    completed: number;
    ignored: number;
  };
  gamification: {
    pointsIssued: number;
    badgesAwarded: number;
    streaksMaintained: number;
  };
}
```

### 11.2 Caching Strategy

```typescript
const notificationCachingStrategy = {
  // User preferences (change infrequently)
  'prefs:{userId}': {
    ttl: 3600,
    invalidation: 'on-preference-change'
  },
  
  // Notification center
  'notifications:{userId}:list': {
    ttl: 60,
    invalidation: 'on-new-notification'
  },
  
  // Unread count
  'notifications:{userId}:unread': {
    ttl: 30,
    invalidation: 'on-notification-read'
  },
  
  // Templates
  'templates:{orgId}:{templateId}': {
    ttl: 86400,
    invalidation: 'on-template-update'
  },
  
  // Gamification
  'gamification:{userId}:points': {
    ttl: 300,
    invalidation: 'on-points-change'
  },
  'gamification:{userId}:badges': {
    ttl: 3600,
    invalidation: 'on-badge-unlocked'
  },
  'gamification:{orgId}:leaderboard:{type}:{period}': {
    ttl: 300,
    invalidation: 'on-points-change'
  },
  
  // Nudges
  'nudges:{userId}:pending': {
    ttl: 60,
    invalidation: 'on-nudge-action'
  }
};
```

---

## 12. Integration Examples

### 12.1 In-App Notification Integration

```typescript
// Frontend: React hook for notifications
function useNotifications() {
  const [notifications, setNotifications] = useState<InAppNotification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);
  const [connected, setConnected] = useState(false);
  
  // Real-time connection
  useEffect(() => {
    const ws = connectRealTime(userId);
    
    ws.on('notification', (notification: InAppNotification) => {
      setNotifications(prev => [notification, ...prev]);
      setUnreadCount(prev => prev + 1);
      
      // Show toast
      showToast(notification);
    });
    
    ws.on('connect', () => setConnected(true));
    ws.on('disconnect', () => setConnected(false));
    
    return () => ws.disconnect();
  }, [userId]);
  
  // Mark as read
  const markAsRead = async (notificationId: string) => {
    await fetch(`/api/notifications/${notificationId}/read`, { method: 'PUT' });
    setNotifications(prev => 
      prev.map(n => n.id === notificationId ? { ...n, readAt: new Date() } : n)
    );
    setUnreadCount(prev => Math.max(0, prev - 1));
  };
  
  return { notifications, unreadCount, connected, markAsRead };
}

// Toast component
function showToast(notification: InAppNotification) {
  toast.show({
    type: notification.styling?.variant || 'info',
    title: notification.title,
    message: notification.message,
    duration: 5000,
    action: notification.action ? {
      label: notification.action.label,
      onClick: () => navigate(notification.action.url)
    } : undefined
  });
}
```

### 12.2 Email Integration

```typescript
// Send email notification
async function sendEmailNotification(recipient: string, templateId: string, variables: Record<string, any>) {
  // Get template
  const template = await getEmailTemplate(templateId);
  
  // Render template
  const rendered = renderTemplate(template, variables);
  
  // Send email
  const result = await emailProvider.send({
    to: recipient,
    subject: rendered.subject,
    html: rendered.html,
    text: rendered.text
  });
  
  // Log delivery
  await logNotificationDelivery({
    notificationId: variables.notificationId,
    channel: 'email',
    status: result.success ? 'sent' : 'failed',
    providerResponse: result
  });
  
  return result;
}

// Template rendering
function renderTemplate(template: EmailTemplate, variables: Record<string, any>) {
  let subject = template.subject;
  let body = template.body;
  let html = template.html;
  let text = template.text;
  
  // Replace variables
  Object.keys(variables).forEach(key => {
    const regex = new RegExp(`{{${key}}}`, 'g');
    const value = typeof variables[key] === 'object' 
      ? JSON.stringify(variables[key]) 
      : variables[key];
    subject = subject.replace(regex, value);
    body = body.replace(regex, value);
    html = html?.replace(regex, value);
    text = text?.replace(regex, value);
  });
  
  return { subject, body, html, text };
}
```

### 12.3 Gamification Integration

```typescript
// Award points for action
async function awardPoints(userId: string, action: string, context?: Record<string, any>) {
  // Get applicable rules
  const rules = await getPointRulesForAction(action);
  
  let totalPoints = 0;
  
  for (const rule of rules) {
    // Check conditions
    if (!evaluateConditions(rule.conditions, context)) continue;
    
    // Check limits
    const earned = await checkPointLimits(userId, rule.id);
    if (earned >= (rule.limits?.maxTimes || Infinity)) continue;
    
    // Calculate points
    let points = rule.points;
    if (rule.bonus?.enabled && evaluateCondition(rule.bonus.condition, context)) {
      points *= rule.bonus.multiplier;
    }
    
    // Award points
    await creditPoints(userId, points, rule.id, rule.name);
    totalPoints += points;
    
    // Check for badge unlock
    await checkBadgeUnlock(userId, action);
    
    // Check for level up
    await checkLevelUp(userId);
  }
  
  return totalPoints;
}

// Check badge unlock
async function checkBadgeUnlock(userId: string, action: string) {
  const user = await getUserWithBadges(userId);
  const unlockedBadgeIds = user.badges.filter(b => b.unlocked).map(b => b.badgeId);
  
  const badges = await getAvailableBadges();
  
  for (const badge of badges) {
    if (unlockedBadgeIds.includes(badge.id)) continue;
    
    const criteriaMet = await evaluateBadgeCriteria(userId, badge);
    
    if (criteriaMet) {
      await unlockBadge(userId, badge.id);
      
      // Send notification
      await sendNotification(userId, {
        type: 'BADGE_EARNED',
        title: `🏆 Badge Unlocked: ${badge.name}`,
        message: badge.description,
        gamification: { badgeIds: [badge.id] }
      });
    }
  }
}
```

### 12.4 Smart Nudge Example

```typescript
// Smart nudge engine
class SmartNudgeEngine {
  async shouldNudge(userId: string, nudge: NudgeDefinition, context: Record<string, any>): Promise<NudgeDecision> {
    // Check if nudging enabled for user
    const preferences = await getUserNudgePreferences(userId);
    if (!preferences.enabled) {
      return { shouldNudge: false, reason: 'disabled_by_user' };
    }
    
    // Check quiet hours
    if (this.isInQuietHours(preferences.quietHours)) {
      return { shouldNudge: false, reason: 'quiet_hours' };
    }
    
    // Check fatigue
    const recentNudges = await getRecentNudgeCount(userId, 24);
    if (recentNudges >= preferences.maxPerDay) {
      return { shouldNudge: false, reason: 'fatigue' };
    }
    
    // Smart timing
    if (nudge.smartTiming?.enabled) {
      const optimalTime = await this.findOptimalTime(userId);
      if (!this.isOptimalTime(optimalTime)) {
        return { shouldNudge: false, reason: 'not_optimal_time', scheduleFor: optimalTime };
      }
    }
    
    // Check conditions
    if (nudge.conditions?.length) {
      const conditionsMet = this.evaluateConditions(nudge.conditions, context);
      if (!conditionsMet) {
        return { shouldNudge: false, reason: 'conditions_not_met' };
      }
    }
    
    return { shouldNudge: true };
  }
  
  async findOptimalTime(userId: string): Promise<Date> {
    const history = await getUserActivityHistory(userId, 30);
    
    // Find best times
    const hourlyActivity = this.aggregateByHour(history);
    const bestHour = Object.entries(hourlyActivity)
      .sort(([,a], [,b]) => b - a)[0]?.[0];
    
    // Return next occurrence of best hour
    const now = new Date();
    const optimal = new Date(now);
    optimal.setHours(parseInt(bestHour), 0, 0, 0);
    if (optimal <= now) {
      optimal.setDate(optimal.getDate() + 1);
    }
    
    return optimal;
  }
}
```

---

## Summary

This comprehensive Notification System configuration flow covers every aspect of multi-channel notifications, nudging, and gamification:

### Key Components

1. **In-App Notifications** - Real-time delivery, notification center, toast notifications
2. **External Notifications** - Email, SMS, push notifications with templates
3. **Nudging System** - Configurable reminders, escalation, smart timing
4. **Gamification** - Points, badges, streaks, leaderboards, levels

### Admin Capabilities

- Configure notification channels and providers
- Create and manage notification templates
- Define nudge triggers, content, and escalation rules
- Configure gamification: points, badges, streaks, leaderboards
- Set up levels and progression rewards
- View analytics and engagement metrics

### User Capabilities

- Receive notifications across multiple channels
- Customize notification preferences per type
- Set quiet hours and DND schedules
- View gamification status (points, badges, streaks)
- Participate in leaderboards
- Share achievements

### Integration Points

- Frontend: React hooks, toast notifications, notification center
- Backend: Event triggers, template rendering, delivery queues
- External: Email (SendGrid, SES), SMS (Twilio), Push (FCM, APNS)
- Integrations: Slack, Teams, Discord, webhooks