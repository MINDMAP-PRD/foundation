# ENTERPRISE ADMIN SYSTEM BUILDER - COMPREHENSIVE SPECIFICATION

## 🎯 PRE-BUILD ANALYSIS PHASE

**Before writing a single line of code, the AI must:**

1. **Classify the Project Type**
   - Analyze user's requirements and classify as: SaaS, CRM, E-commerce, Analytics Platform, Content Management, Marketplace, etc.
   - Define core business domain and primary user journeys
   - Identify all stakeholder types (Owner, Admins, End Users, API Consumers, etc.)

2. **Map Feature Dependencies**
   - Create a dependency tree of all features
   - Identify which features require which capabilities
   - Plan database relationships before any code

3. **Define Success Criteria**
   - What makes each feature "complete"?
   - What edge cases must be handled?
   - What are the rollback/fallback states?

---

## 👑 HIERARCHY & AUTHORITY SYSTEM

### Owner (God Mode)
```
STATUS: Seeded at system initialization
COUNT: Exactly 1 (immutable)
VISIBILITY: Invisible to all other users including admins
PERMISSIONS: Unrestricted access to all system capabilities
UNIQUE POWERS:
  - Create/delete admins
  - Access System Logs
  - Configure global RBAC policies
  - Enable/disable entire features
  - Manage MCP integrations
  - Override any permission
  - Access audit trails
  - Configure branding & themes
```

### Admin Roles
```
CREATOR: Only Owner can create admins
TYPES: Super Admin, Admin, Custom Roles (owner-defined)
PERMISSIONS: Defined by Owner via RBAC
ISOLATION: Cannot see Owner account
FALLBACK: If permission unclear → deny by default
```

### User Roles
```
DEFAULT ROLES: User, Guest, Custom (owner-defined)
VISIBILITY: Counted separately from Owner
PERMISSIONS: Granular RBAC per feature
COMMUNICATION: Ticket system, chatbot support
```

---

## 🔐 RBAC ARCHITECTURE (Role-Based Access Control)

### Granularity Levels

**1. NAVIGATION-LEVEL RBAC**
```yaml
Each Nav Item Controls:
  - Visibility (role-based, individual-based)
  - Access (read, write, delete)
  - Settings icon visibility (toggle by Owner)
  - Sub-navigation tabs configuration
  - Frontend rendering permissions
```

**2. FEATURE-LEVEL RBAC**
```yaml
Each Feature Controls:
  - CRUD operations (Create, Read, Update, Delete)
  - Individual vs Role-based permissions
  - Batch operations
  - Export capabilities
  - Audit trail access
```

**3. ELEMENT-LEVEL RBAC**
```yaml
Each UI Element Controls:
  - Visibility (show/hide)
  - Interactivity (enabled/disabled)
  - Styling (font size, color, position)
  - Text content (customizable labels)
  - Tooltips & help text
```

**4. DATA-LEVEL RBAC**
```yaml
Row-Level Security:
  - User can only see their own data
  - Manager sees team data
  - Admin sees org data
  - Owner sees everything
```

### RBAC Management Interface

**Two Access Points:**

1. **Per-Nav Settings Icon** (Category-based)
   - Click settings icon on any nav item
   - Configure RBAC for that category
   - Changes apply to related frontend components
   - Live preview of permission changes

2. **Centralized Settings → RBAC Tab** (System-wide)
   - Manage all nav RBAC in one place
   - Matrix view (Roles × Features)
   - Bulk permission updates
   - Permission templates
   - Copy permissions between roles

### Dynamic Capability Creation
```typescript
Admin Can Define:
  - Custom capabilities (e.g., "EXPORT_FINANCIAL_REPORTS")
  - Capability groups (bundle related permissions)
  - Conditional permissions (time-based, IP-based, etc.)
  - Permission inheritance (child roles inherit parent)
  - Permission conflicts (explicit deny overrides allow)
```

---

## 🔌 MCP INTEGRATION ARCHITECTURE (Model Context Protocol)

### Core Principle: **Capability-Driven, Provider-Agnostic**

```typescript
// ❌ NEVER DO THIS
if (emailProvider === "resend") {
  sendViaResend(email)
} else if (emailProvider === "sendgrid") {
  sendViaSendgrid(email)
}

// ✅ ALWAYS DO THIS
await invokeCapability("SEND_EMAIL", {
  to: email,
  subject: subject,
  body: body
})
```

### Architecture Layers

**Layer 1: Business Logic**
```typescript
// Application never knows about providers
class UserService {
  async sendWelcomeEmail(user: User) {
    await invokeCapability("SEND_EMAIL", {
      to: user.email,
      template: "welcome",
      data: { name: user.name }
    })
  }
}
```

**Layer 2: Capability Registry**
```typescript
interface CapabilityRegistry {
  SEND_EMAIL: EmailProvider[]
  SEND_SMS: SMSProvider[]
  SEND_PUSH: PushProvider[]
  PROCESS_PAYMENT: PaymentProvider[]
  STORE_FILE: StorageProvider[]
  GENERATE_TEXT: AIProvider[]
  TRACK_EVENT: AnalyticsProvider[]
  PLACE_CALL: VoiceProvider[]
  RESOLVE_LOCATION: MapsProvider[]
  SEND_WEBHOOK: WebhookProvider[]
  UPSERT_CONTACT: CRMProvider[]
}
```

**Layer 3: Provider Adapters**
```typescript
// Each provider implements standard interface
interface EmailProvider {
  name: string
  capabilities: Capability[]
  connect(config: ConnectionConfig): Promise<ProviderResult>
  sendEmail(payload: EmailPayload): Promise<ProviderResult>
  disconnect(): Promise<void>
}

// Example implementations
class ResendProvider implements EmailProvider {
  name = "Resend"
  capabilities = ["SEND_EMAIL"]
  
  async connect(config: OAuthConfig | APIKeyConfig | TokenConfig) {
    // Multi-auth support: OAuth, API Key, Bearer Token
  }
  
  async sendEmail(payload: EmailPayload): Promise<ProviderResult> {
    // Resend-specific implementation
  }
}

class SendGridProvider implements EmailProvider {
  name = "SendGrid"
  capabilities = ["SEND_EMAIL"]
  // Same interface, different implementation
}
```

**Layer 4: Connection Management**
```typescript
Connection Options Per Provider:
  - OAuth 2.0 flow
  - API Key authentication
  - Bearer Token
  - Username/Password (legacy)
  - Service Account (Google/AWS)
  - Webhook registration
  
Status States:
  - DISCONNECTED (initial)
  - CONNECTING (auth in progress)
  - CONNECTED (active and healthy)
  - ERROR (connection failed)
  - RATE_LIMITED (throttled)
  - SUSPENDED (manually disabled)
```

### MCP Management Dashboard

**Owner Interface: Integrations Management**
```
Tabs:
  - Active Connections
  - Available Providers
  - Capability Map
  - Usage Analytics
  - Connection Logs
  - Failover Rules

Features Per Provider:
  ├── Connection Status (real-time)
  ├── Connection Method (OAuth/API/Token)
  ├── Test Connection (live ping)
  ├── Usage Statistics (calls, errors, latency)
  ├── Rate Limits (current/max)
  ├── Cost Tracking (if applicable)
  ├── Logs (last 1000 operations)
  ├── Webhook Management (if applicable)
  ├── Reconnect (handles token refresh)
  └── Disconnect (graceful shutdown)
```

### Failover & Redundancy
```typescript
Failover Strategy:
  - Primary provider fails → Auto-switch to secondary
  - Log failover event immutably
  - Notify Owner of degraded state
  - Attempt primary reconnection in background
  
Example:
  SEND_EMAIL: [ResendProvider (primary), SendGridProvider (fallback)]
  
If Resend fails:
  1. Log error with full context
  2. Switch to SendGrid
  3. Send admin alert
  4. Retry Resend every 5 minutes
  5. Switch back when healthy
```

---

## 🎨 UI/UX ARCHITECTURE

### Design System

**Component Library: Shadcn UI or Material UI** (Owner configurable)
```
Principles:
  - Consistent spacing (4px/8px/16px/24px grid)
  - Accessible contrast ratios (WCAG AA minimum)
  - Responsive breakpoints (mobile/tablet/desktop)
  - Dark/light mode support
  - RTL layout support
```

### Layout Structure

```
┌─────────────────────────────────────────────────┐
│  TOPBAR (fixed, z-index: 1000)                  │
│  - Brand Logo/Name (configurable position)      │
│  - Search (global)                              │
│  - Notifications                                │
│  - User Menu                                    │
└─────────────────────────────────────────────────┘
┌─────────┬───────────────────────────────────────┐
│ SIDEBAR │  MAIN CONTENT AREA                    │
│ (toggle)│  ┌─────────────────────────────────┐  │
│         │  │  Breadcrumbs                    │  │
│ Nav 1   │  └─────────────────────────────────┘  │
│ Nav 2 ⚙│  ┌─────────────────────────────────┐  │
│ Nav 3   │  │  Page Tabs (horizontal)         │  │
│ Nav 4   │  │  [Tab 1] [Tab 2] [Tab 3]        │  │
│ Nav 5   │  └─────────────────────────────────┘  │
│         │  ┌─────────────────────────────────┐  │
│ ▼ More  │  │                                 │  │
│         │  │  PREVIEW PANE                   │  │
│ [Min]   │  │  (where modals render)          │  │
│         │  │                                 │  │
│         │  │                                 │  │
└─────────┴──┴─────────────────────────────────┴──┘

Mobile (< 768px):
  - Sidebar collapses
  - Hamburger menu (top-left)
  - Tabs scroll horizontally
  - Modals full-screen
```

### Navigation System

**Each Nav Item Must Have:**
```typescript
interface NavItem {
  id: string
  label: string (owner-configurable)
  icon: string (owner-configurable)
  route: string
  settingsEnabled: boolean (owner toggles)
  rbacConfig: RBACConfig
  tabs: NavTab[] // Sub-navigation INSIDE the page
  badge?: number // Real data count, never hardcoded
  order: number // Drag-to-reorder
}

interface NavTab {
  id: string
  label: string
  component: React.Component
  rbacConfig: RBACConfig
  order: number
}
```

**Example: Users Nav**
```
Users (Nav Item)
  ├── ⚙ Settings Icon (if enabled by Owner)
  │   └── Opens: Users RBAC Configuration Modal
  │
  └── Page Content
      ├── Tab: User List
      ├── Tab: Analytics
      ├── Tab: Activity Logs
      ├── Tab: Bulk Actions
      └── Tab: Settings
```

### Modal System

**Principles:**
- ❌ NO `window.alert()`, `window.confirm()`, `window.prompt()`
- ✅ ONLY custom modals with consistent styling
- Modals render in PREVIEW PANE, not as overlays
- Support for stacked modals (modal within modal)

**Modal Types:**
```typescript
- ConfirmModal (for destructive actions)
- FormModal (for data input)
- DetailModal (for viewing records)
- WizardModal (multi-step flows)
- FullScreenModal (for complex operations)
```

---

## 📊 DATA & DATABASE RULES

### SQL Schema Generation

**CRITICAL RULE: Code-DB Synchronization**
```
When code references:
  - A table → Generate SQL CREATE TABLE immediately
  - A column → Ensure column exists in schema
  - A foreign key → Create relationship in schema
  - A function → Create stored procedure/trigger
  
Zero tolerance for:
  - Calling non-existent tables
  - Referencing missing columns
  - Undefined foreign keys
  - Missing indexes on frequently queried fields
```

**Schema File Organization:**
```
/database
  /migrations
    001_create_users_table.sql
    002_create_roles_table.sql
    003_create_permissions_table.sql
    004_create_audit_logs_table.sql
    ...
  /seeds
    001_seed_owner.sql (creates THE owner)
    002_seed_default_roles.sql
    003_seed_capabilities.sql
  /triggers
    audit_log_trigger.sql
    updated_at_trigger.sql
  /views
    user_permissions_view.sql
    analytics_dashboard_view.sql
```

### No Hardcoded Data

**❌ NEVER:**
```tsx
<Card>
  <h3>Total Users</h3>
  <p>1,234</p> {/* WRONG! */}
</Card>
```

**✅ ALWAYS:**
```tsx
<Card>
  <h3>Total Users</h3>
  <p>{userCount ?? <Skeleton />}</p> {/* Fetched from DB */}
</Card>
```

**Every Number Must Be:**
- Fetched from database
- Updated in real-time (or near real-time)
- Cached appropriately
- Show loading state while fetching
- Show error state if fetch fails

---

## 🔄 STATE MACHINES & FLOW ARCHITECTURE

### Every Element Has a Complete Flow

**Flow Components:**
1. **States** - Discrete conditions (idle, loading, success, error)
2. **Branches** - Decision points (if/else logic)
3. **Transitions** - How to move between states
4. **Side Effects** - What happens during transitions

**Example: Delete User Flow**
```typescript
States:
  - IDLE (button visible, enabled)
  - CONFIRMING (modal open, awaiting confirmation)
  - DELETING (API call in progress, button disabled)
  - SUCCESS (user deleted, showing success toast)
  - ERROR (deletion failed, showing error modal)
  - ROLLED_BACK (if deletion failed, reverted changes)

Branches:
  - User clicks delete → IDLE → CONFIRMING
  - User confirms → CONFIRMING → DELETING
    ├── API success → DELETING → SUCCESS → IDLE
    └── API error → DELETING → ERROR
        ├── Retry → ERROR → DELETING
        └── Cancel → ERROR → IDLE

Side Effects:
  - On CONFIRMING: Log "user attempted delete"
  - On DELETING: Disable button, show spinner
  - On SUCCESS: Refresh user list, show toast, log action
  - On ERROR: Log error details, show retry option
  - On ROLLED_BACK: Restore previous state, log rollback
```

### Empty States & Error States

**Every List/Table Must Have:**
```tsx
interface ListStates {
  empty: React.Component // No data exists yet
  loading: React.Component // Fetching data
  error: React.Component // Fetch failed
  noResults: React.Component // Search returned nothing
  success: React.Component // Data loaded successfully
}

Example Empty State:
  - Illustration (SVG)
  - Clear message: "No users yet"
  - Action button: "Create First User"
  - Help text: "Users will appear here once added"
```

---

## ⚙️ SETTINGS SYSTEM

### Owner Settings (Comprehensive Tabs)

**Settings → Profile**
```
Owner Personal Info:
  - Name, Email, Phone
  - Avatar upload (with crop tool)
  - Password change (with strength meter)
  - Two-factor authentication (SMS/Authenticator App)
  - Session management (view active sessions, remote logout)
  - API token generation (for programmatic access)
```

**Settings → Branding**
```
Live Preview Panel (right side):
  - Shows changes in real-time
  - No "Save" button needed for preview
  - "Apply" button commits changes

Theme Presets:
  [Modern Blue] [Corporate Gray] [Vibrant Purple] [Dark Mode]
  - Click to apply instantly
  - Preview updates immediately

Custom Colors:
  ○ ○ ○ ○ ○ ○ ○ ○ (Color circles)
  - Click to apply
  - Color picker for custom hex
  - Saves to theme palette

Brand Assets:
  - Logo upload (SVG/PNG, max 2MB)
  - Favicon upload (ICO/PNG, 32×32)
  - Brand name (text input)
  - Tagline (optional)

Layout Controls:
  - Logo position (left, center, right)
  - Brand name position
  - Logo size slider
  - Font family selector

Typography:
  - Headings font (dropdown)
  - Body font (dropdown)
  - Monospace font (for code)
  - Font size scale (small, medium, large)

User Theme Control:
  ☐ Allow users to customize themes
  ☐ Force users to use admin-selected theme
  ☐ Allow dark/light mode toggle
```

**Settings → RBAC (Centralized)**
```
Matrix View:
           | Create | Read | Update | Delete | Export
-----------|--------|------|--------|--------|--------
Owner      |   ✓    |  ✓   |   ✓    |   ✓    |   ✓
Admin      |   ✓    |  ✓   |   ✓    |   ✓    |   ✓
User       |   ✓    |  ✓   |   ✓    |   ✗    |   ✗

Role Management:
  - Create custom roles
  - Clone existing roles
  - Define role hierarchy
  - Set role colors/icons
  - Assign default landing pages

Permission Templates:
  - "Read Only"
  - "Power User"
  - "Content Manager"
  - Save custom templates
```

**Settings → Integrations (MCP)**
```
Connected Services:
  [Resend Email] [Status: Active] [Reconnect] [Disconnect]
  [Stripe Payments] [Status: Active] [Configure] [Logs]
  [Twilio SMS] [Status: Error] [Reconnect] [View Error]

Available Integrations:
  - Browse marketplace
  - Search by capability
  - One-click connect (OAuth preferred)
  - View documentation
  - Test connection before activating

Capability Mapping:
  SEND_EMAIL → [Resend (Primary), SendGrid (Fallback)]
  PROCESS_PAYMENT → [Stripe (Primary)]
  SEND_SMS → [Twilio (Primary), Termii (Fallback)]
```

**Settings → Notifications**
```
Email Notifications:
  ☑ New user registrations
  ☑ Failed login attempts
  ☑ System errors
  ☐ Weekly usage reports

Push Notifications:
  ☑ Critical system alerts
  ☐ User support tickets
  
Digest Schedule:
  ○ Real-time
  ● Daily digest (9:00 AM)
  ○ Weekly digest (Monday 9:00 AM)
```

**Settings → Security**
```
Password Policies:
  - Minimum length: [8] characters
  - Require: ☑ Uppercase ☑ Lowercase ☑ Numbers ☑ Symbols
  - Expiry: [90] days
  - Prevent reuse: [5] previous passwords

Session Management:
  - Session timeout: [30] minutes
  - Max concurrent sessions: [3]
  - ☑ Require re-auth for sensitive actions

IP Whitelisting:
  - Add trusted IP ranges
  - Block suspicious IPs automatically
  - View login attempts by IP

API Security:
  - Rate limiting: [100] requests per minute
  - API key rotation policy
  - Webhook signature verification
```

**Settings → Logs & Audit**
```
Immutable Audit Trail:
  - All logs write-only (cannot be edited/deleted)
  - Tamper-proof timestamps
  - Cryptographic hashing for integrity

Log Categories:
  ├── Authentication Logs
  ├── RBAC Changes
  ├── Data Modifications
  ├── API Calls
  ├── Integration Events
  ├── System Errors
  └── Security Events

Log Detail View:
  Click any log entry to expand:
    - Timestamp (UTC + local)
    - User who triggered (with avatar)
    - Action performed
    - Resource affected
    - IP address & location
    - User agent (browser/device)
    - Request/response payload
    - Stack trace (if error)
    - Related events (clickable links)

Filters & Search:
  - Date range picker
  - User filter
  - Action type filter
  - Resource type filter
  - Severity level
  - Full-text search

Export Options:
  - CSV (for spreadsheets)
  - JSON (for processing)
  - PDF (for compliance reports)
```

**Settings → Billing (if applicable)**
```
Subscription Management:
  - Current plan details
  - Usage metrics
  - Upgrade/downgrade
  - Payment methods
  - Billing history
  - Invoices (download PDF)
```

---

## 📝 COMPREHENSIVE FEATURE IMPLEMENTATION

### Rule: NO Half-Built Features

**Example: Stripe Integration**

**❌ INCOMPLETE:**
```tsx
// Just showing a "Connect Stripe" button
<Button>Connect Stripe</Button>
```

**✅ COMPLETE:**
```
Stripe Management Dashboard:

Navigation:
  ├── Products & Pricing
  │   ├── Create Product
  │   ├── Edit Product
  │   ├── Delete Product
  │   └── Product List (with search/filter)
  │
  ├── Pricing Plans
  │   ├── Create Plan (monthly/yearly/one-time)
  │   ├── Edit Plan (price, features, limits)
  │   ├── Trial Period Configuration
  │   └── Plan Comparison Table
  │
  ├── Checkout Page Builder
  │   ├── Layout Editor (drag-drop components)
  │   ├── Color Customization (brand colors)
  │   ├── Text Customization (all labels)
  │   ├── Logo Upload
  │   ├── Custom Fields (collect extra data)
  │   ├── Success/Cancel URL Configuration
  │   └── Live Preview Panel
  │
  ├── Subscriptions
  │   ├── Active Subscriptions List
  │   ├── Subscription Details View
  │   ├── Cancel Subscription (with retention flow)
  │   ├── Upgrade/Downgrade Flow
  │   ├── Billing Portal Link
  │   └── Subscription Analytics
  │
  ├── Payments
  │   ├── Transaction History
  │   ├── Refund Management
  │   ├── Failed Payments (with retry)
  │   └── Payment Analytics
  │
  ├── Webhooks
  │   ├── Webhook Endpoint Configuration
  │   ├── Event Subscriptions (which events to listen)
  │   ├── Webhook Logs (all events received)
  │   ├── Retry Failed Webhooks
  │   └── Test Webhook Delivery
  │
  ├── Tax Settings
  │   ├── Tax Rates by Region
  │   ├── Automatic Tax Calculation
  │   └── Tax Exempt Customers
  │
  ├── Coupons & Discounts
  │   ├── Create Coupon
  │   ├── Coupon Types (%, fixed, free trial)
  │   ├── Usage Limits
  │   └── Coupon Analytics
  │
  └── Settings
      ├── API Keys (publishable & secret)
      ├── Test Mode Toggle
      ├── Currency Settings
      ├── Statement Descriptor
      └── Email Receipt Customization
```

**Every sub-feature has:**
- CRUD operations
- Validation (frontend + backend)
- Error handling
- Loading states
- Empty states
- Success feedback
- Audit logging
- RBAC enforcement

---

## 👥 USER MANAGEMENT

### Users Dashboard (Comprehensive)

**Tab: User List**
```
Features:
  - Searchable/filterable table
  - Bulk actions (delete, suspend, change role)
  - Export (CSV, Excel)
  - Column customization (show/hide columns)
  - Advanced filters:
    ├── Role
    ├── Status (active/suspended/pending)
    ├── Registration date
    ├── Last login
    └── Custom fields

Row Actions:
  - View Details
  - Edit User
  - Change Role
  - Reset Password
  - Suspend Account
  - Delete Account (with confirmation)
  - View Activity Log
  - Impersonate User (admin testing)
```

**Tab: Analytics**
```
Metrics:
  ├── Total Users (excluding Owner)
  ├── Active Users (last 30 days)
  ├── New Registrations (this month)
  ├── Churn Rate
  ├── User Growth Chart
  ├── User Distribution by Role (pie chart)
  ├── Geographic Distribution (map)
  └── User Engagement Score

Filters:
  - Date range
  - User segment
  - Comparison periods
```

**Tab: Activity Logs**
```
Per-User Activity Stream:
  - Login/logout events
  - Feature usage
  - API calls made
  - Errors encountered
  - Support tickets created
  - Settings changed

Filterable by:
  - Activity type
  - Date range
  - Severity
```

**Tab: Bulk Actions**
```
Operations:
  - Import users (CSV upload)
  - Export users (CSV/Excel)
  - Bulk role assignment
  - Bulk email (with template)
  - Bulk password reset
  - Bulk account suspension
```

**Tab: Settings**
```
Registration:
  ☐ Allow public registration
  ☑ Require email verification
  ☐ Require admin approval
  ☑ Enable CAPTCHA

User Defaults:
  - Default role for new users
  - Default permissions
  - Welcome email template
  - Default landing page

User Fields:
  - Required fields (name, email, etc.)
  - Optional fields (phone, company, etc.)
  - Custom fields (add your own)
```

### User Communication System

**Support Ticket System**
```
User Side:
  - Create ticket (form with categories)
  - Upload attachments
  - View ticket status
  - Reply to admin responses
  - Close ticket

Admin Side:
  - Ticket queue (inbox)
  - Assign tickets to admins
  - Priority levels (low, medium, high, urgent)
  - Status workflow (new → assigned → in progress → resolved → closed)
  - Canned responses (templates)
  - Internal notes (not visible to user)
  - Ticket analytics (response time, resolution time, satisfaction)
```

**Chatbot Interface**
```
Implementation:
  - AI-powered chatbot (using GENERATE_TEXT capability)
  - Handles common questions
  - Escalates to human if needed
  - Learns from interactions
  - Available 24/7

Admin Configuration:
  - Training data upload
  - Escalation rules
  - Greeting messages
  - Fallback responses
  - Conversation logs
```

---

## 📱 MOBILE RESPONSIVENESS

### Breakpoints
```css
Mobile: < 768px
Tablet: 768px - 1024px
Desktop: > 1024px
```

### Mobile-Specific Behavior

**Sidebar:**
```
Desktop: Always visible (with minimize option)
Mobile: Hidden by default
  - Hamburger icon (top-left) toggles sidebar
  - Sidebar slides in from left
  - Backdrop overlay when open
  - Swipe to close
```

**Buttons:**
```
Rule: Text must NEVER wrap mid-word

If button text too long:
  ❌ WRONG: "Submit     ← wrapped text
             Application"
  
  ✅ RIGHT: Wrap entire button to new line
  [  Submit Application  ]  ← whole button wraps
```

**Tables:**
```
Desktop: Traditional table
Mobile: Card-based layout
  - Each row becomes a card
  - Columns stack vertically
  - Most important info at top
  - Actions at bottom
```

**Modals:**
```
Desktop: Centered overlay (max-width: 600px)
Mobile: Full-screen
  - Header with close button
  - Scrollable content
  - Fixed footer (if actions present)
```

---

## 🔍 LOGGING SYSTEM

### Immutable Audit Logs

**Principle: Write-Only, Never Delete**
```sql
CREATE TABLE audit_logs (
  id BIGSERIAL PRIMARY KEY,
  timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  user_id UUID, -- NULL for system events
  action VARCHAR(255) NOT NULL,
  resource_type VARCHAR(100),
  resource_id VARCHAR(255),
  ip_address INET,
  user_agent TEXT,
  request_payload JSONB,
  response_payload JSONB,
  status VARCHAR(50), -- success, error, warning
  error_message TEXT,
  stack_trace TEXT,
  metadata JSONB, -- extensible
  hash VARCHAR(64) NOT NULL -- SHA-256 for tamper detection
);

-- Prevent updates/deletes
CREATE RULE audit_log_no_update AS ON UPDATE TO audit_logs DO INSTEAD NOTHING;
CREATE RULE audit_log_no_delete AS ON DELETE TO audit_logs DO INSTEAD NOTHING;
```

### What to Log

**Authentication:**
- Login attempts (success/failure)
- Logout events
- Password changes
- 2FA enable/disable
- Session expirations

**RBAC:**
- Permission changes
- Role assignments
- Access denied events

**Data Operations:**
- Create, update, delete on any entity
- Bulk operations
- Imports/exports

**Integration Events:**
- MCP connection/disconnection
- API calls to external services
- Webhook deliveries
- Failover events

**System Events:**
- Application startup/shutdown
- Database migrations
- Cron job executions
- Background task completions

**Errors:**
- Uncaught exceptions
- Failed API calls
- Database query errors
- Validation failures
- Rate limit exceeded

**Security:**
- Suspicious activity (multiple failed logins)
- IP whitelist violations
- API key usage
- Privilege escalation attempts

### Log Management Dashboard

**System Logs Tab:**
```
Features:
  - Real-time log streaming
  - Search & filter
  - Log level filter (debug, info, warn, error)
  - Date range picker
  - User filter
  - Resource filter
  - Export logs

Detail View (click any log):
  ┌─────────────────────────────────────────┐
  │ [ERROR] Failed to send email            │
  │ ─────────────────────────────────────── │
  │ Timestamp: 2025-02-14 10:23:45 UTC      │
  │ User: john@example.com (ID: abc123)     │
  │ IP: 192.168.1.1 (Rotterdam, NL)         │
  │ Browser: Chrome 121 on macOS            │
  │ ─────────────────────────────────────── │
  │ Triggered By:                           │
  │   User attempted to send welcome email  │
  │   to new user registration              │
  │ ─────────────────────────────────────── │
  │ Why It Failed:                          │
  │   Resend API returned 429 (rate limit)  │
  │   Daily quota exceeded (5000/5000)      │
  │ ─────────────────────────────────────── │
  │ Request Payload:                        │
  │   { to: "newuser@...", template: ... }  │
  │ ─────────────────────────────────────── │
  │ Stack Trace:                            │
  │   [Expandable stack trace here]         │
  │ ─────────────────────────────────────── │
  │ Related Events:                         │
  │   → View user registration event        │
  │   → View Resend connection logs         │
  │ ─────────────────────────────────────── │
  │ Actions:                                │
  │   [Retry Operation] [Contact Support]   │
  └─────────────────────────────────────────┘
```

**Connection Logs Tab:**
```
For each integration:
  - Connection attempts
  - Authentication success/failure
  - Rate limit warnings
  - Downtime events
  - Failover activations
```

**Performance Logs Tab:**
```
Metrics:
  - API response times
  - Database query performance
  - Slow endpoints
  - Memory usage
  - CPU usage
```

---

## 🎯 CRITICAL SUCCESS CRITERIA

### Before Marking ANY Element "Complete"

**Checklist:**
```
☐ 1. Purpose defined - Why does this exist?
☐ 2. Dependencies implemented - All required functions exist
☐ 3. Admin configuration - Full management dashboard exists
☐ 4. RBAC applied - Permissions control every aspect
☐ 5. Flow documented - States, branches, transitions mapped
☐ 6. Error handling - All error cases covered
☐ 7. Empty states - "No data" UI designed
☐ 8. Loading states - Skeleton screens/spinners
☐ 9. Success states - Confirmation feedback
☐ 10. SQL schema - All tables/columns exist
☐ 11. Validation - Frontend + backend
☐ 12. Audit logging - All actions logged
☐ 13. Mobile responsive - Works on all screen sizes
☐ 14. Real data - No hardcoded numbers
☐ 15. Modals (not alerts) - Custom UI for all confirmations
☐ 16. Testing - CRUD operations verified working
```

---

## 🚀 IMPLEMENTATION SEQUENCE

### Build Order (Strict)

**Phase 1: Foundation**
1. Database schema (with owner seed)
2. Authentication system
3. RBAC infrastructure
4. Audit logging system

**Phase 2: Core Admin**
5. Owner dashboard
6. Admin management
7. User management
8. Settings system

**Phase 3: Features**
9. MCP capability registry
10. First integration (email)
11. Navigation system
12. First feature dashboard

**Phase 4: Polish**
13. Mobile responsive
14. Theme system
15. Analytics
16. Documentation

---

This specification ensures **zero shortcuts**, **complete implementations**, and **enterprise-grade quality** from day one. Every pixel, every query, every flow is production-ready.         ``