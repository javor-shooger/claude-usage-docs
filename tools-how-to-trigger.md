# How to Trigger Claude Code Tools

A practical guide on what to say to make Claude use specific tools — both implicitly (natural language) and explicitly (direct references).

---

## Implicit Triggers (Natural Language)

These are phrases and patterns that naturally cause each tool to be used without naming it directly.

### File Operations

| Tool | Implicit Triggers (just say...) |
|---|---|
| **Read** | "show me", "what's in", "open", "look at", "check the file", "read", "what does X look like" |
| **Write** | "create a file", "make a new file", "write a file called", "save this as" |
| **Edit** | "change X to Y", "replace", "update the line", "rename", "fix the typo", "modify" |
| **Glob** | "find files named", "where are the", "list all *.cs files", "what files match" |
| **Grep** | "search for", "find where X is used", "grep for", "which files contain", "look for references to" |

### Execution

| Tool | Implicit Triggers |
|---|---|
| **Bash** | "run", "execute", "build", "git status", "npm install", "compile", "msbuild", any shell command |
| **Task (Explore)** | "explore the codebase", "how does X work across the project", "find how X is implemented" |
| **Task (Plan)** | "plan how to", "design an approach for", "what's the best way to implement" |
| **Task (general-purpose)** | "research", "investigate", "figure out" (for broad multi-step questions) |

### Web

| Tool | Implicit Triggers |
|---|---|
| **WebSearch** | "search the web for", "google", "look up", "what's the latest on", "find online" |
| **WebFetch** | "fetch this URL", "what's on this page", "read this link", "check this website" |

### Planning & Interaction

| Tool | Implicit Triggers |
|---|---|
| **TodoWrite** | "track these tasks", "make a checklist", "let's plan the steps" (also used automatically for complex multi-step work) |
| **EnterPlanMode** | "let's plan this first", "design the approach before coding", "I want to review the plan first" |
| **AskUserQuestion** | Used automatically when clarification is needed. You can say: "ask me before deciding" |

### Browser (Playwright)

| Tool | Implicit Triggers |
|---|---|
| **browser_navigate** | "go to [URL]", "open [URL] in the browser", "visit this page" |
| **browser_snapshot** | "what's on the page", "show me the page structure", "what elements are on the page" |
| **browser_take_screenshot** | "take a screenshot", "capture the page", "show me what it looks like" |
| **browser_click** | "click [element]", "press the button", "select that link" |
| **browser_type** | "type [text] into [field]", "enter [text] in the input" |
| **browser_fill_form** | "fill out the form", "complete the fields" |
| **browser_evaluate** | "run this JavaScript on the page", "execute JS in the browser" |
| **browser_console_messages** | "check the console", "any console errors?", "show me browser logs" |
| **browser_network_requests** | "check network requests", "what API calls were made", "show network traffic" |

---

## Explicit Triggers (Direct References)

When you want to guarantee a specific tool is used, reference it by name or use these patterns:

| What to Say | Tool Used |
|---|---|
| "use **Read** to look at X" | Read |
| "use **Grep** to search for X" | Grep |
| "use **Glob** to find X" | Glob |
| "use **Edit** to change X" | Edit |
| "**run in bash**: `command`" | Bash |
| "use a **sub-agent** to explore" | Task (Explore) |
| "**search the web** for X" | WebSearch |
| "**fetch** this URL" | WebFetch |
| "**navigate** the browser to X" | browser_navigate |
| "take a **screenshot**" | browser_take_screenshot |
| "get a **snapshot** of the page" | browser_snapshot |
| "check **console** messages" | browser_console_messages |
| "show **network** requests" | browser_network_requests |
| "**run Playwright code**: ..." | browser_run_code |

---

## Pro Tips

### 1. Force Parallel Execution
Say: **"do these in parallel"** or **"run these at the same time"**
- Example: "In parallel, search for all .cshtml files and grep for 'BusinessLocation'"

### 2. Force Background Execution
Say: **"run this in the background"**
- The tool runs asynchronously and you can continue working

### 3. Force a Sub-Agent
Say: **"use an agent to..."** or **"spawn an agent for..."**
- Example: "Use an explore agent to find how caching works in this project"

### 4. Combine Browser Tools
Say things like: "Open localhost:3000, take a screenshot, then check for console errors"
- This chains: browser_navigate → browser_take_screenshot → browser_console_messages

### 5. Be Specific About Search Scope
- "Search **file names** for X" → Glob
- "Search **file contents** for X" → Grep
- "Search **the web** for X" → WebSearch

### 6. Control Edit Behavior
- "Change **every** X to Y" → Edit with `replace_all: true`
- "Change X to Y on **line 42**" → Edit with surrounding context to target the exact location

### 7. Request a Plan First
Say: **"plan this before coding"** or **"enter plan mode"**
- Forces a review step before any code is written

### 8. Skill Invocation
Use slash commands: **"/commit"**, **"/review-pr"**, etc.
- These invoke registered skills via the Skill tool

---

## Quick Reference Card

| I want to... | Say... |
|---|---|
| See a file | "show me [path]" |
| Find a file | "find files matching [pattern]" |
| Search code | "search for [term] in the codebase" |
| Edit code | "change [old] to [new] in [file]" |
| Run a command | "run [command]" |
| Search the internet | "search the web for [query]" |
| Open a webpage | "go to [URL] in the browser" |
| Screenshot a page | "take a screenshot" |
| Check browser console | "any console errors?" |
| Track tasks | "track these as a todo list" |
| Plan before coding | "plan this first" |
| Run things in parallel | "do X and Y in parallel" |
| Use a sub-agent | "use an explore agent to find..." |
