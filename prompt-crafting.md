# Claude Code: Prompt Crafting

*Last verified against [official docs](https://code.claude.com/docs/en/) on 2026-02-09*

How to write effective prompts that get the right result on the first try.

---

## Why This Matters

Claude Code is capable, but it's not a mind reader. A vague prompt leads to broad exploration, wasted context, and results that miss the mark. A well-crafted prompt gets you there in one shot.

The difference isn't about being formal or verbose — it's about giving Claude the **right information to act on**.

---

## The Three Components of a Good Prompt

### 1. What (the task)

Be specific about what you want done:

| Vague | Specific |
|---|---|
| "Fix the bug" | "Fix the 401 error on the /api/users endpoint" |
| "Make it faster" | "The product list page takes 3 seconds to load — optimize the database query in getProducts" |
| "Add a feature" | "Add a dark mode toggle to the settings page" |
| "Clean this up" | "Extract the validation logic from handleSubmit into a separate validateForm function" |

### 2. Where (the scope)

Tell Claude where to look — this saves context and avoids wrong turns:

| No scope | Scoped |
|---|---|
| "Fix the login bug" | "Fix the login bug in src/auth/controller.ts" |
| "Search for the config" | "Search for database config in the src/config/ directory" |
| "Update the tests" | "Update the tests in tests/auth/ to cover the new OAuth flow" |

### 3. How (constraints and preferences)

If you have a preference for how something should be done, say it upfront:

| Unconstrained | Constrained |
|---|---|
| "Add caching" | "Add caching using Redis — we already have a Redis client in src/lib/redis.ts" |
| "Write a test" | "Write a test using Jest — follow the pattern in tests/users.test.ts" |
| "Refactor this" | "Refactor this to use the repository pattern — don't change the public API" |

---

## Prompt Patterns That Work

### Pattern 1: Problem + Location + Expected Behavior

> "The checkout page shows the wrong total when a discount code is applied. The calculation happens in src/cart/pricing.ts. It should subtract the discount before applying tax."

**Why it works:** Claude knows the problem, where to look, and what "correct" looks like.

### Pattern 2: Do X, Don't Do Y

> "Add input validation to the registration form. Validate email format and password strength. Don't change the existing form layout or styling."

**Why it works:** Explicit boundaries prevent Claude from "improving" things you didn't ask about.

### Pattern 3: Follow the Pattern

> "Add a new API endpoint for /api/orders. Follow the same pattern as the existing /api/users endpoint in src/routes/users.ts."

**Why it works:** Claude reads the referenced pattern and replicates it. Consistent code with minimal explanation.

### Pattern 4: Step-by-Step

> "I need to add email notifications:
> 1. First, create an email service in src/services/
> 2. Then add a sendWelcomeEmail function
> 3. Call it from the registration handler
> 4. Add tests for the new service"

**Why it works:** Clear sequence prevents Claude from trying to do everything at once or making assumptions about order.

### Pattern 5: Context Dump

> "I'm working on a .NET 8 Web API. The project uses EF Core for data access and follows the repository pattern. The database is PostgreSQL. I need to add a new endpoint for retrieving user activity logs."

**Why it works:** Frontloading key context means Claude doesn't need to explore to learn your stack. Especially useful at the start of a session.

### Pattern 6: Let Claude Interview You

For complex tasks, instead of trying to specify everything upfront, let Claude ask the questions:

> "I want to add OAuth2 support. Interview me about the requirements before starting."

**Why it works:** You don't have to anticipate every detail Claude will need. Claude uses the AskUserQuestion tool to ask targeted, structured questions — which providers, which flows, where to store tokens, etc. — so the implementation matches your actual requirements. This is especially valuable for tasks where you know the goal but haven't nailed down the specifics.

### Pattern 7: Give Claude Verification Criteria

Tell Claude how to verify its own work so you don't have to check manually:

> "Add the email service and verify it works by running the tests in `tests/email.test.ts`."

**Why it works:** Claude runs the tests (or build, or linter) itself and fixes any failures before reporting back. This reduces back-and-forth and catches issues in the same turn they're introduced.

---

## Scoping Techniques

### Narrow the blast radius

The more files Claude might touch, the more important it is to scope:

| Broad (risky) | Narrow (controlled) |
|---|---|
| "Fix all the TypeScript errors" | "Fix the TypeScript errors in src/components/Header.tsx" |
| "Update the database schema" | "Add a `lastLoginAt` column to the users table migration" |
| "Refactor the auth system" | "Extract the token refresh logic from auth.ts into a separate refreshToken.ts" |

### Use "only" and "just"

> "**Only** modify the pricing module — don't touch the cart or checkout code."
> "**Just** add the new field — don't refactor the existing fields."

### Reference existing code

> "Follow the same pattern as src/services/userService.ts"
> "Use the existing DatabaseClient from src/lib/db.ts"

This prevents Claude from creating duplicate code or reinventing existing utilities.

### Pro tip: Provide rich context

- **`@` file references** — type `@` followed by a file path to include its content directly in your message
- **URLs** — give URLs for documentation and API references. Use `/permissions` to allowlist frequently-used domains.
- **Images** — paste images (`Ctrl+V`) for visual context: screenshots of errors, UI mockups, expected vs actual output
- **External editor** — press `Ctrl+G` to open your prompt in an external text editor for long, structured prompts

You can type `@` followed by a file path to include that file's content in your message. This gives Claude precise context without vague descriptions:

> "Fix the bug in @src/services/auth.ts — the JWT validation on line 45 isn't checking token expiry."

Claude receives the actual file content inline, so you skip the "read this file first" step entirely. This is especially useful when you know exactly which file is relevant.

---

## Iterating on Results

Sometimes the first result isn't right. Here's how to course-correct efficiently:

### Be specific about what's wrong

| Unhelpful | Helpful |
|---|---|
| "That's not right" | "The error handling is wrong — it should return a 400, not a 500" |
| "Try again" | "Keep the function but change it to use async/await instead of promises" |
| "Not what I wanted" | "I wanted the validation on the server side, not the client side" |

### Build on what's there

> "Good, but also add error handling for the case where the user doesn't exist."
> "The logic is right, but move it from the controller into the service layer."

### Redirect without re-explaining

> "Actually, use the approach from Pattern X instead."
> "Forget the last change — go back to the version before and instead just add input validation."

### When to start fresh

If you've corrected Claude more than twice on the same issue, run `/clear` and start a new conversation with a more specific prompt. Iterating in a polluted context often makes things worse — a clean start with better instructions is faster.

---

## Prompt Length: Finding the Sweet Spot

| Too short | Just right | Too long |
|---|---|---|
| "Fix the bug" | "Fix the null pointer in getUserById — it crashes when the user doesn't exist. Return a 404 instead." | (Three paragraphs explaining the entire user management system, the history of the bug, team dynamics, etc.) |

**Rule of thumb:** Include everything Claude needs to act. Exclude everything it doesn't. If Claude can figure it out by reading the code, you don't need to explain it.

---

## Special Situations

### Starting a new session

Frontload context at the start:
> "I'm working on the auth refactor. We're migrating from session-based auth to JWT. So far I've updated the login endpoint and the middleware. Next up is the token refresh logic in src/auth/refresh.ts."

This is more efficient than Claude having to piece together context from session memory + file reads.

### Complex multi-step tasks

Ask for a plan first:
> "Plan how to add WebSocket support. Consider our existing Express setup and the real-time requirements for the chat feature. Don't code anything yet."

### When you're not sure what you want

Be honest about it:
> "I need to improve the performance of the dashboard page. I'm not sure what's causing the slowness. Can you investigate and suggest approaches before making changes?"

### When Claude should ask you questions

> "Before implementing, ask me about any design decisions you're unsure about."

See also **Pattern 6: Let Claude Interview You** above for a more structured version of this.

---

## Common Mistakes

### 1. Over-explaining the obvious
**Bad:** "Please read the file src/auth.ts. This file contains authentication logic. I want you to look at it and understand what it does. After reading it..."
**Good:** "Check src/auth.ts — is the token expiry set correctly?"

### 2. Under-explaining the non-obvious
**Bad:** "Add the feature we discussed"
**Good:** "Add the rate limiting feature — max 100 requests per minute per IP, return 429 when exceeded"

### 3. Asking for multiple unrelated things
**Bad:** "Fix the login bug, also update the README, and run the tests, oh and check if the docker config is right"
**Good:** Ask one thing at a time, or use a structured list:
> "I need three things:
> 1. Fix the login bug (401 on /api/login)
> 2. Update the README with the new setup steps
> 3. Run the test suite and fix any failures"

### 4. Not saying when to stop
**Bad:** "Improve the code quality" (Claude might refactor your entire codebase)
**Good:** "Improve the code quality in src/utils/helpers.ts — rename unclear variables and extract the duplicated validation logic"

---

## Quick Reference

| I want to... | Prompt pattern |
|---|---|
| Fix a specific bug | Problem + location + expected behavior |
| Add a feature | What to add + where + constraints + existing patterns to follow |
| Refactor code | What to change + what NOT to change + target structure |
| Explore/understand code | Ask a specific question about a specific area |
| Get a plan before coding | "Plan how to [X]. Don't code anything yet." |
| Limit the scope | "Only modify [X]. Don't touch [Y]." |
| Follow existing patterns | "Follow the pattern in [file]" |
| Course-correct | Say what's specifically wrong + what to do differently |
| Clarify requirements first | "Interview me about the requirements before starting." |
| Self-verify the work | "Do [X] and verify by running [tests/build/linter]." |
| Give precise file context | Use `@path/to/file` inline to include file content |

---

For more prompting techniques, see the [official best practices](https://code.claude.com/docs/en/best-practices).
