# Epic 001 — Repository Comprehension Implementation Plan

## Overview

Implement `vestara open .` as the defining first impression of the Vestara platform. The command opens a repository, establishes its identity, analyzes it deterministically, indexes it for knowledge, presents a synthesized understanding, and drops the user into a workspace-aware interactive session.

The architecture is built for reuse from day one: a `WorkspaceRuntime` in a new `@vestara/workspace` package handles all orchestration, while the CLI remains a thin command handler.

## Current State Analysis

### What Already Exists (verified against actual code)

| Component | Location | Status |
|-----------|----------|--------|
| File tree walker + ignore logic | `packages/knowledge/src/index.ts` — `DefaultKnowledgeIndexer.indexDirectory()` (L381-421) | ✅ Complete |
| Document parser (20+ langs) | `packages/knowledge/src/index.ts` — `DefaultDocumentParser` (L91-121) | ✅ Complete |
| Chunk engine | `packages/knowledge/src/index.ts` — `DefaultChunkEngine` (L131-152) | ✅ Complete |
| SQLite knowledge storage | `packages/knowledge/src/index.ts` — `KnowledgeStorage` (L191-289) | ✅ Complete |
| Knowledge Engine (orchestrator) | `packages/knowledge/src/index.ts` — `DefaultKnowledgeEngine` (L458-522) | ✅ Complete |
| Repository analyzer (basic) | `packages/knowledge/src/index.ts` — `DefaultRepositoryAnalyzer` (L299-348) — detects node/go/rust/python/ruby, package manager, monorepo flags | ✅ Functional, shallow |
| Memory runtime (4 layers) | `packages/memory/src/index.ts` — `DefaultMemoryRuntime` (L165-429) | ✅ Complete |
| Reasoning runtime (8 strategies) | `packages/reasoning/src/index.ts` — `DefaultReasoningRuntime` (L278-389), strategy selector (L230-258) | ✅ Complete |
| Cognitive pipeline (5 stages) | `packages/cognitive/src/index.ts` — `DefaultCognitiveEngine` (L280-357) | ✅ Complete |
| CLI with REPL/doctor/golden-path | `apps/cli/src/index.ts` — `main()`, `runDoctor()`, `runGoldenPath()`, `startRepl()` | ✅ Complete |

### What's Missing

1. **Pipeline orchestrator** — No `WorkspaceRuntime` exists to chain the stages
2. **Repository identity** — No fingerprinting: git root, remote, branch, commit, content hash
3. **Deep repository analysis** — `DefaultRepositoryAnalyzer` only inspects root filenames; no entry point detection, no risk detection, no package relationship mapping
4. **AI synthesis** — No Executive Brain invocation to produce the narrative summary
5. **`.vestara/` manifest** — No on-disk cache for workspace state or incremental reindexing
6. **`vestara open` CLI command** — No command handler; CLI only has `doctor`, `demo golden-path`, and default REPL
7. **Workspace-aware session** — After `open`, the REPL prompt should reflect the active repository; all commands should operate in workspace context
8. **Presentation layer** — No `RepositoryPresenter` to render understanding as CLI output, JSON, or HTML
9. **Full build order** — `build-order.sh` doesn't build all packages; doesn't reflect the real dependency graph

## Desired End State

After this plan is complete, running `pnpm vestara open .` in the `vestara-ai-core/` directory produces:

```
$ pnpm vestara open .

Opening repository...

✓ Repository detected
✓ Repository identified
✓ Project analyzed
✓ Entry points found
✓ Workspace created
✓ Knowledge indexed
✓ Repository understood

vestara-ai-core >
```

The user can then ask questions in the context of that repository, and the response draws on the indexed knowledge and workspace memory.

## What We're NOT Doing (Phase 1)

- **Background indexing** — Phase 1 indexes synchronously. Background indexing is Phase 2.
- **File watching (inotify/fsevents)** — Phase 2.
- **Incremental indexing** — Phase 2.
- **Multiple workspace sessions** — Single workspace per process for now.
- **REST API or desktop UI** — CLI is the only consumer in Phase 1. The architecture supports future consumers via `WorkspaceRuntime`.
- **Embeddings or vector search** — The existing FTS (LIKE-based) knowledge storage is sufficient for Phase 1.
- **Architecture pattern detection** — Phase 3. Phase 1 ships "small but correct" entry points and risks.

---

## Implementation Approach

### Architectural Principle

Build the architecture you ultimately want, ship the smallest useful capability first.

The CLI never imports knowledge, memory, or reasoning directly. It imports `WorkspaceRuntime`, which owns all orchestration. Future consumers (REST API, desktop UI, Vestara OS) import the same `WorkspaceRuntime`.

### The `@vestara/workspace` Package

```
@vestara/workspace
  │
  ├── WorkspaceRuntime         ← open(), close(), getSession(), status()
  ├── WorkspaceSession         ← repository info, engines, conversation
  ├── WorkspaceManifest        ← .vestara/workspace.json read/write
  ├── RepositoryDiscovery      ← file walk + stats
  ├── RepositoryFingerprint    ← git identity + content hash
  ├── RepositoryIntelligence   ← analysis (entry points, risks, packages)
  └── RepositoryPresenter      ← render understanding (CLI, JSON, etc.)
```

### Pipeline Flow

Each stage enriches a single `RepositoryWorkspace` domain object rather than producing unrelated DTOs. The pipeline is a progressive enrichment of one canonical model.

```
WorkspaceRuntime.open(path)
  │
  ├── 1. RepositoryDiscovery
  │     Walk files, count stats
  │     Status: Discovering
  │     Enriches: RepositoryWorkspace.discovery
  │
  ├── 2. RepositoryFingerprint
  │     Establish immutable identity: git root, remote, branch,
  │     commit, content hash. If workspace.json exists and
  │     fingerprint matches, subsequent stages can be skipped.
  │     Status: Fingerprinting
  │     Enriches: RepositoryWorkspace.identity
  │
  ├── 3. RepositoryIntelligence
  │     Deterministic analysis: language, framework, entry points,
  │     package map, risks (TODO count, large files, missing tests)
  │     Status: Analyzing
  │     Enriches: RepositoryWorkspace.analysis
  │
  ├── 4. WorkspaceManifest
  │     Create .vestara/ directory + workspace.json
  │     Serialize RepositoryWorkspace to disk for cache
  │     Status: (transient, implicit)
  │
  ├── 5. KnowledgeIndex
  │     Index all supported files via KnowledgeEngine
  │     Store chunks in .vestara/knowledge/
  │     Status: Indexing
  │     Enriches: RepositoryWorkspace.index
  │
  ├── 6. RepositoryPresenter
  │     Feed RepositoryWorkspace to Executive Brain
  │     Generate narrative: purpose, suggested starting points
  │     Render as CLI output, JSON, etc.
  │     Status: Summarizing → Ready
  │     Enriches: RepositoryWorkspace.presentation
  │
  └── 7. WorkspaceSession
      Create in-memory session from the fully-enriched
      RepositoryWorkspace. All engines initialized from its state.
      Return: WorkspaceSession
```

### RepositoryWorkspace — The Canonical Domain Object

```
RepositoryWorkspace
  ├── identity: RepositoryFingerprint     (Stage 2)
  │     id, name, path, git root, remote,
  │     branch, commit, content hash
  │
  ├── discovery: DiscoveryResult          (Stage 1)
  │     file list, total count, total size,
  │     by-extension breakdown
  │
  ├── analysis: RepositoryProfile         (Stage 3)
  │     language, framework, package map,
  │     entry points, risks, test framework
  │
  ├── index: IndexReport                  (Stage 5)
  │     documents indexed, chunks created,
  │     indexing duration
  │
  └── presentation: PresentedSummary      (Stage 6)
        purpose, starting points,
        observations, rendered output
```

Advantages of this model:
- Every stage has one input (the workspace) and one output (the enriched workspace)
- Logging is simpler: "Workspace state after Analyze"
- Serialization of the full workspace is straightforward
- Testing is deterministic: construct a RepositoryWorkspace, run a stage, inspect the enrichment
- Future stages extend the model without changing every interface
- Aligns with the Vestara Object Model philosophy: canonical representation, not transient DTOs

### WorkspaceStatus Enum

```typescript
enum WorkspaceStatus {
  Discovering     = 'discovering',
  Fingerprinting  = 'fingerprinting',
  Analyzing       = 'analyzing',
  Indexing        = 'indexing',
  Summarizing     = 'summarizing',
  Ready           = 'ready',
  IndexingBackground = 'indexing-background',
  Error           = 'error',
}
```

This is the canonical lifecycle. Every UI (CLI, REST, desktop, OS) observes the same enum. No interface invents its own status model.

---

## Phase 1: Functional — "Vestara understands a repository"

### Success Criteria

```
□ `vestara open .` completes without crashing
□ Creates `.vestara/` directory with workspace.json
□ Creates a WorkspaceSession with all engines
□ Produces a RepositoryPresenter output (CLI summary)
□ Drops into workspace-aware REPL with changed prompt
□ No stubs — every analysis rule is "small but correct"
```

### Changes Required

#### 1. New Package: `@vestara/workspace`

**File**: `packages/workspace/package.json`

```json
{
  "name": "@vestara/workspace",
  "version": "0.1.0",
  "private": true,
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "scripts": { "build": "tsc" },
  "dependencies": {
    "@vestara/shared": "workspace:*",
    "@vestara/knowledge": "workspace:*",
    "@vestara/memory": "workspace:*",
    "@vestara/reasoning": "workspace:*",
    "@vestara/conversation": "workspace:*",
    "@vestara/context": "workspace:*",
    "@vestara/logger": "workspace:*",
    "@vestara/event-bus": "workspace:*",
    "@vestara/stream": "workspace:*"
  }
}
```

**File**: `packages/workspace/tsconfig.json`

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": { "outDir": "./dist", "rootDir": "./src" },
  "include": ["src"]
}
```

**File**: `packages/workspace/src/index.ts`

```typescript
export { WorkspaceRuntime } from './workspace-runtime';
export { WorkspaceSession } from './workspace-session';
export { WorkspaceManifest } from './workspace-manifest';
export { WorkspaceStatus } from './types';
export { RepositoryFingerprint } from './repository-fingerprint';
export { RepositoryProfile, EntryPoint, DetectedRisk } from './repository-intelligence';
export { RepositoryPresenter } from './repository-presenter';
export type { WorkspaceStatus, OpenResult } from './types';
```

---

#### 2. WorkspaceRuntime (orchestrator)

**File**: `packages/workspace/src/workspace-runtime.ts`

The central orchestrator. Every pipeline stage is a distinct module — the runtime simply sequences them and tracks status.

```typescript
export class WorkspaceRuntime {
  private status: WorkspaceStatus = WorkspaceStatus.Discovering;
  private session: WorkspaceSession | null = null;
  private eventBus: EventBus;
  private logger: Logger;

  constructor(opts: { eventBus: EventBus; logger: Logger }) { ... }

  get currentStatus(): WorkspaceStatus { return this.status; }

  async open(path: string): Promise<WorkspaceSession> {
    const resolved = path.resolve(path);
    // Stage 1: Discover
    this.status = WorkspaceStatus.Discovering;
    const files = await RepositoryDiscovery.walk(resolved);
    this.emit('workspace:discovered', { fileCount: files.length });

    // Stage 2: Fingerprint
    this.status = WorkspaceStatus.Fingerprinting;
    const fingerprint = await RepositoryFingerprint.create(resolved);
    this.emit('workspace:fingerprinted', { repo: fingerprint.name });

    // Stage 3: Analyze
    this.status = WorkspaceStatus.Analyzing;
    const profile = await RepositoryIntelligence.analyze(files, resolved);
    this.emit('workspace:analyzed', { language: profile.language });

    // Stage 4: Manifest
    const workspaceDir = path.join(resolved, '.vestara');
    const manifest = await WorkspaceManifest.create(workspaceDir, {
      fingerprint, profile,
    });
    this.emit('workspace:manifest.created');

    // Stage 5: Index
    this.status = WorkspaceStatus.Indexing;
    const knowledgeEngine = this.createKnowledgeEngine(workspaceDir);
    const indexReport = await knowledgeEngine.index(resolved);
    this.emit('workspace:indexed', { documents: indexReport.documentsIndexed });

    // Stage 6: Present
    this.status = WorkspaceStatus.Summarizing;
    const presenter = new RepositoryPresenter({ reasoningRuntime: this.createReasoningRuntime() });
    const summary = await presenter.present(profile, indexReport);
    this.emit('workspace:presented');

    // Stage 7: Session
    this.status = WorkspaceStatus.Ready;
    this.session = new WorkspaceSession({
      rootPath: resolved,
      workspaceDir,
      fingerprint,
      profile,
      manifest,
      knowledge: knowledgeEngine,
      memory: this.createMemoryRuntime(workspaceDir),
      conversation: this.createConversation(),
    });
    this.emit('workspace:ready', { session: this.session });
    return this.session;
  }

  async close(): Promise<void> { ... }
  getSession(): WorkspaceSession { ... }
  private emit(type: string, payload: any): void { ... }
}
```

---

#### 3. RepositoryDiscovery

**File**: `packages/workspace/src/repository-discovery.ts`

Pure function. Walks the directory tree and returns a list of files. No analysis.

```typescript
export class RepositoryDiscovery {
  static async walk(rootDir: string): Promise<string[]> {
    // Same ignore logic as DefaultKnowledgeIndexer (node_modules, .git, dist, etc.)
    // Returns relative paths
  }

  static async stats(files: string[]): Promise<{
    totalFiles: number;
    totalSizeKB: number;
    byExtension: Record<string, number>;
  }> { ... }
}
```

---

#### 4. RepositoryFingerprint

**File**: `packages/workspace/src/repository-fingerprint.ts`

Repository identity. Everything below depends on this — memory, knowledge, missions, agent state, plugins, analytics — all answer "which repository does this belong to?"

```typescript
export interface RepositoryFingerprint {
  id: string;                   // hash(canonicalPath)
  name: string;                 // directory basename
  canonicalPath: string;        // resolved absolute path
  gitRoot: string | null;       // git rev-parse --show-toplevel
  gitRemote: string | null;     // git remote get-url origin
  gitBranch: string | null;     // git branch --show-current
  gitCommit: string | null;     // git rev-parse HEAD
  repositoryHash: string;       // sha256 of key config files (package.json, tsconfig, etc.)
  fingerprintedAt: string;      // ISO timestamp
}

export class RepositoryFingerprint {
  static async create(rootDir: string): Promise<RepositoryFingerprint> {
    // Run git commands, compute content hash
    // Handle non-git repos gracefully (git fields become null)
  }
}
```

---

#### 5. RepositoryIntelligence

**File**: `packages/workspace/src/repository-intelligence.ts`

"Small but correct" analysis. No stubs. Phase 1 ships three real risk rules and real entry point detection.

```typescript
export interface EntryPoint {
  path: string;              // relative path from repo root
  type: 'cli' | 'app' | 'api' | 'library' | 'worker';
  source: 'package.json' | 'convention';
}

export interface DetectedRisk {
  category: 'large-file' | 'todo-hotspot' | 'circular-dependency' | 'missing-tests';
  severity: 'low' | 'medium' | 'high';
  location: string;          // file path or package name
  detail: string;
}

export interface PackageNode {
  name: string;
  path: string;
  dependencies: string[];
  devDependencies: string[];
  isPrivate: boolean;
}

export interface RepositoryProfile {
  // Machine facts (deterministic, no AI)
  name: string;
  language: string;
  framework?: string;
  packageManager?: string;
  buildTool?: string;
  testFramework?: string;
  isMonorepo: boolean;
  fileCount: number;
  totalSizeKB: number;
  packageCount: number;
  dependencyCount: number;
  entryPoints: EntryPoint[];
  risks: DetectedRisk[];
  packages: PackageNode[];
  hasDocker: boolean;
  hasCI: boolean;
  detectedAt: string;
}

export class RepositoryIntelligence {
  static async analyze(files: string[], rootDir: string): Promise<RepositoryProfile> {
    // 1. Reuse DefaultRepositoryAnalyzer for language/framework detection
    // 2. Entry points: parse package.json bin/main/module/exports, then scan common dirs
    // 3. Package map: read all package.json files in the tree
    // 4. Risks: three real rules (see below)
  }
}
```

**Entry point detection (Phase 1, three strategies):**

| Strategy | What it checks |
|----------|---------------|
| `package.json` fields | `bin`, `main`, `module`, `exports` fields |
| Common filenames | `src/main.ts`, `src/index.ts`, `main.ts`, `index.ts`, `app.ts`, `server.ts`, `cli.ts` |
| Workspace apps | All files under `apps/*/src/index.ts` or `packages/*/src/index.ts` that have a sibling `package.json` with a `bin` field |

**Risk detection (Phase 1, three real rules):**

| Rule | Threshold | Detects |
|------|-----------|---------|
| Large files | Files >2000 lines | Files that are hard to understand and likely need refactoring |
| TODO/FIXME hotspots | >5 TODOs per file, or any file with >20 TODOs | Areas of incomplete work or technical debt |
| Missing tests | Packages with no `__tests__/` dir, no `*.test.ts` files, and no `test` script in `package.json` | Packages that lack test coverage |

These are correct for what they detect. They don't claim to detect everything. Users forgive incomplete; they don't forgive inaccurate.

**Circular dependency detection** is deferred to Phase 3 (requires full module graph resolution).

---

#### 6. WorkspaceManifest

**File**: `packages/workspace/src/workspace-manifest.ts`

```typescript
export interface WorkspaceManifestData {
  schemaVersion: 1;
  id: string;                       // from fingerprint
  name: string;
  fingerprint: RepositoryFingerprint;
  analysis: RepositoryProfile;
  knowledge: {
    version: number;
    documents: number;
    chunks: number;
    lastIndexedAt: string | null;
  };
  memory: {
    version: number;
    count: number;
    lastConsolidatedAt: string | null;
  };
  openedAt: string;
  lastOpenedAt: string;
}

export class WorkspaceManifest {
  static path(workspaceDir: string): string {
    return path.join(workspaceDir, 'workspace.json');
  }

  static async create(workspaceDir: string, data: {
    fingerprint: RepositoryFingerprint;
    profile: RepositoryProfile;
  }): Promise<WorkspaceManifestData> { ... }

  static async load(workspaceDir: string): Promise<WorkspaceManifestData | null> { ... }

  static async save(workspaceDir: string, data: WorkspaceManifestData): Promise<void> { ... }

  /** Detect whether the cached analysis is still fresh */
  static async isStale(workspaceDir: string, fingerprint: RepositoryFingerprint): Promise<boolean> {
    const manifest = await this.load(workspaceDir);
    if (!manifest) return true;
    return manifest.fingerprint.repositoryHash !== fingerprint.repositoryHash;
  }
}
```

**`.vestara/` directory structure:**

```
.vestara/
  workspace.json          ← WorkspaceManifestData
  knowledge/
    chunks.db             ← SQLite (from KnowledgeStorage)
  memory/
    memories.db           ← SQLite (from MemoryRuntime)
  sessions/
    last.session          ← last active conversation ID
```

---

#### 7. RepositoryPresenter

**File**: `packages/workspace/src/repository-presenter.ts`

Separates the presentation of repository understanding from the runtime. The `WorkspaceRuntime` owns analysis and indexing; the presenter owns rendering.

```typescript
export interface PresentedSummary {
  /** Deterministic facts */
  facts: {
    language: string;
    framework: string | null;
    packageManager: string | null;
    fileCount: number;
    packageCount: number;
    dependencyCount: number;
    isMonorepo: boolean;
    entryPoints: string[];
    risks: Array<{ category: string; severity: string; detail: string }>;
  };
  /** AI-synthesized narrative */
  narrative: {
    purpose: string;
    suggestedStartingPoints: string[];
    keyObservations: string[];
  } | null;  // null until AI call completes
}

export class RepositoryPresenter {
  private reasoningRuntime: ReasoningRuntime;

  constructor(opts: { reasoningRuntime: ReasoningRuntime }) { ... }

  /** Produce a structured summary with AI narrative */
  async present(profile: RepositoryProfile, indexReport: IndexReport): Promise<PresentedSummary> {
    const narrative = await this.synthesizeNarrative(profile);
    return {
      facts: this.extractFacts(profile),
      narrative,
    };
  }

  /** Render as formatted text for CLI output */
  renderCli(summary: PresentedSummary): string {
    // ASCII table with purpose, architecture, entry points, risks
    // e.g.:
    //
    // Repository Summary
    // ────────────────────────────────────
    // Language: TypeScript
    // Framework: React + Fastify
    // Packages: 42
    // Entry Points: apps/cli, packages/kernel
    // Risks: 3 large files, 2 TODO hotspots
    // ...
  }

  /** Render as JSON for API consumers */
  renderJson(summary: PresentedSummary): string { ... }

  /** Render as markdown for documentation */
  renderMarkdown(summary: PresentedSummary): string { ... }

  private async synthesizeNarrative(profile: RepositoryProfile): Promise<...> {
    // Build structured prompt from RepositoryProfile
    // Call reasoningRuntime.reason() with 'deep-analysis' strategy
    // Parse result into { purpose, suggestedStartingPoints, keyObservations }
    // Return null if AI unavailable (degrade gracefully)
  }

  private extractFacts(profile: RepositoryProfile): PresentedSummary['facts'] { ... }
}
```

The presenter works even when the AI call fails (narrative is `null`, facts are always available). This matches the hybrid model: machine facts are the source of truth; AI enhances them.

---

#### 8. WorkspaceSession

**File**: `packages/workspace/src/workspace-session.ts`

```typescript
export class WorkspaceSession {
  readonly rootPath: string;
  readonly workspaceDir: string;
  readonly fingerprint: RepositoryFingerprint;
  readonly profile: RepositoryProfile;
  readonly manifest: WorkspaceManifestData;
  readonly knowledge: KnowledgeEngine;
  readonly memory: MemoryRuntime;
  readonly conversation: ConversationService;

  constructor(opts: { ... }) { ... }

  /** Search indexed knowledge within this workspace */
  async search(query: string): Promise<SearchResult[]> { ... }

  /** Get workspace-scoped memories for context assembly */
  async getContextMemories(): Promise<Memory[]> { ... }

  /** Persist current state to .vestara/ */
  async persist(): Promise<void> { ... }
}
```

---

#### 9. CLI: `open` Command

**File**: `apps/cli/src/commands/open.ts` (new)

The `open` command handler. Imports only `WorkspaceRuntime` — not knowledge, memory, or reasoning directly.

```typescript
import { WorkspaceRuntime, WorkspaceStatus } from '@vestara/workspace';

export async function runOpen(path: string): Promise<void> {
  const runtime = new WorkspaceRuntime({ /* logger, eventBus */ });

  // Render progressive status
  runtime.on('workspace:discovered', () => renderStep('Repository detected'));
  runtime.on('workspace:fingerprinted', (e) => renderStep('Repository identified', e.repo));
  runtime.on('workspace:analyzed', (e) => renderStep('Project analyzed', e.language));
  runtime.on('workspace:manifest.created', () => renderStep('Workspace created'));
  runtime.on('workspace:indexed', (e) => renderStep('Knowledge indexed', `${e.documents} documents`));
  runtime.on('workspace:presented', () => renderStep('Repository understood'));

  const session = await runtime.open(path);
  renderSummary(session);
  await startWorkspaceRepl(runtime, session);
}
```

**File**: `apps/cli/src/repl-workspace.ts` (new)

Workspace-aware REPL:
- Prompt: `{repo-name} > ` instead of `> `
- All existing REPL commands (health, status, history, exit, help)
- Generic questions routed through workspace conversation service (which has access to indexed knowledge + workspace memory)
- Shows the repository summary on first display

**File**: `apps/cli/src/index.ts` — modifications

```typescript
if (args[0] === 'open') {
  const path = args[1] ?? '.';
  const { runOpen } = await import('./commands/open');
  await runOpen(path);
  return;
}
```

**File**: `apps/cli/package.json` — add `"@vestara/workspace": "workspace:*"` to dependencies

---

#### 10. Build Order

**File**: `build-order.sh` — expanded to reflect the full dependency graph:

```bash
for pkg in \
  shared \
  configuration \
  logger \
  metrics \
  event-bus \
  service-registry \
  health \
  permission \
  stream \
  provider-runtime \
  providers/opencode \
  context \
  memory \
  cognitive \
  knowledge \
  reasoning \
  action \
  state-runtime \
  conversation \
  tools/filesystem \
  workspace \
  kernel \
  cli; do
  echo "  Building $pkg..."
  npx tsc -p "packages/$pkg" --outDir "packages/$pkg/dist" 2>&1 | grep -v "TS6305" | grep -v "TS7016" | grep -v "TS2307" || true
done
```

This is now executable documentation of the package layering. If someone later introduces a circular dependency (e.g., `knowledge → workspace → knowledge`), the build script exposes it immediately.

Note: `tools/filesystem` uses a different path pattern (`packages/tools/filesystem`). The build script handles this with the src directory mapping in the `tsc -p` flag.

---

### Phase 1 Verification

#### Automated:
- [ ] `bash build-order.sh` completes without errors
- [ ] TypeScript compiles with `strict: true` (no `any`)
- [ ] `pnpm vestara open .` runs the pipeline without crashing
- [ ] `.vestara/workspace.json` is created with valid JSON
- [ ] WorkspaceSession exposes knowledge, memory, conversation engines
- [ ] RepositoryProfile contains non-empty `entryPoints` and `risks` arrays
- [ ] IndexReport returns non-zero documents and chunks

#### Manual:
- [ ] `vestara open .` produces progressive status output (7+ status lines)
- [ ] Prompt changes to `vestara-ai-core > ` after workspace opens
- [ ] User can type a question and get a response
- [ ] User can type `exit`, and `.vestara/` directory persists
- [ ] Re-opening reuses cached state (fast)
- [ ] `vestara open /nonexistent` produces a clear error
- [ ] `vestara open` (no path) defaults to `.`

---

## Phase 2: Performance — "Vestara understands it quickly"

### Success Criteria

```
□ Workspace usable before indexing completes
□ Incremental indexing (only changed files reindexed)
□ File watching (inotify/fsevents)
□ Progress reporting during indexing
□ Resume interrupted indexing
```

### Changes

#### 1. Asynchronous Pipeline

Split `WorkspaceRuntime.open()` into fast path (sync) and slow path (async):

```
open(path):
  # Fast path (sync)
  1. Discover (file walk)
  2. Fingerprint (git identity + content hash)
  3. if manifest is fresh → load cached profile, skip to step 6
  4. Analyze (deterministic, <50ms)
  5. Manifest init (.vestara/)
  6. Create WorkspaceSession
  7. emit 'workspace:ready'
  → CLI switches to REPL immediately

  # Slow path (async — continues in background)
  8. Index files
     - emit 'workspace:indexing.progress'
     - CLI shows: ████░░░░ 42%
  9. Present (AI call)
     - emit 'workspace:presented' when done
```

#### 2. Incremental Reindexing

On re-open, compare `workspace.fingerprint.repositoryHash` against current:
- Files changed → compare mtimes, reindex only changed files
- Files deleted → remove from knowledge storage
- No changes → skip indexing entirely (instant open, <100ms)

#### 3. File Watching

```typescript
// Phase 2 addition to WorkspaceRuntime
startWatching(): void {
  const chokidar = require('chokidar');  // or fs.watch
  this.watcher = chokidar.watch(this.session.rootPath, {
    ignored: ['node_modules', '.git', 'dist'],
  });
  this.watcher.on('change', (path) => this.session.knowledge.indexFile(path));
  this.watcher.on('unlink', (path) => this.session.knowledge.removeFile(path));
}
```

#### 4. CLI Progress Rendering

**File**: `apps/cli/src/render/progress.ts` (new)

```typescript
export function renderProgressBar(percent: number, label: string): string {
  const width = 20;
  const filled = Math.round(width * percent / 100);
  const empty = width - filled;
  return `${label} ${'█'.repeat(filled)}${'░'.repeat(empty)} ${percent}%`;
}
```

---

#### Automated Verification:
- [ ] `workspace:ready` fires before indexing completes
- [ ] REPL is interactive while indexing runs in background
- [ ] Re-opening an unchanged repo skips reindexing (verify via log)
- [ ] Modified files are detected and reindexed
- [ ] Deleted files are removed from the index

#### Manual Verification:
- [ ] Large repo opens in <1 second before indexing completes
- [ ] Progress bar visible during background indexing
- [ ] User can ask questions immediately and get responses
- [ ] File saved externally → index updates automatically (file watching)

---

## Phase 3: Intelligence — "Vestara understands it deeply"

### Success Criteria

```
□ Architecture pattern detection (layered, event-driven, plugin, etc.)
□ Full module dependency graph resolution
□ Circular dependency detection with path reporting
□ TODO/FIXME/HACK/XXX hotspot mapping with severity scoring
□ Unused dependency detection (declared but never imported)
□ Repository health score (composite of all risk factors)
□ Language-specific analysis (Go modules, Python imports, Rust crates)
```

### Changes

#### 1. Architecture Pattern Detection

Extend `RepositoryIntelligence` to classify the codebase's architectural style:

| Pattern | Detection heuristic |
|---------|-------------------|
| Layered | `controller/`, `service/`, `repository/`, `middleware/` directory conventions |
| Event-driven | Event bus usage: `emit()`, `on()`, `subscribe()`, `publish()` calls |
| Plugin | Plugin directories, `loadPlugin()`, `registerPlugin()`, `use()` patterns |
| Microservices | Multiple `Dockerfile`s, separate deploy configs, service directories |
| Hexagonal | `port/`, `adapter/`, `driver/` directory conventions |

Each pattern gets a confidence score based on evidence count.

#### 2. Full Dependency Graph

- Parse `import`/`require`/`from` statements from TypeScript/JavaScript files
- Parse `use` statements from Rust, `import` from Python, etc.
- Build a directed graph of module dependencies
- Support workspace-aware resolution (local packages resolve correctly)

#### 3. Circular Dependency Detection

- Run Tarjan's or DFS-based cycle detection on the dependency graph
- Report each cycle with the full path: `A → B → C → A`
- Assign severity based on cycle length and number of affected modules

#### 4. TODO/FIXME Hotspot Mapping

- Walk all indexed files for `TODO`, `FIXME`, `HACK`, `XXX`, `WORKAROUND`, `BUG`
- Cluster by directory
- Score files by density (TODOs per 100 lines)
- Surface top-10 hotspots in risks

#### 5. Unused Package Detection

- Collect all `import`/`require` targets from indexed files
- Compare against `dependencies` in all `package.json` files
- Flag packages declared but never imported
- Flag workspace packages with no consumers

#### 6. Repository Health Score

```typescript
interface HealthScore {
  overall: number;           // 0.0 — 10.0
  categories: {
    testCoverage: number;    // % of packages with tests
    techDebt: number;        // inverse of TODO density
    complexity: number;      // inverse of large-file ratio
    dependencyHealth: number; // 10 - (circular deps * 2)
    completeness: number;    // % of packages with descriptions
  };
}
```

---

#### Automated Verification:
- [ ] Architecture patterns correctly classify test fixtures with known layouts
- [ ] Circular dependency detection finds known cycles
- [ ] TODO hotspot counts match raw grep results
- [ ] Unused dependency detection matches manual audit
- [ ] Health score is deterministic (same repo → same score)

#### Manual Verification:
- [ ] `vestara open` on `vestara-ai-core/` detects its own architecture patterns
- [ ] Risk report surfaces genuine issues (not false positives)
- [ ] Health score matches developer intuition about codebase quality
- [ ] Entry point detection correctly identifies CLI, library, app entry points
- [ ] Large monorepos produce accurate package maps

---

## Testing Strategy

### Unit Tests

| Module | What to Test |
|--------|-------------|
| `RepositoryIntelligence` | Entry point detection on crafted dir structures; risk detection on files with TODOs/large files; package map from fake package.jsons |
| `RepositoryFingerprint` | Git identity resolution; non-git fallback; content hash consistency |
| `WorkspaceManifest` | Save/load round-trip; stale detection; error handling for corrupt manifests |
| `RepositoryPresenter` | Prompt construction from RepositoryProfile; CLI rendering format; graceful degradation when AI unavailable |
| `WorkspaceRuntime` | Pipeline ordering; status transitions; event emission; error propagation |

### Integration Tests

| Scenario | What to Verify |
|----------|---------------|
| `open` on `vestara-ai-core/` | Full pipeline runs; manifest created; session returned; engines initialized |
| `open` on a Go monorepo | Language detection; Go module parsing; correct framework |
| `open` on empty directory | Graceful error; no crash |
| `open` on nonexistent path | Error propagation; non-zero exit code |
| REPL workflow | Workspace commands work; context persists across messages |

### Golden Path Extension

The existing golden path (`pnpm vestara demo golden-path`) should gain a new test:

```
1. Boot Runtime
2. Open repository (vestara-ai-core/)
3. Verify pipeline status output
4. Ask a question in workspace context
5. Verify response references repository knowledge
6. Exit and verify .vestara/ persistence
7. Re-open and verify cache reuse
```

---

## Performance Considerations

| Operation | Phase 1 Target | Notes |
|-----------|---------------|-------|
| Repository discovery | <100ms per 10k files | Sync readdir + stat; filesystem-bound |
| Fingerprinting | <50ms | Git commands + content hash |
| Analysis (Phase 1) | <50ms | Deterministic, no I/O beyond file headers |
| Manifest creation | <10ms | Single JSON write |
| Knowledge index | ~30ms per 100 files | Sync read + parse + chunk + write |
| AI presentation | 2-10s | Provider API call; network-bound |
| **Total cold open** | ~2-15s | Dominated by index + AI |
| **Total warm open** | <100ms | All cached; no index, no AI |
| Memory | ~50MB baseline + ~1KB per 100 docs | SQLite WASM heap |

---

## Migration Notes

- Existing `vestara-state.db` and `vestara-golden-path.db` files are unaffected. The workspace creates its own SQLite databases inside `.vestara/knowledge/` and `.vestara/memory/`.
- No changes to existing package interfaces — all additions are additive.
- The `DefaultRepositoryAnalyzer` in `@vestara/knowledge` remains unchanged. `RepositoryIntelligence` in `@vestara/workspace` delegates to it for basic detection, then layers additional analysis on top.
- The existing `tests/integration/` and `tests/performance/` directories remain empty as before. New tests go alongside their modules in `__tests__/` directories.

---

## References

- Pipeline vision: user design brief (July 2026) — Epic 001: Repository Comprehension
- File walker + ignore logic: `packages/knowledge/src/index.ts` L381-421
- Document parser: `packages/knowledge/src/index.ts` L70-121
- Repository analyzer (basic): `packages/knowledge/src/index.ts` L299-348
- Reasoning runtime: `packages/reasoning/src/index.ts` L278-389
- Strategy selector: `packages/reasoning/src/index.ts` L230-258
- Memory runtime: `packages/memory/src/index.ts` L165-429
- CLI entry point: `apps/cli/src/index.ts`
- Build order: `build-order.sh`
- Architecture freeze: `vestara-runtime/architecture-v1.0/ARCHITECTURE-FREEZE.md`
- Brain architecture: `vestara-blueprint/05-ai-core/BRAIN-ARCHITECTURE.md`
- Cognitive architecture: `vestara-blueprint/05-ai-core/COGNITIVE-ARCHITECTURE.md`
