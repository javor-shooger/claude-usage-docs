# Claude Code: Extending Claude Code

*Last verified against [official docs](https://code.claude.com/docs/en/) on 2026-02-11*

A comparison and decision guide for the five ways to extend Claude Code beyond its built-in capabilities. Each mechanism has a dedicated deep-dive chapter — this page helps you **choose which to use** and understand **how they work together**.

For the official overview, see the [features overview](https://code.claude.com/docs/en/features-overview).

---

## The Five Extension Mechanisms

| Mechanism            | What It Does                                          | One-Liner                                              | Deep Dive                                                    |
| -------------------- | ----------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| **Skills**           | Give Claude reusable knowledge and workflows          | "A prompt template Claude can load on demand"          | [Skills Deep Dive](skills.md)                                |
| **Hooks**            | Run scripts automatically at specific points          | "If X happens, always do Y — guaranteed"               | [Official hooks docs](https://code.claude.com/docs/en/hooks) |
| **Custom Subagents** | Delegate tasks to specialized workers                 | "A focused assistant with its own context window"      | [Custom Subagents](custom-agents.md)                         |
| **MCP Servers**      | Connect Claude to external tools and services         | "An open-protocol bridge to databases, APIs, browsers" | [MCP Setup](mcp-setup.md)                                    |
| **Plugins**          | Bundle skills + hooks + agents + MCP into one package | "An installable extension pack"                        | [Plugins Deep Dive](plugins.md)                              |

> **Also:** [Agent Teams](https://code.claude.com/docs/en/agent-teams) let you coordinate multiple Claude instances working in parallel. This is an experimental feature (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`).

---

## Skills

Skills are markdown files that give Claude **domain knowledge** or **reusable workflows**. Unlike CLAUDE.md (which loads every session), skills load **on demand** — only when Claude determines they're relevant or when you invoke them with a slash command.

- **Two types:** *Reference* skills add knowledge (API conventions, style guides). *Task* skills define workflows (`/deploy`, `/fix-issue`).
- **Key property:** Skills are **advisory** — Claude decides when they're relevant. Use `disable-model-invocation: true` for manual-only invocation.
- **Context cost:** Descriptions load at startup (small). Full content loads only when invoked.

```markdown
---
description: REST API design conventions for our services
---
- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
```

For the full guide — creation, frontmatter fields, arguments, dynamic context, subagent integration — see **[Skills Deep Dive](skills.md)** and the [official skills documentation](https://code.claude.com/docs/en/skills).

---

## Hooks

Hooks are **scripts that run automatically** at specific points in Claude's workflow. Unlike CLAUDE.md instructions (which are advisory), hooks are **deterministic and guaranteed**.

- **Three types:** `command` (shell script — most common), `prompt` (single LLM call), `agent` (multi-turn subagent with tools).
- **14 events:** PreToolUse, PostToolUse, Stop, SessionStart, UserPromptSubmit, and [9 more](https://code.claude.com/docs/en/hooks).
- **Key property:** Hooks **always execute** — Claude cannot skip or ignore them. Exit code `2` blocks the action.

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx eslint --fix" }]
    }]
  }
}
```

For hook events, configuration, and advanced patterns, see the [official hooks documentation](https://code.claude.com/docs/en/hooks).

---

## Custom Subagents

Custom subagents are **specialized assistants** you define in markdown, each with its own system prompt, restricted tools, and isolated context window.

- **Where they live:** `.claude/agents/<name>.md` (project) or `~/.claude/agents/<name>.md` (personal).
- **Key property:** Each subagent gets its **own context window** — work doesn't pollute the main conversation. Results are summarized when returned.
- **Configurable:** Model (Haiku for cheap tasks, Opus for complex), tool restrictions (allowlist or denylist), preloaded skills, and MCP servers.

```markdown
---
description: Reviews code for quality issues. Use PROACTIVELY after code changes.
tools: [Read, Grep, Glob]
model: haiku
---
Review the code for bugs, security issues, and style problems. Be specific and actionable.
```

For the full guide — creation, frontmatter fields, delegation, background execution — see **[Custom Subagents](custom-agents.md)** and the [official subagents documentation](https://code.claude.com/docs/en/sub-agents).

---

## MCP Servers

MCP servers connect Claude to **external tools and services** via the open [Model Context Protocol](https://code.claude.com/docs/en/mcp). They add tools Claude can invoke — database queries, browser automation, API calls.

- **Where configured:** `~/.claude.json` (personal) or `.mcp.json` (project, team-shared).
- **Key property:** Open standard — not Claude-specific. Same servers work with any MCP-compatible client.
- **Context cost:** Tool definitions load at session start (cost varies per server — run `/mcp` to check). Tool Search defers loading when definitions exceed 10% of context.

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic/mcp-playwright"]
    }
  }
}
```

For configuration, transports, environment variables, and team sharing — see **[MCP Setup](mcp-setup.md)** and the [official MCP documentation](https://code.claude.com/docs/en/mcp).

---

## Plugins

Plugins are **installable packages** that bundle skills, hooks, subagents, MCP servers, and LSP servers into a single unit. One install adds multiple capabilities at once.

- **Install with:** `/plugin` to browse the marketplace, or `claude plugin install <name>`.
- **Key property:** Code intelligence plugins (LSP) are the most impactful — they give Claude precise "go to definition" and "find references" navigation.
- **42+ official plugins** covering code intelligence, development workflows, and external integrations (GitHub, Slack, Firebase, etc.).

For the full guide — the official collection, code intelligence setup, scopes, marketplaces — see **[Plugins Deep Dive](plugins.md)** and the [official plugins documentation](https://code.claude.com/docs/en/plugins).

---

## Choosing the Right Mechanism

### Decision Table

| I want to... | Use |
|---|---|
| Give Claude project knowledge that loads when relevant | **Skill** (reference type) — [Skills Deep Dive](skills.md) |
| Create a reusable workflow invoked with `/command` | **Skill** (with `disable-model-invocation: true`) — [Skills Deep Dive](skills.md) |
| Guarantee a script runs after every file edit | **Hook** (PostToolUse) |
| Block Claude from writing to certain files | **Hook** (PreToolUse with exit code 2) |
| Validate or transform commands before execution | **Hook** (PreToolUse) |
| Run tests automatically when Claude finishes | **Hook** (Stop event) |
| Delegate a task to a focused worker | **Custom Subagent** — [Custom Subagents](custom-agents.md) |
| Isolate verbose analysis from the main conversation | **Custom Subagent** (with `context: fork` in skills or Task tool) |
| Run cheap analysis tasks to save cost | **Custom Subagent** with `model: haiku` |
| Connect Claude to a database or external API | **MCP Server** — [MCP Setup](mcp-setup.md) |
| Add browser automation | **MCP Server** (Playwright) — [MCP Setup](mcp-setup.md) |
| Install a pre-made extension pack | **Plugin** — [Plugins Deep Dive](plugins.md) |
| Give Claude "go to definition" navigation | **Plugin** (code intelligence / LSP) — [Plugins Deep Dive](plugins.md) |
| Share tools with my whole team | **Plugin** (`--scope project`) or `.mcp.json` committed to git |
| Add rules that apply every session | **CLAUDE.md** (not an extension — see [Memory & Instructions](memory-and-instructions.md)) |
| Coordinate multiple Claude instances in parallel | **Agent Teams** (experimental — [official docs](https://code.claude.com/docs/en/agent-teams)) |

### Comparison Matrix

| Aspect | Skills | Hooks | Custom Subagents | MCP Servers | Plugins |
|---|---|---|---|---|---|
| **What it is** | On-demand markdown prompts | Auto-running scripts at lifecycle events | Isolated assistants with custom prompts | External tool connections via open protocol | Bundled extension packs |
| **Where defined** | `.claude/skills/*/SKILL.md` | Settings JSON or skill/agent frontmatter | `.claude/agents/*.md` | `~/.claude.json` or `.mcp.json` | Installed via `/plugin` |
| **Loads when** | On demand (relevant or `/invoked`) | Session start → fires at trigger events | On demand (delegated by Claude) | Session start (tools always available) | Session start (components load per type) |
| **Deterministic?** | No — advisory (Claude decides) | **Yes — guaranteed** every time | No — advisory (Claude delegates) | N/A — tools available on request | Depends on bundled components |
| **Runs code?** | Yes (`` !`command` `` preprocessing, tool calls) | Yes (shell commands, LLM prompts, subagents) | Yes (tool calls within isolated context) | Yes (external process) | Yes (all bundled components) |
| **Context cost** | Descriptions at startup; full on invoke | None (runs as subprocess) | Own isolated context window | Tool definitions at startup (varies per server) | Depends on components |
| **Can be shared?** | Commit `.claude/skills/` to git | Commit settings to git | Commit `.claude/agents/` to git | Commit `.mcp.json` to git | `--scope project` or marketplace |
| **Created by** | You | You (or ask Claude to write them) | You | External packages / you | Community / Anthropic / you |
| **Best for** | Knowledge, workflows, conventions | Hard rules, validation, automation | Specialized tasks, cost optimization | External services, APIs, browsers | One-click team setup, code intelligence |

### How They Work Together

These mechanisms are complementary. Real setups often combine several:

**MCP + Skill** — MCP provides the connection, the skill teaches Claude the conventions.
> Example: MCP server connects to your database. A skill documents your query patterns and schema conventions. As Anthropic puts it: "Use both together: MCP for connectivity, Skills for procedures."

**Skill + Hook** — Hooks can boost skill reliability. Community testing shows skill auto-activation achieves only ~20% activation rate with simple descriptions, but a "forced evaluation" hook that makes Claude explicitly assess each skill raises this to ~84%.

**Subagent + Skill** — Preload skills into a subagent via the `skills:` frontmatter field. The subagent gets focused expertise without loading it into the main conversation.

**Hook + Subagent** — A hook can spawn an `agent`-type handler for multi-turn validation. Example: a Stop hook that runs a code review subagent after every implementation.

**Plugin = Bundle** — A plugin packages any combination of the above for one-click team installation. Start with standalone `.claude/` config, convert to a plugin when ready to share.

### Practical Scenarios

**Setting up a new team project:**
CLAUDE.md (always-on conventions) + project skills (API patterns, deploy checklist) + hooks (auto-lint after edits, test runner on Stop) + code intelligence plugin (LSP for your language). Each layer serves a different need — CLAUDE.md for basics everyone should follow, skills for on-demand knowledge, hooks for guarantees, plugin for IDE-quality navigation.

**Adding a database integration:**
MCP server (connection to the database) + skill (query conventions and schema documentation). The MCP server gives Claude the tools to query, the skill teaches Claude *how* to query properly for your project.

**Automating code review:**
Custom subagent (isolated reviewer with read-only tools: Read, Grep, Glob) + hook (Stop event spawns the review). The subagent isolates the review from main context. The hook guarantees it runs after every implementation — Claude can't skip it.

**Enforcing code standards:**
Hook, not a skill. Skills are advisory — Claude can choose to ignore them. Hooks execute unconditionally. Community reports document real failures when relying on CLAUDE.md for enforcement: leaked API keys and dangerous commands despite explicit prohibition. Use hooks for anything that *must* happen.

### Common Mistakes

| Don't do this | Do this instead | Why |
|---|---|---|
| Use a skill for enforcement | Use a **hook** | Skills are advisory — Claude can skip them. Hooks are guaranteed. |
| Use MCP to teach conventions | Use a **skill** | MCP connects to data. Skills teach what to do with it. |
| Write a plugin for one project | Use standalone `.claude/` config | Convert to a plugin only when ready to share. |
| Give subagents all tools | Apply **least privilege** (allowlist) | Restrict to only the tools the subagent needs. |
| Enable all MCP servers at once | Select only what you need | Too many servers can shrink your 200K context window to ~70K. |
| Write 1000-line subagent definitions | Keep concise (~300 lines), extract reusable parts as skills | Community testing shows 54% size reduction *improved* quality scores. |
| Rely only on CLAUDE.md for safety rules | Add **hooks** as enforcement layer | CLAUDE.md is advisory. Hooks guarantee execution. |

---

## Quick Reference

| Extension | Where defined | Loads when | Created by | Deep Dive |
|---|---|---|---|---|
| **Skills** | `.claude/skills/*/SKILL.md` | On demand (relevant or invoked) | You | [Skills Deep Dive](skills.md) |
| **Hooks** | Settings JSON or skill/agent frontmatter | At trigger events (14 events) | You | [Official docs](https://code.claude.com/docs/en/hooks) |
| **Custom Subagents** | `.claude/agents/*.md` | On demand (delegated) | You | [Custom Subagents](custom-agents.md) |
| **MCP Servers** | `~/.claude.json` or `.mcp.json` | At session start | External packages / you | [MCP Setup](mcp-setup.md) |
| **Plugins** | Installed via `/plugin` | At session start | Community / Anthropic / you | [Plugins Deep Dive](plugins.md) |
