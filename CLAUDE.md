# AGENTS.md - Comprehensive Guide for AI Coding Agents

This file provides guidance for AI coding agents (Claude Code, Cursor, Codex, Amp, etc.) when working with code in this repository.

## Essential Commands - Quick Reference

### Build & Development
- **Build**: `pnpm build` (root) or `turbo build`
- **Dev**: `pnpm dev` (root) or navigate to package and run `pnpm dev`
- **Setup (core)**: `pnpm setup-dev` — core DBs (Doltgres, Postgres, SpiceDB), env config, migrations, admin user
- **Setup (isolated)**: `pnpm setup-dev --isolated <name>`

### Verification

**Pre-push** (run both, in order):
```bash
pnpm format     # auto-fix formatting
pnpm check      # lint + typecheck + test + format:check
```

**Single-command iteration:** `pnpm typecheck`, `pnpm lint` (`lint:fix`), `pnpm test`, `cd <pkg> && pnpm test --run <file>`

### Database Operations
- **Generate migrations**: `pnpm db:generate`
- **Apply migrations**: `pnpm db:migrate`
- **Drop migrations**: `pnpm db:drop`
- **Database studio**: `pnpm db:studio`

### Creating Changelog Entries (Changesets)

```bash
pnpm bump <patch|minor|major> --pkg <package> "<message>"
```

**Valid package names:** `agents-cli`, `agents-core`, `agents-api`, `agents-manage-ui`, `agents-work-apps`, `agents-sdk`, `create-agents`, `ai-sdk-provider`

**Semver guidance:**
- **Major**: Reserved — do not use without explicit approval
- **Minor**: Schema changes requiring migration, significant behavior changes
- **Patch**: Bug fixes, additive features, non-breaking changes

## Code Style (Biome enforced)
- **Imports**: Use type imports (`import type { Foo } from './bar'`), organize imports enabled
- **Formatting**: Single quotes, semicolons required, 100 char line width, 2 space indent
- **Types**: Explicit types preferred, avoid `any` where possible, use Zod for validation
- **Naming**: camelCase for variables/functions, PascalCase for types/components, kebab-case for files
- **Error Handling**: Use try-catch, validate with Zod schemas, handle errors explicitly
- **React Compiler**: Do not add `memo`, `useMemo`, or `useCallback`; rely on the compiler
- **No Comments**: Do not add comments unless explicitly requested

## TypeScript Configuration

All packages should extend the shared strict baseline at `tsconfig.base.json`. Only override target/module/output settings in per-package config. Strictness flags live in the base.

## Testing (Vitest)
- Place tests in `__tests__/` directories adjacent to code
- Name: `*.test.ts` or `*.spec.ts`
- Pattern: `import { describe, it, expect, beforeEach, vi } from 'vitest'`
- Run with `--run` flag to avoid watch mode
- 60-second timeouts for A2A interactions

## Package Manager
- Always use `pnpm` (not npm, yarn, or bun)
- **Never delete `pnpm-lock.yaml`** and regenerate from scratch — start from base branch's lockfile

## Architecture Overview

Inkeep Agent Framework — multi-agent AI system with A2A (Agent-to-Agent) communication. OpenAI Chat Completions compatible API with sophisticated agent orchestration.

### Core Components

#### Unified API (`agents-api`)
- **`/domains/manage/`** — Agent configuration, projects, tools, and admin operations
- **`/domains/run/`** — Agent execution, conversations, A2A communication, runtime operations
- **`/domains/evals/`** — Evaluation workflows, dataset management, evaluation triggers

## Key Implementation Details

### CRUD HTTP Method Conventions

| Operation | Method | Path Pattern |
|-----------|--------|-------------|
| Create | POST | `/resources` |
| Read (list) | GET | `/resources` |
| Read (single) | GET | `/resources/{id}` |
| Update (partial) | PATCH | `/resources/{id}` |
| Delete | DELETE | `/resources/{id}` |

### Route Authorization Pattern (`createProtectedRoute`)

All API routes **must** use `createProtectedRoute()` from `@inkeep/agents-core/middleware`.

### Internal Self-Calls: `getInProcessFetch()` vs `fetch`

Any code making internal A2A calls **MUST** use `getInProcessFetch()` from `@inkeep/agents-core` instead of global `fetch`. Global `fetch` sends the request over the network where a load balancer may route to a different instance.

## File Locations

- **API Domains**: `agents-api/src/domains/`
- **Database Layer**: `packages/agents-core/src/data-access/`
- **Builder Patterns**: `packages/agents-sdk/src/`
- **Schemas**: `packages/agents-core/src/db/`
- **Tests**: `agents-api/src/__tests__/`
- **UI Components**: `agents-manage-ui/src/components/`
- **Documentation**: `agents-docs/`
- **Examples**: `agents-cookbook/`

## Development Guidelines

### Required for All New Features

✅ **1. Unit Tests** — comprehensive Vitest tests in `__tests__/` directories

✅ **2. Agent Builder UI Components** — corresponding components in `agents-manage-ui/src/components/`

✅ **3. Documentation** — create or update docs in `agents-docs/content/docs/` (MDX format)

**Before marking any feature complete, verify:**
- [ ] `pnpm check` passes
- [ ] Changeset created via `pnpm bump`
- [ ] UI components implemented
- [ ] Documentation added

## Standard Development Workflow

1. Create a branch: `git checkout -b feature/your-feature-name`
2. Create a changeset: `pnpm bump <patch|minor> --pkg <package> "<message>"`
3. Run verification before pushing
4. Commit, then create PR
