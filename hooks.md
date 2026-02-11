# Claude Code: Hooks

*Last verified against [official docs](https://code.claude.com/docs/en/hooks) on 2026-02-11*

Hooks are **scripts that run automatically** at specific points in Claude's workflow. Unlike CLAUDE.md instructions (which are advisory — Claude can choose to ignore them), hooks are **deterministic and guaranteed** — they execute every time their trigger event fires, unconditionally.

This makes hooks the right tool for **enforcement**: linting, blocking dangerous commands, running tests, validating outputs. If it *must* happen, use a hook.

For a comparison of hooks alongside all other extension mechanisms (skills, subagents, MCP, plugins), see [Extending Claude Code](extending-claude-code.md).

---

## Adding Your First Hook

The fastest way to add a hook is the `/hooks` menu — run it in any session, select an event, and add a handler interactively. Changes take effect immediately.

You can also add hooks directly to your settings JSON. Here's a minimal example that auto-formats files with Prettier after every edit:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**What this does:** After every `Edit` or `Write` tool call, it extracts the file path from the hook's JSON input and runs Prettier on it. The `|| true` prevents a non-zero exit code from showing errors if Prettier isn't configured for that file type.

**Where to put it:** Add to `~/.claude/settings.json` (all your projects) or `.claude/settings.json` (this project, shared with team).

Here's a second example — blocking dangerous shell commands with exit code 2:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.command' | grep -qE '\\brm\\s+-rf\\b' && echo 'Blocked: rm -rf is not allowed' >&2 && exit 2 || exit 0"
          }
        ]
      }
    ]
  }
}
```

**What this does:** Before every `Bash` tool call, it checks if the command contains `rm -rf`. If it does, it prints a message to stderr and exits with code 2, which **blocks the tool call entirely**. Claude sees the error message and must find another approach.

---

## Where Hooks Are Defined

Hooks can live in 6 locations, each with a different scope:

| Location | Scope | Shareable? |
|---|---|---|
| `~/.claude/settings.json` | All your projects | No (local to your machine) |
| `.claude/settings.json` | This project | Yes (commit to git) |
| `.claude/settings.local.json` | This project, you only | No (gitignored) |
| Managed policy settings | Organization-wide | Yes (admin-controlled) |
| Plugin `hooks/hooks.json` | When plugin is enabled | Yes (bundled with plugin) |
| Skill or agent frontmatter | While component is active | Yes (defined in component file) |

### JSON Structure

Hook configuration uses 3 levels of nesting:

```json
{
  "hooks": {
    "PreToolUse": [              // Level 1: Event name
      {
        "matcher": "Bash",       // Level 2: Matcher group (regex filter)
        "hooks": [               // Level 3: Array of handlers
          {
            "type": "command",
            "command": "echo 'hook fired'"
          }
        ]
      }
    ]
  }
}
```

Multiple matcher groups can be defined per event, and multiple handlers per matcher group. All matching hooks run **in parallel**. Identical handlers are deduplicated automatically.

### The `/hooks` Menu

Type `/hooks` in any session to open the interactive hooks manager. You can view, add, and delete hooks without editing JSON files. Labels show where each hook is defined: `[User]`, `[Project]`, `[Local]`, `[Plugin]` (read-only). A toggle at the bottom disables all hooks.

Changes made via `/hooks` take effect immediately. Direct file edits mid-session trigger a security warning — you'll need to review them in `/hooks` before they activate.

---

## Hook Events

Claude Code fires hooks at 14 points in its lifecycle. Events that support **blocking** (exit code 2) can prevent the action from proceeding.

### Lifecycle Events

| Event | When It Fires | Can Block? | Matcher Values |
|---|---|---|---|
| `SessionStart` | Session begins or resumes | No | `startup`, `resume`, `clear`, `compact` |
| `PreCompact` | Before context compaction | No | `manual`, `auto` |
| `SessionEnd` | Session terminates | No | `clear`, `logout`, `prompt_input_exit`, `bypass_permissions_disabled`, `other` |

- **SessionStart** is useful for injecting context (especially after compaction), setting environment variables, and running setup scripts.
- **SessionEnd** is useful for cleanup tasks like clearing temporary files.

### Tool Events

| Event | When It Fires | Can Block? | Matcher Values |
|---|---|---|---|
| `PreToolUse` | Before a tool call executes | **Yes** | Tool name (`Bash`, `Edit`, `Write`, `Read`, `Glob`, `Grep`, `Task`, `WebFetch`, `WebSearch`, MCP tools) |
| `PermissionRequest` | When a permission dialog appears | **Yes** | Tool name (same as PreToolUse) |
| `PostToolUse` | After a tool call succeeds | No | Tool name (same as PreToolUse) |
| `PostToolUseFailure` | After a tool call fails | No | Tool name (same as PreToolUse) |

- **PreToolUse** is the most commonly used event — use it to validate, block, or modify tool inputs before execution.
- **PermissionRequest** does **not** fire in headless/non-interactive mode (`claude -p`). Use PreToolUse instead for programmatic use.
- **PostToolUse** runs after the tool succeeds — use it for formatting, logging, or follow-up actions.

### Response Events

| Event | When It Fires | Can Block? | Matcher Values |
|---|---|---|---|
| `Stop` | Claude finishes responding | **Yes** | None (always fires) |
| `SubagentStart` | A subagent is spawned | No | Agent type (`Bash`, `Explore`, `Plan`, or custom agent names) |
| `SubagentStop` | A subagent finishes | **Yes** | Agent type (same as SubagentStart) |

- **Stop** is the second most commonly used event. Use it to run tests, code review, or validation after Claude finishes. Blocking a Stop hook (exit code 2) makes Claude continue working — it sees your stderr as feedback.
- **Infinite loop prevention:** The `stop_hook_active` field in the input JSON is `true` when Claude is already continuing due to a previous Stop hook. Check this and exit 0 early to avoid infinite loops.
- Stop hooks do **not** fire on user interrupts.

### Prompt & Task Events

| Event | When It Fires | Can Block? | Matcher Values |
|---|---|---|---|
| `UserPromptSubmit` | User submits a prompt | **Yes** | None (always fires) |
| `Notification` | Claude sends a notification | No | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog` |
| `TeammateIdle` | Agent team teammate about to idle | **Yes** | None (always fires) |
| `TaskCompleted` | A task is marked completed | **Yes** | None (always fires) |

- **UserPromptSubmit** can modify or block user prompts before Claude processes them.
- **TeammateIdle** and **TaskCompleted** are for [Agent Teams](https://code.claude.com/docs/en/agent-teams) (experimental).

---

## Hook Types

### Command Hooks (`type: "command"`)

The most common type. Runs a shell command that receives JSON input on stdin and communicates via exit codes and stdout/stderr.

| Field | Required? | Default | Description |
|---|---|---|---|
| `type` | Yes | — | Must be `"command"` |
| `command` | Yes | — | Shell command to run |
| `timeout` | No | 600s | Max execution time in seconds |
| `statusMessage` | No | — | Message shown in UI while hook runs |
| `once` | No | `false` | Run once per session then remove (skill hooks only) |
| `async` | No | `false` | Run in background — cannot block or return decisions |

**Exit codes:** 0 = success (proceed), 2 = block the action, any other = non-blocking error.

Async hooks (`"async": true`) run in the background. Their output is delivered on the next conversation turn. They cannot block actions or return decisions — use them for long-running tasks like test suites.

### Prompt Hooks (`type: "prompt"`)

Sends a prompt to a Claude model for single-turn evaluation. The model returns a JSON verdict: `{ "ok": true/false, "reason": "..." }`.

| Field | Required? | Default | Description |
|---|---|---|---|
| `type` | Yes | — | Must be `"prompt"` |
| `prompt` | Yes | — | Prompt text (use `$ARGUMENTS` for hook input JSON) |
| `model` | No | Haiku | Which model to use |
| `timeout` | No | 30s | Max execution time in seconds |
| `statusMessage` | No | — | Message shown in UI while hook runs |
| `once` | No | `false` | Run once per session then remove (skill hooks only) |

When `ok` is `false`, the hook blocks the action and feeds the `reason` back to Claude. Use prompt hooks for nuanced evaluation that's hard to express in a shell script — like checking whether generated code meets quality criteria.

**Supported events:** PreToolUse, PostToolUse, PostToolUseFailure, PermissionRequest, UserPromptSubmit, Stop, SubagentStop, TaskCompleted. Not supported: TeammateIdle.

### Agent Hooks (`type: "agent"`)

Spawns a multi-turn subagent with access to Read, Grep, and Glob tools. Up to 50 turns. Returns the same `{ "ok": true/false, "reason": "..." }` format as prompt hooks.

| Field | Required? | Default | Description |
|---|---|---|---|
| `type` | Yes | — | Must be `"agent"` |
| `prompt` | Yes | — | Prompt text (use `$ARGUMENTS` for hook input JSON) |
| `model` | No | Haiku | Which model to use |
| `timeout` | No | 60s | Max execution time in seconds |
| `statusMessage` | No | — | Message shown in UI while hook runs |
| `once` | No | `false` | Run once per session then remove (skill hooks only) |

Use agent hooks when evaluation requires reading files or searching code — things a simple prompt can't do. Example: verifying that unit tests exist for new functions before allowing Claude to stop.

Same supported events as prompt hooks.

---

## Matcher Syntax

Matchers are **regex strings** that filter when a hook fires. They are **case-sensitive**.

| Pattern | Matches |
|---|---|
| `"Bash"` | Bash tool only |
| `"Edit\|Write"` | Edit or Write tool |
| `"mcp__.*"` | Any MCP tool |
| `"mcp__github__.*"` | All tools from the GitHub MCP server |
| `"mcp__.*__write.*"` | Write tools from any MCP server |
| `"Notebook.*"` | NotebookEdit, NotebookRead, etc. |

Omit the `matcher` field (or use `"*"` or `""`) to match all occurrences of the event.

**MCP tool naming pattern:** `mcp__<server>__<tool>` — e.g., `mcp__memory__create_entities`, `mcp__filesystem__read_file`.

For lifecycle events, matchers filter by the event's context value: SessionStart matches on `source` (`startup`, `resume`, `clear`, `compact`), PreCompact matches on `trigger` (`manual`, `auto`), SessionEnd matches on `reason`, and Notification matches on `notification_type`.

---

## Input & Output

### What Hooks Receive (stdin JSON)

Every hook receives a JSON object on stdin with these common fields:

| Field | Description |
|---|---|
| `session_id` | Current session identifier |
| `transcript_path` | Path to conversation transcript JSON |
| `cwd` | Current working directory |
| `permission_mode` | Current mode: `"default"`, `"plan"`, `"acceptEdits"`, `"dontAsk"`, or `"bypassPermissions"` |
| `hook_event_name` | Name of the event that fired |

**Event-specific fields** (most commonly needed):

| Event | Key Fields |
|---|---|
| `PreToolUse` | `tool_name`, `tool_input` (tool-specific object), `tool_use_id` |
| `PostToolUse` | `tool_name`, `tool_input`, `tool_response` (result from tool), `tool_use_id` |
| `UserPromptSubmit` | `prompt` (the user's text) |
| `SessionStart` | `source` (`startup`/`resume`/`clear`/`compact`), `model` |
| `Stop` | `stop_hook_active` (boolean — true when already continuing from a Stop hook) |
| `SubagentStop` | `agent_id`, `agent_type`, `agent_transcript_path` |
| `Notification` | `message`, `notification_type` |

**Extracting fields:** Use `jq` to pull values from stdin. For example, `jq -r '.tool_input.command'` extracts the command from a Bash PreToolUse hook, and `jq -r '.tool_input.file_path'` extracts the file path from an Edit/Write hook.

For the complete input schemas for every event and every tool, see the [official hooks documentation](https://code.claude.com/docs/en/hooks).

### What Hooks Return (stdout JSON)

**Exit codes** determine the primary outcome:

| Exit Code | Meaning | Behavior |
|---|---|---|
| **0** | Success | Action proceeds. stdout JSON is parsed for additional instructions. |
| **2** | Block | Action is blocked. stderr is fed to Claude as error feedback. stdout is ignored. |
| **Other** | Non-blocking error | stderr shown in verbose mode (`Ctrl+O`). Execution continues. |

**Important:** JSON output is only processed on exit code 0. If you exit 2, any JSON in stdout is ignored — only stderr matters.

**stdout visibility:** For `SessionStart` and `UserPromptSubmit`, stdout text is added as context visible to Claude. For all other events, stdout is only visible in verbose mode (`Ctrl+O`).

On exit 0, you can return a JSON object with these universal fields:

| Field | Default | Description |
|---|---|---|
| `continue` | `true` | If `false`, Claude stops processing entirely |
| `stopReason` | — | Message shown to user when `continue` is `false` (not shown to Claude) |
| `suppressOutput` | `false` | If `true`, hides stdout from verbose mode |
| `systemMessage` | — | Warning message shown to user |

**Decision patterns** vary by event. The most common:

- **Blocking events** (PreToolUse, UserPromptSubmit, Stop, SubagentStop): Return `{ "decision": "block", "reason": "..." }` or use exit code 2.
- **PreToolUse** also supports: `permissionDecision` (`"allow"`, `"deny"`, `"ask"`), `updatedInput` (modify tool inputs before execution), and `additionalContext` (inject context for Claude). These go inside a `hookSpecificOutput` object — see the [official docs](https://code.claude.com/docs/en/hooks) for the full schema.

---

## Practical Examples

### 1. Auto-format after edits

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit|Write",
      "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write 2>/dev/null || true" }]
    }]
  }
}
```

### 2. Block edits to protected files

```bash
#!/bin/bash
# protect-files.sh — exit 2 to block, 0 to allow
FILE=$(jq -r '.tool_input.file_path')
case "$FILE" in
  *.lock|*package-lock.json|*.env|*.secret*)
    echo "Protected file: $FILE" >&2
    exit 2
    ;;
esac
exit 0
```

```json
{ "PreToolUse": [{ "matcher": "Edit|Write", "hooks": [{ "type": "command", "command": "./scripts/protect-files.sh" }] }] }
```

### 3. Block dangerous shell commands

```bash
#!/bin/bash
# block-dangerous.sh
CMD=$(jq -r '.tool_input.command')
if echo "$CMD" | grep -qE '\brm\s+-rf\b|DROP\s+TABLE|--force\s+push'; then
  echo "Blocked dangerous command: $CMD" >&2
  exit 2
fi
exit 0
```

### 4. Auto-run tests when Claude finishes

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "jq -r '.stop_hook_active' | grep -q 'true' && exit 0; npm test 2>&1 | tail -20; if [ ${PIPESTATUS[0]} -ne 0 ]; then echo 'Tests failed — please fix' >&2; exit 2; fi"
      }]
    }]
  }
}
```

Note the `stop_hook_active` check at the start — this prevents an infinite loop if the Stop hook fires again after Claude tries to fix failing tests.

### 5. Re-inject context after compaction

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "compact",
      "hooks": [{
        "type": "command",
        "command": "echo 'Reminder: Always run tests before committing. Use ESLint for JS files. See CONVENTIONS.md for naming rules.'"
      }]
    }]
  }
}
```

For `SessionStart`, stdout is added as context visible to Claude — so this text gets injected into the conversation after every compaction.

### 6. Desktop notifications

```json
{
  "hooks": {
    "Notification": [{
      "matcher": "idle_prompt",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.message' | xargs -I {} powershell -Command \"[System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms'); [System.Windows.Forms.MessageBox]::Show('{}')\""
      }]
    }]
  }
}
```

macOS: use `osascript -e 'display notification \"$MSG\"'`. Linux: use `notify-send "$MSG"`.

### 7. Async test runner

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "./scripts/run-tests-async.sh",
        "async": true,
        "statusMessage": "Running tests in background..."
      }]
    }]
  }
}
```

Async hooks run in the background and deliver output on the next conversation turn. They cannot block actions.

### 8. Prompt-based quality gate

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "prompt",
        "prompt": "Review the conversation in $ARGUMENTS. Check: 1) Were all requested changes made? 2) Are there obvious bugs? 3) Were tests updated if needed? Return {\"ok\": false, \"reason\": \"...\"} if any check fails.",
        "statusMessage": "Running quality check..."
      }]
    }]
  }
}
```

Prompt hooks use an LLM (default: Haiku) for evaluation. The `$ARGUMENTS` placeholder is replaced with the hook's input JSON.

---

## Hooks in Skills and Agents

Skills and subagents can define hooks in their YAML frontmatter. These hooks are **scoped to the component's lifecycle** — they activate when the skill/agent loads and are cleaned up when it finishes.

```yaml
---
name: secure-deploy
description: Deploy with security validation
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/security-check.sh"
---
```

**Key differences from settings hooks:**
- **Skill hooks** support the `once: true` field — the hook runs once per session then auto-removes.
- **Agent hooks** do not support `once`.
- In subagents, `Stop` hooks are automatically converted to `SubagentStop` (since the subagent doesn't have its own Stop event).

For details on skill frontmatter, see [Skills Deep Dive](skills.md). For agent frontmatter, see [Custom Subagents](custom-agents.md).

---

## Environment Variables

Hooks have access to these environment variables:

| Variable | Available In | Description |
|---|---|---|
| `$CLAUDE_PROJECT_DIR` | All hooks | The project root directory |
| `${CLAUDE_PLUGIN_ROOT}` | Plugin hooks | The plugin's root directory |
| `$CLAUDE_CODE_REMOTE` | All hooks | Set to `"true"` in remote web environments; not set locally |
| `$CLAUDE_ENV_FILE` | SessionStart hooks only | Path to a file where you can write `export` statements to set env vars for subsequent Bash commands |

**Persisting environment variables:** In a SessionStart hook, write `export VAR=value` lines to the file at `$CLAUDE_ENV_FILE`. These variables will be available in all subsequent Bash tool calls during the session.

---

## Security & Debugging

### Security

Hooks run with **your full system user permissions**. They can read, modify, and delete any files your user account can access. Always review hook commands before adding them.

**Best practices:**
- Validate and sanitize inputs — don't trust `tool_input` blindly
- Always quote shell variables (`"$VAR"` not `$VAR`)
- Use absolute paths or `$CLAUDE_PROJECT_DIR` for project-relative paths
- Skip sensitive files (`.env`, `.git/`, keys, credentials)
- Block path traversal (check for `..` in file paths)

**Hook snapshot:** Claude Code captures all hooks at session startup and uses that snapshot throughout the session. If you edit hook files directly mid-session, the changes won't take effect until you review them in the `/hooks` menu or restart.

### Enterprise Controls

- `allowManagedHooksOnly` — blocks user, project, and plugin hooks (only managed and SDK hooks load)
- `disableAllHooks: true` — disables all hooks and custom status lines

### Debugging

- **`claude --debug`** — shows full hook execution details: which hooks matched, exit codes, stdout output
- **`Ctrl+O`** — toggles verbose mode in the transcript to see hook output
- **`/hooks`** — view all active hooks and their sources
- **Manual testing:** Pipe JSON to your hook script to test it outside Claude Code:
  ```bash
  echo '{"tool_input":{"command":"rm -rf /"}}' | ./scripts/block-dangerous.sh
  echo $?  # Should output 2
  ```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| **Hook not firing** | Check `/hooks` to confirm it's loaded. Verify the matcher is case-sensitive and matches the exact tool name. Verify you're using the right event type. |
| **Hook fires but errors** | Test manually with piped JSON (see Debugging above). Use absolute paths. Ensure scripts are executable (`chmod +x`). Install `jq` if using it. |
| **`/hooks` shows no hooks** | Restart the session. Validate JSON syntax (no trailing commas, no comments). Check the file is in the right location. |
| **Stop hook runs forever** | Check the `stop_hook_active` field in input and exit 0 early when it's `true`. |
| **JSON validation failed** | Shell profile `echo` statements can pollute stdout. Guard them with `if [[ $- == *i* ]]` (interactive shell check). stdout must contain only the JSON object. |
| **PermissionRequest not firing** | PermissionRequest doesn't fire in headless mode (`claude -p`). Use PreToolUse instead. |
| **stdout not being processed** | JSON output is only parsed on exit code 0. If you exit 2, stdout is ignored — only stderr matters. |
| **Mid-session changes not working** | Hooks are snapshot at startup. Review changes in `/hooks` menu or restart the session. |

---

## Quick Reference

| I want to... | Do this |
|---|---|
| Run a script after every file edit | `PostToolUse` hook with `Edit\|Write` matcher |
| Block Claude from modifying certain files | `PreToolUse` hook with `Edit\|Write` matcher, exit code 2 |
| Run tests when Claude finishes | `Stop` event hook (check `stop_hook_active` to avoid loops) |
| Validate commands before execution | `PreToolUse` hook with `Bash` matcher |
| Add a hook interactively | `/hooks` menu |
| Add a hook for the whole team | `.claude/settings.json` (commit to git) |
| Add a hook scoped to a skill | `hooks:` in skill YAML frontmatter |
| Use AI to evaluate a condition | `prompt` type hook (default model: Haiku) |
| Use AI with file access to evaluate | `agent` type hook (has Read, Grep, Glob tools) |
| Run a hook in the background | `"async": true` (command type only) |
| Re-inject context after compaction | `SessionStart` hook with `compact` matcher |
| Persist env vars for the session | SessionStart hook writing to `$CLAUDE_ENV_FILE` |
| Debug why a hook isn't working | `claude --debug` or `Ctrl+O` for verbose mode |
| See all active hooks | `/hooks` |

For the complete specification — all input/output schemas, every field for every event, and the full example collection — see the [official hooks documentation](https://code.claude.com/docs/en/hooks).
