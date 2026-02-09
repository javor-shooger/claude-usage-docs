# Claude Code: Extending Claude Code

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

Skills are markdown files that give Claude **domain knowledge** or **reusable workflows**. Unlike CLAUDE.md (which loads every session), skills load **on demand** — only when Claude determines they're relevant or when you invoke them directly with a slash command.

### Where They Live

```
.claude/skills/
├── api-conventions/
│   └── SKILL.md
├── fix-issue/
│   └── SKILL.md
└── deploy-checklist/
    └── SKILL.md
```

### SKILL.md Format

```markdown
---
name: api-conventions
description: REST API design conventions for our services
---
# API Conventions
- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
- Version APIs in the URL path (/v1/, /v2/)
```

The `description` field tells Claude when this skill is relevant. When Claude works on API code and sees this description, it loads the skill automatically.

### Slash-Command Skills

Skills can also be invoked manually as slash commands — useful for workflows with side effects:

```markdown
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---
Analyze and fix the GitHub issue: $ARGUMENTS.

1. Use `gh issue view` to get the issue details
2. Search the codebase for relevant files
3. Implement the fix
4. Write and run tests
5. Create a commit and push a PR
```

Usage: `/fix-issue 1234`

The `disable-model-invocation: true` flag means Claude won't auto-invoke this skill — you have to trigger it manually. This is good for workflows that make changes.

### Key Points

- Skills load on demand, not at session start — they **don't bloat your base context**
- Claude sees skill descriptions at startup but only loads the full content when needed
- Use skills for domain knowledge that's only relevant sometimes
- Use CLAUDE.md for rules that apply every session

For the full skill configuration options, see the [official skills documentation](https://code.claude.com/docs/en/skills).

---

## Hooks

Hooks are **shell scripts that run automatically** at specific points in Claude's workflow. Unlike CLAUDE.md instructions (which are advisory — Claude can choose to ignore them), hooks are **deterministic and guaranteed**.

### When Hooks Fire

| Hook Event | When It Runs | Example Use |
|---|---|---|
| **PreToolUse** | Before Claude uses a tool | Validate a Bash command, block writes to certain files |
| **PostToolUse** | After Claude uses a tool | Run linter after every file edit |
| **Stop** | When Claude finishes a response | Run tests after implementation |

### Simple Example

Run ESLint automatically after every file edit:

```json
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix $TOOL_INPUT_FILE_PATH"
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
| Give Claude project knowledge that loads only when relevant | **Skill** |
| Create a reusable workflow I invoke with `/command` | **Skill** (with `disable-model-invocation: true`) |
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
| **Skills** | `.claude/skills/*/SKILL.md` | On demand (when relevant or invoked) | You |
| **Hooks** | `.claude/settings.json` or managed settings | Automatically at configured trigger points | You |
| **Plugins** | Installed via `/plugin` | At session start | Community / you |
| **MCP servers** | Settings JSON (`mcpServers` key) | At session start | External packages |
