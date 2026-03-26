# Actual monorepo bootstrap walkthrough (npm workspaces + strict TS)

This is the practical setup guide for Phase 0.  
Goal: create a clean monorepo foundation that can scale as we add NestJS concepts.

---

## 0) Preconditions

- You are inside the repo root: `todo_nestjs`
- Node.js and npm are installed
- Repo currently has docs and `README.md`

Why this matters:
- We want one repo to orchestrate all apps/packages from day one.

---

## 1) Create workspace folder structure

Create top-level folders:
- `apps/`
- `packages/`

Why:
- `apps/*` holds runnable applications.
- `packages/*` holds reusable config/contracts used by apps.
- This enforces architecture boundaries early.

---

## 2) Configure root `package.json` for npm workspaces

Create/update root `package.json` with:
- `private: true`
- `workspaces: ["apps/*", "packages/*"]`
- root scripts:
  - `dev`
  - `build`
  - `lint`
  - `typecheck`
  - `test`

Why:
- `private: true` prevents accidental publish of root package.
- `workspaces` tells npm where local packages/apps are.
- root scripts give one consistent command surface for the whole repo.

---

## 3) Scaffold client app in `apps/client`

Run Vite React TypeScript scaffold in `apps/client`.

Why:
- Gives a fast frontend baseline.
- Vite is familiar for you and quick for iteration.
- Starting with TypeScript avoids migration debt later.

Verification:
- `npm run dev -w apps/client` starts the frontend.

---

## 4) Scaffold server app in `apps/server`

Run NestJS scaffold in `apps/server`.

Why:
- We want idiomatic Nest structure from the start (modules/controllers/services).
- Nest CLI gives the right initial project shape and scripts.

Verification:
- `npm run start:dev -w apps/server` starts the API.

---

## 5) Add shared strict TypeScript config package

Create `packages/tsconfig`:
- `package.json` with a name like `@todo/tsconfig`
- `base.json` with strict compiler options
- optional `nest.json` and `react.json` presets extending `base.json`

Recommended strict options in `base.json`:
- `strict`
- `noImplicitAny`
- `exactOptionalPropertyTypes`
- `noUncheckedIndexedAccess`
- `useUnknownInCatchVariables`
- `noImplicitOverride`
- `noFallthroughCasesInSwitch`

Then make:
- `apps/client/tsconfig.json` extend shared config
- `apps/server/tsconfig.json` extend shared config

Why:
- Strictness is centrally enforced.
- Prevents client/server drift in TypeScript quality.

---

## 6) Add shared ESLint config package

Create `packages/eslint-config`:
- common lint rules for TS
- app-specific extensions if needed (React/Nest differences)

Update client/server ESLint config to consume shared config.

Why:
- One lint policy across all workspaces.
- Fewer style and quality inconsistencies.

---

## 7) Add root scripts that orchestrate all workspaces

At root `package.json`, define scripts that run in all relevant workspaces:
- `dev`: run client + server in parallel
- `build`: build client + server
- `lint`: lint all
- `typecheck`: typecheck all
- `test`: run tests all

Why:
- Standardized developer workflow.
- Easy CI integration later.

---

## 8) Decide contracts strategy now (we choose B)

Use `packages/contracts` with runtime schema contracts (zod), not just TS types.

What to include:
- request/response schemas for key endpoints
- inferred TS types exported from schemas

Why:
- compile-time + runtime contract consistency
- fewer integration bugs between client and server
- perfect fit for "over-engineered for learning" goals

---

## 9) Environment setup baseline

Define env strategy:
- `apps/server/.env` for backend secrets/config
- `apps/client/.env` for frontend public vars (Vite `VITE_` prefix)
- `.env.example` templates committed for both apps

Why:
- Avoids env confusion and collisions in monorepo setups.
- Makes onboarding deterministic.

---

## 10) Bootstrap verification checklist (must pass)

From repo root, verify:
- install works once (`npm install`)
- client runs
- server runs
- lint passes
- typecheck passes

Why:
- This is our "done definition" for Phase 0.
- We only move to Nest feature work after foundation is stable.

---

## 11) Commands reference (example flow)

Note: keep these as a sequence reference; we will run and adapt them together.

```bash
# from repo root
mkdir -p apps packages
npm init -y

# scaffold client
npm create vite@latest apps/client -- --template react-ts

# scaffold server (requires Nest CLI usage through npx)
npx @nestjs/cli new apps/server --package-manager npm

# install workspace deps from root
npm install
```

If `mkdir -p` behaves differently on your shell, create folders manually in Explorer or with shell-equivalent commands.

---

## 12) Common pitfalls to avoid during bootstrap

- Forgetting `"private": true` in root `package.json`
- Not declaring direct dependencies per workspace (hoisting can hide this)
- Different tsconfig strictness between apps
- Putting shared runtime code in app folders instead of `packages/*`
- Starting feature coding before lint/typecheck baseline is clean

---

## 13) Definition of done for this document's setup

You are done with bootstrap when:
- monorepo structure exists and is clean
- root scripts orchestrate both apps
- strict TS is centrally defined and used in both apps
- contract package direction is chosen (`zod` runtime schemas)
- checks pass consistently from root

After this, we start implementing Phase 1 (Nest fundamentals) on top of a stable base.
