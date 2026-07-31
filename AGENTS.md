# AGENTS.md

High-signal, verified facts for agents working on Vestara.
If docs conflict with code or scripts, trust the code.

---

## Repo layout

| Path | Role |
|------|------|
| `vestara-ai-core/` | **Sole code home** — pnpm monorepo, all source (gitlink to `evillan0315/vestara-ai-core`) |
| `vestara-blueprint/` | Frozen design docs — stale claims, verify before trusting (gitlink to `evillan0315/vestara-blueprint`) |
| `vestara-foundation/`, `vestara-runtime/`, `vestara-specifications/`, `vestara-labs/`, `vestara-reference/` | Architecture/strategy/research docs (read only if asked). Each is published as a standalone public repo under `github.com/evillan0315/` |
| `.vestara/` | Runtime persistence (at repo root + `vestara-ai-core/` + `apps/api/`) |
| `.opencode/` | OpenCode config: agents, skills, plugin deps |
| Everything else at root | Config, instruction files, assets — **no `package.json`** |

## Where to work

- Implementation/fix work goes in **`vestara-ai-core/`**.
- The top-level repo is config, instruction files, and design artifacts.
- Read `vestara-blueprint/`, `vestara-foundation/`, etc. only when the task explicitly asks for strategy, governance, or architecture context.

## Entrypoints

| Entry | Path |
|-------|------|
| CLI / REPL | `apps/cli/src/index.ts` |
| API gateway (http+ws) | `apps/api/src/index.ts` |
| UI (React 19 + Vite) | `apps/workspace/src/main.tsx` |
| Onboarding Lab (dev test rig) | `apps/onboarding-lab/src/index.ts` |
| Core runtime base class | `packages/runtime/src/index.ts` |
| Shared lifecycle contract | `packages/shared/src/index.ts` |
| Kernel (boot orchestrator) | `packages/kernel/src/index.ts` |

## Key architectural insight

**Coordinator-composes-specialists** pattern appears at 4 layers (all verified in source):

| Layer | Coordinator | Specialists |
|-------|-------------|-------------|
| Execution | `RuntimeGroup` | `Runtime` instances |
| Analysis | `WorkspaceRuntime` | Pipeline stages |
| Knowledge | `UnderstandingAssembler` | `UnderstandingProducer` (8 producers) |
| Evaluation | `EvaluationHarness` | Corpus entries |

Rule: coordinators compose, specialists decide, every concern has one owner.

The **kernel** (`@vestara/kernel`) orchestrates the boot sequence — job manager, recovery manager, task scheduler, worker manager — and wires the foundation services (configuration, logger, metrics, event-bus, health, scheduler).

Every `Runtime` instance uses `@vestara/state-machine` for lifecycle state transitions (`created → initializing → running → stopped → ...`). The state machine is zero-dependency and generic.

`@vestara/workspace` is the integration hub — every consumer (CLI, API, UI) imports this package and calls `WorkspaceRuntime.open()`. It exports 80+ services and is the only consumer of `@vestara/knowledge`; `@vestara/understanding` is also used by `@vestara/evaluation`. Workspace also depends on memory (but so does `@vestara/cognitive`). Nothing depends on `@vestara/reasoning`.

`@vestara/events` defines the typed event system — `WorkspaceEvent` with 30+ event types across 10 categories (`conversation`, `workspace`, `planning`, `implementation`, `verification`, `collaboration`, `system`, `agent`, `memory`, `profile`). Used by `EventBus` for all inter-service communication.

**WorkspaceRuntime pipeline stages** (verified in `workspace-runtime.ts`): `Idle → Discovering → Fingerprinting → Analyzing → Indexing → Presenting → Ready`. Each stage enriches the `RepositoryWorkspace` domain object.

## Commands (run from `vestara-ai-core/`)

```bash
pnpm install                          # Install deps
bash build-order.sh                   # Canonical sequential build
pnpm lint                             # biome check --write
pnpm format                           # biome format --write
pnpm test                             # vitest run (requires prior build)
pnpm --filter @vestara/workspace-ui dev  # UI dev server on port 5173 (proxies /api → :3001, /ws → :3001)
pnpm dev:api                            # Start API server only (requires prior build)
pnpm dev                                # API + UI concurrently
pnpm vestara                          # node apps/cli/dist/index.js (REPL)
pnpm vestara open .                   # Open workspace, changes REPL prompt to {repo-name} >
pnpm doctor                           # Runtime health check
pnpm vestara validate <path>          # CAP-001 Workspace Orientation
pnpm conversation-audit               # Conversation feature health scan (src: packages/conversation-runtime/src/audit/)
pnpm benchmark                        # Pipeline performance benchmarks
pnpm benchmark-index                  # Indexing performance benchmarks
pnpm generate-docs                    # TypeDoc API docs generation
pnpm ensure-docs                      # Docs integrity check
pnpm milestone-status                 # Milestone progress report
pnpm vestara architecture list        # List Architecture Knowledge Graph ADRs
pnpm vestara architecture show <id>   # Show a single ADR with metadata
pnpm vestara architecture impact <id> # Impact analysis: what depends on an ADR
pnpm vestara blueprint verify         # AKG integrity check (broken deps, cycles, duplicates)
pnpm vestara ops demo                 # Engineering telemetry demo (feed + status)
pnpm vestara ops status               # Live agent statuses
```

**All runtime commands (`pnpm vestara`, `doctor`, `conversation-audit`, `benchmark*`) run compiled JS and require `bash build-order.sh` first.**

### Single package test
```bash
pnpm --filter @vestara/conversation test
```
`pnpm test -- <path>` filters vitest by test file path (positional filters). There is no vitest `--filter` flag. Use `pnpm --filter <package> test` for package filtering.

### Verification loop
```bash
pnpm lint && bash build-order.sh && pnpm test
```
No `pnpm typecheck` script exists.

## Build quirks

- **`build-order.sh`** is the canonical build. Builds apps first (api, cli, onboarding-lab via `npx tsc -p`), then 47 packages in 7 dependency groups. Filters 3 known TS errors via `grep -v`: `TS6305`, `TS7016`, `TS2307`.
- Does NOT build every package. Skipped (each builds via its own `tsc`): `settings-framework`, `providers/opencode`, `job`, `scheduler`, `worker`, and `tools/{knowledge,memory,project,shell}`. Note `tools/filesystem` **is** built (group 5).
- `pnpm build` runs `tsc -b` from root — the root `tsconfig.json` has no project references, so this only compiles root-level files. Not useful.
- `pnpm-lock.yaml` lives in `vestara-ai-core/`, not at repo root.
- pnpm workspace includes `packages/*`, `packages/providers/*`, `packages/tools/*`, and `apps/*`.
- Shell scripts live in `scripts/`: `benchmark.sh`, `benchmark-index.sh`, `ensure-docs.sh`, `generate-docs.sh`, `milestone-status.sh`.
- `vestara-ai-core/.gitignore` (475 lines) is comprehensive: ignores `*.js` / `*.d.ts` next to `.ts` sources (stale pre- `build-order.sh` artifacts), `dist/`, `.vestara/`, `*.db`. Workspace UI has its own `.gitignore`.
- **Stale compiled artifacts** (`*.js`, `*.d.ts`, `*.js.map`) may linger next to `.ts` sources from prior `tsc` runs (gitignored via `vestara-ai-core/.gitignore`). Always read `.ts` — the `.js` may be stale. Applies to `src/` and `__tests__/` alike.
- `apps/api/src/server.ts` is a compact raw Node.js `http` + `ws` request handler — no framework. Routes live in `apps/api/src/routes/` (16 route handler files; `index.ts`/`types.ts` are shared helpers), not in the server file.
- `apps/cli/src/index.ts` and `apps/cli/src/repl-workspace.ts` are the CLI entry points. The `commands/` subdirectory has ~20 sub-commands including `config`, `open`, `provider`, `doctor`, `validate`, `task`, `benchmark`, `demo`, `session`, etc.
- Root `vite.config.ts` and `apps/workspace/vite.config.ts` differ: root targets `localhost:3000` with vitest globals/jsdom setup; workspace targets `localhost:3001` with dev proxy to the API. Don't confuse them.

## Tests

- **Vitest 4**, real SQLite (sql.js WASM, in-memory — no setup needed)
- Tests in `__tests__/**/*.test.ts` per package
- **vitest aliases `@vestara/*` to `packages/*/dist/`** — tests resolve from pre-built dist, so `bash build-order.sh` must run first
- 15s default timeout
- Tests use explicit vitest imports (`import { describe, expect, it } from 'vitest'`), not globals.
- Helper factories defined locally per test file (e.g. `createRuntime()`, `makeGroup()`). No shared test utilities. Exception: `packages/evaluation/fixtures/` ships fixture repos (`nestjs-monorepo`, `vite-react-basic`, `empty-project`) used by evaluation tests.
- `__tests__/` directories also have stale `*.js`/`*.d.ts`/`*.js.map` from prior compilations. Always read the `.ts` version.

## Module system

- **Workspace UI** (`apps/workspace/`) is ESM (`"type": "module"`). Everything else is CJS. Don't mix import styles.
- **`settings-framework`** (`packages/settings-framework/`) is also ESM (`"type": "module"`) — the only other ESM package. It is also the only non-private package (no `"private": true`) and the only one with Zod as a peer dependency.
- Workspace UI tsconfig differs from the rest: `module: "ESNext"`, `moduleResolution: "bundler"`, `jsx: "react-jsx"`, `noEmit: true`, `lib: ["ES2022", "DOM", "DOM.Iterable"]`. The rest of the project uses `module: "nodenext"` with CJS emits.

## Toolchain (verified in config and code)

| Aspect | Reality |
|--------|---------|
| Lint + format | **Biome 2.x** only (no ESLint, no Prettier) |
| TypeScript | ES2022 target, `nodenext` module, CJS emits, `strict: true` + `skipLibCheck: true` |
| API server | **Node.js `http` + `ws`** — NOT Fastify. No `VestaraApp` type. |
| Database | **`sql.js` WASM** in app code |
| Validation | Zod used **locally** per package. No `@vestara/validation` package. |
| UI workspace | React 19 + Vite 6 + MUI 6 + Tailwind 4. Separate `biome.json`. Built via `tsc -b && vite build`. |
| Default provider | **OpenCode** (`@vestara/provider-opencode`) — works without API keys. Provider lifecycle managed by `@vestara/provider-runtime` |
| CI pipeline | `pnpm install --frozen-lockfile` → `bash build-order.sh` → `pnpm test` → benchmarks. **No lint or typecheck in CI.** |
| API docs | TypeDoc configured at root, 25 entrypoint packages listed in `typedoc.json` |

## Environment

| Variable | Default | Used by |
|----------|---------|---------|
| `VESTARA_API_PORT` | `3001` | API server (http + ws) |
| `VESTARA_REPO` | `process.cwd()` | API workspace path |

- `.vestara/` persists at repo root, `vestara-ai-core/`, and `apps/api/`. Contents vary by location: core + api have `prefs.db` and `workspace.json`; root has `metrics/` instead.
- `vestara-ai-core/.vestara/workspace.json` contains the full machine-readable dependency graph (nodes + edges), package inventory, entry points, and risk analysis.
- `vestara-ai-core/` is an **independent git repo** with its own origin (`vestara-ai-core.git`), not a submodule.
- `.env` is gitignored. **Never commit, log, or reference its values** — it contains live API keys for ~7 AI providers.

## Coding conventions (verified)

- `@vestara/types` for domain types
- `VestaraService` lifecycle interface from `@vestara/shared`: `initialize() → start() → stop() → health() → dispose()`
- Parameterized SQL: `db.prepare('...').run(params)` (sql.js style)
- sql.js init pattern: `const initSqlJs = (await import('sql.js')).default; const SQL = await initSqlJs();` — always async, needs `declare module 'sql.js'` in a local `sql.d.ts`
- `EventBus` for inter-service communication
- Services live one-per-directory in `packages/*` — each is self-contained with its own `src/`, `__tests__/`, `tsconfig.json`, `package.json`
- **Biome disables** `noExplicitAny`, `noBannedTypes`, `noStaticOnlyClass`, `noNonNullAssertion`, `noImplicitAnyLet`, `noAssignInExpressions`, `useIterableCallbackReturn`, `noDuplicateElseIf`, and `useBlockStatements`
- `console.log`/`warn`/`error` used in production code
- CLI uses raw ANSI escape codes (`\x1b[`), no chalk/colors library.
- CJS packages require `.js` extension in local imports (nodenext module resolution). Dropping it causes TS errors.
- Biome JS style: single quotes, trailing commas, semicolons always

## OpenCode config

- `opencode.json` references `AGENTS.md`, `vestara-ai-core/README.md`, `vestara-ai-core/daily-engineering-planner-prompt.md`, `vestara-blueprint/AGENTS.md`, and `vestara-blueprint/VESTARA_CONSTITUTION.md` as instructions.
- `vestara-blueprint/AGENTS.md` and `VESTARA_CONSTITUTION.md` contain stale claims (Fastify, `@vestara/validation`, `VestaraApp`) — actual code takes precedence.
- Agents in `.opencode/agents/`:
  - `vestara-context.md` — discovers repository state (primary)
  - `vestara-planner.md` — analyzes, prioritizes, recommends (primary)
  - `vestara-engineer.md` — implements approved tasks (primary)
  - `vestara-reviewer.md` — inspects implementations (subagent)
  - `vestara-verifier.md` — proves correctness via evidence (subagent)
- `opencode.json` also defines 8 agent profiles (`coder`, `developer`, `context`, `planner`, `engineer`, `reviewer`, `verifier`, `researcher`). Edit/write permission for coding profiles is scoped to `vestara-ai-core/{apps,packages,scripts}` — which matches "work goes in `vestara-ai-core/`".
- Skills in `.opencode/skills/`:
  - `vestara-lifecycle/SKILL.md` — daily development lifecycle (`/init`, `/morning`, `/work`, `/review`, `/verify`, `/evening`)
  - `testing-and-quality/SKILL.md` — test/lint/format workflow
  - `git-commit-push/SKILL.md` — git commit and push workflow

## Development Lifecycle

> Agents don't perform work. They participate in a software development lifecycle.

The Vestara development lifecycle formalizes daily engineering work into a repeatable workflow with specialized agents. See `vestara-ai-core/docs/foundation/02-development-lifecycle.md` for the full philosophy, or load the `vestara-lifecycle` skill for workflow details.

Participants:

| Agent | Role | Can Edit? | Can Plan? | Can Decide Scope? |
|-------|------|-----------|-----------|-------------------|
| Context | Discover | No | No | No |
| Planner | Recommend | No | Yes | No |
| Engineer | Implement | Yes | No | No |
| Reviewer | Inspect | No | No | No |
| Verifier | Prove | No | No | No |
| Human | Approve | Yes | Yes | Yes |
