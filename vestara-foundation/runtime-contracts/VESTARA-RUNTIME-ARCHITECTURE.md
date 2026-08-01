---
id: "FND-002"
title: "Vestara Runtime Architecture (VRA) — What Runs Inside Vestara"
owner: "@chief-architect"
status: "ratified"
blueprint-ref: "04-platform/01-platform-overview.md"
foundation-version: "1.1.0"
---

# Vestara Runtime Architecture (VRA)
## The Internal Operating Model of the Vestara Platform

> **This document answers one question: "What actually runs inside Vestara?" Every runtime component is a managed, observable, replaceable unit with a standard lifecycle.**

---

## 🏗️ Runtime Stack

```
┌───────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                       │
│  Workspace UI  │  CLI  │  Mobile  │  API Gateway         │
├───────────────────────────────────────────────────────────┤
│                     SERVICE LAYER                          │
│  Identity  │  Workspace  │  Project  │  Knowledge         │
│  Conversation  │  Memory  │  Notification  │  Settings    │
├───────────────────────────────────────────────────────────┤
│                      AI LAYER                              │
│  Agent Runtime  │  Provider Manager  │  Prompt Engine     │
│  Planning  │  Reasoning  │  Evaluation  │  Safety         │
├───────────────────────────────────────────────────────────┤
│                    TOOL LAYER                              │
│  Filesystem  │  Git  │  Shell  │  Search  │  Browser     │
│  Knowledge  │  Memory  │  Code  │  Terminal              │
├───────────────────────────────────────────────────────────┤
│                   FOUNDATION LAYER                         │
│  Kernel  │  Host Runtime │  Boot Runtime │  Event Bus      │
│  Logging  │  Observability  │  Plugin Loader             │
├───────────────────────────────────────────────────────────┤
│                    OPERATING SYSTEM                        │
│  Linux Host (OS-0) │ Future: Immutable A/B + Secure Boot  │
└───────────────────────────────────────────────────────────┘
```

---

## 🔷 Runtime Components

### OS-0 machine boundary

OS-0 adds two implemented first-class runtimes beneath the workspace control
plane:

```text
Linux/systemd -> Host Runtime -> Boot Runtime -> Kernel services -> Workspace
```

`HostRuntime` owns typed, read-only machine observation. `BootRuntime` owns
durable ordered progress from `firmware-complete` through `workspace-ready`.
Both join the kernel dependency graph through lifecycle adapters. Power
operations are disabled by default and are not exposed through OS-0 API or CLI
surfaces.

Storage, devices, networking, identity, updates, and recovery remain services
or adapters until they require independent durable lifecycle and recovery. A
bootable ISO, installer, immutable A/B image, and Secure Boot are future layers,
not OS-0 implementation claims.

### 1. Kernel

```yaml
component: Kernel
purpose: "Core runtime orchestrator. Boots the platform, manages component lifecycle."
responsibilities:
  - "Initialize runtime environment"
  - "Load configuration"
  - "Start Service Bus"
  - "Orchestrate startup sequence"
  - "Monitor health"
  - "Graceful shutdown"

lifecycle:
  - Boot: "Initialize kernel, load config"
  - Starting: "Start core services in dependency order"
  - Running: "Platform operational"
  - Draining: "Graceful shutdown, stop accepting work"
  - Stopped: "All services stopped"

interface:
  boot(): Promise<void>
  shutdown(): Promise<void>
  getStatus(): RuntimeStatus
  getHealth(): HealthStatus
```

---

### 2. Service Bus

```yaml
component: ServiceBus
purpose: "Inter-service communication, discovery, and lifecycle management."
responsibilities:
  - "Service registry (who provides what)"
  - "Service discovery (find by capability)"
  - "Request routing (sync calls)"
  - "Event distribution (async pub/sub)"
  - "Health monitoring"
  - "Circuit breaking"

pattern: "In-process for Gen 1, distributed (NATS/gRPC) for Gen 3+"

interface:
  registerService(service: VestaraService): Promise<void>
  unregisterService(serviceId: string): Promise<void>
  getService(capability: string): VestaraService | null
  listServices(): ServiceInfo[]
  call<R>(serviceId: string, method: string, payload: unknown): Promise<R>
  emit(event: VestaraEvent): Promise<void>
  subscribe(type: string, handler: EventHandler): Unsubscribe
```

---

### 3. Agent Runtime

```yaml
component: AgentRuntime
purpose: "Creates and executes AI agents. Manages agent lifecycle, tool registration, sandboxing."
responsibilities:
  - "Agent creation and configuration"
  - "Execution scheduling and sandboxing"
  - "Tool registry and invocation"
  - "Streaming execution results"
  - "Agent persistence and restore"
  - "Resource limits and cancellation"
  - "Multi-agent coordination (Gen 3)"

interface:
  createAgent(config: AgentConfig): Promise<Agent>
  executeAgent(agentId: string, task: Task, context?: ExecutionContext): Promise<ExecutionResult>
  streamExecution(agentId: string): AsyncIterable<ExecutionChunk>
  cancelExecution(agentId: string): Promise<void>
  pauseAgent(agentId: string): Promise<void>
  resumeAgent(agentId: string): Promise<void>
  registerTool(tool: Tool): Promise<void>
  getAgentStatus(agentId: string): Promise<AgentStatus>
```

---

### 4. Tool Runtime

```yaml
component: ToolRuntime
purpose: "Sandboxed tool execution with resource limits, permission checks, and timeout."
responsibilities:
  - "Tool invocation with parameter validation"
  - "Sandboxed execution (isolated process/container)"
  - "Permission verification (user approval flow)"
  - "Timeout enforcement"
  - "Result collection"
  - "Error handling with retry"
  - "Audit logging"

interface:
  execute(toolId: string, params: Record<string, unknown>, context: ToolContext): Promise<ToolResult>
  validate(toolId: string, params: Record<string, unknown>): ValidationResult
  getTool(toolId: string): Tool
  listTools(): ToolInfo[]
  getUserApproval(toolId: string, params: Record<string, unknown>): Promise<boolean>
```

---

### 5. Memory Runtime

```yaml
component: MemoryRuntime
purpose: "Manages memory layers, consolidation, importance scoring, and search."
responsibilities:
  - "Short-term memory (session-scoped)"
  - "Long-term memory (SQLite persistent)"
  - "Importance scoring algorithm"
  - "Consolidation scheduling"
  - "Hybrid search (FTS + vector, Gen 2)"
  - "Memory pruning and archiving"

interface:
  store(memory: MemoryInput): Promise<Memory>
  search(query: string, options?: SearchOptions): Promise<MemorySearchResult>
  getContext(userId: string, limit?: number): Promise<Memory[]>
  consolidate(userId: string): Promise<ConsolidationResult>
  setImportance(memoryId: string, score: number): Promise<void>
```

---

### 6. Provider Runtime

```yaml
component: ProviderRuntime
purpose: "AI provider lifecycle management, routing, fallback, health monitoring."
responsibilities:
  - "Provider registration and configuration"
  - "Provider health monitoring"
  - "Request routing (model → provider)"
  - "Fallback chains (primary fails → try next)"
  - "Cost tracking and budgeting"
  - "Rate limiting"
  - "Provider capability discovery"

interface:
  registerProvider(provider: AIProvider): Promise<void>
  complete(request: CompletionRequest): Promise<CompletionResponse>
  stream(request: CompletionRequest): AsyncIterable<StreamChunk>
  getModels(filters?: ModelFilter): Promise<Model[]>
  healthCheck(): Promise<ProviderHealthReport>
  estimateCost(request: CompletionRequest): Promise<CostEstimate>
```

---

### 7. Workspace Runtime

```yaml
component: WorkspaceRuntime
purpose: "Manages workspace state, project lifecycle, and UI state."
responsibilities:
  - "Workspace CRUD"
  - "Project CRUD and lifecycle"
  - "Task management"
  - "Context assembly and persistence"
  - "State synchronization (local + cloud)"

interface:
  openWorkspace(workspaceId: string): Promise<Workspace>
  closeWorkspace(): Promise<void>
  getActiveProject(): Promise<Project | null>
  setActiveProject(projectId: string): Promise<void>
  listProjects(filter?: ProjectFilter): Promise<Project[]>
  createProject(input: CreateProjectInput): Promise<Project>
  updateProject(projectId: string, input: UpdateProjectInput): Promise<Project>
  archiveProject(projectId: string): Promise<void>
```

---

### 8. Application Runtime

```yaml
component: ApplicationRuntime
purpose: "Hosts and manages UI applications (Workspace, CLI, Mobile) and their lifecycle."
responsibilities:
  - "Application registration"
  - "Routing between applications"
  - "Shared state management"
  - "Theme and layout persistence"
  - "Plugin UI integration"

interface:
  registerApp(app: VestaraApplication): Promise<void>
  navigate(target: string): Promise<void>
  getActiveApp(): string
  listApps(): ApplicationInfo[]
```

---

## 🔄 Startup Sequence

```
1. Kernel.boot()
2.   └─ Configuration loaded (config files, env, defaults)
3.   └─ Logging initialized
4.     └─ ServiceBus.initialize()
5.       └─ Plugin Loader: scan and load plugins
6.         └─ Memory Runtime: initialize stores
7.           └─ Provider Runtime: register configured providers
8.             └─ Agent Runtime: initialize, register built-in tools
9.               └─ Workspace Runtime: open last workspace or create default
10.                └─ Application Runtime: boot UI
11.                  └─ Kernel.status = Running
```

---

## 🔄 Shutdown Sequence

```
1. Kernel.shutdown()
2.   └─ Application Runtime: save state, close UI
3.     └─ Workspace Runtime: save workspace state
4.       └─ Agent Runtime: save agent states, cancel running
5.         └─ Provider Runtime: health check stop
6.           └─ Memory Runtime: final consolidation
7.             └─ Plugin Loader: deactivate plugins
8.               └─ ServiceBus: drain events
9.                 └─ Kernel.status = Stopped
```

---

## 🔗 Runtime Component Dependencies

```
Kernel ← depends on nothing (bootstraps everything)
Host Runtime ← depends on: Kernel lifecycle services
Boot Runtime ← depends on: Host Runtime
ServiceBus ← depends on: Configuration
Plugin Loader ← depends on: ServiceBus
Memory Runtime ← depends on: ServiceBus (for events), Plugin Loader (for storage plugins)
Provider Runtime ← depends on: ServiceBus, Plugin Loader
Agent Runtime ← depends on: ServiceBus, Provider Runtime, Tool Runtime, Memory Runtime
Tool Runtime ← depends on: ServiceBus (for permission service)
Workspace Runtime ← depends on: ServiceBus, Memory Runtime
Application Runtime ← depends on: Workspace Runtime
```

Implementation evidence: `evillan0315/vestara-ai-core` at `579df3f`,
`packages/host-runtime`, `packages/boot-runtime`, and `os/systemd`.

## 🎯 Universal Interface (All Runtimes)

Every runtime component implements:

```typescript
interface VestaraRuntime {
  id: string;
  version: string;
  
  initialize(config?: Record<string, unknown>): Promise<void>;
  start(): Promise<void>;
  stop(): Promise<void>;
  health(): Promise<HealthStatus>;
  dispose(): Promise<void>;
}
```

---

*The VRA defines what runs inside Vestara. Every runtime component shares the same lifecycle, communicates through the same bus, and is independently replaceable.*
