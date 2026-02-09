# Claude Code: Custom Subagents

*Last verified against [official docs](https://code.claude.com/docs/en/) on 2026-02-09*

How to create your own specialized subagents — reusable AI assistants with focused roles, restricted tools, and custom prompts.

---

## What Are Custom Subagents?

Custom subagents are **specialized assistants you define** in markdown files. Each one gets:

- A **custom system prompt** describing its role and behavior
- A **restricted tool set** (e.g., read-only for reviewers, full access for fixers)
- An optional **model override** (e.g., use Haiku for cheap tasks, Opus for complex ones)
- Its own **isolated context window** — just like built-in subagents

Claude can automatically delegate to your custom subagents based on their description, or you can request them explicitly.

---

## Where They Live

| Location | Scope | Shared? |
|---|---|---|
| `.claude/agents/` | This project only | Yes — commit to git for team use |
| `~/.claude/agents/` | All your projects | No — personal to your machine |

Project-level agents take priority over user-level agents when names conflict.

---

## Creating a Custom Subagent

### Option A: Interactive (Recommended)

```
/agents
```

Select **Create new agent**, choose project or user level, and either write it yourself or select **Generate with Claude** to have Claude draft it from a description.

### Option B: Manual

Create a markdown file with YAML frontmatter:

```markdown
<!-- .claude/agents/code-reviewer.md -->
---
name: code-reviewer
description: Reviews code for quality, security, and best practices. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior code reviewer. When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Review for correctness, security, and readability

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

The YAML frontmatter configures the subagent. The markdown body becomes its system prompt.

---

## Frontmatter Fields

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Unique identifier (lowercase, hyphens) |
| `description` | Yes | When Claude should delegate to this subagent — write this clearly |
| `tools` | No | Which tools the subagent can use. Inherits all if omitted |
| `disallowedTools` | No | Tools to explicitly deny (alternative to allowlisting with `tools`) |
| `model` | No | `sonnet`, `opus`, `haiku`, or `inherit` (default: `inherit`) |
| `permissionMode` | No | `default`, `acceptEdits`, `plan`, `dontAsk`, `delegate`, `bypassPermissions` |
| `maxTurns` | No | Maximum agentic turns before stopping |
| `skills` | No | Skills to preload into agent context at startup |
| `mcpServers` | No | MCP servers available to this agent |
| `hooks` | No | Lifecycle hooks scoped to this agent |
| `memory` | No | Persistent memory scope: `user`, `project`, or `local` |

The `delegate` permission mode is for agent team leads — it restricts the agent to coordination-only, using team management tools rather than direct code tools.

For detailed examples of advanced fields, see the [official subagents documentation](https://code.claude.com/docs/en/sub-agents).

---

## Examples

### Code Reviewer (Read-Only)

```markdown
---
name: code-reviewer
description: Expert code review specialist. Use proactively after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: sonnet
---

You are a senior code reviewer ensuring high standards of code quality and security.

Review checklist:
- Code is clear and readable
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Good test coverage

Provide specific, actionable feedback.
```

### Debugger (Can Edit)

```markdown
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger. When invoked:
1. Capture the error message and stack trace
2. Isolate the failure location
3. Implement a minimal fix
4. Verify the fix works

Focus on fixing the underlying issue, not the symptoms.
```

### Data Analyst (Specific Model)

```markdown
---
name: data-analyst
description: Data analysis expert for SQL queries and data insights.
tools: Bash, Read, Write
model: haiku
---

You are a data analyst. Write efficient SQL queries, analyze results, and present findings clearly.
Always explain your query approach and highlight key findings.
```

---

## How Claude Uses Custom Subagents

Claude delegates automatically based on the `description` field. If you say "review my recent changes", Claude sees the `code-reviewer` description and delegates to it.

You can also be explicit:

```
Use the code-reviewer subagent to check the auth module
Have the debugger subagent investigate why login fails
```

### Foreground vs Background

- **Foreground** (default): Blocks your conversation until the subagent finishes. You can answer any permission prompts.
- **Background**: Runs concurrently. Press `Ctrl+B` to background a running subagent, or ask Claude to "run this in the background."

### Proactive Delegation

Include phrases like "use proactively" in the description to encourage Claude to use the subagent without being asked:

```yaml
description: Reviews code for quality and security. Use proactively after code changes.
```

---

## Custom Subagents vs CLAUDE.md

| | Custom Subagents | CLAUDE.md |
|---|---|---|
| **Loaded** | Only when Claude delegates to them | Every session automatically |
| **Purpose** | Role-specific behavior with isolated context | Project-wide rules and conventions |
| **Context** | Own isolated context window | Shares your main context |
| **Best for** | "Act as a code reviewer" | "Use TypeScript strict mode" |

They complement each other: CLAUDE.md provides the project baseline. Custom subagents add specialized behavior on top.

---

## Custom Subagents vs Built-in Subagents

| | Built-in (Explore, Plan, etc.) | Custom |
|---|---|---|
| **Defined by** | Claude Code | You |
| **Prompt** | Pre-defined | Your markdown body |
| **Tools** | Fixed per type | You choose |
| **Model** | Fixed or inherited | You choose |
| **Can be spawned by Claude** | Yes | Yes |

Custom subagents work exactly like built-in ones — they're spawned via the Task tool, get isolated context, and return a summary.

**Resuming subagents:** Claude can resume a previous subagent's work by passing its agent ID. The resumed agent retains its full conversation history (tool calls, results, reasoning), so it can continue where it left off without re-doing initial exploration.

---

## Managing Subagents

```
/agents            — view, create, edit, delete subagents
```

Subagents are loaded at session start. If you create one manually by adding a file, restart your session or use `/agents` to load it immediately.

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Create a custom subagent | `/agents` → Create new agent, or add a `.md` file to `.claude/agents/` |
| Use a subagent | Ask Claude, or say "use the [name] subagent to..." |
| Share subagents with my team | Put them in `.claude/agents/` and commit to git |
| Use a subagent across all projects | Put it in `~/.claude/agents/` |
| Restrict what a subagent can do | Use the `tools` field in frontmatter |
| Use a cheaper model for a subagent | Set `model: haiku` in frontmatter |
| Run a subagent in the background | Say "run this in the background" or press `Ctrl+B` |
| Add advanced features (hooks, memory) | See the [official subagents docs](https://code.claude.com/docs/en/sub-agents) |
