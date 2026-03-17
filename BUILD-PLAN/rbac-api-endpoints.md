# RBAC System - Complete API Architecture

> **Foundation wiring note:** These APIs should be implemented as shared `GeneralApp` contracts and consumed through foundation SDK/services. Do not fork them per module.

## API Endpoints Overview

This document provides complete REST API endpoints for the RBAC (Role-Based Access Control) system including user management, communication, roles, capabilities, and audit features.

**Base URL**: `/api/v1`

**Authentication**: All endpoints require valid JWT token in `Authorization: Bearer <token>` header unless marked as public.

**Permission Requirements**: Each endpoint lists required capabilities.

---

## Table of Contents

1. [User Management APIs](#user-management-apis)
2. [User Inspection APIs](#user-inspection-apis)
3. [Communication APIs](#communication-apis)
4. [Template Management APIs](#template-management-apis)
5. [Role Management APIs](#role-management-apis)
6. [Capability Management APIs](#capability-management-apis)
7. [Permission Check APIs](#permission-check-apis)
8. [Audit & Analytics APIs](#audit--analytics-apis)
9. [Segment Management APIs](#segment-management-apis)
10. [Campaign Management APIs](#campaign-management-apis)

---

## User Management APIs

### 1. List All Users

**Endpoint**: `GET /api/v1/admin/users`

**Required Capability**: `users:read` (scope: organization)

**Query Parameters**:
```typescript
{
  page?: number           // Default: 1
  limit?: number          // Default: 20, Max: 100
  search?: string         // Search in name, email, username
  status?: 'active' | 'inactive' | 'suspended' | 'banned' | 'deleted'
  role?: string           // Filter by role slug
  authMethod?: 'email_password' | 'google' | 'github' | 'oauth'
  emailVerified?: boolean
  phoneVerified?: boolean
  mfaEnabled?: boolean
  sortBy?: 'created_at' | 'last_login_at' | 'email' | 'status'
  sortOrder?: 'asc' | 'desc'
  createdAfter?: string   // ISO date
  createdBefore?: string  // ISO date
  lastLoginAfter?: string
  lastLoginBefore?: string
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "uuid",
        "email": "user@example.com",
        "username": "johndoe",
        "firstName": "John",
        "lastName": "Doe",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "status": "active",
        "accountState": "active",
        "emailVerified": true,
        "phoneVerified": false,
        "mfaEnabled": true,
        "primaryAuthMethod": "google",
        "hasPasswordSet": false,
        "roles": [
          {
            "id": "uuid",
            "name": "Admin",
            "slug": "admin",
            "assignedAt": "2024-01-15T10:30:00Z"
          }
        ],
        "lastLoginAt": "2024-02-15T09:30:00Z",
        "lastLoginIp": "192.168.1.100",
        "loginCount": 342,
        "createdAt": "2023-06-10T14:20:00Z",
        "updatedAt": "2024-02-15T09:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 1234,
      "totalPages": 62,
      "hasNext": true,
      "hasPrev": false
    }
  },
  "meta": {
    "timestamp": "2024-02-15T10:00:00Z",
    "requestId": "req_abc123"
  }
}
```

---

### 2. Get User by ID

**Endpoint**: `GET /api/v1/admin/users/:userId`

**Required Capability**: `users:read` (scope: own for self, organization for others)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "username": "johndoe",
    "firstName": "John",
    "lastName": "Doe",
    "displayName": "John Doe",
    "avatarUrl": "https://...",
    "phone": "+1234567890",
    "phoneCountryCode": "+1",
    "status": "active",
    "accountState": "active",
    "emailVerified": true,
    "phoneVerified": true,
    "mfaEnabled": true,
    "primaryAuthMethod": "google",
    "hasPasswordSet": true,
    "riskScore": 0.15,
    "engagementScore": 87.5,
    "lastLoginAt": "2024-02-15T09:30:00Z",
    "lastLoginIp": "192.168.1.100",
    "lastLoginDevice": "Chrome 120 on macOS",
    "lastLoginLocation": {
      "country": "United States",
      "city": "San Francisco",
      "coordinates": { "lat": 37.7749, "lng": -122.4194 }
    },
    "loginCount": 342,
    "metadata": {},
    "createdAt": "2023-06-10T14:20:00Z",
    "updatedAt": "2024-02-15T09:30:00Z"
  }
}
```

---

### 3. Update User

**Endpoint**: `PATCH /api/v1/admin/users/:userId`

**Required Capability**: `users:update` (scope: own for self, organization for others)

**Request Body**:
```json
{
  "firstName": "Jane",
  "lastName": "Smith",
  "displayName": "Jane Smith",
  "phone": "+1234567890",
  "status": "active",
  "metadata": {
    "department": "Engineering"
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "firstName": "Jane",
    "lastName": "Smith",
    // ... full user object
  },
  "meta": {
    "updated": ["firstName", "lastName", "displayName"]
  }
}
```

---

### 4. Delete User

**Endpoint**: `DELETE /api/v1/admin/users/:userId`

**Required Capability**: `users:delete` (scope: organization)

**Query Parameters**:
```typescript
{
  hard?: boolean  // true = permanent delete, false = soft delete (default)
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "deleted": true,
    "deletedAt": "2024-02-15T10:00:00Z",
    "deletionType": "soft"
  }
}
```

---

### 5. Suspend User

**Endpoint**: `POST /api/v1/admin/users/:userId/suspend`

**Required Capability**: `users:suspend` (scope: organization)

**Request Body**:
```json
{
  "reason": "Violation of terms of service",
  "duration": 30,           // days, null = indefinite
  "notifyUser": true
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "suspended",
    "suspendedAt": "2024-02-15T10:00:00Z",
    "suspendedUntil": "2024-03-15T10:00:00Z",
    "reason": "Violation of terms of service"
  }
}
```

---

### 6. Unsuspend User

**Endpoint**: `POST /api/v1/admin/users/:userId/unsuspend`

**Required Capability**: `users:suspend` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "active",
    "unsuspendedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

## User Inspection APIs

### 7. Get User Full Profile (360° View)

**Endpoint**: `GET /api/v1/admin/users/:userId/profile`

**Required Capability**: `users:inspect` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "overview": {
      "basicInfo": {
        "id": "uuid",
        "email": "john@example.com",
        "username": "johndoe",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "memberSince": "2023-06-10T14:20:00Z",
        "daysSinceJoined": 250,
        "accountAge": "8 months",
        "status": "active",
        "accountState": "active",
        "timezone": "America/Los_Angeles",
        "language": "en-US"
      },
      "activitySummary": {
        "lastLogin": "2024-02-15T09:30:00Z",
        "lastLoginAgo": "30 minutes ago",
        "loginCount": 342,
        "avgLoginsPerWeek": 12,
        "lastActivityAt": "2024-02-15T09:45:00Z",
        "activeSessionCount": 2,
        "engagementScore": 87.5,
        "riskScore": 0.15
      }
    },

    "authentication": {
      "authMethods": {
        "primary": "google",
        "available": ["email_password", "google", "github"],
        "password": {
          "hasPassword": true,
          "passwordSetAt": "2023-06-10T14:25:00Z",
          "passwordChangedAt": "2024-01-05T11:00:00Z",
          "daysSinceChange": 41,
          "strength": "strong",
          "requiresChange": false
        },
        "oauth": {
          "google": {
            "connected": true,
            "connectedAt": "2023-06-10T14:20:00Z",
            "email": "john@gmail.com",
            "profileId": "google_12345",
            "isPrimary": true,
            "lastUsed": "2024-02-15T09:30:00Z"
          },
          "github": {
            "connected": true,
            "connectedAt": "2023-07-20T10:00:00Z",
            "username": "johndoe",
            "profileId": "github_67890",
            "isPrimary": false,
            "lastUsed": "2024-02-01T15:00:00Z"
          },
          "apple": {
            "connected": false
          },
          "microsoft": {
            "connected": false
          },
          "facebook": {
            "connected": false
          },
          "linkedin": {
            "connected": false
          },
          "twitter": {
            "connected": false
          }
        }
      },
      "emailVerification": {
        "verified": true,
        "verifiedAt": "2023-06-10T14:22:00Z",
        "verificationMethod": "email_link"
      },
      "phoneVerification": {
        "verified": true,
        "phone": "+1234567890",
        "verifiedAt": "2023-06-11T09:00:00Z",
        "verificationMethod": "sms_code"
      },
      "mfa": {
        "enabled": true,
        "enabledAt": "2023-06-15T10:00:00Z",
        "methods": [
          {
            "type": "totp",
            "name": "Authenticator App",
            "isPrimary": true,
            "addedAt": "2023-06-15T10:00:00Z"
          },
          {
            "type": "sms",
            "name": "SMS to +1234567890",
            "isPrimary": false,
            "addedAt": "2023-06-15T10:05:00Z"
          }
        ],
        "backupCodesRemaining": 8
      }
    },

    "sessions": {
      "activeSessions": [
        {
          "id": "session_abc123",
          "device": {
            "type": "desktop",
            "os": "macOS 14.2",
            "browser": "Chrome 120",
            "deviceName": "John's MacBook Pro",
            "fingerprint": "fp_xyz789"
          },
          "location": {
            "ip": "192.168.1.100",
            "country": "United States",
            "city": "San Francisco",
            "region": "California",
            "coordinates": {
              "lat": 37.7749,
              "lng": -122.4194
            }
          },
          "startedAt": "2024-02-15T09:30:00Z",
          "lastActivityAt": "2024-02-15T09:45:00Z",
          "expiresAt": "2024-02-22T09:30:00Z",
          "isCurrent": true
        }
      ],
      "totalSessions": 2,
      "sessionHistory": {
        "total": 342,
        "last30Days": 28,
        "avgDuration": "2h 15m"
      }
    },

    "loginHistory": {
      "recentLogins": [
        {
          "id": "login_123",
          "timestamp": "2024-02-15T09:30:00Z",
          "success": true,
          "method": "google",
          "device": {
            "type": "desktop",
            "os": "macOS 14.2",
            "browser": "Chrome 120"
          },
          "location": {
            "ip": "192.168.1.100",
            "country": "United States",
            "city": "San Francisco"
          },
          "newDevice": false,
          "newLocation": false,
          "suspicious": false,
          "mfaUsed": true
        },
        {
          "id": "login_122",
          "timestamp": "2024-02-14T18:20:00Z",
          "success": true,
          "method": "password",
          "device": {
            "type": "mobile",
            "os": "iOS 17",
            "browser": "Safari"
          },
          "location": {
            "ip": "192.168.1.50",
            "country": "United States",
            "city": "San Francisco"
          },
          "newDevice": false,
          "newLocation": false,
          "suspicious": false,
          "mfaUsed": true
        }
      ],
      "failedLogins": [
        {
          "timestamp": "2024-02-10T14:30:00Z",
          "method": "password",
          "reason": "invalid_credentials",
          "location": {
            "ip": "203.0.113.45",
            "country": "Unknown"
          },
          "suspicious": true
        }
      ],
      "stats": {
        "totalLogins": 342,
        "successRate": 99.1,
        "last30Days": {
          "total": 28,
          "failed": 0,
          "newDevices": 1,
          "newLocations": 0
        }
      }
    },

    "rolesAndPermissions": {
      "roles": [
        {
          "id": "role_admin",
          "name": "Admin",
          "slug": "admin",
          "level": 10,
          "assignedAt": "2023-06-10T14:30:00Z",
          "assignedBy": {
            "id": "user_founder",
            "name": "Founder"
          },
          "expiresAt": null,
          "source": "manual"
        }
      ],
      "capabilities": [
        {
          "id": "cap_users_manage",
          "name": "Manage Users",
          "slug": "users:manage",
          "resource": "users",
          "actions": ["create", "read", "update", "delete"],
          "scope": "organization",
          "source": "role:admin",
          "grantedAt": "2023-06-10T14:30:00Z"
        }
      ],
      "effectivePermissions": {
        "users": {
          "create": true,
          "read": true,
          "update": true,
          "delete": true,
          "scope": "organization"
        },
        "roles": {
          "create": true,
          "read": true,
          "update": true,
          "delete": true,
          "scope": "global"
        }
      }
    },

    "activity": {
      "recentActions": [
        {
          "timestamp": "2024-02-15T09:45:00Z",
          "action": "user.updated",
          "resource": "users",
          "resourceId": "user_abc",
          "details": {
            "changes": ["email", "firstName"]
          }
        }
      ],
      "auditLog": {
        "totalEvents": 1234,
        "last30Days": 45,
        "criticalEvents": 2
      }
    },

    "communication": {
      "emailStats": {
        "sent": 45,
        "delivered": 44,
        "opened": 38,
        "clicked": 22,
        "bounced": 1,
        "openRate": 86.4,
        "clickRate": 50.0
      },
      "smsStats": {
        "sent": 12,
        "delivered": 12,
        "failed": 0
      },
      "pushStats": {
        "sent": 89,
        "delivered": 87,
        "clicked": 34
      },
      "preferences": {
        "emailMarketing": true,
        "emailTransactional": true,
        "smsMarketing": false,
        "smsTransactional": true,
        "pushNotifications": true,
        "inAppNotifications": true
      }
    },

    "devices": {
      "knownDevices": [
        {
          "id": "device_mac",
          "name": "John's MacBook Pro",
          "type": "desktop",
          "os": "macOS 14.2",
          "browser": "Chrome 120",
          "fingerprint": "fp_xyz789",
          "firstSeen": "2023-06-10T14:20:00Z",
          "lastSeen": "2024-02-15T09:30:00Z",
          "loginCount": 280,
          "trusted": true,
          "active": true
        },
        {
          "id": "device_iphone",
          "name": "John's iPhone",
          "type": "mobile",
          "os": "iOS 17.2",
          "browser": "Safari",
          "fingerprint": "fp_abc456",
          "firstSeen": "2023-06-11T08:00:00Z",
          "lastSeen": "2024-02-14T18:20:00Z",
          "loginCount": 62,
          "trusted": true,
          "active": false
        }
      ],
      "totalDevices": 2,
      "trustedDevices": 2
    },

    "locations": {
      "knownLocations": [
        {
          "country": "United States",
          "city": "San Francisco",
          "region": "California",
          "coordinates": {
            "lat": 37.7749,
            "lng": -122.4194
          },
          "firstSeen": "2023-06-10T14:20:00Z",
          "lastSeen": "2024-02-15T09:30:00Z",
          "loginCount": 320,
          "isPrimary": true
        },
        {
          "country": "United States",
          "city": "New York",
          "region": "New York",
          "coordinates": {
            "lat": 40.7128,
            "lng": -74.0060
          },
          "firstSeen": "2024-01-15T10:00:00Z",
          "lastSeen": "2024-01-20T16:00:00Z",
          "loginCount": 15,
          "isPrimary": false
        }
      ],
      "totalLocations": 2
    },

    "security": {
      "riskAssessment": {
        "score": 0.15,
        "level": "low",
        "factors": [
          {
            "factor": "mfa_enabled",
            "impact": -0.3,
            "description": "MFA is enabled"
          },
          {
            "factor": "email_verified",
            "impact": -0.2,
            "description": "Email verified"
          },
          {
            "factor": "consistent_locations",
            "impact": -0.1,
            "description": "Logins from consistent locations"
          }
        ],
        "lastUpdated": "2024-02-15T09:30:00Z"
      },
      "securityEvents": {
        "recentAlerts": [],
        "totalAlerts": 0,
        "resolvedAlerts": 0
      }
    },

    "metadata": {
      "customFields": {
        "department": "Engineering",
        "employeeId": "EMP-12345",
        "startDate": "2023-06-01"
      },
      "tags": ["premium", "early_adopter", "beta_tester"]
    }
  }
}
```

---

### 8. Get User Authentication Methods

**Endpoint**: `GET /api/v1/admin/users/:userId/auth-methods`

**Required Capability**: `users:inspect` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "primary": "google",
    "password": {
      "hasPassword": true,
      "setAt": "2023-06-10T14:25:00Z",
      "lastChanged": "2024-01-05T11:00:00Z"
    },
    "oauth": {
      "google": {
        "connected": true,
        "email": "john@gmail.com",
        "linkedAt": "2023-06-10T14:20:00Z"
      },
      "github": {
        "connected": true,
        "username": "johndoe",
        "linkedAt": "2023-07-20T10:00:00Z"
      }
    },
    "mfa": {
      "enabled": true,
      "methods": ["totp", "sms"]
    }
  }
}
```

---

### 9. Get User Sessions

**Endpoint**: `GET /api/v1/admin/users/:userId/sessions`

**Required Capability**: `users:inspect` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "activeSessions": [
      {
        "id": "session_abc",
        "device": {
          "type": "desktop",
          "os": "macOS",
          "browser": "Chrome"
        },
        "location": {
          "ip": "192.168.1.100",
          "country": "United States",
          "city": "San Francisco"
        },
        "startedAt": "2024-02-15T09:30:00Z",
        "expiresAt": "2024-02-22T09:30:00Z"
      }
    ],
    "total": 2
  }
}
```

---

### 10. Revoke User Session

**Endpoint**: `DELETE /api/v1/admin/users/:userId/sessions/:sessionId`

**Required Capability**: `users:manage_sessions` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "sessionId": "session_abc",
    "revokedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 11. Get User Login History

**Endpoint**: `GET /api/v1/admin/users/:userId/login-history`

**Required Capability**: `users:inspect` (scope: organization)

**Query Parameters**:
```typescript
{
  limit?: number      // Default: 50, Max: 200
  offset?: number
  startDate?: string  // ISO date
  endDate?: string
  success?: boolean   // Filter by success/failed
  suspicious?: boolean
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "logins": [
      {
        "timestamp": "2024-02-15T09:30:00Z",
        "success": true,
        "method": "google",
        "device": {
          "type": "desktop",
          "os": "macOS",
          "browser": "Chrome"
        },
        "location": {
          "ip": "192.168.1.100",
          "country": "United States",
          "city": "San Francisco"
        },
        "newDevice": false,
        "newLocation": false,
        "suspicious": false
      }
    ],
    "total": 342,
    "stats": {
      "successRate": 99.1,
      "failedAttempts": 3,
      "suspiciousAttempts": 1
    }
  }
}
```

---

## Communication APIs

### 12. Send Message to User(s)

**Endpoint**: `POST /api/v1/admin/messages/send`

**Required Capability**: `messages:send` (scope: organization)

**Request Body**:
```json
{
  "recipients": {
    "mode": "individual",
    "userIds": ["user_abc", "user_def"]
  },
  "channels": {
    "email": {
      "enabled": true,
      "subject": "Welcome to our platform",
      "body": "<p>Hi {{user.firstName}},</p><p>Welcome!</p>",
      "from": {
        "name": "BlueprintForge Team",
        "email": "team@blueprintforge.com"
      },
      "templateId": null
    },
    "sms": {
      "enabled": false
    },
    "push": {
      "enabled": true,
      "title": "Welcome!",
      "body": "Thanks for joining us"
    },
    "inApp": {
      "enabled": true,
      "title": "Welcome",
      "message": "Check out our getting started guide"
    }
  },
  "personalization": {
    "enabled": true,
    "variables": {
      "user.firstName": true,
      "user.email": true
    }
  },
  "scheduling": {
    "sendAt": null,  // null = send immediately
    "timezone": "America/Los_Angeles"
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "messageId": "msg_abc123",
    "recipients": {
      "total": 2,
      "email": 2,
      "sms": 0,
      "push": 2,
      "inApp": 2
    },
    "status": "sent",
    "sentAt": "2024-02-15T10:00:00Z",
    "estimatedDelivery": "2024-02-15T10:01:00Z"
  }
}
```

---

### 13. Send Bulk Message

**Endpoint**: `POST /api/v1/admin/messages/bulk`

**Required Capability**: `messages:bulk_send` (scope: organization)

**Request Body**:
```json
{
  "recipients": {
    "mode": "filtered",
    "filters": {
      "status": "active",
      "emailVerified": true,
      "roles": ["user", "premium"],
      "createdAfter": "2024-01-01T00:00:00Z"
    }
  },
  "channels": {
    "email": {
      "enabled": true,
      "templateId": "template_welcome",
      "subject": "Important Update",
      "variables": {
        "updateTitle": "New Features Released"
      }
    }
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "messageId": "msg_bulk_abc",
    "recipients": {
      "total": 1234,
      "matched": 1234,
      "queued": 1234
    },
    "estimatedDeliveryTime": "15 minutes",
    "status": "queued"
  }
}
```

---

### 14. Get Message Status

**Endpoint**: `GET /api/v1/admin/messages/:messageId`

**Required Capability**: `messages:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "msg_abc123",
    "type": "one_time",
    "channel": "email",
    "subject": "Welcome to our platform",
    "recipientCount": 2,
    "status": "delivered",
    "sentAt": "2024-02-15T10:00:00Z",
    "deliveredCount": 2,
    "openedCount": 1,
    "clickedCount": 0,
    "bouncedCount": 0,
    "failedCount": 0,
    "stats": {
      "deliveryRate": 100,
      "openRate": 50,
      "clickRate": 0
    }
  }
}
```

---

### 15. Get Message Analytics

**Endpoint**: `GET /api/v1/admin/messages/:messageId/analytics`

**Required Capability**: `messages:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "messageId": "msg_abc123",
    "overview": {
      "sent": 1000,
      "delivered": 980,
      "opened": 490,
      "clicked": 147,
      "bounced": 15,
      "failed": 5
    },
    "rates": {
      "deliveryRate": 98.0,
      "openRate": 50.0,
      "clickRate": 30.0,
      "bounceRate": 1.5
    },
    "timeline": [
      {
        "timestamp": "2024-02-15T10:00:00Z",
        "event": "sent",
        "count": 1000
      },
      {
        "timestamp": "2024-02-15T10:05:00Z",
        "event": "delivered",
        "count": 980
      }
    ]
  }
}
```

---

## Template Management APIs

### 16. List Templates

**Endpoint**: `GET /api/v1/admin/templates`

**Required Capability**: `templates:read` (scope: organization)

**Query Parameters**:
```typescript
{
  type?: 'email' | 'sms' | 'push' | 'in_app'
  category?: string
  search?: string
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "id": "template_welcome",
        "name": "Welcome Email",
        "type": "email",
        "category": "onboarding",
        "subject": "Welcome to {{app.name}}!",
        "bodyPreview": "Hi {{user.firstName}}, welcome to our platform...",
        "variables": ["user.firstName", "user.email", "app.name"],
        "createdAt": "2023-06-01T10:00:00Z",
        "updatedAt": "2024-02-01T15:00:00Z",
        "createdBy": {
          "id": "user_admin",
          "name": "Admin User"
        },
        "usageCount": 1234,
        "lastUsed": "2024-02-15T09:00:00Z"
      }
    ],
    "total": 45
  }
}
```

---

### 17. Get Template by ID

**Endpoint**: `GET /api/v1/admin/templates/:templateId`

**Required Capability**: `templates:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "template_welcome",
    "name": "Welcome Email",
    "type": "email",
    "category": "onboarding",
    "subject": "Welcome to {{app.name}}!",
    "body": "<html>...</html>",
    "bodyText": "Plain text version...",
    "variables": [
      {
        "key": "user.firstName",
        "description": "User's first name",
        "required": true,
        "defaultValue": "there"
      },
      {
        "key": "app.name",
        "description": "Application name",
        "required": true
      }
    ],
    "metadata": {
      "tags": ["welcome", "onboarding"],
      "notes": "Primary welcome email for new users"
    },
    "createdAt": "2023-06-01T10:00:00Z",
    "updatedAt": "2024-02-01T15:00:00Z"
  }
}
```

---

### 18. Create Template

**Endpoint**: `POST /api/v1/admin/templates`

**Required Capability**: `templates:create` (scope: organization)

**Request Body**:
```json
{
  "name": "Password Reset",
  "type": "email",
  "category": "security",
  "subject": "Reset your password",
  "body": "<html><p>Hi {{user.firstName}},</p><p>Click here to reset: {{resetLink}}</p></html>",
  "bodyText": "Hi {{user.firstName}}, Click here to reset: {{resetLink}}",
  "variables": [
    {
      "key": "user.firstName",
      "description": "User's first name",
      "required": true
    },
    {
      "key": "resetLink",
      "description": "Password reset link",
      "required": true
    }
  ],
  "metadata": {
    "tags": ["security", "password"]
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "template_new123",
    "name": "Password Reset",
    // ... full template object
    "createdAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 19. Update Template

**Endpoint**: `PATCH /api/v1/admin/templates/:templateId`

**Required Capability**: `templates:update` (scope: organization)

**Request Body**:
```json
{
  "subject": "Reset your password - {{app.name}}",
  "body": "<html>Updated content...</html>"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "template_abc",
    // ... full updated template
    "updatedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 20. Delete Template

**Endpoint**: `DELETE /api/v1/admin/templates/:templateId`

**Required Capability**: `templates:delete` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "template_abc",
    "deleted": true,
    "deletedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 21. Preview Template

**Endpoint**: `POST /api/v1/admin/templates/:templateId/preview`

**Required Capability**: `templates:read` (scope: organization)

**Request Body**:
```json
{
  "variables": {
    "user.firstName": "John",
    "user.email": "john@example.com",
    "resetLink": "https://app.example.com/reset/abc123"
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "subject": "Reset your password",
    "bodyHtml": "<html><p>Hi John,</p><p>Click here to reset: https://app.example.com/reset/abc123</p></html>",
    "bodyText": "Hi John, Click here to reset: https://app.example.com/reset/abc123"
  }
}
```

---

## Role Management APIs

### 22. List All Roles

**Endpoint**: `GET /api/v1/admin/roles`

**Required Capability**: `roles:read` (scope: organization)

**Query Parameters**:
```typescript
{
  type?: 'system' | 'custom'
  includeInherited?: boolean
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "roles": [
      {
        "id": "role_admin",
        "name": "Admin",
        "slug": "admin",
        "description": "Full system access",
        "level": 10,
        "type": "system",
        "isSystem": true,
        "isDefault": false,
        "userCount": 5,
        "capabilityCount": 45,
        "color": "#FF5722",
        "icon": "shield",
        "badge": "ADMIN",
        "inheritsFrom": [],
        "createdAt": "2023-01-01T00:00:00Z",
        "updatedAt": "2023-01-01T00:00:00Z"
      },
      {
        "id": "role_user",
        "name": "User",
        "slug": "user",
        "description": "Standard user access",
        "level": 100,
        "type": "system",
        "isSystem": true,
        "isDefault": true,
        "userCount": 1234,
        "capabilityCount": 12,
        "color": "#2196F3",
        "icon": "user",
        "badge": null,
        "inheritsFrom": [],
        "createdAt": "2023-01-01T00:00:00Z",
        "updatedAt": "2023-01-01T00:00:00Z"
      }
    ],
    "total": 8
  }
}
```

---

### 23. Get Role by ID

**Endpoint**: `GET /api/v1/admin/roles/:roleId`

**Required Capability**: `roles:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "role_admin",
    "name": "Admin",
    "slug": "admin",
    "description": "Full system access",
    "level": 10,
    "type": "system",
    "isSystem": true,
    "isDefault": false,
    "userLimit": null,
    "color": "#FF5722",
    "icon": "shield",
    "badge": "ADMIN",
    "inheritsFrom": [],
    "capabilities": [
      {
        "id": "cap_users_manage",
        "name": "Manage Users",
        "slug": "users:manage",
        "resource": "users",
        "actions": ["create", "read", "update", "delete"],
        "scope": "organization",
        "grantedAt": "2023-01-01T00:00:00Z"
      }
    ],
    "users": [
      {
        "id": "user_abc",
        "email": "admin@example.com",
        "name": "Admin User",
        "assignedAt": "2023-06-10T14:30:00Z"
      }
    ],
    "metadata": {},
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-01T00:00:00Z"
  }
}
```

---

### 24. Create Role

**Endpoint**: `POST /api/v1/admin/roles`

**Required Capability**: `roles:create` (scope: organization)

**Request Body**:
```json
{
  "name": "Content Manager",
  "slug": "content_manager",
  "description": "Can manage all content",
  "level": 50,
  "color": "#4CAF50",
  "icon": "file-text",
  "inheritsFrom": ["role_user"],
  "capabilityIds": [
    "cap_content_create",
    "cap_content_update",
    "cap_content_delete"
  ],
  "conditions": {
    "requiresApproval": false
  },
  "autoAssignmentRules": []
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "role_new123",
    "name": "Content Manager",
    "slug": "content_manager",
    // ... full role object
    "createdAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 25. Update Role

**Endpoint**: `PATCH /api/v1/admin/roles/:roleId`

**Required Capability**: `roles:update` (scope: organization)

**Request Body**:
```json
{
  "description": "Updated description",
  "color": "#673AB7"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "role_abc",
    // ... full updated role
    "updatedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 26. Delete Role

**Endpoint**: `DELETE /api/v1/admin/roles/:roleId`

**Required Capability**: `roles:delete` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "role_abc",
    "deleted": true,
    "deletedAt": "2024-02-15T10:00:00Z",
    "affectedUsers": 12
  }
}
```

---

### 27. Assign Role to User

**Endpoint**: `POST /api/v1/admin/users/:userId/roles`

**Required Capability**: `roles:assign` (scope: organization)

**Request Body**:
```json
{
  "roleId": "role_admin",
  "expiresAt": "2025-02-15T00:00:00Z",  // null for permanent
  "metadata": {
    "reason": "Promoted to team lead"
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "userId": "user_abc",
    "roleId": "role_admin",
    "assignedAt": "2024-02-15T10:00:00Z",
    "assignedBy": "user_current",
    "expiresAt": "2025-02-15T00:00:00Z"
  }
}
```

---

### 28. Remove Role from User

**Endpoint**: `DELETE /api/v1/admin/users/:userId/roles/:roleId`

**Required Capability**: `roles:assign` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "userId": "user_abc",
    "roleId": "role_admin",
    "removedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

## Capability Management APIs

### 29. List All Capabilities

**Endpoint**: `GET /api/v1/admin/capabilities`

**Required Capability**: `capabilities:read` (scope: organization)

**Query Parameters**:
```typescript
{
  resource?: string
  scope?: 'own' | 'team' | 'organization' | 'global'
  type?: 'system' | 'custom'
  dangerous?: boolean
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "capabilities": [
      {
        "id": "cap_users_manage",
        "name": "Manage Users",
        "slug": "users:manage",
        "description": "Full user management access",
        "resource": "users",
        "actions": ["create", "read", "update", "delete"],
        "scope": "organization",
        "type": "system",
        "category": "user_management",
        "tags": ["admin", "users"],
        "dangerous": true,
        "roleCount": 2,
        "userCount": 5,
        "createdAt": "2023-01-01T00:00:00Z"
      }
    ],
    "total": 67
  }
}
```

---

### 30. Get Capability by ID

**Endpoint**: `GET /api/v1/admin/capabilities/:capabilityId`

**Required Capability**: `capabilities:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "cap_users_manage",
    "name": "Manage Users",
    "slug": "users:manage",
    "description": "Full user management access",
    "resource": "users",
    "actions": ["create", "read", "update", "delete"],
    "scope": "organization",
    "type": "system",
    "category": "user_management",
    "tags": ["admin", "users"],
    "dangerous": true,
    "conditions": {
      "requireMfa": true
    },
    "requires": [],
    "conflicts": [],
    "implies": ["cap_users_read"],
    "rateLimiting": {
      "enabled": true,
      "maxRequests": 100,
      "windowMinutes": 60
    },
    "approvalRequired": false,
    "auditEnabled": true,
    "roles": [
      {
        "id": "role_admin",
        "name": "Admin",
        "slug": "admin"
      }
    ],
    "metadata": {},
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-01T00:00:00Z"
  }
}
```

---

### 31. Create Capability

**Endpoint**: `POST /api/v1/admin/capabilities`

**Required Capability**: `capabilities:create` (scope: organization)

**Request Body**:
```json
{
  "name": "Export Reports",
  "slug": "reports:export",
  "description": "Can export reports in various formats",
  "resource": "reports",
  "actions": ["export"],
  "scope": "organization",
  "category": "reporting",
  "tags": ["reports", "export"],
  "dangerous": false,
  "conditions": {},
  "rateLimiting": {
    "enabled": true,
    "maxRequests": 10,
    "windowMinutes": 60
  },
  "approvalRequired": false,
  "auditEnabled": true
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "cap_new123",
    "name": "Export Reports",
    "slug": "reports:export",
    // ... full capability object
    "createdAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 32. Update Capability

**Endpoint**: `PATCH /api/v1/admin/capabilities/:capabilityId`

**Required Capability**: `capabilities:update` (scope: organization)

**Request Body**:
```json
{
  "description": "Updated description",
  "dangerous": true
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "cap_abc",
    // ... full updated capability
    "updatedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 33. Delete Capability

**Endpoint**: `DELETE /api/v1/admin/capabilities/:capabilityId`

**Required Capability**: `capabilities:delete` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "cap_abc",
    "deleted": true,
    "deletedAt": "2024-02-15T10:00:00Z",
    "affectedRoles": 3,
    "affectedUsers": 15
  }
}
```

---

### 34. Add Capability to Role

**Endpoint**: `POST /api/v1/admin/roles/:roleId/capabilities`

**Required Capability**: `roles:manage_capabilities` (scope: organization)

**Request Body**:
```json
{
  "capabilityId": "cap_reports_export"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "roleId": "role_manager",
    "capabilityId": "cap_reports_export",
    "grantedAt": "2024-02-15T10:00:00Z",
    "grantedBy": "user_admin"
  }
}
```

---

### 35. Remove Capability from Role

**Endpoint**: `DELETE /api/v1/admin/roles/:roleId/capabilities/:capabilityId`

**Required Capability**: `roles:manage_capabilities` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "roleId": "role_manager",
    "capabilityId": "cap_reports_export",
    "removedAt": "2024-02-15T10:00:00Z"
  }
}
```

---

## Permission Check APIs

### 36. Check User Permission

**Endpoint**: `POST /api/v1/permissions/check`

**Required Capability**: Public (checks own permissions) or `users:inspect` (check others)

**Request Body**:
```json
{
  "userId": "user_abc",  // optional, defaults to current user
  "capability": "users:update",
  "resource": "users",
  "resourceId": "user_def",
  "context": {
    "ip": "192.168.1.100",
    "time": "2024-02-15T10:00:00Z"
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "allowed": true,
    "capability": "users:update",
    "scope": "organization",
    "source": "role:admin",
    "conditions": {
      "requireMfa": true,
      "mfaSatisfied": true
    },
    "reasoning": [
      "User has role 'admin'",
      "Role 'admin' has capability 'users:update' with scope 'organization'",
      "MFA requirement satisfied"
    ]
  }
}
```

---

### 37. Get User Effective Permissions

**Endpoint**: `GET /api/v1/users/:userId/permissions`

**Required Capability**: Public (own permissions) or `users:inspect` (others)

**Response**:
```json
{
  "success": true,
  "data": {
    "permissions": {
      "users": {
        "create": {
          "allowed": true,
          "scope": "organization",
          "source": "role:admin"
        },
        "read": {
          "allowed": true,
          "scope": "organization",
          "source": "role:admin"
        },
        "update": {
          "allowed": true,
          "scope": "organization",
          "source": "role:admin"
        },
        "delete": {
          "allowed": true,
          "scope": "organization",
          "source": "role:admin"
        }
      },
      "roles": {
        "create": {
          "allowed": true,
          "scope": "global",
          "source": "role:admin"
        }
      }
    },
    "roles": [
      {
        "id": "role_admin",
        "name": "Admin",
        "slug": "admin"
      }
    ],
    "capabilities": [
      {
        "id": "cap_users_manage",
        "slug": "users:manage",
        "resource": "users",
        "actions": ["create", "read", "update", "delete"],
        "scope": "organization"
      }
    ]
  }
}
```

---

### 38. Get Permission Matrix

**Endpoint**: `GET /api/v1/admin/permissions/matrix`

**Required Capability**: `permissions:view_matrix` (scope: organization)

**Query Parameters**:
```typescript
{
  roleIds?: string[]  // Filter by specific roles
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "matrix": [
      {
        "resource": "users",
        "actions": {
          "create": {
            "admin": true,
            "manager": false,
            "user": false
          },
          "read": {
            "admin": true,
            "manager": true,
            "user": "own"
          },
          "update": {
            "admin": true,
            "manager": "team",
            "user": "own"
          },
          "delete": {
            "admin": true,
            "manager": false,
            "user": false
          }
        }
      },
      {
        "resource": "roles",
        "actions": {
          "create": {
            "admin": true,
            "manager": false,
            "user": false
          }
        }
      }
    ],
    "roles": [
      {
        "id": "role_admin",
        "name": "Admin",
        "slug": "admin"
      },
      {
        "id": "role_manager",
        "name": "Manager",
        "slug": "manager"
      },
      {
        "id": "role_user",
        "name": "User",
        "slug": "user"
      }
    ]
  }
}
```

---

## Audit & Analytics APIs

### 39. Get Audit Logs

**Endpoint**: `GET /api/v1/admin/audit-logs`

**Required Capability**: `audit:read` (scope: organization)

**Query Parameters**:
```typescript
{
  actorId?: string
  targetType?: string
  targetId?: string
  action?: string
  resource?: string
  startDate?: string
  endDate?: string
  page?: number
  limit?: number
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "logs": [
      {
        "id": "audit_abc123",
        "actionType": "user.updated",
        "targetType": "user",
        "targetId": "user_def",
        "actor": {
          "id": "user_admin",
          "name": "Admin User",
          "email": "admin@example.com"
        },
        "actorType": "user",
        "action": "update",
        "resource": "users",
        "changes": {
          "email": {
            "old": "old@example.com",
            "new": "new@example.com"
          },
          "status": {
            "old": "active",
            "new": "suspended"
          }
        },
        "reason": "User requested email change",
        "ipAddress": "192.168.1.100",
        "userAgent": "Mozilla/5.0...",
        "metadata": {},
        "createdAt": "2024-02-15T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 5678,
      "totalPages": 114
    }
  }
}
```

---

### 40. Get User Activity Log

**Endpoint**: `GET /api/v1/admin/users/:userId/activity`

**Required Capability**: `users:inspect` (scope: organization)

**Query Parameters**:
```typescript
{
  startDate?: string
  endDate?: string
  actionType?: string
  limit?: number
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "userId": "user_abc",
    "activities": [
      {
        "id": "activity_123",
        "timestamp": "2024-02-15T10:00:00Z",
        "action": "user.login",
        "details": {
          "method": "google",
          "device": "desktop",
          "location": "San Francisco, CA"
        }
      },
      {
        "id": "activity_124",
        "timestamp": "2024-02-15T09:30:00Z",
        "action": "profile.updated",
        "details": {
          "fields": ["firstName", "lastName"]
        }
      }
    ],
    "total": 1234
  }
}
```

---

### 41. Get Analytics Dashboard

**Endpoint**: `GET /api/v1/admin/analytics/dashboard`

**Required Capability**: `analytics:read` (scope: organization)

**Query Parameters**:
```typescript
{
  period?: '24h' | '7d' | '30d' | '90d' | 'custom'
  startDate?: string
  endDate?: string
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "period": {
      "start": "2024-01-15T00:00:00Z",
      "end": "2024-02-15T00:00:00Z"
    },
    "users": {
      "total": 12345,
      "active": 8765,
      "new": 456,
      "churn": 123,
      "growth": 2.7
    },
    "logins": {
      "total": 45678,
      "unique": 8765,
      "avgPerUser": 5.2,
      "byMethod": {
        "password": 12000,
        "google": 20000,
        "github": 8000,
        "magic_link": 5678
      }
    },
    "roles": {
      "distribution": [
        {
          "role": "user",
          "count": 11000,
          "percentage": 89.1
        },
        {
          "role": "premium",
          "count": 1200,
          "percentage": 9.7
        },
        {
          "role": "admin",
          "count": 145,
          "percentage": 1.2
        }
      ]
    },
    "security": {
      "mfaEnabled": 7890,
      "mfaRate": 63.9,
      "suspiciousLogins": 45,
      "failedLogins": 234,
      "accountsLocked": 12
    },
    "communication": {
      "emailsSent": 45000,
      "emailsDelivered": 44100,
      "deliveryRate": 98.0,
      "openRate": 42.5,
      "clickRate": 18.3
    }
  }
}
```

---

### 42. Get User Analytics

**Endpoint**: `GET /api/v1/admin/analytics/users`

**Required Capability**: `analytics:read` (scope: organization)

**Query Parameters**:
```typescript
{
  period?: '24h' | '7d' | '30d' | '90d'
  metric?: 'signups' | 'logins' | 'active' | 'engagement'
  groupBy?: 'day' | 'week' | 'month'
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "metric": "signups",
    "period": "30d",
    "groupBy": "day",
    "data": [
      {
        "date": "2024-02-15",
        "value": 45,
        "cumulative": 12345
      },
      {
        "date": "2024-02-14",
        "value": 38,
        "cumulative": 12300
      }
    ],
    "summary": {
      "total": 456,
      "average": 15.2,
      "peak": 67,
      "peakDate": "2024-02-10"
    }
  }
}
```

---

## Segment Management APIs

### 43. List Segments

**Endpoint**: `GET /api/v1/admin/segments`

**Required Capability**: `segments:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "segments": [
      {
        "id": "segment_premium",
        "name": "Premium Users",
        "description": "All users with premium subscription",
        "type": "dynamic",
        "filters": {
          "roles": ["premium"],
          "status": "active"
        },
        "userCount": 1200,
        "lastUpdated": "2024-02-15T09:00:00Z",
        "createdAt": "2023-06-01T10:00:00Z"
      }
    ],
    "total": 15
  }
}
```

---

### 44. Create Segment

**Endpoint**: `POST /api/v1/admin/segments`

**Required Capability**: `segments:create` (scope: organization)

**Request Body**:
```json
{
  "name": "Active Beta Users",
  "description": "Users actively participating in beta",
  "type": "dynamic",
  "filters": {
    "tags": ["beta_tester"],
    "status": "active",
    "lastLoginAfter": "2024-02-01T00:00:00Z"
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "segment_new123",
    "name": "Active Beta Users",
    "userCount": 234,
    // ... full segment object
    "createdAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 45. Get Segment Users

**Endpoint**: `GET /api/v1/admin/segments/:segmentId/users`

**Required Capability**: `segments:read` (scope: organization)

**Query Parameters**:
```typescript
{
  page?: number
  limit?: number
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "segmentId": "segment_premium",
    "users": [
      {
        "id": "user_abc",
        "email": "user@example.com",
        "name": "John Doe",
        "roles": ["premium"],
        "addedToSegment": "2024-01-15T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 1200,
      "totalPages": 24
    }
  }
}
```

---

## Campaign Management APIs

### 46. List Campaigns

**Endpoint**: `GET /api/v1/admin/campaigns`

**Required Capability**: `campaigns:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "campaigns": [
      {
        "id": "campaign_welcome",
        "name": "Welcome Series",
        "type": "drip",
        "status": "active",
        "channel": "email",
        "recipientCount": 1234,
        "deliveredCount": 1200,
        "openedCount": 600,
        "clickedCount": 180,
        "startedAt": "2024-01-01T00:00:00Z",
        "createdAt": "2023-12-15T10:00:00Z"
      }
    ],
    "total": 23
  }
}
```

---

### 47. Create Campaign

**Endpoint**: `POST /api/v1/admin/campaigns`

**Required Capability**: `campaigns:create` (scope: organization)

**Request Body**:
```json
{
  "name": "Product Launch Announcement",
  "type": "one_time",
  "channel": "email",
  "templateId": "template_announcement",
  "recipientSegments": ["segment_active_users"],
  "scheduling": {
    "sendAt": "2024-02-20T10:00:00Z",
    "timezone": "America/Los_Angeles"
  },
  "personalization": {
    "enabled": true,
    "variables": {
      "productName": "New Feature X"
    }
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "campaign_new123",
    "name": "Product Launch Announcement",
    "status": "scheduled",
    "estimatedRecipients": 8765,
    // ... full campaign object
    "createdAt": "2024-02-15T10:00:00Z"
  }
}
```

---

### 48. Get Campaign Analytics

**Endpoint**: `GET /api/v1/admin/campaigns/:campaignId/analytics`

**Required Capability**: `campaigns:read` (scope: organization)

**Response**:
```json
{
  "success": true,
  "data": {
    "campaignId": "campaign_welcome",
    "overview": {
      "sent": 1234,
      "delivered": 1200,
      "opened": 600,
      "clicked": 180,
      "bounced": 20,
      "failed": 14
    },
    "rates": {
      "deliveryRate": 97.2,
      "openRate": 50.0,
      "clickRate": 30.0,
      "bounceRate": 1.6
    },
    "timeline": [
      {
        "timestamp": "2024-02-15T10:00:00Z",
        "event": "sent",
        "count": 1234
      }
    ],
    "topLinks": [
      {
        "url": "https://example.com/feature",
        "clicks": 120,
        "uniqueClicks": 95
      }
    ]
  }
}
```

---

## Error Responses

All endpoints return standardized error responses:

```json
{
  "success": false,
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "You do not have permission to perform this action",
    "details": {
      "required": "users:delete",
      "scope": "organization",
      "current": ["users:read", "users:update"]
    }
  },
  "meta": {
    "timestamp": "2024-02-15T10:00:00Z",
    "requestId": "req_abc123"
  }
}
```

### Common Error Codes

- `UNAUTHORIZED` - No valid authentication token
- `INSUFFICIENT_PERMISSIONS` - Missing required capability
- `NOT_FOUND` - Resource not found
- `VALIDATION_ERROR` - Request body validation failed
- `RATE_LIMIT_EXCEEDED` - Too many requests
- `CONFLICT` - Resource already exists
- `FORBIDDEN` - Action not allowed (even with permissions)
- `INTERNAL_SERVER_ERROR` - Server error

---

## Rate Limiting

All endpoints are subject to rate limiting:

**Response Headers**:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 997
X-RateLimit-Reset: 2024-02-15T11:00:00Z
```

**Rate Limit Exceeded Response**:
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 45 seconds",
    "details": {
      "limit": 1000,
      "windowMinutes": 60,
      "resetAt": "2024-02-15T11:00:00Z"
    }
  }
}
```

---

## Pagination

All list endpoints support cursor-based or offset pagination:

**Request**:
```
GET /api/v1/admin/users?page=2&limit=20
```

**Response**:
```json
{
  "data": { ... },
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 1234,
    "totalPages": 62,
    "hasNext": true,
    "hasPrev": true,
    "nextCursor": "cursor_xyz",
    "prevCursor": "cursor_abc"
  }
}
```

---

## Webhooks

Subscribe to events via webhooks:

### 49. List Webhooks

**Endpoint**: `GET /api/v1/admin/webhooks`

**Response**:
```json
{
  "success": true,
  "data": {
    "webhooks": [
      {
        "id": "webhook_123",
        "url": "https://example.com/webhook",
        "events": ["user.created", "user.updated"],
        "active": true,
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

### 50. Create Webhook

**Endpoint**: `POST /api/v1/admin/webhooks`

**Request Body**:
```json
{
  "url": "https://example.com/webhook",
  "events": ["user.created", "user.deleted"],
  "secret": "webhook_secret_key"
}
```

---

## Summary

This API documentation covers **50+ endpoints** for:

- ✅ User Management (6 endpoints)
- ✅ User Inspection (5 endpoints)
- ✅ Communication (4 endpoints)
- ✅ Template Management (6 endpoints)
- ✅ Role Management (7 endpoints)
- ✅ Capability Management (7 endpoints)
- ✅ Permission Checks (3 endpoints)
- ✅ Audit & Analytics (4 endpoints)
- ✅ Segment Management (3 endpoints)
- ✅ Campaign Management (3 endpoints)
- ✅ Webhooks (2 endpoints)

All endpoints include:
- Required capabilities
- Request/response examples
- Error handling
- Rate limiting
- Pagination support
