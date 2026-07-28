---
id: "FND-003"
title: "Service Catalog — Complete Service Definitions"
owner: "@backend-engineer"
status: "ratified"
blueprint-ref: "04-platform/PLATFORM_OVERVIEW.md"
foundation-version: "1.0.0"
---

# Service Catalog
## Every Service in the Vestara Platform — Defined Before Implementation

> **Everything in Vestara is a service. Every service has a purpose, API, events, dependencies, owner, SLA, and security model. No service is implemented before it is defined here.**

---

## Service Architecture

```typescript
interface VestaraService {
  id: string;                    // Unique service identifier
  version: string;               // Semantic version
  status: 'starting' | 'running' | 'degraded' | 'stopped';
  
  initialize(config?: Record<string, unknown>): Promise<void>;
  start(): Promise<void>;
  stop(): Promise<void>;
  health(): Promise<HealthStatus>;
  dispose(): Promise<void>;
}
```

---

## 📋 Service Catalog

### SVC-001: Identity Service

```yaml
id: "SVC-001"
name: "Identity Service"
purpose: "User authentication, authorization, session management, role-based access control"
owner: "@backend-engineer"
sla: "99.9% uptime, <50ms p95 latency"

dependencies:
  - "Storage (SQLite)"

capabilities:
  - "OS-based authentication (username/password)"
  - "JWT session management"
  - "Role-based authorization (admin, editor, user)"
  - "Session refresh and revocation"

events:
  publishes:
    - "auth:login.success"
    - "auth:login.failed"
    - "auth:token.refreshed"
    - "auth:token.revoked"
    - "auth:session.expired"
  subscribes: []

api:
  - "POST /api/v1/auth/os-login"
  - "POST /api/v1/auth/os-auto-login"
  - "GET /api/v1/auth/me"
  - "DELETE /api/v1/auth/logout"
  - "POST /api/v1/auth/refresh"

security:
  - "Passwords: Argon2id hashed"
  - "Tokens: JWT with RS256, 24h expiry"
  - "Rate limiting: 10 attempts/min per IP"
  - "Session rotation on privilege escalation"

health:
  - "Database connectivity"
  - "Token signing key availability"
```

### SVC-002: Workspace Service

```yaml
id: "SVC-002"
name: "Workspace Service"
purpose: "Workspace and project lifecycle management, task tracking, activity timeline"
owner: "@backend-engineer"
sla: "99.9% uptime, <100ms p95 latency"

dependencies:
  - "Storage (SQLite)"
  - "Event Bus"
  - "Filesystem Service"

capabilities:
  - "Workspace CRUD"
  - "Project CRUD with archive/clone"
  - "Task CRUD with Kanban, sub-tasks, tags, time tracking"
  - "Bulk task operations"
  - "Activity timeline"
  - "Project sync (.vestara folder)"

events:
  publishes:
    - "project:created", "project:updated", "project:archived", "project:deleted"
    - "task:created", "task:updated", "task:status.changed", "task:assigned"
  subscribes:
    - "auth:login.success"  # Load last workspace

api:
  - Projects: CRUD + archive + clone
  - Tasks: CRUD + bulk + subtasks
  - Activity: GET timeline

security:
  - "Project isolation: user scoped"
  - "Validation: Zod on all inputs"
  - "Rate limiting: 120/min reads, 30/min writes"
```

### SVC-003: Conversation Service

```yaml
id: "SVC-003"
name: "Conversation Service"
purpose: "Multi-turn AI conversation management, context assembly, streaming responses"
owner: "@ai-core-team"
sla: "99.5% uptime, <500ms p95 time-to-first-token"

dependencies:
  - "Event Bus"
  - "Memory Service"
  - "Knowledge Service"
  - "Provider Service"
  - "Agent Service"

capabilities:
  - "Conversation CRUD"
  - "Context assembly (system + memories + knowledge + history)"
  - "Context window optimization (token budgeting)"
  - "Streaming responses via SSE/WebSocket"
  - "Tool orchestration within conversation"
  - "Conversation history search"
  - "Conversation branching (Gen 2)"

events:
  publishes:
    - "conversation:started"
    - "conversation:message.sent"
    - "conversation:response.start"
    - "conversation:response.complete"
    - "conversation:tool.executing"
    - "conversation:tool.completed"
    - "conversation:error"
  subscribes:
    - "memory:consolidated"  # Refresh context

api:
  - "POST /api/v1/chat"
  - "GET /api/v1/chat/history"
  - "GET /api/v1/chat/:id"
  - "DELETE /api/v1/chat/:id"
  - "WebSocket /ws (streaming)"

security:
  - "Message sanitization (XSS prevention)"
  - "PII detection on output"
  - "Rate limiting: 100/min per user"
  - "Content safety filtering"
```

### SVC-004: Memory Service

```yaml
id: "SVC-004"
name: "Memory Service"
purpose: "Cross-session persistent memory with consolidation, importance scoring, and search"
owner: "@ai-core-team"
sla: "99.9% uptime, <50ms p95 search latency"

dependencies:
  - "Storage (SQLite)"
  - "Event Bus"
  - "Embedding Service (Gen 2)"

capabilities:
  - "Store memories (fact, preference, event, decision)"
  - "Importance scoring (recency, frequency, feedback, novelty, impact)"
  - "Scheduled consolidation (every 50 interactions)"
  - "Hybrid search (FTS + vector)"
  - "Context assembly for conversations"
  - "Memory pruning and archiving"

events:
  publishes:
    - "memory:stored"
    - "memory:searched"
    - "memory:consolidated"
    - "memory:deleted"
  subscribes:
    - "conversation:response.complete"  # Extract memories
    - "project:created"  # Initialize project memory

api:
  - "POST /api/v1/memory"
  - "GET /api/v1/memory/search"
  - "DELETE /api/v1/memory/:id"

security:
  - "Memory isolation: scoped to user"
  - "No PII stored in memory (filtered at ingestion)"
  - "User can delete all memories"
```

### SVC-005: Knowledge Service

```yaml
id: "SVC-005"
name: "Knowledge Service"
purpose: "Document storage, full-text/vector search, RAG pipeline"
owner: "@ai-core-team"
sla: "99.9% uptime, <100ms p95 search latency"

dependencies:
  - "Storage (SQLite + FTS5)"
  - "Event Bus"
  - "Filesystem Service (auto-indexing)"

capabilities:
  - "Document CRUD with metadata"
  - "Full-text search (SQLite FTS5)"
  - "Vector search (Gen 2)"
  - "RAG pipeline: query → retrieve → rerank → generate"
  - "Auto-indexing via filesystem watcher"
  - "Document chunking for RAG"

events:
  publishes:
    - "knowledge:added"
    - "knowledge:searched"
    - "knowledge:updated"
    - "knowledge:deleted"
    - "knowledge:indexed"
  subscribes:
    - "filesystem:file.changed"  # Auto-index

api:
  - "POST /api/v1/knowledge"
  - "GET /api/v1/knowledge/search"
  - "GET /api/v1/knowledge/:id"
  - "DELETE /api/v1/knowledge/:id"

security:
  - "Content isolation: scoped to project"
  - "No sensitive data in knowledge (filtered)"
```

### SVC-006: Agent Service

```yaml
id: "SVC-006"
name: "Agent Service"
purpose: "Agent creation, execution, tool registration, lifecycle management"
owner: "@ai-core-team"
sla: "99.5% uptime, <200ms p95 execution startup"

dependencies:
  - "Provider Service"
  - "Tool Runtime"
  - "Memory Service"
  - "Knowledge Service"
  - "Event Bus"

capabilities:
  - "Agent CRUD"
  - "Agent execution with streaming progress"
  - "Tool registration and invocation"
  - "Agent persistence and state restore"
  - "Execution sandbox with resource limits"
  - "Multi-agent orchestration (Gen 3)"

events:
  publishes:
    - "agent:created", "agent:updated", "agent:deleted"
    - "agent:execution.started"
    - "agent:execution.progress"
    - "agent:execution.completed"
    - "agent:execution.failed"
  subscribes:
    - "mission:planning.completed"  # Assign tasks to agents

api:
  - Agents: CRUD
  - "POST /api/v1/agents/:id/execute"

security:
  - "Sandboxed execution (no host access)"
  - "Tool permissions: user-approve for dangerous tools"
  - "Resource limits: 5min max execution, 50MB memory"
  - "Audit logging: all executions recorded"
```

### SVC-007: Provider Service

```yaml
id: "SVC-007"
name: "Provider Service"
purpose: "AI provider management, routing, fallback, health monitoring, cost tracking"
owner: "@ai-core-team"
sla: "99.9% uptime, <10ms p95 routing latency"

dependencies:
  - "Configuration"
  - "Event Bus"

capabilities:
  - "Provider registration and configuration"
  - "Provider health monitoring"
  - "Model discovery and listing"
  - "Request routing (model → provider)"
  - "Fallback chains (primary → secondary → tertiary)"
  - "Cost estimation and tracking"
  - "Rate limiting per provider"

events:
  publishes:
    - "provider:changed"
    - "provider:status.changed"
    - "model:loaded"
    - "model:unloaded"
  subscribes: []

api:
  - "GET /api/v1/providers"
  - "GET /api/v1/providers/models"
  - "POST /api/v1/providers/opencode/start"
  - "POST /api/v1/providers/check"

security:
  - "API keys: stored in OS keychain, never logged"
  - "Provider isolation: no cross-provider data leakage"
  - "Fallback: never fail over to untrusted provider"
```

### SVC-008: Notification Service

```yaml
id: "SVC-008"
name: "Notification Service"
purpose: "In-app notifications, activity log, priority-based delivery"
owner: "@backend-engineer"
sla: "99.9% uptime, <10ms p95 delivery latency"

dependencies:
  - "Storage (SQLite)"
  - "Event Bus"

capabilities:
  - "Notification creation with priority levels"
  - "Notification delivery (in-app + WebSocket)"
  - "Notification read/unread tracking"
  - "Notification archiving"
  - "Activity log (append-only)"

events:
  publishes:
    - "notification:created"
    - "notification:read"
    - "notification:archived"
  subscribes:
    - "*"  # Subscribe to all events for notifications

api:
  - "GET /api/v1/notifications"
  - "PATCH /api/v1/notifications/:id/read"
  - "POST /api/v1/notifications/read-all"

security:
  - "Notification isolation: scoped to user"
```

### SVC-009: Settings Service

```yaml
id: "SVC-009"
name: "Settings Service"
purpose: "Key-value settings with global/user/project scoping, schema validation, hot-reload"
owner: "@backend-engineer"
sla: "99.99% uptime, <5ms p95 latency"

dependencies:
  - "Storage (SQLite)"

capabilities:
  - "Setting CRUD with schema validation"
  - "Scoped settings (global, user, project)"
  - "Setting change notifications"
  - "Settings export/import"

events:
  publishes:
    - "settings:changed"
  subscribes: []

api:
  - "GET /api/v1/settings"
  - "PATCH /api/v1/settings/:key"

security:
  - "Setting isolation by scope"
  - "Sensitive settings encrypted at rest"
```

### SVC-010: Filesystem Service

```yaml
id: "SVC-010"
name: "Filesystem Service"
purpose: "File operations, .vestara folder management, file watching, sync layer"
owner: "@backend-engineer"
sla: "99.9% uptime, <50ms p95 latency"

dependencies:
  - "Event Bus"

capabilities:
  - "File read/write/delete"
  - "Directory operations"
  - ".vestara folder management"
  - "File watching (inotify)"
  - "Git integration (init, status, commit)"
  - "Path traversal protection"

events:
  publishes:
    - "filesystem:file.created"
    - "filesystem:file.changed"
    - "filesystem:file.deleted"
  subscribes: []

security:
  - "Path traversal: blocked"
  - "Sandbox: operations limited to project directory"
  - "Binary files: size limits, malware scanning (Gen 2)"
```

---

## Service Dependency Graph

```
Identity ← no deps
Settings ← no deps
Filesystem ← no deps
Knowledge ← Filesystem, Events
Memory ← Events (Gen 2: + Embedding)
Provider ← Configuration
Notification ← Events (subscribes to all)
Workspace ← Storage, Events, Filesystem
Conversation ← Memory, Knowledge, Provider, Agent, Events
Agent ← Provider, Tool Runtime, Memory, Knowledge, Events
Application ← Workspace, Settings
```

---

## Service SLA Matrix

| Service | Uptime | Latency (p95) | Criticality |
|---------|--------|---------------|-------------|
| Identity | 99.9% | 50ms | Critical |
| Workspace | 99.9% | 100ms | High |
| Conversation | 99.5% | 500ms | Critical |
| Memory | 99.9% | 50ms | High |
| Knowledge | 99.9% | 100ms | High |
| Agent | 99.5% | 200ms | High |
| Provider | 99.9% | 10ms | Critical |
| Notification | 99.9% | 10ms | Low |
| Settings | 99.99% | 5ms | Medium |
| Filesystem | 99.9% | 50ms | Medium |

---

*This service catalog defines every service in the Vestara platform. No service is implemented before it is defined here. Each service has a clear owner, SLA, and set of capabilities that the rest of the platform depends on.*
