# Comprehensive RBAC, Roles & Capabilities System Configuration Flow

> **Foundation wiring note:** This is canonical shared authorization input for `GeneralApp`. Roles, capabilities, visibility, element state, and policy inspection must be foundation-owned and wired into both runtime enforcement and owner/admin UIs.

## Table of Contents
1. [System Overview](#system-overview)
2. [Universal RBAC Configuration Pattern](#universal-rbac-configuration-pattern)
3. [Role Hierarchy & Inheritance](#role-hierarchy--inheritance)
4. [Default System Roles](#default-system-roles)
5. [Custom Roles](#custom-roles)
6. [Capabilities & Permissions](#capabilities--permissions)
7. [Permission Resolution Engine](#permission-resolution-engine)
8. [Resource-Level Access Control](#resource-level-access-control)
9. [UI Element-Level RBAC](#ui-element-level-rbac)
10. [Attribute-Based Access Control (ABAC)](#attribute-based-access-control-abac)
11. [Multi-Tenant RBAC](#multi-tenant-rbac)
12. [Admin Configuration Interface](#admin-configuration-interface)
13. [User-Side Experience](#user-side-experience)
14. [Permission Matrix](#permission-matrix)
15. [Permission Templates & Presets](#permission-templates--presets)
16. [Role Assignment & Lifecycle](#role-assignment--lifecycle)
17. [Delegation & Impersonation](#delegation--impersonation)
18. [Scope & Context-Based Permissions](#scope--context-based-permissions)
19. [API Architecture](#api-architecture)
20. [Database Schema](#database-schema)
21. [Middleware & Guards](#middleware--guards)
22. [Audit & Compliance](#audit--compliance)
23. [Edge Cases & Conflict Resolution](#edge-cases--conflict-resolution)
24. [Performance & Caching](#performance--caching)
25. [Integration Examples](#integration-examples)
26. [Migration & Versioning](#migration--versioning)

---

## System Overview

### Core Principles
1. **Admin Full Control**: Every role, capability, and permission configurable without code changes
2. **Deny-by-Default**: No access unless explicitly granted — principle of least privilege
3. **Hierarchical Inheritance**: Roles inherit from parent roles; explicit overrides take precedence
4. **Multi-Tenant Isolation**: Each organization has its own RBAC namespace; no cross-tenant leakage
5. **Granular to Atomic**: Permissions can target entire resource types, specific resources, or individual UI elements
6. **Composable**: Roles can be combined; capabilities can be stacked; conditions can be layered
7. **Auditable**: Every permission change, role assignment, and access decision is logged
8. **Real-Time Enforcement**: Permission changes take effect immediately (no stale caches)
9. **Scalable**: Handle thousands of roles and millions of permission checks per second

### RBAC Architecture
```
System Roles (OWNER, ADMIN, MEMBER, VIEWER)
    ↓ inheritance
Custom Roles (per organization)
    ↓ composed of
Capabilities (grouped permissions)
    ↓ resolved into
Permissions (resource + action + effect + conditions)
    ↓ evaluated by
Permission Resolution Engine (hierarchy → explicit → deny-wins → conditions)
    ↓ enforced at
Middleware / API Guards / UI Conditional Rendering
```

### Access Decision Flow
```
User Request → Extract User Context (role, org, attributes)
    → Load Effective Permissions (cached)
    → Match Resource + Action
    → Evaluate Conditions (time, IP, device, attribute)
    → Apply Conflict Resolution (explicit DENY wins)
    → Return ALLOW / DENY + Reason
    → Log Decision (audit trail)
```

---

## Universal RBAC Configuration Pattern

Every RBAC element follows this universal structure:

```typescript
interface RBACConfiguration {
  // ============================================
  // CORE IDENTITY
  // ============================================
  id: string                              // "rbac-config-main"
  name: string                            // "RBAC System Configuration"
  category: RBACCategory                  // roles | permissions | capabilities | policies
  type: RBACConfigType                    // system | organization | project | resource

  // Enable/Disable
  enabled: boolean                        // Master switch for entire RBAC system
  enforcementMode: "strict" | "permissive" | "audit_only"
  // strict: deny everything not explicitly allowed
  // permissive: allow everything not explicitly denied (development mode)
  // audit_only: allow everything but log what would be denied

  // ============================================
  // ROLE CONFIGURATION
  // ============================================
  roleConfig: {
    // System default roles (immutable structure, configurable permissions)
    systemRoles: {
      owner: SystemRoleConfig
      admin: SystemRoleConfig
      member: SystemRoleConfig
      viewer: SystemRoleConfig
    }

    // Custom role settings
    customRoles: {
      enabled: boolean                    // Allow organizations to create custom roles
      maxCustomRolesPerOrg: number        // Limit custom roles (plan-based)
      maxPermissionsPerRole: number       // Prevent permission explosion
      requireDescription: boolean         // Force documentation
      requireApproval: boolean            // Require OWNER approval for new roles
      allowRoleInheritance: boolean       // Can custom roles inherit from other custom roles
      maxInheritanceDepth: number         // Prevent circular/deep inheritance
      allowRoleCombination: boolean       // Can users have multiple custom roles
      maxRolesPerUser: number             // Limit role stacking
      namingConvention: string            // Regex for role names
      reservedNames: string[]             // Cannot be used as custom role names
      autoExpiry: {
        enabled: boolean
        defaultExpiryDays: number         // Auto-expire custom roles
        notifyBeforeExpiry: number        // Days before expiry to notify
      }
    }

    // Role hierarchy
    hierarchy: {
      enabled: boolean
      mode: "strict" | "flexible"
      // strict: OWNER > ADMIN > MEMBER > VIEWER (fixed)
      // flexible: admin can reorder within constraints
      allowCrossLevelGrants: boolean      // Can ADMIN grant ADMIN-level permissions?
      preventPrivilegeEscalation: boolean // Cannot grant permissions you don't have
    }

    // Default role assignment
    defaultAssignment: {
      newUserRole: "MEMBER" | "VIEWER" | string  // Default role for new users
      invitedUserRole: "MEMBER" | "VIEWER" | string
      ssoUserRole: "MEMBER" | "VIEWER" | string
      apiUserRole: "MEMBER" | "VIEWER" | string
      roleByEmailDomain: {
        enabled: boolean
        mappings: {
          domain: string                  // "company.com"
          role: string                    // "ADMIN"
          customRoleId?: string
        }[]
      }
      roleByInviteLink: {
        enabled: boolean
        allowRoleInInvite: boolean        // Inviter can specify role
        maxRoleInInvite: string           // Maximum role inviter can assign
      }
    }
  }

  // ============================================
  // CAPABILITY CONFIGURATION
  // ============================================
  capabilityConfig: {
    // Capability groups (logical grouping of permissions)
    groups: {
      id: string                          // "project-management"
      name: string                        // "Project Management"
      description: string
      icon: string
      category: CapabilityCategory        // content | admin | billing | system | integration
      capabilities: Capability[]
    }[]

    // Custom capabilities
    customCapabilities: {
      enabled: boolean
      maxPerOrg: number
      requireApproval: boolean
      allowConditions: boolean            // Can custom capabilities have conditions
      allowUIProperties: boolean          // Can custom capabilities control UI elements
    }

    // Capability inheritance
    inheritance: {
      enabled: boolean
      mode: "additive" | "override"
      // additive: child inherits all parent capabilities + its own
      // override: child capabilities replace parent's for same resource
    }
  }

  // ============================================
  // PERMISSION CONFIGURATION
  // ============================================
  permissionConfig: {
    // Permission model
    model: "rbac" | "abac" | "rbac_abac_hybrid"

    // Resource types
    resourceTypes: ResourceTypeConfig[]

    // Action types
    actionTypes: ActionTypeConfig[]

    // Permission effects
    effects: {
      allow: boolean                      // Enable ALLOW effect
      deny: boolean                       // Enable DENY effect
      conditional: boolean                // Enable conditional permissions
    }

    // Conflict resolution
    conflictResolution: {
      strategy: "deny_wins" | "allow_wins" | "most_specific_wins" | "explicit_wins"
      // deny_wins: if any rule says DENY, result is DENY
      // allow_wins: if any rule says ALLOW, result is ALLOW
      // most_specific_wins: resource-specific > type-level > global
      // explicit_wins: explicit user permission > role permission > inherited
      logConflicts: boolean               // Log when conflicts are resolved
      notifyAdminOnConflict: boolean
    }

    // Wildcard permissions
    wildcards: {
      enabled: boolean                    // Allow "*" for all resources/actions
      allowInCustomRoles: boolean         // Can custom roles use wildcards
      requireOwnerApproval: boolean       // Wildcard permissions need OWNER approval
    }

    // Time-based permissions
    timeBased: {
      enabled: boolean
      allowScheduledPermissions: boolean  // Permissions active only during certain times
      allowTemporaryElevation: boolean    // Temporary role upgrades
      maxElevationDuration: number        // Hours
      requireJustification: boolean       // Must provide reason for elevation
    }

    // IP-based permissions
    ipBased: {
      enabled: boolean
      allowIpRestrictions: boolean        // Restrict permissions to specific IPs
      allowIpWhitelist: boolean
      allowIpBlacklist: boolean
      allowGeoRestrictions: boolean
    }

    // Resource ownership
    ownership: {
      enabled: boolean
      ownerHasFullAccess: boolean         // Resource creator has all permissions
      ownerCanDelegate: boolean           // Owner can grant access to others
      ownerCanTransfer: boolean           // Owner can transfer ownership
      inheritOrganizationPermissions: boolean
    }
  }

  // ============================================
  // ADMIN INTERFACE CONFIGURATION
  // ============================================
  adminInterface: {
    // Permission matrix view
    matrixView: {
      enabled: boolean
      showResourceTypes: boolean
      showActions: boolean
      showRoles: boolean
      showCustomRoles: boolean
      showConditions: boolean
      showUIProperties: boolean
      allowBulkEdit: boolean
      allowExport: boolean
      allowImport: boolean
      colorCoding: {
        allow: string                     // "#22c55e" green
        deny: string                      // "#ef4444" red
        conditional: string               // "#f59e0b" amber
        inherited: string                 // "#6366f1" indigo
        notSet: string                    // "#9ca3af" gray
      }
    }

    // Role management view
    roleManagement: {
      enabled: boolean
      showUserCount: boolean
      showPermissionCount: boolean
      showInheritanceTree: boolean
      allowCloneRole: boolean
      allowCompareRoles: boolean
      showEffectivePermissions: boolean   // Show resolved permissions after inheritance
    }

    // Audit view
    auditView: {
      enabled: boolean
      showAccessDecisions: boolean
      showPermissionChanges: boolean
      showRoleAssignments: boolean
      retentionDays: number
      allowExport: boolean
      realTimeStream: boolean
    }

    // Testing & simulation
    simulation: {
      enabled: boolean
      allowTestAsUser: boolean            // "What would user X see?"
      allowTestAsRole: boolean            // "What would role Y allow?"
      showAccessDecisionExplanation: boolean
      allowDryRunChanges: boolean         // Preview permission changes before applying
    }
  }

  // ============================================
  // VISIBILITY & CONDITIONAL RENDERING
  // ============================================
  visibility: {
    // Who can see RBAC configuration
    visibleToRoles: string[]              // ["OWNER", "ADMIN"]
    visibleToCustomRoles: string[]

    // What users can see about their own permissions
    userSelfService: {
      canViewOwnRole: boolean
      canViewOwnPermissions: boolean
      canViewOwnCapabilities: boolean
      canRequestRoleChange: boolean
      canRequestPermissionChange: boolean
      canViewRoleHierarchy: boolean
      canViewAvailableRoles: boolean
    }

    // Conditional rendering rules
    conditionalRendering: {
      hideUnauthorizedElements: boolean   // Hide vs disable unauthorized UI elements
      showDisabledWithTooltip: boolean    // Show disabled with "Upgrade to access"
      showUpgradePrompt: boolean          // Show upgrade CTA for plan-restricted features
      customDeniedMessage: string         // "You don't have permission to..."
      redirectOnDenied: string            // "/dashboard" or "/upgrade"
    }
  }

  // ============================================
  // PLAN-BASED RESTRICTIONS
  // ============================================
  planRestrictions: {
    enabled: boolean
    plans: {
      planId: string                      // "FREE", "STARTER", "PRO", "ENTERPRISE"
      maxCustomRoles: number
      maxPermissionsPerRole: number
      maxCapabilities: number
      allowedFeatures: string[]           // Feature flags tied to plan
      allowCustomCapabilities: boolean
      allowConditions: boolean
      allowTimeBased: boolean
      allowIPBased: boolean
      allowUILevelRBAC: boolean
      allowABAC: boolean
      allowDelegation: boolean
      allowImpersonation: boolean
      allowAuditExport: boolean
      maxAuditRetentionDays: number
    }[]
  }
}
```

---

## Role Hierarchy & Inheritance

### Hierarchy Model

```typescript
interface RoleHierarchy {
  // ============================================
  // SYSTEM ROLE HIERARCHY (Immutable Order)
  // ============================================
  systemHierarchy: {
    // Level 0 - Platform Owner (God Mode)
    OWNER: {
      level: 0
      inheritsFrom: null                  // Top of hierarchy
      canManage: ["ADMIN", "MEMBER", "VIEWER", "CUSTOM"]
      canCreate: ["ADMIN", "MEMBER", "VIEWER", "CUSTOM"]
      canDelete: ["ADMIN", "MEMBER", "VIEWER", "CUSTOM"]
      canImpersonate: ["ADMIN", "MEMBER", "VIEWER", "CUSTOM"]
      restrictions: {
        canBeDeleted: false               // Cannot delete last OWNER
        canBeDemoted: false               // Cannot demote last OWNER
        requiresMFA: true                 // OWNER must have MFA
        maxPerOrganization: number        // Usually 1-3
      }
    }

    // Level 1 - Organization Admin
    ADMIN: {
      level: 1
      inheritsFrom: null                  // Does NOT inherit from OWNER
      canManage: ["MEMBER", "VIEWER", "CUSTOM"]
      canCreate: ["MEMBER", "VIEWER", "CUSTOM"]
      canDelete: ["MEMBER", "VIEWER"]     // Cannot delete custom roles (OWNER only)
      canImpersonate: ["MEMBER", "VIEWER"]
      restrictions: {
        canBeDeleted: true
        canBeDemoted: true
        requiresMFA: boolean              // Configurable
        maxPerOrganization: number
        cannotModifyOwner: true           // Cannot change OWNER settings
        cannotAccessOwnerAudit: boolean   // Configurable
      }
    }

    // Level 2 - Regular Member
    MEMBER: {
      level: 2
      inheritsFrom: null
      canManage: []                       // Cannot manage other users
      canCreate: []
      canDelete: []
      canImpersonate: []
      restrictions: {
        canBeDeleted: true
        canBeDemoted: true
        requiresMFA: boolean              // Configurable per org
        maxPerOrganization: number        // Plan-based limit
      }
    }

    // Level 3 - Read-Only Viewer
    VIEWER: {
      level: 3
      inheritsFrom: null
      canManage: []
      canCreate: []
      canDelete: []
      canImpersonate: []
      restrictions: {
        canBeDeleted: true
        canBeDemoted: false               // Already lowest
        requiresMFA: boolean
        maxPerOrganization: number
      }
    }
  }

  // ============================================
  // CUSTOM ROLE HIERARCHY
  // ============================================
  customRoleHierarchy: {
    // Custom roles are anchored to a base system role
    // They can ONLY have permissions equal to or less than their base role
    // They can NEVER exceed their base role's level

    // Example: "Project Manager" based on MEMBER
    // Can have: all MEMBER permissions + specific project management permissions
    // Cannot have: ADMIN-level permissions (user management, billing, etc.)

    inheritanceRules: {
      // What a custom role inherits from its base role
      inheritsAllBasePermissions: boolean   // true = starts with all base role perms
      inheritsUIProperties: boolean
      inheritsConditions: boolean

      // Override behavior
      canOverrideBasePermissions: boolean   // Can restrict base role permissions
      canAddPermissions: boolean            // Can add permissions beyond base role
      addPermissionScope: "same_level" | "below_level" | "any"
      // same_level: can only add permissions at same level as base role
      // below_level: can only add permissions below base role level
      // any: can add any permission (dangerous, requires OWNER approval)
    }

    // Inheritance chain resolution
    resolution: {
      maxDepth: number                      // Maximum inheritance chain depth
      circularDetection: boolean            // Prevent circular inheritance
      conflictStrategy: "parent_wins" | "child_wins" | "most_restrictive"
    }
  }
}
```

### Inheritance Resolution Algorithm

```typescript
function resolveEffectivePermissions(
  user: User,
  organization: Organization
): EffectivePermission[] {
  const permissions: EffectivePermission[] = []

  // Step 1: Get system role permissions
  const systemRolePerms = getSystemRolePermissions(user.role)
  permissions.push(...systemRolePerms.map(p => ({
    ...p,
    source: 'system_role',
    priority: 1
  })))

  // Step 2: Get custom role permissions (if assigned)
  if (user.customRoleId) {
    const customRole = getCustomRole(user.customRoleId)

    // Step 2a: Get base role permissions (inherited)
    if (customRole.inheritsAllBasePermissions) {
      const basePerms = getSystemRolePermissions(customRole.baseRole)
      permissions.push(...basePerms.map(p => ({
        ...p,
        source: 'custom_role_inherited',
        priority: 2
      })))
    }

    // Step 2b: Get custom role's own permissions
    const customPerms = getCustomRolePermissions(customRole.id)
    permissions.push(...customPerms.map(p => ({
      ...p,
      source: 'custom_role_explicit',
      priority: 3
    })))

    // Step 2c: Resolve inheritance chain (if role inherits from another custom role)
    if (customRole.parentRoleId) {
      const inheritedPerms = resolveInheritanceChain(customRole.parentRoleId, 0)
      permissions.push(...inheritedPerms.map(p => ({
        ...p,
        source: 'custom_role_chain',
        priority: 2.5
      })))
    }
  }

  // Step 3: Get direct user permissions (highest priority)
  const directPerms = getDirectUserPermissions(user.id, organization.id)
  permissions.push(...directPerms.map(p => ({
    ...p,
    source: 'direct_user',
    priority: 4
  })))

  // Step 4: Resolve conflicts
  return resolveConflicts(permissions)
}

function resolveConflicts(permissions: EffectivePermission[]): EffectivePermission[] {
  const grouped = groupBy(permissions, p => `${p.resourceType}:${p.resourceId}:${p.action}`)

  return Object.values(grouped).map(group => {
    // Sort by priority (highest first)
    group.sort((a, b) => b.priority - a.priority)

    // Check for explicit DENY (deny always wins regardless of priority)
    const explicitDeny = group.find(p => p.effect === 'DENY' && p.source === 'direct_user')
    if (explicitDeny) return { ...explicitDeny, resolvedBy: 'explicit_deny' }

    // Check for any DENY at same or higher priority
    const anyDeny = group.find(p => p.effect === 'DENY')
    const anyAllow = group.find(p => p.effect === 'ALLOW')

    if (anyDeny && anyAllow) {
      // Conflict! Apply resolution strategy
      switch (config.conflictResolution.strategy) {
        case 'deny_wins':
          return { ...anyDeny, resolvedBy: 'deny_wins' }
        case 'most_specific_wins':
          // Resource-specific > type-level > global
          const mostSpecific = group.sort((a, b) =>
            (b.resourceId ? 1 : 0) - (a.resourceId ? 1 : 0)
          )[0]
          return { ...mostSpecific, resolvedBy: 'most_specific' }
        case 'explicit_wins':
          return { ...group[0], resolvedBy: 'highest_priority' }
        default:
          return { ...anyDeny, resolvedBy: 'default_deny' }
      }
    }

    // No conflict - return highest priority
    return { ...group[0], resolvedBy: 'no_conflict' }
  })
}
```

---

## Default System Roles

### OWNER Role Configuration

```typescript
interface OwnerRoleConfig {
  id: "OWNER"
  name: "Platform Owner"
  description: "Full platform control. Super administrator with unrestricted access."
  level: 0
  isSystem: true
  isDeletable: false
  isModifiable: false                     // Cannot modify OWNER role structure

  // Permissions (all resources, all actions, ALLOW)
  defaultPermissions: {
    // Platform Administration
    "SYSTEM_CONFIG:*": "ALLOW"            // All system configuration
    "USER_MANAGEMENT:*": "ALLOW"          // All user management
    "ORGANIZATION_MANAGEMENT:*": "ALLOW"  // All org management
    "BILLING:*": "ALLOW"                  // All billing
    "SETTINGS:*": "ALLOW"                 // All settings

    // Content & Features
    "PROJECT:*": "ALLOW"
    "BLUEPRINT:*": "ALLOW"
    "MINDMAP:*": "ALLOW"
    "AI_GENERATION:*": "ALLOW"
    "EXPORT:*": "ALLOW"
    "IMPORT:*": "ALLOW"
    "COLLABORATION:*": "ALLOW"

    // RBAC Management
    "RBAC:*": "ALLOW"                     // Can manage all RBAC settings
    "ROLE:*": "ALLOW"                     // Can manage all roles
    "PERMISSION:*": "ALLOW"              // Can manage all permissions
    "CAPABILITY:*": "ALLOW"              // Can manage all capabilities

    // Audit & Compliance
    "AUDIT:*": "ALLOW"
    "COMPLIANCE:*": "ALLOW"

    // Danger Zone
    "IMPERSONATION:*": "ALLOW"           // Can impersonate any user
    "DATA_EXPORT:*": "ALLOW"             // Can export all data
    "DATA_DELETE:*": "ALLOW"             // Can delete any data
    "ORGANIZATION_DELETE:*": "ALLOW"     // Can delete organization
  }

  // UI Visibility
  uiAccess: {
    adminPanel: true
    rbacManager: true
    auditLogs: true
    systemSettings: true
    billingSettings: true
    dangerZone: true
    impersonation: true
    allDashboards: true
    allReports: true
  }

  // Restrictions
  restrictions: {
    requireMFA: true
    requireStrongPassword: true
    sessionTimeout: 30                    // Minutes (shorter for security)
    ipWhitelist: string[]                 // Optional IP restriction
    cannotBeImpersonated: true
    cannotBeBulkModified: true
    requireReauthForDangerousActions: true
  }

  // Admin-configurable aspects of OWNER role
  configurableBy: "OWNER"                // Only OWNER can configure OWNER
  configurable: {
    sessionTimeout: boolean               // Can change session timeout
    ipWhitelist: boolean                  // Can set IP whitelist
    mfaRequirement: boolean               // Can toggle MFA requirement
    notificationPreferences: boolean      // Can change notification settings
  }
}
```

### ADMIN Role Configuration

```typescript
interface AdminRoleConfig {
  id: "ADMIN"
  name: "Organization Admin"
  description: "Organization-level administration. Can manage users, roles, and settings."
  level: 1
  isSystem: true
  isDeletable: false
  isModifiable: true                      // OWNER can modify ADMIN permissions

  defaultPermissions: {
    // User Management (within organization)
    "USER_MANAGEMENT:VIEW": "ALLOW"
    "USER_MANAGEMENT:CREATE": "ALLOW"
    "USER_MANAGEMENT:EDIT": "ALLOW"
    "USER_MANAGEMENT:DELETE": "ALLOW"     // Can delete non-admin users
    "USER_MANAGEMENT:MANAGE": "ALLOW"

    // Organization Settings
    "ORGANIZATION_MANAGEMENT:VIEW": "ALLOW"
    "ORGANIZATION_MANAGEMENT:EDIT": "ALLOW"
    "ORGANIZATION_MANAGEMENT:CONFIGURE": "ALLOW"
    // Note: No DELETE - cannot delete organization

    // RBAC (limited)
    "RBAC:VIEW": "ALLOW"
    "RBAC:CONFIGURE": "ALLOW"             // Can configure RBAC
    "ROLE:VIEW": "ALLOW"
    "ROLE:CREATE": "ALLOW"                // Can create custom roles
    "ROLE:EDIT": "ALLOW"
    "ROLE:DELETE": "ALLOW"                // Can delete custom roles
    "PERMISSION:VIEW": "ALLOW"
    "PERMISSION:CREATE": "ALLOW"
    "PERMISSION:EDIT": "ALLOW"
    "PERMISSION:DELETE": "ALLOW"
    // Note: Cannot create roles above ADMIN level
    // Note: Cannot grant permissions they don't have

    // Content & Features
    "PROJECT:*": "ALLOW"
    "BLUEPRINT:*": "ALLOW"
    "MINDMAP:*": "ALLOW"
    "AI_GENERATION:*": "ALLOW"
    "EXPORT:*": "ALLOW"
    "IMPORT:*": "ALLOW"
    "COLLABORATION:*": "ALLOW"

    // Billing (view only by default)
    "BILLING:VIEW": "ALLOW"
    "BILLING:MANAGE": "DENY"              // Cannot manage billing by default

    // Settings
    "SETTINGS:VIEW": "ALLOW"
    "SETTINGS:EDIT": "ALLOW"
    "SETTINGS:CONFIGURE": "ALLOW"

    // Audit (limited)
    "AUDIT:VIEW": "ALLOW"
    "AUDIT:EXPORT": "DENY"               // Cannot export audit logs by default

    // Danger Zone
    "IMPERSONATION:EXECUTE": "DENY"      // Cannot impersonate by default
    "DATA_DELETE:EXECUTE": "DENY"         // Cannot bulk delete by default
    "ORGANIZATION_DELETE:EXECUTE": "DENY" // Cannot delete organization
  }

  // What OWNER can configure about ADMIN role
  ownerConfigurable: {
    canManageBilling: boolean             // Toggle billing management
    canExportAudit: boolean               // Toggle audit export
    canImpersonate: boolean               // Toggle impersonation
    canBulkDelete: boolean                // Toggle bulk delete
    canManageIntegrations: boolean        // Toggle integration management
    canAccessSystemConfig: boolean        // Toggle system config access
    maxUsersCanManage: number             // Limit users admin can manage
    canCreateAdmins: boolean              // Can admin create other admins
    canModifyOwnPermissions: boolean      // Can admin modify own permissions
  }

  restrictions: {
    requireMFA: boolean                   // Configurable by OWNER
    sessionTimeout: number                // Configurable by OWNER
    cannotModifyOwner: true               // Hard restriction
    cannotDeleteOwner: true               // Hard restriction
    cannotEscalateOwnRole: true           // Hard restriction
    cannotGrantAboveOwnLevel: true        // Hard restriction
  }
}
```

### MEMBER Role Configuration

```typescript
interface MemberRoleConfig {
  id: "MEMBER"
  name: "Member"
  description: "Standard user with content creation and collaboration access."
  level: 2
  isSystem: true
  isDeletable: false
  isModifiable: true                      // OWNER/ADMIN can modify

  defaultPermissions: {
    // Own Profile
    "SETTINGS:VIEW": "ALLOW"              // Own settings only
    "SETTINGS:EDIT": "ALLOW"              // Own settings only

    // Content (own + shared)
    "PROJECT:VIEW": "ALLOW"               // Projects they have access to
    "PROJECT:CREATE": "ALLOW"
    "PROJECT:EDIT": "ALLOW"               // Own projects + shared with edit
    "PROJECT:DELETE": "ALLOW"             // Own projects only

    "BLUEPRINT:VIEW": "ALLOW"
    "BLUEPRINT:CREATE": "ALLOW"
    "BLUEPRINT:EDIT": "ALLOW"
    "BLUEPRINT:DELETE": "ALLOW"           // Own blueprints only

    "MINDMAP:VIEW": "ALLOW"
    "MINDMAP:CREATE": "ALLOW"
    "MINDMAP:EDIT": "ALLOW"
    "MINDMAP:DELETE": "ALLOW"             // Own mindmaps only

    // Features
    "AI_GENERATION:VIEW": "ALLOW"
    "AI_GENERATION:EXECUTE": "ALLOW"      // Within quota
    "EXPORT:EXECUTE": "ALLOW"             // Own content
    "IMPORT:EXECUTE": "ALLOW"
    "COLLABORATION:VIEW": "ALLOW"
    "COLLABORATION:EXECUTE": "ALLOW"

    // Admin (denied)
    "USER_MANAGEMENT:*": "DENY"
    "ORGANIZATION_MANAGEMENT:*": "DENY"
    "BILLING:*": "DENY"
    "SYSTEM_CONFIG:*": "DENY"
    "RBAC:*": "DENY"
    "AUDIT:*": "DENY"
    "IMPERSONATION:*": "DENY"
  }

  // What ADMIN/OWNER can configure about MEMBER role
  adminConfigurable: {
    canCreateProjects: boolean
    canDeleteOwnProjects: boolean
    canExport: boolean
    canImport: boolean
    canUseAI: boolean
    canCollaborate: boolean
    canInviteMembers: boolean             // Can invite other members
    canViewTeamMembers: boolean           // Can see team list
    canViewOrgSettings: boolean           // Can view (not edit) org settings
    maxProjects: number                   // Per-user project limit
    maxBlueprints: number
    maxAICredits: number
    allowedExportFormats: string[]
  }

  // Ownership rules
  ownership: {
    hasFullAccessToOwnResources: true
    canShareOwnResources: boolean         // Configurable
    canTransferOwnership: boolean         // Configurable
    canSetResourcePermissions: boolean    // Can set who can view/edit their resources
  }
}
```

### VIEWER Role Configuration

```typescript
interface ViewerRoleConfig {
  id: "VIEWER"
  name: "Viewer"
  description: "Read-only access. Can view content but cannot create or modify."
  level: 3
  isSystem: true
  isDeletable: false
  isModifiable: true

  defaultPermissions: {
    // Own Profile (limited)
    "SETTINGS:VIEW": "ALLOW"              // Own settings only

    // Content (read-only)
    "PROJECT:VIEW": "ALLOW"               // Projects shared with them
    "PROJECT:CREATE": "DENY"
    "PROJECT:EDIT": "DENY"
    "PROJECT:DELETE": "DENY"

    "BLUEPRINT:VIEW": "ALLOW"
    "BLUEPRINT:CREATE": "DENY"
    "BLUEPRINT:EDIT": "DENY"
    "BLUEPRINT:DELETE": "DENY"

    "MINDMAP:VIEW": "ALLOW"
    "MINDMAP:CREATE": "DENY"
    "MINDMAP:EDIT": "DENY"
    "MINDMAP:DELETE": "DENY"

    // Features (limited)
    "AI_GENERATION:VIEW": "ALLOW"
    "AI_GENERATION:EXECUTE": "DENY"
    "EXPORT:EXECUTE": "DENY"              // Cannot export by default
    "IMPORT:EXECUTE": "DENY"
    "COLLABORATION:VIEW": "ALLOW"
    "COLLABORATION:EXECUTE": "DENY"       // Cannot collaborate (comment, etc.)

    // Admin (all denied)
    "USER_MANAGEMENT:*": "DENY"
    "ORGANIZATION_MANAGEMENT:*": "DENY"
    "BILLING:*": "DENY"
    "SYSTEM_CONFIG:*": "DENY"
    "RBAC:*": "DENY"
    "AUDIT:*": "DENY"
    "IMPERSONATION:*": "DENY"
  }

  adminConfigurable: {
    canExport: boolean                    // Allow export for viewers
    canComment: boolean                   // Allow commenting
    canReact: boolean                     // Allow reactions/likes
    canViewTeamMembers: boolean
    canViewActivityFeed: boolean
    canDownloadAttachments: boolean
    canViewAnalytics: boolean             // View-only analytics
  }
}
```

---

## Custom Roles

### Custom Role Configuration

```typescript
interface CustomRoleConfiguration {
  // ============================================
  // CUSTOM ROLE DEFINITION
  // ============================================
  id: string                              // cuid
  organizationId: string
  name: string                            // "Project Manager"
  slug: string                            // "project-manager" (auto-generated)
  description: string                     // Required description
  icon: string                            // "shield-check"
  color: string                           // "#8b5cf6" for visual identification

  // Base role (determines maximum permission level)
  baseRole: "ADMIN" | "MEMBER" | "VIEWER" // Cannot base on OWNER
  // If baseRole is MEMBER, this custom role can NEVER have ADMIN-level permissions

  // Inheritance
  parentCustomRoleId?: string             // Inherit from another custom role
  inheritsAllParentPermissions: boolean   // Start with parent's permissions

  // Status
  isActive: boolean
  isDefault: boolean                      // Auto-assign to new users matching criteria
  isTemporary: boolean                    // Auto-expires
  expiresAt?: Date

  // Assignment criteria (auto-assign based on conditions)
  autoAssignCriteria?: {
    emailDomains: string[]                // Auto-assign for @company.com
    departments: string[]                 // Auto-assign for "Engineering"
    locations: string[]                   // Auto-assign for "US"
    onboardingAnswers: Record<string, string>  // Based on onboarding questionnaire
  }

  // Metadata
  createdById: string
  createdAt: Date
  updatedAt: Date
  version: number                         // Track changes for audit

  // ============================================
  // CUSTOM ROLE PERMISSIONS
  // ============================================
  permissions: CustomRolePermission[]

  // ============================================
  // CUSTOM ROLE CAPABILITIES
  // ============================================
  capabilities: string[]                  // Capability IDs

  // ============================================
  // CUSTOM ROLE RESTRICTIONS
  // ============================================
  restrictions: {
    maxUsers: number                      // Max users with this role
    requireMFA: boolean
    sessionTimeout: number                // Minutes
    ipRestrictions: string[]              // CIDR ranges
    timeRestrictions: {
      enabled: boolean
      allowedDays: number[]               // 0=Sun, 1=Mon, ..., 6=Sat
      allowedHoursStart: string           // "09:00"
      allowedHoursEnd: string             // "17:00"
      timezone: string                    // "America/New_York"
    }
    deviceRestrictions: {
      enabled: boolean
      allowedPlatforms: string[]
      requireTrustedDevice: boolean
    }
  }
}
```

### Custom Role CRUD Scenarios

```typescript
// ============================================
// SCENARIO 1: Admin Creates "Project Manager" Role
// ============================================
// Route: POST /api/admin/rbac/roles
// Auth: OWNER or ADMIN
// Body:
{
  name: "Project Manager",
  description: "Can manage projects, assign team members, and view analytics",
  baseRole: "MEMBER",
  permissions: [
    // Project management
    { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW" },
    { resourceType: "PROJECT", action: "CREATE", effect: "ALLOW" },
    { resourceType: "PROJECT", action: "EDIT", effect: "ALLOW" },
    { resourceType: "PROJECT", action: "DELETE", effect: "ALLOW" },
    { resourceType: "PROJECT", action: "MANAGE", effect: "ALLOW" },

    // Team viewing (but not management)
    { resourceType: "USER_MANAGEMENT", action: "VIEW", effect: "ALLOW" },

    // Blueprint access
    { resourceType: "BLUEPRINT", action: "VIEW", effect: "ALLOW" },
    { resourceType: "BLUEPRINT", action: "CREATE", effect: "ALLOW" },
    { resourceType: "BLUEPRINT", action: "EDIT", effect: "ALLOW" },

    // Analytics
    { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
      conditions: { scope: "analytics_only" }
    },

    // Export
    { resourceType: "EXPORT", action: "EXECUTE", effect: "ALLOW",
      conditions: { formats: ["pdf", "csv"] }
    }
  ]
}

// Response:
{
  role: {
    id: "clx...",
    name: "Project Manager",
    baseRole: "MEMBER",
    permissions: [...],
    _count: { users: 0, permissions: 12 }
  }
}

// ============================================
// SCENARIO 2: Admin Creates "Billing Manager" Role
// ============================================
{
  name: "Billing Manager",
  description: "Can view and manage billing, subscriptions, and invoices",
  baseRole: "ADMIN",                      // Needs ADMIN base for billing access
  permissions: [
    { resourceType: "BILLING", action: "VIEW", effect: "ALLOW" },
    { resourceType: "BILLING", action: "MANAGE", effect: "ALLOW" },
    { resourceType: "BILLING", action: "CONFIGURE", effect: "ALLOW" },

    // Deny everything else that ADMIN normally has
    { resourceType: "USER_MANAGEMENT", action: "CREATE", effect: "DENY" },
    { resourceType: "USER_MANAGEMENT", action: "EDIT", effect: "DENY" },
    { resourceType: "USER_MANAGEMENT", action: "DELETE", effect: "DENY" },
    { resourceType: "RBAC", action: "CONFIGURE", effect: "DENY" },
    { resourceType: "SYSTEM_CONFIG", action: "CONFIGURE", effect: "DENY" },
  ]
}

// ============================================
// SCENARIO 3: Admin Creates "Content Reviewer" Role
// ============================================
{
  name: "Content Reviewer",
  description: "Can view and comment on content but not create or modify",
  baseRole: "VIEWER",
  permissions: [
    // Upgrade from VIEWER: allow commenting
    { resourceType: "COLLABORATION", action: "EXECUTE", effect: "ALLOW",
      conditions: { actions: ["comment", "react"] }
    },
    // Allow viewing analytics
    { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
      conditions: { scope: "analytics_only" }
    }
  ]
}

// ============================================
// SCENARIO 4: Admin Creates Temporary "Contractor" Role
// ============================================
{
  name: "Contractor",
  description: "Temporary access for external contractors",
  baseRole: "MEMBER",
  isTemporary: true,
  expiresAt: "2026-03-15T00:00:00Z",     // Auto-expires
  restrictions: {
    maxUsers: 10,
    requireMFA: true,
    ipRestrictions: ["203.0.113.0/24"],   // Office IP only
    timeRestrictions: {
      enabled: true,
      allowedDays: [1, 2, 3, 4, 5],      // Weekdays only
      allowedHoursStart: "09:00",
      allowedHoursEnd: "18:00",
      timezone: "America/New_York"
    }
  },
  permissions: [
    { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW",
      resourceId: "specific-project-id"   // Only specific project
    },
    { resourceType: "PROJECT", action: "EDIT", effect: "ALLOW",
      resourceId: "specific-project-id"
    },
    { resourceType: "EXPORT", action: "EXECUTE", effect: "DENY" },  // No export
    { resourceType: "IMPORT", action: "EXECUTE", effect: "DENY" },  // No import
  ]
}

// ============================================
// SCENARIO 5: Clone Existing Role
// ============================================
// Route: POST /api/admin/rbac/roles/clone
{
  sourceRoleId: "clx-project-manager-id",
  newName: "Senior Project Manager",
  newDescription: "Extended project manager with team management",
  additionalPermissions: [
    { resourceType: "USER_MANAGEMENT", action: "MANAGE", effect: "ALLOW",
      conditions: { scope: "team_only" }
    }
  ],
  removedPermissions: []                  // Permission IDs to remove from clone
}

// ============================================
// SCENARIO 6: Compare Two Roles
// ============================================
// Route: GET /api/admin/rbac/roles/compare?roleA=clx-pm&roleB=clx-spm
// Response:
{
  roleA: { name: "Project Manager", permissions: [...] },
  roleB: { name: "Senior Project Manager", permissions: [...] },
  differences: {
    onlyInA: [],                          // Permissions only in role A
    onlyInB: [                            // Permissions only in role B
      { resourceType: "USER_MANAGEMENT", action: "MANAGE", effect: "ALLOW" }
    ],
    different: [],                        // Same resource+action but different effect
    identical: [...]                      // Same in both
  }
}

// ============================================
// SCENARIO 7: Admin Tries to Create Role Above Their Level
// ============================================
// Admin (level 1) tries to create role based on OWNER (level 0)
// Route: POST /api/admin/rbac/roles
{
  name: "Super Admin",
  baseRole: "OWNER"                       // ← REJECTED
}
// Response: 403 Forbidden
// { error: "Cannot create role based on OWNER. Maximum base role for your level: ADMIN" }

// ============================================
// SCENARIO 8: Delete Role with Active Users
// ============================================
// Route: DELETE /api/admin/rbac/roles/clx-contractor-id
// Response: 409 Conflict
{
  error: "Cannot delete role with active users",
  activeUsers: 5,
  options: [
    { action: "reassign", description: "Reassign users to another role before deleting" },
    { action: "force_delete", description: "Force delete and revert users to base role",
      requiresOwner: true }
  ]
}

// Route: DELETE /api/admin/rbac/roles/clx-contractor-id?action=reassign&targetRole=MEMBER
// Response: 200 OK
{
  success: true,
  reassignedUsers: 5,
  deletedRole: "Contractor"
}
```

---

## Capabilities & Permissions

### Capability Definition

```typescript
interface Capability {
  // ============================================
  // CAPABILITY IDENTITY
  // ============================================
  id: string                              // "cap-project-create"
  name: string                            // "Create Projects"
  slug: string                            // "project.create"
  description: string                     // "Allows creating new projects"
  category: CapabilityCategory
  icon: string
  
  // ============================================
  // CAPABILITY PERMISSIONS
  // ============================================
  // A capability is a named group of one or more permissions
  permissions: {
    resourceType: ResourceType
    action: ActionType
    effect: PermissionEffect
    resourceId?: string                   // Specific resource (optional)
    conditions?: PermissionCondition      // Conditional access
    uiProperties?: UIProperties           // UI element control
  }[]

  // ============================================
  // CAPABILITY METADATA
  // ============================================
  isSystem: boolean                       // System-defined (cannot be deleted)
  isCustom: boolean                       // Organization-defined
  organizationId?: string                 // null for system capabilities
  
  // Dependencies
  requires: string[]                      // Other capability IDs required
  conflicts: string[]                     // Capability IDs that conflict
  
  // Plan restrictions
  availableInPlans: string[]              // ["FREE", "STARTER", "PRO", "ENTERPRISE"]
  
  // Visibility
  visibleToRoles: string[]                // Who can see this capability exists
  assignableByRoles: string[]             // Who can assign this capability
}

// ============================================
// CAPABILITY CATEGORIES
// ============================================
type CapabilityCategory =
  | "content_management"                  // Projects, blueprints, mindmaps
  | "team_management"                     // Users, invitations, teams
  | "organization_admin"                  // Org settings, branding
  | "billing_management"                  // Subscriptions, invoices
  | "system_admin"                        // System config, maintenance
  | "security"                            // MFA, sessions, audit
  | "integration"                         // APIs, webhooks, MCP
  | "ai_features"                         // AI generation, credits
  | "analytics"                           // Reports, dashboards
  | "custom"                              // Organization-defined
```

### System Capabilities Catalog

```typescript
const SYSTEM_CAPABILITIES: Capability[] = [
  // ============================================
  // CONTENT MANAGEMENT CAPABILITIES
  // ============================================
  {
    id: "cap-project-full",
    name: "Full Project Access",
    slug: "project.full",
    description: "Create, view, edit, delete, and manage all projects",
    category: "content_management",
    permissions: [
      { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "CREATE", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "EDIT", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "DELETE", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "MANAGE", effect: "ALLOW" },
    ],
    availableInPlans: ["FREE", "STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-project-readonly",
    name: "View Projects",
    slug: "project.readonly",
    description: "View projects only (no create, edit, or delete)",
    category: "content_management",
    permissions: [
      { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "CREATE", effect: "DENY" },
      { resourceType: "PROJECT", action: "EDIT", effect: "DENY" },
      { resourceType: "PROJECT", action: "DELETE", effect: "DENY" },
    ],
    availableInPlans: ["FREE", "STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-blueprint-full",
    name: "Full Blueprint Access",
    slug: "blueprint.full",
    description: "Create, view, edit, delete, and manage all blueprints",
    category: "content_management",
    permissions: [
      { resourceType: "BLUEPRINT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "CREATE", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "EDIT", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "DELETE", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "MANAGE", effect: "ALLOW" },
    ],
    availableInPlans: ["FREE", "STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-ai-generation",
    name: "AI Generation",
    slug: "ai.generate",
    description: "Use AI generation features within credit quota",
    category: "ai_features",
    permissions: [
      { resourceType: "AI_GENERATION", action: "VIEW", effect: "ALLOW" },
      { resourceType: "AI_GENERATION", action: "EXECUTE", effect: "ALLOW" },
    ],
    availableInPlans: ["STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-export",
    name: "Export Content",
    slug: "content.export",
    description: "Export projects, blueprints, and data",
    category: "content_management",
    permissions: [
      { resourceType: "EXPORT", action: "EXECUTE", effect: "ALLOW" },
    ],
    availableInPlans: ["STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-import",
    name: "Import Content",
    slug: "content.import",
    description: "Import projects, blueprints, and data",
    category: "content_management",
    permissions: [
      { resourceType: "IMPORT", action: "EXECUTE", effect: "ALLOW" },
    ],
    availableInPlans: ["STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-collaboration",
    name: "Collaboration",
    slug: "collaboration.full",
    description: "Real-time collaboration, comments, and sharing",
    category: "content_management",
    permissions: [
      { resourceType: "COLLABORATION", action: "VIEW", effect: "ALLOW" },
      { resourceType: "COLLABORATION", action: "EXECUTE", effect: "ALLOW" },
    ],
    availableInPlans: ["STARTER", "PRO", "ENTERPRISE"],
  },

  // ============================================
  // TEAM MANAGEMENT CAPABILITIES
  // ============================================
  {
    id: "cap-user-management-full",
    name: "Full User Management",
    slug: "users.full",
    description: "Create, view, edit, delete, and manage all users",
    category: "team_management",
    permissions: [
      { resourceType: "USER_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "USER_MANAGEMENT", action: "CREATE", effect: "ALLOW" },
      { resourceType: "USER_MANAGEMENT", action: "EDIT", effect: "ALLOW" },
      { resourceType: "USER_MANAGEMENT", action: "DELETE", effect: "ALLOW" },
      { resourceType: "USER_MANAGEMENT", action: "MANAGE", effect: "ALLOW" },
    ],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },
  {
    id: "cap-user-view",
    name: "View Users",
    slug: "users.view",
    description: "View team members and their profiles",
    category: "team_management",
    permissions: [
      { resourceType: "USER_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
    ],
    availableInPlans: ["FREE", "STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-invite-users",
    name: "Invite Users",
    slug: "users.invite",
    description: "Send invitations to new team members",
    category: "team_management",
    permissions: [
      { resourceType: "USER_MANAGEMENT", action: "CREATE", effect: "ALLOW",
        conditions: { method: "invite_only" }
      },
    ],
    availableInPlans: ["STARTER", "PRO", "ENTERPRISE"],
  },

  // ============================================
  // ORGANIZATION ADMIN CAPABILITIES
  // ============================================
  {
    id: "cap-org-settings",
    name: "Organization Settings",
    slug: "org.settings",
    description: "View and edit organization settings",
    category: "organization_admin",
    permissions: [
      { resourceType: "ORGANIZATION_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "ORGANIZATION_MANAGEMENT", action: "EDIT", effect: "ALLOW" },
      { resourceType: "ORGANIZATION_MANAGEMENT", action: "CONFIGURE", effect: "ALLOW" },
    ],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },
  {
    id: "cap-org-branding",
    name: "Organization Branding",
    slug: "org.branding",
    description: "Customize organization branding (logo, colors, domain)",
    category: "organization_admin",
    permissions: [
      { resourceType: "ORGANIZATION_MANAGEMENT", action: "EDIT", effect: "ALLOW",
        conditions: { scope: "branding_only" }
      },
    ],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },

  // ============================================
  // BILLING CAPABILITIES
  // ============================================
  {
    id: "cap-billing-full",
    name: "Full Billing Access",
    slug: "billing.full",
    description: "View and manage billing, subscriptions, and invoices",
    category: "billing_management",
    permissions: [
      { resourceType: "BILLING", action: "VIEW", effect: "ALLOW" },
      { resourceType: "BILLING", action: "MANAGE", effect: "ALLOW" },
      { resourceType: "BILLING", action: "CONFIGURE", effect: "ALLOW" },
    ],
    availableInPlans: ["STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-billing-view",
    name: "View Billing",
    slug: "billing.view",
    description: "View billing information and invoices (no changes)",
    category: "billing_management",
    permissions: [
      { resourceType: "BILLING", action: "VIEW", effect: "ALLOW" },
    ],
    availableInPlans: ["FREE", "STARTER", "PRO", "ENTERPRISE"],
  },

  // ============================================
  // SYSTEM ADMIN CAPABILITIES
  // ============================================
  {
    id: "cap-system-config",
    name: "System Configuration",
    slug: "system.config",
    description: "Access and modify system-level configuration",
    category: "system_admin",
    permissions: [
      { resourceType: "SYSTEM_CONFIG", action: "VIEW", effect: "ALLOW" },
      { resourceType: "SYSTEM_CONFIG", action: "CONFIGURE", effect: "ALLOW" },
    ],
    availableInPlans: ["ENTERPRISE"],
  },
  {
    id: "cap-rbac-management",
    name: "RBAC Management",
    slug: "rbac.manage",
    description: "Create and manage roles, permissions, and capabilities",
    category: "system_admin",
    permissions: [
      { resourceType: "USER_MANAGEMENT", action: "MANAGE", effect: "ALLOW",
        conditions: { scope: "rbac_only" }
      },
    ],
    requires: ["cap-user-view"],          // Must be able to view users
    availableInPlans: ["PRO", "ENTERPRISE"],
  },

  // ============================================
  // SECURITY CAPABILITIES
  // ============================================
  {
    id: "cap-audit-view",
    name: "View Audit Logs",
    slug: "audit.view",
    description: "View audit logs and access decisions",
    category: "security",
    permissions: [
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
        conditions: { scope: "audit_only" }
      },
    ],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },
  {
    id: "cap-audit-export",
    name: "Export Audit Logs",
    slug: "audit.export",
    description: "Export audit logs for compliance",
    category: "security",
    permissions: [
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
        conditions: { scope: "audit_only" }
      },
      { resourceType: "EXPORT", action: "EXECUTE", effect: "ALLOW",
        conditions: { scope: "audit_only" }
      },
    ],
    requires: ["cap-audit-view"],
    availableInPlans: ["ENTERPRISE"],
  },
  {
    id: "cap-impersonation",
    name: "User Impersonation",
    slug: "security.impersonate",
    description: "Impersonate other users for debugging (audit logged)",
    category: "security",
    permissions: [
      { resourceType: "USER_MANAGEMENT", action: "MANAGE", effect: "ALLOW",
        conditions: { scope: "impersonation_only" }
      },
    ],
    availableInPlans: ["ENTERPRISE"],
  },

  // ============================================
  // INTEGRATION CAPABILITIES
  // ============================================
  {
    id: "cap-api-access",
    name: "API Access",
    slug: "integration.api",
    description: "Access REST and GraphQL APIs",
    category: "integration",
    permissions: [
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
        conditions: { scope: "api_only" }
      },
      { resourceType: "SETTINGS", action: "CONFIGURE", effect: "ALLOW",
        conditions: { scope: "api_keys_only" }
      },
    ],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },
  {
    id: "cap-webhook-management",
    name: "Webhook Management",
    slug: "integration.webhooks",
    description: "Create and manage webhooks",
    category: "integration",
    permissions: [
      { resourceType: "SETTINGS", action: "CONFIGURE", effect: "ALLOW",
        conditions: { scope: "webhooks_only" }
      },
    ],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },
  {
    id: "cap-mcp-management",
    name: "MCP Configuration",
    slug: "integration.mcp",
    description: "Configure MCP (Model Context Protocol) servers",
    category: "integration",
    permissions: [
      { resourceType: "SETTINGS", action: "CONFIGURE", effect: "ALLOW",
        conditions: { scope: "mcp_only" }
      },
    ],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },

  // ============================================
  // ANALYTICS CAPABILITIES
  // ============================================
  {
    id: "cap-analytics-view",
    name: "View Analytics",
    slug: "analytics.view",
    description: "View dashboards and reports",
    category: "analytics",
    permissions: [
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
        conditions: { scope: "analytics_only" }
      },
    ],
    availableInPlans: ["STARTER", "PRO", "ENTERPRISE"],
  },
  {
    id: "cap-analytics-export",
    name: "Export Analytics",
    slug: "analytics.export",
    description: "Export analytics data and reports",
    category: "analytics",
    permissions: [
      { resourceType: "EXPORT", action: "EXECUTE", effect: "ALLOW",
        conditions: { scope: "analytics_only" }
      },
    ],
    requires: ["cap-analytics-view"],
    availableInPlans: ["PRO", "ENTERPRISE"],
  },
]
```

### Custom Capability Creation

```typescript
// ============================================
// SCENARIO: Admin Creates Custom Capability
// ============================================
// Route: POST /api/admin/rbac/capabilities
{
  name: "Blueprint Reviewer",
  slug: "blueprint.reviewer",
  description: "Can view and comment on blueprints but not edit",
  category: "custom",
  permissions: [
    { resourceType: "BLUEPRINT", action: "VIEW", effect: "ALLOW" },
    { resourceType: "COLLABORATION", action: "EXECUTE", effect: "ALLOW",
      conditions: { actions: ["comment", "react"], resourceTypes: ["BLUEPRINT"] }
    },
    { resourceType: "BLUEPRINT", action: "EDIT", effect: "DENY" },
    { resourceType: "BLUEPRINT", action: "DELETE", effect: "DENY" },
  ],
  visibleToRoles: ["OWNER", "ADMIN"],
  assignableByRoles: ["OWNER", "ADMIN"],
}

// ============================================
// SCENARIO: Assign Capability to Role
// ============================================
// Route: POST /api/admin/rbac/roles/{roleId}/capabilities
{
  capabilityIds: ["cap-project-full", "cap-blueprint-reviewer", "cap-analytics-view"]
}

// ============================================
// SCENARIO: Assign Capability Directly to User
// ============================================
// Route: POST /api/admin/rbac/users/{userId}/capabilities
{
  capabilityIds: ["cap-export"],
  reason: "Needs export access for quarterly report",
  expiresAt: "2026-03-01T00:00:00Z"      // Temporary capability
}
```

---

## Permission Resolution Engine

### Resolution Priority Order

```typescript
interface PermissionResolutionEngine {
  // Resolution order (highest to lowest priority):
  // 1. Explicit DENY on user (always wins)
  // 2. Explicit ALLOW on user
  // 3. Custom role explicit DENY
  // 4. Custom role explicit ALLOW
  // 5. Custom role inherited DENY (from parent custom role)
  // 6. Custom role inherited ALLOW (from parent custom role)
  // 7. System role DENY
  // 8. System role ALLOW
  // 9. Organization default DENY
  // 10. Organization default ALLOW
  // 11. System default (DENY - deny by default)

  resolutionSteps: [
    {
      step: 1,
      name: "Check Direct User Permissions",
      description: "Permissions assigned directly to the user override everything",
      priority: "highest",
      source: "Permission.userId = user.id"
    },
    {
      step: 2,
      name: "Check Custom Role Permissions",
      description: "Permissions from user's custom role",
      priority: "high",
      source: "Permission.customRoleId = user.customRoleId"
    },
    {
      step: 3,
      name: "Check Custom Role Inheritance Chain",
      description: "Walk up the custom role inheritance chain",
      priority: "medium-high",
      source: "CustomRole.parentCustomRoleId chain"
    },
    {
      step: 4,
      name: "Check System Role Permissions",
      description: "Permissions from user's system role (OWNER/ADMIN/MEMBER/VIEWER)",
      priority: "medium",
      source: "Permission.role = user.role"
    },
    {
      step: 5,
      name: "Check Organization Defaults",
      description: "Organization-wide default permissions",
      priority: "low",
      source: "Organization.settings.defaultPermissions"
    },
    {
      step: 6,
      name: "Apply System Default",
      description: "If no permission found, deny by default",
      priority: "lowest",
      result: "DENY"
    }
  ]
}
```

### Permission Check Implementation

```typescript
interface PermissionCheck {
  // Input
  userId: string
  organizationId: string
  resourceType: ResourceType
  action: ActionType
  resourceId?: string                     // Specific resource
  context?: {                             // For ABAC conditions
    ip: string
    userAgent: string
    timestamp: Date
    geoLocation?: { country: string; region: string }
    deviceId?: string
    sessionAge?: number                   // Minutes
    requestCount?: number                 // Rate limiting context
  }

  // Output
  result: {
    allowed: boolean
    effect: "ALLOW" | "DENY"
    reason: string                        // Human-readable explanation
    source: string                        // Where the permission came from
    permissionId?: string                 // The specific permission that decided
    conditions?: any                      // Conditions that were evaluated
    resolvedAt: Date
    evaluationTimeMs: number              // Performance tracking
  }
}

// ============================================
// PERMISSION CHECK FUNCTION
// ============================================
async function checkPermission(
  check: PermissionCheck
): Promise<PermissionCheckResult> {
  const startTime = Date.now()

  // Step 0: Check cache
  const cacheKey = `perm:${check.userId}:${check.resourceType}:${check.action}:${check.resourceId || '*'}`
  const cached = await permissionCache.get(cacheKey)
  if (cached && !cached.expired) {
    return { ...cached, fromCache: true }
  }

  // Step 1: Load user context
  const user = await loadUserContext(check.userId)
  if (!user || !user.isActive) {
    return deny("User not found or inactive", "system")
  }

  // Step 2: Check direct user permissions (resource-specific first, then type-level)
  const directPerms = await getDirectUserPermissions(
    check.userId, check.organizationId, check.resourceType, check.action
  )

  // Check resource-specific permission first
  if (check.resourceId) {
    const specificPerm = directPerms.find(p => p.resourceId === check.resourceId)
    if (specificPerm) {
      if (await evaluateConditions(specificPerm, check.context)) {
        return resolve(specificPerm, "direct_user_specific")
      }
    }
  }

  // Check type-level permission
  const typePerm = directPerms.find(p => !p.resourceId)
  if (typePerm) {
    if (await evaluateConditions(typePerm, check.context)) {
      return resolve(typePerm, "direct_user_type")
    }
  }

  // Step 3: Check custom role permissions
  if (user.customRoleId) {
    const customRoleResult = await checkCustomRolePermissions(
      user.customRoleId, check
    )
    if (customRoleResult) return customRoleResult
  }

  // Step 4: Check system role permissions
  const systemRoleResult = await checkSystemRolePermissions(
    user.role, check
  )
  if (systemRoleResult) return systemRoleResult

  // Step 5: Check organization defaults
  const orgDefault = await checkOrganizationDefaults(
    check.organizationId, check
  )
  if (orgDefault) return orgDefault

  // Step 6: Default deny
  const result = deny(
    `No permission found for ${check.resourceType}:${check.action}`,
    "default_deny"
  )

  // Cache result
  await permissionCache.set(cacheKey, result, { ttl: 300 }) // 5 min cache

  // Log decision
  await logAccessDecision(check, result)

  result.evaluationTimeMs = Date.now() - startTime
  return result
}

// ============================================
// CONDITION EVALUATION
// ============================================
async function evaluateConditions(
  permission: Permission,
  context?: PermissionContext
): Promise<boolean> {
  if (!permission.conditions) return true // No conditions = always applies

  const conditions = permission.conditions as PermissionConditions

  // Time-based condition
  if (conditions.timeWindow) {
    const now = new Date()
    const { start, end, timezone, daysOfWeek } = conditions.timeWindow
    if (!isWithinTimeWindow(now, start, end, timezone, daysOfWeek)) {
      return false
    }
  }

  // IP-based condition
  if (conditions.ipRestriction && context?.ip) {
    if (!matchesIpRestriction(context.ip, conditions.ipRestriction)) {
      return false
    }
  }

  // Geo-based condition
  if (conditions.geoRestriction && context?.geoLocation) {
    if (!matchesGeoRestriction(context.geoLocation, conditions.geoRestriction)) {
      return false
    }
  }

  // Device-based condition
  if (conditions.deviceRestriction && context?.userAgent) {
    if (!matchesDeviceRestriction(context.userAgent, conditions.deviceRestriction)) {
      return false
    }
  }

  // Attribute-based condition (ABAC)
  if (conditions.attributes) {
    if (!matchesAttributes(context, conditions.attributes)) {
      return false
    }
  }

  // Rate limiting condition
  if (conditions.rateLimit && context?.requestCount) {
    if (context.requestCount >= conditions.rateLimit.maxRequests) {
      return false
    }
  }

  // Resource ownership condition
  if (conditions.ownerOnly) {
    // Must check if user owns the resource
    // This is handled at the API level, not here
    return true // Defer to API-level check
  }

  // Scope condition
  if (conditions.scope) {
    // Scope is evaluated at the API level
    return true // Defer to API-level check
  }

  return true
}
```

---

## Resource-Level Access Control

### Resource Permission Model

```typescript
interface ResourcePermission {
  // ============================================
  // RESOURCE-LEVEL PERMISSIONS
  // ============================================
  // Permissions can target:
  // 1. All resources of a type (resourceId = null)
  // 2. A specific resource (resourceId = "clx...")
  // 3. Resources matching a pattern (resourceId = "project:team-*")

  // Resource types and their actions
  resourceActions: {
    PROJECT: {
      VIEW: "Can see the project in lists and open it"
      CREATE: "Can create new projects"
      EDIT: "Can modify project settings and content"
      DELETE: "Can delete the project"
      MANAGE: "Can manage project members and settings"
      EXECUTE: "Can run project-specific actions (deploy, export)"
      CONFIGURE: "Can configure project integrations and webhooks"
    }

    BLUEPRINT: {
      VIEW: "Can see the blueprint"
      CREATE: "Can create new blueprints"
      EDIT: "Can modify blueprint nodes and connections"
      DELETE: "Can delete the blueprint"
      MANAGE: "Can manage blueprint versions and sharing"
      EXECUTE: "Can generate code from blueprint"
      CONFIGURE: "Can configure blueprint settings"
    }

    MINDMAP: {
      VIEW: "Can see the mindmap"
      CREATE: "Can create new mindmaps"
      EDIT: "Can modify mindmap nodes"
      DELETE: "Can delete the mindmap"
      MANAGE: "Can manage mindmap sharing"
      EXECUTE: "Can run AI generation on mindmap"
    }

    AI_GENERATION: {
      VIEW: "Can see AI generation history"
      EXECUTE: "Can trigger AI generation"
      CONFIGURE: "Can configure AI providers and models"
      MANAGE: "Can manage AI credits and quotas"
    }

    USER_MANAGEMENT: {
      VIEW: "Can see user list and profiles"
      CREATE: "Can create/invite new users"
      EDIT: "Can edit user profiles and roles"
      DELETE: "Can delete/deactivate users"
      MANAGE: "Can manage user roles and permissions"
    }

    ORGANIZATION_MANAGEMENT: {
      VIEW: "Can see organization settings"
      EDIT: "Can edit organization settings"
      DELETE: "Can delete the organization"
      CONFIGURE: "Can configure organization features"
      MANAGE: "Can manage organization billing and plan"
    }

    BILLING: {
      VIEW: "Can see billing information"
      MANAGE: "Can manage subscriptions and payments"
      CONFIGURE: "Can configure billing settings"
    }

    SETTINGS: {
      VIEW: "Can see settings"
      EDIT: "Can edit settings"
      CONFIGURE: "Can configure advanced settings"
    }

    SYSTEM_CONFIG: {
      VIEW: "Can see system configuration"
      CONFIGURE: "Can modify system configuration"
      MANAGE: "Can manage system maintenance"
    }

    EXPORT: {
      EXECUTE: "Can export data"
    }

    IMPORT: {
      EXECUTE: "Can import data"
    }

    COLLABORATION: {
      VIEW: "Can see collaboration features"
      EXECUTE: "Can use collaboration features (comment, share)"
    }

    // UI Element Resources
    BUTTON: {
      VIEW: "Can see the button"
      EXECUTE: "Can click the button"
    }

    LINK: {
      VIEW: "Can see the link"
      EXECUTE: "Can navigate via the link"
    }

    MENU_ITEM: {
      VIEW: "Can see the menu item"
      EXECUTE: "Can click the menu item"
    }

    TAB: {
      VIEW: "Can see the tab"
    }

    CARD: {
      VIEW: "Can see the card"
    }

    FORM_FIELD: {
      VIEW: "Can see the form field"
      EDIT: "Can edit the form field"
    }

    DIALOG: {
      VIEW: "Can see/open the dialog"
      EXECUTE: "Can submit the dialog"
    }

    SIDEBAR_SECTION: {
      VIEW: "Can see the sidebar section"
    }
  }
}
```

### Resource Ownership & Sharing

```typescript
interface ResourceOwnership {
  // ============================================
  // OWNERSHIP MODEL
  // ============================================
  
  // Every resource has an owner (creator)
  owner: {
    userId: string
    organizationId: string
    createdAt: Date
  }

  // Owner permissions (configurable by admin)
  ownerPermissions: {
    hasFullAccess: boolean                // Owner always has full access
    canShare: boolean                     // Owner can share with others
    canTransferOwnership: boolean         // Owner can transfer to another user
    canSetPermissions: boolean            // Owner can set per-user permissions
    canMakePublic: boolean                // Owner can make resource public within org
    canMakeExternal: boolean              // Owner can share outside org
  }

  // Sharing model
  sharing: {
    // Per-user sharing
    userShares: {
      userId: string
      permissions: ActionType[]           // ["VIEW", "EDIT"]
      grantedBy: string                   // Who shared
      grantedAt: Date
      expiresAt?: Date                    // Temporary share
    }[]

    // Per-role sharing
    roleShares: {
      role: UserRole | string             // System role or custom role ID
      permissions: ActionType[]
      grantedBy: string
      grantedAt: Date
    }[]

    // Organization-wide sharing
    orgShare: {
      enabled: boolean
      permissions: ActionType[]           // ["VIEW"] for org-wide read access
    }

    // Public sharing (link sharing)
    publicShare: {
      enabled: boolean
      token: string                       // Unique share token
      permissions: ActionType[]           // Usually ["VIEW"]
      expiresAt?: Date
      password?: string                   // Optional password protection
      maxViews?: number                   // View limit
      viewCount: number
    }
  }
}

// ============================================
// RESOURCE ACCESS CHECK SCENARIOS
// ============================================

// Scenario 1: User tries to view a project they don't own
async function canViewProject(userId: string, projectId: string): Promise<boolean> {
  // Check 1: Is user the owner?
  const project = await getProject(projectId)
  if (project.createdById === userId) return true

  // Check 2: Is project shared with user directly?
  const userShare = await getResourceShare(projectId, userId)
  if (userShare?.permissions.includes("VIEW")) return true

  // Check 3: Is project shared with user's role?
  const user = await getUser(userId)
  const roleShare = await getResourceRoleShare(projectId, user.role)
  if (roleShare?.permissions.includes("VIEW")) return true

  // Check 4: Is project shared org-wide?
  const orgShare = await getResourceOrgShare(projectId)
  if (orgShare?.enabled && orgShare.permissions.includes("VIEW")) return true

  // Check 5: Does user have global VIEW permission for projects?
  const globalPerm = await checkPermission({
    userId, organizationId: project.organizationId,
    resourceType: "PROJECT", action: "VIEW"
  })
  if (globalPerm.allowed) return true

  // Check 6: Does user have resource-specific permission?
  const specificPerm = await checkPermission({
    userId, organizationId: project.organizationId,
    resourceType: "PROJECT", action: "VIEW", resourceId: projectId
  })
  return specificPerm.allowed
}

// Scenario 2: User tries to delete a project
async function canDeleteProject(userId: string, projectId: string): Promise<boolean> {
  const project = await getProject(projectId)
  const user = await getUser(userId)

  // OWNER/ADMIN can always delete (if they have the permission)
  if (user.role === "OWNER" || user.role === "ADMIN") {
    return await checkPermission({
      userId, organizationId: project.organizationId,
      resourceType: "PROJECT", action: "DELETE"
    }).then(r => r.allowed)
  }

  // For MEMBER: can only delete own projects (if allowed)
  if (project.createdById !== userId) {
    return false // Cannot delete others' projects
  }

  return await checkPermission({
    userId, organizationId: project.organizationId,
    resourceType: "PROJECT", action: "DELETE",
    resourceId: projectId
  }).then(r => r.allowed)
}
```

---

## UI Element-Level RBAC

### UI Permission Model

```typescript
interface UIElementPermission {
  // ============================================
  // UI ELEMENT RBAC
  // ============================================
  // Control visibility and interactivity of individual UI elements

  elementId: string                       // "btn-create-project"
  elementType: "BUTTON" | "LINK" | "MENU_ITEM" | "TAB" | "CARD" | "FORM_FIELD" | "DIALOG" | "SIDEBAR_SECTION"
  
  // Visibility rules
  visibility: {
    // Who can see this element
    visibleToRoles: string[]              // ["OWNER", "ADMIN", "MEMBER"]
    visibleToCustomRoles: string[]        // Custom role IDs
    visibleToUsers: string[]              // Specific user IDs
    
    // Conditions for visibility
    visibleWhen: {
      userHasPermission: string           // "PROJECT:CREATE"
      userHasPlan: string[]               // ["PRO", "ENTERPRISE"]
      userHasCapability: string           // "cap-project-full"
      featureFlag: string                 // "feature-new-editor"
      condition: string                   // Custom expression
    }

    // What to show when not visible
    whenHidden: "remove" | "disable" | "blur" | "upgrade_prompt"
    hiddenMessage?: string                // "Upgrade to PRO to access this feature"
    hiddenRedirect?: string               // "/upgrade"
  }

  // Interactivity rules
  interactivity: {
    // Who can interact with this element
    enabledForRoles: string[]
    enabledForCustomRoles: string[]
    enabledForUsers: string[]

    // Conditions for interactivity
    enabledWhen: {
      userHasPermission: string
      userHasPlan: string[]
      resourceOwner: boolean              // Only enabled for resource owner
      condition: string
    }

    // What to show when disabled
    whenDisabled: "gray_out" | "tooltip" | "click_redirect"
    disabledMessage?: string
    disabledRedirect?: string
  }

  // Dynamic properties (controlled by RBAC)
  dynamicProperties: {
    text?: Record<string, string>         // { "ADMIN": "Manage Users", "MEMBER": "View Team" }
    icon?: Record<string, string>         // Different icons per role
    color?: Record<string, string>        // Different colors per role
    href?: Record<string, string>         // Different links per role
    className?: Record<string, string>    // Different styles per role
  }
}
```

### UI RBAC Implementation

```typescript
// ============================================
// REACT COMPONENT: Permission-Aware Button
// ============================================
interface PermissionButtonProps {
  permission: string                      // "PROJECT:CREATE"
  children: React.ReactNode
  fallback?: "hidden" | "disabled" | "upgrade"
  upgradeMessage?: string
  onClick?: () => void
}

function PermissionButton({
  permission,
  children,
  fallback = "hidden",
  upgradeMessage,
  onClick
}: PermissionButtonProps) {
  const { hasPermission, isLoading } = usePermission(permission)
  const { plan } = useOrganization()

  if (isLoading) return <ButtonSkeleton />

  if (!hasPermission) {
    switch (fallback) {
      case "hidden":
        return null
      case "disabled":
        return (
          <Button disabled title="You don't have permission">
            {children}
          </Button>
        )
      case "upgrade":
        return (
          <Button variant="outline" onClick={() => router.push("/upgrade")}>
            <LockIcon className="mr-2 h-4 w-4" />
            {upgradeMessage || "Upgrade to access"}
          </Button>
        )
    }
  }

  return <Button onClick={onClick}>{children}</Button>
}

// ============================================
// REACT HOOK: usePermission
// ============================================
function usePermission(permission: string): {
  hasPermission: boolean
  isLoading: boolean
  source: string
  reason: string
} {
  const { user } = useSession()
  const { permissions } = useRBACContext()

  const [resourceType, action] = permission.split(":")

  // Check cached permissions
  const result = useMemo(() => {
    if (!permissions) return { hasPermission: false, isLoading: true }

    return checkLocalPermission(permissions, resourceType, action, user)
  }, [permissions, resourceType, action, user])

  return result
}

// ============================================
// REACT HOOK: useRole
// ============================================
function useRole(): {
  role: UserRole
  customRole: CustomRole | null
  isOwner: boolean
  isAdmin: boolean
  isMember: boolean
  isViewer: boolean
  hasCustomRole: boolean
  effectiveLevel: number
} {
  const { user } = useSession()
  const { customRoles } = useRBACContext()

  return useMemo(() => ({
    role: user.role,
    customRole: user.customRoleId
      ? customRoles.find(r => r.id === user.customRoleId) || null
      : null,
    isOwner: user.role === "OWNER",
    isAdmin: user.role === "ADMIN",
    isMember: user.role === "MEMBER",
    isViewer: user.role === "VIEWER",
    hasCustomRole: !!user.customRoleId,
    effectiveLevel: getRoleLevel(user.role),
  }), [user, customRoles])
}

// ============================================
// REACT HOOK: useCapability
// ============================================
function useCapability(capabilitySlug: string): {
  hasCapability: boolean
  isLoading: boolean
} {
  const { capabilities } = useRBACContext()

  return useMemo(() => ({
    hasCapability: capabilities.some(c => c.slug === capabilitySlug),
    isLoading: !capabilities,
  }), [capabilities, capabilitySlug])
}

// ============================================
// REACT COMPONENT: Role-Based Rendering
// ============================================
function RoleGate({
  allowedRoles,
  deniedRoles,
  children,
  fallback
}: {
  allowedRoles?: string[]
  deniedRoles?: string[]
  children: React.ReactNode
  fallback?: React.ReactNode
}) {
  const { role, customRole } = useRole()

  const isAllowed = useMemo(() => {
    if (deniedRoles?.includes(role)) return false
    if (deniedRoles && customRole && deniedRoles.includes(customRole.id)) return false

    if (allowedRoles) {
      return allowedRoles.includes(role) ||
        (customRole && allowedRoles.includes(customRole.id))
    }

    return true
  }, [role, customRole, allowedRoles, deniedRoles])

  if (!isAllowed) return fallback || null
  return <>{children}</>
}

// ============================================
// USAGE EXAMPLES
// ============================================

// Example 1: Admin-only sidebar section
<RoleGate allowedRoles={["OWNER", "ADMIN"]}>
  <SidebarSection title="Administration">
    <SidebarItem href="/admin/users" icon={UsersIcon}>Users</SidebarItem>
    <SidebarItem href="/admin/rbac" icon={ShieldIcon}>Roles & Permissions</SidebarItem>
    <SidebarItem href="/admin/settings" icon={SettingsIcon}>Settings</SidebarItem>
  </SidebarSection>
</RoleGate>

// Example 2: Permission-based button
<PermissionButton
  permission="PROJECT:CREATE"
  fallback="upgrade"
  upgradeMessage="Upgrade to create projects"
  onClick={() => router.push("/projects/new")}
>
  <PlusIcon className="mr-2 h-4 w-4" />
  New Project
</PermissionButton>

// Example 3: Capability-based feature
function AIGenerationPanel() {
  const { hasCapability } = useCapability("ai.generate")

  if (!hasCapability) {
    return <UpgradePrompt feature="AI Generation" plan="STARTER" />
  }

  return <AIGenerationForm />
}

// Example 4: Dynamic navigation based on role
function DashboardNavigation() {
  const { isOwner, isAdmin, isMember, isViewer } = useRole()
  const { hasPermission } = usePermission("BILLING:VIEW")

  return (
    <nav>
      <NavItem href="/dashboard">Dashboard</NavItem>
      <NavItem href="/projects">Projects</NavItem>
      <NavItem href="/blueprints">Blueprints</NavItem>

      {(isOwner || isAdmin) && (
        <>
          <NavItem href="/admin">Admin Panel</NavItem>
          <NavItem href="/admin/users">User Management</NavItem>
          <NavItem href="/admin/rbac">RBAC</NavItem>
        </>
      )}

      {hasPermission && <NavItem href="/billing">Billing</NavItem>}

      {!isViewer && (
        <NavItem href="/settings">Settings</NavItem>
      )}
    </nav>
  )
}

// Example 5: Form field visibility based on role
function ProjectForm() {
  const { isOwner, isAdmin } = useRole()
  const { hasPermission } = usePermission("PROJECT:CONFIGURE")

  return (
    <form>
      <Input name="name" label="Project Name" required />
      <Textarea name="description" label="Description" />

      {/* Only admins can set project visibility */}
      <RoleGate allowedRoles={["OWNER", "ADMIN"]}>
        <Select name="visibility" label="Visibility">
          <option value="private">Private</option>
          <option value="team">Team</option>
          <option value="organization">Organization</option>
        </Select>
      </RoleGate>

      {/* Only users with CONFIGURE permission see advanced settings */}
      {hasPermission && (
        <Accordion title="Advanced Settings">
          <Input name="webhook" label="Webhook URL" />
          <Select name="aiModel" label="AI Model" />
        </Accordion>
      )}

      {/* Owner-only danger zone */}
      {isOwner && (
        <DangerZone>
          <Button variant="destructive">Delete Project</Button>
        </DangerZone>
      )}
    </form>
  )
}
```

---

## Attribute-Based Access Control (ABAC)

### ABAC Configuration

```typescript
interface ABACConfiguration {
  // ============================================
  // ATTRIBUTE-BASED ACCESS CONTROL
  // ============================================
  // ABAC extends RBAC with dynamic conditions based on attributes

  enabled: boolean
  mode: "standalone" | "rbac_extension"   // Use alone or extend RBAC

  // ============================================
  // ATTRIBUTE SOURCES
  // ============================================
  attributeSources: {
    // User attributes
    user: {
      role: string
      customRoleId: string
      department: string
      location: string
      clearanceLevel: number
      joinDate: Date
      lastLoginAt: Date
      mfaEnabled: boolean
      emailVerified: boolean
      plan: string
      customAttributes: Record<string, any>
    }

    // Resource attributes
    resource: {
      type: string
      owner: string
      createdAt: Date
      sensitivity: "public" | "internal" | "confidential" | "restricted"
      department: string
      tags: string[]
      status: string
      customAttributes: Record<string, any>
    }

    // Environment attributes
    environment: {
      currentTime: Date
      dayOfWeek: number
      ipAddress: string
      geoLocation: { country: string; region: string; city: string }
      deviceType: string
      browser: string
      isVPN: boolean
      isTor: boolean
      sessionAge: number
      requestsInWindow: number
    }

    // Action attributes
    action: {
      type: string
      isBulk: boolean
      affectsOthers: boolean
      isDestructive: boolean
      requiresApproval: boolean
    }
  }

  // ============================================
  // ABAC POLICIES
  // ============================================
  policies: ABACPolicy[]
}

interface ABACPolicy {
  id: string
  name: string                            // "Restrict Confidential Access to Verified Users"
  description: string
  priority: number                        // Higher = evaluated first
  enabled: boolean

  // Target: What this policy applies to
  target: {
    resourceTypes: ResourceType[]         // Which resource types
    actions: ActionType[]                 // Which actions
    roles?: string[]                      // Which roles (optional filter)
  }

  // Condition: When this policy applies
  condition: {
    // All conditions must be true (AND logic)
    all?: ABACCondition[]
    // Any condition must be true (OR logic)
    any?: ABACCondition[]
    // No condition must be true (NOT logic)
    none?: ABACCondition[]
  }

  // Effect: What happens when condition is met
  effect: "ALLOW" | "DENY"
  
  // Obligation: Additional actions when policy applies
  obligations?: {
    logAccess: boolean
    notifyAdmin: boolean
    requireMFA: boolean
    requireApproval: boolean
    addWatermark: boolean                 // For exports
    limitFields: string[]                 // Only show certain fields
    redactFields: string[]                // Redact sensitive fields
  }
}

interface ABACCondition {
  attribute: string                       // "user.department"
  operator: "equals" | "not_equals" | "contains" | "not_contains" |
            "greater_than" | "less_than" | "in" | "not_in" |
            "matches" | "starts_with" | "ends_with" |
            "between" | "before" | "after" | "is_null" | "is_not_null"
  value: any                              // The value to compare against
}

// ============================================
// ABAC POLICY EXAMPLES
// ============================================
const EXAMPLE_POLICIES: ABACPolicy[] = [
  // Policy 1: Only verified users can access confidential resources
  {
    id: "policy-confidential-access",
    name: "Confidential Resource Access",
    description: "Only email-verified users with MFA can access confidential resources",
    priority: 100,
    enabled: true,
    target: {
      resourceTypes: ["PROJECT", "BLUEPRINT", "MINDMAP"],
      actions: ["VIEW", "EDIT", "DELETE"],
    },
    condition: {
      all: [
        { attribute: "resource.sensitivity", operator: "equals", value: "confidential" },
      ],
      none: [
        { attribute: "user.emailVerified", operator: "is_null", value: null },
        { attribute: "user.mfaEnabled", operator: "equals", value: false },
      ]
    },
    effect: "DENY",                       // Deny if user is NOT verified + MFA
    obligations: {
      logAccess: true,
      notifyAdmin: true,
    }
  },

  // Policy 2: Time-based access restriction
  {
    id: "policy-business-hours",
    name: "Business Hours Only",
    description: "Restrict access to business hours for certain roles",
    priority: 50,
    enabled: true,
    target: {
      resourceTypes: ["BILLING", "SYSTEM_CONFIG"],
      actions: ["EDIT", "DELETE", "CONFIGURE"],
      roles: ["ADMIN"],                   // Only applies to ADMIN role
    },
    condition: {
      any: [
        { attribute: "environment.dayOfWeek", operator: "in", value: [0, 6] },  // Weekend
        { attribute: "environment.currentTime", operator: "before", value: "09:00" },
        { attribute: "environment.currentTime", operator: "after", value: "18:00" },
      ]
    },
    effect: "DENY",
    obligations: {
      logAccess: true,
      notifyAdmin: true,
    }
  },

  // Policy 3: Geo-restriction
  {
    id: "policy-geo-restriction",
    name: "Geographic Restriction",
    description: "Block access from restricted countries",
    priority: 200,
    enabled: true,
    target: {
      resourceTypes: ["PROJECT", "BLUEPRINT", "EXPORT"],
      actions: ["VIEW", "EDIT", "EXECUTE"],
    },
    condition: {
      any: [
        { attribute: "environment.geoLocation.country", operator: "in",
          value: ["BLOCKED_COUNTRY_1", "BLOCKED_COUNTRY_2"] },
        { attribute: "environment.isVPN", operator: "equals", value: true },
        { attribute: "environment.isTor", operator: "equals", value: true },
      ]
    },
    effect: "DENY",
    obligations: {
      logAccess: true,
      notifyAdmin: true,
    }
  },

  // Policy 4: Rate limiting for AI generation
  {
    id: "policy-ai-rate-limit",
    name: "AI Generation Rate Limit",
    description: "Limit AI generation requests per hour",
    priority: 75,
    enabled: true,
    target: {
      resourceTypes: ["AI_GENERATION"],
      actions: ["EXECUTE"],
    },
    condition: {
      all: [
        { attribute: "environment.requestsInWindow", operator: "greater_than", value: 50 },
      ]
    },
    effect: "DENY",
    obligations: {
      logAccess: true,
    }
  },

  // Policy 5: New user restriction
  {
    id: "policy-new-user-restriction",
    name: "New User Restriction",
    description: "Restrict destructive actions for users who joined less than 7 days ago",
    priority: 60,
    enabled: true,
    target: {
      resourceTypes: ["PROJECT", "BLUEPRINT"],
      actions: ["DELETE"],
    },
    condition: {
      all: [
        { attribute: "user.joinDate", operator: "after",
          value: "NOW-7d" },                // Joined less than 7 days ago
      ]
    },
    effect: "DENY",
    obligations: {
      logAccess: true,
    }
  },

  // Policy 6: Resource owner bypass
  {
    id: "policy-owner-bypass",
    name: "Resource Owner Full Access",
    description: "Resource owners always have full access to their own resources",
    priority: 300,                        // High priority
    enabled: true,
    target: {
      resourceTypes: ["PROJECT", "BLUEPRINT", "MINDMAP"],
      actions: ["VIEW", "EDIT", "DELETE", "MANAGE", "CONFIGURE"],
    },
    condition: {
      all: [
        { attribute: "resource.owner", operator: "equals", value: "$user.id" },
      ]
    },
    effect: "ALLOW",
  },
]
```

---

## Multi-Tenant RBAC

### Tenant Isolation

```typescript
interface MultiTenantRBAC {
  // ============================================
  // TENANT ISOLATION
  // ============================================
  
  // Every RBAC operation is scoped to an organization
  tenantIsolation: {
    // Permissions are organization-scoped
    permissionScope: "organization"       // Cannot cross organization boundaries
    
    // Custom roles are organization-scoped
    roleScope: "organization"             // Each org has its own custom roles
    
    // System roles are global but permissions differ per org
    systemRoleScope: "global_definition_local_permissions"
    
    // Capabilities can be system-wide or org-specific
    capabilityScope: "hybrid"             // System + org-specific
  }

  // ============================================
  // CROSS-TENANT SCENARIOS
  // ============================================
  crossTenant: {
    // User belongs to multiple organizations
    multiOrgMembership: {
      enabled: boolean
      maxOrganizations: number            // Per user
      rolePerOrganization: boolean        // Different role in each org
      switchOrganizationUI: boolean       // UI to switch between orgs
      defaultOrganization: string         // Default org on login
    }

    // Shared resources across organizations
    crossOrgSharing: {
      enabled: boolean
      requireBothOrgApproval: boolean     // Both orgs must approve
      allowedResourceTypes: ResourceType[]
      maxSharedResources: number
    }

    // Platform-level admin (super admin across all orgs)
    platformAdmin: {
      enabled: boolean
      role: "PLATFORM_ADMIN"              // Special role above OWNER
      canAccessAllOrgs: boolean
      canImpersonateAnyUser: boolean
      canModifyAnyOrg: boolean
      requiresSeparateAuth: boolean       // Separate login for platform admin
      ipRestriction: string[]             // Platform admin IP whitelist
    }
  }

  // ============================================
  // ORGANIZATION-LEVEL RBAC SETTINGS
  // ============================================
  orgSettings: {
    // Each organization can customize these
    customRolesEnabled: boolean           // Can this org create custom roles
    maxCustomRoles: number                // Based on plan
    abacEnabled: boolean                  // Can this org use ABAC
    uiRbacEnabled: boolean                // Can this org use UI-level RBAC
    delegationEnabled: boolean            // Can this org use delegation
    impersonationEnabled: boolean         // Can this org use impersonation
    
    // Default permissions for new users in this org
    defaultNewUserPermissions: Permission[]
    
    // Organization-wide permission overrides
    orgWideOverrides: {
      // Force certain permissions for all users
      forceAllow: Permission[]            // Always allowed for everyone
      forceDeny: Permission[]             // Always denied for everyone
    }
  }
}

// ============================================
// MULTI-ORG USER SCENARIO
// ============================================

// User "john@example.com" belongs to:
// - Org A: Role = ADMIN, Custom Role = "Project Manager"
// - Org B: Role = MEMBER, Custom Role = null
// - Org C: Role = VIEWER, Custom Role = "Content Reviewer"

// When John logs in:
// 1. Show organization switcher
// 2. Load permissions for selected organization
// 3. UI renders based on that org's permissions
// 4. API calls are scoped to that org

// When John switches from Org A to Org B:
// 1. Clear permission cache
// 2. Load Org B permissions
// 3. Re-render UI (admin panel disappears, create buttons may change)
// 4. API calls now scoped to Org B
```

---

## Admin Configuration Interface

### RBAC Admin Dashboard

```typescript
interface RBACAdminDashboard {
  // ============================================
  // ADMIN RBAC MANAGEMENT PAGES
  // ============================================

  // Route: /admin/rbac
  mainPage: {
    tabs: [
      {
        id: "overview",
        label: "Overview",
        icon: "LayoutDashboard",
        content: {
          // Statistics cards
          stats: {
            totalRoles: number            // System + custom roles
            totalPermissions: number
            totalCapabilities: number
            activeUsers: number
            recentChanges: number          // Last 24h
            conflictsDetected: number
          }

          // Quick actions
          quickActions: [
            "Create Custom Role",
            "Create Permission",
            "Create Capability",
            "Import Permissions",
            "Export Configuration",
            "Run Audit",
          ]

          // Recent activity feed
          recentActivity: {
            action: string                // "Role Created", "Permission Updated"
            performedBy: string
            target: string
            timestamp: Date
          }[]

          // Health indicators
          health: {
            orphanedPermissions: number   // Permissions with no role/user
            unusedRoles: number           // Roles with no users
            conflictingPermissions: number
            expiredTemporaryRoles: number
          }
        }
      },

      {
        id: "roles",
        label: "Roles",
        icon: "Shield",
        content: {
          // System roles (read-only structure, configurable permissions)
          systemRoles: {
            OWNER: { userCount: number, permissionCount: number, isConfigurable: false }
            ADMIN: { userCount: number, permissionCount: number, isConfigurable: true }
            MEMBER: { userCount: number, permissionCount: number, isConfigurable: true }
            VIEWER: { userCount: number, permissionCount: number, isConfigurable: true }
          }

          // Custom roles (full CRUD)
          customRoles: {
            list: CustomRole[]
            actions: ["create", "edit", "delete", "clone", "compare", "assign"]
            filters: ["baseRole", "status", "createdBy", "hasUsers"]
            sorting: ["name", "createdAt", "userCount", "permissionCount"]
            search: boolean
            pagination: boolean
            bulkActions: ["delete", "activate", "deactivate", "export"]
          }

          // Role creation wizard
          createWizard: {
            steps: [
              "Basic Info (name, description, icon, color)",
              "Base Role Selection",
              "Permission Assignment (matrix or capability-based)",
              "Restrictions (time, IP, device)",
              "Review & Create"
            ]
          }
        }
      },

      {
        id: "permissions",
        label: "Permissions",
        icon: "Key",
        content: {
          // Permission list
          list: {
            filters: ["resourceType", "action", "effect", "role", "customRole"]
            sorting: ["resourceType", "action", "createdAt"]
            search: boolean
            pagination: boolean
            bulkActions: ["delete", "update_effect", "export"]
          }

          // Permission creation
          create: {
            fields: [
              "resourceType (dropdown)",
              "resourceId (optional, autocomplete)",
              "action (dropdown)",
              "effect (ALLOW/DENY toggle)",
              "target (role/customRole/user selector)",
              "conditions (JSON editor or form builder)",
              "uiProperties (visual editor)",
              "description (text)"
            ]
          }
        }
      },

      {
        id: "matrix",
        label: "Permission Matrix",
        icon: "Grid3x3",
        content: {
          // Interactive permission matrix
          matrix: {
            rows: ResourceType[]          // Resource types
            columns: string[]             // Roles (system + custom)
            cells: {
              // Each cell shows the permission for that resource+role
              resourceType: ResourceType
              role: string
              actions: {
                action: ActionType
                effect: "ALLOW" | "DENY" | null
                isInherited: boolean
                hasConditions: boolean
                permissionId: string | null
              }[]
            }

            // Interaction
            clickToToggle: boolean        // Click cell to toggle ALLOW/DENY
            rightClickMenu: boolean       // Right-click for conditions, details
            dragToSelect: boolean         // Drag to select multiple cells
            bulkApply: boolean            // Apply same permission to selection

            // Visual
            colorCoding: {
              allow: "#22c55e"
              deny: "#ef4444"
              conditional: "#f59e0b"
              inherited: "#6366f1"
              notSet: "#e5e7eb"
            }
            showInheritanceIndicator: boolean
            showConditionIndicator: boolean
          }
        }
      },

      {
        id: "capabilities",
        label: "Capabilities",
        icon: "Puzzle",
        content: {
          // System capabilities (read-only)
          systemCapabilities: {
            categories: CapabilityCategory[]
            list: Capability[]
          }

          // Custom capabilities (CRUD)
          customCapabilities: {
            list: Capability[]
            actions: ["create", "edit", "delete", "assign"]
          }

          // Capability assignment
          assignment: {
            // Assign capabilities to roles
            roleAssignment: {
              role: string
              capabilities: string[]
            }[]

            // Assign capabilities to users
            userAssignment: {
              userId: string
              capabilities: string[]
              expiresAt?: Date
            }[]
          }
        }
      },

      {
        id: "policies",
        label: "ABAC Policies",
        icon: "FileCheck",
        content: {
          // ABAC policy management
          policies: {
            list: ABACPolicy[]
            actions: ["create", "edit", "delete", "enable", "disable", "test"]
            
            // Policy builder (visual)
            builder: {
              conditionBuilder: boolean   // Visual condition builder
              jsonEditor: boolean         // Raw JSON editor
              testSimulator: boolean      // Test policy against sample data
            }
          }
        }
      },

      {
        id: "templates",
        label: "Templates",
        icon: "FileTemplate",
        content: {
          // Permission templates
          systemTemplates: PermissionTemplate[]
          customTemplates: PermissionTemplate[]
          
          actions: ["create", "edit", "delete", "apply", "export", "import"]
          
          // Apply template wizard
          applyWizard: {
            steps: [
              "Select Template",
              "Select Target (role/user)",
              "Review Permissions",
              "Apply (merge or replace)"
            ]
          }
        }
      },

      {
        id: "audit",
        label: "Audit Log",
        icon: "ScrollText",
        content: {
          // RBAC audit log
          auditLog: {
            filters: [
              "action (created, updated, deleted, assigned, revoked)",
              "resourceType (role, permission, capability, policy)",
              "performedBy (user selector)",
              "dateRange",
              "organizationId"
            ]
            columns: [
              "timestamp",
              "action",
              "resourceType",
              "resourceId",
              "performedBy",
              "previousValue",
              "newValue",
              "reason"
            ]
            export: ["csv", "json", "pdf"]
            realTimeStream: boolean
          }

          // Access decision log
          accessDecisionLog: {
            filters: [
              "userId",
              "resourceType",
              "action",
              "result (allowed/denied)",
              "source (role/permission/policy)",
              "dateRange"
            ]
            columns: [
              "timestamp",
              "userId",
              "resourceType",
              "resourceId",
              "action",
              "result",
              "source",
              "reason",
              "evaluationTimeMs"
            ]
          }
        }
      },

      {
        id: "simulation",
        label: "Simulation",
        icon: "FlaskConical",
        content: {
          // Permission simulation tool
          simulation: {
            // "What can user X do?"
            userSimulation: {
              selectUser: boolean
              showEffectivePermissions: boolean
              showPermissionSources: boolean
              showDeniedPermissions: boolean
              showConflicts: boolean
            }

            // "What would happen if I change this permission?"
            changeSimulation: {
              selectPermissionChange: boolean
              showAffectedUsers: boolean
              showBeforeAfter: boolean
              dryRun: boolean             // Preview without applying
            }

            // "Who has access to resource X?"
            resourceSimulation: {
              selectResource: boolean
              showUsersWithAccess: boolean
              showAccessLevel: boolean    // VIEW, EDIT, DELETE, etc.
              showAccessSource: boolean   // Why they have access
            }

            // "Compare role A vs role B"
            roleComparison: {
              selectTwoRoles: boolean
              showDifferences: boolean
              showOverlap: boolean
              showUniquePermissions: boolean
            }
          }
        }
      }
    ]
  }
}
```

### Admin RBAC Configuration Scenarios

```typescript
// ============================================
// SCENARIO 1: Admin Sets Up RBAC for New Organization
// ============================================
// Step 1: Review default system roles
// Route: GET /api/admin/rbac/roles
// Shows: OWNER (1 user), ADMIN (0), MEMBER (0), VIEWER (0)

// Step 2: Customize MEMBER role permissions
// Route: POST /api/admin/rbac/matrix
{
  updates: [
    // Allow members to create projects
    { resourceType: "PROJECT", action: "CREATE", role: "MEMBER", effect: "ALLOW" },
    // Allow members to use AI (within quota)
    { resourceType: "AI_GENERATION", action: "EXECUTE", role: "MEMBER", effect: "ALLOW" },
    // Deny members from accessing billing
    { resourceType: "BILLING", action: "VIEW", role: "MEMBER", effect: "DENY" },
    // Deny members from system config
    { resourceType: "SYSTEM_CONFIG", action: "VIEW", role: "MEMBER", effect: "DENY" },
  ]
}

// Step 3: Create custom roles for the team
// Route: POST /api/admin/rbac/roles
{
  name: "Team Lead",
  description: "Can manage team members and projects",
  baseRole: "MEMBER",
  permissions: [
    { resourceType: "USER_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
    { resourceType: "USER_MANAGEMENT", action: "CREATE", effect: "ALLOW",
      conditions: { method: "invite_only", maxRole: "MEMBER" }
    },
    { resourceType: "PROJECT", action: "MANAGE", effect: "ALLOW" },
  ]
}

// Step 4: Apply permission template
// Route: POST /api/admin/rbac/templates/apply
{
  templateId: "template-standard-team",
  targetRole: "MEMBER",
  mode: "merge"                           // merge or replace
}

// ============================================
// SCENARIO 2: Admin Restricts Feature Access by Plan
// ============================================
// Route: POST /api/admin/rbac/matrix
{
  updates: [
    // FREE plan: no AI generation
    { resourceType: "AI_GENERATION", action: "EXECUTE", role: "MEMBER",
      effect: "DENY", conditions: { plan: "FREE" }
    },
    // FREE plan: no export
    { resourceType: "EXPORT", action: "EXECUTE", role: "MEMBER",
      effect: "DENY", conditions: { plan: "FREE" }
    },
    // STARTER plan: limited AI
    { resourceType: "AI_GENERATION", action: "EXECUTE", role: "MEMBER",
      effect: "ALLOW", conditions: { plan: "STARTER", maxPerDay: 10 }
    },
    // PRO plan: full AI
    { resourceType: "AI_GENERATION", action: "EXECUTE", role: "MEMBER",
      effect: "ALLOW", conditions: { plan: "PRO" }
    },
  ]
}

// ============================================
// SCENARIO 3: Admin Handles Security Incident
// ============================================
// Step 1: Immediately revoke all permissions for compromised user
// Route: POST /api/admin/rbac/users/{userId}/revoke-all
{
  reason: "Security incident - account compromised",
  notifyUser: true,
  lockAccount: true
}

// Step 2: Review what the user had access to
// Route: GET /api/admin/rbac/users/{userId}/effective-permissions
// Response: Full list of what user could access

// Step 3: Review access decision log for the user
// Route: GET /api/admin/rbac/audit/access-decisions?userId={userId}&last=24h
// Response: All access decisions for the user in last 24 hours

// Step 4: After investigation, restore with reduced permissions
// Route: POST /api/admin/rbac/users/{userId}/permissions
{
  permissions: [
    { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW" },
    // Only view access until investigation complete
  ],
  reason: "Restored with limited access pending investigation"
}

// ============================================
// SCENARIO 4: Admin Configures Temporary Elevated Access
// ============================================
// Developer needs ADMIN access for 2 hours to debug production issue
// Route: POST /api/admin/rbac/users/{userId}/elevate
{
  targetRole: "ADMIN",
  duration: 120,                          // Minutes
  reason: "Production debugging - ticket #1234",
  restrictions: {
    ipRestriction: ["office-ip"],
    logAllActions: true,
    notifyOnExpiry: true
  }
}
// Response:
{
  elevation: {
    id: "elev-clx...",
    userId: "user-id",
    previousRole: "MEMBER",
    elevatedRole: "ADMIN",
    expiresAt: "2026-02-15T17:26:00Z",
    reason: "Production debugging - ticket #1234",
    grantedBy: "admin-id"
  }
}

// Auto-reverts after 2 hours with audit log entry

// ============================================
// SCENARIO 5: Admin Bulk-Assigns Role to Department
// ============================================
// Route: POST /api/admin/rbac/roles/{roleId}/bulk-assign
{
  criteria: {
    department: "Engineering",
    // OR
    emailDomain: "@engineering.company.com",
    // OR
    userIds: ["user1", "user2", "user3"]
  },
  notifyUsers: true,
  reason: "Department restructuring"
}

// ============================================
// SCENARIO 6: Admin Exports RBAC Configuration
// ============================================
// Route: GET /api/admin/rbac/export?format=json
// Response: Complete RBAC configuration as JSON
{
  exportedAt: "2026-02-15T15:26:00Z",
  organization: "org-id",
  systemRoles: { ... },
  customRoles: [ ... ],
  permissions: [ ... ],
  capabilities: [ ... ],
  policies: [ ... ],
  templates: [ ... ]
}

// Route: POST /api/admin/rbac/import
// Import RBAC configuration (with conflict resolution)
{
  config: { ... },                        // Exported config
  mode: "merge" | "replace",
  conflictResolution: "keep_existing" | "use_imported" | "manual"
}
```

---

## User-Side Experience

### What Users See Based on Their Role

```typescript
interface UserRBACExperience {
  // ============================================
  // USER SEES THEIR OWN PERMISSIONS
  // ============================================
  
  // Route: GET /api/user/permissions
  // What the user can see about their own access
  selfService: {
    // My Role
    myRole: {
      systemRole: "MEMBER"
      customRole: "Project Manager" | null
      effectiveLevel: 2
      capabilities: ["project.full", "blueprint.full", "ai.generate"]
    }

    // My Permissions (simplified view)
    myPermissions: {
      canDo: [
        { action: "Create Projects", icon: "plus" },
        { action: "Edit Blueprints", icon: "edit" },
        { action: "Use AI Generation", icon: "sparkles" },
        { action: "Export Data", icon: "download" },
      ]
      cannotDo: [
        { action: "Manage Users", reason: "Requires Admin role", upgradeAction: "contact_admin" },
        { action: "Access Billing", reason: "Requires Billing Manager role" },
        { action: "System Configuration", reason: "Requires Owner role" },
      ]
    }

    // My Restrictions
    myRestrictions: {
      sessionTimeout: 60                  // Minutes
      mfaRequired: false
      ipRestriction: null
      timeRestriction: null
      maxProjects: 10
      maxAICredits: 100
    }
  }

  // ============================================
  // USER REQUESTS PERMISSION CHANGE
  // ============================================
  
  // Route: POST /api/user/permission-requests
  permissionRequest: {
    requestedPermission: string           // "BILLING:VIEW"
    reason: string                        // "Need to review invoices for expense report"
    duration?: number                     // Hours (for temporary access)
    urgency: "low" | "medium" | "high"
  }

  // Admin receives notification
  // Route: GET /api/admin/rbac/permission-requests
  // Admin can approve/deny with optional modifications

  // ============================================
  // USER SEES PERMISSION DENIED
  // ============================================
  
  // When user tries to access something they can't:
  deniedExperience: {
    // Option 1: Element is hidden (user never sees it)
    hidden: {
      // Admin sidebar items not shown to MEMBER
      // Delete button not shown to VIEWER
      // Billing tab not shown to non-billing roles
    }

    // Option 2: Element is disabled with explanation
    disabled: {
      message: "You don't have permission to delete this project"
      tooltip: "Contact your admin to request access"
      action: null
    }

    // Option 3: Upgrade prompt
    upgradePrompt: {
      message: "AI Generation requires a Starter plan or higher"
      currentPlan: "FREE"
      requiredPlan: "STARTER"
      upgradeUrl: "/settings/billing"
      ctaText: "Upgrade Now"
    }

    // Option 4: Request access
    requestAccess: {
      message: "You need the 'Export' capability to download this report"
      requestUrl: "/api/user/permission-requests"
      autoFillPermission: "EXPORT:EXECUTE"
    }

    // Option 5: Redirect
    redirect: {
      from: "/admin/users"
      to: "/dashboard"
      message: "You don't have access to the admin panel"
      flash: true                         // Show flash message after redirect
    }
  }

  // ============================================
  // USER NAVIGATION ADAPTS TO ROLE
  // ============================================
  
  // OWNER sees:
  ownerNavigation: [
    "Dashboard", "Projects", "Blueprints", "Team",
    "Admin Panel", "Users", "RBAC", "Billing", "Settings",
    "System Config", "Audit Logs"
  ]

  // ADMIN sees:
  adminNavigation: [
    "Dashboard", "Projects", "Blueprints", "Team",
    "Admin Panel", "Users", "RBAC", "Settings"
  ]

  // MEMBER sees:
  memberNavigation: [
    "Dashboard", "Projects", "Blueprints", "Team", "Settings"
  ]

  // VIEWER sees:
  viewerNavigation: [
    "Dashboard", "Projects (read-only)", "Blueprints (read-only)"
  ]

  // Custom "Project Manager" sees:
  projectManagerNavigation: [
    "Dashboard", "Projects", "Blueprints", "Team (view)", "Analytics", "Settings"
  ]
}
```

---

## Permission Matrix

### Matrix Data Structure

```typescript
interface PermissionMatrix {
  // ============================================
  // MATRIX STRUCTURE
  // ============================================
  
  // Rows: Resource Types
  rows: {
    resourceType: ResourceType
    label: string
    icon: string
    category: string                      // "Content", "Admin", "System"
    actions: ActionType[]                 // Available actions for this resource
  }[]

  // Columns: Roles
  columns: {
    // System roles
    systemRoles: {
      id: string                          // "OWNER"
      name: string
      type: "system"
      level: number
      userCount: number
    }[]

    // Custom roles
    customRoles: {
      id: string
      name: string
      type: "custom"
      baseRole: string
      userCount: number
    }[]
  }

  // Cells: Permission state
  cells: {
    [resourceType: string]: {
      [action: string]: {
        [roleId: string]: {
          effect: "ALLOW" | "DENY" | null
          permissionId: string | null
          isInherited: boolean
          inheritedFrom: string | null    // Role ID it's inherited from
          hasConditions: boolean
          conditions: any | null
          hasUIProperties: boolean
          uiProperties: any | null
        }
      }
    }
  }

  // Statistics
  stats: {
    totalCells: number
    allowCount: number
    denyCount: number
    conditionalCount: number
    inheritedCount: number
    emptyCount: number
    conflictCount: number
  }
}

// ============================================
// MATRIX API
// ============================================

// GET /api/admin/rbac/matrix
// Query params:
//   resourceTypes: comma-separated filter
//   actions: comma-separated filter
//   roles: comma-separated filter
//   includeCustomRoles: boolean
//   includeInherited: boolean
//   includeConditions: boolean

// POST /api/admin/rbac/matrix
// Bulk update matrix cells
{
  updates: [
    {
      resourceType: "PROJECT",
      action: "CREATE",
      role: "MEMBER",                     // or customRoleId
      effect: "ALLOW",                    // or "DENY" or null (remove)
      conditions: null,                   // optional
      uiProperties: null                  // optional
    }
  ]
}

// ============================================
// DEFAULT PERMISSION MATRIX
// ============================================
// This is the out-of-the-box permission matrix

const DEFAULT_MATRIX = {
  // Resource: PROJECT
  PROJECT: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "ALLOW" },
    CREATE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    EDIT:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    DELETE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    MANAGE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    EXECUTE:   { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    CONFIGURE: { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: BLUEPRINT
  BLUEPRINT: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "ALLOW" },
    CREATE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    EDIT:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    DELETE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    MANAGE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    EXECUTE:   { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    CONFIGURE: { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: AI_GENERATION
  AI_GENERATION: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "ALLOW" },
    EXECUTE:   { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    CONFIGURE: { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    MANAGE:    { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: USER_MANAGEMENT
  USER_MANAGEMENT: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    CREATE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    EDIT:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    DELETE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    MANAGE:    { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: BILLING
  BILLING: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    MANAGE:    { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
    CONFIGURE: { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: ORGANIZATION_MANAGEMENT
  ORGANIZATION_MANAGEMENT: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    EDIT:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    DELETE:    { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
    CONFIGURE: { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
    MANAGE:    { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: SYSTEM_CONFIG
  SYSTEM_CONFIG: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
    CONFIGURE: { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
    MANAGE:    { OWNER: "ALLOW", ADMIN: "DENY",  MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: SETTINGS
  SETTINGS: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "ALLOW" },
    EDIT:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
    CONFIGURE: { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "DENY",  VIEWER: "DENY"  },
  },

  // Resource: EXPORT
  EXPORT: {
    EXECUTE:   { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
  },

  // Resource: IMPORT
  IMPORT: {
    EXECUTE:   { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
  },

  // Resource: COLLABORATION
  COLLABORATION: {
    VIEW:      { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "ALLOW" },
    EXECUTE:   { OWNER: "ALLOW", ADMIN: "ALLOW", MEMBER: "ALLOW", VIEWER: "DENY"  },
  },
}
```

---

## Permission Templates & Presets

### System Templates

```typescript
interface PermissionTemplate {
  id: string
  name: string
  description: string
  isSystem: boolean
  organizationId?: string
  
  // Template permissions
  permissions: {
    resourceType: ResourceType
    action: ActionType
    effect: PermissionEffect
    conditions?: any
    uiProperties?: any
  }[]

  // Metadata
  category: string                        // "starter", "team", "enterprise"
  tags: string[]
  usageCount: number
}

const SYSTEM_TEMPLATES: PermissionTemplate[] = [
  // ============================================
  // TEMPLATE: Minimal Read-Only
  // ============================================
  {
    id: "template-minimal-readonly",
    name: "Minimal Read-Only",
    description: "Absolute minimum access - view projects and blueprints only",
    isSystem: true,
    category: "starter",
    permissions: [
      { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW" },
    ]
  },

  // ============================================
  // TEMPLATE: Standard Team Member
  // ============================================
  {
    id: "template-standard-team",
    name: "Standard Team Member",
    description: "Full content access with collaboration, no admin access",
    isSystem: true,
    category: "team",
    permissions: [
      { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "CREATE", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "EDIT", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "DELETE", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "CREATE", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "EDIT", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "DELETE", effect: "ALLOW" },
      { resourceType: "AI_GENERATION", action: "VIEW", effect: "ALLOW" },
      { resourceType: "AI_GENERATION", action: "EXECUTE", effect: "ALLOW" },
      { resourceType: "EXPORT", action: "EXECUTE", effect: "ALLOW" },
      { resourceType: "IMPORT", action: "EXECUTE", effect: "ALLOW" },
      { resourceType: "COLLABORATION", action: "VIEW", effect: "ALLOW" },
      { resourceType: "COLLABORATION", action: "EXECUTE", effect: "ALLOW" },
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW" },
      { resourceType: "SETTINGS", action: "EDIT", effect: "ALLOW" },
    ]
  },

  // ============================================
  // TEMPLATE: Project Manager
  // ============================================
  {
    id: "template-project-manager",
    name: "Project Manager",
    description: "Team member + project management + team viewing",
    isSystem: true,
    category: "team",
    permissions: [
      // All standard team permissions plus:
      { resourceType: "PROJECT", action: "MANAGE", effect: "ALLOW" },
      { resourceType: "PROJECT", action: "CONFIGURE", effect: "ALLOW" },
      { resourceType: "USER_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "BLUEPRINT", action: "MANAGE", effect: "ALLOW" },
    ]
  },

  // ============================================
  // TEMPLATE: Department Admin
  // ============================================
  {
    id: "template-department-admin",
    name: "Department Admin",
    description: "Admin access scoped to a department",
    isSystem: true,
    category: "enterprise",
    permissions: [
      { resourceType: "USER_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "USER_MANAGEMENT", action: "CREATE", effect: "ALLOW",
        conditions: { scope: "department_only" }
      },
      { resourceType: "USER_MANAGEMENT", action: "EDIT", effect: "ALLOW",
        conditions: { scope: "department_only" }
      },
      { resourceType: "PROJECT", action: "MANAGE", effect: "ALLOW",
        conditions: { scope: "department_only" }
      },
      { resourceType: "ORGANIZATION_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
    ]
  },

  // ============================================
  // TEMPLATE: External Contractor
  // ============================================
  {
    id: "template-external-contractor",
    name: "External Contractor",
    description: "Limited access for external contractors with time and IP restrictions",
    isSystem: true,
    category: "enterprise",
    permissions: [
      { resourceType: "PROJECT", action: "VIEW", effect: "ALLOW",
        conditions: { assignedOnly: true }
      },
      { resourceType: "PROJECT", action: "EDIT", effect: "ALLOW",
        conditions: { assignedOnly: true }
      },
      { resourceType: "BLUEPRINT", action: "VIEW", effect: "ALLOW",
        conditions: { assignedOnly: true }
      },
      { resourceType: "BLUEPRINT", action: "EDIT", effect: "ALLOW",
        conditions: { assignedOnly: true }
      },
      { resourceType: "EXPORT", action: "EXECUTE", effect: "DENY" },
      { resourceType: "IMPORT", action: "EXECUTE", effect: "DENY" },
    ]
  },

  // ============================================
  // TEMPLATE: Billing Only
  // ============================================
  {
    id: "template-billing-only",
    name: "Billing Manager",
    description: "Access to billing and subscription management only",
    isSystem: true,
    category: "enterprise",
    permissions: [
      { resourceType: "BILLING", action: "VIEW", effect: "ALLOW" },
      { resourceType: "BILLING", action: "MANAGE", effect: "ALLOW" },
      { resourceType: "BILLING", action: "CONFIGURE", effect: "ALLOW" },
      { resourceType: "ORGANIZATION_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW" },
    ]
  },

  // ============================================
  // TEMPLATE: Security Auditor
  // ============================================
  {
    id: "template-security-auditor",
    name: "Security Auditor",
    description: "Read-only access to audit logs and security settings",
    isSystem: true,
    category: "enterprise",
    permissions: [
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
        conditions: { scope: "security_only" }
      },
      { resourceType: "SETTINGS", action: "VIEW", effect: "ALLOW",
        conditions: { scope: "audit_only" }
      },
      { resourceType: "USER_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
      { resourceType: "ORGANIZATION_MANAGEMENT", action: "VIEW", effect: "ALLOW" },
    ]
  },
]
```

---

## Role Assignment & Lifecycle

### Role Assignment Flows

```typescript
interface RoleAssignment {
  // ============================================
  // ASSIGNMENT METHODS
  // ============================================

  // Method 1: Admin assigns role to user
  adminAssignment: {
    // Route: PATCH /api/admin/users/{userId}
    body: {
      role: "ADMIN"                       // System role change
      // OR
      customRoleId: "clx-project-manager" // Custom role assignment
    }
    // Validation:
    // - Admin cannot assign role above their own level
    // - Admin cannot assign OWNER role (only OWNER can)
    // - Custom role must belong to same organization
    // - User must belong to same organization
  }

  // Method 2: Auto-assignment on registration
  autoAssignment: {
    // Based on email domain
    emailDomainMapping: {
      "@company.com": { role: "MEMBER", customRoleId: null }
      "@admin.company.com": { role: "ADMIN", customRoleId: null }
      "@contractor.company.com": { role: "MEMBER", customRoleId: "clx-contractor" }
    }

    // Based on invitation
    invitationMapping: {
      // Inviter specifies role in invitation
      inviteRole: "MEMBER"
      inviteCustomRole: "clx-project-manager"
    }

    // Based on SSO attributes
    ssoMapping: {
      // Map SSO groups to roles
      "sso-group-admins": { role: "ADMIN" }
      "sso-group-developers": { role: "MEMBER", customRoleId: "clx-developer" }
      "sso-group-viewers": { role: "VIEWER" }
    }

    // Based on onboarding answers
    onboardingMapping: {
      // Map onboarding questionnaire answers to roles
      "role_selection": {
        "manager": { customRoleId: "clx-project-manager" },
        "developer": { customRoleId: "clx-developer" },
        "designer": { customRoleId: "clx-designer" },
      }
    }
  }

  // Method 3: Self-service role request
  selfServiceRequest: {
    // Route: POST /api/user/role-requests
    body: {
      requestedRole: "ADMIN"              // or customRoleId
      reason: "Need admin access for project setup"
      duration?: number                   // Hours (for temporary)
    }
    // Approval flow:
    // 1. User submits request
    // 2. Admin/Owner receives notification
    // 3. Admin reviews and approves/denies
    // 4. User is notified of decision
    // 5. If approved, role is assigned (with optional expiry)
  }

  // Method 4: Bulk assignment
  bulkAssignment: {
    // Route: POST /api/admin/rbac/bulk-assign
    body: {
      userIds: string[]                   // or criteria-based selection
      role?: string
      customRoleId?: string
      reason: string
    }
  }

  // ============================================
  // ROLE LIFECYCLE EVENTS
  // ============================================
  lifecycle: {
    // Event: Role Assigned
    onRoleAssigned: {
      actions: [
        "Log to audit trail",
        "Send notification to user",
        "Send notification to admin",
        "Update permission cache",
        "Trigger webhook (if configured)",
        "Update UI in real-time (WebSocket)"
      ]
    }

    // Event: Role Changed
    onRoleChanged: {
      actions: [
        "Log previous and new role to audit trail",
        "Invalidate permission cache",
        "Send notification to user",
        "Send notification to admin",
        "Re-evaluate active sessions",
        "Trigger webhook"
      ]
    }

    // Event: Role Revoked
    onRoleRevoked: {
      actions: [
        "Log to audit trail",
        "Invalidate permission cache",
        "Revert to default role (MEMBER or VIEWER)",
        "Send notification to user",
        "Send notification to admin",
        "Terminate active sessions (if configured)",
        "Trigger webhook"
      ]
    }

    // Event: Role Expired
    onRoleExpired: {
      actions: [
        "Log to audit trail",
        "Revert to previous role",
        "Invalidate permission cache",
        "Send notification to user",
        "Send notification to admin",
        "Trigger webhook"
      ]
    }

    // Event: Role Deleted
    onRoleDeleted: {
      actions: [
        "Reassign all users to fallback role",
        "Delete all associated permissions",
        "Log to audit trail",
        "Send notification to affected users",
        "Send notification to admin",
        "Invalidate permission cache for all affected users"
      ]
    }
  }
}
```

---

## Delegation & Impersonation

### Delegation Model

```typescript
interface DelegationModel {
  // ============================================
  // PERMISSION DELEGATION
  // ============================================
  // Users can delegate their permissions to others (if allowed)

  delegation: {
    enabled: boolean
    allowedRoles: string[]                // Who can delegate
    maxDelegationDepth: 1                 // Cannot re-delegate
    requireApproval: boolean              // Admin must approve delegation
    maxDuration: number                   // Hours
    auditAll: boolean                     // Log all delegated actions

    // Delegation record
    record: {
      id: string
      delegatorId: string                 // Who is delegating
      delegateeId: string                 // Who receives delegation
      permissions: string[]               // Which permissions are delegated
      reason: string
      startAt: Date
      expiresAt: Date
      approvedBy?: string                 // Admin who approved
      status: "pending" | "active" | "expired" | "revoked"
    }
  }

  // ============================================
  // USER IMPERSONATION
  // ============================================
  // Admin/Owner can impersonate users for debugging

  impersonation: {
    enabled: boolean
    allowedRoles: ["OWNER"]               // Usually only OWNER
    canImpersonate: string[]              // Which roles can be impersonated
    cannotImpersonate: ["OWNER"]          // Cannot impersonate other OWNERs
    requireReason: boolean
    maxDuration: number                   // Minutes
    auditAll: boolean                     // Log every action during impersonation
    showBanner: boolean                   // Show "Impersonating X" banner
    allowDestructiveActions: boolean      // Can delete things while impersonating

    // Impersonation session
    session: {
      id: string
      impersonatorId: string
      impersonatedUserId: string
      reason: string
      startedAt: Date
      expiresAt: Date
      actionsPerformed: {
        action: string
        resourceType: string
        resourceId: string
        timestamp: Date
      }[]
      status: "active" | "ended" | "expired"
    }
  }

  // ============================================
  // IMPERSONATION API
  // ============================================

  // Start impersonation
  // Route: POST /api/admin/rbac/impersonate
  startImpersonation: {
    body: {
      userId: string                      // User to impersonate
      reason: string                      // Required reason
      duration: number                    // Minutes
    }
    response: {
      sessionId: string
      impersonatedUser: { id: string, name: string, role: string }
      expiresAt: Date
      token: string                       // Special impersonation token
    }
  }

  // End impersonation
  // Route: POST /api/admin/rbac/impersonate/end
  endImpersonation: {
    body: {
      sessionId: string
    }
    response: {
      actionsPerformed: number
      duration: number                    // Minutes
      auditLogId: string
    }
  }

  // Get impersonation history
  // Route: GET /api/admin/rbac/impersonate/history
  impersonationHistory: {
    sessions: {
      id: string
      impersonator: string
      impersonatedUser: string
      reason: string
      startedAt: Date
      endedAt: Date
      actionsPerformed: number
    }[]
  }
}
```

---

## Scope & Context-Based Permissions

### Permission Scopes

```typescript
interface PermissionScopes {
  // ============================================
  // SCOPE TYPES
  // ============================================

  // Global scope: applies to all resources of a type
  global: {
    resourceType: "PROJECT"
    action: "VIEW"
    effect: "ALLOW"
    // No resourceId = applies to ALL projects
  }

  // Resource-specific scope: applies to one resource
  resourceSpecific: {
    resourceType: "PROJECT"
    action: "EDIT"
    effect: "ALLOW"
    resourceId: "clx-project-123"         // Only this project
  }

  // Pattern-based scope: applies to resources matching a pattern
  patternBased: {
    resourceType: "PROJECT"
    action: "VIEW"
    effect: "ALLOW"
    conditions: {
      namePattern: "team-*"              // Projects starting with "team-"
      tags: ["public"]                    // Projects tagged "public"
      department: "engineering"           // Projects in engineering department
    }
  }

  // Ownership scope: applies to resources owned by the user
  ownershipBased: {
    resourceType: "PROJECT"
    action: "DELETE"
    effect: "ALLOW"
    conditions: {
      ownerOnly: true                     // Only own projects
    }
  }

  // Team scope: applies to resources shared with user's team
  teamBased: {
    resourceType: "PROJECT"
    action: "EDIT"
    effect: "ALLOW"
    conditions: {
      teamOnly: true                      // Only team projects
    }
  }

  // Department scope: applies to resources in user's department
  departmentBased: {
    resourceType: "USER_MANAGEMENT"
    action: "MANAGE"
    effect: "ALLOW"
    conditions: {
      departmentOnly: true                // Only users in same department
    }
  }

  // ============================================
  // CONTEXT-BASED PERMISSIONS
  // ============================================

  // Time context
  timeContext: {
    resourceType: "SYSTEM_CONFIG"
    action: "CONFIGURE"
    effect: "ALLOW"
    conditions: {
      timeWindow: {
        start: "09:00"
        end: "17:00"
        timezone: "America/New_York"
        daysOfWeek: [1, 2, 3, 4, 5]      // Mon-Fri
      }
    }
  }

  // Location context
  locationContext: {
    resourceType: "BILLING"
    action: "MANAGE"
    effect: "ALLOW"
    conditions: {
      ipRestriction: {
        allowedCIDRs: ["10.0.0.0/8", "172.16.0.0/12"]
        allowedCountries: ["US", "GB"]
      }
    }
  }

  // Device context
  deviceContext: {
    resourceType: "SYSTEM_CONFIG"
    action: "CONFIGURE"
    effect: "ALLOW"
    conditions: {
      deviceRestriction: {
        requireTrustedDevice: true
        allowedPlatforms: ["desktop"]     // No mobile
        requireMFA: true
      }
    }
  }

  // Session context
  sessionContext: {
    resourceType: "BILLING"
    action: "MANAGE"
    effect: "ALLOW"
    conditions: {
      sessionRequirements: {
        maxAge: 30                        // Session must be < 30 minutes old
        requireRecentAuth: true           // Must have authenticated recently
        requireMFA: true                  // Must have completed MFA
      }
    }
  }
}
```

---

## API Architecture

### Complete RBAC API Routes

```typescript
interface RBACAPIRoutes {
  // ============================================
  // ROLE MANAGEMENT
  // ============================================
  roles: {
    // List all roles (system + custom)
    "GET /api/admin/rbac/roles": {
      auth: "OWNER | ADMIN"
      query: { includeSystem?: boolean, includeCustom?: boolean, search?: string }
      response: { systemRoles: SystemRole[], customRoles: CustomRole[] }
    }

    // Create custom role
    "POST /api/admin/rbac/roles": {
      auth: "OWNER | ADMIN"
      body: { name: string, description: string, baseRole: UserRole, permissions?: Permission[] }
      response: { role: CustomRole }
      validation: [
        "name is unique within organization",
        "baseRole cannot be OWNER (unless requester is OWNER)",
        "permissions cannot exceed requester's own permissions"
      ]
    }

    // Get role details
    "GET /api/admin/rbac/roles/:id": {
      auth: "OWNER | ADMIN"
      response: { role: CustomRole, permissions: Permission[], users: User[] }
    }

    // Update role
    "PATCH /api/admin/rbac/roles/:id": {
      auth: "OWNER | ADMIN"
      body: { name?: string, description?: string, baseRole?: UserRole }
      response: { role: CustomRole }
    }

    // Delete role
    "DELETE /api/admin/rbac/roles/:id": {
      auth: "OWNER | ADMIN"
      query: { action?: "reassign" | "force", targetRole?: string }
      response: { success: boolean, reassignedUsers?: number }
    }

    // Clone role
    "POST /api/admin/rbac/roles/:id/clone": {
      auth: "OWNER | ADMIN"
      body: { newName: string, newDescription?: string, additionalPermissions?: Permission[] }
      response: { role: CustomRole }
    }

    // Compare roles
    "GET /api/admin/rbac/roles/compare": {
      auth: "OWNER | ADMIN"
      query: { roleA: string, roleB: string }
      response: { differences: RoleDifference }
    }

    // Get role's effective permissions (after inheritance)
    "GET /api/admin/rbac/roles/:id/effective-permissions": {
      auth: "OWNER | ADMIN"
      response: { permissions: EffectivePermission[] }
    }

    // Assign role to users
    "POST /api/admin/rbac/roles/:id/assign": {
      auth: "OWNER | ADMIN"
      body: { userIds: string[], reason?: string }
      response: { assignedCount: number }
    }

    // Bulk assign role
    "POST /api/admin/rbac/roles/:id/bulk-assign": {
      auth: "OWNER | ADMIN"
      body: { criteria: AssignmentCriteria, reason: string }
      response: { assignedCount: number, users: string[] }
    }
  }

  // ============================================
  // PERMISSION MANAGEMENT
  // ============================================
  permissions: {
    // List permissions
    "GET /api/admin/rbac/permissions": {
      auth: "OWNER | ADMIN"
      query: { resourceType?: string, action?: string, role?: string, customRoleId?: string }
      response: { permissions: Permission[] }
    }

    // Create permission
    "POST /api/admin/rbac/permissions": {
      auth: "OWNER | ADMIN"
      body: {
        resourceType: ResourceType, resourceId?: string, action: ActionType,
        effect: PermissionEffect, role?: UserRole, customRoleId?: string,
        userId?: string, conditions?: any, uiProperties?: any, description?: string
      }
      response: { permission: Permission }
    }

    // Get permission
    "GET /api/admin/rbac/permissions/:id": {
      auth: "OWNER | ADMIN"
      response: { permission: Permission }
    }

    // Update permission
    "PATCH /api/admin/rbac/permissions/:id": {
      auth: "OWNER | ADMIN"
      body: Partial<Permission>
      response: { permission: Permission }
    }

    // Delete permission
    "DELETE /api/admin/rbac/permissions/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }

    // Bulk update permissions
    "PUT /api/admin/rbac/permissions": {
      auth: "OWNER | ADMIN"
      body: { permissions: Permission[] }
      response: { permissions: Permission[] }
    }
  }

  // ============================================
  // PERMISSION MATRIX
  // ============================================
  matrix: {
    // Get permission matrix
    "GET /api/admin/rbac/matrix": {
      auth: "OWNER | ADMIN"
      query: {
        resourceTypes?: string, actions?: string, roles?: string,
        includeCustomRoles?: boolean, includeInherited?: boolean
      }
      response: { matrix: PermissionMatrix, stats: MatrixStats }
    }

    // Bulk update matrix
    "POST /api/admin/rbac/matrix": {
      auth: "OWNER | ADMIN"
      body: { updates: MatrixUpdate[] }
      response: { results: { created: string[], updated: string[], deleted: string[], errors: any[] } }
    }
  }

  // ============================================
  // CAPABILITY MANAGEMENT
  // ============================================
  capabilities: {
    // List capabilities
    "GET /api/admin/rbac/capabilities": {
      auth: "OWNER | ADMIN"
      query: { category?: string, includeSystem?: boolean, includeCustom?: boolean }
      response: { capabilities: Capability[] }
    }

    // Create custom capability
    "POST /api/admin/rbac/capabilities": {
      auth: "OWNER | ADMIN"
      body: { name: string, slug: string, description: string, category: string, permissions: Permission[] }
      response: { capability: Capability }
    }

    // Update capability
    "PATCH /api/admin/rbac/capabilities/:id": {
      auth: "OWNER | ADMIN"
      body: Partial<Capability>
      response: { capability: Capability }
    }

    // Delete capability
    "DELETE /api/admin/rbac/capabilities/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }

    // Assign capability to role
    "POST /api/admin/rbac/roles/:roleId/capabilities": {
      auth: "OWNER | ADMIN"
      body: { capabilityIds: string[] }
      response: { assignedCount: number }
    }

    // Assign capability to user
    "POST /api/admin/rbac/users/:userId/capabilities": {
      auth: "OWNER | ADMIN"
      body: { capabilityIds: string[], reason?: string, expiresAt?: Date }
      response: { assignedCount: number }
    }
  }

  // ============================================
  // ABAC POLICIES
  // ============================================
  policies: {
    // List policies
    "GET /api/admin/rbac/policies": {
      auth: "OWNER | ADMIN"
      response: { policies: ABACPolicy[] }
    }

    // Create policy
    "POST /api/admin/rbac/policies": {
      auth: "OWNER | ADMIN"
      body: ABACPolicy
      response: { policy: ABACPolicy }
    }

    // Update policy
    "PATCH /api/admin/rbac/policies/:id": {
      auth: "OWNER | ADMIN"
      body: Partial<ABACPolicy>
      response: { policy: ABACPolicy }
    }

    // Delete policy
    "DELETE /api/admin/rbac/policies/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }

    // Test policy
    "POST /api/admin/rbac/policies/:id/test": {
      auth: "OWNER | ADMIN"
      body: { userId: string, resourceType: string, action: string, context?: any }
      response: { result: "ALLOW" | "DENY", reason: string, evaluationPath: string[] }
    }
  }

  // ============================================
  // TEMPLATES
  // ============================================
  templates: {
    // List templates
    "GET /api/admin/rbac/templates": {
      auth: "OWNER | ADMIN"
      query: { category?: string, includeSystem?: boolean }
      response: { templates: PermissionTemplate[] }
    }

    // Create template
    "POST /api/admin/rbac/templates": {
      auth: "OWNER | ADMIN"
      body: { name: string, description: string, permissions: Permission[] }
      response: { template: PermissionTemplate }
    }

    // Apply template
    "POST /api/admin/rbac/templates/:id/apply": {
      auth: "OWNER | ADMIN"
      body: { targetRole?: string, targetUserId?: string, mode: "merge" | "replace" }
      response: { appliedPermissions: number }
    }

    // Delete template
    "DELETE /api/admin/rbac/templates/:id": {
      auth: "OWNER | ADMIN"
      response: { success: boolean }
    }
  }

  // ============================================
  // USER PERMISSIONS
  // ============================================
  userPermissions: {
    // Get user's effective permissions
    "GET /api/admin/rbac/users/:userId/effective-permissions": {
      auth: "OWNER | ADMIN"
      response: { permissions: EffectivePermission[], sources: PermissionSource[] }
    }

    // Set direct user permission
    "POST /api/admin/rbac/users/:userId/permissions": {
      auth: "OWNER | ADMIN"
      body: { permissions: Permission[], reason?: string }
      response: { permissions: Permission[] }
    }

    // Revoke all user permissions
    "POST /api/admin/rbac/users/:userId/revoke-all": {
      auth: "OWNER | ADMIN"
      body: { reason: string, lockAccount?: boolean }
      response: { revokedCount: number }
    }

    // Elevate user temporarily
    "POST /api/admin/rbac/users/:userId/elevate": {
      auth: "OWNER | ADMIN"
      body: { targetRole: string, duration: number, reason: string, restrictions?: any }
      response: { elevation: Elevation }
    }
  }

  // ============================================
  // DELEGATION & IMPERSONATION
  // ============================================
  delegation: {
    // Create delegation
    "POST /api/admin/rbac/delegations": {
      auth: "OWNER | ADMIN | MEMBER (if allowed)"
      body: { delegateeId: string, permissions: string[], reason: string, duration: number }
      response: { delegation: Delegation }
    }

    // List delegations
    "GET /api/admin/rbac/delegations": {
      auth: "OWNER | ADMIN"
      response: { delegations: Delegation[] }
    }

    // Revoke delegation
    "DELETE /api/admin/rbac/delegations/:id": {
      auth: "OWNER | ADMIN | delegator"
      response: { success: boolean }
    }
  }

  impersonation: {
    // Start impersonation
    "POST /api/admin/rbac/impersonate": {
      auth: "OWNER"
      body: { userId: string, reason: string, duration: number }
      response: { session: ImpersonationSession }
    }

    // End impersonation
    "POST /api/admin/rbac/impersonate/end": {
      auth: "OWNER (impersonating)"
      response: { actionsPerformed: number }
    }

    // Impersonation history
    "GET /api/admin/rbac/impersonate/history": {
      auth: "OWNER"
      response: { sessions: ImpersonationSession[] }
    }
  }

  // ============================================
  // AUDIT
  // ============================================
  audit: {
    // Get RBAC audit log
    "GET /api/admin/rbac/audit": {
      auth: "OWNER | ADMIN"
      query: { action?: string, resourceType?: string, performedBy?: string, dateRange?: string }
      response: { logs: RBACAuditLog[], total: number }
    }

    // Get access decision log
    "GET /api/admin/rbac/audit/access-decisions": {
      auth: "OWNER | ADMIN"
      query: { userId?: string, resourceType?: string, result?: string, dateRange?: string }
      response: { decisions: AccessDecision[], total: number }
    }

    // Export audit log
    "GET /api/admin/rbac/audit/export": {
      auth: "OWNER"
      query: { format: "csv" | "json" | "pdf", dateRange?: string }
      response: File
    }
  }

  // ============================================
  // SIMULATION
  // ============================================
  simulation: {
    // Check permission for user
    "POST /api/admin/rbac/simulate/check": {
      auth: "OWNER | ADMIN"
      body: { userId: string, resourceType: string, action: string, resourceId?: string, context?: any }
      response: { result: "ALLOW" | "DENY", reason: string, source: string, evaluationPath: string[] }
    }

    // Simulate permission change
    "POST /api/admin/rbac/simulate/change": {
      auth: "OWNER | ADMIN"
      body: { changes: Permission[], targetUsers?: string[] }
      response: { affectedUsers: number, beforeAfter: { userId: string, before: string, after: string }[] }
    }

    // Who has access to resource
    "GET /api/admin/rbac/simulate/resource-access": {
      auth: "OWNER | ADMIN"
      query: { resourceType: string, resourceId: string, action: string }
      response: { users: { userId: string, name: string, role: string, source: string }[] }
    }
  }

  // ============================================
  // EXPORT / IMPORT
  // ============================================
  config: {
    // Export RBAC configuration
    "GET /api/admin/rbac/export": {
      auth: "OWNER"
      query: { format: "json", includeAudit?: boolean }
      response: { config: RBACExport }
    }

    // Import RBAC configuration
    "POST /api/admin/rbac/import": {
      auth: "OWNER"
      body: { config: RBACExport, mode: "merge" | "replace", conflictResolution: string }
      response: { imported: number, conflicts: number, errors: any[] }
    }
  }

  // ============================================
  // USER SELF-SERVICE
  // ============================================
  userSelfService: {
    // Get my permissions
    "GET /api/user/permissions": {
      auth: "any authenticated user"
      response: { role: string, customRole?: string, permissions: SimplifiedPermission[], capabilities: string[] }
    }

    // Request permission change
    "POST /api/user/permission-requests": {
      auth: "any authenticated user"
      body: { requestedPermission: string, reason: string, duration?: number }
      response: { request: PermissionRequest }
    }

    // Get my permission requests
    "GET /api/user/permission-requests": {
      auth: "any authenticated user"
      response: { requests: PermissionRequest[] }
    }

    // Check specific permission
    "GET /api/user/can": {
      auth: "any authenticated user"
      query: { resourceType: string, action: string, resourceId?: string }
      response: { allowed: boolean, reason?: string }
    }
  }
}
```

---

## Database Schema

### Complete RBAC Schema

```prisma
// ============================================
// ENUMS
// ============================================

enum UserRole {
  OWNER       // Platform owner - super admin
  ADMIN       // Organization admin
  MEMBER      // Regular user
  VIEWER      // Read-only access
}

enum PermissionEffect {
  ALLOW
  DENY
}

enum ResourceType {
  // UI Elements
  BUTTON
  LINK
  MENU_ITEM
  TAB
  CARD
  FORM_FIELD
  DIALOG
  SIDEBAR_SECTION

  // Features
  PROJECT
  BLUEPRINT
  MINDMAP
  AI_GENERATION
  EXPORT
  IMPORT
  COLLABORATION

  // Admin
  USER_MANAGEMENT
  BILLING
  SETTINGS
  SYSTEM_CONFIG
  ORGANIZATION_MANAGEMENT

  // Data Operations
  CREATE
  READ
  UPDATE
  DELETE
}

enum ActionType {
  VIEW
  CREATE
  EDIT
  DELETE
  EXECUTE
  CONFIGURE
  MANAGE
}

// ============================================
// CUSTOM ROLES
// ============================================

model CustomRole {
  id             String  @id @default(cuid())
  organizationId String
  name           String
  slug           String?
  description    String?
  icon           String?
  color          String?

  // Base role to inherit from
  baseRole UserRole @default(MEMBER)

  // Inheritance
  parentRoleId String?
  parentRole   CustomRole?  @relation("RoleInheritance", fields: [parentRoleId], references: [id])
  childRoles   CustomRole[] @relation("RoleInheritance")

  // Status
  isActive    Boolean   @default(true)
  isDefault   Boolean   @default(false)
  isTemporary Boolean   @default(false)
  expiresAt   DateTime?

  // Auto-assignment criteria
  autoAssignCriteria Json?

  // Restrictions
  restrictions Json?

  // Version tracking
  version Int @default(1)

  // Timestamps
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?

  // Relations
  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  users        User[]
  permissions  Permission[]
  capabilities RoleCapability[]

  @@unique([organizationId, name])
  @@index([organizationId])
  @@index([baseRole])
  @@index([isActive])
  @@index([deletedAt])
}

// ============================================
// PERMISSIONS
// ============================================

model Permission {
  id             String @id @default(cuid())
  organizationId String

  // What is being controlled
  resourceType ResourceType
  resourceId   String?
  action       ActionType

  // Who gets this permission
  role         UserRole?
  customRoleId String?
  userId       String?

  // The permission itself
  effect PermissionEffect @default(ALLOW)

  // Conditions (ABAC)
  conditions Json?

  // UI-specific properties
  uiProperties Json?

  // Metadata
  description String?
  createdById String
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  deletedAt   DateTime?

  // Relations
  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  customRole   CustomRole?  @relation(fields: [customRoleId], references: [id], onDelete: Cascade)

  @@index([organizationId])
  @@index([resourceType])
  @@index([customRoleId])
  @@index([userId])
  @@index([role])
  @@index([resourceType, action])
  @@index([organizationId, resourceType, action, role])
  @@index([organizationId, resourceType, action, customRoleId])
  @@index([organizationId, resourceType, action, userId])
  @@index([deletedAt])
}

// ============================================
// CAPABILITIES
// ============================================

model Capability {
  id             String  @id @default(cuid())
  name           String
  slug           String
  description    String?
  icon           String?
  category       String

  // System or custom
  isSystem       Boolean @default(false)
  organizationId String?

  // Permissions in this capability
  permissions Json

  // Dependencies
  requires  String[]
  conflicts String[]

  // Plan restrictions
  availableInPlans String[]

  // Visibility
  visibleToRoles    String[]
  assignableByRoles String[]

  // Timestamps
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?

  // Relations
  roleCapabilities RoleCapability[]
  userCapabilities UserCapability[]

  @@unique([slug, organizationId])
  @@index([isSystem])
  @@index([organizationId])
  @@index([category])
  @@index([deletedAt])
}

model RoleCapability {
  id           String @id @default(cuid())
  customRoleId String
  capabilityId String

  customRole CustomRole @relation(fields: [customRoleId], references: [id], onDelete: Cascade)
  capability Capability @relation(fields: [capabilityId], references: [id], onDelete: Cascade)

  assignedAt DateTime @default(now())
  assignedBy String

  @@unique([customRoleId, capabilityId])
}

model UserCapability {
  id           String    @id @default(cuid())
  userId       String
  capabilityId String
  reason       String?
  expiresAt    DateTime?

  capability Capability @relation(fields: [capabilityId], references: [id], onDelete: Cascade)

  assignedAt DateTime @default(now())
  assignedBy String

  @@unique([userId, capabilityId])
  @@index([userId])
  @@index([expiresAt])
}

// ============================================
// ABAC POLICIES
// ============================================

model ABACPolicy {
  id             String  @id @default(cuid())
  organizationId String
  name           String
  description    String?
  priority       Int     @default(0)
  enabled        Boolean @default(true)

  // Policy definition
  target    Json
  condition Json
  effect    String    // "ALLOW" | "DENY"
  obligations Json?

  // Timestamps
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?

  @@index([organizationId])
  @@index([enabled])
  @@index([priority])
  @@index([deletedAt])
}

// ============================================
// PERMISSION TEMPLATES
// ============================================

model PermissionTemplate {
  id          String  @id @default(cuid())
  name        String
  description String?
  category    String?
  tags        String[]

  // Template permissions
  permissions Json

  // Scope
  isSystem       Boolean @default(false)
  organizationId String?

  // Usage tracking
  usageCount Int @default(0)

  // Timestamps
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?

  @@unique([name, organizationId])
  @@index([isSystem])
  @@index([organizationId])
  @@index([category])
  @@index([deletedAt])
}

// ============================================
// DELEGATION
// ============================================

model Delegation {
  id             String @id @default(cuid())
  organizationId String

  delegatorId  String
  delegateeId  String
  permissions  String[]
  reason       String
  
  startAt   DateTime @default(now())
  expiresAt DateTime
  
  approvedBy String?
  status     String   @default("pending") // pending, active, expired, revoked

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([organizationId])
  @@index([delegatorId])
  @@index([delegateeId])
  @@index([status])
  @@index([expiresAt])
}

// ============================================
// IMPERSONATION
// ============================================

model ImpersonationSession {
  id             String @id @default(cuid())
  organizationId String

  impersonatorId     String
  impersonatedUserId String
  reason             String
  
  startedAt DateTime @default(now())
  endedAt   DateTime?
  expiresAt DateTime

  actionsPerformed Json @default("[]")
  status           String @default("active") // active, ended, expired

  @@index([organizationId])
  @@index([impersonatorId])
  @@index([impersonatedUserId])
  @@index([status])
}

// ============================================
// ROLE ELEVATION (Temporary)
// ============================================

model RoleElevation {
  id             String @id @default(cuid())
  organizationId String

  userId       String
  previousRole String
  elevatedRole String
  reason       String
  
  grantedBy String
  grantedAt DateTime @default(now())
  expiresAt DateTime
  revertedAt DateTime?

  restrictions Json?
  status       String @default("active") // active, expired, reverted

  @@index([organizationId])
  @@index([userId])
  @@index([status])
  @@index([expiresAt])
}

// ============================================
// PERMISSION REQUESTS (User Self-Service)
// ============================================

model PermissionRequest {
  id             String @id @default(cuid())
  organizationId String

  requesterId         String
  requestedPermission String
  requestedDuration   Int?      // Hours
  reason              String
  urgency             String    @default("low") // low, medium, high

  reviewedBy String?
  reviewedAt DateTime?
  decision   String?  // approved, denied
  decisionReason String?

  status String @default("pending") // pending, approved, denied, expired

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([organizationId])
  @@index([requesterId])
  @@index([status])
}

// ============================================
// RBAC AUDIT LOG
// ============================================

model RBACAuditLog {
  id             String @id @default(cuid())
  organizationId String

  // What changed
  action       String
  resourceType String
  resourceId   String?

  // Before/after
  previousValue Json?
  newValue      Json?

  // Who made the change
  performedById String
  performedAt   DateTime @default(now())

  // Reason
  reason String?

  @@index([organizationId])
  @@index([performedAt])
  @@index([action])
  @@index([resourceType])
  @@index([performedById])
}

// ============================================
// ACCESS DECISION LOG
// ============================================

model AccessDecisionLog {
  id             String @id @default(cuid())
  organizationId String

  userId       String
  resourceType String
  resourceId   String?
  action       String
  
  result       String    // "ALLOW" | "DENY"
  reason       String
  source       String    // "direct_user", "custom_role", "system_role", "policy", "default"
  permissionId String?
  
  // Context
  ipAddress    String?
  userAgent    String?
  geoLocation  Json?
  
  evaluationTimeMs Int?
  
  decidedAt DateTime @default(now())

  @@index([organizationId])
  @@index([userId])
  @@index([resourceType])
  @@index([result])
  @@index([decidedAt])
}
```

---

## Middleware & Guards

### Server-Side Permission Guards

```typescript
// ============================================
// API ROUTE GUARD
// ============================================

// Middleware for API routes
async function withPermission(
  request: NextRequest,
  requiredPermission: { resourceType: ResourceType, action: ActionType },
  handler: (request: NextRequest, user: AuthenticatedUser) => Promise<NextResponse>
): Promise<NextResponse> {
  // Step 1: Authenticate
  const session = await auth()
  if (!session?.user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 })
  }

  // Step 2: Load user
  const user = await prisma.user.findUnique({
    where: { id: session.user.id },
    select: { id: true, role: true, customRoleId: true, organizationId: true, isActive: true }
  })

  if (!user || !user.isActive) {
    return NextResponse.json({ error: "Account inactive" }, { status: 403 })
  }

  // Step 3: Check permission
  const result = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType: requiredPermission.resourceType,
    action: requiredPermission.action,
    context: {
      ip: request.headers.get("x-forwarded-for") || "unknown",
      userAgent: request.headers.get("user-agent") || "unknown",
      timestamp: new Date(),
    }
  })

  if (!result.allowed) {
    // Log denied access
    await logAccessDecision(user, requiredPermission, result)

    return NextResponse.json(
      { error: "Forbidden", reason: result.reason },
      { status: 403 }
    )
  }

  // Step 4: Execute handler
  return handler(request, user as AuthenticatedUser)
}

// Usage in API route:
export async function GET(request: NextRequest) {
  return withPermission(
    request,
    { resourceType: "PROJECT", action: "VIEW" },
    async (request, user) => {
      const projects = await prisma.project.findMany({
        where: { organizationId: user.organizationId }
      })
      return NextResponse.json({ projects })
    }
  )
}

// ============================================
// RESOURCE-SPECIFIC GUARD
// ============================================

async function withResourcePermission(
  request: NextRequest,
  resourceType: ResourceType,
  action: ActionType,
  resourceId: string,
  handler: (request: NextRequest, user: AuthenticatedUser) => Promise<NextResponse>
): Promise<NextResponse> {
  const session = await auth()
  if (!session?.user) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 })
  }

  const user = await prisma.user.findUnique({
    where: { id: session.user.id },
    select: { id: true, role: true, customRoleId: true, organizationId: true, isActive: true }
  })

  if (!user || !user.isActive) {
    return NextResponse.json({ error: "Account inactive" }, { status: 403 })
  }

  // Check resource-specific permission
  const result = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType,
    action,
    resourceId,
    context: {
      ip: request.headers.get("x-forwarded-for") || "unknown",
      userAgent: request.headers.get("user-agent") || "unknown",
      timestamp: new Date(),
    }
  })

  if (!result.allowed) {
    return NextResponse.json(
      { error: "Forbidden", reason: result.reason },
      { status: 403 }
    )
  }

  return handler(request, user as AuthenticatedUser)
}

// ============================================
// ROLE GUARD (Simple)
// ============================================

function requireRole(...roles: UserRole[]) {
  return async function(
    request: NextRequest,
    handler: (request: NextRequest, user: AuthenticatedUser) => Promise<NextResponse>
  ): Promise<NextResponse> {
    const session = await auth()
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 })
    }

    const user = session.user as any
    if (!roles.includes(user.role)) {
      return NextResponse.json({ error: "Forbidden" }, { status: 403 })
    }

    return handler(request, user)
  }
}

// Usage:
export const GET = requireRole("OWNER", "ADMIN")(async (request, user) => {
  // Only OWNER and ADMIN can access this
})

// ============================================
// MIDDLEWARE PERMISSION CHECK (Next.js Middleware)
// ============================================

// In middleware.ts - route-level protection
const ROUTE_PERMISSIONS: Record<string, { resourceType: ResourceType, action: ActionType }> = {
  "/admin": { resourceType: "USER_MANAGEMENT", action: "VIEW" },
  "/admin/users": { resourceType: "USER_MANAGEMENT", action: "VIEW" },
  "/admin/rbac": { resourceType: "USER_MANAGEMENT", action: "MANAGE" },
  "/admin/settings": { resourceType: "SETTINGS", action: "CONFIGURE" },
  "/admin/billing": { resourceType: "BILLING", action: "VIEW" },
  "/admin/system": { resourceType: "SYSTEM_CONFIG", action: "VIEW" },
  "/admin/audit": { resourceType: "SETTINGS", action: "VIEW" },
}

// In middleware, check route permissions before rendering page
export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl

  // Find matching route permission
  const routePermission = Object.entries(ROUTE_PERMISSIONS).find(
    ([route]) => pathname.startsWith(route)
  )

  if (routePermission) {
    // Check if user has permission for this route
    // Note: Full permission check requires DB access, which middleware can't do
    // So we do a lightweight role check here and full check in the page/API
    const session = await getSessionFromCookie(request)
    if (session?.user?.role === "VIEWER" || session?.user?.role === "MEMBER") {
      if (pathname.startsWith("/admin")) {
        return NextResponse.redirect(new URL("/dashboard", request.url))
      }
    }
  }

  return NextResponse.next()
}
```

---

## Audit & Compliance

### Audit Configuration

```typescript
interface RBACAuditConfiguration {
  // ============================================
  // WHAT TO AUDIT
  // ============================================
  auditEvents: {
    // Permission changes
    permissionCreated: boolean
    permissionUpdated: boolean
    permissionDeleted: boolean
    permissionBulkUpdated: boolean

    // Role changes
    roleCreated: boolean
    roleUpdated: boolean
    roleDeleted: boolean
    roleAssigned: boolean
    roleRevoked: boolean
    roleElevated: boolean
    roleElevationExpired: boolean

    // Capability changes
    capabilityCreated: boolean
    capabilityUpdated: boolean
    capabilityDeleted: boolean
    capabilityAssigned: boolean
    capabilityRevoked: boolean

    // Policy changes
    policyCreated: boolean
    policyUpdated: boolean
    policyDeleted: boolean
    policyEnabled: boolean
    policyDisabled: boolean

    // Access decisions
    accessAllowed: boolean                // Log every ALLOW decision
    accessDenied: boolean                 // Log every DENY decision
    accessConflict: boolean               // Log when conflicts are resolved

    // Delegation & impersonation
    delegationCreated: boolean
    delegationRevoked: boolean
    impersonationStarted: boolean
    impersonationEnded: boolean
    impersonationAction: boolean          // Every action during impersonation

    // Configuration changes
    rbacConfigChanged: boolean
    rbacExported: boolean
    rbacImported: boolean

    // Self-service
    permissionRequested: boolean
    permissionRequestApproved: boolean
    permissionRequestDenied: boolean
  }

  // ============================================
  // AUDIT RETENTION
  // ============================================
  retention: {
    permissionChanges: number             // Days to retain (365)
    accessDecisions: number               // Days to retain (90)
    impersonationLogs: number             // Days to retain (365)
    configChanges: number                 // Days to retain (730 = 2 years)
    autoArchive: boolean                  // Archive old logs
    archiveFormat: "compressed_json" | "parquet"
  }

  // ============================================
  // AUDIT ALERTS
  // ============================================
  alerts: {
    // Alert when suspicious activity detected
    suspiciousActivity: {
      enabled: boolean
      triggers: [
        "Multiple permission denials in short time",
        "Permission escalation attempt",
        "Bulk permission changes",
        "Off-hours admin activity",
        "Impersonation of high-privilege user",
        "RBAC configuration export",
        "Role deletion with active users"
      ]
      notifyChannels: ("email" | "slack" | "webhook")[]
      notifyRoles: string[]               // ["OWNER"]
    }

    // Compliance alerts
    compliance: {
      enabled: boolean
      triggers: [
        "Orphaned permissions detected",
        "Unused roles for > 30 days",
        "Users with excessive permissions",
        "Missing audit logs",
        "Permission conflicts detected"
      ]
      scheduleCheck: "daily" | "weekly"
      notifyRoles: string[]
    }
  }

  // ============================================
  // COMPLIANCE REPORTS
  // ============================================
  reports: {
    // SOC 2 compliance report
    soc2: {
      enabled: boolean
      schedule: "monthly" | "quarterly"
      includes: [
        "Access control changes",
        "Permission audit trail",
        "Role assignment history",
        "Impersonation log",
        "Failed access attempts"
      ]
    }

    // GDPR compliance report
    gdpr: {
      enabled: boolean
      includes: [
        "Data access permissions",
        "User consent tracking",
        "Data export permissions",
        "Data deletion permissions"
      ]
    }

    // Custom compliance report
    custom: {
      enabled: boolean
      template: string                    // Report template
      schedule: string                    // Cron expression
      recipients: string[]                // Email addresses
    }
  }
}
```

---

## Edge Cases & Conflict Resolution

### Edge Case Handling

```typescript
interface RBACEdgeCases {
  // ============================================
  // EDGE CASE 1: Last OWNER Tries to Demote Themselves
  // ============================================
  lastOwnerDemotion: {
    scenario: "Only OWNER in org tries to change their role to ADMIN"
    handling: "DENY - Cannot demote last OWNER"
    response: {
      error: "Cannot demote the last OWNER. Assign another OWNER first.",
      code: "LAST_OWNER_PROTECTION"
    }
  }

  // ============================================
  // EDGE CASE 2: Admin Tries to Grant Permissions They Don't Have
  // ============================================
  privilegeEscalation: {
    scenario: "ADMIN tries to create a custom role with SYSTEM_CONFIG:CONFIGURE"
    handling: "DENY - Cannot grant permissions above own level"
    response: {
      error: "Cannot grant permission SYSTEM_CONFIG:CONFIGURE. You don't have this permission.",
      code: "PRIVILEGE_ESCALATION_PREVENTED"
    }
    // Prevention: Before creating/updating permissions, verify the requester has
    // all permissions they're trying to grant
  }

  // ============================================
  // EDGE CASE 3: Circular Role Inheritance
  // ============================================
  circularInheritance: {
    scenario: "Role A inherits from Role B, Role B inherits from Role A"
    handling: "DENY - Detect and prevent circular inheritance"
    detection: "Walk inheritance chain before saving; if current role appears, reject"
    response: {
      error: "Circular inheritance detected. Role 'A' cannot inherit from 'B' because 'B' already inherits from 'A'.",
      code: "CIRCULAR_INHERITANCE"
    }
  }

  // ============================================
  // EDGE CASE 4: Permission Conflict Between Role and Direct User Permission
  // ============================================
  permissionConflict: {
    scenario: "User's role says ALLOW for PROJECT:DELETE, but direct user permission says DENY"
    handling: "Direct user permission wins (higher priority)"
    resolution: "DENY (direct user permission takes precedence)"
    logging: "Log conflict resolution with both sources"
  }

  // ============================================
  // EDGE CASE 5: Custom Role Deleted While Users Are Assigned
  // ============================================
  roleDeletedWithUsers: {
    scenario: "Admin deletes 'Project Manager' role with 15 users assigned"
    handling: "Require explicit action: reassign or force delete"
    options: [
      "Reassign all users to another role before deleting",
      "Force delete and revert users to base role (requires OWNER)"
    ]
    response: {
      error: "Cannot delete role with 15 active users",
      activeUsers: 15,
      options: ["reassign", "force_delete"]
    }
  }

  // ============================================
  // EDGE CASE 6: Organization Plan Downgrade Removes RBAC Features
  // ============================================
  planDowngrade: {
    scenario: "Org downgrades from PRO to FREE, losing custom roles"
    handling: "Graceful degradation"
    steps: [
      "1. Notify admin about features being removed",
      "2. Grace period (30 days) to export RBAC config",
      "3. Custom roles are deactivated (not deleted)",
      "4. Users revert to system roles",
      "5. ABAC policies are deactivated",
      "6. If org upgrades again, custom roles can be reactivated"
    ]
  }

  // ============================================
  // EDGE CASE 7: User Belongs to Multiple Organizations
  // ============================================
  multiOrgUser: {
    scenario: "User has ADMIN in Org A and VIEWER in Org B"
    handling: "Permissions are always scoped to active organization"
    rules: [
      "Permissions from Org A never leak into Org B",
      "Switching orgs completely reloads permissions",
      "Session token includes active organizationId",
      "API calls validate organizationId matches session"
    ]
  }

  // ============================================
  // EDGE CASE 8: Temporary Elevation Expires During Active Session
  // ============================================
  elevationExpiry: {
    scenario: "User was elevated to ADMIN for 2 hours; elevation expires while they're using admin panel"
    handling: "Real-time permission revocation"
    steps: [
      "1. Background job checks for expired elevations every minute",
      "2. On expiry: revert role, invalidate permission cache",
      "3. Send WebSocket event to user's browser",
      "4. UI immediately re-renders (admin panel elements disappear)",
      "5. If user tries admin action, API returns 403",
      "6. Show notification: 'Your temporary admin access has expired'"
    ]
  }

  // ============================================
  // EDGE CASE 9: Permission Check on Non-Existent Resource
  // ============================================
  nonExistentResource: {
    scenario: "Permission check for resourceId that doesn't exist"
    handling: "Fall back to type-level permission"
    rules: [
      "If resourceId is provided but resource doesn't exist: return 404 (not 403)",
      "If checking type-level permission (no resourceId): normal permission check",
      "Never reveal resource existence through permission errors"
    ]
  }

  // ============================================
  // EDGE CASE 10: Concurrent Permission Modifications
  // ============================================
  concurrentModification: {
    scenario: "Two admins modify the same permission simultaneously"
    handling: "Optimistic locking with version check"
    steps: [
      "1. Each permission has a version number",
      "2. Update includes expected version",
      "3. If version mismatch: return 409 Conflict",
      "4. Client must refresh and retry",
      "5. Audit log captures both attempts"
    ]
  }

  // ============================================
  // EDGE CASE 11: Wildcard Permission vs Specific Deny
  // ============================================
  wildcardVsSpecific: {
    scenario: "User has PROJECT:* ALLOW but PROJECT:DELETE DENY"
    handling: "Specific permission overrides wildcard"
    resolution: "PROJECT:DELETE is DENY (specific wins over wildcard)"
  }

  // ============================================
  // EDGE CASE 12: Capability Dependency Not Met
  // ============================================
  capabilityDependency: {
    scenario: "Admin assigns 'Export Audit Logs' capability without 'View Audit Logs'"
    handling: "Auto-resolve or warn"
    options: [
      "Auto-add required capability with notification",
      "Warn admin and require explicit confirmation",
      "Reject assignment until dependency is met"
    ]
    configurable: true
  }

  // ============================================
  // EDGE CASE 13: ABAC Policy Conflicts with RBAC Permission
  // ============================================
  abacRbacConflict: {
    scenario: "RBAC says ALLOW for PROJECT:VIEW, but ABAC policy says DENY (geo-restriction)"
    handling: "ABAC DENY overrides RBAC ALLOW (security-first)"
    rules: [
      "ABAC DENY always wins over RBAC ALLOW",
      "ABAC ALLOW does NOT override RBAC DENY",
      "This ensures security policies cannot be bypassed by role permissions"
    ]
  }

  // ============================================
  // EDGE CASE 14: Impersonation During Impersonation
  // ============================================
  nestedImpersonation: {
    scenario: "OWNER impersonating ADMIN tries to impersonate MEMBER"
    handling: "DENY - No nested impersonation"
    response: {
      error: "Cannot impersonate while already impersonating another user",
      code: "NESTED_IMPERSONATION_DENIED"
    }
  }

  // ============================================
  // EDGE CASE 15: Permission Cache Invalidation Race Condition
  // ============================================
  cacheRaceCondition: {
    scenario: "Admin revokes permission; user makes request before cache invalidates"
    handling: "Dual-layer protection"
    steps: [
      "1. Permission change triggers immediate cache invalidation",
      "2. Cache invalidation is broadcast via pub/sub (Redis)",
      "3. If cache miss, always check database (fallback)",
      "4. Short cache TTL (5 minutes) as safety net",
      "5. Critical operations (delete, billing) always check database directly"
    ]
  }

  // ============================================
  // EDGE CASE 16: Soft-Deleted Role Referenced by Active Permission
  // ============================================
  softDeletedRoleReference: {
    scenario: "Custom role is soft-deleted but permissions still reference it"
    handling: "Cascade soft-delete"
    steps: [
      "1. Soft-delete role sets deletedAt timestamp",
      "2. All permissions referencing this role are also soft-deleted",
      "3. All users with this customRoleId have it set to null",
      "4. Audit log captures all cascaded changes",
      "5. Restore role also restores permissions and user assignments"
    ]
  }
}
```

---

## Performance & Caching

### Permission Caching Strategy

```typescript
interface PermissionCaching {
  // ============================================
  // CACHING LAYERS
  // ============================================

  layers: {
    // Layer 1: In-memory cache (per-request)
    requestCache: {
      scope: "single request"
      ttl: 0                              // Request lifetime
      purpose: "Avoid multiple DB queries in same request"
      implementation: "Map in request context"
    }

    // Layer 2: Application cache (Redis)
    applicationCache: {
      scope: "all requests for same user"
      ttl: 300                            // 5 minutes
      purpose: "Avoid DB queries for repeated permission checks"
      implementation: "Redis with key pattern: perm:{orgId}:{userId}:{resourceType}:{action}"
      invalidation: "On permission change, invalidate all keys for affected users"
    }

    // Layer 3: Precomputed effective permissions
    precomputedCache: {
      scope: "user's complete permission set"
      ttl: 3600                           // 1 hour
      purpose: "Serve complete permission set to frontend"
      implementation: "Redis hash: effective:{orgId}:{userId}"
      invalidation: "On any permission/role change affecting user"
      precomputation: "Background job computes effective permissions"
    }
  }

  // ============================================
  // CACHE INVALIDATION
  // ============================================
  invalidation: {
    // Events that trigger cache invalidation
    triggers: {
      permissionCreated: ["affected_users", "affected_roles"]
      permissionUpdated: ["affected_users", "affected_roles"]
      permissionDeleted: ["affected_users", "affected_roles"]
      roleCreated: []                     // No users affected yet
      roleUpdated: ["all_users_with_role"]
      roleDeleted: ["all_users_with_role"]
      roleAssigned: ["specific_user"]
      roleRevoked: ["specific_user"]
      policyChanged: ["all_org_users"]    // ABAC policy affects everyone
      orgSettingsChanged: ["all_org_users"]
    }

    // Invalidation strategy
    strategy: {
      method: "targeted"                  // Only invalidate affected keys
      broadcast: "redis_pubsub"           // Broadcast invalidation to all instances
      fallback: "database"                // Always fall back to DB on cache miss
      maxStaleness: 300                   // Maximum seconds a stale cache is acceptable
    }
  }

  // ============================================
  // PERFORMANCE TARGETS
  // ============================================
  targets: {
    permissionCheckLatency: {
      p50: 1                              // ms (from cache)
      p95: 10                             // ms (cache miss, DB query)
      p99: 50                             // ms (complex ABAC evaluation)
    }
    cacheHitRate: 0.95                    // 95% cache hit rate
    maxPermissionsPerUser: 1000           // Performance degrades above this
    maxRolesPerOrg: 100                   // Performance degrades above this
    maxPoliciesPerOrg: 50                 // ABAC policies
  }

  // ============================================
  // OPTIMIZATION STRATEGIES
  // ============================================
  optimizations: {
    // Batch permission checks
    batchCheck: {
      enabled: true
      description: "Check multiple permissions in single query"
      usage: "checkPermissions([{resource, action}, ...]) instead of multiple checkPermission()"
    }

    // Preload permissions on login
    preloadOnLogin: {
      enabled: true
      description: "Load all user permissions on login and send to frontend"
      payload: "Effective permissions as compact JSON"
      maxPayloadSize: "50KB"
    }

    // Indexed permission lookups
    indexing: {
      compositeIndexes: [
        "[organizationId, resourceType, action, role]",
        "[organizationId, resourceType, action, customRoleId]",
        "[organizationId, resourceType, action, userId]",
        "[organizationId, userId]",
      ]
    }

    // Permission denormalization
    denormalization: {
      enabled: true
      description: "Store effective permissions in a denormalized table for fast lookups"
      table: "EffectivePermission"
      refreshStrategy: "on_change"        // Refresh when source permissions change
    }
  }
}
```

---

## Integration Examples

### Complete Integration: Protected API Route

```typescript
// ============================================
// EXAMPLE: Full RBAC-Protected API Route
// ============================================
// File: app/api/projects/[id]/route.ts

import { NextRequest, NextResponse } from 'next/server'
import { auth } from '@/lib/auth'
import { prisma } from '@/lib/db'
import { checkPermission } from '@/lib/rbac'
import { logAccessDecision } from '@/lib/rbac/audit'

export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  // Step 1: Authenticate
  const session = await auth()
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { id: projectId } = await params
  const user = session.user as any

  // Step 2: Check type-level permission
  const typePermission = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType: 'PROJECT',
    action: 'VIEW',
    context: {
      ip: request.headers.get('x-forwarded-for') || 'unknown',
      userAgent: request.headers.get('user-agent') || 'unknown',
      timestamp: new Date(),
    }
  })

  if (!typePermission.allowed) {
    await logAccessDecision(user, 'PROJECT', 'VIEW', projectId, typePermission)
    return NextResponse.json(
      { error: 'You do not have permission to view projects' },
      { status: 403 }
    )
  }

  // Step 3: Check resource-specific permission
  const resourcePermission = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType: 'PROJECT',
    action: 'VIEW',
    resourceId: projectId,
    context: {
      ip: request.headers.get('x-forwarded-for') || 'unknown',
      userAgent: request.headers.get('user-agent') || 'unknown',
      timestamp: new Date(),
    }
  })

  if (!resourcePermission.allowed) {
    // Check if user is the owner (ownership bypass)
    const project = await prisma.project.findFirst({
      where: { id: projectId, organizationId: user.organizationId }
    })

    if (!project) {
      return NextResponse.json({ error: 'Project not found' }, { status: 404 })
    }

    if (project.createdById !== user.id) {
      await logAccessDecision(user, 'PROJECT', 'VIEW', projectId, resourcePermission)
      return NextResponse.json(
        { error: 'You do not have permission to view this project' },
        { status: 403 }
      )
    }
  }

  // Step 4: Fetch and return project
  const project = await prisma.project.findFirst({
    where: { id: projectId, organizationId: user.organizationId },
    include: {
      createdBy: { select: { id: true, name: true, avatar: true } },
      blueprints: { select: { id: true, name: true, status: true } },
    }
  })

  if (!project) {
    return NextResponse.json({ error: 'Project not found' }, { status: 404 })
  }

  // Step 5: Filter response based on permissions
  // If user is VIEWER, remove sensitive fields
  const canEdit = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType: 'PROJECT',
    action: 'EDIT',
    resourceId: projectId,
  })

  const canManage = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType: 'PROJECT',
    action: 'MANAGE',
    resourceId: projectId,
  })

  return NextResponse.json({
    project: {
      ...project,
      // Include edit/manage capabilities in response for UI rendering
      _permissions: {
        canEdit: canEdit.allowed,
        canDelete: (await checkPermission({
          userId: user.id,
          organizationId: user.organizationId,
          resourceType: 'PROJECT',
          action: 'DELETE',
          resourceId: projectId,
        })).allowed,
        canManage: canManage.allowed,
        canConfigure: (await checkPermission({
          userId: user.id,
          organizationId: user.organizationId,
          resourceType: 'PROJECT',
          action: 'CONFIGURE',
          resourceId: projectId,
        })).allowed,
      }
    }
  })
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const session = await auth()
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { id: projectId } = await params
  const user = session.user as any

  // Check delete permission
  const permission = await checkPermission({
    userId: user.id,
    organizationId: user.organizationId,
    resourceType: 'PROJECT',
    action: 'DELETE',
    resourceId: projectId,
    context: {
      ip: request.headers.get('x-forwarded-for') || 'unknown',
      userAgent: request.headers.get('user-agent') || 'unknown',
      timestamp: new Date(),
    }
  })

  if (!permission.allowed) {
    await logAccessDecision(user, 'PROJECT', 'DELETE', projectId, permission)
    return NextResponse.json(
      { error: permission.reason || 'You do not have permission to delete this project' },
      { status: 403 }
    )
  }

  // Verify project exists and belongs to org
  const project = await prisma.project.findFirst({
    where: { id: projectId, organizationId: user.organizationId }
  })

  if (!project) {
    return NextResponse.json({ error: 'Project not found' }, { status: 404 })
  }

  // For non-admin users, verify ownership
  if (user.role !== 'OWNER' && user.role !== 'ADMIN' && project.createdById !== user.id) {
    return NextResponse.json(
      { error: 'You can only delete your own projects' },
      { status: 403 }
    )
  }

  // Soft delete
  await prisma.project.update({
    where: { id: projectId },
    data: { deletedAt: new Date() }
  })

  // Audit log
  await prisma.rBACAuditLog.create({
    data: {
      organizationId: user.organizationId,
      action: 'RESOURCE_DELETED',
      resourceType: 'Project',
      resourceId: projectId,
      performedById: user.id,
      previousValue: { name: project.name, status: project.status },
    }
  })

  return NextResponse.json({ success: true })
}
```

### Complete Integration: RBAC-Aware React Page

```tsx
// ============================================
// EXAMPLE: Full RBAC-Aware Dashboard Page
// ============================================
// File: app/(dashboard)/projects/page.tsx

'use client'

import { usePermission, useRole, useCapability, RoleGate, PermissionButton } from '@/lib/rbac/client'

export default function ProjectsPage() {
  const { role, isOwner, isAdmin, customRole } = useRole()
  const { hasPermission: canCreate } = usePermission('PROJECT:CREATE')
  const { hasPermission: canManage } = usePermission('PROJECT:MANAGE')
  const { hasPermission: canExport } = usePermission('EXPORT:EXECUTE')
  const { hasCapability: hasAI } = useCapability('ai.generate')

  return (
    <div>
      <header className="flex justify-between items-center">
        <h1>Projects</h1>

        <div className="flex gap-2">
          {/* Only show create button if user has permission */}
          <PermissionButton
            permission="PROJECT:CREATE"
            fallback="upgrade"
            upgradeMessage="Upgrade to create projects"
            onClick={() => router.push('/projects/new')}
          >
            <PlusIcon className="mr-2 h-4 w-4" />
            New Project
          </PermissionButton>

          {/* Export button - hidden for users without permission */}
          {canExport && (
            <Button variant="outline" onClick={handleExport}>
              <DownloadIcon className="mr-2 h-4 w-4" />
              Export
            </Button>
          )}

          {/* Admin-only bulk actions */}
          <RoleGate allowedRoles={['OWNER', 'ADMIN']}>
            <DropdownMenu>
              <DropdownMenuTrigger asChild>
                <Button variant="outline">
                  <MoreHorizontalIcon className="h-4 w-4" />
                </Button>
              </DropdownMenuTrigger>
              <DropdownMenuContent>
                <DropdownMenuItem onClick={handleBulkDelete}>
                  Bulk Delete
                </DropdownMenuItem>
                <DropdownMenuItem onClick={handleBulkArchive}>
                  Bulk Archive
                </DropdownMenuItem>
                <DropdownMenuItem onClick={handleImport}>
                  Import Projects
                </DropdownMenuItem>
              </DropdownMenuContent>
            </DropdownMenu>
          </RoleGate>
        </div>
      </header>

      {/* Project list */}
      <ProjectList
        projects={projects}
        renderActions={(project) => (
          <div className="flex gap-1">
            {/* View - always available if they can see the list */}
            <Button size="sm" variant="ghost" onClick={() => viewProject(project.id)}>
              <EyeIcon className="h-4 w-4" />
            </Button>

            {/* Edit - based on permission + ownership */}
            {(project._permissions.canEdit || project.createdById === userId) && (
              <Button size="sm" variant="ghost" onClick={() => editProject(project.id)}>
                <EditIcon className="h-4 w-4" />
              </Button>
            )}

            {/* AI Generate - based on capability */}
            {hasAI && project._permissions.canEdit && (
              <Button size="sm" variant="ghost" onClick={() => generateAI(project.id)}>
                <SparklesIcon className="h-4 w-4" />
              </Button>
            )}

            {/* Delete - based on permission + ownership */}
            {project._permissions.canDelete && (
              <Button size="sm" variant="ghost" className="text-red-500"
                onClick={() => deleteProject(project.id)}>
                <TrashIcon className="h-4 w-4" />
              </Button>
            )}

            {/* Manage - admin only */}
            {project._permissions.canManage && (
              <Button size="sm" variant="ghost" onClick={() => manageProject(project.id)}>
                <SettingsIcon className="h-4 w-4" />
              </Button>
            )}
          </div>
        )}
      />

      {/* Empty state varies by role */}
      {projects.length === 0 && (
        <EmptyState
          title={canCreate ? "No projects yet" : "No projects available"}
          description={canCreate
            ? "Create your first project to get started"
            : "You don't have access to any projects yet. Contact your admin."
          }
          action={canCreate ? (
            <Button onClick={() => router.push('/projects/new')}>
              Create Project
            </Button>
          ) : null}
        />
      )}
    </div>
  )
}
```

---

## Migration & Versioning

### RBAC Configuration Versioning

```typescript
interface RBACVersioning {
  // ============================================
  // VERSION TRACKING
  // ============================================
  versioning: {
    // Every RBAC configuration change creates a version
    versionFormat: "semver"               // 1.0.0, 1.1.0, 2.0.0
    autoVersion: boolean                  // Auto-increment on change
    
    // Version types
    major: "Breaking changes (role deletion, permission model change)"
    minor: "New roles, new permissions, new capabilities"
    patch: "Permission value changes, condition updates"

    // Rollback
    rollback: {
      enabled: boolean
      maxVersionsToKeep: number           // 50
      canRollbackTo: "any" | "previous_only"
      requireOwnerApproval: boolean
      createBackupBeforeRollback: boolean
    }
  }

  // ============================================
  // MIGRATION SCENARIOS
  // ============================================
  migrations: {
    // Scenario 1: Adding new resource type
    addResourceType: {
      steps: [
        "1. Add new ResourceType to enum",
        "2. Run database migration",
        "3. Create default permissions for new resource type",
        "4. Update permission matrix with defaults for all roles",
        "5. Notify admins of new resource type",
        "6. Update UI to show new resource in matrix"
      ]
    }

    // Scenario 2: Splitting a role into two
    splitRole: {
      steps: [
        "1. Create new custom role with subset of permissions",
        "2. Identify users who should move to new role",
        "3. Bulk reassign users (with notification)",
        "4. Remove excess permissions from original role",
        "5. Audit log captures all changes"
      ]
    }

    // Scenario 3: Merging two custom roles
    mergeRoles: {
      steps: [
        "1. Create new role with union of both roles' permissions",
        "2. Reassign all users from both roles to new role",
        "3. Delete old roles",
        "4. Update any references (templates, policies)",
        "5. Audit log captures merge"
      ]
    }

    // Scenario 4: Migrating from simple role-based to full RBAC
    migrateToFullRBAC: {
      steps: [
        "1. Map existing role checks to permission checks",
        "2. Create permissions for each existing role",
        "3. Deploy permission checking middleware",
        "4. Run in audit_only mode for 1 week",
        "5. Review audit logs for unexpected denials",
        "6. Switch to strict enforcement",
        "7. Monitor for issues"
      ]
    }
  }

  // ============================================
  // EXPORT/IMPORT FORMAT
  // ============================================
  exportFormat: {
    version: string                       // "1.0.0"
    exportedAt: Date
    exportedBy: string
    organizationId: string

    data: {
      systemRolePermissions: {
        role: UserRole
        permissions: {
          resourceType: ResourceType
          action: ActionType
          effect: PermissionEffect
          conditions?: any
        }[]
      }[]

      customRoles: {
        name: string
        description: string
        baseRole: UserRole
        permissions: {
          resourceType: ResourceType
          action: ActionType
          effect: PermissionEffect
          conditions?: any
          uiProperties?: any
        }[]
        restrictions?: any
      }[]

      capabilities: {
        name: string
        slug: string
        description: string
        category: string
        permissions: any[]
      }[]

      policies: {
        name: string
        description: string
        priority: number
        target: any
        condition: any
        effect: string
        obligations?: any
      }[]

      templates: {
        name: string
        description: string
        category: string
        permissions: any[]
      }[]
    }

    metadata: {
      totalRoles: number
      totalPermissions: number
      totalCapabilities: number
      totalPolicies: number
      totalTemplates: number
      checksum: string                    // For integrity verification
    }
  }
}
```

---

## Summary

This comprehensive RBAC, Roles & Capabilities system provides:

1. **4 System Roles** (OWNER, ADMIN, MEMBER, VIEWER) with configurable permissions
2. **Custom Roles** per organization with inheritance, restrictions, and auto-assignment
3. **Granular Permissions** at resource-type, resource-specific, and UI-element levels
4. **Capabilities** as named permission groups for easy assignment
5. **ABAC Policies** for attribute-based conditions (time, IP, geo, device)
6. **Permission Matrix** for visual management of all permissions
7. **Permission Templates** for quick setup and standardization
8. **Delegation & Impersonation** for advanced access management
9. **Full Audit Trail** for every permission change and access decision
10. **Real-time Enforcement** with caching and instant invalidation
11. **Multi-tenant Isolation** ensuring no cross-organization leakage
12. **Plan-based Restrictions** tying RBAC features to subscription plans
13. **User Self-Service** for viewing permissions and requesting changes
14. **Simulation Tools** for testing permission changes before applying
15. **Export/Import** for RBAC configuration portability
16. **Edge Case Handling** for every conflict, race condition, and boundary scenario

Every aspect is **admin-configurable without code changes** and **enforced at API, middleware, and UI levels**.
