# CLAUDE.md

# Vestara AI OS — Claude Code Instructions

## Project Context

Vestara is a portable AI operating system that boots from an external SSD.
Tech: TypeScript (strict), Node.js 22, Fastify 5, SQLite, React 19, Tailwind 4, Turborepo, pnpm 10.

## Critical Setup (READ THESE FIRST)

1. vestara-blueprint/VESTARA_CONSTITUTION.md  ← Supreme authority
2. vestara-blueprint/00-governance/01-ai-constitution.md  ← Master prompt
3. vestara-blueprint/00-governance/02-engineering-rules.md  ← Non-negotiable rules
4. vestara-blueprint/AI_CONTEXT.md  ← Current project state
5. vestara-blueprint/00-governance/06-ai-development-framework.md  ← VADF workflow

## Commands

- pnpm install    — Install dependencies
- pnpm build      — Build all packages
- pnpm dev        — Start development servers
- pnpm lint       — Run linting
- pnpm typecheck  — TypeScript type checking
- pnpm test       — Run tests

## Engineering Rules (ALWAYS APPLY)

- Use @vestara/types for type definitions
- Use @vestara/validation for Zod schemas
- Use VestaraApp type (not FastifyInstance) for routes
- SQLite only — no PostgreSQL/MySQL
- OpenCode is default provider (works without API keys)
- Ollama is on-demand only (no auto-start)
- Parameterized queries always — no SQL injection
- Zero any types — strict TypeScript everywhere
- Feature-first module organization
- Run pnpm lint && pnpm typecheck && pnpm build && pnpm test before commit
- Shell is /usr/bin/sh for execSync/spawn

## VADF Compliance

- Determine your AIDL phase before starting
- Assume the correct agent role
- Create ADR for architectural decisions
- Update Blueprint with every architecture change
