# Vestara AI OS — GitHub Copilot Instructions

## Project
AI-native engineering platform with CLI-first architecture.
TypeScript (strict), Node.js 24, sql.js (SQLite WASM), React 19, Vite 6, Tailwind CSS 4.

## Critical Context (ALWAYS APPLY)
- Read `vestara-blueprint/00-governance/01-ai-constitution.md` before coding
- Read `vestara-blueprint/00-governance/02-engineering-rules.md` for non-negotiable rules
- Read `vestara-ai-core/docs/VSDE/VSDE-001-lifecycle.md` for the engineering lifecycle
- Read `vestara-ai-core/docs/PRODUCT-PRINCIPLES.md` for product governance

## Where the Code Is
All active implementation is in `vestara-ai-core/`. The blueprint repository contains frozen architecture documents.

## Architecture
- `WorkspaceRuntime` is the orchestration boundary — CLI imports it, not knowledge/memory/reasoning directly
- `RepositoryWorkspace` is the canonical domain object enriched by every pipeline stage
- 22 packages building in dependency order via `build-order.sh`
- OpenCode is the sole AI provider (default: `https://opencode.ai/zen/v1`)
- sql.js (WASM) for SQLite — NOT better-sqlite3
- AI is optional — all pipelines degrade gracefully with deterministic output

## VSDE Rules
- Specifications are the primary engineering artifact
- Every capability requires a Capability Specification Package (CSP) before implementation
- Documentation quality is a build gate
- AI implements documented behavior — never invent behavior not in the CSP
- Capability maturity is tracked: specification, architecture, implementation, verification, documentation

## Key Commands (from vestara-ai-core/)
```bash
bash build-order.sh   # Build 22 packages in dependency order
pnpm vestara open .   # Open a repository
pnpm vestara doctor   # Health diagnostics
pnpm test             # 55 tests across 11 files
pnpm vestara          # Interactive REPL
```

## NEVER Do
- Modify frozen blueprint documents without ADR
- Add comments to code unless asked
- Assume better-sqlite3 is the database driver (use sql.js)
- Add emoji to files unless explicitly requested
- Expect CI to catch errors (no CI configured for code)
