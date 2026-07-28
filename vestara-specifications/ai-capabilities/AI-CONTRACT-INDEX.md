---
id: "SPEC-AI-000"
title: "AI Capability Contracts — Complete AI Subsystem Specifications"
owner: "@ai-engineer"
status: "draft"
blueprint-ref: "05-ai-core/AI_OVERVIEW.md"
version: "1.0.0"
---

# AI Capability Contracts
## Precise Specifications for Every AI Subsystem

> **Instead of "use AI," Vestara defines exactly what every AI subsystem must do. Each capability has clearly defined inputs, outputs, responsibilities, performance targets, and interfaces. These contracts make AI subsystems independently implementable and testable.**

---

## Contract Index

| ID | Subsystem | Owner | Status | Gen |
|----|-----------|-------|--------|-----|
| AI-CON-001 | Memory Engine | AI Core Team | 🔄 Draft | Gen 1 |
| AI-CON-002 | Knowledge Engine | AI Core Team | 🔄 Draft | Gen 1 |
| AI-CON-003 | Conversation Engine | AI Core Team | 🔄 Draft | Gen 1 |
| AI-CON-004 | Provider Manager | AI Core Team | ✅ Complete | Gen 1 |
| AI-CON-005 | Agent Runtime | AI Core Team | 🔄 Draft | Gen 1 |
| AI-CON-006 | Prompt Engine | AI Core Team | 📋 Planned | Gen 1 |
| AI-CON-007 | Evaluation Engine | AI Core Team | 📋 Planned | Gen 2 |
| AI-CON-008 | Safety Layer | AI Core Team | 🔄 Draft | Gen 1 |
| AI-CON-009 | Planning Engine | AI Core Team | 🔮 Future | Gen 2 |
| AI-CON-010 | Reasoning Engine | AI Core Team | 🔮 Future | Gen 2 |
| AI-CON-011 | Voice Engine | AI Core Team | 🔮 Future | Gen 2 |
| AI-CON-012 | Vision Engine | AI Core Team | 🔮 Future | Gen 3 |
| AI-CON-013 | Automation Engine | AI Core Team | 🔮 Future | Gen 2 |
| AI-CON-014 | Learning Engine | AI Core Team | 🔮 Future | Gen 4 |

---

## AI-CON-001: Memory Engine

```yaml
id: "AI-CON-001"
name: "Memory Engine"
description: "Persistent memory across sessions with automatic consolidation, importance scoring, and search"
blueprint-ref: "05-ai-core/memory/01-memory-architecture.md"
```

### Inputs

| Input | Source | Frequency | Format |
|-------|--------|-----------|--------|
| Conversation text | Conversation Engine | Per message | `{ userId, content, timestamp }` |
| User preferences | Settings Service | On change | `{ userId, key, value }` |
| Project context | Project Service | On change | `{ projectId, name, description }` |
| Agent execution results | Agent Runtime | Per execution | `{ agentId, task, result }` |
| Explicit saves | User | On demand | `{ userId, content, importance }` |
| Deletion requests | User | On demand | `{ memoryId }` |

### Outputs

| Output | Consumer | Format |
|--------|----------|--------|
| Relevant memories | Conversation Engine | `{ memories: Memory[], scores: number[] }` |
| Memory summaries | Agent Runtime | `{ summary: string, keyPoints: string[] }` |
| Importance scores | Internal | `{ memoryId, score: 0-10 }` |
| Consolidated archives | Storage | `{ period: DateRange, compressed: boolean }` |
| Search results | Any consumer | `{ results: Memory[], total: number }` |

### Responsibilities

| Responsibility | Implementation | Verification |
|----------------|----------------|--------------|
| **Short-term memory** | Session-scoped, in-memory store, auto-expire on session end | Session isolation test |
| **Long-term memory** | SQLite persistent store, indexed by userId + projectId | Data persistence test |
| **Importance scoring** | Algorithm: recency(0.3) + frequency(0.2) + userFeedback(0.2) + novelty(0.15) + impact(0.15) | Scoring consistency test |
| **Forgetting/consolidation** | Scheduled every 50 interactions; low-score → summary → archive; high-score preserved | Consolidation accuracy test |
| **Search** | Hybrid FTS5 + vector similarity; ranked by relevance + importance | Search relevance benchmark |
| **Memory ranking** | Sort by importance score; top 100 loaded into context window | Context assembly test |
| **Cross-session persistence** | Memories survive system restarts, OS updates, SSD migrations | Persistence integration test |

### Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Store latency | <5ms p95 | Instrumented |
| Search latency (FTS) | <50ms for 100k memories | Benchmarked |
| Search latency (vector) | <100ms for 100k memories (Gen 2) | Benchmarked |
| Consolidation cycle | <5s for 10k memories | Timed automation |
| Memory load for context | <200ms for 100 memories | Timed |
| Storage per memory | <1KB average | Measured |
| Maximum memories/user | 1M (Gen 1) | Load tested |

### Interfaces

```typescript
interface MemoryService {
  // Store
  addMemory(userId: string, type: MemoryType, content: string, metadata?: Record<string, unknown>): Promise<Memory>;
  addMemories(userId: string, memories: MemoryInput[]): Promise<Memory[]>;
  deleteMemory(userId: string, memoryId: string): Promise<void>;
  
  // Search
  searchMemories(userId: string, query: string, options?: SearchOptions): Promise<MemorySearchResult>;
  getMemoriesForContext(userId: string, limit?: number): Promise<Memory[]>;
  
  // Consolidation
  triggerConsolidation(userId: string): Promise<ConsolidationResult>;
  getConsolidationStatus(userId: string): Promise<ConsolidationStatus>;
  
  // Scoring
  setImportance(userId: string, memoryId: string, score: number): Promise<void>;
  getImportanceDistribution(userId: string): Promise<ImportanceDistribution>;
}

interface Memory {
  id: string;
  userId: string;
  type: 'fact' | 'preference' | 'event' | 'decision' | 'conversation';
  content: string;             // The memory content
  summary?: string;            // Consolidated summary
  importance: number;          // 0-10
  embedding?: number[];        // Vector embedding (Gen 2)
  metadata: Record<string, unknown>;
  createdAt: string;
  updatedAt: string;
  consolidatedAt?: string;
}
```

### Dependencies

| Dependency | Purpose |
|------------|---------|
| Database (SQLite) | Persistent storage |
| EventBus | Emit memory events |
| Embedding Service (Gen 2) | Vector embeddings for search |
| Filesystem | .vestara/memory/ storage |

---

## AI-CON-002: Knowledge Engine

```yaml
id: "AI-CON-002"
name: "Knowledge Engine"
description: "Document storage, full-text and vector search, and retrieval-augmented generation (RAG)"
blueprint-ref: "05-ai-core/knowledge/"
```

### Inputs

| Input | Source | Format |
|-------|--------|--------|
| Documents | User upload, web import, Git import | `{ content, type, metadata }` |
| Search queries | Conversation Engine, Agent Runtime | `{ query, filters, limit }` |
| Indexing requests | Filesystem watcher, webhook | `{ path, changed, deleted }` |

### Outputs

| Output | Consumer | Format |
|--------|----------|--------|
| Search results | Conversation Engine, Agent Runtime | `{ documents, scores }` |
| RAG context | Conversation Engine | `{ context, sources }` |
| Index status | Admin UI | `{ indexed, pending, failed }` |

### Responsibilities

| Responsibility | Detail |
|----------------|--------|
| **Document management** | CRUD, versioning, metadata, tagging |
| **Full-text search** | SQLite FTS5 tokenization, ranking, highlighting |
| **Vector search (Gen 2)** | Embedding generation, vector index, similarity search |
| **Hybrid search (Gen 2)** | Combined FTS + vector with reciprocal rank fusion |
| **RAG pipeline** | Query → retrieve → rerank → augment → generate |
| **Auto-indexing** | Filesystem watcher → auto-detect new content |
| **Chunking** | Smart document chunking with overlap for RAG |

### Performance Targets

| Metric | Target |
|--------|--------|
| FTS search latency | <50ms p95 |
| RAG pipeline latency | <1s p95 (excluding generation) |
| Document indexing | <100ms for 1K document |
| Chunking throughput | >1MB/s |
| Storage efficiency | <2x original document size (with indexes) |

---

## AI-CON-003: Conversation Engine

```yaml
id: "AI-CON-003"
name: "Conversation Engine"
description: "Multi-turn conversation management with context assembly, tool calling, and streaming"
blueprint-ref: "05-ai-core/conversation/"
```

### Responsibilities

| Responsibility | Detail |
|----------------|--------|
| **Session management** | Create, resume, archive conversations |
| **Context assembly** | Combine system prompt + memory + knowledge + conversation history |
| **Context window optimization** | Sliding window, token budgeting, priority-based truncation |
| **Tool orchestration** | Parse tool calls, execute, collect results, continue |
| **Streaming** | Real-time token delivery via SSE |
| **Cancellation** | User can cancel in-flight generation |
| **Error recovery** | Provider failover, retry with backoff, graceful degradation |

### Context Assembly Algorithm

```
1. Load system prompt (permanent, fixed tokens)
2. Load relevant memories (top 100 by importance)
3. Load relevant knowledge (if query detected)
4. Load conversation history (sliding window of last N messages)
5. Token budget calculation: target = model.maxTokens - reservedForResponse
6. If over budget: truncate conversation history (oldest first)
7. If still over budget: summarizd old conversation segments
8. If still over budget: reduce memories (lower importance removed)
9. Assemble final context: System + Memories + Knowledge + History
```

---

## AI-CON-005: Agent Runtime

```yaml
id: "AI-CON-005"
name: "Agent Runtime"
description: "Create, execute, and manage AI agents with tools, memory, and persistence"
blueprint-ref: "05-ai-core/agents/"
```

### Agent Lifecycle

```
Created → Idle → Executing → Paused → Completed/Failed → Archived
                        ↑         |
                        └─────────┘
                           (Resumed)
```

### Responsibilities

| Responsibility | Detail |
|----------------|--------|
| **Agent creation** | Define agent name, model, tools, instructions |
| **Execution** | Receive task → plan → execute → return result |
| **Tool registry** | Register, discover, invoke tools |
| **Sandbox** | Isolated execution environment with resource limits |
| **Persistence** | Agent state survives restarts |
| **Streaming** | Real-time execution progress |
| **Cancellation** | Graceful agent stop |
| **Multi-agent** | Agent-to-agent communication (Gen 3) |

### Built-in Tools

```typescript
const BUILTIN_TOOLS = {
  read_file: {
    description: "Read file contents",
    parameters: { path: "string" },
    permission: "read-only",
    sandbox: true,
  },
  write_file: {
    description: "Write content to file",
    parameters: { path: "string", content: "string" },
    permission: "user-approve",
    sandbox: true,
  },
  execute_command: {
    description: "Execute a shell command",
    parameters: { command: "string" },
    permission: "user-approve",
    sandbox: true,
    timeout: 30000,
  },
  search_knowledge: {
    description: "Search the knowledge base",
    parameters: { query: "string" },
    permission: "read-only",
  },
  search_memory: {
    description: "Search user memories",
    parameters: { query: "string" },
    permission: "read-only",
  },
  web_search: {
    description: "Search the web",
    parameters: { query: "string" },
    permission: "user-approve",
    requires: "network",
  },
  create_task: {
    description: "Create a task in the current project",
    parameters: { title: "string", description: "string", status: "string" },
    permission: "user-approve",
  },
};
```

### Performance Targets

| Metric | Target |
|--------|--------|
| Agent creation | <100ms |
| Execution startup | <500ms |
| Tool invocation | <100ms |
| Tool timeout default | 30s |
| Max execution time | 5min |
| Memory per agent | <50MB |

---

## AI-CON-008: Safety Layer

```yaml
id: "AI-CON-008"
name: "Safety Layer"
description: "Prevent harmful outputs, protect user privacy, enforce boundaries"
blueprint-ref: "05-ai-core/safety/"
```

### Guard Chain (Applied to Every AI Interaction)

```
User Input
  ↓
┌─────────────────────┐
│  Input Guard Rails  │
│  ─────────────────  │
│  • Prompt injection │
│  • PII detection    │
│  • Toxicity filter  │
│  • Jailbreak check  │
│  • Rate limit       │
└─────────┬───────────┘
          ↓ (pass/fail)
┌─────────────────────┐
│  Context Assembly   │
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│  AI Provider        │
└─────────┬───────────┘
          ↓
┌─────────────────────┐
│  Output Guard Rails │
│  ─────────────────  │
│  • Content safety   │
│  • PII leak detect  │
│  • Code safety      │
│  • Hallucination    │
└─────────┬───────────┘
          ↓ (pass/fail)
User Output
```

### Guard Specifications

| Guard | Check | Action on Violation |
|-------|-------|---------------------|
| **Prompt injection** | Regex + ML pattern matching | Block + log + alert |
| **PII detection** | Regex patterns (email, SSN, phone, credit card) | Redact + log |
| **Toxicity** | ML content classifier | Block + log |
| **Jailbreak** | Known escape pattern matching | Block + log + alert |
| **Rate limit** | Token bucket per user | HTTP 429 + retry-after |
| **Content safety** | Output ML classifier | Block + regenerate |
| **PII leak** | Output regex scanning | Redact + log |
| **Code safety** | SAST on generated code | Block dangerous patterns |
| **Factual accuracy** | Entailment checking | Flag + cite sources |

### Performance Targets

| Metric | Target |
|--------|--------|
| Input guard latency | <10ms |
| Output guard latency | <50ms |
| False positive rate | <1% |
| False negative rate | <0.1% |
| Coverage | 100% of AI interactions |

---

**Total AI Contracts Defined: 8** across Gen 1-4.

*Every AI capability in Vestara has a precise contract. These contracts make AI independently implementable, testable, and improvable.*
