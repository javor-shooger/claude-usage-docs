# Claude Code: Writing Effective CLAUDE.md Files

*Last verified against [official docs](https://code.claude.com/docs/en/) on 2026-02-09*

How to write instructions that actually improve Claude's behavior — not just fill space.

---

## The Goal

A good CLAUDE.md file makes Claude behave as if it already knows your project. It should answer the questions Claude would otherwise ask or guess wrong about:

- What's the tech stack?
- How do I build and test this?
- What patterns does the codebase follow?
- What should I avoid doing?

**Tip:** Run `/init` to generate a starter CLAUDE.md based on your project structure. It analyzes your codebase to detect build systems, test frameworks, and code patterns. Then refine it over time.

---

## Principles

### 1. Instructions, Not Documentation

CLAUDE.md is for telling Claude **what to do**, not explaining how your code works. Claude can read your code — it needs to know your conventions and preferences.

| Documentation (wrong) | Instruction (right) |
|---|---|
| "The auth module uses JWT tokens with a 1-hour expiry. Tokens are generated in the authService using the jsonwebtoken library..." | "Use JWT for auth. Tokens expire in 1 hour. Don't change the expiry without discussing it." |
| "Our testing framework is Jest. We use React Testing Library for component tests. Test files are colocated..." | "Use Jest + React Testing Library. Tests go next to the files they test: `Component.test.tsx`." |

### 2. Concise Over Complete

Every line consumes context tokens. Say more with less.

| Wordy | Concise |
|---|---|
| "When you are writing TypeScript code in this project, please make sure to always use strict mode and enable all strict compiler options." | "TypeScript: strict mode, always." |
| "For all database operations, please use the repository pattern. Create a repository class for each entity that extends the BaseRepository." | "Data access: use repository pattern. Extend `BaseRepository`." |

### 3. Actionable Over Descriptive

Claude needs to know what to **do**, not just what exists.

| Descriptive (less useful) | Actionable (more useful) |
|---|---|
| "We have a CI/CD pipeline." | "Run `npm test` and `npm run build` before committing — CI will fail otherwise." |
| "The project uses ESLint." | "Run `npm run lint` after changes. Fix lint errors, don't disable rules." |
| "We follow REST conventions." | "API endpoints: use plural nouns (`/users`, `/orders`), standard HTTP methods, return 2xx/4xx/5xx appropriately." |

### 4. Specific Over General

Generic advice is ignored. Specific rules are followed.

| Generic | Specific |
|---|---|
| "Write clean code" | "Max function length: 30 lines. Extract if longer." |
| "Follow best practices" | "Always handle errors with try/catch. Log errors with `logger.error()`, never `console.log()`." |
| "Write tests" | "Every new function needs a test. Use the `describe/it` pattern. Aim for edge cases, not just happy path." |

---

## What to Include

### Must-have (for any project)

```markdown
## Tech Stack
- Backend: Node.js 20 + Express + TypeScript
- Database: PostgreSQL 16 + Prisma ORM
- Frontend: React 18 + Vite + TailwindCSS
- Tests: Jest + React Testing Library

## Build & Run
- Install: `pnpm install`
- Dev: `pnpm dev` (starts on localhost:3000)
- Build: `pnpm build`
- Test: `pnpm test`
- Lint: `pnpm lint`
```

This alone saves Claude from having to explore your package.json, tsconfig, and build scripts.

> **Tip:** Including verification instructions is the single highest-leverage thing you can add. "Run `pnpm test` after changes" or "Run `dotnet build` to verify" lets Claude catch its own mistakes before reporting back.

### Highly recommended

```markdown
## Conventions
- Use named exports, not default exports
- API routes: `/api/v1/{resource}` (plural, kebab-case)
- Database: migrations in `prisma/migrations/`, seed in `prisma/seed.ts`
- Environment: `.env.local` for secrets (never commit)

## Patterns
- Services handle business logic (`src/services/`)
- Controllers handle HTTP (`src/controllers/`)
- Use Zod for request validation
- Errors: throw `AppError` from `src/lib/errors.ts`

## Don'ts
- Don't use `any` type — use `unknown` and narrow
- Don't write SQL directly — use Prisma
- Don't add new dependencies without asking first
```

### Optional but useful

```markdown
## Current Work
- Currently refactoring auth from sessions to JWT (branch: feature/jwt-auth)
- The legacy API in `src/api-v1/` is deprecated — don't modify it

## Known Issues
- The E2E tests are flaky on CI — retry once before investigating
- `src/utils/legacy.ts` has circular dependency issues — avoid importing from it
```

---

## What NOT to Include

| Don't include | Why |
|---|---|
| Full API documentation | Too long, burns context. Reference the docs instead: "See `docs/api.md` for endpoints." |
| Code examples for every pattern | Claude can read your actual code. Just name the pattern and point to a reference file. |
| History or changelog | Claude doesn't need to know what changed last sprint. |
| Obvious things | "Use git for version control" — Claude already knows. |
| Duplicated info | If it's in your README, reference it: "See README.md for setup instructions." |
| Comments about the CLAUDE.md itself | "This file tells Claude how to work" — wastes tokens. |
| Specialized workflow instructions | Move these to skills (`.claude/skills/`) — they load on demand instead of every session. See [Extending Claude Code](extending-claude-code.md). |

---

## Sizing Guide

| File | Ideal size | Max recommended |
|---|---|---|
| `~/.claude/CLAUDE.md` (global) | 5–15 lines | 30 lines |
| `CLAUDE.md` (project root) | 15–50 lines | 100 lines |
| `.claude/CLAUDE.md` (project-level) | 5–15 lines | 30 lines |
| Each file in `.claude/rules/` | 10–30 lines | 50 lines |

**Total across all files:** Try to stay under 150 lines combined. Every line beyond what's useful is wasted context.

> **Warning:** If Claude keeps doing something you've explicitly told it not to, the file is probably too long and the rule is getting lost in noise. Trim aggressively. You can also emphasize critical rules with **IMPORTANT** or **YOU MUST** to improve adherence.

---

## Examples

### Good: Small API Project

```markdown
## Tech Stack
Node.js 20, Express, TypeScript, PostgreSQL, Prisma

## Commands
- Dev: `pnpm dev`
- Test: `pnpm test`
- Build: `pnpm build`
- Migrate: `pnpm prisma migrate dev`

## Rules
- Strict TypeScript — no `any`
- All endpoints need request validation (use Zod)
- Errors: throw `AppError`, never raw Error
- Tests: colocated, `*.test.ts`, Jest

## Structure
- `src/routes/` — Express route definitions
- `src/services/` — business logic
- `src/middleware/` — Express middleware
- `prisma/schema.prisma` — database schema
```

### Good: Large Monorepo

```markdown
## Overview
Monorepo with 3 packages: `api`, `web`, `shared`

## Commands
- Install: `pnpm install` (from root)
- API: `pnpm --filter api dev`
- Web: `pnpm --filter web dev`
- Test all: `pnpm test`
- Test one: `pnpm --filter api test`

## Rules
- Shared types go in `packages/shared/`
- API and Web import from shared, never from each other
- Each package has its own tsconfig extending root
- PRs need tests for changed packages

## API Conventions
- REST, versioned at `/api/v1/`
- Use DTOs from `shared/types/`
- Repository pattern for data access

## Web Conventions
- Next.js App Router
- Server Components by default, Client Components only when needed
- Styling: Tailwind only, no CSS modules
```

### Bad: Too Long and Vague

```markdown
# Project Overview

This is a web application that we have been building for the past 6 months.
The team consists of 4 developers and we follow agile methodology with
two-week sprints. Our product owner is Sarah and she reviews PRs on Mondays...

## Architecture
The application follows a microservices architecture pattern where each
service is responsible for a specific domain. The services communicate
through a message queue (RabbitMQ) and also expose REST APIs for
synchronous communication. The frontend is a single-page application
built with React that communicates with the API gateway which routes
requests to the appropriate microservice...

[continues for 200+ more lines]
```

**Why it's bad:** Most of this is background that Claude doesn't need. It wastes context and buries the useful instructions.

---

## Maintaining Your CLAUDE.md

- **Update when conventions change.** If you switch from npm to pnpm, update the file.
- **Remove stale info.** If you finished the auth refactor, remove the "currently working on" note.
- **Review quarterly.** Skim through and ask: "Is Claude still following this? Is anything outdated?"
- **Check after issues.** If Claude does something wrong repeatedly, add a rule to prevent it.
- **Use `@` imports for large references.** Instead of pasting docs into CLAUDE.md, import them: `See @docs/api.md for API conventions.` Imports resolve relative to the CLAUDE.md file.

---

## Quick Reference

| Question | Answer |
|---|---|
| How long should it be? | 15–50 lines for the project CLAUDE.md |
| What format? | Bullet points and short lines — not paragraphs |
| What goes in global vs project? | Personal prefs → global. Project rules → project. |
| Should I commit it? | Yes — project root `CLAUDE.md` should be in git |
| How often to update? | When conventions change or Claude gets something wrong |
| What's the biggest mistake? | Writing documentation instead of instructions |

---

For the full official guide on CLAUDE.md best practices, see the [official best practices documentation](https://code.claude.com/docs/en/best-practices#write-an-effective-claudemd).
