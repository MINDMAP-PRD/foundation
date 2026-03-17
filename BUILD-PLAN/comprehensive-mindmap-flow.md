# Comprehensive Mindmap System Configuration Flow

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Mindmap Generation Engine](#2-mindmap-generation-engine)
3. [Map, Tree, Branch, and State Structure](#3-map-tree-branch-and-state-structure)
4. [Map Refactoring & Impact Analysis](#4-map-refactoring--impact-analysis)
5. [Export & Download System](#5-export--download-system)
6. [AI Integration & Building](#6-ai-integration--building)
7. [Collaboration & Access Control](#7-collaboration--access-control)
8. [AI Prompt Configuration](#8-ai-prompt-configuration)
9. [Auto-Generation & Exhaustion Handling](#9-auto-generation--exhaustion-handling)
10. [Project Management](#10-project-management)
11. [GitHub Integration](#11-github-integration)
12. [API Routes](#12-api-routes)
13. [Database Schema](#13-database-schema)
14. [Edge Cases](#14-edge-cases)
15. [Performance & Caching](#15-performance--caching)

---

## 1. System Overview

### 1.1 Core Principles

The Mindmap System in BlueprintForge transforms user ideas into comprehensive, multi-dimensional blueprints with intelligent branching, state management, and AI-powered generation. Each idea spawns a hierarchical structure of maps, trees, branches, and states that can be explored, edited, exported, and used to generate actual project code.

**Core Design Principles:**

- **Hierarchical Structure** - Ideas decompose into Maps → Trees → Branches → States
- **AI-Powered Generation** - Automatic expansion of ideas into comprehensive blueprints
- **Real-Time Refactoring** - Changes propagate through entire project structure
- **Multi-Format Export** - Export individual elements or entire projects
- **AI Building Integration** - Send maps/trees to Lovable, Base44, or custom AI builders
- **Collaboration** - Invite team members with granular permissions
- **Prompt Governance** - Admin-configurable AI rules that constrain AI behavior

### 1.2 System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      MINDMAP SYSTEM ARCHITECTURE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │
│  │   Idea      │    │   AI        │    │   Manual    │                    │
│  │   Input     │    │   Generator │    │   Editor    │                    │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘                    │
│         │                  │                  │                            │
│         └──────────────────┼──────────────────┘                            │
│                            │                                                │
│                    ┌───────▼───────┐                                        │
│                    │   Mindmap     │                                        │
│                    │   Engine      │                                        │
│                    └───────┬───────┘                                        │
│                            │                                                │
│  ┌─────────────────────────┼─────────────────────────────────────────┐    │
│  │                         │                                         │    │
│  ▼                         ▼                                         ▼    │
│ ┌─────────────┐      ┌─────────────┐      ┌─────────────┐            │
│ │   Maps      │      │   Trees     │      │   Branches │            │
│ └──────┬──────┘      └──────┬──────┘      └──────┬──────┘            │
│        │                     │                     │                      │
│        ▼                     ▼                     ▼                      │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐            │
│  │   States   │      │   Nodes     │      │   States   │            │
│  └─────────────┘      └─────────────┘      └─────────────┘            │
│                            │                                                │
│         ┌──────────────────┼──────────────────┐                            │
│         │                  │                  │                            │
│  ┌──────▼──────┐    ┌──────▼──────┐    ┌──────▼──────┐               │
│  │  Export     │    │  AI Build   │    │  Collab     │               │
│  │  Engine     │    │  Connector  │    │  System     │               │
│  └─────────────┘    └─────────────┘    └─────────────┘               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.3 Data Flow

```
User Idea → Intent Decomposition → Map Generation → Tree Expansion 
         → Branch Creation → State Definition → Flow Validation
         → Impact Analysis → Export/Build/Collaborate
```

---

## 2. Mindmap Generation Engine

### 2.1 GenerationConfiguration Interface

```typescript
interface MindmapGenerationConfig {
  // Generation settings
  generation: GenerationConfig;
  
  // Map structure
  structure: MapStructureConfig;
  
  // AI settings
  ai: AIMindmapConfig;
  
  // Refactoring
  refactoring: RefactoringConfig;
  
  // Export
  export: ExportConfig;
  
  // Collaboration
  collaboration: CollaborationConfig;
  
  // Access control
  access: MindmapAccessConfig;
}

interface GenerationConfig {
  // Auto-generation behavior
  autoGenerate: boolean;
  
  // Maximum depth for automatic expansion
  maxDepth: number;
  
  // Expansion strategy
  expansionStrategy: 'comprehensive' | 'focused' | 'minimal';
  
  // Phases generation
  phases: {
    enabled: boolean;
    types: PhaseType[];
    autoExpand: boolean;
  };
  
  // Stages within phases
  stages: {
    enabled: boolean;
    types: StageType[];
    autoExpand: boolean;
  };
  
  // Branch generation
  branches: {
    enabled: boolean;
    maxPerLevel: number;
    autoGenerate: boolean;
  };
  
  // State generation
  states: {
    enabled: boolean;
    types: StateType[];
    autoGenerate: boolean;
  };
  
  // Exhaustion handling
  exhaustion: ExhaustionConfig;
}

interface MapStructureConfig {
  // Map types supported
  mapTypes: MapType[];
  
  // Tree types
  treeTypes: TreeType[];
  
  // Branch types
  branchTypes: BranchType[];
  
  // State types
  stateTypes: StateType[];
  
  // Relationship types
  relationshipTypes: RelationshipType[];
  
  // Validation rules
  validation: ValidationRules;
}

interface AIMindmapConfig {
  // Enable AI generation
  enabled: boolean;
  
  // Default AI provider
  provider: 'openai' | 'anthropic' | 'google' | 'custom';
  
  // Model selection
  model: string;
  
  // System prompts
  prompts: AIPromptConfig;
  
  // Generation parameters
  parameters: {
    temperature: number;
    maxTokens: number;
    topP: number;
  };
  
  // Governance
  governance: AIGovernanceConfig;
  
  // Building integration
  building: AIBuildingConfig;
}

interface AIPromptConfig {
  // Default system prompt
  systemPrompt: string;
  
  // Custom prompts per dimension
  dimensionPrompts: Record<string, string>;
  
  // Prompt templates
  templates: PromptTemplate[];
  
  // Admin-defined rules
  rules: AdminPromptRule[];
}

interface AIGovernanceConfig {
  // Enable governance
  enabled: boolean;
  
  // Rules that AI must follow
  rules: GovernanceRule[];
  
  // Forbidden topics
  forbiddenTopics: string[];
  
  // Scope limitation
  scopeLimit: {
    enabled: boolean;
    allowedDomains: string[];
  };
  
  // Response constraints
  constraints: {
    maxLength: number;
    formatRequirements: string[];
  };
  
  // Drift detection
  driftDetection: {
    enabled: boolean;
    threshold: number;
  };
}

interface AIBuildingConfig {
  // Enable AI building
  enabled: boolean;
  
  // Supported platforms
  platforms: BuildingPlatform[];
  
  // Connection settings
  connections: PlatformConnection[];
  
  // Build triggers
  triggers: BuildTrigger[];
  
  // Real-time build view
  realTimeView: boolean;
}
```

### 2.2 Default Generation Configuration

```typescript
const defaultMindmapGenerationConfig: MindmapGenerationConfig = {
  generation: {
    autoGenerate: true,
    maxDepth: 10,
    expansionStrategy: 'comprehensive',
    phases: {
      enabled: true,
      types: ['discovery', 'design', 'development', 'testing', 'deployment', 'maintenance'],
      autoExpand: true
    },
    stages: {
      enabled: true,
      types: ['requirements', 'architecture', 'implementation', 'review', 'validation'],
      autoExpand: true
    },
    branches: {
      enabled: true,
      maxPerLevel: 20,
      autoGenerate: true
    },
    states: {
      enabled: true,
      types: ['draft', 'active', 'completed', 'archived', 'cancelled'],
      autoGenerate: true
    },
    exhaustion: {
      mode: 'warn_then_continue',
      maxAutoDepth: 50,
      warningThreshold: 40,
      continueWithUserConfirmation: true
    }
  },
  structure: {
    mapTypes: ['project', 'feature', 'technical', 'business'],
    treeTypes: ['phase', 'module', 'feature', 'workflow'],
    branchTypes: ['main', 'alternative', 'exception', 'extension'],
    stateTypes: ['active', 'completed', 'pending', 'blocked', 'cancelled'],
    relationshipTypes: ['depends_on', 'implements', 'extends', 'references'],
    validation: {
      requirePhaseBeforeStage: true,
      requireStateInBranch: true,
      maxNestingDepth: 15,
      cyclicReferenceAllowed: false
    }
  },
  ai: {
    enabled: true,
    provider: 'anthropic',
    model: 'claude-3-sonnet-20240229',
    prompts: {
      systemPrompt: `You are BlueprintForge, an expert system architect. 
Generate comprehensive mindmap structures following these rules:
1. Always decompose into phases and stages
2. Each branch must have defined states
3. Include happy path, error paths, and edge cases
4. Maintain consistency with existing structure
5. Do not deviate from project scope`,
      dimensionPrompts: {},
      templates: [],
      rules: []
    },
    parameters: {
      temperature: 0.7,
      maxTokens: 4000,
      topP: 0.9
    },
    governance: {
      enabled: true,
      rules: [
        {
          id: 'scope-limit',
          name: 'Project Scope Limitation',
          description: 'AI must not answer questions outside the project scope',
          enabled: true,
          action: 'block'
        },
        {
          id: 'source-secrecy',
          name: 'Source Code Protection',
          description: 'AI must not reveal internal implementation secrets',
          enabled: true,
          action: 'redact'
        }
      ],
      forbiddenTopics: [],
      scopeLimit: {
        enabled: true,
        allowedDomains: []
      },
      constraints: {
        maxLength: 10000,
        formatRequirements: ['json', 'markdown']
      },
      driftDetection: {
        enabled: true,
        threshold: 0.7
      }
    },
    building: {
      enabled: true,
      platforms: ['lovable', 'base44', 'custom'],
      connections: [],
      triggers: ['manual', 'on_complete', 'scheduled'],
      realTimeView: true
    }
  },
  refactoring: {
    enabled: true,
    autoPropagate: true,
    impactAnalysis: true,
    validationBeforeApply: true,
    createBackup: true
  },
  export: {
    formats: ['json', 'markdown', 'pdf', 'docx', 'prd'],
    includeMetadata: true,
    includeDiagrams: true,
    compression: true
  },
  collaboration: {
    inviteEnabled: true,
    roles: ['viewer', 'editor', 'admin'],
    defaultRole: 'viewer',
    requireApproval: false
  },
  access: {
    viewRoles: ['OWNER', 'ADMIN', 'MEMBER', 'VIEWER'],
    editRoles: ['OWNER', 'ADMIN', 'MEMBER'],
    adminRoles: ['OWNER', 'ADMIN'],
    deleteRoles: ['OWNER']
  }
};
```

---

## 3. Map, Tree, Branch, and State Structure

### 3.1 Core Data Models

```typescript
interface MindMap {
  id: string;
  organizationId: string;
  projectId: string;
  
  // Identification
  name: string;
  description: string;
  type: MapType;
  
  // Root element (the original idea)
  rootIdea: string;
  
  // Structure
  phases: Phase[];
  trees: Tree[];
  
  // Metadata
  metadata: MapMetadata;
  
  // Generation
  generationStatus: 'pending' | 'generating' | 'completed' | 'failed';
  generationProgress: number;
  depth: number;
  
  // Versioning
  version: number;
  versions: MapVersion[];
  
  // Settings
  settings: MapSettings;
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
  generatedAt?: Date;
}

interface Phase {
  id: string;
  mindmapId: string;
  
  // Content
  name: string;
  description: string;
  type: PhaseType;
  order: number;
  
  // Children
  stages: Stage[];
  
  // Status
  status: 'draft' | 'active' | 'completed' | 'archived';
  progress: number;
  
  // Trees under this phase
  trees: Tree[];
  
  // Flow
  flow?: PhaseFlow;
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}

interface Stage {
  id: string;
  phaseId: string;
  
  // Content
  name: string;
  description: string;
  type: StageType;
  order: number;
  
  // Branches
  branches: Branch[];
  
  // Status
  status: 'draft' | 'active' | 'completed' | 'archived';
  progress: number;
  
  // Flow
  flow?: StageFlow;
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}

interface Tree {
  id: string;
  mindmapId: string;
  phaseId?: string;
  stageId?: string;
  parentTreeId?: string;
  
  // Content
  name: string;
  description: string;
  type: TreeType;
  
  // Structure
  rootNode: TreeNode;
  nodes: TreeNode[];
  edges: TreeEdge[];
  
  // Parent reference
  parentTree?: Tree;
  childTrees: Tree[];
  
  // Branches
  branches: Branch[];
  
  // Metadata
  metadata: TreeMetadata;
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}

interface TreeNode {
  id: string;
  treeId: string;
  parentNodeId?: string;
  
  // Content
  title: string;
  description: string;
  type: NodeType;
  
  // Position
  position: { x: number; y: number };
  
  // Branch reference
  branchId?: string;
  
  // States
  states: NodeState[];
  
  // Metadata
  metadata: NodeMetadata;
  
  // Children
  children: TreeNode[];
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}

interface Branch {
  id: string;
  treeId: string;
  stageId?: string;
  parentBranchId?: string;
  
  // Content
  name: string;
  description: string;
  type: BranchType;
  
  // Flow type
  flowType: 'happy_path' | 'error_path' | 'edge_case' | 'alternative' | 'extension';
  
  // States
  states: BranchState[];
  
  // Path
  path: string;
  depth: number;
  
  // Children
  childBranches: Branch[];
  
  // Nodes under this branch
  nodes: TreeNode[];
  
  // Status
  status: BranchStatus;
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}

interface BranchState {
  id: string;
  branchId: string;
  
  // Content
  name: string;
  description: string;
  type: StateType;
  
  // Flow
  flow: StateFlow;
  
  // Transitions
  transitions: StateTransition[];
  
  // Actions
  actions: StateAction[];
  
  // Conditions
  conditions: StateCondition[];
  
  // Metadata
  metadata: StateMetadata;
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}

interface StateFlow {
  // Entry
  entry: {
    action?: string;
    condition?: string;
    automatic: boolean;
  };
  
  // Processing
  processing: {
    steps: FlowStep[];
    parallel: boolean;
  };
  
  // Exit
  exit: {
    success: string;
    failure: string;
    always: string;
  };
  
  // Side effects
  sideEffects: SideEffect[];
}

interface FlowStep {
  id: string;
  name: string;
  description: string;
  type: 'action' | 'validation' | 'transformation' | 'integration';
  
  // Implementation
  implementation?: {
    type: 'manual' | 'auto' | 'ai';
    code?: string;
    prompt?: string;
  };
  
  // Error handling
  errorHandling: {
    onError: 'retry' | 'skip' | 'abort' | 'fallback';
    retries: number;
    fallback?: string;
  };
  
  // Dependencies
  dependsOn: string[];
  provides: string[];
}
```

### 3.2 Map Type Definitions

```typescript
enum MapType {
  PROJECT = 'project',
  FEATURE = 'feature',
  TECHNICAL = 'technical',
  BUSINESS = 'business',
  USER_JOURNEY = 'user_journey',
  API = 'api',
  DATABASE = 'database',
  INTEGRATION = 'integration'
}

enum PhaseType {
  DISCOVERY = 'discovery',
  DESIGN = 'design',
  DEVELOPMENT = 'development',
  TESTING = 'testing',
  DEPLOYMENT = 'deployment',
  MAINTENANCE = 'maintenance',
  MARKETING = 'marketing',
  OPERATIONS = 'operations'
}

enum StageType {
  REQUIREMENTS = 'requirements',
  ARCHITECTURE = 'architecture',
  IMPLEMENTATION = 'implementation',
  REVIEW = 'review',
  VALIDATION = 'validation',
  DOCUMENTATION = 'documentation'
}

enum TreeType {
  MODULE = 'module',
  FEATURE = 'feature',
  WORKFLOW = 'workflow',
  COMPONENT = 'component',
  SERVICE = 'service',
  ENTITY = 'entity'
}

enum BranchType {
  MAIN = 'main',
  ALTERNATIVE = 'alternative',
  ERROR = 'error',
  EDGE_CASE = 'edge_case',
  EXTENSION = 'extension',
  OPTIONAL = 'optional'
}

enum StateType {
  DRAFT = 'draft',
  ACTIVE = 'active',
  PENDING = 'pending',
  COMPLETED = 'completed',
  BLOCKED = 'blocked',
  CANCELLED = 'cancelled',
  ARCHIVED = 'archived',
  FAILED = 'failed'
}

enum NodeType {
  ENTITY = 'entity',
  ACTION = 'action',
  PROCESS = 'process',
  DECISION = 'decision',
  INTEGRATION = 'integration',
  STATE = 'state',
  EVENT = 'event',
  USER = 'user',
  DATA = 'data',
  API = 'api'
}
```

### 3.3 Complete Generation Flow

```typescript
interface GenerationFlow {
  // Step 1: Intent Decomposition
  step: 'decomposition';
  input: {
    idea: string;
    context?: Record<string, any>;
  };
  output: {
    projectType: string;
    entities: string[];
    actions: string[];
    flows: FlowDescription[];
  };
  
  // Step 2: Map Creation
  step: 'map_creation';
  input: {
    projectType: string;
    entities: string[];
  };
  output: {
    map: MindMap;
    rootPhase: Phase;
  };
  
  // Step 3: Phase Generation
  step: 'phase_generation';
  input: {
    mapId: string;
    projectType: string;
  };
  output: {
    phases: Phase[];
  };
  
  // Step 4: Stage Generation
  step: 'stage_generation';
  input: {
    phaseId: string;
  };
  output: {
    stages: Stage[];
  };
  
  // Step 5: Tree Generation
  step: 'tree_generation';
  input: {
    stageId: string;
    entity: string;
  };
  output: {
    trees: Tree[];
  };
  
  // Step 6: Branch Generation
  step: 'branch_generation';
  input: {
    treeId: string;
    flowType: FlowType;
  };
  output: {
    branches: Branch[];
  };
  
  // Step 7: State Definition
  step: 'state_definition';
  input: {
    branchId: string;
  };
  output: {
    states: BranchState[];
  };
  
  // Step 8: Flow Completion
  step: 'flow_completion';
  input: {
    branchId: string;
    states: BranchState[];
  };
  output: {
    transitions: StateTransition[];
    sideEffects: SideEffect[];
  };
  
  // Step 9: Validation
  step: 'validation';
  input: {
    mindmapId: string;
  };
  output: {
    valid: boolean;
    issues: ValidationIssue[];
  };
  
  // Step 10: Refactoring Check
  step: 'refactoring_check';
  input: {
    mindmapId: string;
    changes: Change[];
  };
  output: {
    impactedElements: ImpactedElement[];
    requiredChanges: RequiredChange[];
  };
}
```

---

## 4. Map Refactoring & Impact Analysis

### 4.1 Refactoring Engine

```typescript
interface RefactoringEngine {
  // Analyze impact of changes
  analyzeImpact(changes: Change[], mindmapId: string): ImpactAnalysis;
  
  // Apply changes with propagation
  applyChanges(changes: Change[], options: ApplyOptions): Promise<RefactorResult>;
  
  // Validate before applying
  validateChanges(changes: Change[]): ValidationResult;
  
  // Create backup before refactoring
  createBackup(mindmapId: string): Promise<Backup>;
  
  // Restore from backup
  restoreBackup(backupId: string): Promise<void>;
}

interface ImpactAnalysis {
  // Overall impact score
  impactScore: number;
  
  // Impacted elements
  impactedElements: ImpactedElement[];
  
  // Required changes
  requiredChanges: RequiredChange[];
  
  // Breaking changes
  breakingChanges: BreakingChange[];
  
  // Recommendations
  recommendations: Recommendation[];
}

interface ImpactedElement {
  elementType: 'map' | 'phase' | 'stage' | 'tree' | 'branch' | 'state' | 'node';
  elementId: string;
  elementName: string;
  changeType: 'modified' | 'deleted' | 'moved' | 'merged' | 'split';
  severity: 'critical' | 'high' | 'medium' | 'low';
  reason: string;
}

interface RequiredChange {
  targetElement: string;
  targetElementType: string;
  change: {
    type: 'update' | 'create' | 'delete';
    field: string;
    oldValue: any;
    newValue: any;
  };
  automatic: boolean;
  requiresConfirmation: boolean;
}

interface Change {
  type: 'update' | 'delete' | 'create' | 'move' | 'merge' | 'split';
  
  // Target
  targetType: 'map' | 'phase' | 'stage' | 'tree' | 'branch' | 'state' | 'node' | 'edge';
  targetId: string;
  
  // Change data
  data?: Record<string, any>;
  
  // Parent for new elements
  newParentId?: string;
  
  // Merge source
  mergeSourceId?: string;
  
  // Options
  options?: {
    propagate?: boolean;
    force?: boolean;
    createVersion?: boolean;
  };
}
```

### 4.2 Refactoring Scenarios

```typescript
const refactoringScenarios = [
  {
    scenario: 'Edit branch changes states',
    description: 'Modifying a branch affects its states',
    changes: [
      {
        type: 'update',
        targetType: 'branch',
        targetId: 'branch-123',
        data: { flowType: 'error_path' }
      }
    ],
    impact: {
      impactedStates: ['state-1', 'state-2'],
      requiredChanges: [
        { targetElement: 'state-1', change: { type: 'update', field: 'type', newValue: 'failed' } }
      ]
    }
  },
  {
    scenario: 'Delete phase cascades',
    description: 'Deleting a phase removes all its children',
    changes: [
      {
        type: 'delete',
        targetType: 'phase',
        targetId: 'phase-design'
      }
    ],
    impact: {
      impactedStages: ['stage-1', 'stage-2'],
      impactedTrees: ['tree-1', 'tree-2', 'tree-3'],
      impactedBranches: ['branch-1', 'branch-2', 'branch-3', 'branch-4'],
      requiredChanges: [
        { targetElement: 'phase-design', change: { type: 'delete', cascade: true } }
      ]
    }
  },
  {
    scenario: 'Merge two trees',
    description: 'Merging trees combines their branches',
    changes: [
      {
        type: 'merge',
        targetType: 'tree',
        targetId: 'tree-main',
        mergeSourceId: 'tree-alternative'
      }
    ],
    impact: {
      impactedBranches: ['branch-alt-1', 'branch-alt-2'],
      requiredChanges: [
        { targetElement: 'tree-alternative', change: { type: 'delete' } },
        { targetElement: 'tree-main', change: { type: 'update', field: 'branches', newValue: [...existing, ...fromMerged] } }
      ]
    }
  },
  {
    scenario: 'Split branch creates new paths',
    description: 'Splitting a branch creates alternative paths',
    changes: [
      {
        type: 'split',
        targetType: 'branch',
        targetId: 'branch-main',
        data: { splitPoint: 'state-3', newBranches: ['branch-main', 'branch-alt-1'] }
      }
    ],
    impact: {
      newBranches: ['branch-alt-1'],
      newStates: ['state-alt-1', 'state-alt-2'],
      requiredChanges: [
        { targetElement: 'state-3', change: { type: 'update', field: 'transitions', newValue: [...existing, { to: 'state-alt-1' }] } }
      ]
    }
  },
  {
    scenario: 'Move stage between phases',
    description: 'Moving a stage to different phase',
    changes: [
      {
        type: 'move',
        targetType: 'stage',
        targetId: 'stage-arch',
        newParentId: 'phase-dev'
      }
    ],
    impact: {
      impactedPhase: 'phase-dev',
      requiredChanges: [
        { targetElement: 'stage-arch', change: { type: 'update', field: 'phaseId', newValue: 'phase-dev' } },
        { targetElement: 'phase-arch', change: { type: 'update', field: 'stages', remove: 'stage-arch' } },
        { targetElement: 'phase-dev', change: { type: 'update', field: 'stages', add: 'stage-arch' } }
      ]
    }
  }
];
```

### 4.3 Validation Rules

```typescript
interface ValidationRules {
  // Structural validation
  structural: {
    requireMapHasPhases: boolean;
    requirePhaseHasStages: boolean;
    requireStageHasBranches: boolean;
    requireBranchHasStates: boolean;
    maxNestingDepth: number;
    allowCircularReferences: boolean;
  };
  
  // Content validation
  content: {
    requireNames: boolean;
    minNameLength: number;
    maxNameLength: number;
    requireDescriptions: boolean;
    allowEmptyNodes: boolean;
  };
  
  // Flow validation
  flow: {
    requireEntryState: boolean;
    requireExitState: boolean;
    allowDeadStates: boolean;
    requireTransitionConditions: boolean;
    validateStateTransitions: boolean;
  };
  
  // Reference validation
  references: {
    allowOrphanedNodes: boolean;
    requireParentReferences: boolean;
    validateForeignKeys: boolean;
  };
}
```

---

## 5. Export & Download System

### 5.1 Export Configuration

```typescript
interface ExportConfig {
  // Available formats
  formats: ExportFormat[];
  
  // Export scope options
  scope: ExportScope[];
  
  // Content options
  content: ExportContentConfig;
  
  // Compression
  compression: {
    enabled: boolean;
    algorithm: 'zip' | 'gzip' | 'tar';
  };
  
  // Include options
  includes: {
    metadata: boolean;
    diagrams: boolean;
    code: boolean;
    documents: boolean;
    history: boolean;
  };
}

enum ExportFormat {
  JSON = 'json',
  MARKDOWN = 'markdown',
  PDF = 'pdf',
  DOCX = 'docx',
  PRD = 'prd',
  Mermaid = 'mermaid',
  SQL = 'sql',
  YAML = 'yaml'
}

enum ExportScope {
  ENTIRE_PROJECT = 'entire_project',
  SINGLE_MAP = 'single_map',
  SINGLE_TREE = 'single_tree',
  SINGLE_BRANCH = 'single_branch',
  SINGLE_STATE = 'single_state',
  CUSTOM_SELECTION = 'custom_selection'
}

interface ExportRequest {
  // What to export
  scope: ExportScope;
  targetIds?: string[];
  
  // Format
  format: ExportFormat;
  
  // Options
  options: {
    includeMetadata: boolean;
    includeDiagrams: boolean;
    includeCode: boolean;
    includeDocuments: boolean;
    includeHistory: boolean;
    compress: boolean;
  };
  
  // Customization
  customization?: {
    title?: string;
    author?: string;
    version?: string;
    template?: string;
  };
}
```

### 5.2 Export Examples

```typescript
const exportExamples = [
  {
    name: 'Export Single Tree as PRD',
    request: {
      scope: 'single_tree',
      targetIds: ['tree-123'],
      format: 'prd',
      options: {
        includeMetadata: true,
        includeDiagrams: true,
        includeCode: false,
        includeDocuments: true,
        includeHistory: false,
        compress: false
      }
    }
  },
  {
    name: 'Export Entire Project as ZIP',
    request: {
      scope: 'entire_project',
      format: 'zip',
      options: {
        includeMetadata: true,
        includeDiagrams: true,
        includeCode: true,
        includeDocuments: true,
        includeHistory: true,
        compress: true
      }
    }
  },
  {
    name: 'Export Branch as Markdown',
    request: {
      scope: 'single_branch',
      targetIds: ['branch-456'],
      format: 'markdown',
      options: {
        includeMetadata: true,
        includeDiagrams: true,
        includeCode: false,
        includeDocuments: true,
        includeHistory: false,
        compress: false
      }
    }
  },
  {
    name: 'Export Multiple Trees as JSON',
    request: {
      scope: 'custom_selection',
      targetIds: ['tree-1', 'tree-2', 'tree-3'],
      format: 'json',
      options: {
        includeMetadata: true,
        includeDiagrams: false,
        includeCode: false,
        includeDocuments: false,
        includeHistory: true,
        compress: false
      }
    }
  }
];
```

### 5.3 View Modes

```typescript
interface ViewModeConfig {
  // Available view modes
  modes: ViewMode[];
  
  // Default mode
  defaultMode: ViewMode;
  
  // Persistence
  persistPreference: boolean;
}

enum ViewMode {
  CANVAS = 'canvas',
  TREE = 'tree',
  OUTLINE = 'outline',
  DROPDOWN = 'dropdown',
  ACCORDION = 'accordion',
  LIST = 'list',
  Gantt = 'gantt',
  Kanban = 'kanban'
}

interface DropdownViewConfig {
  // Dropdown behavior
  behavior: {
    expandOnClick: boolean;
    collapseOnOutsideClick: boolean;
    animate: boolean;
    animationDuration: number;
  };
  
  // Levels to show
  levels: {
    showPhases: boolean;
    showStages: boolean;
    showTrees: boolean;
    showBranches: boolean;
    maxExpandedLevel: number;
  };
  
  // Styling
  styling: {
    indentSize: number;
    showIcons: boolean;
    showCounts: boolean;
    highlightActive: boolean;
  };
  
  // Actions
  actions: {
    allowEdit: boolean;
    allowDelete: boolean;
    allowExport: boolean;
    allowAddChild: boolean;
  };
}

interface AccordionViewConfig {
  // Accordion behavior
  behavior: {
    allowMultipleOpen: boolean;
    expandOnHover: boolean;
    animate: boolean;
  };
  
  // Levels
  levels: {
    phases: boolean;
    stages: boolean;
    trees: boolean;
  };
  
  // Styling
  styling: {
    showIcons: boolean;
    showDescriptions: boolean;
    compact: boolean;
  };
}
```

---

## 6. AI Integration & Building

### 6.1 Building Platform Configuration

```typescript
interface BuildingPlatform {
  id: string;
  name: string;
  type: 'lovable' | 'base44' | 'custom' | 'github';
  
  // Connection
  connection: {
    type: 'api' | 'oauth' | 'webhook';
    config: PlatformConnectionConfig;
  };
  
  // Capabilities
  capabilities: {
    pushCode: boolean;
    pullCode: boolean;
    realTimeSync: boolean;
    buildStatus: boolean;
  };
  
  // Authentication
  auth: {
    type: 'api_key' | 'oauth' | 'basic';
    credentials: EncryptedCredentials;
  };
  
  // Active
  enabled: boolean;
}

interface PlatformConnectionConfig {
  // API configuration
  apiUrl: string;
  apiVersion: string;
  
  // Webhook configuration
  webhookUrl?: string;
  webhookSecret?: string;
  
  // OAuth configuration
  oauthConfig?: {
    clientId: string;
    clientSecret: string;
    redirectUri: string;
    scopes: string[];
  };
}

interface BuildRequest {
  // Source
  source: {
    type: 'map' | 'tree' | 'branch' | 'state' | 'selection';
    ids: string[];
  };
  
  // Target platform
  platform: string;
  
  // Build options
  options: {
    generateCode: boolean;
    generateTests: boolean;
    generateDocs: boolean;
    deploy: boolean;
  };
  
  // Custom prompt
  customPrompt?: string;
  
  // Follow prompt rules
  followRules: boolean;
}
```

### 6.2 AI Building Flow

```typescript
const aiBuildingFlow = {
  // Step 1: Prepare for Build
  prepare: {
    description: 'Prepare the selected elements for AI building',
    steps: [
      'Validate all elements have required data',
      'Resolve all references',
      'Generate build input from map structure',
      'Apply governance rules to input',
      'Validate against prompt rules'
    ]
  },
  
  // Step 2: Generate Code
  generate: {
    description: 'Send to AI building platform',
    platforms: {
      lovables: {
        description: 'Push to Lovable project',
        steps: [
          'Format code according to Lovable structure',
          'Create/update files via API',
          'Trigger build process',
          'Monitor build status'
        ]
      },
      base44: {
        description: 'Push to Base44 project',
        steps: [
          'Format code according to Base44 structure',
          'Push via Git integration',
          'Trigger deployment',
          'Monitor status'
        ]
      },
      custom: {
        description: 'Send to custom AI builder',
        steps: [
          'Apply custom prompt templates',
          'Send to AI with full context',
          'Receive generated code',
          'Process and validate output'
        ]
      }
    }
  },
  
  // Step 3: Build Status
  status: {
    description: 'Monitor and display build progress',
    events: [
      'build_started',
      'build_in_progress',
      'build_completed',
      'build_failed',
      'deploy_started',
      'deploy_completed',
      'deploy_failed'
    ]
  },
  
  // Step 4: Integration
  integration: {
    description: 'Connect and sync with building platforms',
    features: [
      'Real-time build preview',
      'Pull existing code into map',
      'Two-way sync',
      'Webhook notifications'
    ]
  }
};
```

### 6.3 Lovable Integration

```typescript
interface LovableIntegration {
  // Connect to Lovable
  connect(config: LovableConfig): Promise<ConnectionResult>;
  
  // Push to Lovable project
  push(request: BuildRequest): Promise<PushResult>;
  
  // Get build status
  getBuildStatus(projectId: string): Promise<BuildStatus>;
  
  // Get project URL
  getProjectUrl(projectId: string): string;
  
  // Pull from Lovable
  pull(projectId: string): Promise<PullResult>;
  
  // List projects
  listProjects(): Promise<LovableProject[]>;
}

interface LovableConfig {
  apiKey: string;
  organizationId?: string;
}
```

### 6.4 Emergent/Other AI Platforms

```typescript
interface GenericAIIntegration {
  // Platform type
  platform: string;
  
  // Connection
  connect(config: PlatformConfig): Promise<void>;
  
  // Send to build
  sendToBuild(input: BuildInput): Promise<BuildResponse>;
  
  // Get status
  getStatus(buildId: string): Promise<BuildStatus>;
  
  // Get output
  getOutput(buildId: string): Promise<BuildOutput>;
  
  // Webhook for updates
  setupWebhook(url: string, events: string[]): Promise<void>;
}
```

---

## 7. Collaboration & Access Control

### 7.1 Collaboration Configuration

```typescript
interface CollaborationConfig {
  // Invitation settings
  invitations: {
    enabled: boolean;
    requireApproval: boolean;
    defaultRole: CollaborationRole;
    maxInvitesPerDay: number;
    inviteExpiry: number; // hours
  };
  
  // Roles
  roles: CollaborationRole[];
  
  // Permissions matrix
  permissions: PermissionMatrix;
  
  // Real-time collaboration
  realtime: {
    enabled: boolean;
    provider: 'websocket' | 'sse';
    maxUsers: number;
  };
  
  // Activity tracking
  activity: {
    enabled: boolean;
    logEdits: boolean;
    logViews: boolean;
    logExports: boolean;
  };
}

enum CollaborationRole {
  VIEWER = 'viewer',
  EDITOR = 'editor',
  ADMIN = 'admin'
}

interface PermissionMatrix {
  // Map-level permissions
  map: {
    view: CollaborationRole[];
    edit: CollaborationRole[];
    delete: CollaborationRole[];
    export: CollaborationRole[];
    share: CollaborationRole[];
    build: CollaborationRole[];
  };
  
  // Tree-level permissions
  tree: {
    view: CollaborationRole[];
    edit: CollaborationRole[];
    delete: CollaborationRole[];
    branch: CollaborationRole[];
  };
  
  // Branch-level permissions
  branch: {
    view: CollaborationRole[];
    edit: CollaborationRole[];
    delete: CollaborationRole[];
    state: CollaborationRole[];
  };
  
  // State-level permissions
  state: {
    view: CollaborationRole[];
    edit: CollaborationRole[];
    delete: CollaborationRole[];
  };
}
```

### 7.2 Invitation System

```typescript
interface Invitation {
  id: string;
  mindmapId: string;
  
  // Inviter
  inviterId: string;
  inviterEmail: string;
  
  // Invitee
  inviteeEmail: string;
  inviteeName?: string;
  
  // Scope
  scope: {
    type: 'entire_map' | 'specific_tree' | 'specific_branch' | 'specific_state';
    targetIds?: string[];
  };
  
  // Role
  role: CollaborationRole;
  
  // Status
  status: 'pending' | 'accepted' | 'declined' | 'expired' | 'revoked';
  
  // Expiry
  expiresAt: Date;
  acceptedAt?: Date;
  
  // Permissions
  permissions: string[];
  
  // Timestamps
  createdAt: Date;
  updatedAt: Date;
}

interface CollaborationSession {
  id: string;
  userId: string;
  mindmapId: string;
  
  // Scope
  scope: {
    type: string;
    targetIds?: string[];
  };
  
  // Role
  role: CollaborationRole;
  
  // Activity
  canEdit: boolean;
  canView: boolean;
  canExport: boolean;
  canBuild: boolean;
  
  // Status
  active: boolean;
  lastActivityAt: Date;
  
  // Timestamps
  joinedAt: Date;
  leftAt?: Date;
}
```

---

## 8. AI Prompt Configuration

### 8.1 Prompt Management System

```typescript
interface PromptConfiguration {
  // System prompts
  systemPrompts: SystemPrompt[];
  
  // Dimension prompts
  dimensionPrompts: DimensionPrompt[];
  
  // Templates
  templates: PromptTemplate[];
  
  // Admin rules
  rules: AdminPromptRule[];
  
  // Governance
  governance: GovernanceConfiguration;
}

interface SystemPrompt {
  id: string;
  name: string;
  description: string;
  
  // Content
  content: string;
  
  // Variables
  variables: PromptVariable[];
  
  // Versioning
  version: number;
  isDefault: boolean;
  isActive: boolean;
  
  // Scope
  scope: {
    type: 'global' | 'organization' | 'project';
    organizationId?: string;
    projectId?: string;
  };
  
  // Metadata
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

interface AdminPromptRule {
  id: string;
  name: string;
  description: string;
  
  // Rule type
  type: 'scope_limit' | 'content_filter' | 'format_constraint' | 'output_guard' | 'source_protection';
  
  // Condition
  condition: {
    trigger: 'always' | 'on_topic' | 'on_keywords';
    keywords?: string[];
    topics?: string[];
  };
  
  // Action
  action: {
    type: 'block' | 'redact' | 'modify' | 'warn' | 'fallback';
    response?: string;
    replacement?: string;
  };
  
  // Priority
  priority: number;
  
  // Enabled
  enabled: boolean;
  
  // Scope
  scope: {
    appliesTo: 'generation' | 'building' | 'both';
    projects?: string[];
  };
}

interface GovernanceConfiguration {
  // Enable governance
  enabled: boolean;
  
  // Strict mode
  strictMode: boolean;
  
  // Rules
  rules: GovernanceRule[];
  
  // Monitoring
  monitoring: {
    enabled: boolean;
    logViolations: boolean;
    alertOnViolation: boolean;
    alertChannel?: string;
  };
  
  // Reporting
  reporting: {
    enabled: boolean;
    dailyReport: boolean;
    weeklyReport: boolean;
    reportRecipients: string[];
  };
}
```

### 8.2 Default Prompt Rules

```typescript
const defaultPromptRules: AdminPromptRule[] = [
  {
    id: 'scope-limit',
    name: 'Project Scope Limitation',
    description: 'AI must not answer questions outside the project scope',
    type: 'scope_limit',
    condition: {
      trigger: 'on_keywords',
      keywords: ['unrelated', 'outside', 'different project']
    },
    action: {
      type: 'block',
      response: 'This question is outside the scope of your project. Please ask project-related questions.'
    },
    priority: 100,
    enabled: true,
    scope: { appliesTo: 'both' }
  },
  {
    id: 'source-protection',
    name: 'Source Code Protection',
    description: 'AI must not reveal internal implementation secrets or proprietary code',
    type: 'source_protection',
    condition: {
      trigger: 'always'
    },
    action: {
      type: 'redact',
      replacement: '[Redacted for security]'
    },
    priority: 90,
    enabled: true,
    scope: { appliesTo: 'both' }
  },
  {
    id: 'format-constraint',
    name: 'Output Format Constraint',
    description: 'Ensure outputs follow defined format requirements',
    type: 'format_constraint',
    condition: {
      trigger: 'always'
    },
    action: {
      type: 'modify',
      response: 'Please format your response in JSON or Markdown format.'
    },
    priority: 50,
    enabled: true,
    scope: { appliesTo: 'generation' }
  },
  {
    id: 'content-filter',
    name: 'Inappropriate Content Filter',
    description: 'Block inappropriate or harmful content',
    type: 'content_filter',
    condition: {
      trigger: 'on_keywords',
      keywords: ['harmful', 'illegal', 'malicious']
    },
    action: {
      type: 'block',
      response: 'I cannot assist with this request.'
    },
    priority: 200,
    enabled: true,
    scope: { appliesTo: 'both' }
  },
  {
    id: 'output-guard',
    name: 'Output Safety Guard',
    description: 'Validate outputs for safety and appropriateness',
    type: 'output_guard',
    condition: {
      trigger: 'always'
    },
    action: {
      type: 'warn',
      response: 'This output has been flagged for review.'
    },
    priority: 150,
    enabled: true,
    scope: { appliesTo: 'building' }
  }
];
```

### 8.3 Prompt Templates

```typescript
interface PromptTemplate {
  id: string;
  name: string;
  description: string;
  
  // Content
  template: string;
  
  // Variables
  variables: TemplateVariable[];
  
  // Usage
  usage: {
    type: 'generation' | 'building' | 'refactoring' | 'documentation';
    dimension?: string;
  };
  
  // Category
  category: string;
  
  // Active
  active: boolean;
  
  // Metadata
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

// Example templates
const promptTemplates: PromptTemplate[] = [
  {
    id: 'entity-generation',
    name: 'Entity Generation',
    description: 'Generate data model entities from requirements',
    template: `Generate data model entities for {{projectName}}.
    
Context: {{projectDescription}}
Industry: {{industry}}
User Types: {{userTypes}}

For each entity, provide:
- Entity name
- Fields with types
- Relationships
- States

{{additionalInstructions}}`,
    variables: [
      { name: 'projectName', type: 'string', required: true },
      { name: 'projectDescription', type: 'string', required: true },
      { name: 'industry', type: 'string', required: false },
      { name: 'userTypes', type: 'array', required: false },
      { name: 'additionalInstructions', type: 'string', required: false }
    ],
    usage: { type: 'generation', dimension: 'data-model' },
    category: 'data-model',
    active: true,
    createdBy: 'system',
    createdAt: new Date(),
    updatedAt: new Date()
  },
  {
    id: 'flow-generation',
    name: 'User Flow Generation',
    description: 'Generate user flow diagrams from use cases',
    template: `Generate user flows for {{projectName}}.

Use Case: {{useCase}}
Actors: {{actors}}

Generate:
- Happy path
- Error paths
- Alternative flows
- Edge cases

{{additionalInstructions}}`,
    variables: [
      { name: 'projectName', type: 'string', required: true },
      { name: 'useCase', type: 'string', required: true },
      { name: 'actors', type: 'array', required: false },
      { name: 'additionalInstructions', type: 'string', required: false }
    ],
    usage: { type: 'generation', dimension: 'user-flow' },
    category: 'flows',
    active: true,
    createdBy: 'system',
    createdAt: new Date(),
    updatedAt: new Date()
  }
];
```

---

## 9. Auto-Generation & Exhaustion Handling

### 9.1 Exhaustion Configuration

```typescript
interface ExhaustionConfig {
  // Enable exhaustion handling
  enabled: boolean;
  
  // Mode
  mode: 'stop' | 'warn_then_continue' | 'auto_continue' | 'yolo';
  
  // Thresholds
  thresholds: {
    // Maximum automatic depth
    maxAutoDepth: number;
    
    // Warning threshold
    warningThreshold: number;
    
    // Critical threshold
    criticalThreshold: number;
    
    // Branch count limit
    maxBranchesPerLevel: number;
    
    // Node count limit
    maxNodes: number;
  };
  
  // Warning settings
  warning: {
    enabled: boolean;
    threshold: number;
    message: string;
    showProgress: boolean;
    showContinueButton: boolean;
  };
  
  // Continuation options
  continuation: {
    // Allow user to continue
    allowUserContinue: boolean;
    
    // Auto-continue after warning
    autoContinueAfterWarning: boolean;
    
    // Delay before auto-continue
    autoContinueDelay: number;
    
    // Require confirmation
    requireConfirmation: boolean;
  };
  
  // YOLO mode (exhaustion)
  yoloMode: {
    enabled: boolean;
    unlimitedDepth: boolean;
    skipValidation: boolean;
    warnAboutPerformance: boolean;
    maxTokensOverride?: number;
  };
}

enum ExhaustionMode {
  STOP = 'stop',
  WARN_THEN_CONTINUE = 'warn_then_continue',
  AUTO_CONTINUE = 'auto_continue',
  YOLO = 'yolo'
}
```

### 9.2 Exhaustion Handling Flow

```typescript
const exhaustionHandlingFlow = {
  // Scenario 1: Stop mode
  stop: {
    when: 'maxAutoDepth reached',
    behavior: 'Generation stops automatically',
    userAction: 'Cannot continue without manual restart',
    notification: 'Notification that generation stopped'
  },
  
  // Scenario 2: Warn then continue
  warn_then_continue: {
    when: 'warningThreshold reached',
    behavior: 'Shows warning dialog with progress',
    userAction: 'Click "Continue" to proceed',
    continuation: 'Generates more after confirmation',
    options: [
      'Continue generating',
      'Stop and review',
      'Export current state',
      'Adjust parameters'
    ]
  },
  
  // Scenario 3: Auto continue
  auto_continue: {
    when: 'warningThreshold reached',
    behavior: 'Automatically continues after delay',
    userAction: 'Can cancel during delay',
    delay: 'Configurable (default 5 seconds)',
    notification: 'Shows notification that generation will continue'
  },
  
  // Scenario 4: YOLO mode
  yolo: {
    when: 'User enables YOLO mode',
    behavior: 'No limits, full speed ahead',
    warnings: 'Shows performance warning',
    limitations: 'May cause performance issues',
    safeguards: 'Timeout protection still applies'
  }
};
```

### 9.3 Progress Tracking

```typescript
interface GenerationProgress {
  // Current status
  status: 'idle' | 'generating' | 'paused' | 'completed' | 'failed' | 'exhausted';
  
  // Progress metrics
  metrics: {
    currentDepth: number;
    maxDepth: number;
    phasesCreated: number;
    stagesCreated: number;
    treesCreated: number;
    branchesCreated: number;
    statesCreated: number;
    totalElements: number;
    percentComplete: number;
  };
  
  // Exhaustion status
  exhaustion: {
    approaching: boolean;
    threshold: number;
    current: number;
    mode: ExhaustionMode;
    canContinue: boolean;
    requiresConfirmation: boolean;
  };
  
  // Current operation
  currentOperation?: {
    type: string;
    description: string;
    startedAt: Date;
  };
  
  // Timeline
  timeline: {
    startedAt: Date;
    estimatedCompletion?: Date;
    pausedAt?: Date;
    resumedAt?: Date;
    completedAt?: Date;
  };
}
```

---

## 10. Project Management

### 10.1 Project Ownership & Access

```typescript
interface ProjectAccessConfig {
  // Owner capabilities
  owner: {
    canViewAllUserProjects: boolean;
    canLockAnyProject: boolean;
    canHardDeleteAnyProject: boolean;
    canTransferOwnership: boolean;
    canOverridePermissions: boolean;
  };
  
  // User capabilities
  user: {
    canCreateProjects: boolean;
    maxProjects: number;
    canDeleteOwnProjects: boolean;
    canLockOwnProjects: boolean;
    canExportOwnProjects: boolean;
  };
  
  // Soft delete
  softDelete: {
    enabled: boolean;
    gracePeriodDays: number;
    ownerCanRestore: boolean;
    ownerCanHardDelete: boolean;
  };
  
  // Locking
  locking: {
    enabled: boolean;
    ownerCanLock: boolean;
    adminCanLock: boolean;
    userCanLockOwn: boolean;
    lockedProjectsEditable: boolean;
  };
}

interface ProjectLock {
  id: string;
  projectId: string;
  
  // Lock info
  lockedBy: string;
  lockedByRole: 'owner' | 'admin' | 'user';
  lockedAt: Date;
  
  // Reason
  reason?: string;
  
  // Type
  type: 'full' | 'partial';
  restrictions?: {
    cannotEdit: boolean;
    cannotExport: boolean;
    cannotDelete: boolean;
    cannotShare: boolean;
    readOnly: boolean;
  };
  
  // Expiry
  expiresAt?: Date;
  
  // Active
  active: boolean;
}
```

### 10.2 Project Deletion Flow

```typescript
const projectDeletionFlow = {
  // Soft delete (user action)
  softDelete: {
    trigger: 'User deletes own project',
    steps: [
      'Mark project as deleted',
      'Set deletedAt timestamp',
      'Hide from user views',
      'Retain for grace period',
      'Notify user of deletion'
    ],
    gracePeriod: 30,
    restoreAvailable: true
  },
  
  // Hard delete (owner action)
  hardDelete: {
    trigger: 'Owner permanently deletes project',
    steps: [
      'Verify owner permission',
      'Confirm deletion dialog',
      'Delete all related data',
      'Delete from backups',
      'Remove from searches'
    ],
    permanent: true,
    restoreAvailable: false
  },
  
  // Admin delete
  adminDelete: {
    trigger: 'Admin deletes user project',
    steps: [
      'Verify admin permission',
      'Check if project belongs to other org',
      'Confirm with reason',
      'Soft delete with notice',
      'Notify project owner'
    ],
    noticeToOwner: true
  },
  
  // Automatic cleanup
  autoCleanup: {
    trigger: 'Grace period expired',
    steps: [
      'Check grace period',
      'Notify owner before deletion',
      'Delete after grace period',
      'Archive metadata only'
    ]
  }
};
```

---

## 11. GitHub Integration

### 11.1 GitHub Connection

```typescript
interface GitHubIntegration {
  // Connect to GitHub
  connect(config: GitHubConfig): Promise<ConnectionResult>;
  
  // Disconnect
  disconnect(): Promise<void>;
  
  // Test connection
  testConnection(): Promise<boolean>;
  
  // List repositories
  listRepositories(options?: ListOptions): Promise<GitHubRepository[]>;
  
  // Import from repository
  importFromRepository(repoId: string, options?: ImportOptions): Promise<ImportResult>;
  
  // Pull code into map
  pullCodeToMap(repoId: string, branch: string, path: string, mapId: string): Promise<PullResult>;
  
  // Push map to repository
  pushMapToRepository(mapId: string, repoId: string, branch: string, path: string): Promise<PushResult>;
  
  // Get commit history
  getCommitHistory(repoId: string, path: string): Promise<Commit[]>;
  
  // Webhooks
  setupWebhook(events: string[], url: string): Promise<void>;
}

interface GitHubConfig {
  // Authentication
  auth: {
    type: 'oauth' | 'personal_access_token';
    token: string;
  };
  
  // Organization
  organizationId?: string;
  
  // Default settings
  defaults: {
    defaultRepo?: string;
    defaultBranch: string;
    autoSync: boolean;
  };
}

interface ImportOptions {
  // Import scope
  scope: 'entire_repo' | 'specific_path' | 'single_file';
  path?: string;
  
  // File types to include
  fileTypes: string[];
  
  // Import as
  importAs: 'tree' | 'branch' | 'nodes';
  
  // Conflict resolution
  conflictResolution: 'skip' | 'overwrite' | 'rename';
}
```

### 11.2 GitHub Import Flow

```typescript
const githubImportFlow = {
  // Step 1: Connect
  connect: {
    description: 'Connect to GitHub account',
    options: ['OAuth flow', 'Personal access token']
  },
  
  // Step 2: Select Repository
  selectRepository: {
    description: 'Choose repository to import from',
    filters: ['Search by name', 'Filter by language', 'Sort by date']
  },
  
  // Step 3: Select Content
  selectContent: {
    description: 'Choose what to import',
    options: [
      'Entire repository',
      'Specific folder/path',
      'Single file',
      'Multiple files'
    ]
  },
  
  // Step 4: Map to Structure
  mapToStructure: {
    description: 'Map imported content to mindmap structure',
    autoMapping: {
      enabled: true,
      rules: [
        'folders → trees',
        'files → branches',
        'classes/functions → nodes',
        'directories → stages'
      ]
    },
    manualMapping: {
      enabled: true,
      preview: true
    }
  },
  
  // Step 5: Import
  import: {
    description: 'Import selected content',
    process: [
      'Fetch files from GitHub',
      'Parse code structure',
      'Map to map/tree/branch',
      'Generate states from code',
      'Validate structure',
      'Save to database'
    ]
  }
};
```

---

## 12. API Routes

### 12.1 Complete API Architecture

```
/api/mindmaps
├── GET /                           # List mindmaps
├── POST /                          # Create mindmap
├── POST /generate                  # Generate from idea
├── GET /[id]                      # Get mindmap details
├── PUT /[id]                      # Update mindmap
├── DELETE /[id]                   # Delete mindmap (soft)
├── POST /[id]/restore             # Restore mindmap
├── GET /[id]/export               # Export mindmap
├── POST /[id]/refactor            # Refactor mindmap
├── GET /[id]/impact               # Get impact analysis
├── GET /[id]/progress             # Get generation progress

/api/mindmaps/[id]/phases
├── GET /                          # List phases
├── POST /                         # Create phase
├── PUT /[phaseId]                # Update phase
├── DELETE /[phaseId]             # Delete phase
├── POST /[phaseId]/reorder       # Reorder phases

/api/mindmaps/[id]/stages
├── GET /                          # List stages
├── POST /                         # Create stage
├── PUT /[stageId]                # Update stage
├── DELETE /[stageId]             # Delete stage

/api/mindmaps/[id]/trees
├── GET /                          # List trees
├── POST /                         # Create tree
├── PUT /[treeId]                 # Update tree
├── DELETE /[treeId]              # Delete tree
├── POST /[treeId]/branches       # Add branch to tree

/api/mindmaps/[id]/branches
├── GET /                          # List branches
├── POST /                         # Create branch
├── PUT /[branchId]              # Update branch
├── DELETE /[branchId]            # Delete branch
├── POST /[branchId]/states       # Add state to branch

/api/mindmaps/[id]/states
├── GET /                          # List states
├── POST /                         # Create state
├── PUT /[stateId]               # Update state
├── DELETE /[stateId]            # Delete state

/api/mindmaps/[id]/collaboration
├── GET /invites                  # List invitations
├── POST /invites                 # Send invitation
├── DELETE /invites/[id]         # Revoke invitation
├── POST /invites/[id]/accept    # Accept invitation
├── POST /invites/[id]/decline   # Decline invitation
├── GET /members                  # List members
├── PUT /members/[id]            # Update member role

/api/mindmaps/[id]/export
├── GET /formats                  # Available formats
├── POST /export                  # Export mindmap
├── GET /[exportId]/status       # Get export status
├── GET /[exportId]/download     # Download export

/api/mindmaps/[id]/build
├── GET /platforms                # Available platforms
├── POST /build                   # Send to build
├── GET /build/[buildId]         # Get build status
├── GET /build/[buildId]/output  # Get build output

/api/admin/mindmaps
├── GET /                         # List all mindmaps (across orgs)
├── GET /[id]                    # Get any mindmap
├── PUT /[id]                    # Update any mindmap
├── DELETE /[id]                 # Hard delete
├── POST /[id]/lock              # Lock mindmap
├── POST /[id]/unlock            # Unlock mindmap
├── GET /prompts                 # Get prompt config
├── PUT /prompts                 # Update prompt config
├── GET /rules                   # Get prompt rules
├── PUT /rules                   # Update prompt rules

/api/admin/governance
├── GET /                         # Get governance config
├── PUT /                         # Update governance config
├── GET /violations              # Get rule violations
├── GET /reports                 # Get governance reports
```

---

## 13. Database Schema

### 13.1 Prisma Schema Models

```prisma
// MindMap model
model MindMap {
  id              String   @id @default(cuid())
  organizationId String
  organization   Organization @relation(fields: [organizationId], references: [id])
  
  // Identification
  name            String
  description     String?  @db.Text
  type            MapType
  
  // Root idea
  rootIdea        String   @db.Text
  
  // Generation
  generationStatus GenerationStatus @default(PENDING)
  generationProgress Int      @default(0)
  depth            Int      @default(0)
  
  // Status
  status          MapStatus @default(DRAFT)
  
  // Lock
  locked          Boolean  @default(false)
  lockedBy        String?
  lockedAt        DateTime?
  lockReason      String?
  
  // Soft delete
  deletedAt       DateTime?
  deletedBy       String?
  
  // Metadata
  metadata        Json?
  
  // Settings
  settings        Json?
  
  // Versioning
  version         Int      @default(1)
  versions        MindMapVersion[]
  
  // Relations
  phases          Phase[]
  trees           Tree[]
  invitations    MindMapInvitation[]
  collaborationSessions CollaborationSession[]
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  generatedAt     DateTime?
  
  // Indexes
  @@index([organizationId, status])
  @@index([organizationId, deletedAt])
  @@index([rootIdea])
  
  @@map("mindmaps")
}

// Phase model
model Phase {
  id              String   @id @default(cuid())
  mindmapId       String
  mindmap         MindMap  @relation(fields: [mindmapId], references: [id], onDelete: Cascade)
  
  // Content
  name            String
  description     String?  @db.Text
  type            PhaseType
  order           Int
  
  // Status
  status          ElementStatus @default(DRAFT)
  progress        Int      @default(0)
  
  // Relations
  stages          Stage[]
  trees           Tree[]
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([mindmapId, order])
  @@map("phases")
}

// Stage model
model Stage {
  id              String   @id @default(cuid())
  phaseId         String
  phase           Phase    @relation(fields: [phaseId], references: [id], onDelete: Cascade)
  
  // Content
  name            String
  description     String?  @db.Text
  type            StageType
  order           Int
  
  // Status
  status          ElementStatus @default(DRAFT)
  progress        Int      @default(0)
  
  // Relations
  branches        Branch[]
  trees           Tree[]
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([phaseId, order])
  @@map("stages")
}

// Tree model
model Tree {
  id              String   @id @default(cuid())
  mindmapId       String
  mindmap         MindMap  @relation(fields: [mindmapId], references: [id], onDelete: Cascade)
  
  // Hierarchy
  phaseId         String?
  phase           Phase?   @relation(fields: [phaseId], references: [id], onDelete: SetNull)
  stageId         String?
  stage           Stage?   @relation(fields: [stageId], references: [id], onDelete: SetNull)
  parentTreeId    String?
  parentTree      Tree?    @relation("TreeHierarchy", fields: [parentTreeId], references: [id], onDelete: SetNull)
  childTrees      Tree[]   @relation("TreeHierarchy")
  
  // Content
  name            String
  description     String?  @db.Text
  type            TreeType
  
  // Metadata
  metadata        Json?
  
  // Relations
  rootNodeId      String?
  nodes           TreeNode[]
  branches        Branch[]
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([mindmapId])
  @@index([parentTreeId])
  @@index([phaseId])
  @@index([stageId])
  
  @@map("trees")
}

// TreeNode model
model TreeNode {
  id              String   @id @default(cuid())
  treeId          String
  tree            Tree     @relation(fields: [treeId], references: [id], onDelete: Cascade)
  
  // Hierarchy
  parentNodeId    String?
  parentNode      TreeNode? @relation("NodeHierarchy", fields: [parentNodeId], references: [id], onDelete: SetNull)
  children        TreeNode[] @relation("NodeHierarchy")
  
  // Content
  title           String
  description     String?  @db.Text
  type            NodeType
  
  // Position
  positionX       Float
  positionY       Float
  
  // Branch reference
  branchId        String?
  
  // Metadata
  metadata        Json?
  
  // States
  states          NodeState[]
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([treeId])
  @@index([parentNodeId])
  @@index([branchId])
  
  @@map("tree_nodes")
}

// Branch model
model Branch {
  id              String   @id @default(cuid())
  treeId          String
  tree            Tree     @relation(fields: [treeId], references: [id], onDelete: Cascade)
  
  // Hierarchy
  stageId         String?
  stage           Stage?   @relation(fields: [stageId], references: [id], onDelete: SetNull)
  parentBranchId  String?
  parentBranch    Branch?  @relation("BranchHierarchy", fields: [parentBranchId], references: [id], onDelete: SetNull)
  childBranches   Branch[] @relation("BranchHierarchy")
  
  // Content
  name            String
  description     String?  @db.Text
  type            BranchType
  
  // Flow type
  flowType        FlowType
  
  // Path
  path            String
  depth           Int
  
  // Status
  status          BranchStatus @default(DRAFT)
  
  // Metadata
  metadata        Json?
  
  // Relations
  nodes           TreeNode[]
  states          BranchState[]
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([treeId])
  @@index([stageId])
  @@index([parentBranchId])
  @@index([path])
  
  @@map("branches")
}

// BranchState model
model BranchState {
  id              String   @id @default(cuid())
  branchId        String
  branch          Branch   @relation(fields: [branchId], references: [id], onDelete: Cascade)
  
  // Content
  name            String
  description     String?  @db.Text
  type            StateType
  
  // Flow
  flow            Json?
  
  // Transitions
  transitions     Json?
  
  // Metadata
  metadata        Json?
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([branchId])
  @@index([type])
  
  @@map("branch_states")
}

// MindMapInvitation model
model MindMapInvitation {
  id              String   @id @default(cuid())
  mindmapId       String
  mindmap         MindMap  @relation(fields: [mindmapId], references: [id], onDelete: Cascade)
  
  // Inviter
  inviterId       String
  inviter         User     @relation("InvitationInviter", fields: [inviterId], references: [id])
  
  // Invitee
  inviteeEmail    String
  
  // Scope
  scope           Json?
  
  // Role
  role            CollaborationRole @default(VIEWER)
  
  // Status
  status          InvitationStatus @default(PENDING)
  
  // Expiry
  expiresAt       DateTime
  
  // Accepted
  acceptedAt      DateTime?
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@index([mindmapId])
  @@index([inviteeEmail])
  @@index([status])
  
  @@map("mindmap_invitations")
}

// CollaborationSession model
model CollaborationSession {
  id              String   @id @default(cuid())
  mindmapId       String
  mindmap         MindMap  @relation(fields: [mindmapId], references: [id], onDelete: Cascade)
  
  userId          String
  user            User     @relation(fields: [userId], references: [id])
  
  // Scope
  scope           Json?
  
  // Role
  role            CollaborationRole
  
  // Permissions
  canEdit         Boolean  @default(false)
  canView         Boolean  @default(true)
  canExport       Boolean  @default(false)
  canBuild        Boolean  @default(false)
  
  // Activity
  active          Boolean  @default(true)
  lastActivityAt  DateTime @default(now())
  
  // Timestamps
  joinedAt        DateTime @default(now())
  leftAt          DateTime?
  
  @@index([mindmapId])
  @@index([userId])
  @@index([active])
  
  @@map("collaboration_sessions")
}

// MindMapVersion model
model MindMapVersion {
  id              String   @id @default(cuid())
  mindmapId       String
  mindmap         MindMap  @relation(fields: [mindmapId], references: [id], onDelete: Cascade)
  
  // Version info
  version         Int
  name            String?
  description     String?
  
  // Snapshot
  snapshot        Json
  
  // Creator
  createdBy       String
  
  // Timestamps
  createdAt       DateTime @default(now())
  
  @@unique([mindmapId, version])
  @@index([mindmapId])
  
  @@map("mindmap_versions")
}

// PromptConfiguration model
model PromptConfiguration {
  id              String   @id @default(cuid())
  organizationId String?
  
  // System prompts
  systemPrompts   Json?
  
  // Dimension prompts
  dimensionPrompts Json?
  
  // Templates
  templates       Json?
  
  // Rules
  rules           Json?
  
  // Governance
  governance      Json?
  
  // Active
  isActive        Boolean  @default(true)
  
  // Timestamps
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  @@unique([organizationId])
  
  @@map("prompt_configurations")
}

// ExportJob model
model ExportJob {
  id              String   @id @default(cuid())
  mindmapId       String
  
  // Request
  scope           String
  targetIds       String[]
  format          ExportFormat
  options         Json
  
  // Status
  status          ExportJobStatus @default(PENDING)
  progress        Int      @default(0)
  
  // Output
  outputUrl       String?
  outputSize      Int?
  
  // Error
  error           String?
  
  // Creator
  createdBy       String
  
  // Timestamps
  createdAt       DateTime @default(now())
  completedAt     DateTime?
  
  @@index([mindmapId])
  @@index([status])
  
  @@map("export_jobs")
}

// BuildJob model
model BuildJob {
  id              String   @id @default(cuid())
  mindmapId       String
  
  // Source
  source          Json
  
  // Platform
  platform        String
  
  // Options
  options         Json
  
  // Status
  status          BuildJobStatus @default(PENDING)
  progress        Int      @default(0)
  
  // Output
  outputUrl       String?
  buildUrl        String?
  
  // Error
  error           String?
  
  // Creator
  createdBy       String
  
  // Timestamps
  createdAt       DateTime @default(now())
  completedAt     DateTime?
  
  @@index([mindmapId])
  @@index([status])
  
  @@map("build_jobs")
}

// Enum definitions
enum MapType {
  PROJECT
  FEATURE
  TECHNICAL
  BUSINESS
  USER_JOURNEY
  API
  DATABASE
  INTEGRATION
}

enum PhaseType {
  DISCOVERY
  DESIGN
  DEVELOPMENT
  TESTING
  DEPLOYMENT
  MAINTENANCE
  MARKETING
  OPERATIONS
}

enum StageType {
  REQUIREMENTS
  ARCHITECTURE
  IMPLEMENTATION
  REVIEW
  VALIDATION
  DOCUMENTATION
}

enum TreeType {
  MODULE
  FEATURE
  WORKFLOW
  COMPONENT
  SERVICE
  ENTITY
}

enum BranchType {
  MAIN
  ALTERNATIVE
  ERROR
  EDGE_CASE
  EXTENSION
  OPTIONAL
}

enum StateType {
  DRAFT
  ACTIVE
  PENDING
  COMPLETED
  BLOCKED
  CANCELLED
  ARCHIVED
  FAILED
}

enum NodeType {
  ENTITY
  ACTION
  PROCESS
  DECISION
  INTEGRATION
  STATE
  EVENT
  USER
  DATA
  API
}

enum FlowType {
  HAPPY_PATH
  ERROR_PATH
  EDGE_CASE
  ALTERNATIVE
  EXTENSION
}

enum MapStatus {
  DRAFT
  GENERATING
  ACTIVE
  ARCHIVED
  LOCKED
}

enum GenerationStatus {
  PENDING
  GENERATING
  COMPLETED
  FAILED
  PAUSED
}

enum ElementStatus {
  DRAFT
  ACTIVE
  COMPLETED
  ARCHIVED
}

enum BranchStatus {
  DRAFT
  ACTIVE
  COMPLETED
  ARCHIVED
}

enum CollaborationRole {
  VIEWER
  EDITOR
  ADMIN
}

enum InvitationStatus {
  PENDING
  ACCEPTED
  DECLINED
  EXPIRED
  REVOKED
}

enum ExportFormat {
  JSON
  MARKDOWN
  PDF
  DOCX
  PRD
  MERMAID
  SQL
  YAML
}

enum ExportJobStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED
}

enum BuildJobStatus {
  PENDING
  BUILDING
  COMPLETED
  FAILED
}
```

---

## 14. Edge Cases

### 14.1 Generation Edge Cases

```typescript
const generationEdgeCases = [
  {
    scenario: 'Circular reference in branches',
    description: 'Branches reference each other creating infinite loop',
    handling: 'Detect cycles during validation, block save, suggest resolution'
  },
  {
    scenario: 'Massive tree depth',
    description: 'User attempts to generate very deep tree',
    handling: 'Apply exhaustion config, warn at threshold, stop at max depth'
  },
  {
    scenario: 'Conflicting branch names',
    description: 'Multiple branches with same name in same level',
    handling: 'Auto-append suffix, warn user, allow rename'
  },
  {
    scenario: 'Empty idea input',
    description: 'User submits blank or very short idea',
    handling: 'Require minimum length, suggest improvements, use templates'
  },
  {
    scenario: 'AI provider fails mid-generation',
    description: 'API call fails during generation',
    handling: 'Save partial progress, allow retry, show error clearly'
  },
  {
    scenario: 'State without transitions',
    description: 'Branch state has no defined transitions',
    handling: 'Warn during validation, allow save but flag, suggest fixes'
  },
  {
    scenario: 'Orphaned nodes',
    description: 'Nodes without parent or connection',
    handling: 'Auto-link to parent, warn user, allow manual fix'
  }
];
```

### 14.2 Collaboration Edge Cases

```typescript
const collaborationEdgeCases = [
  {
    scenario: 'Owner leaves organization',
    description: 'Only owner deletes account',
    handling: 'Transfer ownership to admin, notify users, maintain project access'
  },
  {
    scenario: 'Invited user already has account',
    description: 'Email matches existing user',
    handling: 'Link to existing account, notify both parties, merge permissions'
  },
  {
    scenario: 'Multiple simultaneous edits',
    description: 'Two users edit same element',
    handling: 'Last write wins with notification, offer merge option, log conflicts'
  },
  {
    scenario: 'Invitation link shared publicly',
    description: 'Invitation link leaked',
    handling: 'Time-limited links, single-use option, IP restriction option'
  },
  {
    scenario: 'User removed while editing',
    description: 'User loses access during active session',
    handling: 'Graceful disconnect, save work, notify user, log event'
  }
];
```

### 14.3 Export Edge Cases

```typescript
const exportEdgeCases = [
  {
    scenario: 'Export very large map',
    description: 'Map has thousands of elements',
    handling: 'Stream generation, show progress, allow background export, chunk output'
  },
  {
    scenario: 'Format conversion fails',
    description: 'Some elements not compatible with target format',
    handling: 'Convert to compatible alternative, flag unsupported, warn user'
  },
  {
    scenario: 'Export timeout',
    description: 'Export takes too long',
    handling: 'Start background job, notify when complete, allow cancel'
  },
  {
    scenario: 'Download link expires',
    description: 'User tries to download after link expired',
    handling: 'Re-generate on demand, store in user library, no extra cost'
  }
];
```

---

## 15. Performance & Caching

### 15.1 Performance Configuration

```typescript
interface MindmapPerformanceConfig {
  // Loading
  loading: {
    lazyLoadPhases: boolean;
    lazyLoadStages: boolean;
    lazyLoadTrees: boolean;
    prefetchNeighbors: boolean;
    cacheLoadedElements: boolean;
    cacheTtl: number;
  };
  
  // Rendering
  rendering: {
    virtualizeLongLists: boolean;
    renderThreshold: number;
    debounceUpdates: boolean;
    batchSize: number;
  };
  
  // Generation
  generation: {
    parallelProcessing: boolean;
    maxConcurrent: number;
    queueSize: number;
    priorityQueue: boolean;
  };
  
  // Export
  export: {
    streamLargeExports: boolean;
    chunkSize: number;
    compressionEnabled: boolean;
    backgroundProcessing: boolean;
  };
  
  // Real-time
  realtime: {
    enabled: boolean;
    maxConnections: number;
    heartbeatInterval: number;
    reconnectAttempts: number;
  };
}
```

### 15.2 Caching Strategy

```typescript
const mindmapCachingStrategy = {
  // Map metadata
  'mindmap:{id}:meta': {
    ttl: 300,
    invalidation: 'on-update'
  },
  
  // Phase list
  'mindmap:{id}:phases': {
    ttl: 60,
    invalidation: 'on-phase-change'
  },
  
  // Stage list per phase
  'phase:{id}:stages': {
    ttl: 60,
    invalidation: 'on-stage-change'
  },
  
  // Tree structure
  'tree:{id}': {
    ttl: 120,
    invalidation: 'on-tree-change'
  },
  
  // Branch with states
  'branch:{id}': {
    ttl: 60,
    invalidation: 'on-branch-change'
  },
  
  // User collaboration
  'user:{id}:collaboration': {
    ttl: 30,
    invalidation: 'on-invite-change'
  },
  
  // Export job
  'export:{id}': {
    ttl: 0,
    invalidation: 'never'
  }
};
```

---

## Summary

This comprehensive Mindmap System configuration flow covers every aspect of the BlueprintForge mindmap functionality:

### Key Components

1. **Mindmap Generation Engine** - AI-powered decomposition of ideas into phases, stages, trees, branches, and states
2. **Map/Tree/Branch/State Structure** - Complete hierarchical data model with flow definitions
3. **Refactoring Engine** - Impact analysis and automatic propagation of changes
4. **Export System** - Multiple formats (JSON, Markdown, PDF, DOCX, PRD) with scope options
5. **View Modes** - Canvas, tree, outline, dropdown, accordion, list, Gantt, Kanban
6. **AI Building Integration** - Lovable, Base44, and custom AI platform connections
7. **Collaboration** - Invitation system with roles and permissions
8. **Prompt Governance** - Admin-configurable rules that constrain AI behavior
9. **Exhaustion Handling** - Stop, warn-then-continue, auto-continue, YOLO modes
10. **Project Management** - Ownership, locking, soft/hard delete
11. **GitHub Integration** - Import existing code as maps, push maps to repositories

### Admin Capabilities

- Configure generation parameters and limits
- Set up AI prompts and templates
- Define governance rules for AI behavior
- Manage all user projects (view, lock, delete)
- Configure export formats and defaults
- Set collaboration permissions
- Monitor AI compliance and violations

### User Capabilities

- Create mindmaps from ideas
- Edit maps, trees, branches, states
- Refactor with automatic impact analysis
- Export individual elements or entire projects
- Switch between view modes
- Invite collaborators with specific roles
- Send to AI building platforms
- Connect GitHub repositories
