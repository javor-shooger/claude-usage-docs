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

### 3. `CLAUDE.local.md` — Project Personal (Recommended for Personal Prefs)

| Property | Detail |
|---|---|
| **Location** | `CLAUDE.local.md` in the project root |
| **Scope** | Applies to **this project only**, but only for **you** |
| **Shared?** | No — **automatically added to .gitignore** |
| **Git tracked?** | No (auto-gitignored) |
| **Created by** | You, manually |

This is the recommended way to add personal project-specific preferences. Because it's auto-gitignored, you don't need to worry about accidentally committing it.

**Example:**
```markdown
- My local PostgreSQL runs on port 5433 (not default 5432)
- I'm currently working on the auth refactor in the feature/oauth branch
- Skip the E2E tests when running locally — they need Docker
```

### 3b. `.claude/CLAUDE.md` — Project (Alternative Location)

| Property | Detail |
|---|---|
| **Location** | `.claude/CLAUDE.md` in the project |
| **Scope** | Applies to **this project only** |
| **Shared?** | Your choice — you can commit it to git |
| **Git tracked?** | Your choice |

This is an alternative location for project-level instructions, equivalent to the root `CLAUDE.md`. Some teams prefer it to keep the project root clean. Either location works.

---

### 4. `.claude/rules/` — Modular Rule Files

| Property | Detail |
|---|---|
| **Location** | `.claude/rules/` directory in the project root |
| **Scope** | Applies to **this project only** |
| **Shared?** | Up to you — can be committed to git |
| **Git tracked?** | Your choice |
| **Created by** | You or your team, manually |

**How it works:**
- All `.md` files are loaded at startup (recursively, including subdirectories)
- Useful for splitting rules into logical groups rather than one big CLAUDE.md
- Can be organized into subdirectories (e.g., `rules/frontend/`, `rules/backend/`)

**Example structure:**
```
.claude/
  rules/
    coding-standards.md
    testing-rules.md
    security-guidelines.md
```

**Path-specific rules** — you can scope rules to specific files using YAML frontmatter:

```markdown
---
paths:
  - "src/api/**/*.ts"
---
# API Development Rules
- All API endpoints must include input validation
- Use the standard error response format
```

Rules without a `paths` field apply to all files. Glob patterns like `**/*.ts`, `src/**/*`, and `src/**/*.{ts,tsx}` are supported.

### 4b. `~/.claude/rules/` — User-Level Rules

Personal rules that apply across all your projects. Same format as project rules, but stored in your home directory. User-level rules load before project rules (project rules take priority).

---

## CLAUDE.md Imports

CLAUDE.md files can import other files using `@path/to/file` syntax:

```markdown
See @README.md for project overview and @package.json for available npm commands.

# Additional Instructions
- Git workflow: @docs/git-instructions.md
- Personal overrides: @~/.claude/my-project-instructions.md
```

- Relative paths resolve relative to the file containing the import
- Absolute and home-directory (`~/`) paths are supported
- Imported files can recursively import others (max depth: 5)
- Imports inside code blocks are ignored
- First time Claude encounters imports in a project, it shows an approval dialog. **This is a one-time decision per project** — once declined, the dialog does not resurface and imports remain disabled.

This is useful for keeping CLAUDE.md concise by referencing existing docs instead of duplicating them.

> **Tip:** Use `--add-dir ../shared` to give Claude access to additional directories. Set `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` to also load CLAUDE.md files from those directories.

---

## Auto Memory System

Separate from CLAUDE.md files, Claude Code has an **auto memory** system that automatically saves useful context — project patterns, key commands, your preferences, and lessons learned — across sessions. You can also tell Claude to save specific things, and you can edit the memory files directly at any time.

> **Note:** Auto memory is being rolled out gradually. If you don't see it, opt in by setting `CLAUDE_CODE_DISABLE_AUTO_MEMORY=0` in your environment.

### Location

```
~/.claude/projects/<project-identifier>/memory/
```

The `<project-identifier>` is derived from the **git repository root**, so all subdirectories within the same repo share one auto memory directory. Outside a git repo, the working directory is used instead.

### MEMORY.md — The Primary Memory File

| Property | Detail |
|---|---|
| **Location** | `~/.claude/projects/<project-id>/memory/MEMORY.md` |
| **Loaded?** | **Always** — injected into the system prompt at session start |
| **Size limit** | Lines after 200 are truncated — keep it concise |
| **Written by** | Claude (the agent) — you can also edit it directly |
| **Purpose** | Key learnings, patterns, and insights from past sessions |

Claude is instructed to:
- Record mistakes and lessons learned
- Note patterns that worked or failed
- Update or remove outdated memories
- Keep it organized by topic, not chronologically

You can also tell Claude to save something specific: "remember that we use pnpm, not npm" or "save to memory that the API tests require a local Redis instance."

Use `/memory` to open a file selector that includes your auto memory and CLAUDE.md files — pick one to edit directly.

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
   a. Managed policy CLAUDE.md          (organization-wide, if deployed)
   b. ~/.claude/CLAUDE.md               (global preferences)
   c. ~/.claude/rules/*                 (user-level rules)
   d. CLAUDE.md (project root + parents)(team/project instructions)
   e. CLAUDE.local.md                   (personal project prefs, auto-gitignored)
   f. .claude/CLAUDE.md                 (project-level, alternative location)
   g. .claude/rules/*                   (project rule files)
   h. Auto memory (MEMORY.md)           (agent's learned notes)
   i. Session memory                    (past session summaries)
4. Skill descriptions (loaded as summaries — full content loads on demand)
5. Custom subagent definitions (.claude/agents/)
6. Conversation history begins...
```

**CLAUDE.md files in child directories** (e.g., `src/api/CLAUDE.md`) are NOT loaded at startup — they load on demand when Claude reads files in those directories.

### Important behaviors:

- **All files are additive.** Nothing overrides — they stack. More specific instructions generally take precedence in practice.
- **All files consume context.** Every line eats into your context budget. Keep them concise.
- **Parent directory loading** — Claude reads CLAUDE.md files recursively from your working directory up to (but not including) the root. Useful in monorepos where both `root/CLAUDE.md` and `root/packages/api/CLAUDE.md` apply.
- **Use `/memory`** to see and edit all loaded memory files.

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
| `~/.claude/rules/*` | All projects | No | No | Always | You |
| `CLAUDE.md` (project root) | This project | Yes (team) | Yes | Always | You / team |
| `CLAUDE.local.md` | This project | No (personal) | Auto-gitignored | Always | You |
| `.claude/CLAUDE.md` | This project | Your choice | Your choice | Always | You |
| `.claude/rules/*` | This project | Your choice | Your choice | Always | You / team |
| Child dir `CLAUDE.md` | Subdirectory | Your choice | Your choice | On demand | You / team |
| `~/.claude/projects/<id>/memory/MEMORY.md` | This project | No | No | Always (first 200 lines) | Claude (agent) |
| `~/.claude/projects/<id>/memory/*.md` | This project | No | No | On demand | Claude (agent) |
| Session transcripts (JSONL) | This project | No | No | Summaries only | Automatic |

---

## Best Practices

1. **Keep CLAUDE.md files concise.** Every line costs context tokens. Use bullet points, not paragraphs. Aim for "instructions", not "documentation".

2. **Use the right file for the right scope:**
   - Personal habits → `~/.claude/CLAUDE.md`
   - Team standards → `CLAUDE.md` in project root
   - Your local quirks → `CLAUDE.local.md` (auto-gitignored)
   - Organized rules → `.claude/rules/`

3. **Commit the project-root `CLAUDE.md` to git.** It's the most valuable file — it gives every team member's Claude the same project context.

4. **Use `CLAUDE.local.md` for personal project prefs.** It's auto-gitignored, so no risk of committing it accidentally.

5. **Use `@` imports to keep CLAUDE.md lean.** Instead of pasting your API docs into CLAUDE.md, reference them: `See @docs/api.md for endpoint specifications.`

6. **Move specialized instructions to skills.** If some instructions only matter for specific workflows (e.g., deployment checklists), put them in `.claude/skills/` instead of CLAUDE.md. Skills load on demand, CLAUDE.md loads every session.

7. **Review auto memory periodically.** Use `/memory` to check MEMORY.md — Claude writes it, and it may accumulate stale or incorrect notes over time.

8. **Use `.claude/rules/` for long or structured rule sets.** If your CLAUDE.md is getting long, split rules into separate files. Use path-specific rules (YAML `paths` frontmatter) to scope rules to relevant files.

For the full memory configuration options, see the [official memory documentation](https://code.claude.com/docs/en/memory).
