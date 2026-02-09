# Claude Code: Permissions & Interaction Modes

How the permission system works, the different interaction modes, and when to use each one.

---

## The Permission Model

Claude Code has a **permission system** that controls which tools can run without your approval. Every tool call is either:

- **Auto-allowed** — runs immediately, no prompt
- **Requires approval** — Claude pauses and asks you to approve or deny

This exists because some tools are safe and read-only (like reading a file), while others have side effects (like running shell commands, writing files, or making web requests).

### What Happens When You Deny a Tool Call

If you deny a tool call:
- Claude does **not** retry the exact same call
- It adjusts its approach — tries a different tool, asks you a question, or explains what it wanted to do
- If it doesn't understand why you denied, it may use AskUserQuestion to clarify

---

## Switching Modes

Press **`Shift+Tab`** to cycle through modes during a session: Default → Auto-accept → Plan. You can also start in a specific mode: `claude --permission-mode plan`.

---

## Interaction Modes

Claude Code offers several modes. The three you'll use most often:

### 1. Ask Mode (Default/Interactive)

**How it works:** Claude asks for permission before running tools that aren't in the auto-allow list. You approve or deny each one.

| Property | Detail |
|---|---|
| **Oversight** | High — you see and approve each significant action |
| **Speed** | Slower — blocked waiting for approvals |
| **Risk** | Low — nothing happens without your consent |

**Best for:**
- Learning how Claude Code works (you see every tool call)
- Working in unfamiliar or sensitive codebases
- Tasks where you want to review each step
- First time using Claude Code on a project

**Typical experience:**
```
You: "Fix the login bug"
Claude: [wants to Read auth.ts] → you approve
Claude: [wants to Grep for error handling] → you approve
Claude: [wants to Edit auth.ts] → you approve (after reviewing the change)
Claude: "Done, here's what I changed..."
```

---

### 2. Auto-Accept Mode

**How it works:** Most or all tool calls are automatically approved. Claude works autonomously, only stopping for truly exceptional situations.

| Property | Detail |
|---|---|
| **Oversight** | Low — Claude acts without asking |
| **Speed** | Fast — no approval bottlenecks |
| **Risk** | Higher — mistakes happen without review gates |

**Best for:**
- Well-understood, routine tasks ("run the tests", "format this file")
- When you trust the scope is limited and safe
- Bulk operations you've done before
- When you're watching Claude work and can interrupt if needed

**Caution areas:**
- Git operations (especially push, force-push, reset)
- File deletions
- Running commands with external side effects
- Modifying configuration files

**Typical experience:**
```
You: "Find and fix all TypeScript strict mode errors in src/"
Claude: [reads, searches, edits 8 files automatically]
Claude: "Fixed 12 strict mode errors across 8 files. Here's a summary..."
```

---

### 3. Plan Mode

**How it works:** Claude enters a **read-only exploration phase**. It can search, read files, and analyze the codebase, but **cannot edit, write, or execute anything**. It produces a plan document for your review. Only after you approve does it proceed to implementation.

| Property | Detail |
|---|---|
| **Oversight** | Very high — full plan review before any changes |
| **Speed** | Slower — two-phase process (plan then implement) |
| **Risk** | Very low — nothing changes until you say so |

**How to enter plan mode:**
- Say: "plan this first", "enter plan mode", "design the approach before coding"
- Claude calls `EnterPlanMode` and switches to read-only exploration
- It investigates the codebase, designs an approach, and writes a plan
- It calls `ExitPlanMode` when the plan is ready for your review
- You approve, request changes, or reject
- After approval, Claude implements the plan

**What Claude CAN do in plan mode:**
- Read files (Read, Glob, Grep)
- Spawn Explore/Plan subagents
- Search the web (WebSearch, WebFetch)
- Ask you questions (AskUserQuestion)
- Write to the plan file only

**What Claude CANNOT do in plan mode:**
- Edit any project files
- Write new files (except the plan file)
- Run Bash commands that modify things
- Make commits or push code

**Best for:**
- Complex features with multiple valid approaches
- Architectural changes affecting many files
- When you want to understand the full scope before any code changes
- When the cost of a wrong approach is high
- When working with unfamiliar codebases

**Typical experience:**
```
You: "Plan how to add OAuth2 support"
Claude: [enters plan mode]
Claude: [reads auth files, explores the codebase, checks dependencies]
Claude: [asks you a clarifying question about provider preference]
Claude: [writes a plan: files to modify, approach, trade-offs]
Claude: "Here's my plan — ready for review."
You: "Looks good, but use PKCE flow instead"
Claude: [updates plan]
You: "Approved"
Claude: [exits plan mode, begins implementation]
```

---

### Other Modes

| Mode | What It Does | When to Use |
|---|---|---|
| **dontAsk** | Auto-denies all tools unless pre-approved via `/permissions` | Scripting where you want minimal interaction |
| **Delegate** | Coordination-only — Claude works through [agent team](https://code.claude.com/docs/en/agent-teams) members, no direct edits | Advanced: multi-agent workflows |
| **bypassPermissions** | Skips all permission checks | **Dangerous** — only in sandboxed/isolated environments |

---

## Permission Configuration

### Manage with `/permissions`

Use `/permissions` to view and manage permission rules interactively. Rules come in three types:

- **Allow** — tool runs without asking
- **Ask** — prompts for confirmation
- **Deny** — blocks the tool entirely

Rules are evaluated in order: **deny → ask → allow**. Deny always wins.

### Settings Files

| File | Scope |
|---|---|
| `~/.claude/settings.json` | Global — applies to all projects |
| `.claude/settings.json` | Project — applies to this project |

### Permission Rule Syntax

Rules follow the format `Tool` or `Tool(specifier)`:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)"
    ],
    "deny": [
      "Bash(git push *)"
    ]
  }
}
```

| Example Rule | What It Matches |
|---|---|
| `Bash` | All Bash commands |
| `Bash(npm run *)` | Any command starting with `npm run ` |
| `Bash(git commit *)` | Git commit commands |
| `Read(./.env)` | Reading the .env file |
| `WebFetch(domain:example.com)` | Fetch requests to example.com |
| `mcp__puppeteer__*` | All tools from the puppeteer MCP server |

Read/Edit rules follow [gitignore patterns](https://git-scm.com/docs/gitignore): `*` matches within a directory, `**` matches recursively.

For the full permission rule reference, see the [official permissions documentation](https://code.claude.com/docs/en/permissions).

### Sandboxing

For stronger isolation, `/sandbox` enables OS-level filesystem and network restrictions for Bash commands. This is complementary to permissions — permissions control which tools Claude attempts to use, sandboxing restricts what Bash commands can actually access. See the [official sandboxing docs](https://code.claude.com/docs/en/sandboxing).

### The `--dangerously-skip-permissions` Flag

The CLI supports a flag that **bypasses all permission checks**. As the name implies, this is dangerous:
- Every tool call runs without any approval
- Useful for CI/CD pipelines or fully automated workflows
- **Not recommended for interactive use** — removes all safety guardrails
- Organizations can disable this with managed settings

---

## Choosing the Right Mode

| Scenario | Recommended Mode | Why |
|---|---|---|
| First time on a project | Ask | Learn the codebase safely |
| Quick bug fix you understand | Auto-accept | Fast, low-risk |
| Adding a major feature | Plan → Auto-accept | Design first, then execute |
| Refactoring across many files | Plan | Understand full scope before changing anything |
| Running tests or builds | Auto-accept | Routine, predictable |
| Modifying CI/CD or infra configs | Ask | High-impact changes need review |
| Exploring / learning the codebase | Ask or Plan | No changes needed, just reading |
| Bulk rename or formatting | Auto-accept | Mechanical, safe |
| Deleting or restructuring code | Ask | Irreversible actions need oversight |

---

## Mode Switching During a Session

Press **`Shift+Tab`** to cycle modes, or tell Claude what you want:

- Start in Ask mode to understand the problem
- Press `Shift+Tab` twice to enter Plan mode for the approach
- After plan approval, press `Shift+Tab` to auto-accept for implementation
- Switch back to Ask mode for the final review/commit

This is often the most effective workflow: **high oversight for decisions, low oversight for execution**.

---

## Key Safety Behaviors

Regardless of mode, Claude Code has built-in safety behaviors:

1. **Destructive actions always warrant caution** — force push, `rm -rf`, dropping tables, etc. Even in auto-accept, Claude is instructed to be careful with these.

2. **Shared/external actions get extra scrutiny** — pushing code, creating PRs, posting comments, sending messages. These affect others and are hard to undo.

3. **Secrets are protected** — Claude avoids committing `.env` files, credentials, or API keys. It warns if you ask it to.

4. **Git safety** — Claude never updates git config, avoids force-push to main/master, creates new commits rather than amending by default.

5. **No brute-forcing** — if an approach is blocked, Claude tries alternatives rather than retrying the same failing action.

6. **Checkpointing** — every file edit is snapshotted. Press `Esc+Esc` to rewind if something goes wrong.

---

## Quick Reference

| Mode | Oversight | Speed | Best For |
|---|---|---|---|
| **Ask** | High | Slow | Learning, sensitive code, first-time use |
| **Auto-accept** | Low | Fast | Routine tasks, trusted operations |
| **Plan** | Very high | Slowest | Complex features, architectural decisions |

| I want to... | Do this... |
|---|---|
| Review every action | Use Ask mode (default) |
| Let Claude work fast | Use Auto-accept mode (`Shift+Tab`) |
| Design before coding | Plan mode (`Shift+Tab` twice) or say "plan this first" |
| Switch modes quickly | `Shift+Tab` to cycle |
| Allow specific commands permanently | `/permissions` → add an allow rule |
| Block a tool entirely | `/permissions` → add a deny rule |
| Enable OS-level isolation | `/sandbox` |
| Undo something Claude did | `Esc+Esc` to rewind |
