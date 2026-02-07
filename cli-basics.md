# Claude Code: Getting Started & Daily Controls

The everyday commands, shortcuts, and controls — covering both the CLI and the VS Code extension.

---

## Slash Commands

Slash commands are typed directly into the input prompt. They are **built-in CLI commands**, not tool calls — they execute instantly on the client side.

| Command | What It Does |
|---|---|
| `/help` | Show help and available commands |
| `/compact` | Manually compress conversation to free context. Optionally add a focus: `/compact focus on the auth work` |
| `/clear` | Clear the conversation history and start fresh (same session) |
| `/init` | Create a `CLAUDE.md` file in the current project root |
| `/cost` | Show token usage and cost for the current session |
| `/status` | Show session info (model, context usage, project) |
| `/model` | Switch the model mid-session (e.g., to Sonnet for cheaper tasks) |
| `/quit` or `/exit` | End the session |
| `/review` | Review a PR (provide PR number or URL) |
| `/bug` | Report a bug |

### Slash Commands vs Skills

These look similar (both start with `/`) but work completely differently:

| | Slash Commands | Skills |
|---|---|---|
| **Examples** | `/compact`, `/clear`, `/model`, `/cost` | `/commit`, `/review`, custom skills |
| **How they run** | Built into the CLI — execute instantly on the client side, no model call | Invoked via the Skill tool — expand into a full prompt that Claude then executes |
| **Speed** | Instant | Takes a full model turn (same as any other Claude action) |
| **What they do** | UI/session management (clear history, switch model, show cost) | Complex multi-step workflows (analyze changes, draft commit message, run git commands) |

### How Skills Work

When you type `/commit`, Claude doesn't just run `git commit`. What happens:

1. The Skill tool is invoked with the skill name
2. The skill expands into a **detailed prompt** — instructions for analyzing staged changes, drafting a message, following conventions, etc.
3. Claude executes that prompt as a normal turn — reading files, running commands, producing output
4. The result is a thoughtful, multi-step action (not a simple command)

This is why `/commit` produces well-crafted commit messages that analyze your changes, while `/clear` just instantly wipes the history.

### Available Skills

Skills vary by configuration. Common built-in skills:

| Skill | What It Does |
|---|---|
| `/commit` | Analyzes staged changes, drafts a commit message following your project's conventions, creates the commit |
| `/review` | Reviews a PR — analyzes changes, identifies issues, provides feedback |

Additional skills may be available depending on your setup — they appear in system reminders during your session.

### Custom Skills

You can create your own skills by adding prompt files. Custom skills work the same way — they expand into a prompt that Claude executes. This lets you create reusable workflows specific to your project (e.g., a skill that runs your full test + lint + build pipeline, or one that generates changelog entries).

---

## Keyboard Shortcuts & Input Controls

### Submitting Messages

| Action | Keys |
|---|---|
| Submit message | `Enter` |
| New line (without submitting) | `Shift+Enter` |

### During Claude's Response

| Action | Keys |
|---|---|
| Interrupt / stop Claude | `Escape` |

Pressing Escape while Claude is generating will stop it mid-response. This is useful when:
- You see Claude going in the wrong direction
- The output is already long enough
- You want to redirect with a new instruction

### Multi-Line Input

Use `Shift+Enter` to add new lines to your message before submitting. This is useful for:
- Providing structured instructions with bullet points
- Pasting code snippets
- Giving multi-part requests

---

## Launching Claude Code

### Basic Launch

```bash
claude
```

Opens Claude Code in the current directory.

### Launch with a Prompt

```bash
claude "fix the login bug in auth.ts"
```

Starts a session and immediately sends the prompt.

### Launch in Non-Interactive Mode

```bash
claude -p "what does this project do?"
```

The `-p` (print) flag runs a single prompt and exits — useful for scripting or quick questions.

### Resume Previous Session

```bash
claude --resume
```

Continues the most recent session with its full context intact.

---

## Session Lifecycle

### Starting a Session

When you launch Claude Code:
1. System prompt is assembled (identity + rules)
2. Tool definitions are loaded (built-in + MCP)
3. CLAUDE.md files are loaded (global → project → personal → rules)
4. Auto memory (MEMORY.md) is loaded
5. Session memory from past sessions is injected
6. You're ready to chat

### During a Session

- Every message and response accumulates in context
- Tool results (file reads, command output) add to context
- Context grows until compaction is needed
- You can `/compact` manually or let it happen automatically
- Todo lists persist through compaction

### Ending a Session

- `/quit`, `/exit`, or close the terminal
- Session transcript is saved locally as JSONL
- A summary is generated for future session memory
- Auto memory (MEMORY.md) persists any notes Claude recorded
- Next session picks up from CLAUDE.md + memory + session summaries

---

## The Input Flow

When you type a message and press Enter:

```
1. Your message is added to conversation history
2. Full context is assembled:
   system prompt + tools + CLAUDE.md + full history + your message
3. Sent to API as one request
4. Claude responds (possibly with tool calls)
5. Tool calls execute (with permission checks)
6. Tool results are added to history
7. If Claude needs another step, goto 3
8. Final response is displayed
```

Each "turn" may involve **multiple API calls** if Claude uses tools — each tool call → result → next response is a separate API round-trip, but it all happens within one conversational turn.

---

## Working with Files

### Providing File Context

You don't need to manually open or paste files. Just reference them:

```
"look at src/auth.ts"
"what's in the config file?"
"read package.json"
```

Claude will use the Read tool to fetch the file contents.

### Drag and Drop

In supported terminals/IDEs, you can drag files into the Claude Code input to reference them.

### Image and PDF Support

Claude can read images (PNG, JPG) and PDFs:

```
"look at screenshot.png"
"read the first 5 pages of spec.pdf"
```

---

## Terminal Output Awareness

Claude can see the output of commands it runs, but it **cannot see your terminal**. If you want Claude to know about an error you're seeing:

- Paste the error message directly
- Tell Claude to run the command itself: "run `npm test` and check the output"
- Provide a screenshot if it's visual

---

## CLI vs VS Code Extension

Claude Code runs in two interfaces. The **core engine is identical** — same model, same tools, same context window, same CLAUDE.md loading, same permissions. The differences are in the UI layer.

### What's the Same (Everything That Matters)

All of the following work identically in both:
- All 17 built-in tools + any MCP tools
- CLAUDE.md files, auto memory, session memory
- Subagents, plan mode, permissions
- Context window mechanics, compaction
- Slash commands (`/compact`, `/clear`, `/init`, etc.)

### What the VS Code Extension Adds

| Feature | Description |
|---|---|
| **IDE selection context** | Highlight code in the editor, then ask Claude about it — Claude sees your selection automatically. No need to paste or reference line numbers. |
| **File open awareness** | Claude knows which file you currently have open in the editor. This can help it understand what you're working on without you saying it explicitly. |
| **Clickable file references** | File paths and line numbers in Claude's responses become clickable links that open directly in the editor (e.g., clicking `src/auth.ts:42` jumps to line 42). |
| **Integrated panel** | Claude lives in a panel inside VS Code — no separate terminal window. You can chat while looking at your code side by side. |
| **Inline diffs** | When Claude edits files, you may see the changes as inline diffs in the editor. |

### What the CLI Has That the Extension Doesn't

| Feature | Description |
|---|---|
| **Non-interactive mode** | `claude -p "question"` — run a single prompt and get output. Useful for scripts, CI/CD, automation. |
| **Piping** | Pipe input/output to/from other commands (e.g., `cat error.log \| claude -p "explain this error"`). |
| **SSH access** | Use Claude Code on remote servers where VS Code may not be available. |
| **Resume flag** | `claude --resume` to continue the most recent session from the terminal. |

### When to Use Which

| Situation | Better choice |
|---|---|
| Working on code in a project | **VS Code extension** — selection context and clickable references make it faster |
| Quick terminal task (git, build) | **Either** — both work fine |
| Scripting or automation | **CLI** — non-interactive mode and piping |
| Remote server via SSH | **CLI** — no IDE needed |
| Reviewing code side-by-side | **VS Code extension** — panel + editor layout |

### Using the Selection Context (VS Code)

This is the biggest practical advantage of the extension. Instead of:
```
"Look at the handleLogin function in src/auth/controller.ts and explain what it does"
```

You can just:
1. Highlight the `handleLogin` function in the editor
2. Type: "explain this"

Claude sees your selection and knows exactly what "this" refers to. This works for:
- Asking about highlighted code
- Requesting changes to selected code
- Debugging a specific section
- Getting explanations of complex logic

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Free up context | `/compact` |
| Check how much context I've used | `/cost` or `/status` |
| Start fresh without quitting | `/clear` |
| Create project instructions | `/init` |
| Stop Claude mid-response | `Escape` |
| Write a multi-line message | `Shift+Enter` for new lines |
| Quick one-off question | `claude -p "question"` (CLI only) |
| Continue last session | `claude --resume` (CLI only) |
| Switch to a cheaper model | `/model` |
| End the session | `/quit` |
| Ask about highlighted code | Select code in editor, then ask (VS Code) |
| Jump to a file Claude mentions | Click the file link in the response (VS Code) |
