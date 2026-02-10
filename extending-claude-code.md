# Claude Code: Extending Claude Code

*Last verified against [official docs](https://code.claude.com/docs/en/) on 2026-02-10*

A beginner-friendly overview of the three main ways to extend Claude Code beyond its built-in capabilities: Skills, Hooks, and Plugins.

---

## Why Extend?

Out of the box, Claude Code has powerful built-in tools (Read, Edit, Bash, etc.) and you can add external tools via MCP servers. But there are three more extension layers that let you customize **what Claude knows**, **what it does automatically**, and **how you invoke workflows**:

| Extension | What It Does | One-Liner |
|---|---|---|
| **Skills** | Give Claude reusable knowledge and workflows | "A prompt template Claude can load on demand" |
| **Hooks** | Run scripts automatically at specific points | "If X happens, always do Y — guaranteed" |
| **Plugins** | Bundle skills + hooks + agents into one package | "An installable extension pack" |

---

## Skills

Skills are markdown files that give Claude **domain knowledge** or **reusable workflows**. Unlike CLAUDE.md (which loads every session), skills load **on demand** — only when Claude determines they're relevant or when you invoke them with a slash command.

| Aspect | How it works |
|---|---|
| **Where they live** | `.claude/skills/<name>/SKILL.md` (project) or `~/.claude/skills/<name>/SKILL.md` (personal) |
| **Format** | YAML frontmatter + markdown content. Only `description` is recommended. |
| **Two types** | *Reference* skills add knowledge (API conventions, style guides). *Task* skills define workflows (`/deploy`, `/fix-issue`). |
| **Auto vs manual** | By default, Claude loads skills when relevant. Add `disable-model-invocation: true` for manual-only (`/skill-name`). |
| **Arguments** | Use `$ARGUMENTS` (or `$0`, `$1`) in content. `/fix-issue 1234` passes `1234`. |
| **Isolation** | Add `context: fork` to run in a subagent with its own context window. |
| **Context cost** | Descriptions load at startup (small). Full content loads only when invoked. |

**Quick example — a reference skill:**
```markdown
---
description: REST API design conventions for our services
---
- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
```

**Quick example — a task skill:**
```markdown
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---
Fix GitHub issue $ARGUMENTS. Search the codebase, implement the fix, write tests, commit.
```

For the full guide — creation walkthrough, all frontmatter fields, arguments, dynamic context injection, subagent integration, permissions, and troubleshooting — see **[Skills Deep Dive](skills.md)**.

For the official specification, see the [official skills documentation](https://code.claude.com/docs/en/skills).

---

## Hooks

Hooks are **shell scripts that run automatically** at specific points in Claude's workflow. Unlike CLAUDE.md instructions (which are advisory — Claude can choose to ignore them), hooks are **deterministic and guaranteed**.

### Hook Types

| Type | Description |
|---|---|
| **`command`** | Run a shell command (most common) |
| **`prompt`** | Send a prompt to an LLM for evaluation |
| **`agent`** | Spawn a subagent with tool access |

### When Hooks Fire

The most commonly used events:

| Hook Event | When It Runs | Example Use |
|---|---|---|
| **PreToolUse** | Before Claude uses a tool | Validate a Bash command, block writes to certain files |
| **PostToolUse** | After a tool succeeds | Run linter after every file edit |
| **PostToolUseFailure** | After a tool fails | Log failures, retry logic |
| **Stop** | When Claude finishes a response | Run tests after implementation |
| **SessionStart** | Session begins or resumes | Load env vars or project context |
| **SessionEnd** | Session terminates | Cleanup actions |
| **UserPromptSubmit** | You submit a prompt | Add context to every prompt |

Additional events: **PermissionRequest**, **Notification**, **SubagentStart**, **SubagentStop**, **TeammateIdle**, **TaskCompleted**, **PreCompact** (14 events total).

For the full list, see the [official hooks reference](https://code.claude.com/docs/en/hooks).

### Simple Example

Run ESLint automatically after every file edit. Hooks receive context via **JSON on stdin** — use `jq` to extract fields:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx eslint --fix"
          }
        ]
      }
    ]
  }
}
```

### CLAUDE.md vs Hooks

| | CLAUDE.md Instruction | Hook |
|---|---|---|
| "Run lint after edits" | Advisory — Claude *usually* follows it | **Guaranteed** — runs every time, no exceptions |
| Can Claude skip it? | Yes, if it decides to | No — hooks execute unconditionally |
| Best for | Guidelines, preferences, conventions | Hard rules that must always happen |

### Key Points

- Use hooks for things that **must** happen every time with zero exceptions
- You can ask Claude to write hooks for you: "Write a hook that runs eslint after every file edit"
- Use `/hooks` for interactive configuration
- The `matcher` field filters which tools trigger the hook (e.g., only `Bash`, only `Edit|Write`)
- **Async hooks** (`"async": true`) run in the background without blocking — only available for `command` type hooks
- **Exit codes matter:** `0` = success, `2` = blocking error (shown to Claude/user), other = non-blocking error

For the full hook event reference and configuration options, see the [official hooks documentation](https://code.claude.com/docs/en/hooks).

---

## Plugins

Plugins are **installable packages** that bundle together skills, hooks, subagents, and MCP servers into a single unit. Think of them as extension packs.

### Using Plugins

```
/plugin                  — browse the marketplace and install plugins
```

Plugins can come from:
- **Anthropic's marketplace** — official and community plugins
- **Your organization** — private plugin marketplaces
- **Local development** — plugins you create yourself

### What Plugins Can Include

| Component | Example |
|---|---|
| Skills | API conventions, deployment checklists |
| Hooks | Auto-lint, test runners |
| Subagents | Code reviewer, security scanner |
| MCP servers | Database tools, monitoring integrations |

### Key Points

- Plugins are the easiest way to add capabilities — one install gets you everything
- **Code intelligence plugins** are especially useful — they give Claude precise "go to definition" and "find references" navigation for typed languages. See the [plugin marketplace](https://code.claude.com/docs/en/discover-plugins) for available language plugins.
- You don't need to create plugins to use them — just browse and install

For details on creating your own plugins, see the [official plugins documentation](https://code.claude.com/docs/en/plugins).

---

## Which Extension Should I Use?

| I want to... | Use |
|---|---|
| Give Claude project knowledge that loads only when relevant | **Skill** — see [Skills Deep Dive](skills.md) |
| Create a reusable workflow I invoke with `/command` | **Skill** (with `disable-model-invocation: true`) — see [Skills Deep Dive](skills.md) |
| Guarantee a script runs after every file edit | **Hook** |
| Block Claude from writing to certain files | **Hook** (PreToolUse with exit code 2) |
| Install a pre-made extension pack | **Plugin** |
| Give Claude "go to definition" navigation | **Plugin** (code intelligence) |
| Add rules that apply every session | **CLAUDE.md** (not an extension — just use your instruction file) |
| Connect to an external service (Figma, DB, etc.) | **MCP server** (see [MCP Setup](mcp-setup.md)) |

---

## Quick Reference

| Extension | Where defined | Loads when | Created by |
|---|---|---|---|
| **Skills** | `.claude/skills/*/SKILL.md` or `~/.claude/skills/*/SKILL.md` | On demand (when relevant or invoked) | You |
| **Hooks** | Settings JSON or managed settings | Automatically at configured trigger points (14 events) | You |
| **Plugins** | Installed via `/plugin` | At session start | Community / you |
| **MCP servers** | `~/.claude.json` or `.mcp.json` (`mcpServers` key) | At session start | External packages |
