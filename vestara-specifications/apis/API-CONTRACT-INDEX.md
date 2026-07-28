---
id: "SPEC-API-000"
title: "API Contract Library Index"
owner: "@backend-engineer"
status: "draft"
blueprint-ref: "04-platform/01-platform-overview.md"
version: "1.0.0"
---

# API Contract Library
## Complete API Contracts for All Vestara Services

> **Every public API in the Vestara platform is defined here before implementation. These contracts are the source of truth for API behavior — endpoints, DTOs, validation, errors, versioning, and authentication.**

---

## API Design Principles

| Principle | Description |
|-----------|-------------|
| **RESTful** | Resource-oriented endpoints, standard HTTP methods |
| **Versioned** | `/api/v1/` prefix, major version in URL path |
| **Type-Safe** | All DTOs defined as TypeScript types + Zod schemas |
| **Consistent** | Uniform error format, pagination, response envelopes |
| **Documented** | OpenAPI 3.0 spec generated from Zod schemas |

## API Response Envelope

```typescript
// Success Response
{
  "success": true,
  "data": T,                   // Typed response data
  "meta": {                    // Optional metadata
    "version": "1.0.0",
    "requestId": "uuid-v7"
  }
}

// Error Response
{
  "success": false,
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "Project with id 'xyz' not found",
    "statusCode": 404,
    "details": {}               // Optional validation errors
  },
  "meta": {
    "version": "1.0.0",
    "requestId": "uuid-v7"
  }
}

// Paginated Response
{
  "success": true,
  "data": T[],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 1234,
    "totalPages": 25
  }
}
```

## Standard Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `VALIDATION_ERROR` | 400 | Zod validation failed |
| `UNAUTHORIZED` | 401 | Missing or invalid JWT |
| `FORBIDDEN` | 403 | Authenticated but insufficient role |
| `NOT_FOUND` | 404 | Resource not found |
| `RATE_LIMITED` | 429 | Too many requests |
| `PROVIDER_ERROR` | 502 | AI provider error |
| `INTERNAL_ERROR` | 500 | Unexpected server error |

---

## API Index

### Identity & Auth

| Method | Endpoint | Auth | Rate Limit | DTOs |
|--------|----------|------|------------|------|
| `GET` | `/api/v1/auth/os-user` | Public | 10/min | — |
| `POST` | `/api/v1/auth/os-login` | Public | 10/min | LoginRequest, LoginResponse |
| `POST` | `/api/v1/auth/os-auto-login` | Public | 10/min | AutoLoginRequest, LoginResponse |
| `GET` | `/api/v1/auth/me` | Required | 60/min | UserResponse |
| `DELETE` | `/api/v1/auth/logout` | Required | 60/min | — |
| `POST` | `/api/v1/auth/refresh` | Required | 60/min | TokenResponse |

### Projects

| Method | Endpoint | Auth | Rate Limit | DTOs |
|--------|----------|------|------------|------|
| `GET` | `/api/v1/projects` | Required | 120/min | ProjectFilter, ProjectListResponse |
| `POST` | `/api/v1/projects` | Required | 30/min | CreateProjectRequest, ProjectResponse |
| `GET` | `/api/v1/projects/:id` | Required | 120/min | ProjectResponse |
| `PATCH` | `/api/v1/projects/:id` | Required | 30/min | UpdateProjectRequest, ProjectResponse |
| `DELETE` | `/api/v1/projects/:id` | Required | 10/min | — |
| `POST` | `/api/v1/projects/:id/archive` | Required | 10/min | ProjectResponse |
| `POST` | `/api/v1/projects/:id/clone` | Required | 10/min | CloneProjectRequest, ProjectResponse |

### Tasks

| Method | Endpoint | Auth | Rate Limit | DTOs |
|--------|----------|------|------------|------|
| `GET` | `/api/v1/projects/:id/tasks` | Required | 120/min | TaskFilter, TaskListResponse |
| `POST` | `/api/v1/projects/:id/tasks` | Required | 60/min | CreateTaskRequest, TaskResponse |
| `PATCH` | `/api/v1/projects/:id/tasks/:taskId` | Required | 60/min | UpdateTaskRequest, TaskResponse |
| `DELETE` | `/api/v1/projects/:id/tasks/:taskId` | Required | 30/min | — |
| `POST` | `/api/v1/projects/:id/tasks/bulk` | Required | 30/min | BulkUpdateRequest, TaskListResponse |
| `GET` | `/api/v1/projects/:id/tasks/:taskId/subtasks` | Required | 120/min | TaskListResponse |

### Activity

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `GET` | `/api/v1/projects/:id/activity` | Required | 60/min |

### Knowledge

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `GET` | `/api/v1/knowledge` | Required | 120/min |
| `POST` | `/api/v1/knowledge` | Required | 30/min |
| `GET` | `/api/v1/knowledge/:id` | Required | 120/min |
| `DELETE` | `/api/v1/knowledge/:id` | Required | 30/min |
| `POST` | `/api/v1/knowledge/search` | Required | 60/min |

### Memory

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `POST` | `/api/v1/memory` | Required | 60/min |
| `GET` | `/api/v1/memory/search` | Required | 120/min |
| `DELETE` | `/api/v1/memory/:id` | Required | 30/min |

### Chat

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `POST` | `/api/v1/chat` | Required | 100/min |
| `GET` | `/api/v1/chat/history` | Required | 60/min |
| `GET` | `/api/v1/chat/:id` | Required | 120/min |
| `DELETE` | `/api/v1/chat/:id` | Required | 30/min |

### Agents

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `GET` | `/api/v1/agents` | Required | 60/min |
| `POST` | `/api/v1/agents` | Required | 30/min |
| `GET` | `/api/v1/agents/:id` | Required | 60/min |
| `PATCH` | `/api/v1/agents/:id` | Required | 30/min |
| `DELETE` | `/api/v1/agents/:id` | Required | 10/min |
| `POST` | `/api/v1/agents/:id/execute` | Required | 30/min |

### Providers

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `GET` | `/api/v1/providers` | Required | 60/min |
| `GET` | `/api/v1/providers/models` | Required | 60/min |
| `POST` | `/api/v1/providers/opencode/start` | Public | 10/min |
| `POST` | `/api/v1/providers/check` | Required | 30/min |

### Notifications

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `GET` | `/api/v1/notifications` | Required | 60/min |
| `PATCH` | `/api/v1/notifications/:id/read` | Required | 60/min |
| `POST` | `/api/v1/notifications/read-all` | Required | 30/min |

### Settings

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `GET` | `/api/v1/settings` | Required | 60/min |
| `PATCH` | `/api/v1/settings/:key` | Required | 60/min |

### System

| Method | Endpoint | Auth | Rate Limit |
|--------|----------|------|------------|
| `GET` | `/api/v1/system/health` | Public | 30/min |
| `GET` | `/api/v1/system/info` | Public | 30/min |

---

## WebSocket API

| Event | Direction | Payload | Description |
|-------|-----------|---------|-------------|
| `auth:challenge` | Server → Client | `{ nonce }` | Connection authentication |
| `auth:response` | Client → Server | `{ signature }` | Auth challenge response |
| `auth:success` | Server → Client | `{ user, token }` | Auth successful |
| `*` | Server → Client | Per event | Broadcast of system events |

---

## Detailed API Contract (Example: Chat API)

### `POST /api/v1/chat`

```yaml
endpoint: "/api/v1/chat"
method: POST
auth: required
rateLimit: 100/min
description: "Send a message to the AI and receive a streaming or non-streaming response"

request:
  body:
    contentType: "application/json"
    schema: "SendMessageRequest"
    validation: "Zod"

response:
  status: 200
  contentType: "application/json"
  schema: "SendMessageResponse"
  streaming: "text/event-stream if stream: true"

errors:
  - code: "VALIDATION_ERROR"
    status: 400
    description: "Invalid message format or missing required fields"
  - code: "UNAUTHORIZED"
    status: 401
  - code: "RATE_LIMITED"
    status: 429
  - code: "PROVIDER_ERROR"
    status: 502
    description: "AI provider returned an error"
```

```typescript
// DTOs
interface SendMessageRequest {
  conversationId?: string;     // Optional — creates new if omitted
  content: string;             // Message text
  model?: string;              // Optional — override model selection
  stream: boolean;             // Enable streaming response
  attachments?: Attachment[];
  tools?: string[];            // Enable specific tools
}

interface SendMessageResponse {
  conversationId: string;      // Updated or created conversation
  messageId: string;           // AI response message ID
  content: string;             // AI response (non-streaming)
  tokens: number;
  cost: number;
  latency: number;
  tools?: ToolCallResult[];    // Tool execution results
}

interface StreamChunk {
  type: 'token' | 'tool-start' | 'tool-end' | 'error' | 'done';
  data: string | ToolCall | ToolResult | ErrorEvent;
  index: number;
}
```

---

## DTO Schemas (All Defined Here)

### Identity DTOs

```typescript
interface LoginRequest {
  username: string;      // z.string().min(1).max(100)
  password: string;      // z.string().min(1)
}

interface LoginResponse {
  token: string;         // JWT
  user: User;
  expiresIn: number;     // seconds
}

interface User {
  id: string;            // UUID v7
  username: string;
  role: 'admin' | 'editor' | 'user';
  createdAt: string;
}
```

### Project DTOs

```typescript
interface CreateProjectRequest {
  name: string;              // z.string().min(1).max(200)
  description?: string;      // z.string().max(5000).optional()
  path: string;              // z.string().min(1)
}

interface UpdateProjectRequest {
  name?: string;
  description?: string;
  path?: string;
}

interface ProjectResponse {
  id: string;
  name: string;
  description: string;
  path: string;
  archivedAt?: string;
  createdAt: string;
  updatedAt: string;
}

interface ProjectFilter {
  status?: 'active' | 'archived';
  search?: string;
  page?: number;
  limit?: number;
}
```

### Task DTOs

```typescript
interface CreateTaskRequest {
  title: string;                 // z.string().min(1).max(500)
  description?: string;          // z.string().max(10000).optional()
  status?: 'todo' | 'in_progress' | 'review' | 'done';
  parentId?: string;             // UUID v7 — for sub-tasks
  assigneeId?: string;           // UUID v7
  tags?: string[];               // z.array(z.string()).max(10)
  estimatedHours?: number;       // z.number().min(0).max(10000).optional()
  sortOrder?: number;            // z.number().int().default(0)
}

interface UpdateTaskRequest {
  title?: string;
  description?: string;
  status?: 'todo' | 'in_progress' | 'review' | 'done';
  assigneeId?: string;
  tags?: string[];
  estimatedHours?: number;
  loggedHours?: number;
  sortOrder?: number;
}

interface TaskResponse {
  id: string;
  projectId: string;
  title: string;
  description: string;
  status: 'todo' | 'in_progress' | 'review' | 'done';
  assigneeId?: string;
  parentId?: string;
  tags: string[];
  estimatedHours: number;
  loggedHours: number;
  sortOrder: number;
  createdAt: string;
  updatedAt: string;
  subTasks?: TaskResponse[];     // Only in subtask endpoint
}
```

---

## API Versioning and Deprecation

| Policy | Rule |
|--------|------|
| **Major version in URL** | `/api/v1/`, `/api/v2/` |
| **Minor/patch changes** | Backward-compatible, no URL change |
| **Breaking changes** | New major version, old deprecated for 2 releases |
| **Deprecation header** | `Sunset: Sat, 01 Jan 2026 00:00:00 GMT` |
| **Migration guide** | Required with every major version |

---

*This API contract library is the authoritative source for all Vestara API behavior. Every endpoint, DTO, and error code is defined here before any code is written.*
