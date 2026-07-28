# Phase 3 Handoff — Architecture to Implementation

## Vestara AI Core v0.1 — The First Working Runtime

> **Architecture is frozen. No new architecture documents unless implementation reveals a genuine gap. From this point forward, every commit must leave the platform in a runnable state. Code earns its right to exist by implementing the architecture — not by redefining it.**

---

## 📋 The Architecture Is Complete

| Repository | Role | Status | 
| --- | --- | --- |
| `vestara-blueprint/` | Governance, vision, standards (WHY) | ✅ Frozen v1.0 |
| `vestara-specifications/` | Contracts, APIs, events (WHAT) | ✅ Frozen v1.0 |
| `vestara-foundation/` | Core primitives, SDKs, interfaces (LANGUAGE) | ✅ Frozen v1.0 |
| `vestara-runtime/` | Kernel, lifecycle, observability (ENGINE) | ✅ Frozen v1.0 |

**The five-layer dependency graph is immutable:**

```text
Blueprint → Specifications → Foundation → Runtime → AI Core → Workspace
```

**No component may depend on a component at a higher layer.** This is enforced by convention and will be enforced by CI.

---

## 🎯 Phase 3 Goal: Vestara AI Core v0.1 — "Minimal Viable Runtime"

**Build vertically, not horizontally.** The goal is not to complete every subsystem to 100%. The goal is to produce a working, demonstrable system as quickly as possible.

### v0.1 Target Features

| # | Feature | Contract Reference | Priority |
| --- | --------- | ------------------- | ---------- |
| 1 | Boot the Runtime Kernel | `RT-001: VESTARA-KERNEL.md` | P0 |
| 2 | Register services via Service Registry | `RT-001: Kernel → Service Registry` | P0 |
| 3 | Load OpenCode provider | `FND-007: PROVIDER-SDK.md` | P0 |
| 4 | Create one conversation | `FND-001: VOM-Conversation` | P0 |
| 5 | Send message, stream response | `CAP-001: Workspace.Chat` | P0 |
| 6 | Persist conversation history | `SPEC-DATA-000: Conversation entity` | P0 |
| 7 | Read project files (tool) | `FND-004: T-001 Read File` | P0 |
| 8 | Execute one tool in conversation | `FND-004: Tool execution flow` | P0 |
| 9 | Recover from restart (state persistence) | `RT-002: Service lifecycle` | P0 |
| 10 | Structured logging | `RT-003: LOGGING-ARCHITECTURE.md` | P0 |
| 11 | Health metrics | `RT-004: METRICS-ARCHITECTURE.md` | P0 |

### v0.1 Non-Goals

| Feature | When |
| --------- | ------ |
| Memory engine | v0.2 |
| RAG / Knowledge engine | v0.2 |
| Multiple providers | v0.3 |
| Agent orchestration | v0.2 |
| Missions | v0.3 |
| Voice | v0.4 |
| Plugin system | v0.4 |
| UI (Workspace) | Phase 4 |
| OS integration | Phase 5 |

---

## 🏗️ Implementation Sequence (Incremental Milestones)

### 3.1 — Runtime Bootstrap

**Goal**: The Kernel boots, loads configuration, initializes logging and metrics.

**Files to create**: `packages/kernel/`, `packages/event-bus/`, `packages/configuration/`, `packages/logging/`, `packages/metrics/`

**Contracts to implement**:

- `RT-001: VESTARA-KERNEL.md` — `VestaraKernel.boot()`, `VestaraKernel.shutdown()`
- `FND-006: UNIVERSAL-INTERFACE.md` — `VestaraService`, `Logger`, `MetricsCollector`, `ConfigurationProvider`
- `RT-003: LOGGING-ARCHITECTURE.md` — Structured JSON logging, level manager, console sink
- `RT-004: METRICS-ARCHITECTURE.md` — Counter, gauge, histogram; Prometheus export endpoint

**Verification**: `Kernel.boot()` completes without errors. Logger writes structured JSON to stdout. Metrics endpoint returns data.

---

### 3.2 — Provider Runtime

**Goal**: Load and verify the OpenCode provider. Health check passes.

**Files to create**: `packages/provider-runtime/`, `packages/providers/opencode/`

**Contracts to implement**:

- `FND-007: PROVIDER-SDK.md` — `AIProvider` interface, `CompletionRequest`, `CompletionResponse`, `StreamChunk`
- `SVC-007: Provider Service` — Health check, model listing
- Provider config: `ProviderConfig` schema, API key injection from OS keychain

**Verification**: `provider.healthCheck()` returns healthy. `provider.listModels()` returns OpenCode models.

---

### 3.3 — Conversation Runtime

**Goal**: Create a conversation, send a message, receive a non-streaming response.

**Files to create**: `packages/conversation-runtime/`

**Contracts to implement**:

- `FND-001: VOM-Conversation` — Conversation lifecycle (Created → Active → Archived)
- `FND-001: VOM-Message` — Message model (role, content, tool_calls)
- `CAP-001: Workspace.Chat` — Context assembly algorithm
- `SVC-003: Conversation Service` — Create, send message, history

**Verification**: `POST /api/v1/chat` with `{"content": "Hello", "stream": false}` returns a valid response.

---

### 3.4 — Streaming

**Goal**: Streaming responses work via SSE.

**Files to modify**: `packages/conversation-runtime/`

**Contracts to implement**:

- `FND-007: PROVIDER-SDK.md` — `AIProvider.stream()`, `StreamChunk` types
- Stream handler: token-by-token delivery, `done` signal, error handling

**Verification**: `POST /api/v1/chat` with `{"stream": true}` returns `text/event-stream` with tokens.

---

### 3.5 — Tool Execution

**Goal**: Execute a file read tool within a conversation.

**Files to create**: `packages/tool-runtime/`, `packages/tools/filesystem/`

**Contracts to implement**:

- `FND-004: TOOL-CATALOG.md` — `T-001: Read File` (JSON Schema params, permission level, timeout)
- Tool execution flow: tool call → permission check → execution → result collection
- File system sandbox: path traversal protection, project root scoping

**Verification**: Ask the AI to "read src/index.ts" — it returns the file contents.

---

### 3.6 — Persistence

**Goal**: Conversations survive process restart.

**Files to create**: `packages/storage/sqlite/`

**Contracts to implement**:

- `FND-001: VOM-Conversation` — `created_at`, `updated_at`, persistence
- `FND-001: VOM-Message` — Message storage, ordered retrieval
- `RT-002: LIFECYCLE-SPECIFICATION.md` — Service lifecycle: save state on stop, restore on start

**Verification**: Start → create conversation → stop → start → conversation history is available.

---

### 3.7 — Service Registry & Lifecycle

**Goal**: All services register, dependency graph resolves, startup/shutdown is graceful.

**Files to modify**: `packages/kernel/`, `packages/service-registry/`

**Contracts to implement**:

- `FND-006: UNIVERSAL-INTERFACE.md` — `ServiceRegistry`, service discovery
- `RT-002: LIFECYCLE-SPECIFICATION.md` — Service lifecycle state machine (Created → Initializing → Healthy → Running → Draining → Stopped)
- Kernel boot sequence: resolve dependencies → start in order
- Kernel shutdown sequence: drain → stop in reverse order

**Verification**: `Kernel.boot()` starts all services in dependency order. `Kernel.shutdown()` drains and stops gracefully.

---

### 3.8 — Recovery

**Goal**: Platform recovers from crash without data loss.

**Files to modify**: `packages/kernel/`, `packages/storage/`

**Contracts to implement**:

- `RT-002: LIFECYCLE-SPECIFICATION.md` — Crash detection, state restoration
- `RT-001: VESTARA-KERNEL.md` — Error handling policy (retry, degrade, halt)
- Storage: WAL mode SQLite, checkpoint on graceful shutdown

**Verification**: Kill the process mid-conversation → restart → conversation resumes from last persisted message.

---

### 3.9 — First Working Assistant

**Goal**: Someone can actually talk to it. A CLI or minimal API demonstrates the full flow.

**Files to create**: `packages/cli/` (minimal REPL)

**Integration test**:

```
$ vestara
> Hello
Hello! I'm Vestara AI. How can I help you today?
> Read src/index.ts
[returns file contents]
> /memory
[shows conversation memory]
> /quit
$ vestara
> /history
[shows previous conversation]
```

---

## 📁 Package Structure for Phase 3

```
vestara-ai-core/
├── package.json
├── tsconfig.json
├── vitest.config.ts
│
├── packages/
│   ├── kernel/              # Kernel, boot manager
│   ├── event-bus/           # Typed pub/sub
│   ├── service-registry/    # Service discovery
│   ├── configuration/       # Config loader, Zod validation
│   ├── logging/             # Structured logger
│   ├── metrics/             # Counters, histograms, export
│   ├── storage/             # SQLite wrapper
│   ├── provider-runtime/    # Provider manager, fallback
│   ├── providers/           # Provider implementations
│   │   ├── opencode/        # OpenCode provider
│   │   ├── ollama/          # Ollama provider (v0.3)
│   │   └── openai/          # OpenAI provider (v0.3)
│   ├── conversation-runtime/ # Conversation engine
│   ├── tool-runtime/        # Tool execution sandbox
│   ├── tools/               # Built-in tool implementations
│   │   ├── filesystem/      # read, write
│   │   ├── shell/           # execute
│   │   └── knowledge/       # search (v0.2)
│   ├── memory-runtime/      # Memory engine (v0.2)
│   ├── knowledge-runtime/   # Knowledge engine (v0.2)
│   ├── agent-runtime/       # Agent manager (v0.2)
│   └── shared/              # Shared utilities
│
├── apps/
│   └── cli/                 # Minimal REPL for testing
│
└── tests/
    ├── integration/          # End-to-end runtime tests
    └── performance/          # Benchmarks
```

---

## 🧪 Testing Strategy

| Test Type | Scope | Tool | Frequency |
| ----------- | ------- | ------ | ----------- |
| **Unit** | Individual services, pure logic | Vitest | Every commit |
| **Integration** | Service chains (boot → provider → conversation) | Vitest + real SQLite | Every PR |
| **E2E** | Full flow from CLI | Vitest + child process | Every merge to main |
| **Performance** | Latency p50/p95/p99, memory, tokens/sec | k6 or custom bench | Weekly |
| **Recovery** | Crash → restart → resume | Custom script | Per milestone |

**Testing principle**: Every test uses real SQLite (`:memory:`), real provider connections (with recorded fixtures for CI), and real file system operations (in temp directories). **No mocks for infrastructure.**

---

## 📏 Engineering Rules (Implementation Phase)

| Rule | Enforcement |
| ------ | ------------- |
| **Every commit is runnable** | CI must pass before merge |
| **No partially integrated subsystems** | Feature flags for work-in-progress, never dead code |
| **Contracts before code** | Interface first, implementation second |
| **Architecture freeze respected** | No silent deviation from frozen contracts |
| **`pnpm build` must pass** | Every PR verified |
| **`pnpm test` coverage ≥80%** | Services only (infrastructure may be lower) |
| **No `console.log`** | Use `packages/logging` |
| **No `any` types** | Strict TypeScript, Zod at boundaries |
| **Provider abstraction always** | No hardcoded provider calls |

---

## 🔗 Quick Reference: Key Contracts for Phase 3

| Need | Open This |
| ------ | ----------- |
| What objects exist? | `vestara-foundation/object-model/VESTARA-OBJECT-MODEL.md` |
| How does the Kernel work? | `vestara-runtime/kernel/VESTARA-KERNEL.md` |
| What's the provider contract? | `vestara-foundation/provider-sdk/PROVIDER-SDK.md` |
| What tools are built-in? | `vestara-foundation/tool-catalog/TOOL-CATALOG.md` |
| What's the conversation spec? | `vestara-specifications/capabilities/CAP-001-workspace-chat.md` |
| What's the logging architecture? | `vestara-runtime/logging/LOGGING-ARCHITECTURE.md` |
| What metrics do I emit? | `vestara-runtime/metrics/METRICS-ARCHITECTURE.md` |
| What's the lifecycle pattern? | `vestara-runtime/lifecycle/LIFECYCLE-SPECIFICATION.md` |
| What are the services? | `vestara-foundation/service-contracts/SERVICE-CATALOG.md` |
| What are the universal interfaces? | `vestara-foundation/interfaces/UNIVERSAL-INTERFACE.md` |
| Can I change the architecture? | No — read `vestara-runtime/architecture-v1.0/ARCHITECTURE-FREEZE.md` |

---

## ✅ v0.1 Definition of Done

```markdown
- [ ] Kernel boots, config loads, logging + metrics initialized
- [ ] OpenCode provider loaded, health check passes
- [ ] Conversation created, message sent, response received (non-streaming)
- [ ] Streaming response works via SSE
- [ ] File read tool executes within conversation
- [ ] Conversation history persists across restarts
- [ ] All services register in Service Registry
- [ ] Graceful shutdown saves state
- [ ] Crash recovery: process killed → restarted → conversation resumes
- [ ] `vestara doctor` returns health score ≥80%
- [ ] `pnpm build` passes
- [ ] `pnpm test` passes with ≥80% coverage on services
```

---

*Architecture is complete. The design phase is over. Phase 3 is where Vestara stops being a specification and starts becoming a platform.*

*Every commit must leave the platform in a runnable state. No long-lived broken branches. No "we'll wire it up later." Small, integrated, continuously executable.*

*From this point forward: the code earns its right to exist by implementing the architecture — not by redefining it.*
