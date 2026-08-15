# Vestara API Restructure — Standalone Directory Plan

## Overview

Move the HTTP + WebSocket API out of `vestara-ai-core/apps/api/` into a dedicated
top-level directory `vestara-api/` at the vestara root, without removing or
breaking the current API behavior. The API continues to run from its current
location until the new standalone directory is fully functional and verified;
only then is the old location retired.

This plan is **aligned with the Marketplace v2 roadmap** in two ways:

1. **Marketplace v0.2 (Workspace Experience)** — the marketplace routes
   (`apps/api/src/routes/marketplace.ts`), the `MarketplaceService` /
   `LocalMarketplaceRegistry` wiring in `workspace-context.ts`, and the
   `marketplace.*` WebSocket event bridge are part of the API surface. The
   restructure carries them into `vestara-api/` intact and is sequenced so it
   never blocks or breaks the v0.2 delivery (`docs/marketplace/MARKETPLACE-V0.2-WORKSPACE-EXPERIENCE.md`).
2. **MP-006 Remote Marketplace / MW-005-006** — the remote/publishing milestone
   anticipates a dedicated `apps/marketplace-service/` with
   registry/search/download/version/signature/publisher/API components
   (`vestara-blueprint/10-developer-platform/marketplace-implementation.md`,
   `vestara-blueprint/20-roadmaps/marketplace-workspace-roadmap.md`). This plan
   establishes the **standalone service directory pattern** (`vestara-api/` as a
   gitlink) that `vestara-marketplace-service/` will follow later, and reserves
   the root workspace / cross-repo reference mechanics it needs.

Two structural decisions are fixed up front:

1. **Repo model** — `vestara-api/` is its own git repository pinned in the vestara
   root as a gitlink (mirroring `vestara-ai-core` and `vestara-blueprint`). Root
   bump commits track its `HEAD`. The same model is reserved for a future
   `vestara-marketplace-service/` (MP-006).
2. **Dependency strategy** — `vestara-api` consumes `@vestara/*` packages from
   `vestara-ai-core/packages/*` through a root pnpm workspace spanning both, so
   `workspace:*` dependencies and `tsc` project references keep working without
   a registry publish.

### Non-goals

- No changes to the HTTP/WS contract, routes, port (`3001`), or protocol.
- No behavioral change to the CLI, Workspace UI, TUI, or systemd units while the
  plan executes.
- No removal of `vestara-ai-core/apps/api` until the final cutover phase.
- No marketplace feature work: v0.2 routes/UI/events are carried as-is; the
  remote `marketplace-service` (MP-006) is a later milestone that only reuses
  the directory pattern established here.

## Current State (verified)

| Fact | Value |
|------|-------|
| API package | `@vestara/api` at `vestara-ai-core/apps/api/` |
| Source | 75 TS files in `src/`, 30 test files in `__tests__/`, ~18k LOC |
| Workspace deps | 44 `@vestara/*` packages via `workspace:*` |
| Build | `tsc -b` through `tsconfig.references.json` → `apps/api/tsconfig.reference.json` |
| Entry | `apps/api/src/index.ts` → `dist/index.js` |
| Port / env | `VESTARA_API_PORT` (default `3001`), `VESTARA_API_URL` (client side) |
| Repo root resolution | `resolveRepoRoot()` walks up from `apps/api/dist/` looking for `.vestara/workspace.json` (overridable via `VESTARA_REPO`) |

### Consumers / integration surface

| Integration | Location | Path-sensitive |
|-------------|----------|----------------|
| pnpm workspace membership | `vestara-ai-core/pnpm-workspace.yaml` (`apps/*`) | yes |
| Build graph | `tsconfig.references.json` + `apps/api/tsconfig.reference.json` | yes |
| Vitest discovery | `vestara-ai-core/vitest.config.ts` (`apps/*/__tests__`) + alias resolution to `packages/*/dist` | yes |
| Root scripts | `vestara-ai-core/package.json` (`dev`, `dev:api`, `vestara`) | yes |
| Image builder | `scripts/build-vestara-image.sh` (artifact check `$ROOT/apps/api/dist/index.js`, rsync to `/opt/vestara`) | yes |
| Host install | `scripts/install-os0.sh` (rsync `vestara-ai-core/` → `/opt/vestara`) | yes |
| systemd units | `os/systemd/vestara-api.service`, `vestara-api-local.service`, `vestara-api-reload.path` | yes |
| CLI → API | `apps/cli/` connects via `VESTARA_API_URL` (default `http://127.0.0.1:3001`) | no (URL) |
| Workspace UI → API | `apps/workspace/vite.config.ts` proxy `/api`, `/ws` → `127.0.0.1:3001` | no (URL) |
| TUI → API | `packages/tui/` connects via `VESTARA_API_URL` | no (URL) |
| Docs automation | `packages/documentation/src/planning.ts` (`readScope: ['packages/**/src/**','apps/api/src/**']`), `apps/cli/src/commands/docs.ts` repository config | yes |
| Docs tables | `scripts/ensure-docs.sh` (`apps/api/` row) | yes |
| Docs mirror comments | `apps/workspace/src/lib/*.ts` (reference `apps/api/src/routes/*`) | yes |
| Skills/agents | root `.opencode/skills/*/SKILL.md`, `vestara-ai-core/AGENTS.md` | yes |
| Project tree snapshot | `vestara_project_tree.md` (root, tracked) | yes |
| CI | `vestara-ai-core/.github/workflows/ci.yml` (installs/builds/tests `vestara-ai-core`) | yes |
| Marketplace v0.2 routes | `apps/api/src/routes/marketplace.ts` (search/assets/categories/registries/installed/updates/rescan/install/update/uninstall/verify/publish/detect) | yes |
| Marketplace service wiring | `apps/api/src/workspace-context.ts` — `MarketplaceService`, `LocalMarketplaceRegistry`, `marketplaceEventSink`, `LocalExtensionManager`, `VESTARA_MARKETPLACE_ROOTS` | yes |
| Marketplace WS bridge | `apps/api/src/server.ts` registration (`/api/marketplace`) + `workspace-context.ts` event sink → `marketplace.*` WS events | yes |
| Marketplace UI client | `apps/workspace/src/lib/marketplace.ts`, `apps/workspace/src/pages/Marketplace/*` (consumes `/api/marketplace`, URL-based) | no (URL) |

### Constraint: pnpm workspace roots

`vestara-ai-core` is itself a pnpm workspace root (`pnpm-workspace.yaml` +
`package.json` + `pnpm-lock.yaml`). Adding `vestara-api` as a workspace member
of that same root would require `vestara-api` to live inside the workspace root
glob, which conflicts with it being a separate top-level gitlink.

**Resolution adopted:** introduce a **root-level pnpm workspace** at the vestara
root that includes both `vestara-api` and `vestara-ai-core/packages/*` (and
`apps/*` as needed). `pnpm` supports nested workspace roots: commands run inside
`vestara-ai-core/` keep resolving to that repo's own workspace; commands at the
root (or inside `vestara-api/`) resolve to the root workspace. `@vestara/*`
resolution from `vestara-api` works because the root workspace globs
`vestara-ai-core/packages/*`. `tsc` project references cross the boundary via
relative paths (`../vestara-ai-core/packages/*/tsconfig.reference.json`).

## Target End State

```text
vestara/                          (coordination repo)
├── vestara-ai-core/              (gitlink — packages, CLI, UI, workspace)
│   └── apps/                     (no api/ after cutover)
├── vestara-api/                  (NEW gitlink — @vestara/api)
│   ├── package.json              (@vestara/api)
│   ├── tsconfig.json / tsconfig.reference.json
│   ├── src/  __tests__/  docs/  README.md
│   └── dist/
├── vestara-marketplace-service/  (RESERVED — MP-006 remote marketplace, later)
├── pnpm-workspace.yaml           (NEW root workspace)
├── package.json                  (NEW root scripts: build/dev/test/lint for the API)
└── vestara-blueprint/            (gitlink, unchanged)
```

`vestara-api/` is the reference implementation of the **standalone service
gitlink pattern**; a future `vestara-marketplace-service/` (MP-006) mirrors it:
own git repo pinned at root, root-workspace `@vestara/*` resolution, systemd
unit + image-builder wiring, and its own API routes — without routing through
`vestara-api`.

## Execution Phases

Each phase is independently verifiable and keeps the current API functional.

### Phase 0 — Baseline snapshot

- Record current green state: `pnpm build && pnpm lint:check && pnpm test` pass in
  `vestara-ai-core`; API serves `GET /api/health` on `3001`.
- Note the pinned `vestara-ai-core` gitlink commit so the old API location stays
  reproducible.
- **Marketplace v0.2 alignment:** record the marketplace surface state —
  `apps/api/src/routes/marketplace.ts` endpoints, `MarketplaceService` wiring in
  `workspace-context.ts`, and the `marketplace.*` WS bridge — and run the v0.2
  smoke (`curl` search/assets/categories/installed + install dry-run →
  awaiting-permission) so any regression during the move is attributable.
- **Gate:** baseline is green; API health check passes; marketplace smoke passes.

### Phase 0 results (2026-08-15)

| Check | Result |
|-------|--------|
| `pnpm build` | ✅ green (96 projects, dependency boundaries valid) |
| `pnpm lint:check` | ✅ green (1253 files, no diagnostics) |
| `pnpm test` | ✅ 264 files / 2205 tests passed |
| API health | ✅ `GET /api/health` → `{"status":"ok"}` on `3001` |
| `vestara-ai-core` gitlink HEAD | `9051186` (working tree intentionally dirty with prior work) |
| GitHub repo aligned | ✅ `Vestara-Tech/api` created (private); local `vestara-api/` git repo initialized, `origin` → `Vestara-Tech/api.git`, initial commit `bdd920f` pushed, default branch `main` |
| Marketplace surface | ✅ routes in `apps/api/src/routes/marketplace.ts`; `MarketplaceService`/`LocalMarketplaceRegistry`/`marketplaceEventSink` in `workspace-context.ts`; `/api/marketplace` registered in `server.ts`; fixtures seeded (`vestara.analysis`, `vestara.git-helper`, `vestara.review-standards`) |
| Marketplace smoke | ✅ rescan → 3 assets / 3 categories; search finds `git-helper`; dry-run → `planning` (2 permissions); unapproved → `awaiting-permission`; approved → `completed` (active); updates empty; verify `completed`; uninstall `completed`; installed count back to 0 |

Notes:

- The API was smoke-tested against `VESTARA_REPO=<vestara-ai-core>` (the seeded
  fixture workspace). Booted against the large dirty tree it takes ~3–4 min to
  bind; this is pre-existing boot cost, not a restructure issue.
- `vestara-api/` contains only the scaffold README + git metadata at this stage;
  the API source moves there in Phase 2. Current API behavior is untouched.

### Phase 1 — Root workspace bootstrap (no API move)

- Add root `pnpm-workspace.yaml`:
  ```yaml
  packages:
    - 'vestara-api'
    - 'vestara-ai-core/packages/*'
    - 'vestara-ai-core/packages/providers/*'
    - 'vestara-ai-core/packages/tools/*'
    - 'vestara-ai-core/apps/*'
  ```
- Add root `package.json` with API-oriented scripts (`build:api`, `test:api`,
  `lint:api`, `dev:api`) that delegate into `vestara-api`.
- Verify `pnpm install` at root resolves `@vestara/*` from
  `vestara-ai-core/packages/*` (create a throwaway import to confirm).
- **Gate:** root `pnpm install` succeeds; `vestara-ai-core` workspace still
  installs/builds independently.

### Phase 2 — Mirror the API into vestara-api (no removal)

- `git init` `vestara-api/`; copy `apps/api/**` (source, tests, `docs/api.yaml`,
  `README.md`, `package.json`, `tsconfig*.json`, `vitest.config.ts` if present).
  This includes the marketplace route file (`routes/marketplace.ts`), the
  `MarketplaceService`/`LocalMarketplaceRegistry` wiring in
  `workspace-context.ts`, and the `marketplace.*` WS event bridge — moved as-is,
  no feature changes.
- Adjust inside the copy only:
  - `resolveRepoRoot()` fallback depth (`__dirname` now `vestara-api/dist/`).
  - `tsconfig.json` `extends` path (root config now one level up, not inside
    `vestara-ai-core`).
  - `package.json` `dev`/`build` script paths.
- Keep `@vestara/api` package name and all `workspace:*` dependencies unchanged.
  The marketplace UI client (`apps/workspace/src/lib/marketplace.ts`,
  `pages/Marketplace/*`) is untouched — it talks to `/api/marketplace` by URL.
- **Gate:** `pnpm --dir vestara-api build` compiles; `pnpm --dir vestara-api test`
  passes (including `marketplace-routes.test.ts`); the original `apps/api` is
  untouched and still runs.

### Phase 3 — Cross-workspace type resolution

- Point `vestara-api/tsconfig.reference.json` references at
  `../vestara-ai-core/packages/*/tsconfig.reference.json`.
- Add `vestara-api/tsconfig.reference.json` to the **root** build graph (new root
  `tsconfig.references.json` or a root `tsc -b` invocation listing both trees).
- Verify `tsc -b` from the root compiles `vestara-api` against
  `vestara-ai-core/packages/*` `.d.ts` outputs.
- **Gate:** root `tsc -b` succeeds; `vestara-ai-core`'s own `tsc -b` still
  succeeds (old API remains part of its graph for now).

### Phase 4 — Runtime equivalence of the new location

- Run `vestara-api` standalone (`node vestara-api/dist/index.js`), open a
  workspace via `VESTARA_REPO`, and confirm:
  - `GET /api/health`, `/api/host`, `/api/boot`
  - WebSocket `/ws` connects
  - A representative route set (agents, sessions, external-runtime, workflow)
  - **Marketplace v0.2 parity:** `GET /api/marketplace/search|assets|categories|
    registries|installed|updates` and `POST /api/marketplace/install` dry-run →
    `planning` / `awaiting-permission` → approved → `completed`; `marketplace.*`
    events arrive over `/ws` with no secret-bearing payloads.
- Compare response shape against the old `apps/api` instance (same `VESTARA_REPO`).
- **Gate:** response parity checklist passes for both instances, including the
  marketplace route set and WS event bridge.

### Phase 5 — Align the integration surface

Update every path-sensitive consumer (exact list in the Integration Checklist
below): root scripts, image builder, install script, systemd units, docs
automation `readScope`, docs tables, workspace mirror comments, skills/agents,
`vestara_project_tree.md`. CLI, Workspace UI, and TUI need no change (URL-based).

- **Gate:** `rg "apps/api"` at the root returns only intentional legacy notes
  (or zero hits); `pnpm lint:check` and `pnpm documentation:check` pass with the
  new locations.

### Phase 6 — CI + gitlink wiring

- Add a root or `vestara-api` CI job (or extend `vestara-ai-core` CI) that
  installs the root workspace, builds `vestara-api`, and runs its tests.
- Commit `vestara-api` (its own repo), then create the root **bump commit**
  pinning the new gitlink (established pattern: `chore: bump vestara-api
  gitlink (...)`).
- **Gate:** root `git status` shows `vestara-api` as a clean gitlink; CI green.

### Phase 7 — Cutover (only after all gates pass)

- Point systemd units / image builder / root scripts at `vestara-api/dist`.
- Remove `vestara-ai-core/apps/api/` (package.json, src, tests, docs) and drop
  `apps/api` from `tsconfig.references.json`, `vitest.config.ts`, and the
  `apps/*` workspace note.
- Commit the removal inside `vestara-ai-core` and bump its gitlink at root.
- **Gate:** full green suite from the root; `vestara api`/`pnpm dev:api` runs
  exclusively from `vestara-api`; old path returns 404/no process.

### Phase 8 — Post-cutover verification & cleanup

- Re-run the parity checklist against the cutover instance.
- Update `AGENTS.md` (root + `vestara-ai-core`), `vestara_project_tree.md`,
  `ensure-docs.sh`, skills, and the docs baseline if drift appears.
- **Gate:** `pnpm build && pnpm lint:check && pnpm test && pnpm documentation:check`
  all green from the root.

## Integration Checklist (Phase 5 + Phase 7)

| # | File | Change |
|---|------|--------|
| 1 | `vestara-ai-core/package.json` | `dev`, `dev:api` → point at `vestara-api/dist/index.js` |
| 2 | root `package.json` (new) | API build/test/lint/dev scripts |
| 3 | root `pnpm-workspace.yaml` (new) | globs `vestara-api` + `vestara-ai-core/{packages,apps}` |
| 4 | `scripts/build-vestara-image.sh` | artifact check → `$ROOT/../vestara-api/dist/index.js`; rsync `vestara-api/` into image |
| 5 | `scripts/install-os0.sh` | rsync `vestara-api/` alongside `vestara-ai-core/` |
| 6 | `os/systemd/vestara-api.service` | `ExecStart` → `/opt/vestara/vestara-api/dist/index.js` |
| 7 | `os/systemd/vestara-api-local.service` | `ExecStart` + `WorkingDirectory` + `ReadWritePaths` |
| 8 | `os/systemd/vestara-api-reload.path` | `PathChanged` → `/opt/vestara/vestara-api/dist` |
| 9 | `packages/documentation/src/planning.ts` | `readScope` adds `../vestara-api/src/**` (or root repo config) |
| 10 | `apps/cli/src/commands/docs.ts` | add `vestara-api` repository entry to `serviceFor()` |
| 11 | `scripts/ensure-docs.sh` | table row `apps/api/` → `vestara-api/` |
| 12 | `apps/workspace/src/lib/*.ts` | comment refs `apps/api/src/routes/*` → `vestara-api/src/routes/*` |
| 13 | root + `vestara-ai-core` `.opencode` skills, `AGENTS.md` | path references |
| 14 | `vestara_project_tree.md` | snapshot update |
| 15 | `vestara-ai-core/tsconfig.references.json`, `vitest.config.ts` | remove `apps/api` at cutover |
| 16 | `vestara-ai-core/.github/workflows/*` | build/test step source updates |
| 17 | `vestara-api/src/routes/marketplace.ts` | moved as-is (v0.2 route set) — no change |
| 18 | `vestara-api/src/workspace-context.ts` | `MarketplaceService`, `LocalMarketplaceRegistry`, `marketplaceEventSink`, `VESTARA_MARKETPLACE_ROOTS` carried as-is |
| 19 | `vestara-api/src/server.ts` | `/api/marketplace` handler registration carried as-is; `marketplace.*` WS bridge intact |
| 20 | `apps/workspace/src/lib/marketplace.ts`, `pages/Marketplace/*` | no change (URL-based against `/api/marketplace`) — verify only |
| 21 | `vestara-ai-core/docs/marketplace/*` | note new API location in v0.2 plan's Grounding/Phase A references (docs drift safe) |

## Marketplace v2 Alignment

### v0.2 — Workspace Experience (in flight, carried as-is)

The restructure treats `packages/marketplace` and the v0.2 UI as **upstream
dependencies**: the API is a thin adapter over `MarketplaceService`. The move
therefore:

- Carries the route handler, service wiring, and `marketplace.*` WS bridge
  verbatim into `vestara-api/` (Phase 2, checklist #17–19).
- Adds marketplace routes + WS bridge to the Phase 4 parity checklist and the
  Phase 0 baseline smoke, so the move is provably regression-free for v0.2.
- Never edits `packages/marketplace`, `apps/workspace/src/lib/marketplace.ts`, or
  `apps/workspace/src/pages/Marketplace/*` — the UI consumes `/api/marketplace`
  by URL and is agnostic to where the API lives.
- **Sequencing rule:** if v0.2 delivery (Phases A–F) is still open, land its
  remaining work on the **current** `apps/api` first and re-baseline; the
  restructure phases then mirror the updated surface. Do not split a v0.2
  feature across the old and new locations.

### MP-006 / MW-005-006 — Remote Marketplace & Publisher (later, pattern reused)

The blueprint anticipates a dedicated `apps/marketplace-service/`
(registry/search/download/version/signature/publisher/api). This plan:

- Establishes the **standalone service gitlink pattern** (`vestara-api/`) that
  the future `vestara-marketplace-service/` reuses: own repo pinned at root,
  root-workspace `@vestara/*` resolution, own `tsconfig.reference.json` in the
  root build graph, systemd unit + image-builder wiring.
- Reserves the root workspace and cross-repo reference mechanics now, so MP-006
  needs no new infra.
- **Decision point (future):** MP-006 `vestara-marketplace-service/` is a peer
  service, not a dependency of `vestara-api`. The Workspace UI would target it
  via a separate `VESTARA_MARKETPLACE_URL` (or a proxy prefix), mirroring how the
  API is reached today. That routing decision belongs to the MP-006 milestone,
  not this restructure.

### Phase ordering against marketplace work

| Marketplace milestone | This plan's dependency |
|----------------------|------------------------|
| v0.2 Phases A–E | Must be green on current `apps/api` before Phase 2 mirror (sequencing rule above) |
| v0.2 Phase F (Graph follow-through) | Independent; no restructure coupling |
| MP-006 Remote Marketplace | After cutover; reuses `vestara-api/` directory pattern |

## Risks & Mitigations

| Risk | Mitigation |
|------|-----------|
| pnpm nested-workspace ambiguity | Keep `vestara-ai-core` self-contained; only root/`vestara-api` rely on the root workspace. Verify with a throwaway import in Phase 1. |
| `resolveRepoRoot()` depth change | Phase 2 makes the new location self-resolving; `VESTARA_REPO` remains the explicit override. |
| Cross-repo `tsc` references | Phase 3 compiles against published `.d.ts` outputs (`declaration: true`), so no source coupling. |
| systemd/image path drift | Phase 5–7 flip each unit in a single commit; Phase 0 baseline makes regressions detectable. |
| Docs drift gate | Re-run `pnpm documentation:baseline` in Phase 8 only after content is correct. |
| Old location accidentally removed early | Phase 7 is the only removal phase; everything before it leaves `apps/api` intact. |
| Marketplace v0.2 regression during move | Phase 0 baseline + Phase 4 parity both include the marketplace route set and WS bridge; v0.2 UI is URL-based and untouched. |
| Marketplace v0.2 feature split across old/new API | Sequencing rule: land v0.2 work on current `apps/api`, re-baseline, then mirror. |
| MP-006 service added before pattern mature | `vestara-api` is the reference pattern; MP-006 reuses it only after cutover (Phase 8). |

## Rollback

Any phase before Phase 7 is reversible by restoring the gitlink pins. If Phase 7
exposes a regression: revert the removal commit in `vestara-ai-core`, point the
systemd units/scripts back at `apps/api`, rebuild, and re-verify — the old API
was never deleted in earlier phases. Marketplace v0.2 behavior rolls back with
the API location automatically (the UI never changes).
