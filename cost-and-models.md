# Claude Code: Cost Awareness & Model Selection

Understanding token costs, choosing the right model, and keeping spending efficient.

---

## How Costs Work

Claude Code charges based on **tokens processed**. Every API call sends the full assembled context (system prompt + tools + CLAUDE.md + conversation history + your message) and receives a response. You pay for both:

- **Input tokens** — everything sent to the API (the full context)
- **Output tokens** — Claude's response (including tool calls and reasoning)

### Typical Costs

- Average: **~$6 per developer per day** (API users)
- 90th percentile: under $12/day
- Monthly with Sonnet: **~$100–200/developer**
- Subscription users (Pro, Max, Teams): usage included in subscription — use `/stats` to view patterns instead of `/cost`.

### Why Costs Grow During a Session

Each turn sends the **entire conversation history** again. Turn 1 might send 15K tokens. Turn 10 might send 80K tokens — because it includes everything from turns 1–9 plus all tool results.

```
Turn  1: 15K input  →  response
Turn  5: 45K input  →  response  (includes turns 1-4)
Turn 10: 80K input  →  response  (includes turns 1-9)
Turn 20: 150K input →  response  (includes turns 1-19, or compacted)
```

**Key insight:** Late-session turns are significantly more expensive than early ones because the input payload is larger.

### Prompt Caching Helps

Claude Code uses **prompt caching** — the server recognizes unchanged portions of the input (system prompt, tool definitions, CLAUDE.md, earlier conversation) and reuses cached computations. Cached tokens cost significantly less than fresh tokens.

**What this means for you:**
- Rapid back-and-forth conversation is cheaper per-token than spaced-out interactions (cache stays warm)
- The system prompt + tools + CLAUDE.md are cached across turns (you don't pay full price every time)
- If you step away for a while, the cache may expire, and the next turn costs more

---

## The Models

Claude Code supports multiple models that you can switch between. They differ in capability, speed, and cost.

| Model | Strengths | Weaknesses | Cost | Best For |
|---|---|---|---|---|
| **Opus 4.6** | Most capable reasoning, adaptive thinking, best at complex tasks | Slowest, most expensive | Highest | Complex features, architectural decisions, tricky bugs |
| **Sonnet 4.5** | Good balance of capability and speed, handles most tasks well | Less capable on very complex reasoning | Medium | General development — the recommended default |
| **Haiku 4.5** | Fastest, cheapest, good at straightforward tasks | Less capable on complex reasoning | Lowest | Simple edits, running commands, quick questions |

### Model Aliases

| Alias | Points To |
|---|---|
| `default` | The default model for your account (usually Sonnet) |
| `sonnet` | Latest Claude Sonnet |
| `opus` | Latest Claude Opus |
| `haiku` | Latest Claude Haiku |
| `sonnet[1m]` | Sonnet with 1M token context window (5x the default 200K) |
| `opusplan` | Automatically uses Opus during plan mode, then switches to Sonnet for implementation |

Set via `/model` — e.g., `/model opusplan`.

### Model Configuration Priority

When multiple model settings exist, this is the resolution order (highest priority first):

1. **`/model` during session** — overrides everything for this session
2. **`claude --model` at startup** — sets the model for this session
3. **`ANTHROPIC_MODEL` environment variable** — default for all sessions
4. **`model` field in settings file** — persistent default

### Switching Models

Use `/model` to switch mid-session. Common strategy:

```
Start with Opus for planning and complex decisions
  ↓
Switch to Sonnet for implementation
  ↓
Switch to Haiku for running tests, formatting, simple fixes
```

Or use `/model opusplan` to automate this — it handles the Opus→Sonnet switching for you.

### Extended Thinking

Extended thinking is **enabled by default** — Claude reasons through complex problems step-by-step before responding. This uses additional tokens but significantly improves quality for complex tasks.

| Setting | How |
|---|---|
| **Adjust depth** | `/model` → set effort level: low, medium, high (default) |
| **Toggle on/off** | `Alt+T` (Windows/Linux) or `Option+T` (macOS) |
| **View thinking** | `Ctrl+O` toggles verbose mode to see reasoning |
| **Disable globally** | `/config` → toggle thinking mode |

On Opus 4.6, thinking uses **adaptive reasoning** — the model dynamically allocates thinking tokens based on your effort level. On other models, thinking uses a fixed budget of up to **31,999 tokens**.

**Thinking tokens are billed as output tokens.** For simple tasks where deep reasoning isn't needed, lower the effort level or disable thinking to save costs.

You can control the thinking budget with the `MAX_THINKING_TOKENS` environment variable — e.g., `MAX_THINKING_TOKENS=8000` for a lower budget.

### Fast Mode

Toggle with `/fast`. Uses the **same Claude Opus 4.6 model** (not a different model) but optimizes for faster output. The setting persists across sessions.

**Key details:**
- Faster output speed at the same quality level
- Useful when speed matters more than thoroughness
- Auto-switches back to standard Opus 4.6 if fast mode encounters issues

---

## What's Expensive

### High-cost actions

| Action | Why it's expensive |
|---|---|
| Reading large files | Entire file content goes into context and stays there for the rest of the session |
| Long Bash output | Build logs, test output, verbose commands — all added to context |
| Many tool calls in one turn | Each call adds input + output tokens |
| Deep sessions (20+ turns) | Each turn re-sends the growing conversation history |
| Broad exploration | "Read every file in src/" burns context and tokens fast |

### Low-cost actions

| Action | Why it's cheap |
|---|---|
| Targeted Grep/Glob | Returns just file names or matching lines, not full file contents |
| Subagents | Their internal context is separate — only a compact summary returns |
| Short, specific questions | Small input, small output |
| Compaction | Reduces the conversation size, making subsequent turns cheaper |
| Cached turns | Rapid conversation benefits from prompt caching |

---

## Cost Optimization Strategies

### 1. Use the right model for the task

Don't use Opus for running `npm test`. Don't use Haiku for designing your architecture. Match the model to the task complexity.

| Task complexity | Model |
|---|---|
| Simple: formatting, renaming, running commands | Haiku |
| Standard: typical features, bug fixes, code review | Sonnet |
| Complex: architecture, tricky logic, multi-file reasoning | Opus |

### 2. Search before reading

```
Expensive: "Read all files in src/services/"
Cheap:     "Search for the email service" → then read only that file
```

Grep and Glob return compact results. Read returns full file contents. Always search first.

### 3. Use subagents for exploration

Subagents have isolated context. Their file reads and searches don't inflate your main session:

```
Expensive: "Read these 10 files and explain how auth works"
Cheap:     "Use an explore agent to explain how auth works"
           → agent reads 10 files internally, returns a 200-token summary
```

### 4. Compact when context is bloated

After heavy exploration or many file reads, `/compact` compresses the history. Subsequent turns send less data = lower cost.

### 5. Start fresh for new tasks

If you're switching to a completely different task, a new session is often cheaper than continuing a bloated one. The new session starts with minimal context instead of re-sending a long history.

### 6. Be specific in prompts

Vague prompts cause Claude to explore broadly (reading many files, trying multiple approaches). Specific prompts go straight to the target.

### 7. Limit verbose output

If running a command with potentially large output:
- Run tests for a specific file rather than the whole suite
- Pipe long output through truncation if possible
- Ask Claude to summarize rather than dump

---

## Checking Your Costs

| Command | What it shows |
|---|---|
| `/cost` | Token usage and cost for the current session |
| `/status` | Version, model, account, and connectivity info |

Check `/cost` periodically, especially during long sessions, to understand where your budget is going.

> **Note:** Claude Code uses a small number of tokens for background processing (conversation summarization for `--resume`, status checks) even when idle — typically under **$0.04 per session**.

---

## Mental Model for Cost

Think of it as a funnel:

```
Early turns:  cheap (small context, cached prefix)
     ↓
Mid session:  moderate (growing context, most cached)
     ↓
Late session: expensive (large context, approaching limits)
     ↓
After compact: drops back to moderate (compressed history)
```

The most cost-efficient pattern is:
1. Start focused — don't over-explore
2. Work through tasks systematically
3. Compact when context gets heavy
4. Start a new session when shifting to unrelated work

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Use a cheaper model | `/model` → select Haiku for simple tasks |
| Check current costs | `/cost` |
| Reduce cost mid-session | `/compact` to compress history |
| Avoid expensive file reads | Search with Grep/Glob first, then read only what's needed |
| Explore without inflating context | Use a subagent: "Use an explore agent to..." |
| Keep turns cheap | Be specific, scope your requests, avoid vague exploration |
| Start cheap again | New session for a new task |

---

For detailed cost management strategies (hooks for preprocessing, team rate limits, agent team costs), see the [official cost documentation](https://code.claude.com/docs/en/costs).
