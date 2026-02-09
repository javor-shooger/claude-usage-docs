# Claude Code Usage Docs

## What This Project Is
A personal reference library documenting how Claude Code works — its tools, memory system, context mechanics, and practical usage patterns. The goal is to help the user understand and work more effectively with Claude Code.

## How to Work in This Project
- **This is a research & documentation project, not a codebase.** The work here is reading, verifying, and writing accurate markdown reference files.
- **Accuracy is the top priority.** Only document what you can confirm from your system prompt, actual tool definitions, or well-established public knowledge. When uncertain, say so explicitly rather than guessing.
- **Keep files consistent with each other.** When updating one file, check if other files reference the same information and keep them aligned (e.g., tool counts, token estimates).
- **Don't overwrite existing files without reading them first.** The user has built these across multiple sessions — always read before editing.
- **Use the same style as existing files:** markdown tables, concise bullet points, practical examples, clear section headers.

## Existing Files (15 files)
See `index.md` for the full list in recommended reading order. Key files:
- `cli-basics.md` — Slash commands, keyboard shortcuts, `@` references, platforms
- `session-summary.md` — Sessions, context window, checkpointing, compaction
- `permissions-and-modes.md` — Ask/Auto-accept/Plan modes, permission rules, sandboxing
- `memory-and-instructions.md` — CLAUDE.md hierarchy, `@` imports, rules, auto memory
- `writing-effective-claude-md.md` — How to write good CLAUDE.md instructions
- `tools-reference.md` — Complete reference of all 39+ tools with context cost estimates
- `tools-how-to-trigger.md` — What to say to trigger each tool, `@` file references
- `prompt-crafting.md` — Prompt patterns, scoping, letting Claude interview you
- `context-management.md` — Strategies: selective reading, subagents, skills, compaction
- `cost-and-models.md` — Token costs, model selection, extended thinking, fast mode
- `subagents.md` — Task tool deep dive, agent types, custom subagents
- `mcp-setup.md` — MCP server configuration, tool search, resources
- `extending-claude-code.md` — Skills, hooks, and plugins overview
- `custom-agents.md` — Custom subagents in `.claude/agents/`
- `workflows-and-patterns.md` — Recipes for common tasks (always last in reading order)

## When the User Wants to Add New Topics
- Suggest a new file rather than appending to an existing one (unless it clearly belongs there)
- Start by reviewing what's already documented to avoid duplication
- Cross-reference related files where appropriate
- **Check the official docs first.** Browse https://code.claude.com/docs/en/ to verify current behavior and discover details before writing. Our docs are a curated learning path, not a mirror — but they must be accurate.
- **Add links to official docs.** When a topic has a full official page, link to it inline: "For details, see the [official X documentation](https://code.claude.com/docs/en/X)." This keeps our docs concise while giving readers a path to dig deeper.

## Workflows Maintenance Rule
When adding or updating a topic doc, check if `workflows-and-patterns.md` should be updated too:
- Does the new feature enable a new workflow recipe? → Add a new workflow
- Does it improve an existing workflow? → Add a tip or update the steps
- Does it affect model/cost/skills usage? → Update the quick reference tables at the bottom
This file is the "putting it all together" doc — it should reflect all documented features.

## Index Ordering Rule
When adding a new doc to `index.md`, place it in the correct position based on this progression — do NOT just append to the end:

1. **Getting started** — basic controls, launching, daily use (CLI Basics)
2. **How it works** — sessions, context, mechanics (Session & Context)
3. **Choosing your mode** — permissions, interaction modes (Permissions & Modes)
4. **Configuration** — persistent instructions, memory setup (Memory & Instructions)
5. **What's available** — tool catalog, how to trigger them (Tools Reference, Triggers)
6. **Being efficient** — context strategies, advanced features (Context Management, Subagents)
7. **Extending** — MCP, plugins, integrations (MCP Setup)
8. **Putting it all together** — workflows, patterns, recipes (Workflows & Patterns — always last)
