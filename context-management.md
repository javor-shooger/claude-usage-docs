# Claude Code: Context Management Strategies

Practical strategies for managing your context budget effectively — the difference between knowing how the engine works and knowing how to drive.

---

## The Core Problem

Claude Code has a **200K token context window** (or **1M** with the `[1m]` model suffix — e.g., `sonnet[1m]`; note: the `[1m]` suffix is observed behavior and may not apply to all models). Everything competes for that space:

- System prompt + tool definitions (~10–15K baseline)
- CLAUDE.md files + memory (~1–5K)
- Every message you send
- Every response Claude gives
- **Every tool result** — file contents, search results, command output

The context fills up as you work. When it's full, the session either compacts (loses detail) or must end. Managing context well means **getting more productive work done before that happens**.

---

## How Context Accumulates

### What costs the most

| Action | Typical Cost | Notes |
|---|---|---|
| Reading a large file | 1,000–10,000+ tokens | A 500-line file ≈ 2,000–4,000 tokens |
| Bash command output | 100–5,000+ tokens | Build output, logs, and test results can be huge |
| Grep/search results | 200–3,000 tokens | Depends on number of matches |
| Claude's response | 200–2,000 tokens | Longer explanations cost more |
| Your message | 50–500 tokens | Usually small |
| Tool call overhead | ~50 tokens each | The call structure itself |

**Key insight:** Tool results — especially file reads and command output — are by far the biggest context consumers. A single `Read` of a large file can cost more than dozens of conversation turns.

---

## Strategy 1: Start Fresh Between Tasks

The simplest and most effective context management technique: **run `/clear` between unrelated tasks** to reset context entirely.

When you finish one task and start something different, the accumulated context from the first task is pure overhead — it doesn't help with the new task and takes up space. `/clear` gives you a clean 200K budget instantly.

**When to `/clear`:**
- Switching from one bug to a different, unrelated one
- Finished exploring — now starting implementation
- Claude seems confused or going in circles
- You've corrected Claude more than twice on the same issue

**What survives `/clear`:** CLAUDE.md, auto memory (MEMORY.md), and session memory summaries all reload. The only thing lost is the current conversation history.

---

## Strategy 2: Read Selectively

**Don't read entire files when you only need part of them.**

| Instead of... | Do this... |
|---|---|
| "Show me the whole file" | "Show me lines 40–80 of that file" (Read with `offset`/`limit`) |
| "Read all the config files" | "Search for the specific setting I need" (Grep first) |
| "Open every file in src/" | "Find which file contains X" (Glob/Grep, then read only that file) |

**Pattern:** Search first (Grep/Glob are cheap), then read only what you need.

---

## Strategy 3: Use Subagents to Protect Context

Subagents (Task tool) get **their own isolated context window**. Only a compact summary returns to your main session.

This is the most powerful context management technique:

| Scenario | Without subagent | With subagent |
|---|---|---|
| Exploring how auth works | 20 file reads flood your context | Agent reads 20 files, returns a 200-token summary |
| Searching across the codebase | Dozens of Grep results pile up | Agent searches and synthesizes, returns findings |
| Running a complex investigation | Every step accumulates | Agent investigates, returns conclusion |

**When to use subagents:**
- Exploratory research ("how does X work in this codebase?")
- Broad searches across many files
- Any task where you need information but not the raw data
- When you're already deep into a session and context is getting tight

**When NOT to use subagents:**
- Simple, targeted reads (1–2 specific files)
- Quick Grep for a known term
- When you need to see the raw content yourself to make decisions

---

## Strategy 4: Compact at the Right Time

**Compaction** summarizes older conversation turns to free up space. It happens automatically when context usage gets high, or you can trigger it manually with `/compact`.

### What survives compaction well
- Todo list state (TodoWrite) — persists as structured data
- Key decisions and conclusions
- The general direction of work

### What gets lost or degraded
- Exact code snippets from earlier in the conversation
- Detailed file contents you read earlier
- Nuanced details from long discussions
- Specific line numbers and error messages

### When to manually compact
- **Before starting a new phase of work** — if you've finished exploring and are about to start implementing, compact to reclaim the exploration context
- **When you notice responses getting slower** — high context usage = more tokens processed = slower responses
- **After a big file dump** — if you had Claude read many files and you've absorbed the key info, compact to reclaim that space

### How to compact effectively
Say: `/compact` or "compact the conversation"

You can also provide a focus: `/compact focus on the auth refactor plan` — this hints at what the summary should prioritize preserving.

**Tip:** You can add compaction instructions to your CLAUDE.md to guide what gets preserved: e.g., `"When compacting, always preserve the full list of modified files and the current plan."`

---

## Strategy 5: Use Todo Lists as Persistent Memory

The todo list (TodoWrite) acts as **short-term structured memory** that survives compaction better than conversation text.

**Use it to anchor your session:**
- Before starting complex work, create a todo list with your goals
- The list persists through compaction, reminding Claude of the plan
- Mark items complete as you go — this keeps the session focused

**Example:** If you're fixing 5 bugs, create a todo list with all 5. Even after compaction, Claude will see the list and know which bugs are done and which remain.

---

## Strategy 6: Structure Your Requests

How you phrase things affects how much context gets used:

| Wasteful | Efficient |
|---|---|
| "Tell me everything about this codebase" | "What framework is used and where are the API routes?" |
| "Read all the test files" | "Find which test file covers the auth module" |
| "What's wrong with my code?" (no context) | "The login endpoint returns 401 — check the auth middleware" |

**Be specific.** Vague requests lead to broad exploration that fills context quickly. Targeted requests get answers with minimal context cost.

---

## Strategy 7: Use Skills for On-Demand Loading

Skills (`.claude/skills/`) only load their full content **when Claude determines they're relevant** or when you invoke them with a slash command. This keeps your base context lean.

| Approach | Context cost |
|---|---|
| Put everything in CLAUDE.md | Loaded every session, every turn |
| Move specialized knowledge to skills | Only loaded when needed |

**Example:** If you have API conventions that only matter when editing API routes, put them in a skill instead of CLAUDE.md. They'll load automatically when Claude works on API code, but won't consume context when you're working on the frontend.

**Zero-cost skills:** Add `disable-model-invocation: true` to a skill's frontmatter to prevent Claude from auto-loading it. The skill is only invoked when you call it explicitly with a slash command — zero context cost until then.

See [Extending Claude Code](extending-claude-code.md) for how to create skills.

---

## Strategy 8: Parallelize with Background Tasks

Background tasks (`run_in_background`) don't block your conversation flow:

- Start a long build/test run in the background
- Continue working on something else
- Check the result when needed

This doesn't save context directly, but it saves **time**, letting you use the session more efficiently.

---

## Strategy 9: Know When to Start Fresh

Sometimes the best context management is a **new session**:

- If you've shifted to a completely different task
- If the session has been heavily compacted and Claude seems to be losing thread
- If you're going in circles on a problem (fresh perspective helps)

Starting a new session costs you the accumulated context, but you get:
- A clean 200K budget
- Session memory summaries from the previous session
- CLAUDE.md and auto memory carrying forward the important bits

---

## Checking What's Using Context

| Command | What it shows |
|---|---|
| `/context` | Breakdown of what's consuming context space (messages, tools, system prompt) |
| `/mcp` | Per-server context cost of MCP tools — useful for identifying expensive MCP integrations |
| `/statusline` | Configure a persistent status line showing ongoing context usage at a glance |

**MCP tool search:** When MCP tools exceed ~10% of your total context, Claude Code automatically defers them — listing tools on demand instead of loading them all at startup. This is automatic, but if context feels tight, check `/mcp` to see if a server is consuming too much.

---

## Safety Net: Checkpointing

Claude Code creates a checkpoint with every prompt you send. If something goes wrong:

- **`Esc+Esc`** opens the rewind menu — restore code, restore conversation, or summarize from a selected message
- **`/rewind`** does the same thing

This makes aggressive strategies safer. You can let Claude make broad changes knowing you can always roll back. This doesn't save context, but it reduces the cost of mistakes — you don't need to spend turns debugging a bad edit when you can just rewind.

---

## Context Budget Mental Model

Think of your 200K context as a budget:

```
200K total context
 - 15K  baseline (system prompt, tools, CLAUDE.md)
 - 40K  reserved for response
 ───────
 145K  available for your session

 Comfortable zone:    0–80K used    (plenty of room)
 Watch zone:         80–130K used   (be selective with reads)
 Compact zone:      130–170K used   (consider compacting manually)
 Auto-compact:      ~190K used      (system compacts at ~95% capacity)
```

> **Note:** Auto-compaction triggers at roughly **95% of context capacity** (~190K for a 200K window). Don't wait for it — manually compact earlier with `/compact` for better control over what gets preserved.

---

## Quick Reference

| I want to... | Strategy |
|---|---|
| Reset context between tasks | `/clear` — simplest and most effective |
| Explore a large codebase | Use an Explore subagent |
| Find specific code | Grep first, then Read only the relevant file |
| Read a huge file | Use offset/limit to read only the section you need |
| Investigate a complex issue | Delegate to a subagent |
| Keep track of multi-step work | Use TodoWrite as persistent anchoring |
| Free up context mid-session | `/compact` (optionally with a focus hint) |
| Start a fundamentally different task | `/clear` or start a new session |
| Run a long command without blocking | Use `run_in_background` |
| Keep specialized knowledge out of base context | Move it to skills (`.claude/skills/`) |
| See what's consuming context | `/context` |
| Monitor context continuously | `/statusline` |
| Check MCP tool costs | `/mcp` |
| Undo a bad edit safely | `Esc+Esc` to rewind |

---

For the full context management reference, see the [official context documentation](https://code.claude.com/docs/en/how-claude-code-works).
