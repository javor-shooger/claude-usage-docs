# Claude Code: Workflows & Patterns

*Last verified against [official docs](https://code.claude.com/docs/en/) on 2026-02-09*

Common effective patterns for real tasks — recipes you can follow and adapt.

---

## Core Principle

The most effective Claude Code sessions follow a pattern:
**Narrow the scope → Understand the context → Take action → Verify the result**

Jumping straight to action without understanding leads to wasted context and wrong changes. Over-exploring without acting wastes your session budget. The recipes below balance both.

---

## Workflow 1: Plan → Implement

**For:** Adding features, refactoring, architectural changes — anything non-trivial.

```
Phase 1: Plan
  "Plan how to add [feature]. Don't write any code yet."
  → Claude enters plan mode
  → Explores codebase, identifies files, considers approaches
  → Presents plan for your review

Phase 2: Review
  → You review, ask questions, request adjustments
  → Approve when satisfied

Phase 3: Implement
  → Claude implements the plan step by step
  → You can watch or auto-accept depending on comfort

Phase 4: Verify
  "Run the tests" / "Build the project"
  → Confirm everything works
```

**Why it works:** Mistakes in the planning phase are free — mistakes in the implementation phase cost context and may need reverting.

**Tips:**
- After planning, consider `/compact focus on the implementation plan` before starting Phase 3. This drops the exploration context from planning and keeps the session lean for implementation.
- If the plan was long and exploratory, `/clear` + re-stating the plan is even cleaner than compacting.

---

## Workflow 2: Explore → Fix

**For:** Bug fixing, understanding unfamiliar code, investigating issues.

```
Step 1: Describe the symptom
  "The /api/users endpoint returns 500 when the email field is missing"

Step 2: Let Claude investigate
  → Claude searches for the endpoint, reads the handler, traces the error
  → (Use a subagent for broad investigation: "Use an explore agent to
     trace the /api/users endpoint from route to database")

Step 3: Review the diagnosis
  → Claude explains what's wrong and why

Step 4: Fix
  "Fix it"
  → Claude makes the targeted edit

Step 5: Verify
  "Run the tests for the users module"
```

**Why it works:** Understanding the bug before fixing it leads to correct fixes, not band-aids.

---

## Workflow 3: Targeted Edit

**For:** Small, specific changes where you know exactly what to do.

```
"In src/config.ts, change the timeout from 5000 to 10000"
```

or

```
"Add a `createdAt` field to the User interface in src/types.ts"
```

**No exploration, no planning — just do it.** This is the fastest workflow and appropriate when:
- You know the exact file and location
- The change is small and clear
- There's no ambiguity about what to do

---

## Workflow 4: Search → Understand → Modify

**For:** Working in unfamiliar codebases or making changes that touch unknown patterns.

```
Step 1: Search
  "Find all files that handle email sending"
  → Grep/Glob identifies relevant files

Step 2: Understand
  "Show me how sendEmail works in src/services/email.ts"
  → Read the key file(s)

Step 3: Modify
  "Add a CC field to the email sending function"
  → Claude makes the change with full context of the existing patterns

Step 4: Find ripple effects
  "Search for all callers of sendEmail"
  → Verify if any callers need updating too
```

**Why it works:** Understanding existing patterns before modifying them means your changes fit naturally into the codebase.

---

## Workflow 5: Test-Driven

**For:** Bug fixes with clear expected behavior, or new features with testable requirements.

```
Step 1: Write the test first
  "Write a test that verifies the discount is applied correctly
   for orders over $100"

Step 2: Run the test (expect failure)
  "Run the tests"
  → Confirms the test fails for the right reason

Step 3: Implement
  "Now implement the discount logic to make the test pass"

Step 4: Run tests again
  "Run the tests"
  → All green
```

**Why it works:** The test anchors the requirement. Claude knows exactly what "done" looks like.

---

## Workflow 6: Parallel Investigation

**For:** Complex problems that need research across multiple areas simultaneously.

```
"In parallel:
 - Use an explore agent to investigate how auth works in this project
 - Use another explore agent to find all the API middleware
 - Use a third to check the test coverage for the auth module"

→ Three agents work simultaneously
→ Each returns a summary
→ You have a comprehensive picture without burning your main context
```

**When to use:** When a problem spans multiple subsystems, or when you need to compare approaches across different parts of the codebase.

---

## Workflow 7: PR / Commit Flow

**For:** Wrapping up work and creating commits or pull requests.

### Quick Commit
```
/commit
→ The /commit skill analyzes staged changes, drafts a conventional message, creates the commit
→ Much better than "commit these changes" — the skill has detailed instructions for message quality
```

Or manually: `"Commit these changes"` — works, but `/commit` produces more consistent results.

### Full PR Flow
```
Step 1: Verify your work
  "Run the tests and build"

Step 2: Commit + PR
  /commit
  → Skill handles analysis, message drafting, and commit creation

  "Create a pull request"
  → Claude creates a PR with title, summary, and test plan
  → Returns the PR URL
```

**Shortcut:** If you have a `/commit-push-pr` skill set up, it can handle commit + push + PR creation in one step. See [Skills Deep Dive](skills.md) to create your own workflow skills.

### PR Review
```
/review 123
→ The /review skill fetches the PR diff, comments, and checks
→ Provides a structured review
```

---

## Workflow 8: Iterative Debugging

**For:** When the bug isn't obvious and needs systematic investigation.

```
Step 1: Reproduce
  "Run this command and show me the output: [failing command]"

Step 2: Hypothesize
  Claude identifies potential causes

Step 3: Gather evidence
  "Check the logs" / "Add a console.log in the handler" / "Read the config"

Step 4: Narrow down
  Repeat steps 2-3 until the root cause is found

Step 5: Fix and verify
  "Fix it and run the tests"
```

**Tips:**
- If the investigation is getting long (lots of file reads), delegate to a subagent: "Use an agent to investigate why the WebSocket connection drops after 30 seconds."
- Use `/model` to switch to Haiku for the "run and check output" steps (Steps 1, 3) — save Opus/Sonnet for the reasoning steps (Steps 2, 4, 5).

---

## Workflow 9: Persistent Task File (Cross-Session)

**For:** Large projects that span multiple sessions, or when you want different sessions/subagents to pick up work where the last one left off.

### The Problem

TodoWrite (the built-in todo list) is **session-scoped** — it disappears when the session ends. If you're working on a multi-day feature with 20 tasks, you lose your checklist every time you start a new session.

### The Solution

Create a **task file** in your project that Claude reads and updates like any other file. Because it's a real file on disk, it persists forever and any session or subagent can access it.

### How to Set It Up

**Step 1:** Create a task file in your project:

```markdown
<!-- tasks.md -->
# Current Tasks

## In Progress
- [ ] Add OAuth2 login flow to the auth module
  - [x] Install dependencies (passport, passport-google-oauth20)
  - [ ] Create OAuth strategy in src/auth/google.strategy.ts
  - [ ] Add callback route

## Up Next
- [ ] Add email verification after registration
- [ ] Write integration tests for auth endpoints

## Done
- [x] Set up JWT token generation
- [x] Create login/register API endpoints
```

**Step 2:** Add an instruction to your CLAUDE.md:

```markdown
## Task Tracking
- At the start of each session, read `tasks.md` to understand current progress
- Update `tasks.md` as you complete tasks (move items, check boxes)
- When starting a task, mark it as in-progress
- When a task reveals sub-tasks, add them to the file
```

**Step 3:** Use it naturally:

```
Session 1: "Read tasks.md and work on the next item"
  → Claude reads the file, picks up the OAuth strategy task, works on it, updates the file

Session 2 (next day): "Check tasks.md and continue where we left off"
  → Claude reads the file, sees what's done and what's next, continues
```

### Key Points

- **The tasks are your own free text.** There's no special format required — markdown checklists, JSON, plain text, whatever works for you.
- **Claude reads and writes it with normal Read/Write tools.** Nothing magical — it's just a file.
- **Subagents can read it too.** A subagent can check the task file to understand what's been done and what remains.
- **You can edit it yourself.** It's your file — add tasks, reprioritize, remove things. Claude will see the changes next time it reads it.

### Variations

| Format | When to use |
|---|---|
| `tasks.md` (markdown checklist) | Most projects — human-readable, easy to edit |
| `tasks.json` (structured JSON) | When you want machine-parseable status fields, priorities, assignments |
| `.claude/tasks.md` | If you want to keep it out of your project's main directory |

### TodoWrite vs Task File

| Property | TodoWrite (built-in) | Task file (this pattern) |
|---|---|---|
| **Persists across sessions** | No | Yes |
| **Visible in UI** | Yes (progress bar) | No (just a file) |
| **Survives compaction** | Yes | Yes (it's on disk) |
| **Editable by you** | No | Yes |
| **Readable by subagents** | No | Yes |
| **Setup required** | None | Create file + CLAUDE.md instruction |
| **Best for** | Single-session multi-step work | Multi-session projects |

**They complement each other:** Use TodoWrite for tracking steps within a session, and a task file for tracking work across sessions.

---

## Workflow 10: Cost-Efficient Feature Build

**For:** Larger features where you want to minimize token spend by matching the model to each phase.

```
Phase 1: Plan (Opus)
  "Plan how to add the notification system"
  → Opus handles the complex architectural reasoning
  → Approve the plan

Phase 2: Compact
  /compact focus on the implementation plan
  → Drop the exploration context, keep the plan

Phase 3: Implement (Sonnet)
  /model → switch to Sonnet
  "Implement step 1 of the plan"
  → Sonnet handles standard coding tasks efficiently

Phase 4: Test & Fix (Haiku for runs, Sonnet for fixes)
  /model → switch to Haiku
  "Run the tests"
  → Haiku runs commands cheaply
  /model → switch to Sonnet (if fixes needed)
  "Fix the failing test"

Phase 5: Commit
  /commit
  → Skill handles the commit workflow
```

**Why it works:** Opus costs significantly more than Sonnet, which costs more than Haiku. By matching the model to the task complexity, you get Opus-quality planning with Sonnet-cost implementation and Haiku-cost test runs.

**Shortcut:** Use `/model opusplan` to automate this. The `opusplan` alias automatically uses Opus during plan mode and switches to Sonnet for implementation — no manual `/model` switching needed.

**See also:** [Cost Awareness & Model Selection](cost-and-models.md) for detailed cost strategies.

---

## Workflow 11: Code Review Session (Custom Agent)

**For:** Regular code review using a consistent set of review criteria.

### Setup (one-time)

Create `.claude/agents/code-reviewer.md`:
```markdown
---
name: code-reviewer
description: Senior code reviewer — checks correctness, security, and maintainability
tools: Read, Glob, Grep, Bash
---
When reviewing code:
- Check for correctness, security, and maintainability
- Flag any OWASP top 10 vulnerabilities
- Verify error handling for all async operations
- Check that new code has tests
- Keep feedback constructive — cite specific line numbers
```

### Usage

```
Start a session as the code-reviewer agent

Step 1: Get context
  "Use an explore agent to understand the changes on this branch
   compared to main"
  → Subagent reads the diff, returns a summary

Step 2: Deep review
  "Review src/auth/oauth.ts — focus on the new OAuth flow"
  → Claude reviews with the reviewer persona's criteria

Step 3: Summarize
  "Write a review summary I can post as a PR comment"
```

**Why it works:** The custom agent loads your review criteria automatically. Subagents handle the heavy exploration, keeping your review session context clean for the actual review work.

**See also:** [Custom Subagents](custom-agents.md) for setup details.

---

## Workflow 12: Rewind & Retry

**For:** When Claude's implementation went wrong and you want to try a different approach.

```
Step 1: Realize the approach isn't working
  → Claude has made several edits that aren't what you wanted

Step 2: Rewind
  Press Esc+Esc (or use /rewind)
  → Choose a checkpoint from before the bad changes
  → All file edits are rolled back to that point

Step 3: Retry with better guidance
  "Try a different approach — instead of X, do Y"
  → Claude starts fresh from the clean state
```

**Why it works:** Without rewind, you'd need Claude to manually undo changes (spending context) or start a new session. Rewind is instant and free — it restores file snapshots without consuming any tokens.

**Tip:** Rewind is especially valuable in auto-accept mode, where Claude may make several changes before you notice a problem.

---

## Anti-Patterns to Avoid

### 1. The Vague Dump
> "Fix my project"

**Why it's bad:** Claude doesn't know where to start, explores broadly, burns context.
**Instead:** "The build fails with error X. The relevant code is in src/Y."

### 2. The Unnecessary Deep Dive
> "Read every file in the src directory"

**Why it's bad:** Massive context cost with no targeted purpose.
**Instead:** "Search for files related to authentication" or use a subagent.

### 3. Editing Without Reading
> "Add error handling to the processOrder function"

**Why it's bad:** Claude might not know the current implementation, existing patterns, or surrounding code.
**Instead:** Claude will (and should) read the file first. Let it.

### 4. Ignoring Plan Mode for Big Changes
> "Refactor the entire data layer" (in auto-accept mode)

**Why it's bad:** Claude starts changing files immediately without a cohesive plan. Halfway through, you might disagree with the approach.
**Instead:** "Plan how to refactor the data layer. Don't change anything yet."

### 5. Staying in One Session Too Long
> Session running for hours, multiple compactions, losing thread

**Why it's bad:** After heavy compaction, Claude loses nuanced details. You spend tokens re-explaining things.
**Instead:** Run `/clear` between unrelated tasks, or start a fresh session. Session memory carries the essentials forward.

---

## Quick Reference: Choosing a Workflow

| Task Type | Recommended Workflow |
|---|---|
| Small, clear edit | Targeted Edit |
| Bug with known location | Explore → Fix |
| Bug with unknown cause | Iterative Debugging |
| New feature (simple) | Search → Understand → Modify |
| New feature (complex) | Plan → Implement |
| Refactoring | Plan → Implement |
| Learning unfamiliar code | Parallel Investigation |
| Writing tests | Test-Driven |
| Wrapping up work | PR / Commit Flow (`/commit`, `/review`) |
| Multi-session project | Persistent Task File |
| Budget-conscious feature work | Cost-Efficient Feature Build |
| Regular code reviews | Code Review Session (custom subagent) |
| Claude went the wrong direction | Rewind & Retry (`Esc+Esc`) |

| Situation | Mode / Model to Use |
|---|---|
| Not sure what approach to take | Plan mode |
| Confident in the approach, many files to change | Auto-accept |
| Working in sensitive code | Ask mode |
| Routine tasks (tests, builds, formatting) | Auto-accept + Haiku |
| Complex reasoning or architecture | Opus |
| Standard coding tasks | Sonnet |
| Running commands, simple edits | Haiku |
| Switching between modes quickly | `Shift+Tab` to cycle |
| Undoing bad changes | `Esc+Esc` to rewind |
