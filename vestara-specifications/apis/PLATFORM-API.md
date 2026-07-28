---
title: "AI Core Platform API — Public Surface Specification"
volume: "10-developer-platform"
book: "Book 4: Engineering"
version: "1.0.0"
status: "ratified"
owner: "@chief-architect"
last-reviewed: "2025-07-23"
tags: ["api", "platform", "interface", "public-surface"]
---

# AI Core Platform API
## The Public Contract Consumed by Every Future Vestara Product

> **No internal package should be imported directly by applications. The AI Core exposes a consistent facade. Every product — Workspace, IDE, OS, Cloud, SDK — consumes this API. Internal packages can be refactored freely without breaking downstream consumers.**

---

## Architecture Boundary

```
Application Layer
────────────────────────────
CLI • Workspace • IDE • OS

──────── Platform API ────────

AI Core Public Surface
  conversation.*
  memory.*
  knowledge.*
  reasoning.*
  planning.*
  mission.*
  action.*
  provider.*

──────── Runtime ─────────────

Internal Packages
  Kernel • State • Metrics • Logger • Events • Configuration
```

---

## Platform API Surface

```typescript
// ─── Conversation ──────────────────────────────────────────

interface ConversationAPI {
  create(userId?: string): Promise<Conversation>;
  send(conversationId: string, content: string): Promise<Message>;
  stream(conversationId: string, content: string): AsyncIterable<StreamChunk>;
  list(userId?: string): Promise<ConversationSummary[]>;
  get(id: string): Promise<Conversation | null>;
  close(id: string): Promise<void>;
}

// ─── Memory ────────────────────────────────────────────────

interface MemoryAPI {
  store(userId: string, input: MemoryInput): Promise<Memory>;
  retrieve(userId: string, query: string, limit?: number): Promise<Memory[]>;
  consolidate(userId: string): Promise<ConsolidationReport>;
  getContext(userId: string, limit?: number): Promise<Memory[]>;
  delete(memoryId: string): Promise<void>;
  stats(userId: string): Promise<MemoryStats>;
}

// ─── Knowledge ─────────────────────────────────────────────

interface KnowledgeAPI {
  index(rootDir: string): Promise<IndexReport>;
  search(query: string, limit?: number): Promise<SearchResult[]>;
  analyzeRepository(files: string[]): ProjectInfo;
  getDocument(id: string): Promise<KnowledgeDocument | null>;
  getStats(): Promise<{ documents: number; chunks: number }>;
}

// ─── Reasoning ─────────────────────────────────────────────

interface ReasoningAPI {
  execute(input: string, context?: ReasoningContext): Promise<ReasoningResult>;
  selectStrategy(input: string): ReasoningStrategyId;
  getMetrics(): ReasoningMetrics;
}

// ─── Action ────────────────────────────────────────────────

interface ActionAPI {
  execute(toolId: string, params: Record<string, unknown>): Promise<ActionResult>;
  registerTool(tool: Tool): void;
  listTools(): ToolDefinition[];
  getPermissionEngine(): PermissionEngine;
}

// ─── Provider ──────────────────────────────────────────────

interface ProviderAPI {
  list(): ProviderInfo[];
  getModels(providerId?: string): AIModel[];
  health(): Promise<ProviderHealthStatus[]>;
  execute(request: CompletionRequest): Promise<CompletionResponse>;
  stream(request: CompletionRequest): AsyncIterable<StreamChunk>;
}
```

---

## Workspace Context

Every conversation automatically includes workspace context:

```typescript
interface WorkspaceContext {
  projectName?: string;
  projectType?: string;
  language?: string;
  framework?: string;
  rootDir?: string;
  recentFiles: string[];
  activeFile?: string;
  repositoryInfo?: ProjectInfo;
  missionId?: string;
}
```

The user shouldn't need to tell Vestara what project they're working on — it infers it from the active workspace.

---

## Canonical Planning Object

Every planning strategy produces the same artifact:

```typescript
interface Plan {
  id: string;
  goal: string;
  constraints: string[];
  assumptions: string[];
  steps: PlanStep[];
  dependencies: PlanDependency[];
  risks: PlanRisk[];
  estimatedCost: string;
  estimatedDuration: string;
  successCriteria: string[];
  createdAt: string;
}

interface PlanStep {
  order: number;
  title: string;
  description: string;
  estimatedEffort: string;
  assignedTo?: string;
  status: 'pending' | 'in_progress' | 'completed' | 'blocked';
}

interface PlanDependency {
  from: string;
  to: string;
  type: 'blocks' | 'requires' | 'related';
}

interface PlanRisk {
  description: string;
  probability: 'low' | 'medium' | 'high';
  impact: 'low' | 'medium' | 'high';
  mitigation: string;
}
```

---

## Knowledge Repository Model (v0.3)

Instead of flat file search, build a semantic model:

```typescript
interface RepositoryModel {
  name: string;
  type: string;
  language: string;
  modules: RepoModule[];
  dependencies: RepoDependency[];
  symbols: RepoSymbol[];
  entryPoints: string[];
}

interface RepoModule {
  name: string;
  path: string;
  exports: string[];
  imports: string[];
  symbols: string[];
}

interface RepoDependency {
  source: string;
  target: string;
  type: 'import' | 'extends' | 'implements' | 'uses';
}

interface RepoSymbol {
  name: string;
  kind: 'class' | 'function' | 'interface' | 'type' | 'variable' | 'component';
  module: string;
  exported: boolean;
  doc?: string;
}
```

This enables questions like:
- "Where is AuthenticationService used?"
- "What modules depend on the database layer?"
- "Show me all exported interfaces in the API package."

---

## Platform API Versioning

| Version | Changes | Status |
|---------|---------|--------|
| v0.1 | Initial conversation, memory, knowledge, action APIs | ✅ Complete |
| v0.2 | Added reasoning API | ✅ Complete |
| v0.3 | Knowledge graph, repository model, workspace context | 📋 Planned |

---

*The Platform API is the public contract. Everything external consumes this surface. Internal packages are implementation details. This boundary gives Vestara the freedom to refactor internals without breaking downstream products.*
