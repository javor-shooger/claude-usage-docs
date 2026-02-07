# Claude Code: Memory & Instruction Files

A detailed reference for all the files that give Claude Code persistent instructions and memory across sessions.

---

## Overview

Claude Code has **no built-in memory between sessions**. Every API call is stateless — the model re-reads everything from scratch each turn. To give Claude persistent knowledge about your preferences, project conventions, and lessons learned, Claude Code loads a set of instruction and memory files into the system prompt at the start of every session.

These files are the primary way you shape Claude's behavior beyond a single conversation.

---

## The CLAUDE.md Hierarchy

Claude Code loads CLAUDE.md files from multiple locations. They are **all additive** — nothing overrides anything else. Every file found is concatenated and injected into the system prompt.

### 1. `~/.claude/CLAUDE.md` — Global (User-Level)

| Property | Detail |
|---|---|
| **Location** | `~/.claude/CLAUDE.md` (your home directory) |
| **Scope** | Applies to **every project** you open with Claude Code |
| **Shared?** | No — personal to your machine |
| **Git tracked?** | No — lives outside any repo |
| **Created by** | You, manually or via `/init` |

**What to put here:**
- Your personal coding style preferences (e.g., "prefer single quotes", "use tabs not spaces")
- Global tool preferences (e.g., "always use pnpm, not npm")
- Language/framework defaults (e.g., "I work primarily in C# and TypeScript")
- Communication preferences (e.g., "be concise", "don't add emojis")

**Example:**
```markdown
- I prefer TypeScript with strict mode enabled
- Use pnpm for all Node.js projects
- When writing C#, follow the Microsoft naming conventions
- Keep responses concise — no unnecessary explanation
```

---

### 2. `CLAUDE.md` — Project Root (Team-Level)

| Property | Detail |
|---|---|
| **Location** | `CLAUDE.md` in the root of your project/repo |
| **Scope** | Applies to **this project only** |
| **Shared?** | Yes — commit it to git for the whole team |
| **Git tracked?** | Yes (should be) |
| **Created by** | You or your team, manually or via `/init` |

**What to put here:**
- Project architecture overview (folder structure, key patterns)
- Tech stack and versions (e.g., ".NET 8, React 18, PostgreSQL 16")
- Build and run commands (e.g., "run `npm run dev` to start the dev server")
- Testing conventions (e.g., "tests are in `__tests__/` folders, use Jest")
- Project-specific coding standards
- Common gotchas or things Claude should know about the codebase

**Example:**
```markdown
# Project: MyApp

## Tech Stack
- Backend: .NET 8 Web API
- Frontend: React 18 + TypeScript + Vite
- Database: PostgreSQL 16 with EF Core

## Build Commands
- Backend: `dotnet build src/MyApp.sln`
- Frontend: `cd client && pnpm install && pnpm dev`
- Tests: `dotnet test` and `pnpm test`

## Conventions
- API controllers go in `src/MyApp.Api/Controllers/`
- Use record types for DTOs
- All endpoints must have integration tests
```

---

### 3. `.claude/CLAUDE.md` — Project Personal (Your Overrides)

| Property | Detail |
|---|---|
| **Location** | `.claude/CLAUDE.md` in the project root |
| **Scope** | Applies to **this project only**, but only for **you** |
| **Shared?** | No — personal to your machine |
| **Git tracked?** | Typically no — add `.claude/` to `.gitignore` |
| **Created by** | You, manually |

**What to put here:**
- Personal overrides for this specific project
- Your local environment details (e.g., "my local DB is on port 5433")
- Work-in-progress notes or reminders
- Things that differ from the team setup

**Example:**
```markdown
- My local PostgreSQL runs on port 5433 (not default 5432)
- I'm currently working on the auth refactor in the feature/oauth branch
- Skip the E2E tests when running locally — they need Docker
```

---

### 4. `.claude/rules/` — Always-Loaded Rule Files

| Property | Detail |
|---|---|
| **Location** | `.claude/rules/` directory in the project root |
| **Scope** | Applies to **this project only** |
| **Shared?** | Up to you — can be committed to git |
| **Git tracked?** | Your choice |
| **Created by** | You or your team, manually |

**How it works:**
- Every file in this directory is loaded at startup
- Files can be `.md` or plain text
- Useful for splitting rules into logical groups rather than one big CLAUDE.md

**Example structure:**
```
.claude/
  rules/
    coding-standards.md
    testing-rules.md
    security-guidelines.md
```

**Example file (`.claude/rules/testing-rules.md`):**
```markdown
## Testing Rules
- Every new feature must have unit tests
- Use xUnit for C# tests, Jest for TypeScript
- Test file names must match: `{ClassName}Tests.cs` or `{module}.test.ts`
- Mock external services — never hit real APIs in tests
```

---

## Auto Memory System

Separate from CLAUDE.md files, Claude Code has an **auto memory** system that lets the agent itself record and retrieve notes across sessions.

### Location

```
~/.claude/projects/<project-identifier>/memory/
```

The `<project-identifier>` is derived from your project path (e.g., `d--my-project` for `d:\my-project`).

### MEMORY.md — The Primary Memory File

| Property | Detail |
|---|---|
| **Location** | `~/.claude/projects/<project-id>/memory/MEMORY.md` |
| **Loaded?** | **Always** — injected into the system prompt at session start |
| **Size limit** | Lines after 200 are truncated — keep it concise |
| **Written by** | Claude (the agent), not you |
| **Purpose** | Key learnings, patterns, and insights from past sessions |

Claude is instructed to:
- Record mistakes and lessons learned
- Note patterns that worked or failed
- Update or remove outdated memories
- Keep it organized by topic, not chronologically

### Topic Files — Extended Memory

| Property | Detail |
|---|---|
| **Location** | `~/.claude/projects/<project-id>/memory/*.md` (e.g., `debugging.md`, `patterns.md`) |
| **Loaded?** | **Not automatically** — only MEMORY.md is auto-loaded |
| **Written by** | Claude, when detailed notes exceed what fits in MEMORY.md |
| **Purpose** | Detailed notes on specific topics, linked from MEMORY.md |

**How they work together:**
- `MEMORY.md` acts as an index — concise summaries with links to topic files
- Topic files hold the detail (e.g., `debugging.md` might have notes on tricky bugs encountered)
- Claude reads topic files on-demand when the content in MEMORY.md points to them

**Example MEMORY.md:**
```markdown
## Project Patterns
- Uses repository pattern for data access — see [patterns.md](patterns.md)
- Auth is JWT-based with refresh tokens

## Known Issues
- EF Core migrations must be run manually after pulling — see [debugging.md](debugging.md)
- The legacy API module has circular dependencies — avoid modifying without a plan
```

---

## Session Memory (Cross-Session Summaries)

This is separate from both CLAUDE.md and auto memory.

| Property | Detail |
|---|---|
| **Storage** | Session transcripts saved as JSONL files in `~/.claude/` |
| **Injected as** | `<session-memory>` blocks in the system prompt |
| **Content** | Brief summaries of relevant past sessions |
| **Reliability** | Explicitly marked as potentially outdated — not full transcripts |

**Key points:**
- You don't control what goes into session memory — it's automatic
- Only brief summaries are injected, not full conversation history
- A new session gets a fresh start with just these summaries for continuity
- Session memory helps Claude recall what you were working on recently, but should not be relied upon for precise details

---

## How Everything Loads Together

When a Claude Code session starts, the system prompt is assembled in this order:

```
1. Core system prompt (identity, behavior rules, safety)
2. Tool definitions (built-in + MCP)
3. ── Memory & Instructions ──────────────────────
   a. ~/.claude/CLAUDE.md              (global preferences)
   b. CLAUDE.md (project root)         (team/project instructions)
   c. .claude/CLAUDE.md                (personal project overrides)
   d. .claude/rules/*                  (all rule files)
   e. Auto memory (MEMORY.md)          (agent's learned notes)
   f. Session memory                   (past session summaries)
4. Custom agent definitions (.claude/agents/)
5. Conversation history begins...
```

### Important behaviors:

- **All files are additive.** Nothing overrides — they stack. If your global CLAUDE.md says "use tabs" and the project CLAUDE.md says "use spaces", Claude sees both instructions and the more specific/recent one typically wins in practice.
- **All files consume context.** Every line in every CLAUDE.md file, rule file, and MEMORY.md eats into your 200K token budget. Keep them concise.
- **Loading happens once at session start.** Changes to CLAUDE.md files mid-session are not picked up until the next session (or until compaction re-reads them).

---

## Context Cost

| Source | Typical Size | Notes |
|---|---|---|
| Global `~/.claude/CLAUDE.md` | 100–500 tokens | Usually small |
| Project `CLAUDE.md` | 200–2,000 tokens | Depends on project complexity |
| `.claude/CLAUDE.md` | 100–500 tokens | Usually small |
| `.claude/rules/*` combined | 200–2,000 tokens | Depends on how many rules |
| `MEMORY.md` | 100–1,000 tokens | Capped at 200 lines |
| Session memory | 200–1,000 tokens | Varies by history |
| **Typical total** | **~500–5,000 tokens** | **~0.25–2.5% of 200K context** |

---

## Quick Reference

| File | Scope | Shared? | Git? | Loaded | Written by |
|---|---|---|---|---|---|
| `~/.claude/CLAUDE.md` | All projects | No | No | Always | You |
| `CLAUDE.md` (project root) | This project | Yes (team) | Yes | Always | You / team |
| `.claude/CLAUDE.md` | This project | No (personal) | No | Always | You |
| `.claude/rules/*` | This project | Your choice | Your choice | Always | You / team |
| `~/.claude/projects/<id>/memory/MEMORY.md` | This project | No | No | Always | Claude (agent) |
| `~/.claude/projects/<id>/memory/*.md` | This project | No | No | On demand | Claude (agent) |
| Session transcripts (JSONL) | This project | No | No | Summaries only | Automatic |

---

## Best Practices

1. **Keep CLAUDE.md files concise.** Every line costs context tokens. Use bullet points, not paragraphs. Aim for "instructions", not "documentation".

2. **Use the right file for the right scope:**
   - Personal habits → `~/.claude/CLAUDE.md`
   - Team standards → `CLAUDE.md` in project root
   - Your local quirks → `.claude/CLAUDE.md`
   - Organized rules → `.claude/rules/`

3. **Commit the project-root `CLAUDE.md` to git.** It's the most valuable file — it gives every team member's Claude the same project context.

4. **Add `.claude/` to `.gitignore`** (or at minimum `.claude/CLAUDE.md`) to keep personal overrides out of version control.

5. **Don't duplicate information.** If the project README already explains the build process, reference it in CLAUDE.md rather than repeating it: "See README.md for build instructions."

6. **Review auto memory periodically.** Check `~/.claude/projects/<id>/memory/MEMORY.md` occasionally — Claude writes it, and it may accumulate stale or incorrect notes over time.

7. **Use `.claude/rules/` for long or structured rule sets.** If your CLAUDE.md is getting long, split rules into separate files in the rules directory.
