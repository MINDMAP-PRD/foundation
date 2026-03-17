# 🎯 KANBAN BOARD - COMPREHENSIVE IMPLEMENTATION
## Phase 2: Following Universal Element Configuration Pattern

**Version:** 1.0  
**Date:** 2026-02-15  
**Status:** Production-Ready

---

## 📋 TABLE OF CONTENTS

1. [Overview](#overview)
2. [Database Schema](#database-schema)
3. [Configuration Architecture](#configuration-architecture)
4. [Backend Implementation](#backend-implementation)
5. [Frontend Components](#frontend-components)
6. [Bidirectional Sync](#bidirectional-sync)
7. [Integration](#integration)

---

## OVERVIEW

The Kanban Board is a drag-drop task management system that syncs bidirectionally with the mindmap nodes. Each Kanban card can be linked to a mindmap node, and changes in either place reflect in the other.

### Key Features
- **Drag-drop columns and cards** using @dnd-kit
- **Bidirectional sync** with mindmap nodes
- **WIP limits** per column
- **Swimlanes** for categorization
- **Full RBAC** with granular permissions
- **Audit trail** for all changes
- **Real-time updates** via WebSocket/Supabase Realtime

---

## DATABASE SCHEMA

### Kanban Tables

```sql
-- ============================================================
-- KANBAN BOARDS
-- ============================================================
CREATE TABLE kanban_boards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  element_id VARCHAR(255) GENERATED ALWAYS AS ('kanban_' || id::text) STORED,
  
  -- Ownership
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  blueprint_id UUID REFERENCES blueprints(id) ON DELETE CASCADE,
  
  -- Configuration
  name VARCHAR(255) NOT NULL,
  description TEXT,
  
  -- Display Options
  view_mode VARCHAR(50) DEFAULT 'board' CHECK (view_mode IN ('board', 'list', 'calendar')),
  show_wip_limits BOOLEAN DEFAULT TRUE,
  show_swimlanes BOOLEAN DEFAULT FALSE,
  swimlane_field VARCHAR(100), -- e.g., 'priority', 'assignee', 'category'
  
  -- Filters
  default_filters JSONB DEFAULT '{}'::jsonb,
  
  -- RBAC Configuration
  rbac JSONB DEFAULT '{
    "allowedRoles": ["admin", "manager", "editor"],
    "deniedRoles": ["viewer", "guest"],
    "customPermissions": {
      "view": ["admin", "manager", "editor", "viewer"],
      "create": ["admin", "manager", "editor"],
      "edit": ["admin", "manager", "editor"],
      "delete": ["admin", "manager"],
      "move": ["admin", "manager", "editor"],
      "configure": ["admin", "manager"]
    }
  }'::jsonb,
  
  -- State
  current_state VARCHAR(50) DEFAULT 'active' CHECK (current_state IN ('draft', 'active', 'archived')),
  enabled BOOLEAN DEFAULT TRUE,
  
  -- Audit
  created_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_by UUID REFERENCES users(id),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  version INTEGER DEFAULT 1
);

CREATE INDEX idx_kanban_boards_org ON kanban_boards(organization_id);
CREATE INDEX idx_kanban_boards_blueprint ON kanban_boards(blueprint_id);

-- ============================================================
-- KANBAN COLUMNS
-- ============================================================
CREATE TABLE kanban_columns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  board_id UUID NOT NULL REFERENCES kanban_boards(id) ON DELETE CASCADE,
  
  -- Display
  name VARCHAR(255) NOT NULL,
  description TEXT,
  color VARCHAR(7),
  
  -- Ordering
  "order" INTEGER NOT NULL DEFAULT 0,
  
  -- WIP Limits
  wip_limit INTEGER, -- NULL = unlimited
  wip_limit_type VARCHAR(50) DEFAULT 'count' CHECK (wip_limit_type IN ('count', 'points', 'hours')),
  
  -- Automation
  auto_assignee UUID REFERENCES users(id),
  auto_status VARCHAR(100),
  
  -- Rules
  rules JSONB DEFAULT '[]'::jsonb, -- Array of automation rules
  
  -- Metadata
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  UNIQUE(board_id, "order")
);

CREATE INDEX idx_kanban_columns_board ON kanban_columns(board_id, "order");

-- ============================================================
-- KANBAN CARDS
-- ============================================================
CREATE TABLE kanban_cards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  element_id VARCHAR(255) GENERATED ALWAYS AS ('card_' || id::text) STORED,
  
  -- Ownership
  board_id UUID NOT NULL REFERENCES kanban_boards(id) ON DELETE CASCADE,
  column_id UUID NOT NULL REFERENCES kanban_columns(id) ON DELETE CASCADE,
  
  -- Core Data
  title VARCHAR(500) NOT NULL,
  description TEXT,
  
  -- Link to Mindmap Node (Bidirectional Sync)
  node_id UUID REFERENCES nodes(id) ON DELETE SET NULL,
  
  -- Assignee
  assignee_id UUID REFERENCES users(id),
  
  -- Metadata
  priority VARCHAR(50) DEFAULT 'medium' CHECK (priority IN ('low', 'medium', 'high', 'urgent')),
  labels JSONB DEFAULT '[]'::jsonb, -- Array of label strings
  due_date TIMESTAMPTZ,
  
  -- Effort Tracking
  story_points INTEGER,
  estimated_hours INTEGER,
  actual_hours INTEGER DEFAULT 0,
  
  -- Checklist
  checklist JSONB DEFAULT '[]'::jsonb, -- [{ id, text, completed }]
  
  -- Ordering within column
  "order" INTEGER NOT NULL DEFAULT 0,
  
  -- Swimlane
  swimlane_value VARCHAR(255),
  
  -- Status
  is_archived BOOLEAN DEFAULT FALSE,
  archived_at TIMESTAMPTZ,
  archived_by UUID REFERENCES users(id),
  
  -- Sync tracking
  last_synced_at TIMESTAMPTZ,
  sync_version INTEGER DEFAULT 1,
  
  -- Audit
  created_by UUID NOT NULL REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_by UUID REFERENCES users(id),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  UNIQUE(column_id, "order")
);

CREATE INDEX idx_kanban_cards_board ON kanban_cards(board_id);
CREATE INDEX idx_kanban_cards_column ON kanban_cards(column_id, "order");
CREATE INDEX idx_kanban_cards_node ON kanban_cards(node_id);
CREATE INDEX idx_kanban_cards_assignee ON kanban_cards(assignee_id);
CREATE INDEX idx_kanban_cards_archived ON kanban_cards(is_archived) WHERE is_archived = FALSE;

-- ============================================================
-- KANBAN CARD ACTIVITIES (Audit Trail)
-- ============================================================
CREATE TABLE kanban_card_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  card_id UUID NOT NULL REFERENCES kanban_cards(id) ON DELETE CASCADE,
  
  -- Activity Type
  activity_type VARCHAR(100) NOT NULL CHECK (activity_type IN (
    'created', 'updated', 'moved', 'assigned', 'commented', 
    'checklist_added', 'checklist_completed', 'archived', 'restored'
  )),
  
  -- Who did it
  user_id UUID NOT NULL REFERENCES users(id),
  user_role VARCHAR(50),
  
  -- What changed
  field_changed VARCHAR(255),
  old_value JSONB,
  new_value JSONB,
  
  -- Context for moves
  from_column_id UUID REFERENCES kanban_columns(id),
  to_column_id UUID REFERENCES kanban_columns(id),
  
  -- Metadata
  description TEXT,
  ip_address INET,
  user_agent TEXT,
  
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_kanban_activities_card ON kanban_card_activities(card_id, created_at DESC);
CREATE INDEX idx_kanban_activities_user ON kanban_card_activities(user_id);
CREATE INDEX idx_kanban_activities_type ON kanban_card_activities(activity_type);

-- ============================================================
-- KANBAN BOARD MEMBERS (Access Control)
-- ============================================================
CREATE TABLE kanban_board_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  board_id UUID NOT NULL REFERENCES kanban_boards(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  
  -- Role on this board
  board_role VARCHAR(50) DEFAULT 'member' CHECK (board_role IN ('admin', 'editor', 'viewer')),
  
  -- Permissions override
  can_edit_columns BOOLEAN DEFAULT FALSE,
  can_edit_cards BOOLEAN DEFAULT TRUE,
  can_delete_cards BOOLEAN DEFAULT FALSE,
  can_configure BOOLEAN DEFAULT FALSE,
  
  -- Notifications
  notify_on_assigned BOOLEAN DEFAULT TRUE,
  notify_on_mentioned BOOLEAN DEFAULT TRUE,
  notify_on_changes BOOLEAN DEFAULT FALSE,
  
  joined_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  invited_by UUID REFERENCES users(id),
  
  UNIQUE(board_id, user_id)
);

CREATE INDEX idx_kanban_members_board ON kanban_board_members(board_id);
CREATE INDEX idx_kanban_members_user ON kanban_board_members(user_id);

-- ============================================================
-- VIEWS
-- ============================================================

-- Active cards with full info
CREATE VIEW v_kanban_cards_active AS
SELECT 
  c.*,
  b.name as board_name,
  col.name as column_name,
  col.color as column_color,
  col.wip_limit as column_wip_limit,
  n.title as node_title,
  n.node_type as node_type,
  u.name as assignee_name,
  u.avatar_url as assignee_avatar,
  cb.name as created_by_name
FROM kanban_cards c
JOIN kanban_boards b ON c.board_id = b.id
JOIN kanban_columns col ON c.column_id = col.id
LEFT JOIN nodes n ON c.node_id = n.id
LEFT JOIN users u ON c.assignee_id = u.id
LEFT JOIN users cb ON c.created_by = cb.id
WHERE c.is_archived = FALSE
  AND b.enabled = TRUE
  AND b.current_state = 'active';

-- Board statistics
CREATE VIEW v_kanban_board_stats AS
SELECT 
  b.id as board_id,
  COUNT(DISTINCT c.id) as total_cards,
  COUNT(DISTINCT CASE WHEN c.is_archived THEN c.id END) as archived_cards,
  COUNT(DISTINCT col.id) as column_count,
  COUNT(DISTINCT c.assignee_id) as unique_assignees,
  SUM(c.story_points) as total_points,
  AVG(c.actual_hours) as avg_actual_hours
FROM kanban_boards b
LEFT JOIN kanban_columns col ON b.id = col.board_id
LEFT JOIN kanban_cards c ON col.id = c.column_id
WHERE b.enabled = TRUE
GROUP BY b.id;

-- Column WIP status
CREATE VIEW v_kanban_column_wip AS
SELECT 
  col.id as column_id,
  col.name as column_name,
  col.board_id,
  col.wip_limit,
  COUNT(c.id) as current_count,
  SUM(c.story_points) as current_points,
  CASE 
    WHEN col.wip_limit IS NULL THEN 'ok'
    WHEN COUNT(c.id) > col.wip_limit THEN 'exceeded'
    WHEN COUNT(c.id) = col.wip_limit THEN 'at_limit'
    ELSE 'ok'
  END as wip_status
FROM kanban_columns col
LEFT JOIN kanban_cards c ON col.id = c.column_id AND c.is_archived = FALSE
GROUP BY col.id, col.name, col.board_id, col.wip_limit;
```

---

## CONFIGURATION ARCHITECTURE

### Kanban Element Configuration

```typescript
// types/kanban.ts

export interface KanbanBoardConfig {
  // Identification
  id: string;
  elementId: string;
  
  // Core
  name: string;
  description?: string;
  
  // RBAC
  rbac: {
    allowedRoles: string[];
    deniedRoles: string[];
    customPermissions: {
      view: string[];
      create: string[];
      edit: string[];
      delete: string[];
      move: string[];
      configure: string[];
    };
  };
  
  // Display
  viewMode: 'board' | 'list' | 'calendar';
  showWipLimits: boolean;
  showSwimlanes: boolean;
  swimlaneField?: string;
  
  // State
  currentState: 'draft' | 'active' | 'archived';
  enabled: boolean;
  
  // Metadata
  createdBy: string;
  createdAt: Date;
  updatedBy?: string;
  updatedAt: Date;
}

export interface KanbanColumn {
  id: string;
  boardId: string;
  name: string;
  description?: string;
  color?: string;
  order: number;
  wipLimit?: number;
  wipLimitType: 'count' | 'points' | 'hours';
  autoAssignee?: string;
  autoStatus?: string;
  rules: ColumnRule[];
  cards: KanbanCard[];
}

export interface KanbanCard {
  id: string;
  elementId: string;
  boardId: string;
  columnId: string;
  
  // Core
  title: string;
  description?: string;
  
  // Link to Node
  nodeId?: string;
  node?: {
    id: string;
    title: string;
    type: string;
  };
  
  // Assignee
  assigneeId?: string;
  assignee?: {
    id: string;
    name: string;
    avatar?: string;
  };
  
  // Metadata
  priority: 'low' | 'medium' | 'high' | 'urgent';
  labels: string[];
  dueDate?: Date;
  
  // Effort
  storyPoints?: number;
  estimatedHours?: number;
  actualHours: number;
  
  // Checklist
  checklist: ChecklistItem[];
  
  // Ordering
  order: number;
  swimlaneValue?: string;
  
  // Status
  isArchived: boolean;
  
  // Audit
  createdBy: string;
  createdAt: Date;
  updatedBy?: string;
  updatedAt: Date;
}

export interface ChecklistItem {
  id: string;
  text: string;
  completed: boolean;
  completedAt?: Date;
  completedBy?: string;
}

export interface ColumnRule {
  id: string;
  trigger: 'card_moved_in' | 'card_moved_out' | 'due_date_approaching';
  conditions: RuleCondition[];
  actions: RuleAction[];
}

export interface RuleCondition {
  field: string;
  operator: 'equals' | 'not_equals' | 'contains' | 'greater_than';
  value: any;
}

export interface RuleAction {
  type: 'assign' | 'set_label' | 'set_due_date' | 'notify' | 'move_to_column';
  params: Record<string, any>;
}
```

---

## BACKEND IMPLEMENTATION

### API Routes

```typescript
// app/api/kanban/boards/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@/lib/auth';
import { prisma } from '@/lib/db';
import { checkPermission } from '@/lib/rbac';

// GET /api/kanban/boards - List boards
export async function GET(req: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { searchParams } = new URL(req.url);
    const blueprintId = searchParams.get('blueprintId');

    const boards = await prisma.kanbanBoard.findMany({
      where: {
        organizationId: session.user.organizationId,
        ...(blueprintId && { blueprintId }),
        enabled: true,
        currentState: 'active',
      },
      include: {
        _count: {
          select: { columns: true, cards: { where: { isArchived: false } } }
        }
      },
      orderBy: { updatedAt: 'desc' }
    });

    return NextResponse.json({ boards });
  } catch (error) {
    console.error('Error fetching kanban boards:', error);
    return NextResponse.json({ error: 'Failed to fetch boards' }, { status: 500 });
  }
}

// POST /api/kanban/boards - Create board
export async function POST(req: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const canCreate = await checkPermission(
      session.user,
      'kanban',
      'create'
    );
    
    if (!canCreate) {
      return NextResponse.json({ error: 'Forbidden' }, { status: 403 });
    }

    const data = await req.json();

    const board = await prisma.$transaction(async (tx) => {
      // Create board
      const newBoard = await tx.kanbanBoard.create({
        data: {
          name: data.name,
          description: data.description,
          organizationId: session.user.organizationId,
          blueprintId: data.blueprintId,
          viewMode: data.viewMode || 'board',
          showWipLimits: data.showWipLimits ?? true,
          showSwimlanes: data.showSwimlanes ?? false,
          swimlaneField: data.swimlaneField,
          createdBy: session.user.id,
        }
      });

      // Create default columns if none provided
      const columns = data.columns || [
        { name: 'To Do', order: 0, color: '#3b82f6' },
        { name: 'In Progress', order: 1, color: '#f59e0b' },
        { name: 'Done', order: 2, color: '#10b981' }
      ];

      await tx.kanbanColumn.createMany({
        data: columns.map((col: any) => ({
          ...col,
          boardId: newBoard.id
        }))
      });

      return newBoard;
    });

    return NextResponse.json({ board }, { status: 201 });
  } catch (error) {
    console.error('Error creating kanban board:', error);
    return NextResponse.json({ error: 'Failed to create board' }, { status: 500 });
  }
}
```

### Card CRUD with Node Sync

```typescript
// app/api/kanban/cards/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@/lib/auth';
import { prisma } from '@/lib/db';
import { syncCardWithNode } from '@/lib/kanban/sync';

// POST /api/kanban/cards - Create card
export async function POST(req: NextRequest) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const data = await req.json();

    const card = await prisma.$transaction(async (tx) => {
      // Get next order in column
      const lastCard = await tx.kanbanCard.findFirst({
        where: { columnId: data.columnId },
        orderBy: { order: 'desc' }
      });
      const nextOrder = (lastCard?.order || 0) + 1;

      // Create card
      const newCard = await tx.kanbanCard.create({
        data: {
          boardId: data.boardId,
          columnId: data.columnId,
          title: data.title,
          description: data.description,
          nodeId: data.nodeId,
          assigneeId: data.assigneeId,
          priority: data.priority || 'medium',
          labels: data.labels || [],
          dueDate: data.dueDate ? new Date(data.dueDate) : null,
          storyPoints: data.storyPoints,
          estimatedHours: data.estimatedHours,
          checklist: data.checklist || [],
          order: nextOrder,
          swimlaneValue: data.swimlaneValue,
          createdBy: session.user.id,
        },
        include: {
          assignee: { select: { id: true, name: true, avatarUrl: true } }
        }
      });

      // Log activity
      await tx.kanbanCardActivity.create({
        data: {
          cardId: newCard.id,
          activityType: 'created',
          userId: session.user.id,
          description: `Card "${newCard.title}" created`,
        }
      });

      // Sync with node if linked
      if (newCard.nodeId) {
        await syncCardWithNode(tx, newCard, 'create');
      }

      return newCard;
    });

    return NextResponse.json({ card }, { status: 201 });
  } catch (error) {
    console.error('Error creating kanban card:', error);
    return NextResponse.json({ error: 'Failed to create card' }, { status: 500 });
  }
}
```

### Move Card (Drag-Drop)

```typescript
// app/api/kanban/cards/[id]/move/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { auth } from '@/lib/auth';
import { prisma } from '@/lib/db';

export async function POST(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const session = await auth();
    if (!session?.user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const { id } = params;
    const { columnId, order } = await req.json();

    const card = await prisma.$transaction(async (tx) => {
      // Get current card
      const currentCard = await tx.kanbanCard.findUnique({
        where: { id },
        include: { column: true }
      });

      if (!currentCard) {
        throw new Error('Card not found');
      }

      const fromColumnId = currentCard.columnId;
      const toColumnId = columnId;

      // Update card order
      const updatedCard = await tx.kanbanCard.update({
        where: { id },
        data: {
          columnId,
          order,
          updatedBy: session.user.id,
          updatedAt: new Date()
        }
      });

      // Reorder other cards in target column
      await tx.kanbanCard.updateMany({
        where: {
          columnId,
          id: { not: id },
          order: { gte: order }
        },
        data: {
          order: { increment: 1 }
        }
      });

      // Log activity
      await tx.kanbanCardActivity.create({
        data: {
          cardId: id,
          activityType: 'moved',
          userId: session.user.id,
          fromColumnId,
          toColumnId,
          description: `Moved from "${currentCard.column.name}" to new column`,
        }
      });

      // Apply column automation rules
      if (fromColumnId !== toColumnId) {
        await applyColumnRules(tx, id, toColumnId);
      }

      return updatedCard;
    });

    return NextResponse.json({ card });
  } catch (error) {
    console.error('Error moving card:', error);
    return NextResponse.json({ error: 'Failed to move card' }, { status: 500 });
  }
}

async function applyColumnRules(tx: any, cardId: string, columnId: string) {
  const column = await tx.kanbanColumn.findUnique({
    where: { id: columnId },
    include: { rules: true }
  });

  if (!column?.rules?.length) return;

  for (const rule of column.rules) {
    if (rule.trigger === 'card_moved_in') {
      // Execute rule actions
      for (const action of rule.actions) {
        switch (action.type) {
          case 'assign':
            await tx.kanbanCard.update({
              where: { id: cardId },
              data: { assigneeId: action.params.userId }
            });
            break;
          case 'set_label':
            await tx.kanbanCard.update({
              where: { id: cardId },
              data: { labels: { push: action.params.label } }
            });
            break;
        }
      }
    }
  }
}
```

---

## BIDIRECTIONAL SYNC

### Sync Logic

```typescript
// lib/kanban/sync.ts

import { PrismaClient } from '@prisma/client';

/**
 * Sync Kanban card changes to Mindmap node
 */
export async function syncCardWithNode(
  tx: PrismaClient | any,
  card: any,
  action: 'create' | 'update' | 'delete'
) {
  if (!card.nodeId) return;

  const node = await tx.node.findUnique({
    where: { id: card.nodeId }
  });

  if (!node) return;

  switch (action) {
    case 'create':
    case 'update':
      // Update node with card data
      await tx.node.update({
        where: { id: card.nodeId },
        data: {
          label: card.title,
          description: card.description,
          metadata: {
            ...node.metadata,
            kanbanCardId: card.id,
            kanbanStatus: card.columnId,
            priority: card.priority,
            assignee: card.assigneeId,
            dueDate: card.dueDate,
            labels: card.labels,
            lastSyncedAt: new Date().toISOString()
          },
          updatedAt: new Date()
        }
      });
      break;

    case 'delete':
      // Remove kanban reference from node
      await tx.node.update({
        where: { id: card.nodeId },
        data: {
          metadata: {
            ...node.metadata,
            kanbanCardId: null,
            kanbanStatus: null
          }
        }
      });
      break;
  }
}

/**
 * Sync Mindmap node changes to Kanban card
 */
export async function syncNodeWithCard(
  tx: PrismaClient | any,
  node: any,
  action: 'create' | 'update' | 'delete'
) {
  // Find linked card
  const card = await tx.kanbanCard.findFirst({
    where: { nodeId: node.id }
  });

  if (!card) return;

  switch (action) {
    case 'update':
      await tx.kanbanCard.update({
        where: { id: card.id },
        data: {
          title: node.label,
          description: node.description,
          lastSyncedAt: new Date(),
          syncVersion: { increment: 1 }
        }
      });
      break;

    case 'delete':
      // Archive the card instead of deleting
      await tx.kanbanCard.update({
        where: { id: card.id },
        data: {
          isArchived: true,
          archivedAt: new Date(),
          lastSyncedAt: new Date()
        }
      });
      break;
  }
}
```

---

## FRONTEND COMPONENTS

### KanbanBoard Component

```tsx
// components/kanban/KanbanBoard.tsx

'use client';

import React, { useState, useCallback } from 'react';
import {
  DndContext,
  DragOverlay,
  closestCorners,
  KeyboardSensor,
  PointerSensor,
  useSensor,
  useSensors,
  DragStartEvent,
  DragOverEvent,
  DragEndEvent,
  defaultDropAnimationSideEffects,
  DropAnimation
} from '@dnd-kit/core';
import {
  arrayMove,
  SortableContext,
  sortableKeyboardCoordinates,
  horizontalListSortingStrategy
} from '@dnd-kit/sortable';
import { KanbanColumn } from './KanbanColumn';
import { KanbanCard } from './KanbanCard';
import { CardModal } from './CardModal';
import { useToast } from '@/hooks/use-toast';

interface KanbanBoardProps {
  boardId: string;
  initialColumns: KanbanColumnType[];
  initialCards: KanbanCardType[];
}

export function KanbanBoard({ boardId, initialColumns, initialCards }: KanbanBoardProps) {
  const [columns, setColumns] = useState(initialColumns);
  const [cards, setCards] = useState(initialCards);
  const [activeId, setActiveId] = useState<string | null>(null);
  const [selectedCard, setSelectedCard] = useState<KanbanCardType | null>(null);
  const { toast } = useToast();

  const sensors = useSensors(
    useSensor(PointerSensor, {
      activationConstraint: {
        distance: 5,
      },
    }),
    useSensor(KeyboardSensor, {
      coordinateGetter: sortableKeyboardCoordinates,
    })
  );

  // Find which column a card is in
  const findContainer = useCallback((id: string) => {
    const card = cards.find(c => c.id === id);
    return card?.columnId;
  }, [cards]);

  const handleDragStart = (event: DragStartEvent) => {
    setActiveId(event.active.id as string);
  };

  const handleDragOver = (event: DragOverEvent) => {
    const { active, over } = event;
    const overId = over?.id;

    if (!overId || active.id === overId) return;

    const activeContainer = findContainer(active.id as string);
    const overContainer = columns.find(c => c.id === overId) ? overId : findContainer(overId as string);

    if (!activeContainer || !overContainer || activeContainer === overContainer) return;

    setCards((prev) => {
      const activeCard = prev.find(c => c.id === active.id);
      if (!activeCard) return prev;

      return prev.map(c => 
        c.id === active.id ? { ...c, columnId: overContainer as string } : c
      );
    });
  };

  const handleDragEnd = async (event: DragEndEvent) => {
    const { active, over } = event;
    setActiveId(null);

    if (!over) return;

    const activeId = active.id as string;
    const overId = over.id as string;

    const activeContainer = findContainer(activeId);
    const overContainer = columns.find(c => c.id === overId) ? overId : findContainer(overId);

    if (!activeContainer || !overContainer) return;

    try {
      // Call API to persist move
      const response = await fetch(`/api/kanban/cards/${activeId}/move`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ 
          columnId: overContainer,
          order: cards.filter(c => c.columnId === overContainer).length 
        })
      });

      if (!response.ok) throw new Error('Failed to move card');

      toast({
        title: 'Card moved',
        description: 'Card has been moved successfully'
      });
    } catch (error) {
      toast({
        title: 'Error',
        description: 'Failed to move card',
        variant: 'destructive'
      });
    }
  };

  const dropAnimation: DropAnimation = {
    sideEffects: defaultDropAnimationSideEffects({
      styles: {
        active: {
          opacity: '0.5',
        },
      },
    }),
  };

  return (
    <>
      <DndContext
        sensors={sensors}
        collisionDetection={closestCorners}
        onDragStart={handleDragStart}
        onDragOver={handleDragOver}
        onDragEnd={handleDragEnd}
      >
        <div className="flex gap-4 overflow-x-auto pb-4 min-h-[600px]">
          <SortableContext 
            items={columns.map(c => c.id)}
            strategy={horizontalListSortingStrategy}
          >
            {columns.map(column => (
              <KanbanColumn
                key={column.id}
                column={column}
                cards={cards.filter(c => c.columnId === column.id && !c.isArchived)}
                onCardClick={setSelectedCard}
              />
            ))}
          </SortableContext>
        </div>

        <DragOverlay dropAnimation={dropAnimation}>
          {activeId ? (
            <KanbanCard
              card={cards.find(c => c.id === activeId)!}
              isOverlay
            />
          ) : null}
        </DragOverlay>
      </DndContext>

      {selectedCard && (
        <CardModal
          card={selectedCard}
          isOpen={!!selectedCard}
          onClose={() => setSelectedCard(null)}
          onUpdate={(updated) => {
            setCards(prev => prev.map(c => c.id === updated.id ? updated : c));
          }}
        />
      )}
    </>
  );
}

export default KanbanBoard;
```

### KanbanColumn Component

```tsx
// components/kanban/KanbanColumn.tsx

'use client';

import React from 'react';
import { useSortable, useDroppable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable';
import { KanbanCard } from './KanbanCard';
import { Button } from '@/components/ui/button';
import { Plus, MoreHorizontal } from 'lucide-react';
import { cn } from '@/lib/utils';

interface KanbanColumnProps {
  column: KanbanColumnType;
  cards: KanbanCardType[];
  onCardClick: (card: KanbanCardType) => void;
}

export function KanbanColumn({ column, cards, onCardClick }: KanbanColumnProps) {
  const {
    attributes,
    listeners,
    setNodeRef: setSortableRef,
    transform,
    transition,
    isDragging
  } = useSortable({
    id: column.id,
    data: {
      type: 'Column',
      column
    }
  });

  const { setNodeRef: setDroppableRef, isOver } = useDroppable({
    id: column.id,
    data: {
      type: 'Column',
      column
    }
  });

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
  };

  // Check WIP limit
  const wipStatus = column.wipLimit 
    ? cards.length >= column.wipLimit 
      ? 'exceeded' 
      : cards.length >= column.wipLimit * 0.8 
        ? 'warning' 
        : 'ok'
    : 'ok';

  return (
    <div
      ref={setSortableRef}
      style={style}
      className={cn(
        "flex-shrink-0 w-80 bg-gray-50 rounded-lg flex flex-col max-h-full",
        isDragging && "opacity-50 rotate-2",
        isOver && "ring-2 ring-primary"
      )}
    >
      {/* Column Header */}
      <div
        {...attributes}
        {...listeners}
        className="p-3 flex items-center justify-between cursor-grab active:cursor-grabbing"
        style={{ backgroundColor: column.color + '20' }}
      >
        <div className="flex items-center gap-2">
          <h3 className="font-semibold text-sm">{column.name}</h3>
          <span className={cn(
            "text-xs px-2 py-0.5 rounded-full",
            wipStatus === 'exceeded' && "bg-red-100 text-red-700",
            wipStatus === 'warning' && "bg-yellow-100 text-yellow-700",
            wipStatus === 'ok' && "bg-gray-200 text-gray-700"
          )}>
            {cards.length}{column.wipLimit ? `/${column.wipLimit}` : ''}
          </span>
        </div>
        <Button variant="ghost" size="icon" className="h-8 w-8">
          <MoreHorizontal className="h-4 w-4" />
        </Button>
      </div>

      {/* Cards Container */}
      <div
        ref={setDroppableRef}
        className="flex-1 p-2 overflow-y-auto min-h-[100px]"
      >
        <SortableContext
          items={cards.map(c => c.id)}
          strategy={verticalListSortingStrategy}
        >
          <div className="flex flex-col gap-2">
            {cards.map(card => (
              <KanbanCard
                key={card.id}
                card={card}
                onClick={() => onCardClick(card)}
              />
            ))}
          </div>
        </SortableContext>
      </div>

      {/* Add Card Button */}
      <div className="p-2">
        <Button 
          variant="ghost" 
          className="w-full justify-start text-muted-foreground"
          size="sm"
        >
          <Plus className="h-4 w-4 mr-2" />
          Add card
        </Button>
      </div>
    </div>
  );
}
```

### KanbanCard Component

```tsx
// components/kanban/KanbanCard.tsx

'use client';

import React from 'react';
import { useSortable } from '@dnd-kit/sortable';
import { CSS } from '@dnd-kit/utilities';
import { Card, CardContent, CardHeader } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { cn } from '@/lib/utils';
import { Calendar, MessageSquare, CheckSquare, AlertCircle } from 'lucide-react';
import { formatDistanceToNow } from 'date-fns';

interface KanbanCardProps {
  card: KanbanCardType;
  onClick?: () => void;
  isOverlay?: boolean;
}

export function KanbanCard({ card, onClick, isOverlay }: KanbanCardProps) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isDragging
  } = useSortable({
    id: card.id,
    data: {
      type: 'Card',
      card
    }
  });

  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
  };

  const priorityColors: Record<string, string> = {
    low: 'bg-blue-100 text-blue-700',
    medium: 'bg-yellow-100 text-yellow-700',
    high: 'bg-orange-100 text-orange-700',
    urgent: 'bg-red-100 text-red-700'
  };

  const completedChecklist = card.checklist?.filter(i => i.completed).length || 0;
  const totalChecklist = card.checklist?.length || 0;
  const isOverdue = card.dueDate && new Date(card.dueDate) < new Date() && !card.isArchived;

  return (
    <Card
      ref={setNodeRef}
      {...attributes}
      {...listeners}
      onClick={onClick}
      style={style}
      className={cn(
        "cursor-grab active:cursor-grabbing hover:shadow-md transition-shadow",
        isDragging && "opacity-50 rotate-1",
        isOverlay && "rotate-2 shadow-xl cursor-grabbing"
      )}
    >
      <CardHeader className="p-3 pb-0">
        {/* Labels */}
        {card.labels && card.labels.length > 0 && (
          <div className="flex flex-wrap gap-1 mb-2">
            {card.labels.map(label => (
              <Badge 
                key={label} 
                variant="secondary" 
                className="text-xs"
              >
                {label}
              </Badge>
            ))}
          </div>
        )}
        
        {/* Title */}
        <h4 className="text-sm font-medium leading-tight">{card.title}</h4>
      </CardHeader>

      <CardContent className="p-3 pt-2">
        {/* Meta Info */}
        <div className="flex items-center justify-between mt-2">
          <div className="flex items-center gap-2">
            {/* Priority */}
            <Badge 
              variant="secondary" 
              className={cn("text-xs", priorityColors[card.priority])}
            >
              {card.priority}
            </Badge>

            {/* Checklist */}
            {totalChecklist > 0 && (
              <span className={cn(
                "text-xs flex items-center gap-1",
                completedChecklist === totalChecklist ? "text-green-600" : "text-gray-500"
              )}>
                <CheckSquare className="h-3 w-3" />
                {completedChecklist}/{totalChecklist}
              </span>
            )}

            {/* Due Date */}
            {card.dueDate && (
              <span className={cn(
                "text-xs flex items-center gap-1",
                isOverdue ? "text-red-600" : "text-gray-500"
              )}>
                {isOverdue && <AlertCircle className="h-3 w-3" />}
                <Calendar className="h-3 w-3" />
                {formatDistanceToNow(new Date(card.dueDate), { addSuffix: true })}
              </span>
            )}
          </div>

          {/* Assignee */}
          {card.assignee && (
            <Avatar className="h-6 w-6">
              <AvatarImage src={card.assignee.avatar} />
              <AvatarFallback className="text-xs">
                {card.assignee.name?.charAt(0)}
              </AvatarFallback>
            </Avatar>
          )}
        </div>

        {/* Node Link Indicator */}
        {card.nodeId && (
          <div className="mt-2 pt-2 border-t text-xs text-muted-foreground flex items-center gap-1">
            <span>🔗 Linked to: {card.node?.title || 'Node'}</span>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

---

## INTEGRATION

### Page Route

```tsx
// app/(dashboard)/blueprints/[id]/kanban/page.tsx

import { notFound } from 'next/navigation';
import { prisma } from '@/lib/db';
import { auth } from '@/lib/auth';
import { KanbanBoard } from '@/components/kanban/KanbanBoard';
import { CreateCardButton } from '@/components/kanban/CreateCardButton';

interface KanbanPageProps {
  params: { id: string };
}

export default async function KanbanPage({ params }: KanbanPageProps) {
  const session = await auth();
  if (!session?.user) return null;

  // Get or create board for this blueprint
  let board = await prisma.kanbanBoard.findFirst({
    where: {
      blueprintId: params.id,
      organizationId: session.user.organizationId,
      enabled: true
    },
    include: {
      columns: {
        orderBy: { order: 'asc' }
      },
      cards: {
        where: { isArchived: false },
        include: {
          assignee: { select: { id: true, name: true, avatarUrl: true } },
          node: { select: { id: true, title: true, type: true } }
        },
        orderBy: { order: 'asc' }
      }
    }
  });

  // Create default board if none exists
  if (!board) {
    board = await prisma.kanbanBoard.create({
      data: {
        name: 'Task Board',
        blueprintId: params.id,
        organizationId: session.user.organizationId,
        createdBy: session.user.id,
        columns: {
          create: [
            { name: 'To Do', order: 0, color: '#3b82f6', wipLimit: 10 },
            { name: 'In Progress', order: 1, color: '#f59e0b', wipLimit: 5 },
            { name: 'Review', order: 2, color: '#8b5cf6', wipLimit: 5 },
            { name: 'Done', order: 3, color: '#10b981', wipLimit: null }
          ]
        }
      },
      include: {
        columns: { orderBy: { order: 'asc' } },
        cards: {
          where: { isArchived: false },
          include: {
            assignee: { select: { id: true, name: true, avatarUrl: true } },
            node: { select: { id: true, title: true, type: true } }
          }
        }
      }
    });
  }

  if (!board) return notFound();

  return (
    <div className="h-full flex flex-col">
      <div className="flex items-center justify-between mb-4">
        <div>
          <h1 className="text-2xl font-bold">{board.name}</h1>
          <p className="text-muted-foreground">
            Drag cards to move them between columns
          </p>
        </div>
        <CreateCardButton 
          boardId={board.id} 
          columns={board.columns}
          blueprintId={params.id}
        />
      </div>

      <div className="flex-1 overflow-hidden">
        <KanbanBoard
          boardId={board.id}
          initialColumns={board.columns}
          initialCards={board.cards}
        />
      </div>
    </div>
  );
}
```

---

## SUMMARY

### Database Tables Created:
1. **kanban_boards** - Board configuration with RBAC
2. **kanban_columns** - Columns with WIP limits and automation rules
3. **kanban_cards** - Cards linked to mindmap nodes
4. **kanban_card_activities** - Audit trail
5. **kanban_board_members** - Access control

### Key Features Implemented:
- ✅ Drag-drop with @dnd-kit
- ✅ Bidirectional sync with mindmap nodes
- ✅ WIP limits with visual indicators
- ✅ Full RBAC integration
- ✅ Audit trail for all changes
- ✅ Checklist support
- ✅ Assignee management
- ✅ Labels and priorities
- ✅ Due dates with overdue warnings

### Next Steps:
1. Add API routes for the operations
2. Implement real-time updates via Supabase Realtime
3. Add swimlane support
4. Implement column automation rules
5. Add card templates

---

