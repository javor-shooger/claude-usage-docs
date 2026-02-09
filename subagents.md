# Claude Code: Subagents (Task Tool) Deep Dive

How the Task tool works, its agent types, and when to use each one.

---

## What Are Subagents?

Subagents are **independent Claude instances** spawned by the main agent via the Task tool. Each one gets:

- Its own **isolated context window** (separate from the parent)
- A specific **set of tools** based on its type
- A **single task prompt** to work on autonomously

When done, only a **compact result** returns to the parent — the subagent's full exploration, file reads, and intermediate steps stay in its own context and don't pollute yours.

---

## Why This Matters

The isolated context is the key benefit:

```
Main session (200K context)
│
├── You: "How does caching work in this project?"
│
├── Subagent spawned (gets its own 200K)
│   ├── Reads 15 files
│   ├── Follows import chains
│   ├── Searches for patterns
│   └── Returns: 300-token summary
│
├── Main session receives: 300 tokens (not 15,000+)
│
└── You continue working with a clean context
```

---

## Agent Types

### Explore

**Purpose:** Fast codebase exploration — find files, search code, answer questions about the codebase.

| Property | Detail |
|---|---|
| **Tools available** | Read, Glob, Grep, Bash (read-only), WebFetch, WebSearch — all tools except Task, Edit, Write, NotebookEdit |
| **Cannot** | Edit files, write files, spawn sub-subagents |
| **Default model** | **Haiku** — this is why Explore agents are fast and cheap |
| **Best for** | "How does X work?", "Find where Y is defined", "What files handle Z?" |
| **Thoroughness** | Specify in your prompt: "quick", "medium", or "very thorough" |

**When to use:**
- Understanding unfamiliar code before modifying it
- Broad searches across many files
- Answering architectural questions
- When you'd otherwise need 5+ Grep/Read calls

**Example prompts:**
- "Do a thorough exploration of how authentication works in this project. Trace the flow from login request to session creation."
- "Quick search: find all files that import the DatabaseService class"

---

### Plan

**Purpose:** Design implementation plans — analyze requirements, consider trade-offs, produce step-by-step strategies.

| Property | Detail |
|---|---|
| **Tools available** | Read, Glob, Grep, Bash (read-only), WebFetch, WebSearch — same as Explore (no edit/write) |
| **Cannot** | Edit files, write files, spawn sub-subagents |
| **Default model** | **Inherits from parent session** |
| **Best for** | Designing approaches, identifying critical files, considering alternatives |

**When to use:**
- Before implementing a complex feature
- When you need to evaluate multiple approaches
- When you want a plan validated against the actual code

**Example prompts:**
- "Plan how to add WebSocket support to this Express app. Identify which files need changes and suggest an approach."

---

### Bash

**Purpose:** Command execution specialist.

| Property | Detail |
|---|---|
| **Tools available** | Bash only |
| **Best for** | Running commands, git operations, builds, tests |

**When to use:**
- Running a test suite and capturing results
- Complex git operations
- Build processes with long output

---

### general-purpose

**Purpose:** Broad multi-step research and task execution.

| Property | Detail |
|---|---|
| **Tools available** | All tools (full access) |
| **Best for** | Complex tasks requiring both research and action, multi-step investigations |

**When to use:**
- Tasks that need both searching and reasoning across many steps
- When you're not sure which tools will be needed
- Complex investigations that might require web searches + code searches + file reads

---

### claude-code-guide

**Purpose:** Answer questions about Claude Code itself, the Claude API, and the Agent SDK.

| Property | Detail |
|---|---|
| **Tools available** | Glob, Grep, Read, WebFetch, WebSearch |
| **Default model** | **Haiku** |
| **Best for** | "How do I configure X in Claude Code?", "What does this Claude API parameter do?" |

**When to use:**
- Questions about Claude Code features, settings, or behavior
- Claude API or Anthropic SDK usage questions
- Agent SDK questions

---

### statusline-setup

**Purpose:** Configure the Claude Code status line setting.

| Property | Detail |
|---|---|
| **Tools available** | Read, Edit |
| **Default model** | **Sonnet** |
| **Best for** | Status line configuration only |

---

> **Note:** Tool lists and default models above are based on the observed system prompt. Official docs describe agent capabilities more generically (e.g., "read-only tools"). The details here are accurate as of the time of writing but may change.

## Key Parameters

### `prompt` (required)
The task description. Be specific — the subagent has no access to your conversation history (unless the agent type notes otherwise).

**Good prompt:**
> "Search the project for all usages of the `UserService` class. List every file that imports it, and describe how each file uses it. The project is a .NET 8 Web API in the `src/` directory."

**Bad prompt:**
> "Look at the user service"

### `subagent_type` (required)
One of: `Explore`, `Plan`, `Bash`, `general-purpose`, `claude-code-guide`, `statusline-setup`

### `model` (optional)
Override the model for this agent:
- `opus` — most capable, best for complex reasoning
- `sonnet` — balanced capability and speed
- `haiku` — fastest and cheapest, good for straightforward tasks

If not specified, inherits from the parent session.

### `run_in_background` (optional)
Set to `true` to run asynchronously. The tool returns immediately with an `output_file` path. Use Read or Bash to check on progress later.

### `resume` (optional)
Pass a previous agent ID to continue where it left off, with full prior context preserved. Useful for follow-up questions on the same topic without re-doing the initial exploration.

### `max_turns` (optional)
Limit the number of API round-trips the subagent can make.

---

## Parallelism

You can launch **multiple subagents simultaneously** in a single turn. This is valuable when:

- You need information from multiple independent areas
- Different aspects of a problem can be researched concurrently
- You want to maximize throughput

**How to trigger it:**
Say "in parallel" or "at the same time":
> "In parallel, use one agent to explore the auth system and another to explore the database layer."

Claude will make multiple Task tool calls in a single response.

**Limit:** Up to 3 parallel agents is typical. More than that has diminishing returns.

---

## Background Execution

With `run_in_background: true`:

1. The agent starts working asynchronously
2. You get back an output file path immediately
3. You can continue your conversation
4. Check on the agent later by reading the output file
5. Use TaskOutput to get the final result (with `block: true` to wait, or `block: false` to check status)
6. Use TaskStop to kill a runaway background agent

**Backgrounding a running agent:** Press **`Ctrl+B`** to send a currently-running foreground subagent to the background, so you can continue working while it finishes.

**Permission pre-prompting:** Background subagents prompt for any needed tool permissions **before** they start running. Once in the background, they inherit the pre-approved permissions and auto-deny anything not pre-approved. If a background subagent fails because it needed an unapproved permission, you can resume it in the foreground for interactive prompting.

**Good for:**
- Long-running investigations while you work on something else
- Running multiple parallel research tasks
- Test suites or builds that take time

---

## Resuming Agents

When a subagent finishes, it returns an **agent ID**. You can resume it later:

```
Turn 1: Spawn Explore agent → gets agent ID "abc123"
Turn 5: Resume agent "abc123" with follow-up question
         → Agent still has its full prior context
```

This avoids re-doing the initial exploration when you have follow-up questions.

**When to resume vs. start fresh:**
- **Resume** when you want to ask a follow-up about the same topic
- **Start fresh** when the new question is unrelated

---

## Custom Subagents

Beyond the built-in types above, you can **create your own subagents** with custom prompts, tool restrictions, and model overrides. Custom subagents are defined as markdown files with YAML frontmatter in `.claude/agents/` (project-level) or `~/.claude/agents/` (user-level).

Claude can delegate to custom subagents automatically based on their description, or you can request them explicitly: "Use the code-reviewer subagent to check this module."

Create and manage them with `/agents`, or add files manually.

See [Custom Subagents](custom-agents.md) for the full guide, or the [official subagents documentation](https://code.claude.com/docs/en/sub-agents) for advanced features like hooks, persistent memory, and skills injection.

---

## What Subagents Cannot Do

- **Subagents cannot spawn their own subagents** — no recursive agent trees
- **Explore and Plan agents cannot edit files** — they are read-only
- **Subagents don't see your conversation history** — provide full context in the prompt
- **Foreground subagents** can pass permission prompts and clarifying questions through to you. **Background subagents** auto-deny anything not pre-approved and cannot ask questions.
- **MCP tools** are not available in background subagents

---

## Quick Reference

| I want to... | Agent type | Example prompt |
|---|---|---|
| Understand how code works | Explore | "How does the payment processing flow work?" |
| Find files and patterns | Explore | "Find all React components that use the useAuth hook" |
| Design an implementation | Plan | "Plan how to add email notifications to the order system" |
| Run commands | Bash | "Run the test suite and summarize any failures" |
| Do complex multi-step research | general-purpose | "Research how to integrate Stripe with our existing payment module" |
| Ask about Claude Code itself | claude-code-guide | "How do I configure MCP servers in Claude Code?" |
| Create a custom subagent | (any custom type) | See [Custom Subagents](custom-agents.md) |

| I want to... | Parameter |
|---|---|
| Run agent in background | `run_in_background: true` |
| Use a cheaper/faster model | `model: "haiku"` |
| Continue a previous agent | `resume: "<agent-id>"` |
| Limit agent runtime | `max_turns: 5` |
| Run multiple agents at once | Ask for parallel execution in your message |
| Create my own subagent | `/agents` or add `.md` file to `.claude/agents/` |
