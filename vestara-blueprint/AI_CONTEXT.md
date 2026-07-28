---
title: "AI Context — Current Project State (Dynamic)"
volume: "00-governance"
book: "Book 1: Vision & Business"
version: "2.0.0"
status: "approved"
owner: "@engineering-manager"
last-reviewed: "2026-07-24"
next-review: "2026-08-24"
tags: ["context", "state", "dynamic", "current-sprint", "blockers"]
---

# AI Context
## Current Project State — Updated Continuously

> **Every AI reads this FIRST. This document is updated at the end of every session/task.**

---

## 📍 CURRENT STATE

**Version**: v2.7 — Outcome Verification
**Era**: Product Era (Engineering Lifecycle Complete)
**Status**: ✅ All 23 capabilities delivered across v0.1 through v2.7

---

## Repository Structure

The implementation lives in `vestara-ai-core/` (not at the root):

```
vestara-ai-core/
  packages/
    shared/           — Core interfaces (zero-dep contract package)
    kernel/           — DefaultKernel, 10-step boot sequence
    event-bus/        — In-process pub/sub with pattern matching
    configuration/    — Config loader, FileConfigSource
    logger/           — Structured JSON logger
    metrics/          — Counters, gauges, histograms
    service-registry/ — Topological sort (Kahn's algorithm)
    health/           — Aggregate health checks
    permission/       — PermissionEngine, role-based access
    stream/           — StreamProcessor, canonical chunk types
    provider-runtime/ — ProviderManager, load/unload/health
    providers/opencode/ — OpenCode provider (SSE streaming)
    context/          — ContextAssembler
    memory/           — 4-layer memory runtime
    cognitive/        — 5-stage cognitive pipeline
    knowledge/        — FTS knowledge engine
    reasoning/        — 8-strategy reasoning runtime
    action/           — ActionRuntime, tool execution
    state-runtime/    — SQLite persistence (sql.js WASM)
    conversation/     — Conversation service
    tools/filesystem/ — Read/write with path traversal protection
    tools/shell/      — STUB
    tools/memory/     — STUB
    tools/knowledge/  — STUB
    tools/project/    — STUB
    workspace/        — WorkspaceRuntime + all services
  apps/
    cli/              — @vestara/cli (REPL + doctor + open)
    workspace/        — React 19 + Vite web UI
  docs/
    VSDE/             — Specification-Driven Engineering standard
    capabilities/     — 23 Capability Specification Packages
    artifacts/        — 7 independent artifact contracts
    PRODUCT-PRINCIPLES.md
    MILESTONES.md
    IMPLEMENTATION_STATUS.md
    DECISIONS.md
    CONTRACT-CATALOG.md
```

---

## Capability Ladder (Complete)

```
vestara open .         → RepositoryWorkspace    (v0.3.0)
vestara explain        → Explanation            (v0.3.3)
vestara plan           → Plan                   (v0.4)
vestara predict        → ImpactAssessment       (v2.4)
vestara recommend      → Decision               (v2.5)
vestara implement      → ChangeSet              (v0.5/v2.6)
vestara verify         → VerificationReport     (v0.6/v2.7)
vestara collaborate    → CollaborationRecord    (v0.7)
```

---

## Engineering Methodology

**Vestara Specification-Driven Engineering (VSDE)** — See `docs/VSDE/`:
- Specifications are the primary artifact
- Every capability needs a CSP before implementation
- Documentation quality is a build gate
- AI implements documented behavior

---

## Key Architecture Decisions

| ADR | Decision | Status |
|-----|----------|--------|
| ADR-016 | Architecture Freeze v1.0 | Accepted |
| ADR-017 | WorkspaceRuntime as Orchestration Boundary | Accepted |
| ADR-018 | RepositoryWorkspace as Canonical Domain Object | Accepted |
| ADR-019 | Vestara Specification-Driven Engineering (VSDE) | Accepted |

---

## Tech Stack (Actual)

| Layer | Technology | Notes |
|-------|-----------|-------|
| Language | TypeScript 7.0 | strict: true, module: nodenext |
| Runtime | Node.js 24 | |
| Package Manager | pnpm 11 | Workspace protocol |
| Build | Sequential tsc (build-order.sh) | 22 packages in dependency order |
| Database | sql.js (SQLite WASM) | NOT better-sqlite3 |
| UI | React 19 + Vite 6 + Tailwind 4 | apps/workspace |
| Provider | OpenCode (opencode.ai/zen/v1) | Sole AI provider |
| Tests | Vitest 4 | 55 tests across 11 files |
| Architecture | Frozen v1.0 | ADR-016 |

---

## Quick Start

```bash
cd vestara-ai-core
pnpm install
bash build-order.sh
pnpm vestara open .    # Open any repository
pnpm vestara doctor     # Health diagnostics
pnpm test               # 55 tests
```

---

## Key Deviations from Original Blueprint

The original blueprint (v1.0.0) described a Fastify API server with a React dashboard and SQLite via better-sqlite3. The actual implementation built:

- A CLI-first engineering platform instead of an API server
- A `WorkspaceRuntime` orchestration layer instead of Fastify routes
- sql.js (WASM) instead of better-sqlite3
- 23 capability versions instead of the original 10-version roadmap
- A specification-driven engineering methodology (VSDE)

All architectural invariants from the frozen blueprint are preserved — the implementation path diverged, not the contracts.

---

## Current Priorities

1. Maintain and extend existing capabilities
2. Deepen enrichment of existing artifacts (health, prediction, accuracy)
3. Connect Workspace UI to the engine (currently static shell)
4. Expand test coverage beyond 55 tests
