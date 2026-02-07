# Claude Code Agent: Session & Context Mechanics

## What Makes Up a Session's Context Window

Claude Code operates within a context window (typically 200K tokens, or 1M in beta). Each API call assembles and sends the following layers:

### 1. System Prompt (~3–4K tokens)
Core identity instructions — tells Claude it's an interactive CLI coding assistant, defines behavior rules and safety guidelines.

### 2. Tool Definitions (~6–9K tokens)
Claude Code ships with 17 built-in tools: `Read`, `Write`, `Edit`, `Bash`, `Grep`, `Glob`, `Task` (for spawning subagents), `TodoWrite`, `WebFetch`, and others. Each tool includes a detailed JSON schema. **MCP server** tool definitions are added here too — each enabled server adds its schemas to this section.

### 3. Memory / CLAUDE.md Files (variable)
Project-specific instructions that persist across sessions, loaded from a hierarchy:
- `~/.claude/CLAUDE.md` — global, user-level
- `CLAUDE.md` in project root — project-level, shared with team
- `.claude/CLAUDE.md` — project-level, personal
- `.claude/rules/` — additional rule files, always loaded at startup

These provide persistent context about project conventions, tech stack, and workflows.

### 4. Custom Agent Definitions (small)
If custom agents are defined in `.claude/agents/`, their descriptions and tool lists are loaded so Claude knows when to delegate.

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

- **Compaction** kicks in automatically around 75–92% context usage. A lighter model (Haiku) summarizes the conversation, compressing older turns while preserving key details. Can be triggered manually with `/compact`.

- **Subagents (Task tool)** get their own isolated context window. Only a summary result returns to the main agent, keeping the parent context clean.

- **Todo lists** function as short-term memory within a session — a structured plan that persists through compaction, acting as a reminder of goals and progress.

- **Prompt caching** ensures the system prompt and tool definitions (unchanged between turns) are cached server-side to reduce cost and latency.

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

## Terminology Note

- **Prompt** — the full input payload assembled and sent to the API on each call
- **Context window** — the total token budget (e.g. 200K) that constrains both input and output
- **Context** — commonly used to refer to everything filling that window (prompt + response)