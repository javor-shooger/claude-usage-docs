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
| `/init` | Create a `CLAUDE.md` file in the current project root (analyzes your project to generate a starter) |
| `/cost` | Show token usage and cost for the current session (API users) |
| `/stats` | Show usage patterns (subscription users — Pro, Max, Teams) |
| `/context` | Show what's consuming context space (tools, MCP servers, conversation) |
| `/model` | Switch the model mid-session. Also configure effort level for extended thinking. |
| `/memory` | Open your memory files (CLAUDE.md + auto memory) in your system editor |
| `/agents` | View, create, edit, and manage custom subagents |
| `/resume` | Switch to a different conversation (opens session picker) |
| `/rename` | Give the current session a memorable name (e.g., `/rename auth-refactor`) |
| `/rewind` | Open the rewind menu to restore conversation and/or code to a previous checkpoint |
| `/config` | Toggle global settings (e.g., thinking mode on/off) |
| `/permissions` | View and manage tool permission rules |
| `/mcp` | Check configured MCP servers and their per-server context costs |
| `/statusline` | Configure the status bar shown at the bottom of the terminal |
| `/hooks` | Interactive hook configuration |
| `/plugin` | Browse and install plugins from the marketplace |
| `/sandbox` | Enable OS-level sandboxing for Bash commands |
| `/fast` | Toggle fast mode (same model, faster output) |
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
| `/commit-push-pr` | Commits, pushes, and opens a PR in one step |
| `/review` | Reviews a PR — analyzes changes, identifies issues, provides feedback |

Additional skills may be available depending on your setup — they appear in system reminders during your session.

### Custom Skills

You can create your own skills by adding `SKILL.md` files in `.claude/skills/`. Each skill is a directory with a `SKILL.md` file containing YAML frontmatter and instructions. Skills load on demand (not at session start), keeping your base context lean. See [Extending Claude Code](extending-claude-code.md) for details.

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
| Rewind to a previous checkpoint | `Escape` twice (Esc+Esc) or `/rewind` |
| Background a running task | `Ctrl+B` |

Pressing Escape once while Claude is generating will stop it mid-response. Pressing Escape **twice** opens the **rewind menu** — letting you restore conversation and/or code to any previous checkpoint.

### Mode & Display Shortcuts

| Action | Keys |
|---|---|
| Cycle permission modes | `Shift+Tab` (cycles: default → auto-accept → plan) |
| Toggle verbose/thinking display | `Ctrl+O` |
| Toggle extended thinking on/off | `Alt+T` (Windows/Linux) or `Option+T` (macOS) |
| Edit plan in your text editor | `Ctrl+G` (when in plan mode) |

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
claude --continue          # Resume the most recent conversation
claude --resume            # Open a session picker or resume by name
claude --resume auth-refactor   # Resume a named session directly
claude --from-pr 123       # Resume sessions linked to a specific PR
```

### Other Useful Flags

```bash
claude --add-dir ../shared  # Give Claude access to an additional directory
claude --fork-session       # Branch off a resumed session without affecting the original
claude --model opus         # Start with a specific model
claude --permission-mode plan  # Start in plan mode
```

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

### `@` References (Quick Context)

Use `@` to include files or directories directly in your message — their content is sent immediately without waiting for Claude to read them:

```
"Explain the logic in @src/utils/auth.js"
"What's the structure of @src/components?"
"Compare @file1.js and @file2.js"
```

- File paths can be relative or absolute
- Directory references show file listings, not full contents
- You can reference multiple files in a single message

### Natural Language References

You can also just describe files naturally — Claude will use the Read tool to fetch them:

```
"look at src/auth.ts"
"what's in the config file?"
"read package.json"
```

### Drag and Drop / Paste

- **Drag and drop** files into the Claude Code input in supported terminals/IDEs
- **Copy and paste** images with `Ctrl+V` (not `Cmd+V` on Mac)

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

## Where Claude Code Runs

Claude Code runs in multiple environments. The most common:

| Platform | Best For |
|---|---|
| **Terminal (CLI)** | Core experience — full control, scripting, SSH access |
| **VS Code extension** | IDE integration — inline diffs, selection context, clickable references |
| **Desktop app** | Standalone app with diff review and parallel sessions via git worktrees |
| **Claude Code on the web** | Browser-based at [claude.ai/code](https://claude.ai/code) — no local setup, parallel tasks |
| **JetBrains plugin** | IntelliJ, PyCharm, WebStorm with IDE diff viewing |

For details on each platform, see the [official platform docs](https://code.claude.com/docs/en/overview#use-claude-code-everywhere).

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
| Free up context | `/compact` (optionally with a focus hint) |
| Check what's using context | `/context` |
| Check token cost | `/cost` (API) or `/stats` (subscription) |
| Start fresh without quitting | `/clear` |
| Create project instructions | `/init` |
| Edit memory/CLAUDE.md files | `/memory` |
| Stop Claude mid-response | `Escape` |
| Undo Claude's changes | `Esc+Esc` or `/rewind` |
| Cycle permission modes | `Shift+Tab` |
| Toggle fast mode | `/fast` |
| Write a multi-line message | `Shift+Enter` for new lines |
| Quick one-off question | `claude -p "question"` (CLI only) |
| Continue last session | `claude --continue` (CLI only) |
| Pick from recent sessions | `claude --resume` or `/resume` |
| Name a session | `/rename auth-refactor` |
| Switch to a cheaper model | `/model` |
| Include a file in your message | `@path/to/file` |
| Manage subagents | `/agents` |
| Browse plugins | `/plugin` |
| End the session | `/quit` |
| Ask about highlighted code | Select code in editor, then ask (VS Code) |
| Jump to a file Claude mentions | Click the file link in the response (VS Code) |
