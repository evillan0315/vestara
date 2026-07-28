---
id: "SPEC-DATA-000"
title: "Data Dictionary — Complete Entity Definitions"
owner: "@backend-engineer"
status: "draft"
blueprint-ref: "12-data/DATA_ARCHITECTURE.md"
version: "1.0.0"
---

# Data Dictionary
## Complete Entity Definitions for the Vestara Platform

> **Every data entity in the Vestara platform is documented here — fields, types, relationships, validation rules, lifecycle, and security. This dictionary replaces guessing database schemas.**

---

## Naming Conventions

| Aspect | Convention | Example |
|--------|------------|---------|
| Table names | `snake_case` plural | `projects`, `activity_log` |
| Column names | `snake_case` | `created_at`, `sort_order` |
| Primary keys | `id TEXT PK` (UUID v7) | `id TEXT PRIMARY KEY NOT NULL` |
| Foreign keys | `table_id TEXT REFERENCES table(id)` | `project_id TEXT REFERENCES projects(id)` |
| Timestamps | `TEXT` (ISO 8601) | `created_at TEXT NOT NULL` |
| Soft delete | `deleted_at TEXT` | `deleted_at TEXT` |
| Status enums | `TEXT CHECK(...)` | `status TEXT CHECK(status IN ('todo','in_progress','review','done'))` |
| JSON storage | `TEXT` with `json_valid()` check | `metadata TEXT CHECK(json_valid(metadata))` |

### Index Naming
```
idx_{table}_{column}      → idx_tasks_project_id
idx_{table}_{col1}_{col2} → idx_tasks_project_id_status
uq_{table}_{column}       → uq_users_username
```

---

## 👤 User

```yaml
entity: User
table: users
purpose: "Represents a user of the Vestara platform"
lifecycle: Created → Active → Suspended → Deleted
security: "PII — encrypted at rest, audit-logged access"

fields:
  id:
    type: TEXT
    pk: true
    description: "UUID v7 primary key"
    
  username:
    type: TEXT
    unique: true
    not-null: true
    validation: "min(1) max(100) alphanumeric+dashes"
    description: "OS username (unique)"
    
  display_name:
    type: TEXT
    validation: "max(200)"
    description: "Optional display name"
    
  role:
    type: TEXT
    not-null: true
    enum: ['admin', 'editor', 'user']
    default: 'user'
    description: "Authorization role"
    
  password_hash:
    type: TEXT
    not-null: true
    description: "Argon2id hashed password"
    security: "NEVER returned in API responses"
    
  preferences:
    type: TEXT
    default: '{}'
    validation: "JSON object"
    description: "User preferences (theme, model, layout)"
    
  last_login_at:
    type: TEXT
    description: "ISO 8601 timestamp of last login"
    
  created_at:
    type: TEXT
    not-null: true
    description: "ISO 8601 creation timestamp"
    
  updated_at:
    type: TEXT
    not-null: true
    description: "ISO 8601 last update timestamp"
    
  deleted_at:
    type: TEXT
    description: "ISO 8601 soft-delete timestamp"

relationships:
  - entity: Session
    type: "has many"
    via: user_id
    on-delete: CASCADE
  - entity: Project
    type: "has many"
    via: user_id
    on-delete: CASCADE
  - entity: Notification
    type: "has many"
    via: user_id
    on-delete: CASCADE

indexes:
  - idx_users_username (UNIQUE)
  - idx_users_role
```

## 🔑 Session

```yaml
entity: Session
table: sessions
purpose: "User authentication session"

fields:
  id: { type: TEXT, pk: true }
  user_id: { type: TEXT, fk: "users(id) ON DELETE CASCADE", not-null: true }
  token: { type: TEXT, unique: true, not-null: true, security: "hashed" }
  expires_at: { type: TEXT, not-null: true }
  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }

indexes:
  - idx_sessions_user_id
  - uq_sessions_token
```

## 📁 Project

```yaml
entity: Project
table: projects
purpose: "Represents a user workspace project"
lifecycle: Created → Active → Archived → Deleted

fields:
  id:
    type: TEXT
    pk: true
    description: "UUID v7"
    
  user_id:
    type: TEXT
    fk: "users(id) ON DELETE CASCADE"
    not-null: true
    
  name:
    type: TEXT
    not-null: true
    validation: "min(1) max(200)"
    
  description:
    type: TEXT
    validation: "max(5000)"
    default: ""

  path:
    type: TEXT
    not-null: true
    description: "Filesystem path to project root"
    
  status:
    type: TEXT
    not-null: true
    enum: ['active', 'archived']
    default: 'active'

  metadata:
    type: TEXT
    default: '{}'
    validation: "JSON"
    description: "Flexible metadata (language, framework, etc.)"

  archived_at:
    type: TEXT
    description: "ISO 8601 archive timestamp"

  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }
  deleted_at: { type: TEXT }

relationships:
  - entity: Task
    type: "has many"
    via: project_id
    on-delete: CASCADE
  - entity: Knowledge
    type: "has many"
    via: project_id
    on-delete: CASCADE

indexes:
  - idx_projects_user_id
  - idx_projects_user_id_status
  - idx_projects_name
```

## ✅ Task

```yaml
entity: Task
table: tasks
purpose: "A task within a project"
lifecycle: Created → Active → Completed/Archived → Deleted

fields:
  id: { type: TEXT, pk: true }
  project_id: { type: TEXT, fk: "projects(id) ON DELETE CASCADE", not-null: true }
  title: { type: TEXT, not-null: true, validation: "min(1) max(500)" }
  description: { type: TEXT, validation: "max(10000)" }
  status:
    type: TEXT
    not-null: true
    enum: ['todo', 'in_progress', 'review', 'done']
    default: 'todo'
  assignee_id: { type: TEXT, description: "User UUID or external reference" }
  parent_id: { type: TEXT, fk: "tasks(id) ON DELETE SET NULL", description: "Self-referencing FK for sub-tasks" }
  tags: { type: TEXT, default: '[]', validation: "JSON array of strings" }
  estimated_hours: { type: REAL, validation: ">= 0" }
  logged_hours: { type: REAL, default: 0, validation: ">= 0" }
  sort_order: { type: INTEGER, default: 0, description: "Kanban drag-and-drop position" }
  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }
  deleted_at: { type: TEXT }

relationships:
  - entity: Task
    type: "has many (self)"
    via: parent_id
    description: "Sub-tasks"
  - entity: Activity
    type: "has many"
    via: resource
    description: "Task activity log entries"

indexes:
  - idx_tasks_project_id
  - idx_tasks_project_id_status
  - idx_tasks_parent_id
  - idx_tasks_assignee_id
```

## 🧠 Knowledge

```yaml
entity: Knowledge
table: knowledge
purpose: "A knowledge base entry with content, type, and optional embedding"
lifecycle: Created → Updated → Deleted

fields:
  id: { type: TEXT, pk: true }
  project_id: { type: TEXT, fk: "projects(id) ON DELETE CASCADE" }
  user_id: { type: TEXT, fk: "users(id) ON DELETE CASCADE" }
  content: { type: TEXT, not-null: true, description: "Document content" }
  type:
    type: TEXT
    not-null: true
    enum: ['document', 'note', 'reference', 'code', 'tutorial', 'other']
  title: { type: TEXT, validation: "max(500)" }
  tags: { type: TEXT, default: '[]', validation: "JSON array" }
  metadata: { type: TEXT, default: '{}', validation: "JSON" }
  embedding: { type: BLOB, description: "Vector embedding (Gen 2)" }
  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }
  deleted_at: { type: TEXT }

indexes:
  - idx_knowledge_project_id
  - idx_knowledge_user_id
  - idx_knowledge_type
  - idx_knowledge_fts (FTS5 on content + title)
```

## 💭 Memory

```yaml
entity: Memory
table: memories
purpose: "User memory with importance scoring and consolidation tracking"
lifecycle: Created → Consolidated → Archived → Deleted

fields:
  id: { type: TEXT, pk: true }
  user_id: { type: TEXT, fk: "users(id) ON DELETE CASCADE", not-null: true }
  project_id: { type: TEXT, fk: "projects(id) ON DELETE SET NULL" }
  type:
    type: TEXT
    not-null: true
    enum: ['fact', 'preference', 'event', 'decision', 'conversation']
  content: { type: TEXT, not-null: true }
  summary: { type: TEXT, description: "Consolidated summary" }
  importance: { type: REAL, default: 1.0, validation: "0.0 - 10.0" }
  embedding: { type: BLOB, description: "Vector embedding (Gen 2)" }
  metadata: { type: TEXT, default: '{}', validation: "JSON" }
  consolidated_at: { type: TEXT, description: "Last consolidation timestamp" }
  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }
  deleted_at: { type: TEXT }

indexes:
  - idx_memories_user_id
  - idx_memories_user_id_type
  - idx_memories_importance
  - idx_memories_fts (FTS5 on content + summary)
```

## 🔔 Notification

```yaml
entity: Notification
table: notifications
purpose: "In-app notification"
lifecycle: Created → Read → Archived → Deleted

fields:
  id: { type: TEXT, pk: true }
  user_id: { type: TEXT, fk: "users(id) ON DELETE CASCADE", not-null: true }
  type: { type: TEXT, not-null: true, description: "Notification type (task:assigned, etc.)" }
  priority: { type: TEXT, not-null: true, enum: ['low', 'normal', 'high', 'critical'] }
  title: { type: TEXT, not-null: true }
  body: { type: TEXT }
  link: { type: TEXT, description: "Deep link to notification context" }
  read_at: { type: TEXT }
  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }
  deleted_at: { type: TEXT }

indexes:
  - idx_notifications_user_id
  - idx_notifications_user_id_priority
  - idx_notifications_user_id_read
```

## 📋 Activity

```yaml
entity: Activity
table: activity_log
purpose: "Immutable activity record for audit and timeline"
lifecycle: Created (append-only, never deleted)

fields:
  id: { type: TEXT, pk: true }
  user_id: { type: TEXT, fk: "users(id) ON DELETE CASCADE", not-null: true }
  action: { type: TEXT, not-null: true, description: "Event name (task:created, etc.)" }
  resource: { type: TEXT, not-null: true, description: "Resource identifier (task:<id>)" }
  metadata: { type: TEXT, default: '{}', validation: "JSON" }
  created_at: { type: TEXT, not-null: true }

indexes:
  - idx_activity_user_id
  - idx_activity_action
  - idx_activity_resource
  - idx_activity_created_at
```

## 🤖 Agent

```yaml
entity: Agent
table: agents
purpose: "An AI agent with configuration, tools, and execution history"
lifecycle: Created → Active → Disabled → Archived → Deleted

fields:
  id: { type: TEXT, pk: true }
  user_id: { type: TEXT, fk: "users(id) ON DELETE CASCADE", not-null: true }
  project_id: { type: TEXT, fk: "projects(id) ON DELETE SET NULL" }
  name: { type: TEXT, not-null: true, validation: "min(1) max(200)" }
  description: { type: TEXT }
  model: { type: TEXT, not-null: true, description: "Model identifier (openai/gpt-4, etc.)" }
  config: { type: TEXT, not-null: true, validation: "JSON", description: "Agent configuration JSON" }
  tools: { type: TEXT, not-null: true, default: '[]', validation: "JSON array" }
  memory_enabled: { type: INTEGER, default: 1, description: "Boolean flag" }
  status: { type: TEXT, default: 'active', enum: ['active', 'disabled', 'archived'] }
  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }
  deleted_at: { type: TEXT }

indexes:
  - idx_agents_user_id
  - idx_agents_project_id
```

## ⚙️ Setting

```yaml
entity: Setting
table: settings
purpose: "Key-value settings with scope scoping"

fields:
  key: { type: TEXT, not-null: true }
  value: { type: TEXT, not-null: true }
  scope:
    type: TEXT
    not-null: true
    enum: ['global', 'user', 'project']
    default: 'global'
  user_id: { type: TEXT, fk: "users(id) ON DELETE CASCADE" }
  project_id: { type: TEXT, fk: "projects(id) ON DELETE CASCADE" }
  created_at: { type: TEXT, not-null: true }
  updated_at: { type: TEXT, not-null: true }

primary_key: ['key', 'scope', 'user_id', 'project_id']
indexes:
  - idx_settings_scope
```

---

## Entity Relationship Diagram (Text)

```
users ──┬── sessions
        ├── projects ──┬── tasks (self-ref via parent_id)
        │              ├── knowledge
        │              └── activity_log
        ├── notifications
        ├── memories
        ├── agents ──── agent_executions
        └── settings
```

---

## Migration Rules

| Rule | Detail |
|------|--------|
| **Additive only** | Never remove columns — use `deleted_at` for soft delete |
| **Additive ALTER TABLE** | `ALTER TABLE ... ADD COLUMN` with `PRAGMA table_info` check |
| **Default values** | New columns must have defaults for existing rows |
| **Index creation** | `CREATE INDEX IF NOT EXISTS ...` |
| **Data migration** | Separate script, run after schema migration |
| **Rollback** | Every migration must have a rollback script |
| **Version tracking** | `_migrations` table with applied timestamps + checksums |

---

*This data dictionary is the authoritative source for all Vestara data models. Every entity, field, relationship, and constraint is defined here before any migration is written.*
