# Setting up monorepo with npm workspaces (strict TypeScript)

This guide explains the "why", "how", and pitfalls of using a monorepo for this learning project.

## Why monorepo for this project
Problems it solves:
- Avoids client/server type drift when both evolve quickly.
- Enables atomic refactors (API + UI changes in one commit).
- Unifies lint/typecheck/test standards.
- Makes future shared packages first-class (`packages/*`).

Why it is good for learning:
- You learn architecture boundaries early.
- You learn how contracts move between frontend/backend.
- You learn workspace-level tooling used in real teams.

## Baseline structure
- `apps/client` - React + Vite app
- `apps/server` - NestJS + Prisma API
- `packages/tsconfig` - shared strict TypeScript config
- `packages/eslint-config` - shared lint setup
- `packages/contracts` - shared API contracts/types (optionally runtime schemas)

## Strict TypeScript policy (non-negotiable)
At minimum in the shared base config:
- `strict: true`
- `noImplicitAny: true`
- `exactOptionalPropertyTypes: true`
- `noUncheckedIndexedAccess: true`
- `useUnknownInCatchVariables: true`
- `noImplicitOverride: true`

Mindset:
- Keep `any` out unless absolutely unavoidable.
- Validate runtime input at boundaries (especially server DTOs/schemas).
- Never leak DB models directly to public API responses.

## npm workspaces mental model
The repo root is the orchestrator:
- One top-level install (`npm install`) resolves all workspace deps.
- Workspace packages can depend on each other via `workspace:*`.
- Root scripts can run commands for all workspaces.

What to watch out for:
- Hoisted dependencies can hide missing local deps. Always declare direct deps explicitly in each workspace package.
- Name internal packages clearly (e.g., `@todo/contracts`) and version intentionally.

## Shared contracts strategy decision
Choose one path early:

### A) Type-only contracts
- `packages/contracts` exports TypeScript types/interfaces only.
- Server validates via Nest DTOs/class-validator.

Pros: simpler.
Cons: runtime validation is separate from shared contract definitions.

### B) Runtime schema contracts (recommended for this project)
- `packages/contracts` exports runtime schemas (for example, zod).
- Types are inferred from schemas and reused by client/server.

Pros: stronger correctness across compile-time and runtime.
Cons: slightly more setup complexity.

Current status: monorepo tooling can start now; we will lock A vs B before wiring API contracts deeply.

## Sequence to set up without pain
1. Initialize root workspace config (`package.json`, `workspaces`).
2. Create `apps/` and `packages/` folders.
3. Scaffold `apps/client` (Vite React TS).
4. Scaffold `apps/server` (NestJS TS).
5. Create and apply shared strict `tsconfig`.
6. Add shared lint config and root scripts.
7. Verify `dev`, `lint`, and `typecheck` from root.
8. Add contracts package and consume it from both apps.

## Root scripts to standardize early
- `dev` - run client + server concurrently
- `build` - build all workspaces
- `lint` - lint all workspaces
- `typecheck` - typecheck all workspaces
- `test` - run all tests

## Common monorepo mistakes and how to avoid them
- Mistake: sharing backend internals with frontend.
  - Fix: share only contracts/types, never services/repositories.
- Mistake: weak workspace boundaries.
  - Fix: keep app code in apps, reusable artifacts in packages.
- Mistake: only one app runs strict TS.
  - Fix: shared `tsconfig` package with both apps extending it.
- Mistake: no phase-level verification.
  - Fix: after each phase run lint + typecheck + tests.

## What we do next
Next execution step is Phase 0 setup with npm workspaces:
- scaffold workspaces
- enforce strict TS base
- bootstrap client/server
- verify root-level commands work
