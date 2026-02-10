# Claude Code: Plugins

*Last verified against [official docs](https://code.claude.com/docs/en/plugins) on 2026-02-10*

Plugins are **installable packages** that bundle skills, hooks, agents, MCP servers, and LSP servers into a single unit. One install can add multiple capabilities at once — think of them as extension packs for Claude Code.

This guide focuses on **using ready-made plugins**, especially the official Anthropic collection. For creating your own plugins, see the [official plugins documentation](https://code.claude.com/docs/en/plugins).

For a quick overview of plugins alongside skills and hooks, see [Extending Claude Code](extending-claude-code.md).

---

## Installing Your First Plugin

**Step 1:** Run `/plugin` in any Claude Code session to open the plugin browser.

**Step 2:** Browse the **Discover** tab to see available plugins from installed marketplaces.

**Step 3:** Select a plugin and choose a scope:
- **User** — available across all your projects (default)
- **Project** — shared with your team via version control
- **Local** — you only, this project only

**Step 4:** The plugin installs and is active immediately (or after restarting the session for some components).

**Step 5 (optional):** Verify it's loaded:
- For plugins with skills: ask Claude "What skills are available?"
- For plugins with MCP servers: run `/mcp` to check
- For plugins with hooks: run `/hooks` to check

You can also install from the CLI:
```bash
claude plugin install plugin-name@claude-plugin-directory
```

> **Trust warning:** Make sure you trust a plugin before installing. Anthropic does not control what MCP servers, files, or other software are included in third-party plugins. See each plugin's homepage for more information.

For the full installation walkthrough, see the [official discover-plugins guide](https://code.claude.com/docs/en/discover-plugins).

---

## The Plugin Browser

The `/plugin` command opens an interactive browser with four tabs (cycle with `Tab`/`Shift+Tab`):

| Tab | What It Shows |
|---|---|
| **Discover** | Browse available plugins from all added marketplaces |
| **Installed** | View, enable, disable, or uninstall your plugins |
| **Marketplaces** | Add, remove, update, and configure auto-updates for marketplaces |
| **Errors** | Plugin loading errors — especially useful for diagnosing LSP binary issues |

Type to filter by plugin name or description in any tab.

**Shortcut:** `/plugin market` is a shortcut for marketplace management commands.

---

## The Official Plugin Collection

Anthropic maintains a curated collection of plugins at the [claude-plugins-official](https://github.com/anthropics/claude-plugins-official) repository. These are available through the built-in marketplace — just run `/plugin` and browse the **Discover** tab. The collection includes both **internal plugins** (built by Anthropic) and **external plugins** (built by third-party partners).

For the full browsable catalog, see the [official discover-plugins page](https://code.claude.com/docs/en/discover-plugins).

### Code Intelligence (LSP)

These plugins give Claude **language server protocol (LSP)** capabilities — "go to definition", "find references", type checking, and diagnostics. They are the most impactful plugins for daily coding work. See [Code Intelligence Plugins](#code-intelligence-plugins-highlight) below for details.

| Plugin | Language |
|---|---|
| `typescript-lsp` | TypeScript / JavaScript |
| `pyright-lsp` | Python |
| `rust-analyzer-lsp` | Rust |
| `gopls-lsp` | Go |
| `jdtls-lsp` | Java |
| `kotlin-lsp` | Kotlin |
| `clangd-lsp` | C / C++ |
| `csharp-lsp` | C# |
| `php-lsp` | PHP |
| `lua-lsp` | Lua |
| `swift-lsp` | Swift |

### Development Workflows

| Plugin | What It Does |
|---|---|
| `commit-commands` | Streamlines git workflow — commit, push, and PR creation commands |
| `code-review` | Automated code review for pull requests with confidence scoring |
| `pr-review-toolkit` | Specialized PR review agents for comments, tests, error handling, and code quality |
| `feature-dev` | Comprehensive feature development workflow with codebase exploration and architecture design |
| `code-simplifier` | Simplifies and refines code for clarity, consistency, and maintainability |
| `frontend-design` | Frontend design skill for UI/UX implementation |
| `playground` | Creates interactive HTML playgrounds with visual controls and live preview |
| `hookify` | Analyzes conversation patterns and creates hooks to prevent unwanted behaviors |
| `ralph-loop` | Continuous iterative development loops for interactive refinement |
| `agent-sdk-dev` | Claude Agent SDK development tools for creating and verifying Agent SDK applications |

### External Integrations

These are built by third-party partners and connect Claude to popular services. They typically bundle pre-configured MCP servers.

**Source control:**

| Plugin | Provider | What It Does |
|---|---|---|
| `github` | GitHub | Issue/PR management, code review, repository interaction |
| `gitlab` | GitLab | Repos, merge requests, CI/CD pipelines, issues |

**Project management & communication:**

| Plugin | Provider | What It Does |
|---|---|---|
| `atlassian` | Atlassian | Jira issues and Confluence pages |
| `asana` | Asana | Task creation, search, workflow integration |
| `linear` | Linear | Issue tracking, project management, status updates |
| `notion` | Notion | Workspace pages, databases, and content |
| `slack` | Slack | Search messages, access channels, team communication |
| `circleback` | Circleback | Meeting, email, and calendar context integration |

**Infrastructure & backend:**

| Plugin | Provider | What It Does |
|---|---|---|
| `firebase` | Google | Firestore, authentication, cloud functions, hosting, storage |
| `supabase` | Supabase | Database operations, authentication, storage, real-time subscriptions |
| `vercel` | Vercel | Deployment platform — projects, deployments, domains |
| `stripe` | Stripe | Payments, webhooks, API operations |
| `pinecone` | Pinecone | Vector database management for semantic search |
| `sentry` | Sentry | Error monitoring and performance tracking |
| `posthog` | PostHog | Analytics platform — feature flags, experiments |

**Code quality & analysis:**

| Plugin | Provider | What It Does |
|---|---|---|
| `coderabbit` | CodeRabbit | Code review partner with 40+ static analyzers |
| `greptile` | Greptile | AI code review agent for GitHub and GitLab PRs |
| `serena` | Oraios | Semantic code analysis for intelligent code understanding |
| `sonatype-guide` | Sonatype | Software supply chain intelligence and dependency security |

**Design, docs & other tools:**

| Plugin | Provider | What It Does |
|---|---|---|
| `figma` | Figma | Design platform integration |
| `playwright` | Microsoft | Browser automation, web interaction, testing, screenshots |
| `context7` | Upstash | Version-specific documentation lookup from source repositories |
| `firecrawl` | Firecrawl | Web scraping, crawling, and structured data extraction |
| `huggingface-skills` | Hugging Face | Open source AI models, datasets, and spaces |
| `laravel-boost` | Laravel | Laravel toolkit — Artisan commands, Eloquent queries, code generation |
| `superpowers` | Superpowers | Brainstorming, subagent development, and TDD approaches |

### Setup & Documentation

| Plugin | What It Does |
|---|---|
| `claude-code-setup` | Analyzes your codebase and recommends tailored automations (hooks, skills, MCP servers, subagents) |
| `claude-md-management` | Tools to maintain CLAUDE.md files with quality audits and session learnings |
| `example-plugin` | Reference implementation demonstrating all Claude Code extension options |
| `plugin-dev` | Plugin development toolkit and language server |

### Output Styles

| Plugin | What It Does |
|---|---|
| `explanatory-output-style` | Adds educational insights about implementation choices during coding |
| `learning-output-style` | Interactive learning mode — requests meaningful contributions at decision points |

### Security

| Plugin | What It Does |
|---|---|
| `security-guidance` | Hook that warns about command injection, XSS, and unsafe code patterns |

---

## Code Intelligence Plugins (Highlight)

Code intelligence plugins are the most impactful category for everyday coding. They give Claude **IDE-quality navigation** through Language Server Protocol (LSP) integration. For the full reference, see the [official discover-plugins page](https://code.claude.com/docs/en/discover-plugins).

### What They Give Claude

**Automatic diagnostics:** After every file edit, the language server analyzes your changes and reports errors and warnings. Claude sees type errors, missing imports, and syntax issues — and can fix them in the same turn. Toggle diagnostic visibility with `Ctrl+O`.

**Code navigation:**

| Without LSP | With LSP |
|---|---|
| Text search (Grep/Glob) to find definitions | Precise "go to definition" — jumps to the exact source |
| Pattern matching to find references | "Find all references" — every usage, correctly resolved |
| No type awareness | Type information and diagnostics |
| May miss indirect references | Follows imports, aliases, and inheritance chains |
| No symbol overview | List all symbols in a file or workspace |

### How to Install

1. Run `/plugin` and browse to your language's LSP plugin
2. Install it (user scope is usually best)
3. Restart your session

**Important:** The LSP plugin provides the Claude Code integration, but the **language server binary** must be installed separately on your system:

| Plugin | Required binary | Install with |
|---|---|---|
| `typescript-lsp` | `typescript-language-server` | `npm install -g typescript-language-server` |
| `pyright-lsp` | `pyright-langserver` | `npm install -g pyright` or `pip install pyright` |
| `rust-analyzer-lsp` | `rust-analyzer` | `rustup component add rust-analyzer` |
| `gopls-lsp` | `gopls` | `go install golang.org/x/tools/gopls@latest` |
| `clangd-lsp` | `clangd` | Included with LLVM/Clang |
| `csharp-lsp` | `csharp-ls` | `dotnet tool install -g csharp-ls` |
| `jdtls-lsp` | `jdtls` | Download from [Eclipse JDT LS](https://github.com/eclipse-jdtls/eclipse.jdt.ls) |
| `kotlin-lsp` | `kotlin-language-server` | Download from [kotlin-language-server](https://github.com/fwcd/kotlin-language-server) |
| `php-lsp` | `intelephense` | `npm install -g intelephense` |
| `lua-lsp` | `lua-language-server` | Download from [lua-language-server](https://github.com/LuaLS/lua-language-server) |
| `swift-lsp` | `sourcekit-lsp` | Included with Swift toolchain |

If you see "Executable not found in $PATH" in the `/plugin` **Errors** tab, the binary is not installed or not in your PATH.

### When to Use Code Intelligence Plugins

- Working in typed languages (TypeScript, Rust, Go, Java, C#) — maximum benefit
- Large codebases where text search returns too many false positives
- Refactoring tasks where you need to find all usages of a symbol
- Navigating unfamiliar code — "go to definition" is much faster than searching

### Performance Notes

- `rust-analyzer` and `pyright` can consume significant memory on large projects. If you notice high memory usage, disable with `claude plugin disable <plugin-name>`.
- In monorepos, language servers may report false positive diagnostics for unresolved internal imports if the workspace isn't configured correctly. This doesn't affect Claude's editing ability.

---

## Skill Namespacing

When a plugin is installed, its skills are prefixed with the plugin name to avoid conflicts:

```
/commit-commands:commit       ← plugin skill (namespaced)
/commit                       ← standalone project skill (no namespace)
```

This means multiple plugins can have skills with the same name without conflicting. Claude resolves the namespace automatically when there's no ambiguity.

---

## Installation Scopes

When installing a plugin, choose a scope that determines who can use it and where it's stored:

| Scope | Config file | Who it affects | When to use |
|---|---|---|---|
| **User** | `~/.claude/settings.json` | You, across all projects | Personal tools you want everywhere (default) |
| **Project** | `.claude/settings.json` | Everyone on this project | Team plugins — commit this file to version control |
| **Local** | `.claude/settings.local.json` | You, this project only | Personal plugins for a specific project (gitignored) |
| **Managed** | Managed settings | Everyone in your org | Installed by admins (read-only) |

Install with a specific scope:
```bash
claude plugin install plugin-name@marketplace --scope project
```

---

## Managing Plugins

### CLI Commands

| Command | What It Does |
|---|---|
| `/plugin` | Interactive browser — discover, install, enable/disable |
| `claude plugin install <source>` | Install from CLI |
| `claude plugin uninstall <name>` | Remove a plugin (aliases: `remove`, `rm`) |
| `claude plugin enable <name>` | Re-enable a disabled plugin |
| `claude plugin disable <name>` | Disable without removing |
| `claude plugin update <name>` | Update to the latest version |

All commands accept `--scope user|project|local`.

### Auto-Updates

- Plugins from the official marketplace **auto-update by default**
- Toggle auto-updates per marketplace in `/plugin` → Marketplaces tab
- Disable all auto-updates with `DISABLE_AUTOUPDATER=true`
- To keep plugin auto-updates while disabling Claude Code updates, set both:
  `DISABLE_AUTOUPDATER=true` and `FORCE_AUTOUPDATE_PLUGINS=true`

---

## Adding Marketplaces

Beyond the built-in official marketplace, you can add third-party or organization marketplaces. For the full marketplace reference, see the [official marketplace documentation](https://code.claude.com/docs/en/plugin-marketplaces).

### Supported Sources

```bash
# GitHub repository (most common)
/plugin marketplace add owner/repo

# Git URL (GitLab, Bitbucket, self-hosted)
/plugin marketplace add https://gitlab.com/company/plugins.git

# Git URL with specific branch or tag
/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0

# SSH
/plugin marketplace add git@gitlab.com:company/plugins.git

# Local path (for development)
/plugin marketplace add ./my-marketplace

# Remote URL (direct link to marketplace.json)
/plugin marketplace add https://example.com/marketplace.json
```

### Managing Marketplaces

```bash
/plugin marketplace list      # See all added marketplaces
/plugin marketplace update    # Refresh marketplace catalog
/plugin marketplace remove    # Remove a marketplace (also uninstalls its plugins)
```

You can also manage marketplaces interactively via `/plugin` → **Marketplaces** tab.

**Team use:** Add marketplaces to `.claude/settings.json` so teammates get them automatically when they clone the project. When team members trust the repository folder, Claude Code prompts them to install the marketplace and its plugins.

---

## Creating Your Own Plugins

If you want to create plugins (rather than just use them), a plugin is a directory with components in conventional locations:

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json        # Manifest (name, version, description)
├── skills/                # Skills (SKILL.md files)
├── commands/              # Slash commands (markdown files)
├── agents/                # Custom subagents
├── hooks/
│   └── hooks.json         # Hook definitions
├── .mcp.json              # MCP server config
└── .lsp.json              # LSP server config
```

**Important:** Only `plugin.json` goes inside `.claude-plugin/`. All other directories must be at the plugin root level.

Test locally with `claude --plugin-dir /path/to/my-plugin` (load multiple with repeated flags).

**Tip:** Start with standalone config in `.claude/` for quick iteration, then convert to a plugin when you're ready to share. The [example-plugin](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/example-plugin) in the official collection is a useful reference.

For the full creation guide — manifest schema, component configuration, marketplace hosting, and distribution — see the [official plugins documentation](https://code.claude.com/docs/en/plugins). For the complete technical reference, see the [plugins reference](https://code.claude.com/docs/en/plugins-reference).

---

## Troubleshooting

| Problem | Solution |
|---|---|
| **`/plugin` command not recognized** | Check your version: `claude --version` (requires v1.0.33+). Update Claude Code if needed. |
| **Plugin not appearing after install** | Restart the session — some plugin components load at session start |
| **LSP plugin says "executable not found"** | Install the language server binary separately. Check the `/plugin` **Errors** tab for details. |
| **LSP causing high memory usage** | `rust-analyzer` and `pyright` can be memory-heavy on large projects. Disable with `claude plugin disable <name>`. |
| **False positive diagnostics in monorepo** | Language servers may report unresolved imports for internal packages. Doesn't affect Claude's editing ability. |
| **Plugin skills not appearing** | Clear the plugin cache: `rm -rf ~/.claude/plugins/cache`, restart, and reinstall. |
| **Plugin skills conflict with project skills** | Plugin skills use `plugin-name:skill-name` namespace — use the full name to disambiguate. |
| **Changes after update not taking effect** | Run `claude plugin update <name>` then restart the session. |
| **Plugin works for me but not teammates** | Check scope — user-scoped plugins are not shared. Use `--scope project`. |
| **Marketplace not loading** | Verify the URL is accessible and the repo contains `.claude-plugin/marketplace.json`. |
| **Want to debug plugin loading** | Run `claude --debug` to see verbose plugin loading details. |

For additional troubleshooting, see the [official discover-plugins guide](https://code.claude.com/docs/en/discover-plugins#troubleshooting).

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Browse and install plugins | `/plugin` |
| Install a specific plugin from CLI | `claude plugin install plugin-name@claude-plugin-directory` |
| Install a plugin for the whole team | `claude plugin install <source> --scope project` |
| Add code intelligence for my language | `/plugin` → select the LSP plugin for your language |
| See what plugins are active | `/plugin` → Installed tab |
| Check if MCP-based plugins are running | `/mcp` |
| Temporarily disable a plugin | `claude plugin disable <name>` |
| Remove a plugin | `claude plugin uninstall <name>` |
| Update all plugins | `claude plugin update <name>` (per plugin) |
| Add a team/org marketplace | `/plugin marketplace add owner/repo` |
| Test a local plugin in development | `claude --plugin-dir /path/to/plugin` |

For the complete specification — creating plugins, manifest schema, marketplace hosting, and enterprise distribution — see the [official plugins documentation](https://code.claude.com/docs/en/plugins).
