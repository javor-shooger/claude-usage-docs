# Claude Code — How It Works

A personal reference library for understanding and working effectively with Claude Code.

---

## Reading Order

Start here and work through in order — each file builds on concepts from the previous ones.

| #   | File                                                                | What You'll Learn                                                                                 |
| --- | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| 1   | [Getting Started & Daily Controls](cli-basics.md)                   | CLI, VS Code, Desktop app, slash commands, keyboard shortcuts, `@` references — start here       |
| 2   | [Session & Context Mechanics](session-summary.md)                   | How sessions work, context window, checkpointing, compaction, session management                  |
| 3   | [Permissions & Interaction Modes](permissions-and-modes.md)         | Ask vs Auto-accept vs Plan mode, permission rules, sandboxing, `Shift+Tab` cycling               |
| 4   | [Memory & Instruction Files](memory-and-instructions.md)            | CLAUDE.md hierarchy, `@` imports, path-specific rules, auto memory, session memory                |
| 5   | [Writing Effective CLAUDE.md Files](writing-effective-claude-md.md) | How to write instructions that actually improve Claude's behavior — concise, actionable, specific |
| 6   | [Tools Reference](tools-reference.md)                               | Complete list of all 39+ tools, grouped by category, with context cost estimates                  |
| 7   | [How to Trigger Tools](tools-how-to-trigger.md)                     | What to say to make Claude use specific tools, `@` file references, skill invocation              |
| 8   | [Prompt Crafting](prompt-crafting.md)                               | Prompt patterns, scoping techniques, `@` references, letting Claude interview you                 |
| 9   | [Context Management Strategies](context-management.md)              | Selective reading, subagents, skills for on-demand loading, compaction, checkpointing             |
| 10  | [Cost Awareness & Model Selection](cost-and-models.md)              | Token costs, Opus vs Sonnet vs Haiku, extended thinking, fast mode, optimization strategies       |
| 11  | [Extending Claude Code](extending-claude-code.md)                   | All 5 extension mechanisms compared — skills, hooks, subagents, MCP, plugins — with decision guide |
| 12  | [Hooks Deep Dive](hooks.md)                                         | All 14 hook events, 3 hook types, matchers, input/output, practical examples                      |
| 13  | [Subagents Deep Dive](subagents.md)                                 | How the Task tool works — agent types, parallelism, background execution, custom subagents        |
| 14  | [MCP Server Configuration](mcp-setup.md)                            | Adding MCP servers, tool search, MCP resources, context cost management                           |
| 15  | [Skills Deep Dive](skills.md)                                       | Creating skills, scopes (project/personal/enterprise), arguments, subagent integration, troubleshooting |
| 16  | [Custom Subagents](custom-agents.md)                                | Create your own subagents in `.claude/agents/` — YAML frontmatter, tools, model overrides        |
| 17  | [Plugins Deep Dive](plugins.md)                                     | Installing and using plugins — official collection, code intelligence, scopes, managing plugins   |
| 18  | [Workflows & Patterns](workflows-and-patterns.md)                   | Putting it all together — recipes for plan→implement, explore→fix, debugging, rewind & retry      |

---

> All files include a "Last verified" date showing when the content was last checked against the [official Claude Code documentation](https://code.claude.com/docs/en/).
