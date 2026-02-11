# Claude Code Tools Reference

*Last verified against [official docs](https://code.claude.com/docs/en/) on 2026-02-09*

A complete reference of all 39 tools available in Claude Code.

---

## Tool Sources & Context Cost

### Built-in vs MCP Tools

Tools come from two sources:

| Source | Tools | Loaded When |
|---|---|---|
| **Built-in** | 17 tools (File Ops, Execution, Web, Planning, Other) | **Always loaded** — tool definitions are injected into the system prompt at the start of every session. They are always present regardless of whether you use them. |
| **MCP (Model Context Protocol)** | 22 Playwright tools | **Loaded at session start if configured** — when an MCP server (like Playwright) is registered in your Claude Code settings, its tool schemas are fetched when the session begins and added to the context. They consume context even if never invoked during the session. |

### Context Size Estimates

> **Important:** These are rough estimates based on the tool definition sizes (name + description + JSON schema). Actual token counts depend on the tokenizer, but these give a useful sense of proportion.

| Category | Tool Count | Est. Context Tokens | Notes |
|---|---|---|---|
| **Built-in tools** | 17 | ~4,000–6,000 tokens | Bash alone is ~800-1,000 tokens due to its extensive usage instructions. Most others are ~150–350 tokens each. |
| **Playwright MCP tools** | 22 | ~2,000–3,000 tokens | Each tool is relatively compact (~80–150 tokens), but 22 of them add up. |
| **Combined total** | 39 | ~6,000–9,000 tokens | This is the baseline cost before any conversation begins. |

For reference, Claude's context window is **200,000 tokens**, so the tool definitions consume roughly **3–5%** of the total context.

### Key Takeaways

- **Built-in tools are always present** — you cannot opt out of them. They are core to Claude Code's functionality.
- **MCP tools are present if the MCP server is configured** — they load at session start, not on first invocation. If you have Playwright MCP configured, those 22 tool definitions are in every session's context whether you use them or not.
- **MCP tool search** — when MCP tools exceed ~10% of context, Claude Code automatically defers them (lists tools on demand instead of loading all at startup). Use `/mcp` to check per-server costs.
- **To reduce context cost from MCP tools**, you would need to remove the MCP server configuration from your Claude Code settings for sessions where you don't need browser automation.
- **Skills and plugins can add more tools** — custom skills (`.claude/skills/`) and installed plugins may provide additional capabilities beyond the built-in and MCP tools listed here. See [Skills Deep Dive](skills.md) and [Extending Claude Code](extending-claude-code.md).
- **The system prompt itself** (instructions, CLAUDE.md content, git status, etc.) also consumes significant context — often more than the tool definitions themselves.
- **Output styles** (`/output-style`) change how Claude formats responses — Default, Explanatory, or Learning modes — but are not tools. They modify the system prompt, not the tool set.

---

## Built-in: File Operations (5 tools)

| Tool | Description |
|---|---|
| **Read** | Read file contents. Supports text files, images (PNG, JPG — displayed visually), PDFs (with page ranges via `pages` parameter, max 20 pages per request), and Jupyter notebooks (`.ipynb`). Returns content with line numbers. Can specify `offset` and `limit` for large files. |
| **Write** | Create or overwrite a file at an absolute path. Overwrites existing files. Requires reading the file first if it already exists. |
| **Edit** | Perform exact string replacements in files. The `old_string` must be unique in the file. Use `replace_all: true` to replace every occurrence. Preserves original indentation. |
| **Glob** | Fast file pattern matching. Supports glob patterns like `**/*.js`, `src/**/*.ts`. Returns matching file paths sorted by modification time. |
| **Grep** | Powerful content search built on ripgrep. Supports full regex, glob/type filtering, multiline matching. Output modes: `files_with_matches` (default), `content` (with context lines via `-A`, `-B`, `-C`), and `count`. |

---

## Built-in: Execution (4 tools)

| Tool | Description |
|---|---|
| **Bash** | Execute shell commands with optional timeout (up to 10 minutes). Working directory persists between calls; shell state does not. Supports `run_in_background` for long-running commands. Use for git, npm, msbuild, docker, and other terminal operations. |
| **Task** | Launch a specialized sub-agent to handle complex tasks autonomously. Each agent type has specific capabilities. Available types: `Explore` (codebase exploration), `Plan` (implementation planning), `Bash` (command execution), `general-purpose` (research & multi-step tasks), `claude-code-guide` (Claude Code / API questions), `statusline-setup` (status line config). Supports `run_in_background` and `resume`. |
| **TaskOutput** | Retrieve output from a running or completed background task. Supports blocking (`block: true`) and non-blocking (`block: false`) modes with configurable timeout. |
| **TaskStop** | Stop a running background task by its ID. |

---

## Built-in: Web & Search (2 tools)

| Tool | Description |
|---|---|
| **WebSearch** | Search the internet and return results with links. Supports domain filtering (`allowed_domains`, `blocked_domains`). Returns search result blocks with markdown hyperlinks. |
| **WebFetch** | Fetch content from a URL, convert HTML to markdown, and process it with a prompt using a fast model. Includes a 15-minute cache. Handles redirects. Not suitable for authenticated/private URLs. |

---

## Built-in: Planning & Interaction (4 tools)

| Tool | Description |
|---|---|
| **TodoWrite** | Create and manage a structured task list. Tasks have `content`, `activeForm`, and `status` (`pending`, `in_progress`, `completed`). Useful for tracking multi-step work and showing progress. |
| **EnterPlanMode** | Transition into plan mode for designing implementation approaches before writing code. Used for non-trivial tasks with multiple valid approaches, architectural decisions, or multi-file changes. |
| **ExitPlanMode** | Signal that the plan is finalized and ready for user review/approval. Reads the plan from the plan file. |
| **AskUserQuestion** | Ask the user 1-4 questions with 2-4 selectable options each. Supports `multiSelect`. Users can always provide custom text via "Other". |

---

## Built-in: Other (2 tools)

| Tool | Description |
|---|---|
| **Skill** | Invoke a registered skill by name (e.g., `keybindings-help`). Skills are user-invocable via `/skill-name` shorthand. |
| **NotebookEdit** | Edit Jupyter notebook cells (`.ipynb`). Modes: `replace` (default), `insert`, `delete`. Specify cell by `cell_id` or index. Requires `cell_type` (`code` or `markdown`) for inserts. |

---

## MCP: Playwright Browser Automation (22 tools)

> **Source:** These tools are provided by the Playwright MCP server, not built into Claude Code.
> They are loaded into context at session start when the MCP server is configured.
> Prefix in tool calls: `mcp__playwright__`

### Navigation

| Tool | Description |
|---|---|
| **browser_navigate** | Navigate to a specified URL. |
| **browser_navigate_back** | Go back to the previous page in browser history. |

### Page Inspection

| Tool | Description |
|---|---|
| **browser_snapshot** | Capture an accessibility snapshot of the current page. Returns structured element tree. Preferred over screenshots for performing actions (provides element `ref` values). |
| **browser_take_screenshot** | Take a screenshot as PNG or JPEG. Options: full page (`fullPage: true`), viewport only, or a specific element (via `ref`). Can save to a custom filename. |
| **browser_console_messages** | Retrieve all console messages. Filterable by level: `error`, `warning`, `info`, `debug`. Each level includes more severe levels. Can save to file. |
| **browser_network_requests** | List all network requests since page load. Option to include static resources (images, fonts, scripts). Can save to file. |

### Interaction

| Tool | Description |
|---|---|
| **browser_click** | Click an element by `ref`. Options: button (`left`, `right`, `middle`), `doubleClick`, modifier keys (`Alt`, `Control`, `Shift`, `Meta`, `ControlOrMeta`). |
| **browser_hover** | Hover over an element by `ref`. |
| **browser_type** | Type text into an editable element. Options: `slowly` (one character at a time for key handlers), `submit` (press Enter after). |
| **browser_press_key** | Press a keyboard key by name (e.g., `ArrowLeft`, `Enter`, `Tab`, `a`). |
| **browser_fill_form** | Fill multiple form fields at once. Supported field types: `textbox`, `checkbox`, `radio`, `combobox`, `slider`. Each field specified by `ref`, `name`, `type`, and `value`. |
| **browser_select_option** | Select one or multiple options in a dropdown by `ref` and `values` array. |
| **browser_drag** | Drag and drop between two elements. Requires `startRef`/`startElement` and `endRef`/`endElement`. |
| **browser_file_upload** | Upload one or multiple files by absolute path. If `paths` is omitted, the file chooser is cancelled. |

### Dialogs & JavaScript

| Tool | Description |
|---|---|
| **browser_handle_dialog** | Accept or dismiss a browser dialog (alert, confirm, prompt). For prompt dialogs, can provide `promptText`. |
| **browser_evaluate** | Evaluate a JavaScript expression on the page or a specific element. Function signature: `() => { ... }` or `(element) => { ... }` when `ref` is provided. |
| **browser_run_code** | Run a full Playwright code snippet. Receives `page` as argument: `async (page) => { ... }`. Useful for complex multi-step browser interactions. |

### Browser Management

| Tool | Description |
|---|---|
| **browser_tabs** | Manage browser tabs. Actions: `list` (show all tabs), `new` (create tab), `close` (close by index or current), `select` (switch to tab by index). |
| **browser_resize** | Resize the browser window to specified `width` and `height`. |
| **browser_wait_for** | Wait for a condition: `text` (text appears), `textGone` (text disappears), or `time` (seconds to wait). |
| **browser_install** | Install the browser specified in config. Use if you get a "browser not installed" error. |
| **browser_close** | Close the browser page. |

---

## Summary

| Source | Category | Count | Est. Context Cost |
|---|---|---|---|
| Built-in | File Operations | 5 | ~1,000–1,500 tokens |
| Built-in | Execution | 4 | ~1,200–1,800 tokens |
| Built-in | Web & Search | 2 | ~400–600 tokens |
| Built-in | Planning & Interaction | 4 | ~800–1,200 tokens |
| Built-in | Other | 2 | ~300–500 tokens |
| **Built-in subtotal** | | **17** | **~4,000–6,000 tokens** |
| MCP | Playwright Browser Automation | 22 | ~2,000–3,000 tokens |
| **Total** | | **39** | **~6,000–9,000 tokens** |

> Built-in tools are always loaded. MCP tools are loaded at session start if the MCP server is configured — not on first invocation.

For details on adding MCP tools, see [MCP Setup](mcp-setup.md). For all extension mechanisms (skills, hooks, subagents, MCP, plugins), see [Extending Claude Code](extending-claude-code.md).
