# AI_RULES.md

## Universal Engineering Rules — All AI Agents Must Obey

> **These rules are LAW. Every AI agent working on Vestara MUST memorize and follow them. No exceptions.**

---

### 🚫 NEVER DO

| Rule | Description |
|------|-------------|
| **NEVER** use `any` | Zero `any` types. Use `unknown` + Zod or proper types. |
| **NEVER** duplicat code | Extract to `@vestara/*` shared packages. |
| **NEVER** ignore lint errors | `pnpm lint` must pass with zero errors. |
| **NEVER** disable TypeScript | No `// @ts-ignore`, `// @ts-expect-error`, config loosening. |
| **NEVER** hardcode secrets | API keys, tokens, passwords in `.env` only (gitignored). |
| **NEVER** bypass security | No disabled CSP, auth, rate limits, validation. |
| **NEVER** hide dependencies | All deps in `package.json`, no global state. |
| **NEVER** use console.log | Use `@vestara/core/logger` with proper levels. |
| **NEVER** commit without build | `pnpm build` must pass before commit. |

### ✅ ALWAYS DO

| Rule | Implementation |
|------|----------------|
| **Always** prefer composition | Composition over inheritance for all modules. |
| **Always** document new modules | JSDoc with `@param`, `@returns`, `@example`, `@throws`. |
| **Always** update Blueprint | Architecture changes = Blueprint update in same PR. |
| **Always** explain breaking changes | Migration guide + deprecation warning required. |
| **Always** use explicit types | Every function parameter and return type explicitly typed. |
| **Always** validate at boundaries | Zod schemas on every API, config, IPC, DB boundary. |
| **Always** parameterize queries | `db.prepare('SELECT * FROM users WHERE id = ?').get(id)`. |
| **Always** handle errors explicitly | `VestaraError` with typed codes, never `throw Error`. |

### 🗄️ DATABASE RULES

| Rule | Standard |
|------|----------|
| Tables | `snake_case` plural: `projects`, `tasks`, `activity_log` |
| Columns | `snake_case`: `created_at`, `user_id`, `sort_order` |
| Primary keys | `id TEXT PRIMARY KEY` — UUID v7 |
| Timestamps | `created_at TEXT`, `updated_at TEXT` (ISO 8601) |
| Foreign keys | Explicit `REFERENCES table(id) ON DELETE CASCADE` |
| Migrations | Additive only: `ALTER TABLE ADD COLUMN` with `PRAGMA table_info` |

### 🌐 API RULES

| Rule | Standard |
|------|----------|
| Route type | `VestaraApp` (NOT `FastifyInstance`) |
| Auth | `{ preHandler: authMiddleware }` for protected routes |
| Validation | Zod `.parse(request.body)` — throws on invalid |
| Response format | `{ success: true, data }` or throw `VestaraError` |

### ⚛️ FRONTEND RULES

| Rule | Standard |
|------|----------|
| Component structure | Props interface, named export, one component per file |
| Styling | Tailwind CSS 4 with `vestara-*` tokens |
| Data fetching | SWR via `useSWR` hook |
| State management | SWR (server) + React Context (global) + useState (local) |

### 🧪 TESTING RULES

| Rule | Standard |
|------|----------|
| Framework | Vitest |
| Naming | `*.test.ts` alongside source |
| Database | Real SQLite in-memory (`:memory:`) — no mocks |
| API | Fastify `inject()` — real HTTP stack |
| Coverage | Services ≥80%, Apps ≥60% |

### 📝 EVERY COMMIT

```markdown
- [ ] Strict TypeScript — zero `any`, explicit types
- [ ] Zod validation at all boundaries
- [ ] Feature-first organization
- [ ] Parameterized queries only
- [ ] Tests written and passing
- [ ] `pnpm lint && pnpm typecheck && pnpm build && pnpm test` passes
- [ ] No console.log, no TODO without ticket
- [ ] Blueprint updated if architecture changed
- [ ] ADR created for architectural decisions
```

---

**READ THE FULL VERSION**: `00-governance/02-engineering-rules.md`
