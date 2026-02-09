# Claude Code: MCP Server Configuration

How to add, configure, and manage MCP (Model Context Protocol) servers to extend Claude Code with additional tools.

---

## What Is MCP?

MCP (Model Context Protocol) is an open protocol that lets Claude Code connect to **external tool servers**. Each MCP server provides a set of tools that Claude can call, just like built-in tools.

**Example:** The Playwright MCP server adds 22 browser automation tools (navigate, click, screenshot, etc.) that Claude can use as if they were built-in.

### MCP vs Built-in Tools

| Property | Built-in Tools | MCP Tools |
|---|---|---|
| **Source** | Shipped with Claude Code | External servers you configure |
| **Always available** | Yes | Only when configured |
| **Context cost** | Always present (~4–6K tokens) | Added when the MCP server is configured |
| **Examples** | Read, Write, Edit, Bash, Grep | Playwright, custom tools, third-party integrations |

---

## How MCP Servers Load

1. **At session start**, Claude Code reads your settings for configured MCP servers
2. For each configured server, it **launches the server process** (or connects to a running one)
3. It **fetches the tool schemas** from the server (names, descriptions, parameters)
4. These schemas are **added to the system prompt** alongside built-in tool definitions
5. The tools are now available for Claude to call throughout the session

**Key point:** MCP tool definitions consume context tokens **whether or not they're used**. If you have Playwright configured but never use browser tools, those ~2–3K tokens are still spent.

---

## Configuration

MCP servers are configured in Claude Code's settings files.

### Settings File Locations

| File | Scope |
|---|---|
| `~/.claude/settings.json` | Global — applies to all projects |
| `.claude/settings.json` | Project-level — applies to this project only |

### Configuration Format

MCP servers are defined under the `mcpServers` key in settings:

```json
{
  "mcpServers": {
    "server-name": {
      "command": "command-to-launch-server",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  }
}
```

### Configuration Fields

| Field | Required | Description |
|---|---|---|
| `command` | Yes | The command to launch the MCP server (e.g., `npx`, `node`, `python`) |
| `args` | No | Array of command-line arguments |
| `env` | No | Environment variables to set for the server process |

---

## Example: Playwright MCP Server

The Playwright MCP server is one of the most common MCP integrations. It provides browser automation tools.

### Setup

```bash
# Quickest way (global)
claude mcp add --scope user playwright -- npx @playwright/mcp@latest

# Or manually in settings.json
```
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

**Prerequisites:** Node.js 18+ and browser binaries (`npx playwright install` if needed).

### How It Works Locally

```
Claude Code (your session) → sends commands
    ↓
Playwright MCP Server (local process) → controls
    ↓
Chromium browser (visible on your screen)
```

Everything runs locally — no code or data leaves your machine. The browser runs as you, so it can access localhost and your local network.

### What It Provides

22 tools for browser automation:
- Navigation (`browser_navigate`, `browser_navigate_back`)
- Page inspection (`browser_snapshot`, `browser_take_screenshot`)
- Interaction (`browser_click`, `browser_type`, `browser_fill_form`)
- JavaScript execution (`browser_evaluate`, `browser_run_code`)
- Tab and window management (`browser_tabs`, `browser_resize`)
- Network and console monitoring (`browser_network_requests`, `browser_console_messages`)

See [tools-reference.md](tools-reference.md) for the complete list.

### Usage Examples

Say "playwright" or "browser" explicitly the first time so Claude picks the MCP tools:

```
"Use playwright to open https://localhost:3000 and describe what you see"

"Use the browser to go to /login, fill in the form with test data,
 and check if validation works"

"Navigate to /dashboard, click every link in the nav, and report any broken pages"

"Browse the signup page and generate a Playwright test script that verifies
 the page loads correctly"
```

After the first use in a session, you can be more casual — Claude remembers the tools are available.

### Tool Naming

MCP tools are prefixed with `mcp__<server-name>__` in tool calls:
```
mcp__playwright__browser_navigate
mcp__playwright__browser_click
mcp__playwright__browser_snapshot
```

You don't need to use these prefixes in conversation — just say "navigate to localhost:3000" and Claude will call the right tool.

---

## Adding an MCP Server

### Option A: CLI Commands (Recommended)

The simplest way to add an MCP server:

```bash
# Add to current project
claude mcp add playwright -- npx @playwright/mcp@latest

# Add globally (all projects)
claude mcp add --scope user playwright -- npx @playwright/mcp@latest

# Verify what's configured
claude mcp list

# Remove a server
claude mcp remove playwright
claude mcp remove --scope user playwright
```

**Important:** MCP servers load at session start. After adding a server, **start a new session** for it to take effect:
```
/exit
claude
```

### Option B: Manual JSON Configuration

For more control (env vars, complex args), edit the settings file directly:

- Global: `~/.claude/settings.json`
- Project: `.claude/settings.json`

```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["my-mcp-server-package"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

After editing, restart Claude Code for changes to take effect.

### Finding MCP Servers

MCP servers are available from:
- **npm packages** — many published as `@scope/mcp-server-name`
- **Python packages** — via pip
- **Custom servers** — you can build your own following the MCP protocol spec
- **Community servers** — growing ecosystem of open-source MCP servers

---

## Multiple MCP Servers

You can configure multiple MCP servers simultaneously:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    },
    "database": {
      "command": "npx",
      "args": ["@my-org/db-mcp-server"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/mydb"
      }
    },
    "custom-tools": {
      "command": "node",
      "args": ["./tools/my-mcp-server.js"]
    }
  }
}
```

Each server's tools are prefixed with its name:
- `mcp__playwright__browser_navigate`
- `mcp__database__query`
- `mcp__custom-tools__my_tool`

### Context Cost Trade-off

Each additional MCP server adds its tool definitions to the context. With many servers, this overhead grows:

| Servers | Estimated Extra Context |
|---|---|
| 1 server (e.g., Playwright, 22 tools) | ~2–3K tokens |
| 2 servers | ~4–6K tokens |
| 3+ servers | ~6K+ tokens |

If you find yourself rarely using certain MCP tools, consider removing that server configuration to save context.

---

## Global vs Project-Level Configuration

| Use global (`~/.claude/settings.json`) when... | Use project-level (`.claude/settings.json`) when... |
|---|---|
| You want the tool available everywhere | The tool is specific to this project |
| It's a general-purpose tool (like Playwright) | It needs project-specific env vars (like a DB URL) |
| You always want it loaded | Not all projects need it |

**Recommendation:** Put Playwright in global settings (useful everywhere). Put project-specific servers (database tools, custom integrations) in project-level settings.

---

## Removing an MCP Server

```bash
# Remove from project
claude mcp remove playwright

# Remove from global
claude mcp remove --scope user playwright
```

Or manually delete the server entry from your settings file. Either way, restart Claude Code for the change to take effect.

---

## Checking MCP Status

Use the `/mcp` slash command inside a session to see:
- Which MCP servers are connected or errored
- Per-server context cost (how many tokens each server's tool definitions consume)
- Total MCP overhead on your context budget

From the CLI (outside a session), `claude mcp list` shows configured servers.

---

## MCP Tool Search (Automatic Deferral)

When MCP tool definitions exceed roughly **~10% of the total context budget**, Claude Code automatically switches to **deferred loading**. Instead of injecting all tool schemas into the system prompt at startup, it lists tools on demand — only loading the schema for a tool when it's actually needed.

This is **fully automatic** — you don't need to configure anything. It keeps context lean when you have many MCP servers or servers with large tool counts.

---

## MCP Resources

MCP servers can expose **resources** (data sources like files, database records, or API responses) in addition to tools. Reference them in conversation using:

```
@server-name:resource-name
```

This tells Claude to fetch that resource from the named MCP server and include it in context.

---

## Tip: Prefer CLI Tools Over MCP When Available

For services that already have good CLI tools — like `gh` (GitHub), `aws` (AWS), `docker`, `kubectl` — prefer using them via Bash rather than adding an MCP server:

- **Simpler** — no server process to manage
- **No context overhead** — CLI tools don't add tool definitions to your context budget
- **No configuration** — if the CLI is installed and authenticated, it just works

Reserve MCP servers for capabilities that don't have a CLI equivalent (like Playwright's browser automation).

---

## Troubleshooting

| Issue | Solution |
|---|---|
| MCP tools not appearing | Check settings file syntax (valid JSON?). Restart Claude Code. Run `/mcp` to check status. |
| "Browser not installed" error | Use `browser_install` tool, or run `npx playwright install` manually |
| Server fails to start | Check that the command/package exists. Run the command manually to see errors. |
| Tools are slow | MCP servers run as separate processes — network latency between Claude Code and the server is usually minimal, but the server's own operations (like browser automation) take real time. |
| Too much context used | Remove MCP servers you don't actively need. Run `/mcp` to see per-server context costs. |

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Add Playwright browser tools | `claude mcp add --scope user playwright -- npx @playwright/mcp@latest` |
| Add a custom MCP server | `claude mcp add <name> -- <command>` or edit settings.json |
| See what's configured | `claude mcp list` |
| Remove an MCP server | `claude mcp remove <name>` |
| Reduce context usage from MCP | Remove servers you rarely use |
| Use MCP tools in conversation | Just describe what you want — Claude picks the right tool |
| Check MCP status and context cost | `/mcp` inside a session |
| Check which MCP tools are loaded | Ask Claude "what MCP tools do you have?" at session start |
| Reference an MCP resource | `@server-name:resource-name` in conversation |
| Use project-specific config | `claude mcp add <name> -- <command>` (without `--scope user`) |
| Use global config | `claude mcp add --scope user <name> -- <command>` |

---

For the full MCP configuration reference, see the [official MCP documentation](https://code.claude.com/docs/en/mcp).
