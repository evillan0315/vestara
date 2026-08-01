---
id: "SPEC-EVT-000"
title: "Event Catalog — Complete Event Definitions"
owner: "@backend-engineer"
status: "draft"
blueprint-ref: "04-platform/01-platform-overview.md"
version: "1.0.0"
---

# Event Catalog
## Complete Event Definitions for the Vestara Platform

> **Vestara is an event-driven platform. Every meaningful action in the system emits an event. This catalog defines every event, its payload, publisher, subscribers, retry policy, and versioning.**

---

## Event Format

Every event follows this structure:

```typescript
interface VestaraEvent {
  id: string;                         // UUID v7 — globally unique
  type: string;                       // `${domain}:${action}` — e.g. "project:created"
  version: number;                    // Semantic version of event schema
  timestamp: string;                  // ISO 8601
  source: string;                     // Service or module that emitted the event
  actor?: {                           // Who triggered the event
    id: string;
    role: 'user' | 'system' | 'agent';
  };
  payload: Record<string, unknown>;   // Domain-specific payload
  metadata: {
    correlationId: string;            // Trace across event chains
    causationId?: string;             // Parent event that caused this one
    retryCount: number;               // Current retry attempt
    ttl: number;                      // Time-to-live in seconds
  };
}
```

---

## Event Catalog Index

### 👤 User Events

```
USER_CREATED          user:created       
USER_LOGIN            user:login         
USER_LOGOUT           user:logout        
USER_DELETED          user:deleted       
USER_SETTINGS_CHANGED user:settings.changed
USER_PREFERENCES_UPDATED user:preferences.updated
```

### 🏗️ Project Events

```
PROJECT_CREATED       project:created    
PROJECT_UPDATED       project:updated    
PROJECT_ARCHIVED      project:archived   
PROJECT_DELETED       project:deleted    
PROJECT_CLONED        project:cloned     
PROJECT_SYNCED        project:synced     
PROJECT_SHARED        project:shared     

TASK_CREATED          task:created       
TASK_UPDATED          task:updated       
TASK_DELETED          task:deleted       
TASK_STATUS_CHANGED   task:status.changed
TASK_ASSIGNED         task:assigned      
TASK_BULK_UPDATED     task:bulk.updated  
```

### 🧠 AI Events

```
CONVERSATION_STARTED         conversation:started    
CONVERSATION_MESSAGE_SENT    conversation:message.sent
CONVERSATION_RESPONSE_START  conversation:response.start
CONVERSATION_RESPONSE_COMPLETE conversation:response.complete
CONVERSATION_TOOL_EXECUTING  conversation:tool.executing
CONVERSATION_TOOL_COMPLETED  conversation:tool.completed
CONVERSATION_ERROR           conversation:error      

MEMORY_STORED         memory:stored       
MEMORY_SEARCHED       memory:searched     
MEMORY_CONSOLIDATED   memory:consolidated 
MEMORY_DELETED        memory:deleted      
MEMORY_IMPORTANCE_UPDATED memory:importance.updated

KNOWLEDGE_ADDED       knowledge:added     
KNOWLEDGE_SEARCHED    knowledge:searched   
KNOWLEDGE_UPDATED     knowledge:updated   
KNOWLEDGE_DELETED     knowledge:deleted   

AGENT_CREATED         agent:created       
AGENT_EXECUTION_STARTED  agent:execution.started
AGENT_EXECUTION_PROGRESS agent:execution.progress
AGENT_EXECUTION_COMPLETED agent:execution.completed
AGENT_EXECUTION_FAILED agent:execution.failed
AGENT_UPDATED         agent:updated       
AGENT_DELETED         agent:deleted       

PROVIDER_CHANGED      provider:changed    
PROVIDER_STATUS_CHANGED provider:status.changed
MODEL_LOADED          model:loaded        
MODEL_UNLOADED        model:unloaded      

AI_EVALUATION_COMPLETED ai:evaluation.completed
AI_SAFETY_TRIGGERED  ai:safety.triggered  
AI_LEARNING_UPDATED   ai:learning.updated 
```

### 📦 Platform Events

```
NOTIFICATION_CREATED  notification:created 
NOTIFICATION_READ     notification:read   
NOTIFICATION_ARCHIVED notification:archived

SETTINGS_CHANGED      settings:changed    

WORKSPACE_OPENED      workspace:opened    
WORKSPACE_CLOSED      workspace:closed    
WORKSPACE_LAYOUT_CHANGED workspace:layout.changed
WORKSPACE_THEME_CHANGED  workspace:theme.changed

PLUGIN_INSTALLED      plugin:installed    
PLUGIN_UNINSTALLED    plugin:uninstalled  
PLUGIN_ACTIVATED      plugin:activated    
PLUGIN_DEACTIVATED    plugin:deactivated  
PLUGIN_ERROR          plugin:error        

SYNC_STARTED          sync:started        
SYNC_PROGRESS         sync:progress       
SYNC_COMPLETED        sync:completed      
SYNC_FAILED           sync:failed         
SYNC_CONFLICT         sync:conflict       

VOICE_ACTIVATED       voice:activated      
VOICE_DEACTIVATED     voice:deactivated    
VOICE_COMMAND_RECEIVED voice:command.received
VOICE_RESPONSE_STARTED voice:response.started
VOICE_RESPONSE_COMPLETED voice:response.completed

VISION_IMAGE_ANALYZED vision:image.analyzed
VISION_IMAGE_GENERATED vision:image.generated

AUTOMATION_TRIGGERED   automation:triggered
AUTOMATION_COMPLETED   automation:completed
AUTOMATION_FAILED      automation:failed   
```

### 🖥️ OS-0 Host and Boot Events

The OS-0 runtimes emit the implementation's dot-delimited runtime event names.
They are internal lifecycle evidence and carry runtime envelope metadata.

```text
host.snapshot.captured  Read-only host snapshot initialized
host.power.requested    Authorized power operation reached execution boundary
boot.stage.changed      Ordered boot stage persisted
boot.completed          workspace-ready persisted
boot.recovery.entered   Recovery state persisted with reason
boot.failed             Terminal boot failure persisted with reason
```

Minimum Boot Runtime payload:

```typescript
interface BootRuntimeEventPayload {
  readonly bootId: string;
  readonly stage: string;
  readonly status: 'booting' | 'ready' | 'recovery' | 'failed';
  readonly runtimeId: string;
  readonly severity: 'info' | 'warning' | 'error';
}
```

`host.power.requested` is not reachable from OS-0 API or CLI surfaces. The Host
Runtime emits it only after explicit enablement, per-request authorization, and
policy permission succeed.

Implementation reference: `evillan0315/vestara-ai-core@579df3f`.

### 🔒 Security Events

```
AUTH_LOGIN_SUCCESS     auth:login.success  
AUTH_LOGIN_FAILED      auth:login.failed   
AUTH_TOKEN_REFRESHED   auth:token.refreshed
AUTH_TOKEN_REVOKED     auth:token.revoked  
AUTH_SESSION_EXPIRED   auth:session.expired
AUTH_RATE_LIMIT_EXCEEDED auth:rate-limit.exceeded

SECURITY_THREAT_DETECTED security:threat.detected
SECURITY_PII_DETECTED    security:pii.detected
SECURITY_AUDIT_EVENT     security:audit.event
```

### ☁️ Cloud Events (Gen 3+)

```
CLOUD_SYNC_STARTED     cloud:sync.started  
CLOUD_SYNC_PROGRESS    cloud:sync.progress 
CLOUD_SYNC_COMPLETED   cloud:sync.completed
CLOUD_SYNC_FAILED      cloud:sync.failed   

CLOUD_WORKER_STARTED   cloud:worker.started
CLOUD_WORKER_COMPLETED cloud:worker.completed
CLOUD_WORKER_FAILED    cloud:worker.failed 

CLOUD_INFERENCE_REQUESTED cloud:inference.requested
CLOUD_INFERENCE_COMPLETED cloud:inference.completed

REMOTE_AGENT_CONNECTED  remote:agent.connected
REMOTE_AGENT_DISCONNECTED remote:agent.disconnected
```

---

## Detailed Event Specifications

### Event: `conversation:message.sent`

```yaml
type: "conversation:message.sent"
version: 1
description: "User sent a message in a conversation"

payload:
  conversationId: string     # UUID v7
  messageId: string          # UUID v7
  userId: string             # UUID v7
  content: string            # Message text
  attachments: array         # Optional file attachments

publisher: "workspace-chat"
subscribers:
  - "memory-service"        # Store in conversation memory
  - "analytics-service"     # Usage tracking
  - "notification-service"  # If @mention detected

retry:
  policy: "exponential-backoff"
  maxAttempts: 3
  initialDelayMs: 1000

security: "authenticated-user"
versioning: "additive-only"
```

### Event: `memory:consolidated`

```yaml
type: "memory:consolidated"
version: 1
description: "Memory consolidation cycle completed"

payload:
  userId: string              # UUID v7
  cycleId: string             # UUID v7 — consolidation run identifier
  memoriesProcessed: number   # Total memories in cycle
  memoriesArchived: number    # Memories moved to archive
  memoriesPruned: number      # Memories removed
  durationMs: number          # Consolidation duration
  importanceDistribution: {   # Histogram of importance scores
    high: number,             # Score 8-10
    medium: number,           # Score 4-7
    low: number               # Score 1-3
  }

publisher: "memory-service"
subscribers:
  - "analytics-service"       # Performance tracking
  - "notification-service"    # If high-importance changes detected

retry:
  policy: "at-least-once"
  maxAttempts: 5

security: "system-internal"
versioning: "additive-only"
```

### Event: `project:status.changed`

```yaml
type: "task:status.changed"
version: 1
description: "A task's status was changed (todo → in_progress → review → done)"

payload:
  projectId: string           # UUID v7
  taskId: string              # UUID v7
  userId: string              # UUID v7 — who changed it
  previousStatus: string      # 'todo' | 'in_progress' | 'review' | 'done'
  newStatus: string           # 'todo' | 'in_progress' | 'review' | 'done'
  timestamp: string           # ISO 8601

publisher: "project-service"
subscribers:
  - "analytics-service"       # Cycle time tracking
  - "notification-service"    # Notify assignee if changed
  - "memory-service"          # Update project memory
  - "workspace-client"        # Real-time UI update

retry:
  policy: "at-least-once"
  maxAttempts: 3

security: "authenticated-user"
versioning: "additive-only"
```

### Event: `ai:safety.triggered`

```yaml
type: "ai:safety.triggered"
version: 1
description: "AI safety layer detected and blocked a harmful input or output"

payload:
  checkType: string           # 'prompt-injection' | 'pii' | 'toxicity' | 'jailbreak'
  severity: string            # 'low' | 'medium' | 'high' | 'critical'
  action: string              # 'block' | 'warn' | 'sanitize' | 'log'
  triggerContent: string      # Truncated trigger content (first 100 chars)
  userId: string              # UUID v7
  conversationId: string      # UUID v7 — if in conversation context
  providerId: string          # Provider being used when triggered

publisher: "ai-safety-layer"
subscribers:
  - "security-service"        # Threat tracking
  - "analytics-service"       # Safety metrics
  - "audit-log"               # Immutable audit record

retry:
  policy: "at-least-once"
  maxAttempts: 5

security: "system-internal"  # Never expose to client
versioning: "additive-only"
```

---

## Event Versioning

| Version Change | When | Migration |
|----------------|------|-----------|
| Major | Breaking schema change (removed field, changed type) | All subscribers must update |
| Minor | Added optional field | Required subscribers should handle gracefully |
| Patch | Documentation, non-functional changes | No action needed |

## Event Security Levels

| Level | Description | Examples |
|-------|-------------|----------|
| `authenticated-user` | Event contains user data, requires authentication | `conversation:message.sent` |
| `system-internal` | Event never leaves service boundaries | `ai:safety.triggered` |
| `public` | Safe to broadcast, no sensitive data | `project:created` |
| `audit-only` | Immutable record, no consumers | `security:audit.event` |

---

## Event Naming Conventions

| Pattern | Example | Description |
|---------|---------|-------------|
| `domain:created` | `project:created` | Entity was created |
| `domain:updated` | `task:updated` | Entity was updated |
| `domain:deleted` | `memory:deleted` | Entity was deleted |
| `domain:action` | `conversation:tool.executing` | Action occurred |
| `domain:completed` | `sync:completed` | Process completed |
| `domain:failed` | `agent:execution.failed` | Process failed |
| `domain:changed` | `provider:changed` | State changed |
| `domain:triggered` | `ai:safety.triggered` | Safety event |

---

**Total Events Defined: 78** across 8 domains.

*This catalog grows as the platform evolves. Every new event must be registered here before it can be emitted.*
