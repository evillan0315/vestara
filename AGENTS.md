# Vestara agent instructions

## Repository boundaries

- `vestara-ai-core/` is the only implementation repository. It is a separate git repository nested here; run code commands and inspect its git status from that directory.
- `vestara-blueprint/` is a separate, frozen design-document repository. Treat it as context, not executable truth; verify claims against `vestara-ai-core/`.
- The root repository contains configuration, instructions, assets, and design artifacts; it has no root `package.json` or root application.
- Do not edit `.vestara/` runtime state or `.env`; `.env` contains live provider credentials and must never be logged or committed.

## Code map

- `apps/cli/src/index.ts`: `vestara` CLI and REPL; `pnpm vestara console` launches the interactive TUI.
- `apps/api/src/index.ts`: API process; `apps/api/src/server.ts` is a raw Node `http` + `ws` gateway and delegates routes to `apps/api/src/routes/`.
- `apps/workspace/src/main.tsx`: React 19/Vite UI entrypoint. The UI dev server is port `5173` and proxies `/api` and `/ws` to API port `3001`.
- `packages/workspace/`: integration hub. Consumers should use `WorkspaceRuntime.open()` rather than importing workspace-analysis concerns directly.
- `packages/kernel/`: boot, service lifecycle, health, scheduling, recovery, workers, and provider orchestration; it coordinates services rather than implementing their domain logic.
- `packages/runtime/`: generic runtime lifecycle implemented with `@vestara/state-machine`; `packages/shared/` owns the `VestaraService` contract.
- `packages/agent-harness/`: the durable single-turn agent loop (model → tool → approval → verification → outcome). It is the execution path; `packages/workspace/src/agent-runtime.ts` is a thin adapter that delegates `run()` to it (durable thread + ExecutionSession).
- `packages/engineering-event-store/`: temporal truth for engineering events; the harness projects `harness.*` events into it through `apps/api/src/bridges/harness-engineering-event-bridge.ts`. ThreadRuntime remains the authoritative execution history.

## Agent Type System

Agents are typed as either `workspace` (local) or `registry` (marketplace-installed):

- **`AgentType`** = `'workspace' | 'registry'` (defined in `packages/workspace/src/types.ts:559`)
- **`AgentDefinition.agentType`** — required field; defaults to `'workspace'` for backward compatibility
- **Storage**: `agent_type TEXT DEFAULT 'workspace'` column in agents table (`packages/workspace/src/agent-storage.ts`)
- **UI**: Agent Registry Modal (`apps/workspace/src/pages/Agents/AgentRegistryModal.tsx`) provides radio selector for type
  - Workspace Agent: uses locally configured provider/model
  - Registry Agent: uses marketplace package source/version
- **API**: POST/PUT `/api/agents` handlers read `body.agentType` with workspace default

Built-in agents (architect, developer, verifier, etc.) are workspace agents. Registry agents are installed from the marketplace and will have version tracking and update notifications in the future.

## Commands

Run these from `vestara-ai-core/`:

```bash
pnpm install
pnpm build                    # generates workspace references, then tsc -b
bash build-order.sh           # compatibility entrypoint for pnpm build
pnpm lint                     # Biome check --write (mutates files)
pnpm lint:check               # read-only Biome check
pnpm format                   # Biome format --write
pnpm test                     # Vitest; build first
pnpm --filter @vestara/conversation test
pnpm test -- path/to/file.test.ts
pnpm check:source-artifacts    # fails if generated .js/.d.ts/.js.map appear under src/ or __tests__/
pnpm dev                      # API + Workspace UI
pnpm dev:api                 # API only; compiled output required
pnpm dev:ui                  # Workspace UI only
pnpm vestara doctor
```

- The normal verification sequence is `pnpm lint && pnpm build && pnpm test`. There is no `pnpm typecheck` script.
- Runtime CLI commands execute `dist` JavaScript, so run `pnpm build` before `pnpm vestara`, `pnpm dev:api`, diagnostics, benchmarks, or documentation commands.
- `pnpm --filter <workspace-package> test` selects a package. `pnpm test -- <path>` passes a positional test-file filter to Vitest; do not use a Vitest `--filter` flag for package selection.
- `scripts/pre-commit.sh` runs staged-file Biome checks and the full test suite.

## Tests and tooling

- Vitest 4 is used. Tests normally live in each package/app's `__tests__/` directory as `*.test.ts`; Workspace UI tests are colocated under `apps/workspace/`.
- Tests resolve `@vestara/*` packages from built `dist/` output, so a successful build is a prerequisite.
- Database tests use `sql.js` WASM with in-memory SQLite; no external database service is required.
- Workspace visual tests use Playwright. `pnpm screenshots` compares baselines; only use `pnpm screenshots:update` when intentionally reviewing and replacing approved baselines.
- Biome is the only formatter/linter. The canonical JavaScript style is single quotes, trailing commas, and semicolons; workspace UI has its own Vite/Biome configuration.
- Most packages emit CommonJS with TypeScript `module: nodenext`; `apps/workspace/` and `apps/console/` are ESM. Check the package manifest before changing module syntax or imports.

## Runtime environment

- API defaults: `VESTARA_API_PORT=3001`, `VESTARA_REPO=process.cwd()`. `VESTARA_API_URL` overrides the CLI/TUI endpoint.
- Harness execution engine: `POST /api/agents/:agentId/runs` plus `GET|POST /api/agent-threads/:threadId[/items|/events|/approvals|/steer|/cancel|/resume]` and `POST /api/agent-threads/:threadId/approvals/:approvalId/resolve`. `AgentRuntime.run()` delegates to the harness; `harness.*` events project into the event store.
- API startup searches upward for `.vestara/workspace.json` unless `VESTARA_REPO` is set; this matters when launching compiled output from a different working directory.
- Use source `.ts`/`.tsx` files, not stale ignored `.js`, `.d.ts`, or source maps left by prior TypeScript builds. Generated output belongs in `dist/`.

When documentation conflicts with manifests, scripts, or source, trust the executable sources and update this file only with facts that can be verified there.
