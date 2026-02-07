# Claude Code: Custom Agents

What custom agents are, how they differ from subagents, and when you'd want one.

---

## What Are Custom Agents?

Custom agents are **reusable session-level personas** you define in your project. Each one is a markdown file containing a custom system prompt that shapes how Claude behaves for that session.

Think of them as **saved roles** — instead of writing "You are a senior code reviewer, focus on security issues, check for OWASP top 10..." at the start of every session, you save that as a custom agent and invoke it by name.

---

## Where They Live

```
.claude/agents/
├── code-reviewer.md
├── docs-writer.md
└── test-writer.md
```

Each `.md` file in `.claude/agents/` defines one custom agent. The filename (minus extension) becomes the agent name.

---

## How to Create One

Create a markdown file in `.claude/agents/` with the instructions for that persona:

### Example: Code Reviewer

```markdown
<!-- .claude/agents/code-reviewer.md -->
You are a senior code reviewer. When reviewing code:

- Focus on correctness, security, and maintainability
- Check for OWASP top 10 vulnerabilities
- Flag any use of `any` type in TypeScript
- Verify error handling is present for all async operations
- Keep feedback constructive and specific — cite line numbers
- Don't suggest style changes unless they affect readability
```

### Example: Test Writer

```markdown
<!-- .claude/agents/test-writer.md -->
You are a test engineer. When writing tests:

- Use Jest + React Testing Library
- Follow the describe/it pattern
- Test edge cases, not just happy paths
- Mock external dependencies, never real APIs
- Colocate test files: `Component.test.tsx` next to `Component.tsx`
- Aim for behavior testing, not implementation details
```

---

## Custom Agents vs Subagents

This is the most important distinction to understand — they are **completely different features** that share the word "agent":

| | Custom Agents | Subagents (Task tool) |
|---|---|---|
| **What they are** | Saved persona/instructions | Independent Claude instances spawned mid-session |
| **Where defined** | `.claude/agents/*.md` | Built-in to Claude Code (Explore, Plan, Bash, etc.) |
| **When they run** | At session start — sets the tone for the whole session | Mid-session — spawned for a specific task, returns a result |
| **Context** | Uses YOUR session's context window | Gets its own isolated context window |
| **Types** | Whatever you define | Fixed set: Explore, Plan, Bash, general-purpose, etc. |
| **Can you combine them?** | You work inside a custom agent session | You spawn subagents from any session (including a custom agent session) |

**Key point:** You cannot spawn a custom agent as a subagent. The Task tool only accepts the built-in subagent types (`Explore`, `Plan`, `Bash`, `general-purpose`, `claude-code-guide`, `statusline-setup`).

However, you **can** use subagents from within a custom agent session. If you start a session as the "code-reviewer" agent, you can still spawn Explore subagents to investigate code, Plan subagents to design approaches, etc.

---

## When Custom Agents Are Useful

| Situation | Why a custom agent helps |
|---|---|
| You do code reviews frequently | Save your review criteria once, invoke the reviewer agent each time |
| Different team members want different styles | Each person creates their own agent with their preferences |
| Specialized workflows | A "migration-assistant" agent that knows your migration conventions |
| Teaching/onboarding | A "codebase-guide" agent that explains your project's patterns to new devs |

### When NOT to use them

- **One-off instructions** — just say it in the prompt
- **Project-wide rules** — put those in CLAUDE.md (loaded automatically for everyone)
- **Per-file rules** — use `.claude/rules/` directory
- **Things that should run as subagents** — custom agents aren't subagent types

---

## Custom Agents vs CLAUDE.md

| | Custom Agents | CLAUDE.md |
|---|---|---|
| **Loaded** | Only when you choose to use that agent | Automatically every session |
| **Purpose** | Role-specific behavior for specialized tasks | Project-wide rules and conventions |
| **Scope** | One persona at a time | Always active |
| **Best for** | "Act as a code reviewer" | "Use TypeScript strict mode, tests go in `*.test.ts`" |

They complement each other: CLAUDE.md provides the project baseline, and a custom agent adds role-specific behavior on top.

---

## Practical Example

**Without custom agents:**
```
Session 1: "Review this PR. Focus on security, check for SQL injection,
           verify auth checks on all endpoints, flag any hardcoded secrets..."

Session 2: "Review this PR. Focus on security, check for SQL injection,
           verify auth checks on all endpoints, flag any hardcoded secrets..."

(repeat every time)
```

**With a custom agent:**
```
Create .claude/agents/security-reviewer.md once with all your criteria

Session 1: Start as security-reviewer → "Review this PR"
Session 2: Start as security-reviewer → "Review this PR"

(instructions are pre-loaded every time)
```

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Create a custom agent | Add a `.md` file to `.claude/agents/` |
| Use a custom agent | Select it when starting a session |
| Add project-wide rules | Use CLAUDE.md instead (loaded automatically) |
| Spawn an agent mid-session | Use the Task tool with a built-in subagent type — custom agents can't be spawned as subagents |
| Combine custom agent + subagents | Start a session as a custom agent, then spawn subagents normally from within it |
