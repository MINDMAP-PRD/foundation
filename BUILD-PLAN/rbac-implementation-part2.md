# RBAC Implementation - Part 2: Database, API & Code Examples

> **Foundation wiring note:** Treat this as shared persistence/API guidance for the foundation authorization layer and align it with the canonical capability, interface, and settings registries.

## Database Schema

### Complete RBAC Database Schema

```sql
-- ============================================
-- USERS TABLE (Extended for RBAC)
-- ============================================
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Basic Info
  email VARCHAR(255) UNIQUE NOT NULL,
  username VARCHAR(50) UNIQUE,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  display_name VARCHAR(100),
  avatar_url TEXT,

  -- Authentication
  password_hash TEXT,
  email_verified BOOLEAN DEFAULT false,
  phone_verified BOOLEAN DEFAULT false,

  -- OAuth/Social
  google_id VARCHAR(255) UNIQUE,
  google_email VARCHAR(255),
  google_linked_at TIMESTAMP,

  github_id VARCHAR(255) UNIQUE,
  github_username VARCHAR(255),
  github_linked_at TIMESTAMP,

  apple_id VARCHAR(255) UNIQUE,
  apple_email VARCHAR(255),
  apple_linked_at TIMESTAMP,

  microsoft_id VARCHAR(255) UNIQUE,
  microsoft_email VARCHAR(255),
  microsoft_linked_at TIMESTAMP,

  facebook_id VARCHAR(255) UNIQUE,
  facebook_email VARCHAR(255),
  facebook_linked_at TIMESTAMP,

  linkedin_id VARCHAR(255) UNIQUE,
  linkedin_email VARCHAR(255),
  linkedin_linked_at TIMESTAMP,

  twitter_id VARCHAR(255) UNIQUE,
  twitter_username VARCHAR(255),
  twitter_linked_at TIMESTAMP,

  -- Primary auth method
  primary_auth_method VARCHAR(50) DEFAULT 'email_password',
  has_password_set BOOLEAN DEFAULT false,

  -- Phone
  phone VARCHAR(20),
  phone_country_code VARCHAR(5),

  -- Status
  status VARCHAR(20) DEFAULT 'active',
  account_state VARCHAR(20) DEFAULT 'pending_verification',

  -- Security
  mfa_enabled BOOLEAN DEFAULT false,
  risk_score DECIMAL(5,2) DEFAULT 0,

  -- Activity
  last_login_at TIMESTAMP,
  last_login_ip INET,
  last_login_device TEXT,
  last_login_location JSONB,
  login_count INTEGER DEFAULT 0,

  -- Engagement
  engagement_score DECIMAL(5,2) DEFAULT 0,

  -- Metadata
  metadata JSONB DEFAULT '{}',

  -- Timestamps
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP,

  CONSTRAINT valid_status CHECK (status IN ('active', 'inactive', 'suspended', 'banned', 'deleted')),
  CONSTRAINT valid_primary_auth CHECK (primary_auth_method IN ('email_password', 'google', 'github', 'apple', 'microsoft', 'facebook', 'linkedin', 'twitter', 'passwordless', 'saml'))
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_users_google_id ON users(google_id);
CREATE INDEX idx_users_github_id ON users(github_id);

-- ============================================
-- ROLES TABLE
-- ============================================
CREATE TABLE roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  level INTEGER NOT NULL DEFAULT 999,
  inherits_from UUID[] DEFAULT '{}',
  type VARCHAR(20) DEFAULT 'custom',
  user_limit INTEGER,
  is_default BOOLEAN DEFAULT false,
  is_system BOOLEAN DEFAULT false,
  color VARCHAR(7),
  icon VARCHAR(50),
  badge VARCHAR(50),
  conditions JSONB DEFAULT '{}',
  auto_assignment_rules JSONB DEFAULT '[]',
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES users(id),

  CONSTRAINT valid_type CHECK (type IN ('system', 'custom'))
);

CREATE INDEX idx_roles_slug ON roles(slug);
CREATE INDEX idx_roles_level ON roles(level);

-- ============================================
-- CAPABILITIES TABLE
-- ============================================
CREATE TABLE capabilities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  resource VARCHAR(100) NOT NULL,
  actions TEXT[] NOT NULL DEFAULT '{}',
  scope VARCHAR(20) DEFAULT 'own',
  type VARCHAR(20) DEFAULT 'custom',
  category VARCHAR(50),
  tags TEXT[] DEFAULT '{}',
  dangerous BOOLEAN DEFAULT false,
  conditions JSONB DEFAULT '{}',
  requires UUID[] DEFAULT '{}',
  conflicts UUID[] DEFAULT '{}',
  implies UUID[] DEFAULT '{}',
  rate_limiting JSONB DEFAULT '{}',
  approval_required BOOLEAN DEFAULT false,
  approvers UUID[] DEFAULT '{}',
  audit_enabled BOOLEAN DEFAULT true,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  created_by UUID REFERENCES users(id),

  CONSTRAINT valid_scope CHECK (scope IN ('own', 'team', 'organization', 'global'))
);

CREATE INDEX idx_capabilities_slug ON capabilities(slug);
CREATE INDEX idx_capabilities_resource ON capabilities(resource);

-- ============================================
-- ROLE_CAPABILITIES
-- ============================================
CREATE TABLE role_capabilities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  capability_id UUID REFERENCES capabilities(id) ON DELETE CASCADE,
  granted_at TIMESTAMP DEFAULT NOW(),
  granted_by UUID REFERENCES users(id),
  UNIQUE(role_id, capability_id)
);

-- ============================================
-- USER_ROLES
-- ============================================
CREATE TABLE user_roles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role_id UUID REFERENCES roles(id) ON DELETE CASCADE,
  assigned_at TIMESTAMP DEFAULT NOW(),
  assigned_by UUID REFERENCES users(id),
  expires_at TIMESTAMP,
  source VARCHAR(50) DEFAULT 'manual',
  metadata JSONB DEFAULT '{}',
  UNIQUE(user_id, role_id)
);

-- ============================================
-- MESSAGES TABLE
-- ============================================
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type VARCHAR(20) NOT NULL,
  channel VARCHAR(20) NOT NULL,
  subject VARCHAR(500),
  body TEXT,
  recipient_count INTEGER DEFAULT 0,
  recipient_filters JSONB,
  recipient_segment_ids UUID[],
  recipient_user_ids UUID[],
  template_id UUID,
  template_variables JSONB,
  sent_by UUID REFERENCES users(id),
  sent_at TIMESTAMP,
  status VARCHAR(20) DEFAULT 'draft',
  delivered_count INTEGER DEFAULT 0,
  opened_count INTEGER DEFAULT 0,
  clicked_count INTEGER DEFAULT 0,
  bounced_count INTEGER DEFAULT 0,
  failed_count INTEGER DEFAULT 0,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- ============================================
-- RBAC AUDIT LOG
-- ============================================
CREATE TABLE rbac_audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  action_type VARCHAR(50) NOT NULL,
  target_type VARCHAR(50) NOT NULL,
  target_id UUID,
  actor_id UUID REFERENCES users(id),
  actor_type VARCHAR(50) DEFAULT 'user',
  action VARCHAR(100) NOT NULL,
  resource VARCHAR(100),
  changes JSONB,
  reason TEXT,
  ip_address INET,
  user_agent TEXT,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_rbac_audit_log_actor ON rbac_audit_log(actor_id);
CREATE INDEX idx_rbac_audit_log_created_at ON rbac_audit_log(created_at);

-- ============================================
-- HELPER FUNCTIONS
-- ============================================
CREATE OR REPLACE FUNCTION get_user_effective_permissions(p_user_id UUID)
RETURNS TABLE (
  capability_id UUID,
  slug VARCHAR(100),
  resource VARCHAR(100),
  actions TEXT[],
  scope VARCHAR(20),
  source VARCHAR(50)
) AS $$
BEGIN
  RETURN QUERY
  SELECT
    c.id, c.slug, c.resource, c.actions, c.scope,
    'direct'::VARCHAR(50) as source
  FROM capabilities c
  JOIN user_capabilities uc ON c.id = uc.capability_id
  WHERE uc.user_id = p_user_id
    AND (uc.expires_at IS NULL OR uc.expires_at > NOW())
  UNION
  SELECT
    c.id, c.slug, c.resource, c.actions, c.scope,
    ('role:' || r.slug)::VARCHAR(50) as source
  FROM capabilities c
  JOIN role_capabilities rc ON c.id = rc.capability_id
  JOIN roles r ON rc.role_id = r.id
  JOIN user_roles ur ON r.id = ur.role_id
  WHERE ur.user_id = p_user_id
    AND (ur.expires_at IS NULL OR ur.expires_at > NOW());
END;
$$ LANGUAGE plpgsql;
```

See the complete document with API endpoints and code examples at:
**[comprehensive-roles-capabilities-rbac-flow.md](comprehensive-roles-capabilities-rbac-flow.md)**
