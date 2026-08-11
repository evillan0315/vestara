# Vestara Repository Instructions

## Repository Boundaries

- The root repository is a coordination repository: it has no root application or `package.json`.
- `vestara-ai-core/` is a separate Git repository and the implementation target. Run its commands and inspect its Git status from that directory.
- `vestara-ai-core/` and `vestara-blueprint/` are tracked in the root repo as pinned gitlinks, not files. Root `git status` shows them as modified whenever their nested `HEAD` differs; syncing one requires a root "bump" commit (see `git log` for the established pattern).
- `vestara-blueprint/` is a separate, frozen design-document repository. Use it as context only; verify implementation claims in `vestara-ai-core/`.
- The remaining `vestara-*` directories (foundation, labs, reference, runtime, specifications) are design/reference content tracked as regular files in the root repo, even though they each contain a nested `.git`. They are not the implementation target; do not apply `vestara-ai-core` commands there.
- `react-dashboard/` is untracked WIP (its own Vite app, not part of the workspaces); treat as scratch until it is committed.
- Never edit `.vestara/` or `.vestara-worktrees/` runtime state. Never commit `.env` (gitignored) or `vestara.env` (not gitignored); both hold credentials and must not be logged.

## Core Layout

- `apps/api/src/index.ts` starts the raw Node HTTP + WebSocket gateway; route handlers live in `apps/api/src/routes/`.
- `apps/cli/src/index.ts` is the CLI/REPL entrypoint; `apps/workspace/src/main.tsx` is the React/Vite UI entrypoint.
- `apps/console/` is an empty stub (node_modules only). The keyboard-first "Console" is a CLI subcommand: `pnpm console` → `node apps/cli/dist/index.js console`.
- `packages/*`, `packages/providers/*`, `packages/tools/*`, and `apps/*` are pnpm workspaces. `packages/kernel/` coordinates lifecycle and providers; `packages/workspace/` is the integration hub.
- `apps/workspace/` (package `@vestara/workspace-ui`) and `packages/workspace/` (package `@vestara/workspace`) are distinct packages sharing the "workspace" name; the UI app and the integration hub are not the same.
- `packages/agent-harness/` owns the durable model/tool/approval/verification turn; `packages/workflow-orchestrator/` owns multi-agent project, plan, and task state.
- Custom OpenCode agents and skills live in `.opencode/` (`agents/` defines the vestara-context/planner/engineer/reviewer/verifier roles; `skills/` holds repo-local skills).

## Commands

Run these from `vestara-ai-core/`:

```bash
pnpm install
pnpm build                    # generates tsconfig.references.json, then runs tsc -b
pnpm lint:check               # read-only Biome check
pnpm lint                     # Biome check --write; mutates files
pnpm test
pnpm --filter @vestara/conversation test
pnpm test -- packages/foo/__tests__/thing.test.ts
pnpm dev                      # API plus Workspace UI
pnpm vestara doctor
```

- Build before testing or running compiled CLI/API commands: tests resolve workspace packages from `dist/`, and runtime commands execute compiled JavaScript.
- Recommended local verification order: `pnpm lint:check && pnpm build && pnpm test`. There is no `typecheck` script.
- `pnpm build` regenerates workspace references. Do not hand-edit `tsconfig.references.json`; use `pnpm dependencies:check` to check package import declarations.
- `pnpm dev` starts the API in the background, launches the Workspace UI, and terminates the background API process when the UI exits.
- `pnpm --filter <workspace-package> test` selects a package; `pnpm test -- <path>` filters Vitest by positional test path.

## Generated Files And Tests

- Do not edit generated `dist/` output or stale ignored `.js`, `.d.ts`, and source-map files next to TypeScript sources. Run `pnpm check:source-artifacts` when source artifacts may be present.
- Vitest discovers tests under package/app `__tests__` directories (`packages/*`, `packages/{providers,tools}/*`, `apps/*`) and Workspace visual tests under `apps/workspace/tests/visual/__tests__`. Tests resolve `@vestara/*` imports from `dist/` via aliases, so a stale build produces misleading failures.
- Workspace visual tests compare approved Playwright baselines by default; use `pnpm screenshots:update` only when intentionally replacing them.
- Database tests use in-memory `sql.js`; its ambient type shim is `types/sql-js.d.ts`.
- Biome is the formatter/linter. JavaScript uses single quotes, trailing commas, and semicolons; `apps/workspace/` has its own Vite/Biome scope.

## CI Pipeline

The CI workflow (`.github/workflows/ci.yml`) runs on push/PR to `main`:

1. `pnpm install --frozen-lockfile`
2. `pnpm dependencies:check` — validates package import declarations
3. OpenCode contract guard — generates and diff-checks `@vestara/opencode-runtime` contracts; on stale output, regenerate with `pnpm --filter @vestara/opencode-runtime opencode:spec:update`
4. `bash build-order.sh` — performs the repository's ordered build
5. `pnpm lint:check` — Biome lint
6. `pnpm test` — Vitest
7. `pnpm documentation:check` — documentation drift gate
8. Benchmarks

The pre-commit hook scripts (`.githooks/pre-commit` → `scripts/pre-commit.sh`) run Biome on staged files and the full test suite, but they are not active in this checkout (no `core.hooksPath` and no `.git/hooks/pre-commit`); run them manually if needed.

## Runtime And Documentation

- The API defaults to `VESTARA_API_PORT=3001`; the Workspace dev server is `5173` and proxies `/api` and `/ws` to the API. `VESTARA_API_URL` overrides CLI/TUI connections, and `VESTARA_REPO` selects the workspace path.
- API startup searches upward for `.vestara/workspace.json` unless `VESTARA_REPO` is set.
- Docs under `vestara-ai-core/docs/` are governed. Run `pnpm docs:validate` for touched Markdown and `pnpm docs:govern` for strict validation; implemented/verified docs require an immutable implementation commit SHA.
- When prose conflicts with manifests, scripts, or source, trust the executable files and update this guide only with verified facts.
