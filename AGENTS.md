# AGENTS.md

High-signal, verified facts for agents working on Vestara. If docs conflict with code or scripts, trust the code.

---

## Repo layout

| Path | Role |
|------|------|
| `/home/eddie/projects/vestara/` | Repo root — docs, config, instruction files; **no `package.json` here** |
| `vestara-ai-core/` | **Sole code home** — pnpm monorepo, all source, own `.git` |
| `vestara-blueprint/` | Frozen design docs — read only if task requires |
| `vestara-ai-core/apps/` | api (http+ws, port `VESTARA_API_PORT` default 3001), cli (REPL entry), onboarding-lab, workspace (React+Vite) |
| `vestara-ai-core/packages/` | 51 workspace entries (including `providers/*` and `tools/*` subdirs) |
| `.vestara/` | Runtime persistence (both repo root and `vestara-ai-core/`) |

## Where to work

- Focus implementation and fix work in `vestara-ai-core/`.
- The top-level repo is primarily documentation, blueprints, and design artifacts.
- Use `vestara-blueprint/`, `vestara-foundation/`, `vestara-runtime/`, and `vestara-specifications/` only when asked for strategy, governance, or architecture context.

Both `.git` repos have zero commits — everything is untracked.

## Pre-work (read before editing)

- `vestara-ai-core/docs/foundation/00-glossary.md` — domain vocabulary
- `vestara-ai-core/docs/foundation/01-language.md` — precise meanings
- `vestara-blueprint/00-governance/04-decision-log.md` — current architecture

## Commands (run from `vestara-ai-core/`)

```bash
pnpm install                          # Install deps
bash build-order.sh                   # Sequential build (canonical — see below)
pnpm build                            # tsc -b from root — not useful (no project refs); prefer build-order.sh
pnpm lint                              # biome check --write
pnpm format                            # biome format --write
pnpm test                              # vitest run (requires prior build)
pnpm --filter @vestara/workspace-ui dev  # UI dev server on port 5173
pnpm vestara                           # node apps/cli/dist/index.js (REPL)
pnpm doctor                            # Same as above — runtime health
pnpm vestara validate <path>           # CAP-001 Workspace Orientation (validation protocol)
pnpm conversation-audit                # Health scan of conversation packages
pnpm benchmark                         # Performance benchmarks
pnpm benchmark-index                   # Indexing performance benchmarks
```

### Single package test

```bash
pnpm --filter @vestara/conversation test
```

Note: `pnpm test -- --filter <name>` passes `--filter` to vitest, which filters **test names**, not packages. Use pnpm workspace filtering (`--filter`) instead.

### Verification after every edit

```bash
pnpm lint && pnpm build && pnpm test
```

There is **no `pnpm typecheck` script** — skip it.

## Build quirks

- **`build-order.sh`** is the canonical build. Runs `npx tsc -p` per package **sequentially** in dependency order. Filters 3 known TS errors via `grep -v`: `TS6305`, `TS7016`, `TS2307`.
- Circular deps break the build immediately.
- `pnpm build` runs `tsc -b` from root — works but `build-order.sh` is the actual CI and testing path. The root `tsconfig.json` has no project references, so `pnpm build` only compiles root-level files and is not useful.
- **Tests require prior build** — vitest aliases (`@vestara/*`) resolve to `packages/*/dist/`.

## Toolchain (actual, not aspirational)

| Aspect | Reality |
|--------|---------|
| Lint + format | **Biome 2.x** only. No ESLint, no Prettier. |
| TypeScript | ES2022 target, `nodenext` module, CJS emits (no `type: "module"` in root/packages). `strict: true` + `skipLibCheck: true` |
| API server | **Node.js `http` + `ws` library** — NOT Fastify. No `VestaraApp` type exists. |
| Database | **`sql.js` WASM** in app code. `better-sqlite3` is devDependency only. |
| Validation | Zod used **locally** per package. No `@vestara/validation` package. |
| UI workspace | React 19 + Vite 6 + MUI 6 + Tailwind 4. Separate `"type": "module"`. Built via `tsc -b && vite build`. |
| Default provider | **OpenCode** (`@vestara/provider-opencode`) — works without API keys |
| CI pipeline | install → `build-order.sh` → `pnpm test` → benchmarks. **No lint or typecheck** in CI. |

## Testing

- **Vitest 4**, real SQLite (sql.js WASM)
- Tests per `__tests__/**/*.test.ts` inside each package and app
- No test database setup needed — sql.js runs in-memory
- Test pattern in `vitest.config.ts`: `packages/*/__tests__/**/*.test.ts` and `apps/*/__tests__/**/*.test.ts`

## Coding conventions (verified in code)

- `@vestara/types` for domain types (no validation package exists)
- `VestaraService` lifecycle interface from `@vestara/shared`: `initialize(config?) → start() → stop() → health() → dispose()`
- Parameterized SQL: `db.prepare('...').run(params)` (sql.js style)
- `EventBus` for inter-service communication
- Feature-first modules in `packages/*`
- **Biome config explicitly disables** `noExplicitAny` — code uses `any` freely despite aspirational "zero any" rules in docs
- `console.log` exists in some production code (`packages/logger`, `packages/events-server`, `packages/conversation-runtime`); `console.warn`/`console.error` also used

## Understanding architecture

Understanding is produced by 7 independent producers, each owning one semantic dimension:

| Producer | ID | Output fields |
|----------|-----|---------------|
| LanguageProducer | `language` | identity.primaryLanguage, identity.languageConfidence |
| FrameworkProducer | `framework` | identity.framework |
| ArchitectureProducer | `architecture` | architecture.kind, architecture.entryPoints |
| MaturityProducer | `maturity` | maturity.level, maturity.testCoverage, etc. |
| RiskProducer | `risks` | maturity.risks |
| HealthProducer | `health` | maturity.healthScore, state.* |
| ActivityProducer | `activity` | activity.*, memory.* |

Each implements `UnderstandingProducer` and contributes only the fields it owns. The `DefaultUnderstandingAssembler` merges all contributions into the final `WorkspaceUnderstanding` snapshot. Producers are independently measurable via the evaluation corpus (see `vestara-ai-core/packages/evaluation/`).

## Architectural pattern (recurring across 4 layers)

| Layer | Coordinator | Specialists | Each specialist owns |
|-------|-------------|-------------|---------------------|
| Execution | `RuntimeGroup` | `Runtime` | Lifecycle behavior |
| Analysis | `WorkspaceRuntime` | Pipeline stages | One analysis stage |
| Knowledge | `UnderstandingAssembler` | `UnderstandingProducer` | One semantic field |
| Evaluation | `EvaluationHarness` | Corpus entries | One assertion set |

Rule: coordinators compose, specialists decide, every concern has one owner.

## Existing instruction files (cross-check before trusting)

- `vestara-blueprint/AGENTS.md` — references **Fastify**, `@vestara/validation`, `VestaraApp`, `useSWR`, `services/` paths that **don't match actual code**
- `vestara-blueprint/CLAUDE.md` — similar stale claims (Fastify, `@vestara/validation`, `VestaraApp`)
- `.opencode/skills/testing-and-quality/SKILL.md` — references ESLint, Prettier, Prisma, swr (none exist in this repo)
- `vestara-blueprint/VESTARA_CONSTITUTION.md` — governance doc; the repo's actual code takes precedence over any claims in it

## Root repo guidance

- Do implementation work in `vestara-ai-core/` unless the request explicitly involves docs, blueprints, or repo-level config.
- Run build/test commands from `vestara-ai-core/`; `bash build-order.sh` is the canonical build path and `pnpm test` requires a prior build.
- Prefer `pnpm lint` and `pnpm format` from `vestara-ai-core/` rather than root-level package scripts.

## OpenCode config

- `opencode.json` at repo root contains only a `$schema` reference
- The real OpenCode integration is through the skills in `.opencode/skills/`