---
id: "FND-001"
title: "Vestara Object Model (VOM) — Core Platform Primitives"
owner: "@chief-architect"
status: "ratified"
blueprint-ref: "04-platform/PLATFORM_OVERVIEW.md"
foundation-version: "1.0.0"
---

# Vestara Object Model (VOM)
## The Core Primitives of the Vestara AI Operating Platform

> **Every concept in Vestara is an object. This document defines every core primitive — its purpose, lifecycle, capabilities, relationships, state machine, and interface. No code should be written that references a concept not defined here.**

---

## 🏛️ Object Hierarchy

```
Vestara (Platform Instance)
├── Organization (Tenant)
│   ├── User (Identity)
│   ├── Workspace (Environment)
│   │   ├── Project (Unit of Work)
│   │   │   ├── Task (Unit of Execution)
│   │   │   ├── Document (Persistent Content)
│   │   │   ├── Conversation (AI Dialogue)
│   │   │   │   └── Message (Exchange Unit)
│   │   │   ├── Knowledge (Structured Information)
│   │   │   └── Artifact (Output)
│   │   ├── Memory (Cross-Session Persistence)
│   │   │   ├── Fact (Declarative)
│   │   │   ├── Preference (User Tuning)
│   │   │   ├── Event (Episodic)
│   │   │   └── Decision (Rationale)
│   │   └── Context (Session State)
│   ├── Agent (Autonomous Actor)
│   │   ├── Tool (Capability Boundary)
│   │   └── Mission (Long-Running Goal)
│   ├── Provider (External AI Source)
│   │   └── Model (AI Capability)
│   ├── Plugin (Extension)
│   ├── Capability (Functional Unit)
│   ├── Event (Notification)
│   ├── Workflow (Automation Process)
│   └── Mission (Business Outcome)
```

---

## 📦 Core Object Specifications

---

### 1. 🔷 Organization

```yaml
id: "VOM-ORG"
name: "Organization"
purpose: "Multi-tenant boundary. Owns users, workspaces, billing, and policies."
lifecycle: "Provisioned → Active → Suspended → Deactivated"

properties:
  id: "UUID v7 — globally unique"
  name: "Human-readable organization name"
  slug: "URL-safe identifier"
  tier: "community | pro | team | enterprise"
  features: "Feature flags based on tier"
  settings: "Organization-wide configuration"
  created_at: "ISO 8601"
  updated_at: "ISO 8601"

capabilities:
  - "Create and manage workspaces"
  - "Invite and manage users"
  - "Configure organization policies"
  - "Apply feature flags"
  - "Billing and subscription management"

state-machine:
  - Provisioned: "Initial state, setup pending"
  - Active: "Fully operational"
  - Suspended: "Payment failure, policy violation"
  - Deactivated: "Organization closed, data retention period"

relationships:
  - "has_many: User"
  - "has_many: Workspace"
  - "has_many: Plugin (org-level)"

security:
  - "Organization isolation: data never crosses org boundaries"
  - "Role-based access: owner, admin, member, viewer"
  - "Audit logging: all configuration changes recorded"

events:
  - "organization:provisioned"
  - "organization:activated"
  - "organization:suspended"
  - "organization:deactivated"
  - "organization:settings.changed"
  - "organization:member.added"
  - "organization:member.removed"
```

---

### 2. 👤 User

```yaml
id: "VOM-USER"
name: "User"
purpose: "Identity, authentication, preferences, and capability boundary."
lifecycle: "Created → Active → Suspended → Deleted"

properties:
  id: "UUID v7"
  username: "Unique, alphanumeric"
  display_name: "Optional, friendly name"
  role: "admin | editor | user"
  preferences: "Theme, model defaults, layout, shortcuts"
  last_login: "ISO 8601"
  created_at: "ISO 8601"

capabilities:
  - "Authenticate (OS login, JWT, future: SSO, MFA)"
  - "Own workspaces and projects"
  - "Configure personal preferences"
  - "Manage personal plugins"
  - "Access personal memory and knowledge"

state-machine:
  - Created: "Account created, not yet active"
  - Active: "Can authenticate and use platform"
  - Suspended: "Temporary restriction"
  - Deleted: "Account removed, data retention period"

relationships:
  - "belongs_to: Organization"
  - "has_many: Workspace"
  - "has_many: Memory"
  - "has_many: Agent"
  - "has_many: Session"

security:
  - "Password: Argon2id hashed"
  - "Session: JWT with rotation"
  - "Rate limiting: per-user across all endpoints"
  - "Data isolation: user data never accessible by other users"

interface:
  initialize(): Promise<void>
  authenticate(credentials: Credentials): Promise<Session>
  getPreferences(): Promise<UserPreferences>
  updatePreferences(prefs: Partial<UserPreferences>): Promise<void>
  getCapabilities(): Promise<Capability[]>
```

---

### 3. 🏢 Workspace

```yaml
id: "VOM-WORKSPACE"
name: "Workspace"
purpose: "The user's primary environment. Contains projects, tools, agents, memory, and knowledge."
lifecycle: "Created → Active → Archived → Deleted"

properties:
  id: "UUID v7"
  name: "Workspace name"
  layout: "Panel configuration, sidebar state"
  theme: "Dark | Light | System"
  active_project_id: "Currently focused project"
  created_at: "ISO 8601"

capabilities:
  - "Host projects"
  - "Provide AI chat interface"
  - "Manage memory and knowledge"
  - "Run agents"
  - "Manage files and documents"
  - "Configure workspace settings"

state-machine:
  - Created: "Initial setup"
  - Active: "Primary workspace state"
  - Archived: "Preserved but inactive"
  - Deleted: "Removed, data retention period"

relationships:
  - "belongs_to: User (or Organization for team workspaces)"
  - "has_one: Context (active session state)"
  - "has_many: Project"
  - "has_many: Agent"
  - "has_many: Memory"

events:
  - "workspace:opened"
  - "workspace:closed"
  - "workspace:layout.changed"
  - "workspace:theme.changed"
  - "workspace:archived"
  - "workspace:deleted"
```

---

### 4. 📁 Project

```yaml
id: "VOM-PROJECT"
name: "Project"
purpose: "Unit of work. Contains tasks, documents, conversations, knowledge, and artifacts."
lifecycle: "Created → Active → Archived → Deleted"

properties:
  id: "UUID v7"
  name: "Project name"
  description: "Optional description"
  path: "Filesystem path (local)"
  status: "active | archived"
  metadata: "Flexible: language, framework, tags"
  created_at: "ISO 8601"
  updated_at: "ISO 8601"

capabilities:
  - "CRUD operations"
  - "Task management with Kanban"
  - "Knowledge base"
  - "Conversation history"
  - "File management via .vestara/"
  - "Git integration"
  - "Clone and archive"

state-machine:
  - Created: "Initial creation"
  - Active: "In use, accepting changes"
  - Archived: "Preserved in .vestara/, read-only"
  - Deleted: "Removed"

relationships:
  - "belongs_to: Workspace"
  - "has_many: Task"
  - "has_many: Conversation"
  - "has_many: Knowledge"
  - "has_many: Document"
  - "has_many: Artifact"

events:
  - "project:created"
  - "project:updated"
  - "project:archived"
  - "project:deleted"
  - "project:cloned"
  - "project:synced"

security:
  - "Project isolation: data scoped to project"
  - "Collaboration: share via organization (Gen 3+)"
```

---

### 5. ✅ Task

```yaml
id: "VOM-TASK"
name: "Task"
purpose: "Unit of execution within a project. Trackable, assignable, timeable."
lifecycle: "Created → Active → Completed → Archived"

properties:
  id: "UUID v7"
  title: "Task title"
  description: "Optional detailed description"
  status: "todo | in_progress | review | done"
  assignee: "User reference"
  parent_id: "Self-referencing for sub-tasks"
  tags: "Categorization"
  estimated_hours: "Effort estimate"
  logged_hours: "Actual time"
  sort_order: "Kanban position"
  created_at: "ISO 8601"
  updated_at: "ISO 8601"

state-machine:
  - Created: "Initial state"
  - Todo: "Ready to work"
  - In_Progress: "Actively being worked"
  - Review: "Waiting for review/approval"
  - Done: "Completed"
  - Archived: "Preserved"

relationships:
  - "belongs_to: Project"
  - "has_many: Task (sub-tasks via parent_id)"
  - "has_one: Task (parent via parent_id)"
  - "has_many: Activity (timeline entries)"

status-transitions:
  - "todo → in_progress"
  - "in_progress → review"
  - "in_progress → todo (un-start)"
  - "review → done"
  - "review → in_progress (re-open)"
  - "done → in_progress (re-open)"

events:
  - "task:created"
  - "task:updated"
  - "task:status.changed"
  - "task:assigned"
  - "task:deleted"
```

---

### 6. 💬 Conversation

```yaml
id: "VOM-CONVERSATION"
name: "Conversation"
purpose: "Multi-turn AI dialogue. The primary human-AI interaction channel."
lifecycle: "Created → Active → Archived → Deleted"

properties:
  id: "UUID v7"
  title: "Auto-generated or user-set"
  model: "AI model providing responses"
  provider: "AI provider routing the requests"
  messages: "Ordered array of Message objects"
  metadata: "Tokens, cost, latency"
  status: "active | archived | deleted"
  created_at: "ISO 8601"
  updated_at: "ISO 8601"

capabilities:
  - "Multi-turn dialogue"
  - "Streaming responses"
  - "Tool execution within conversation"
  - "Context assembly (memory + knowledge + history)"
  - "Message editing and branching (Gen 2)"

state-machine:
  - Created: "First message sent"
  - Active: "Accepting new messages"
  - Archived: "Read-only history"
  - Deleted: "Removed"

relationships:
  - "belongs_to: Project (optional, can be workspace-level)"
  - "belongs_to: User"
  - "has_many: Message"
  - "has_many: Memory (derived from conversation)"

events:
  - "conversation:started"
  - "conversation:message.sent"
  - "conversation:response.complete"
  - "conversation:archived"
  - "conversation:deleted"
```

---

### 7. 📝 Message

```yaml
id: "VOM-MESSAGE"
name: "Message"
purpose: "Single exchange unit in a conversation."
lifecycle: "Created (append-only)"

properties:
  id: "UUID v7"
  role: "user | assistant | system | tool"
  content: "Markdown-formatted text"
  tool_calls: "Array of tool invocations"
  tool_results: "Array of tool execution results"
  attachments: "Optional file attachments"
  metadata: "Tokens, latency, cost"
  created_at: "ISO 8601"

relationships:
  - "belongs_to: Conversation"
  - "has_many: ToolCall (if assistant role)"

security:
  - "Content sanitized (XSS prevention)"
  - "PII detection on output"
  - "Input validation via Zod"

events:
  - "conversation:message.sent (user role)"
  - "conversation:response.start (assistant role)"
  - "conversation:response.complete (assistant role)"
```

---

### 8. 🧠 Memory

```yaml
id: "VOM-MEMORY"
name: "Memory"
purpose: "Cross-session persistent knowledge about the user, their preferences, and context."
lifecycle: "Created → Consolidated → Archived → Deleted"

types:
  - Fact: "Declarative knowledge ('User prefers dark mode')"
  - Preference: "User settings and tuning"
  - Event: "Episodic memory of past interactions"
  - Decision: "Rationale for past choices"

properties:
  id: "UUID v7"
  type: "fact | preference | event | decision"
  content: "Memory content"
  summary: "Consolidated summary (for low-importance memories)"
  importance: "0.0 — 10.0 (scored by frequency, recency, user feedback)"
  embedding: "Vector representation (Gen 2)"
  metadata: "Context, source, tags"
  consolidated_at: "Last consolidation"
  created_at: "ISO 8601"

importance_scoring:
  recency_weight: 0.3
  frequency_weight: 0.2
  user_feedback_weight: 0.2
  novelty_weight: 0.15
  impact_weight: 0.15

consolidation:
  high_importance: "8-10: Full resolution, permanent"
  medium_importance: "4-7: Summary with key details"
  low_importance: "1-3: Keyword-only reference"
  pruning: "Score < 1, age > 90 days"

relationships:
  - "belongs_to: User"
  - "belongs_to: Project (optional)"
  - "belongs_to: Conversation (source)"

events:
  - "memory:stored"
  - "memory:consolidated"
  - "memory:searched"
  - "memory:deleted"
```

---

### 9. 📚 Knowledge

```yaml
id: "VOM-KNOWLEDGE"
name: "Knowledge"
purpose: "Structured information stored for retrieval-augmented generation (RAG)."
lifecycle: "Created → Indexed → Updated → Deleted"

types:
  - document: "Full document (imported or created)"
  - note: "Quick note or annotation"
  - reference: "External reference link"
  - code: "Code snippet or pattern"
  - tutorial: "How-to guide"

properties:
  id: "UUID v7"
  title: "Optional title"
  content: "Document content"
  type: "document | note | reference | code | tutorial"
  tags: "Categorization tags"
  embedding: "Vector for semantic search (Gen 2)"
  metadata: "Source, author, version"
  created_at: "ISO 8601"
  updated_at: "ISO 8601"

capabilities:
  - "Full-text search (SQLite FTS5)"
  - "Vector search (Gen 2)"
  - "RAG pipeline: query → retrieve → rerank → generate"
  - "Auto-indexing via filesystem watcher"
  - "Chunking for large documents"

relationships:
  - "belongs_to: Project"
  - "belongs_to: User (personal knowledge)"

events:
  - "knowledge:added"
  - "knowledge:searched"
  - "knowledge:updated"
  - "knowledge:deleted"
```

---

### 10. 🤖 Agent

```yaml
id: "VOM-AGENT"
name: "Agent"
purpose: "Autonomous actor with tools, memory, and a defined mission."
lifecycle: "Created → Idle → Executing → Completed → Archived"

properties:
  id: "UUID v7"
  name: "Agent name"
  description: "Purpose and behavior description"
  model: "AI model powering the agent"
  config: "Agent-specific configuration"
  tools: "Registered tool IDs"
  memory_enabled: "Can access user memory"
  status: "idle | executing | paused | completed | failed | archived"
  created_at: "ISO 8601"

capabilities:
  - "Execute tasks autonomously"
  - "Use registered tools"
  - "Access memory and knowledge"
  - "Stream execution progress"
  - "Handle interruptions and resume"
  - "Multi-agent coordination (Gen 3+)"

state-machine:
  - Created: "Defined but not yet run"
  - Idle: "Ready for execution"
  - Executing: "Active, processing"
  - Paused: "Interrupted, can resume"
  - Completed: "Task finished successfully"
  - Failed: "Task finished with error"
  - Archived: "Preserved for history"

relationships:
  - "belongs_to: User"
  - "belongs_to: Project (optional)"
  - "has_many: Tool (registered tools)"
  - "has_many: Mission (assigned missions)"

events:
  - "agent:created"
  - "agent:execution.started"
  - "agent:execution.progress"
  - "agent:execution.completed"
  - "agent:execution.failed"
  - "agent:updated"
  - "agent:deleted"

interface:
  initialize(config: AgentConfig): Promise<void>
  execute(task: Task, context: ExecutionContext): Promise<ExecutionResult>
  cancel(): Promise<void>
  pause(): Promise<void>
  resume(): Promise<void>
  getStatus(): AgentStatus
```

---

### 11. 🔧 Tool

```yaml
id: "VOM-TOOL"
name: "Tool"
purpose: "Capability boundary that agents invoke to interact with the world."
lifecycle: "Registered → Available → Deprecated → Removed"

properties:
  id: "UUID v7 — or well-known string ID"
  name: "Tool name"
  description: "What the tool does"
  parameters: "JSON Schema for invocation parameters"
  returns: "Return type schema"
  permission_level: "read-only | user-confirm | admin-only"
  requires: "Optional: network, filesystem, etc."
  timeout: "Maximum execution time"
  sandbox: "Whether execution is sandboxed"

built_in_tools:
  read_file: { permission: "read-only", sandbox: false }
  write_file: { permission: "user-confirm", sandbox: true }
  execute_command: { permission: "user-confirm", sandbox: true, timeout: 30000 }
  search_knowledge: { permission: "read-only" }
  search_memory: { permission: "read-only" }
  web_search: { permission: "user-confirm", requires: "network" }
  create_task: { permission: "user-confirm" }

relationships:
  - "belongs_to: Agent (via registration)"
  - "can be: Built-in or Plugin-provided"

interface:
  execute(params: Record<string, unknown>): Promise<ToolResult>
  validate(params: Record<string, unknown>): ValidationResult
  getSchema(): ToolSchema
```

---

### 12. 🔌 Provider

```yaml
id: "VOM-PROVIDER"
name: "Provider"
purpose: "External AI service abstraction. Any provider plugs in identically."
lifecycle: "Registered → Available → Degraded → Unavailable"

properties:
  id: "opencode | ollama | openai | anthropic | google"
  name: "Human-readable provider name"
  models: "Array of available Model objects"
  features: "Capability bitmask (chat, streaming, embeddings, vision, speech, etc.)"
  status: "available | degraded | unavailable | loading"
  config: "Provider-specific configuration"

capabilities:
  - "List available models"
  - "Complete chat requests (non-streaming)"
  - "Stream chat responses"
  - "Count tokens"
  - "Estimate costs"
  - "Health check"
  - "Embeddings (Gen 2)"
  - "Vision (Gen 3)"
  - "Speech (Gen 3)"

relationships:
  - "has_many: Model"
  - "used_by: Agent, Conversation"

interface:
  initialize(config: ProviderConfig): Promise<void>
  complete(request: CompletionRequest): Promise<CompletionResponse>
  stream(request: CompletionRequest): AsyncIterable<StreamChunk>
  countTokens(text: string): number
  estimateCost(input: string, output: string): CostEstimate
  healthCheck(): Promise<HealthStatus>
  listModels(): Promise<Model[]>
```

---

### 13. 🧩 Plugin

```yaml
id: "VOM-PLUGIN"
name: "Plugin"
purpose: "Extension point. Third-party code that extends Vestara."
lifecycle: "Installed → Enabled → Disabled → Uninstalled"

properties:
  id: "UUID v7 — or package name"
  name: "Plugin name"
  version: "Semantic version"
  manifest: "Plugin manifest (capabilities, permissions, UI)"
  capabilities: "Array of provided capabilities"
  permissions: "Granted permissions"
  status: "installed | enabled | disabled | error"
  installed_at: "ISO 8601"

capabilities:
  - "Register tools"
  - "Register commands"
  - "Add UI panels"
  - "Subscribe to events"
  - "Add settings pages"
  - "Add themes"
  - "Add providers"

relationships:
  - "belongs_to: User (or Organization for org-level plugins)"
  - "may provide: Tool, Provider, Command, UI Panel"

interface:
  activate(context: PluginContext): Promise<void>
  deactivate(): Promise<void>
  handleEvent(event: VestaraEvent): Promise<void>
```

---

### 14. 🎯 Mission

```yaml
id: "VOM-MISSION"
name: "Mission"
purpose: "Long-lived business outcome that survives sessions and coordinates multiple agents."
lifecycle: "Created → Planning → Executing → Reviewing → Completed → Archived"

properties:
  id: "UUID v7"
  name: "Mission name"
  description: "Business outcome description"
  objective: "Primary success criterion"
  status: "planning | executing | reviewing | completed | failed | archived"
  progress: "0.0 — 1.0 completion percentage"
  agents: "Assigned agents"
  tasks: "Decomposed work items"
  artifacts: "Produced outputs"
  created_at: "ISO 8601"
  completed_at: "ISO 8601"

capabilities:
  - "Decompose high-level goal into tasks"
  - "Assign tasks to agents"
  - "Track progress across sessions"
  - "Replan on failure"
  - "Produce artifacts"
  - "Report status"

state-machine:
  - Created: "Mission defined"
  - Planning: "AI Planning Engine decomposes goal"
  - Executing: "Agents executing tasks"
  - Reviewing: "Results being validated"
  - Completed: "Objective achieved"
  - Failed: "Objective not achievable"
  - Archived: "Preserved for learning"

relationships:
  - "belongs_to: User (or Organization)"
  - "has_many: Agent (assigned to mission)"
  - "has_many: Task (decomposed work)"
  - "has_many: Artifact (produced outputs)"

events:
  - "mission:created"
  - "mission:planning.completed"
  - "mission:execution.progress"
  - "mission:completed"
  - "mission:failed"
  - "mission:archived"
```

---

### 15. ⚡ Event

```yaml
id: "VOM-EVENT"
name: "Event"
purpose: "Immutable notification of something that happened in the platform."
lifecycle: "Emitted (append-only, immutable)"

properties:
  id: "UUID v7"
  type: "domain:action (project:created, memory:stored, etc.)"
  version: "Schema version"
  timestamp: "ISO 8601"
  source: "Emitting service/module"
  actor: "Who/what triggered the event"
  payload: "Domain-specific data"
  metadata: "Correlation ID, causation ID, TTL"

security:
  - "Events are immutable once emitted"
  - "Event payloads scoped to need-to-know"
  - "Audit events never leave the platform"
```

---

### 16. 🔄 Workflow

```yaml
id: "VOM-WORKFLOW"
name: "Workflow"
purpose: "Automated multi-step process. Trigger → Condition → Plan → Execute → Validate."
lifecycle: "Defined → Enabled → Running → Completed → Disabled"

properties:
  id: "UUID v7"
  name: "Workflow name"
  trigger: "Event or schedule or manual"
  steps: "Ordered array of workflow steps"
  status: "enabled | disabled | running | completed | failed"
  created_at: "ISO 8601"

relationships:
  - "belongs_to: User (or Organization)"
  - "has_many: Step"
  - "has_many: Execution (run history)"

events:
  - "workflow:triggered"
  - "workflow:step.completed"
  - "workflow:completed"
  - "workflow:failed"
```

---

### 17. 🌐 Context

```yaml
id: "VOM-CONTEXT"
name: "Context"
purpose: "Current session state — the assembled environment for AI interactions."
lifecycle: "Session-bound"

properties:
  workspace_id: "Current workspace"
  project_id: "Current project (if any)"
  user_id: "Current user"
  conversation_id: "Active conversation"
  memories: "Loaded memories for context"
  knowledge: "Retrieved knowledge for context"
  session_start: "ISO 8601"
  last_activity: "ISO 8601"

capabilities:
  - "Assemble context from memory + knowledge + history"
  - "Optimize context window (token budget)"
  - "Persist session state on interruption"
  - "Restore session state on resume"

relationships:
  - "belongs_to: Workspace (one active context per workspace)"
```

---

### 18. 📄 Document

```yaml
id: "VOM-DOCUMENT"
name: "Document"
purpose: "Persistent content file within a project."
lifecycle: "Created → Modified → Archived → Deleted"

properties:
  id: "UUID v7"
  path: "Project-relative path"
  content: "File content"
  mime_type: "MIME type"
  size: "File size in bytes"
  hash: "Content hash (SHA-256)"
  version: "Document version"
  created_at: "ISO 8601"
  updated_at: "ISO 8601"

relationships:
  - "belongs_to: Project"
  - "may be: Knowledge source"
```

---

### 19. 🎨 Artifact

```yaml
id: "VOM-ARTIFACT"
name: "Artifact"
purpose: "Output produced by an agent or workflow."
lifecycle: "Created → Reviewed → Approved → Rejected → Archived"

properties:
  id: "UUID v7"
  type: "code | document | image | design | configuration | report"
  content: "Artifact content or reference"
  produced_by: "Agent or tool that created it"
  mission_id: "Optional mission association"
  status: "draft | reviewed | approved | rejected"
  created_at: "ISO 8601"

relationships:
  - "belongs_to: Mission (optional)"
  - "belongs_to: Agent (creator)"
  - "belongs_to: Project"
```

---

## 📊 Object Model Summary

| # | Object | Primary Key | Lifecycle | Core Events | Gen |
|---|--------|-------------|-----------|-------------|-----|
| 1 | Organization | `id: UUID` | 4 states | 6 events | 3 |
| 2 | User | `id: UUID` | 4 states | 4 events | 1 |
| 3 | Workspace | `id: UUID` | 4 states | 5 events | 1 |
| 4 | Project | `id: UUID` | 4 states | 5 events | 1 |
| 5 | Task | `id: UUID` | 5 states | 5 events | 1 |
| 6 | Conversation | `id: UUID` | 4 states | 4 events | 1 |
| 7 | Message | `id: UUID` | Append-only | 3 events | 1 |
| 8 | Memory | `id: UUID` | 4 states | 4 events | 1 |
| 9 | Knowledge | `id: UUID` | 4 states | 4 events | 1 |
| 10 | Agent | `id: UUID` | 6 states | 6 events | 1 |
| 11 | Tool | `string ID` | 4 states | — | 1 |
| 12 | Provider | `string ID` | 4 states | 2 events | 1 |
| 13 | Plugin | `string ID` | 5 states | 4 events | 2 |
| 14 | Mission | `id: UUID` | 6 states | 5 events | 2 |
| 15 | Event | `id: UUID` | Append-only | — | 1 |
| 16 | Workflow | `id: UUID` | 5 states | 4 events | 2 |
| 17 | Context | session | Session-bound | — | 1 |
| 18 | Document | `id: UUID` | 4 states | 3 events | 1 |
| 19 | Artifact | `id: UUID` | 5 states | 3 events | 2 |

---

**Total Objects Defined: 19 core primitives** — covering the complete vocabulary of the Vestara platform.

*No code should reference a concept not defined here. Every new concept must be added to this model before implementation.*
