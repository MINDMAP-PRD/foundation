# BlueprintForge Admin Capabilities

> **Foundation wiring note:** Treat this file as comprehensive `GeneralApp` admin capability input. Its controls should resolve through shared foundation registries, settings, owner-config surfaces, and RBAC/capability enforcement rather than module-local hardcoding.

## Overview
This document outlines all the administrative capabilities available in the BlueprintForge application.

---

## 1. RBAC (Role-Based Access Control) Management

### Access
- **URL**: `/admin/rbac`
- **Permissions**: OWNER, ADMIN

### Features

#### Custom Roles
- Create custom roles beyond the default 4 (OWNER, ADMIN, MEMBER, VIEWER)
- Assign base role inheritance
- View role usage statistics (users count, permissions count)
- Edit and delete custom roles

#### Granular Permissions
Create permissions with the following properties:
- **Resource Type**: BUTTON, LINK, MENU_ITEM, TAB, CARD, FORM_FIELD, DIALOG, SIDEBAR_SECTION, PROJECT, MINDMAP, AI_GENERATION, EXPORT, IMPORT, COLLABORATION, USER_MANAGEMENT, BILLING, SETTINGS, SYSTEM_CONFIG, ORGANIZATION_MANAGEMENT
- **Action Type**: VIEW, CREATE, EDIT, DELETE, EXECUTE, CONFIGURE, MANAGE
- **Effect**: ALLOW or DENY
- **Role Assignment**: Assign to default roles or custom roles
- **Resource ID**: Optional specific resource targeting
- **Conditions**: JSON conditions (plan, time range, days of week, feature flags)
- **UI Properties**: JSON configuration for visibility, styling, text, etc.
- **Bulk Updates**: Update multiple permissions at once

#### Audit Logging
- All permission changes are logged
- Track who made changes and when
- View before/after values

### API Endpoints
```
GET    /api/admin/rbac/permissions         # List permissions
POST   /api/admin/rbac/permissions         # Create permission
PUT    /api/admin/rbac/permissions         # Bulk update
GET    /api/admin/rbac/permissions/[id]    # Get single
PATCH  /api/admin/rbac/permissions/[id]    # Update
DELETE /api/admin/rbac/permissions/[id]    # Delete

GET    /api/admin/rbac/roles               # List custom roles
POST   /api/admin/rbac/roles               # Create role
GET    /api/admin/rbac/roles/[id]          # Get role with permissions
PATCH  /api/admin/rbac/roles/[id]          # Update role
DELETE /api/admin/rbac/roles/[id]          # Delete role
```

---

## 2. UI Configuration Management

### Access
- **URL**: `/admin/ui-config`
- **Permissions**: OWNER, ADMIN

### Features

#### Element Configuration
Configure individual UI elements with:
- **Typography**: Font family, size, weight, line height, letter spacing
- **Colors**: Primary, secondary, background, text, border colors
- **Layout**: Padding, margin, width, height, display
- **Content**: Text, label, placeholder, tooltip, icon
- **State**: Visible, disabled, readOnly, required
- **Custom**: Custom CSS styles, custom CSS classes
- **Priority**: Higher priority configurations override lower ones
- **Scope**: global, organization, section, group, element

#### Element Groups
- Group related elements for shared styling
- Apply styles to all elements in a group
- Manage element membership

#### UI Sections
Configure page sections (sidebar, header, main content):
- **Layout**: Grid, flex, list configurations
- **Styling**: Background color, text color, padding
- **Element Ordering**: Define the order of elements
- **Role Visibility**: Control which roles can see each section

#### Global Theme
- Set global font family, sizes, weights
- Configure primary, secondary, and background colors
- Apply organization-wide styling

### API Endpoints
```
GET    /api/admin/ui/config                # List configurations
POST   /api/admin/ui/config                # Create configuration
PUT    /api/admin/ui/config                # Bulk update
GET    /api/admin/ui/config/[id]           # Get single
PATCH  /api/admin/ui/config/[id]           # Update
DELETE /api/admin/ui/config/[id]           # Delete

GET    /api/admin/ui/groups                # List groups
POST   /api/admin/ui/groups                # Create group
GET    /api/admin/ui/groups/[id]           # Get group
PATCH  /api/admin/ui/groups/[id]           # Update group
DELETE /api/admin/ui/groups/[id]           # Delete group

GET    /api/admin/ui/sections              # List sections
POST   /api/admin/ui/sections              # Create section
GET    /api/admin/ui/sections/[id]         # Get section
PATCH  /api/admin/ui/sections/[id]         # Update section
DELETE /api/admin/ui/sections/[id]         # Delete section

GET    /api/ui/config                      # Get current user configs
```

---

## 3. Platform Administration

### Access
- **URL**: `/admin`
- **Permissions**: OWNER (full access), ADMIN (limited)

### Features

#### Organizations Management (OWNER only)
- View all organizations on the platform
- Search and filter organizations
- Manage organization plans (FREE, STARTER, PRO, ENTERPRISE)
- Suspend/unsuspend organizations
- View user and project counts per organization

#### System Settings (OWNER only)
Configure global system settings:
- **Authentication**: Enable/disable Google, GitHub, Microsoft sign-in
- **Registration**: Enable/disable new signups
- **Email Verification**: Require email verification
- **Maintenance Mode**: Enable/disable maintenance mode
- **AI Configuration**: Default provider, rate limits
- **Feature Flags**: Real-time collaboration, exports, public sharing

#### System Health
- Monitor service status (API, Database, AI Providers, Email, Storage)
- View uptime statistics

### API Endpoints
```
GET    /api/admin/organizations            # List all organizations
PATCH  /api/admin/organizations            # Update organization
GET    /api/admin/settings                 # Get system settings
PATCH  /api/admin/settings                 # Update system settings
```

---

## 4. Permission Enforcement

### Client-Side Components
All UI components respect RBAC permissions:

```tsx
// Guard component - only renders if permission granted
<RBACGuard resourceType={ResourceType.BUTTON} action={ActionType.EXECUTE}>
  <Button>Action</Button>
</RBACGuard>

// RBAC Button with dynamic properties
<RBACButton 
  resourceType={ResourceType.BUTTON}
  action={ActionType.EXECUTE}
  resourceId="btn-create-project"
>
  Create Project
</RBACButton>

// Menu items
<RBACMenuItem resourceType={ResourceType.MENU_ITEM} action={ActionType.VIEW}>
  Menu Option
</RBACMenuItem>

// Conditional rendering
<RBACIf resourceType={ResourceType.CARD} action={ActionType.VIEW}>
  <Card>Content</Card>
</RBACIf>
```

### Server-Side Enforcement
```typescript
// Require permission or throw error
await requirePermission({
  userId,
  organizationId,
  role,
  resourceType: ResourceType.PROJECT,
  action: ActionType.DELETE,
  resourceId: projectId,
});

// Check permission
const result = await checkPermission({...});
if (result.allowed) {
  // Access granted
}

// Get UI properties
const uiProps = await getElementUIProps({...});
// Returns: { visible, disabled, color, text, etc. }
```

---

## 5. Default Permission Templates

The system includes default permission templates for each role:

### OWNER
- Full access to all resources and actions
- Can manage system configuration
- Can manage organizations

### ADMIN
- Full access except system configuration
- Can manage users and billing
- Can create/edit/delete projects and mind maps

### MEMBER
- Can create and edit projects/mind maps
- Can use AI generation
- Cannot access user management, billing, or settings

### VIEWER
- Read-only access to projects and mind maps
- Can export content
- Cannot create, edit, or delete

---

## 6. Database Schema Additions

### UIConfiguration Model
```prisma
model UIConfiguration {
  id              String   @id @default(cuid())
  organizationId  String?
  scope           String   @default("global")
  targetId        String?
  
  // Typography
  fontFamily      String?
  fontSize        String?
  fontWeight      String?
  lineHeight      String?
  letterSpacing   String?
  
  // Colors
  primaryColor    String?
  secondaryColor  String?
  backgroundColor String?
  textColor       String?
  borderColor     String?
  
  // Layout
  padding         String?
  margin          String?
  width           String?
  height          String?
  display         String?
  
  // Content
  text            String?
  label           String?
  placeholder     String?
  tooltip         String?
  icon            String?
  
  // State
  visible         Boolean  @default(true)
  disabled        Boolean  @default(false)
  readOnly        Boolean  @default(false)
  required        Boolean  @default(false)
  
  // Custom
  customStyles    Json?
  customClasses   String[]
  
  // Metadata
  permissionId    String?
  description     String?
  isActive        Boolean  @default(true)
  priority        Int      @default(0)
  
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  createdById     String
}
```

---

## Summary

As an **OWNER** or **ADMIN**, you now have complete control over:

1. **Who can do what** - Granular RBAC with custom roles and permissions
2. **What users see** - UI configuration for visibility and access
3. **How it looks** - Fonts, colors, styling at element, group, and section levels
4. **What it says** - Customizable text, labels, tooltips, placeholders
5. **Platform management** - Organization management and system settings

All configurations are:
- Multi-tenant (organization-scoped)
- Auditable (change tracking)
- Hierarchical (global → organization → section → group → element)
- Conditional (based on plan, time, features, roles)
