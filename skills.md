# Claude Code: Skills

*Last verified against [official docs](https://code.claude.com/docs/en/skills) on 2026-02-10*

Skills are markdown files that give Claude **domain knowledge** or **reusable workflows**. Unlike CLAUDE.md (which loads every session), skills load **on demand** — only when Claude determines they're relevant or when you invoke them with a slash command.

Skills follow the [Agent Skills open standard](https://agentskills.io), extended with Claude Code-specific features.

For a quick overview of skills alongside hooks and plugins, see [Extending Claude Code](extending-claude-code.md).

---

## Creating Your First Skill

A skill is a directory containing a `SKILL.md` file. Here's a minimal example:

**Step 1:** Create the directory:
```
.claude/skills/api-conventions/
```

**Step 2:** Create `SKILL.md` inside it:
```markdown
---
description: REST API design conventions for our services
---
# API Conventions
- Use kebab-case for URL paths
- Use camelCase for JSON properties
- Always include pagination for list endpoints
- Version APIs in the URL path (/v1/, /v2/)
```

**Step 3:** That's it. Next time Claude works on API code and the description matches, it loads the full skill content automatically. You can also invoke it with `/api-conventions`.

**Step 4 (optional):** Verify it's available:
- Ask Claude: "What skills are available?"
- Or run `/context` to check if it appears in the loaded skill descriptions

---

## Where Skills Live (Scope and Priority)

| Location | Path | Applies to |
|---|---|---|
| **Enterprise** | Managed settings (see [permissions docs](https://code.claude.com/docs/en/permissions)) | All users in your organization |
| **Personal** | `~/.claude/skills/<skill-name>/SKILL.md` | All your projects |
| **Project** | `.claude/skills/<skill-name>/SKILL.md` | This project only |
| **Plugin** | `<plugin>/skills/<skill-name>/SKILL.md` | Where plugin is enabled |

**Priority when names conflict:** enterprise > personal > project. Plugin skills use a `plugin-name:skill-name` namespace, so they never conflict.

### Project-Level Skills

```
.claude/skills/
├── api-conventions/
│   └── SKILL.md
├── fix-issue/
│   └── SKILL.md
└── deploy-checklist/
    └── SKILL.md
```

- Commit these to git so your whole team gets them
- Scoped to this project only

### Personal-Level Skills

```
~/.claude/skills/
└── my-workflow/
    └── SKILL.md
```

- Available across all your projects
- Not shared with teammates
- Good for personal workflow shortcuts

### Monorepo Discovery

When you work in subdirectories, Claude automatically discovers skills from nested `.claude/skills/` directories. For example, editing a file in `packages/frontend/` also picks up skills from `packages/frontend/.claude/skills/`. This supports monorepos where packages have their own conventions.

### Skills from `--add-dir`

Skills in `.claude/skills/` within directories added via `--add-dir` are loaded automatically and support live change detection — you can edit them during a session without restarting.

---

## Skill Directory Structure

Each skill is a directory with `SKILL.md` as the entrypoint. Other files are optional:

```
my-skill/
├── SKILL.md           # Main instructions (required)
├── template.md        # Template for Claude to fill in
├── examples/
│   └── sample.md      # Example output showing expected format
└── scripts/
    └── validate.sh    # Script Claude can execute
```

Reference supporting files from your SKILL.md so Claude knows what they contain and when to load them:

```markdown
## Additional resources
- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```

**Tip:** Keep `SKILL.md` under 500 lines. Move detailed reference material to separate files.

---

## SKILL.md Format

A SKILL.md file has optional YAML frontmatter followed by markdown content:

```markdown
---
name: api-conventions
description: REST API design conventions for our services
---
# API Conventions
- Use kebab-case for URL paths
- Use camelCase for JSON properties
...
```

Only `description` is recommended — it tells Claude when to use the skill. All other frontmatter fields are optional.

### Frontmatter Reference

| Field                      | Description                                                                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                     | Display name. If omitted, uses directory name. Lowercase, numbers, hyphens only (max 64 chars).                                             |
| `description`              | What the skill does and when to use it. Claude uses this to decide when to load the skill. If omitted, uses the first paragraph of content. |
| `argument-hint`            | Hint for expected arguments shown in autocomplete (e.g., `[issue-number]`).                                                                 |
| `disable-model-invocation` | Set to `true` to prevent Claude from auto-loading. User must invoke manually with `/name`. Default: `false`.                                |
| `user-invocable`           | Set to `false` to hide from `/` menu. Use for background knowledge. Default: `true`.                                                        |
| `allowed-tools`            | Tools Claude can use without permission when this skill is active.                                                                          |
| `model`                    | Model to use when this skill is active.                                                                                                     |
| `context`                  | Set to `fork` to run in a subagent (isolated context).                                                                                      |
| `agent`                    | Which subagent type when `context: fork` is set (`Explore`, `Plan`, `general-purpose`, or a custom agent).                                  |
| `hooks`                    | Hooks scoped to this skill's lifecycle — cleaned up when done.                                                                              |

---

## Two Types of Skill Content

### Reference Skills (Knowledge)

Add conventions, patterns, or domain knowledge that Claude applies to your current work. These run inline alongside your conversation:

```markdown
---
description: API design patterns for this codebase
---
When writing API endpoints:
- Use RESTful naming conventions
- Return consistent error formats
- Include request validation
```

### Task Skills (Workflows)

Step-by-step instructions for a specific action. These are often invoked manually with `/skill-name`. Add `disable-model-invocation: true` so Claude doesn't trigger them automatically:

```markdown
---
name: deploy
description: Deploy the application to production
context: fork
disable-model-invocation: true
---
Deploy the application:
1. Run the test suite
2. Build the application
3. Push to the deployment target
```

---

## Controlling Who Can Invoke a Skill

By default, both you and Claude can invoke any skill. Two frontmatter fields change this:

| Frontmatter | You can invoke | Claude can invoke | When loaded into context |
|---|---|---|---|
| *(default)* | Yes | Yes | Description always in context; full skill loads when invoked |
| `disable-model-invocation: true` | Yes | No | Description NOT in context; loads only when you invoke |
| `user-invocable: false` | No | Yes | Description always in context; loads when Claude invokes |

**Key detail:** Skill *descriptions* are loaded at startup so Claude knows what's available, but the *full content* only loads when invoked. Skills with `disable-model-invocation: true` don't even load their description — zero context cost until you call them.

---

## Passing Arguments

Both you and Claude can pass arguments when invoking a skill. Arguments are available via `$ARGUMENTS`:

```markdown
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---
Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Search the codebase for relevant files
3. Implement the fix
4. Write tests
5. Create a commit
```

Usage: `/fix-issue 1234` — Claude receives "Fix GitHub issue 1234 following our coding standards..."

If your skill doesn't include `$ARGUMENTS`, Claude Code appends `ARGUMENTS: <your input>` to the end automatically.

### Positional Arguments

Use `$ARGUMENTS[N]` or shorthand `$N` for specific arguments:

```markdown
---
name: migrate-component
description: Migrate a component from one framework to another
---
Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```

`/migrate-component SearchBar React Vue` replaces `$0` with `SearchBar`, `$1` with `React`, `$2` with `Vue`.

### All String Substitutions

| Variable | Description |
|---|---|
| `$ARGUMENTS` | All arguments passed when invoking |
| `$ARGUMENTS[N]` | Specific argument by 0-based index |
| `$N` | Shorthand for `$ARGUMENTS[N]` |
| `${CLAUDE_SESSION_ID}` | Current session ID (useful for logging) |

---

## Dynamic Context Injection

The `` !`command` `` syntax runs shell commands **before** the skill content reaches Claude. The output replaces the placeholder:

```markdown
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---
## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
```

When this skill runs:
1. Each `` !`command` `` executes immediately (before Claude sees anything)
2. The output replaces the placeholder
3. Claude receives the fully-rendered prompt with actual data

This is preprocessing — Claude only sees the final result, not the commands.

---

## Running Skills in a Subagent

Add `context: fork` to run a skill in an isolated subagent. The skill content becomes the prompt that drives the subagent — it won't have access to your conversation history.

```markdown
---
name: deep-research
description: Research a topic thoroughly
context: fork
agent: Explore
---
Research $ARGUMENTS thoroughly:

1. Find relevant files using Glob and Grep
2. Read and analyze the code
3. Summarize findings with specific file references
```

When this runs:
1. A new isolated context is created
2. The subagent receives the skill content as its prompt
3. The `agent` field determines the execution environment (model, tools, permissions)
4. Results are summarized and returned to your main conversation

The `agent` field accepts built-in types (`Explore`, `Plan`, `general-purpose`) or custom agents from `.claude/agents/`. Defaults to `general-purpose` if omitted.

**Warning:** `context: fork` only makes sense for skills with explicit instructions. A skill containing guidelines without a task gives the subagent nothing actionable.

### How Skills and Subagents Interact (Two Directions)

| Approach | System prompt | Task | Also loads |
|---|---|---|---|
| Skill with `context: fork` | From agent type (`Explore`, `Plan`, etc.) | SKILL.md content | CLAUDE.md |
| Subagent with `skills` field | Subagent's markdown body | Claude's delegation message | Preloaded skills + CLAUDE.md |

With `context: fork`, you write the task in your skill and pick an agent type to execute it. For the inverse — defining a custom subagent that uses skills as reference material — see below.

### Preloading Skills into Subagents (Inverse Pattern)

You can also inject skills into custom subagents using the `skills` field in the agent's frontmatter:

```markdown
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---
Implement API endpoints. Follow the conventions and patterns from the preloaded skills.
```

The full content of each listed skill is injected into the subagent's context at startup. See [Custom Subagents](custom-agents.md) for details.

---

## Hooks Inside Skills

Skills can define hooks in their frontmatter. These are scoped to the skill's lifecycle and cleaned up when it finishes:

```yaml
---
name: secure-operations
description: Perform operations with security checks
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

Add `once: true` to a hook handler to run it only once per session, then auto-remove. (Note: `once` is available in skill hooks but not agent hooks.)

---

## Restricting Tool Access

Use `allowed-tools` to limit which tools Claude can use when a skill is active:

```markdown
---
name: safe-reader
description: Read files without making changes
allowed-tools: Read, Grep, Glob
---
```

---

## Permissions and Skill Access

Three ways to control which skills Claude can invoke:

1. **Disable all skills** — deny the `Skill` tool in `/permissions`
2. **Allow/deny specific skills** — use permission rules:
   ```
   Skill(commit)        # exact match
   Skill(review-pr *)   # prefix match with any arguments
   ```
3. **Hide individual skills** — add `disable-model-invocation: true` to their frontmatter

Note: `user-invocable` only controls menu visibility, not Skill tool access. Use `disable-model-invocation: true` to block programmatic invocation. Built-in commands like `/compact` and `/init` are not available through the Skill tool.

---

## Context Budget for Skill Descriptions

Skill descriptions are loaded at startup so Claude knows what's available. If you have many skills, they may exceed the character budget:

- Budget scales dynamically at **2% of the context window** (fallback: **16,000 characters**)
- Run `/context` to check for warnings about excluded skills
- Override with the `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable

---

## Relationship to Legacy Custom Commands

Custom slash commands (`.claude/commands/*.md`) have been merged into skills. Both create `/command-name` and work the same way. Existing `.claude/commands/` files keep working. Skills add optional features: a directory for supporting files, frontmatter to control invocation, and auto-loading when relevant.

If a skill and a command share the same name, the skill takes precedence.

---

## Sharing Skills

| Method | How |
|---|---|
| **Team (project-level)** | Commit `.claude/skills/` to version control |
| **Plugins** | Include a `skills/` directory in your plugin (see [Plugins Deep Dive](plugins.md)) |
| **Organization-wide** | Deploy through managed settings |

---

## Advanced Pattern: Generating Visual Output

Skills can bundle and run scripts to generate output beyond what's possible in a prompt. A common pattern is generating interactive HTML files:

```markdown
---
name: codebase-visualizer
description: Generate interactive visualization of the codebase
allowed-tools: Bash(python *)
disable-model-invocation: true
---
Run the visualization script at scripts/visualize.py to generate
an interactive HTML report of the codebase structure.
```

This works for dependency graphs, test coverage reports, API documentation, database schema visualizations, etc. Use only Python built-in libraries to avoid install steps.

---

## Extended Thinking in Skills

Include the word "ultrathink" anywhere in your skill content to enable extended thinking when the skill runs.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| **Skill not triggering** | Check that the `description` includes keywords matching your requests. Ask "What skills are available?" to verify. Try `/skill-name` directly. |
| **Skill triggers too often** | Make the description more specific. Add `disable-model-invocation: true` for manual-only. |
| **Claude doesn't see all skills** | Context budget exceeded — run `/context` to check. Override with `SLASH_COMMAND_TOOL_CHAR_BUDGET`. |

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Create a skill for my team | Add `SKILL.md` to `.claude/skills/<name>/` and commit |
| Create a personal skill | Add `SKILL.md` to `~/.claude/skills/<name>/` |
| Make a skill manual-only | Add `disable-model-invocation: true` to frontmatter |
| Hide a skill from the `/` menu | Add `user-invocable: false` to frontmatter |
| Pass arguments to a skill | Use `$ARGUMENTS` or `$0`, `$1` in the content |
| Run a skill in isolation | Add `context: fork` (and optionally `agent: Explore`) |
| Inject dynamic data | Use `` !`command` `` syntax for shell preprocessing |
| Check available skills | Ask Claude or run `/context` |
| Limit tools during a skill | Add `allowed-tools: Read, Grep, Glob` |
| Preload a skill into a subagent | Add `skills: [skill-name]` to agent frontmatter |

For the complete specification, see the [official skills documentation](https://code.claude.com/docs/en/skills).
