# Claude Code Agent: Session & Context Mechanics

## What Makes Up a Session's Context Window

Claude Code operates within a context window (200K tokens by default, or 1M with the `[1m]` model suffix — e.g., `sonnet[1m]`). Each API call assembles and sends the following layers:

### 1. System Prompt (~3–4K tokens)
Core identity instructions — tells Claude it's an interactive CLI coding assistant, defines behavior rules and safety guidelines.

### 2. Tool Definitions (~6–9K tokens)
Claude Code ships with 17 built-in tools: `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `Task` (for spawning subagents), `TodoWrite`, `WebFetch`, and others. Each tool includes a detailed JSON schema. **MCP server** tool definitions are added here too — each enabled server adds its schemas to this section.

### 3. Memory / CLAUDE.md Files (variable)
Project-specific instructions that persist across sessions, loaded from a hierarchy:
- `~/.claude/CLAUDE.md` — global, user-level
- `CLAUDE.md` in project root — project-level, shared with team
- `CLAUDE.local.md` — project-level, personal (auto-gitignored)
- `.claude/CLAUDE.md` — project-level (can be committed or kept personal)
- `.claude/rules/` — additional rule files (can have path-specific scoping)
- `~/.claude/rules/` — user-level rule files

CLAUDE.md files in parent directories above your working directory are loaded automatically. CLAUDE.md files in child directories load on-demand when Claude reads files there. Files can import other files using `@path/to/file` syntax.

### 4. Custom Subagent Definitions (small)
If custom subagents are defined in `.claude/agents/` or `~/.claude/agents/`, their descriptions are loaded so Claude knows when to delegate.

### 4b. Skill Descriptions (small)
If skills are defined in `.claude/skills/`, their descriptions are loaded (but full content loads on demand, not at startup).

### 5. Session Memory (past sessions)
Summaries of relevant past sessions are injected in a `<session-memory>` block. These are explicitly marked as potentially outdated — only brief summaries, not full transcripts.

### 6. Conversation History (the bulk)
Every user message, Claude's responses, tool calls and their results accumulate here:
- Your prompts
- Claude's reasoning and responses
- File contents returned by Read tool
- Bash command outputs
- Search/grep results
- Todo list state

### 7. Response Buffer (~40K tokens reserved)
Space reserved for Claude's next response, including extended thinking.

---

## Key Mechanics

- **Files are NOT pre-loaded.** Claude doesn't read your whole codebase upfront. It uses tools (Read, Grep, Glob) to pull in files on-demand. Each file read adds to the conversation history.

- **Compaction** kicks in automatically when context gets high. It summarizes the conversation, compressing older turns while preserving key decisions and code. Can be triggered manually with `/compact` — optionally with a focus: `/compact focus on the API changes`. Use `/context` to see what's consuming space.

- **Checkpointing** — every user prompt creates a new checkpoint (a snapshot of both conversation and file state). Press `Esc+Esc` or run `/rewind` to open the rewind menu with five options: restore code and conversation, restore conversation only, restore code only, **summarize from here** (compacts messages from that point forward), or cancel. Checkpoints persist across sessions and are auto-cleaned after 30 days. See [official checkpointing docs](https://code.claude.com/docs/en/checkpointing) for details.

- **Subagents (Task tool)** get their own isolated context window. Only a summary result returns to the main agent, keeping the parent context clean.

- **Todo lists** function as short-term memory within a session — a structured plan that persists through compaction, acting as a reminder of goals and progress.

- **Prompt caching** ensures the system prompt and tool definitions (unchanged between turns) are cached server-side to reduce cost and latency.

- **Skills** load on demand — Claude sees their descriptions at startup, but full content only enters context when a skill is actually used. This keeps the base context lean.

---

## Where the Context Lives

- **All state lives on your local machine.** Claude Code (the CLI client) manages conversation history, files, tool results, and session transcripts locally.

- **The API is stateless.** Anthropic's servers have zero memory between calls. Claude doesn't "remember" the previous turn — it re-reads the entire conversation every time.

- **Every API call sends the full assembled context.** The client stitches together system prompt + tools + CLAUDE.md + full conversation history + your new message into one request payload and sends it.

- **Prompt caching** is a server-side performance optimization (recognizing unchanged prefix tokens), not persistent memory. The cache expires if calls are spaced too far apart.

### Local persistence
- **Session transcripts** — stored in `~/.claude/` as JSONL files
- **CLAUDE.md files** — in your repo
- **Settings** — `~/.claude/settings.json` and `.claude/settings.json`
- **Todo/task files** — written to disk to survive compaction

---

## Within vs. Across Sessions

| | Within a Session | Across Sessions |
|---|---|---|
| **History sent** | Full conversation (every message, response, tool result) | Only brief summaries in `<session-memory>` |
| **Growth** | Accumulates until compaction compresses it | Fresh start with minimal carry-over |
| **State** | Grows with every turn | Previous transcripts stored locally as JSONL |

---

## Session Management

- **Sessions are per-directory.** Each new session is tied to where you launched Claude. When you resume, you only see sessions from that directory.
- **Name your sessions** with `/rename auth-refactor` so you can find them later with `claude --resume auth-refactor`.
- **Resume** with `claude --continue` (most recent) or `claude --resume` (session picker). Note: session-scoped permissions are NOT restored on resume — you'll need to re-approve them.
- **Fork** with `claude --continue --fork-session` to branch off and try a different approach without affecting the original session. If you resume the same session in multiple terminals without forking, messages from both get interleaved — use `--fork-session` for parallel work.
- **Parallel sessions** — use [git worktrees](https://git-scm.com/docs/git-worktree) to create separate directories for different branches, then run Claude in each one independently.

---

## Terminology Note

- **Prompt** — the full input payload assembled and sent to the API on each call
- **Context window** — the total token budget (e.g. 200K) that constrains both input and output
- **Context** — commonly used to refer to everything filling that window (prompt + response)