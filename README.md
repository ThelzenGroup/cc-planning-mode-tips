![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)

# cc-planning-mode-tips

> 53 tips covering the full planning mode workflow — from writing a prompt that gets a useful plan, to guiding exploration, to approving and iterating. Concrete before/after examples and explicit anti-patterns throughout.

Planning mode is Claude Code's workflow for exploring your codebase in read-only mode, writing a plan, and requesting approval before making changes. This repo covers the full workflow from first prompt to plan approval, organized by the phases you'll actually experience. It's built for intermediate Claude Code users who want to move beyond the basics.

*New to planning mode? Start with [Before You Start](#before-you-start). Already using it but getting vague plans? Jump to [Writing a Great Plan Prompt](#writing-a-great-plan-prompt). Getting surprising results mid-implementation? See [Anti-Patterns & Traps](#anti-patterns--traps). Unfamiliar with planning-mode terminology? See the [Glossary](#glossary) at the bottom.*

## Contents

| Category | Tips | What you'll learn |
|---|---|---|
| [Before You Start](#before-you-start) | 6 | When to use plan mode, what it does under the hood, how to set expectations |
| [Writing a Great Plan Prompt](#writing-a-great-plan-prompt) | 8 | How to write prompts that produce plans you'll actually approve |
| [Guiding Exploration](#guiding-exploration) | 7 | How to steer Claude's read-only exploration phase productively |
| [Shaping the Design](#shaping-the-design) | 8 | How to influence the architecture and structure of the plan Claude produces |
| [Evaluating & Iterating on the Plan](#evaluating--iterating-on-the-plan) | 7 | How to read a plan critically, push back, and iterate without starting over |
| [Workflow & Collaboration](#workflow--collaboration) | 7 | How to integrate plan mode into your day-to-day and team workflows |
| [Anti-Patterns & Traps](#anti-patterns--traps) | 10 | The most common ways intermediate users misuse or underuse plan mode |

---

## Before You Start

### Understand what planning mode actually does

**Why & when it matters:**
Planning mode is a read-only exploration phase followed by a written plan — Claude can't edit files until you approve, which prevents misaligned implementations.

**Example:**
You ask Claude to "add error handling to the auth module." Without planning mode, Claude might immediately start writing code based on assumptions about your error handling strategy. With planning mode, Claude first explores your existing error patterns, identifies where auth errors occur, proposes a specific approach, and waits for your approval before touching any code.

**How to apply it:**
Invoke planning mode with `/plan` before starting any task that touches multiple files or requires architectural decisions. Claude will spawn Explore agents to read your codebase, write a plan to a markdown file, then call `ExitPlanMode` to request your approval. You can iterate on the plan before any code is written.

**Pitfalls:**
Don't assume planning mode is automatic — you have to explicitly invoke it. Don't skip reading the plan file before approving — that defeats the purpose of the read-only phase.

---

### Use plan mode when touching more than 3 files

**Why & when it matters:**
Multi-file changes have hidden dependencies — planning mode surfaces them before you're halfway through implementation.

**Example:**
You need to add a new API endpoint. Without planning, you might update the route handler and forget to update the OpenAPI spec, the client SDK, and the integration tests. With planning mode, Claude identifies all four files upfront and sequences the changes correctly.

**How to apply it:**
If your mental model of the task includes "and then I need to update..." more than twice, use planning mode. The exploration phase will catch the files you forgot and the ones you didn't know existed.

**Pitfalls:**
Don't use this as a hard rule — a 5-file refactor that's purely mechanical (renaming a function) doesn't need planning. Use it when the changes have semantic dependencies, not just when the file count is high.

---

### Planning mode is a collaboration, not an oracle

**Why & when it matters:**
The plan is a starting point for discussion — you're expected to push back, redirect, and iterate before approving.

**Example:**
Claude proposes adding a new database table for a feature. You read the plan and realize an existing table could be extended instead. You reject the plan, explain the existing table, and ask Claude to revise. The second plan reuses the existing schema and is much simpler. This back-and-forth is the intended workflow.

**How to apply it:**
Treat `ExitPlanMode` as "here's my draft, what do you think?" not "this is final, approve or reject." Use `AskUserQuestion` responses to steer Claude toward your actual intent. If the plan feels off, say why and ask for a revision.

**Pitfalls:**
Don't approve a plan just because it's technically correct — approve it because it's the approach you actually want. Don't treat iteration as failure — it's cheaper to iterate on a plan than on half-written code.

---

### Know when planning mode is overkill

**Why & when it matters:**
Planning mode adds latency — for trivial tasks, the overhead outweighs the benefit.

**Example:**
Fixing a typo in a comment, updating a dependency version in package.json, or renaming a single variable across 2 files — these don't need planning. Just prompt Claude directly: "fix the typo in README.md line 47" or "rename `userId` to `accountId` in auth.ts."

**How to apply it:**
Skip planning mode for: single-file changes, typo fixes, dependency updates, simple renames, adding a console.log, or any task where you already know exactly what needs to change and there's no architectural decision.

**Pitfalls:**
Don't use "I know what I want" as a reason to skip planning on complex tasks. If the task involves any uncertainty about how existing code works or what files are affected, use planning mode.

---

### Understand the cost of skipping planning

**Why & when it matters:**
Skipping planning on complex tasks leads to misaligned implementations that waste tokens and require rewrites.

**Example:**
You ask Claude to "add caching to the API" without planning. Claude implements an in-memory cache. You realize halfway through that you needed Redis because the app runs on multiple servers. Now you're rewriting the entire implementation. With planning mode, Claude would have asked about your deployment model during exploration and proposed Redis upfront.

**How to apply it:**
When you're tempted to skip planning to save time, ask: "If Claude makes a wrong assumption here, how expensive is it to undo?" If the answer is "I'd have to rewrite significant code," use planning mode.

**Pitfalls:**
Don't use planning mode as a crutch for underspecified prompts. "Add caching" is vague whether you plan or not — give Claude the constraints upfront (Redis, TTL requirements, cache key strategy) so the plan can be specific.

---

### Set the right expectations: planning slows you down to speed you up

**Why & when it matters:**
Planning mode adds 2-5 minutes upfront but saves 20-60 minutes of implementation churn on complex tasks.

**Example:**
Without planning: "Add user roles" → Claude writes code → you realize it conflicts with existing permissions logic → 30 minutes of back-and-forth fixes. With planning: "Add user roles" → Claude explores existing permissions → proposes integration approach → you approve → implementation is clean on first try.

**How to apply it:**
Treat the planning phase as an investment, not overhead. If you're impatient, remind yourself that the time spent reading and iterating on a plan is time you're NOT spending debugging a misaligned implementation.

**Pitfalls:**
Don't rush through plan approval to "get to the real work" — the plan IS the real work. A bad plan approved quickly produces bad code slowly.

---

## Writing a Great Plan Prompt

### Give Claude the "why" behind the task

**Why & when it matters:**
Without intent, Claude optimizes for your literal request instead of your actual goal.

**Example:**
```
❌ "Add error handling to the auth module"
✅ "Add error handling to src/auth/login.ts:handleOAuthCallback — specifically for 
   the case where the token exchange fails silently. Don't change the function 
   signature. The goal is to surface these failures as structured errors that 
   upstream callers can catch."
```

**How to apply it:**
Before invoking plan mode, write one sentence explaining why you're making this change. What problem does it solve? What breaks if you don't do it? Include that sentence in your plan prompt. Claude will use it to evaluate tradeoffs and propose solutions that match your actual intent.

**Pitfalls:**
Don't confuse "why" with implementation details. "Why: because we need to use Redis" is a constraint, not a reason. "Why: the app runs on multiple servers so in-memory cache doesn't work" is the actual reason.

---

### Name the exact files and functions involved

**Why & when it matters:**
Vague scope forces Claude to guess — naming files upfront focuses exploration and produces specific plans.

**Example:**
```
❌ "Refactor the authentication logic"
✅ "Refactor src/auth/middleware.ts:validateSession and src/auth/tokens.ts:refreshToken 
   to share the token validation logic that's currently duplicated between them"
```

**How to apply it:**
If you know which files or functions need to change, name them in the prompt. Use `file:function` format. If you don't know the exact names, give Claude a starting point: "the auth middleware, probably in src/auth/" is better than "the auth code."

**Pitfalls:**
Don't over-specify if you're genuinely uncertain — "I think it's in auth.ts but I'm not sure" is fine. Claude will explore and correct. But if you DO know, say it.

---

### Specify what you DON'T want

**Why & when it matters:**
Negative constraints prevent Claude from proposing solutions you'll reject.

**Example:**
```
❌ "Add caching to the API"
✅ "Add caching to the API. Don't add new dependencies — use the existing Redis 
   client in src/cache.ts. Don't cache authenticated endpoints. Don't change 
   the response format."
```

**How to apply it:**
Think about what would make you reject a plan. Common negative constraints: no new dependencies, don't change the API contract, don't touch the database schema, keep the existing test structure, don't refactor unrelated code. State these upfront.

**Pitfalls:**
Don't over-constrain to the point where only one solution exists — that defeats the purpose of planning. Negative constraints should rule out bad approaches, not prescribe the exact implementation.

---

### Describe the desired end state, not the steps

**Why & when it matters:**
Prescribing steps locks Claude into your approach — describing outcomes lets Claude find better paths.

**Example:**
```
❌ "First add a cache layer, then update the API handlers to check the cache, 
   then add cache invalidation on writes"
✅ "API responses should be cached with a 5-minute TTL. Cache should invalidate 
   when the underlying data changes. Authenticated requests should not be cached."
```

**How to apply it:**
Write your prompt as acceptance criteria, not implementation steps. "When I'm done, X should be true" not "First do A, then B, then C." Let Claude figure out the sequence.

**Pitfalls:**
If you have a strong reason to prefer a specific approach, state it as a constraint ("prefer X over Y because...") not as steps. Claude can still propose alternatives if your preference has downsides.

---

### Include known constraints upfront

**Why & when it matters:**
Constraints discovered mid-planning force rewrites — stating them upfront produces viable plans on the first try.

**Example:**
```
❌ "Add user roles to the app"
✅ "Add user roles to the app. Constraints: we're on Postgres 12 (no JSONB 
   operators from 13+), the User table is in a shared schema we can't modify, 
   and the mobile app expects the existing /me endpoint format unchanged."
```

**How to apply it:**
Before invoking plan mode, list: database/framework versions, API contracts that can't change, deployment constraints (serverless, multi-region, etc.), performance requirements, and any "we tried X before and it didn't work" history.

**Pitfalls:**
Don't list constraints you're not sure about. "I think we're on Postgres 12" is worse than checking first. Incorrect constraints produce plans that won't work.

---

### Use examples from the existing codebase

**Why & when it matters:**
Pointing to existing patterns anchors Claude's plan to your actual conventions, not generic best practices.

**Example:**
```
❌ "Add a new API endpoint for user settings"
✅ "Add a new API endpoint for user settings. Follow the same pattern as 
   src/api/profile.ts:getProfile — same auth middleware, same response 
   envelope, same error handling."
```

**How to apply it:**
If a similar feature already exists, reference it by file and function. "Do it like X does Y" is one of the most effective ways to get a plan that matches your codebase style. Claude will read the example and replicate its patterns.

**Pitfalls:**
Don't reference examples that are themselves problematic. If the existing code has known issues, say so: "Follow the pattern in profile.ts but fix the error handling — it swallows errors silently."

---

### Know when to split one big request into multiple planning sessions

**Why & when it matters:**
Plans that try to do too much produce vague, unactionable output — splitting produces specific, executable plans.

**Example:**
A request like "Migrate the app from REST to GraphQL" is too large for one planning session. Split it: Session 1 plans the GraphQL schema design. Session 2 plans the resolver implementation for one domain. Session 3 plans the client migration strategy. Each produces a specific, reviewable plan.

**How to apply it:**
If your task has multiple independent decisions (schema design AND implementation AND migration), plan them separately. If your task touches more than 10 files, consider splitting by domain or layer. Use the first planning session to create a roadmap, then plan each piece.

**Pitfalls:**
Don't split tasks that are tightly coupled. "Add a feature" and "write tests for that feature" should be one planning session, not two — the test strategy depends on the implementation approach.

---

### Front-load context, back-load the ask

**Why & when it matters:**
Claude's exploration is more focused when the prompt establishes context before stating the request.

**Example:**
```
❌ "Add rate limiting. We're using Express. The API is in src/api/. We've had 
   issues with abuse."
✅ "Context: We're seeing API abuse — 10k requests/min from single IPs. The API 
   is Express-based, in src/api/, currently has no rate limiting. We have Redis 
   available. Request: Add rate limiting with per-IP limits and a global fallback."
```

**How to apply it:**
Structure your prompt: Context (what exists, what's broken, what's constrained) → Request (what you want) → Constraints (what you don't want). This order helps Claude build the right mental model before exploring.

**Pitfalls:**
Don't bury the actual request in paragraphs of context. One paragraph of context, one clear sentence for the request, one paragraph of constraints.

---

## Guiding Exploration

### Understand how Claude uses Explore agents

**Why & when it matters:**
Phase 1 spawns parallel subagents to read your codebase — knowing this helps you guide exploration efficiently.

**Example:**
You ask Claude to plan adding authentication. During Phase 1, Claude spawns Explore agents to search for existing auth patterns, read relevant files, and identify where auth checks happen. These agents run in parallel and report back before the plan is written. If you see Claude "exploring" for 30 seconds, this is what's happening.

**How to apply it:**
Don't interrupt exploration — let it complete. If exploration takes longer than expected, it means Claude is reading more files than you anticipated, which usually indicates the task is more complex than you thought. When exploration finishes, Claude will summarize what it found before writing the plan.

**Pitfalls:**
Don't assume exploration is comprehensive — it's focused on what Claude thinks is relevant. If you know a critical file wasn't mentioned in the exploration summary, point it out before approving the plan.

---

### Provide known file paths upfront

**Why & when it matters:**
Exploration is faster and more focused when you give Claude starting points instead of making it search.

**Example:**
```
❌ "Add logging to the API"
✅ "Add logging to the API. Start by reading src/api/handlers/ and src/lib/logger.ts 
   to understand the existing logging setup"
```

**How to apply it:**
If you know which files are relevant, list them in your initial prompt. Use `src/path/to/file.ts` format. Claude will prioritize reading those files during exploration. If you're not sure of exact paths, give directory hints: "probably in src/auth/" is better than nothing.

**Pitfalls:**
Don't list files you're not confident about — wrong paths waste exploration time. If you're guessing, say so: "I think it's in src/api/ but not sure."

---

### Tell Claude what NOT to explore

**Why & when it matters:**
Negative scope prevents Claude from reading irrelevant code that could confuse the plan.

**Example:**
```
❌ "Refactor the user service"
✅ "Refactor src/services/user.ts. Don't explore src/services/admin/ — that's a 
   separate system. Don't read the test files yet — we'll update those after 
   the refactor plan is approved."
```

**How to apply it:**
If your codebase has areas that look related but aren't (legacy code, separate services, generated files), explicitly exclude them. Common exclusions: test files (unless you're planning test changes), build output, vendor/node_modules, deprecated code marked for deletion.

**Pitfalls:**
Don't exclude files that might have hidden dependencies. If you're not sure whether something is relevant, let Claude explore it — false negatives are worse than false positives.

---

### Balance free exploration vs. precise direction

**Why & when it matters:**
Over-directing exploration misses insights; under-directing wastes time on irrelevant code.

**Example:**
When to direct precisely: "Read src/auth/middleware.ts and src/auth/session.ts — those are the only two files that handle sessions." (You know the scope exactly)

When to explore freely: "Figure out how authentication works in this codebase." (You don't know the scope — let Claude discover it)

**How to apply it:**
If you know exactly what needs to change, be precise. If you're asking Claude to understand an unfamiliar area, give a starting point but let exploration be broad. The prompt "understand X" signals free exploration; "change X" signals directed exploration.

**Pitfalls:**
Don't micromanage exploration on complex tasks — you might know the entry point but not the full dependency graph. Let Claude find the edges.

---

### Read Claude's exploration findings before approving

**Why & when it matters:**
The exploration summary reveals what Claude understands about your codebase — misunderstandings here become bad plans.

**Example:**
Claude's exploration summary says: "Found authentication in src/auth/basic.ts using HTTP Basic Auth." You know the app also has OAuth in src/auth/oauth.ts that Claude missed. If you approve without correcting this, the plan will ignore OAuth and break existing users.

**How to apply it:**
When Claude finishes exploration and presents findings, read them critically. Ask yourself: "Did Claude find everything relevant? Did it misinterpret anything? Are there files or patterns it missed?" If yes, correct before the plan is written: "You missed src/auth/oauth.ts — we support both Basic and OAuth."

**Pitfalls:**
Don't assume Claude's exploration is exhaustive. Exploration is heuristic — it reads what seems relevant based on file names, imports, and keywords. Edge cases and non-obvious dependencies can be missed.

---

### Use exploration to surface unknown unknowns

**Why & when it matters:**
Exploration often reveals complexity you didn't know existed — use this to refine your request before planning.

**Example:**
You ask Claude to "add rate limiting to the API." Exploration reveals the API has three separate entry points (REST, GraphQL, WebSocket) that you forgot about. Before approving the plan, you clarify: "Only add rate limiting to the REST API for now — GraphQL and WebSocket can wait."

**How to apply it:**
Treat exploration findings as a discovery phase. If Claude's summary includes files or patterns you didn't expect, pause and ask: "Does this change what I'm asking for?" Adjust your request based on what exploration revealed, then let Claude re-plan with the updated scope.

**Pitfalls:**
Don't ignore surprising exploration findings. If Claude says "Found 5 implementations of X" and you thought there was only 1, that's a signal to investigate before proceeding.

---

### Ask Claude to look for existing utilities before proposing new ones

**Why & when it matters:**
Codebases accumulate utilities — exploration can find them so you reuse instead of duplicate.

**Example:**
```
❌ "Add input validation to the API"
✅ "Add input validation to the API. During exploration, check if we already have 
   a validation utility or schema library. If we do, use it. If not, propose one."
```

**How to apply it:**
When your task involves common functionality (validation, logging, error handling, caching), explicitly ask Claude to search for existing implementations during exploration. Use phrases like "check if we already have," "look for existing patterns," or "see how other endpoints handle this."

**Pitfalls:**
Don't assume Claude will automatically find and reuse utilities — it needs to be told to look. Exploration focuses on the task at hand, not on discovering all possible utilities.

---

## Shaping the Design

### Push Claude toward your preferred architecture without over-constraining

**Why & when it matters:**
You want Claude to consider your architectural preferences while still finding better solutions you haven't thought of.

**Example:**
```
❌ "Use a factory pattern with dependency injection"
✅ "I prefer keeping business logic separate from framework code. Consider how 
   to structure this so the core logic doesn't depend on Express directly."
```

**How to apply it:**
State architectural principles, not specific patterns. "Keep X decoupled from Y" is better than "use pattern Z." Explain why you prefer an approach ("so we can test without mocking the framework") so Claude can apply that reasoning to find the best solution.

**Pitfalls:**
Don't prescribe patterns when you're actually just stating a constraint. If you say "use dependency injection" but what you really mean is "make this testable," say the latter — Claude might find a simpler solution.

---

### Ask for multiple approaches with explicit tradeoffs

**Why & when it matters:**
Complex decisions benefit from seeing alternatives side-by-side before committing to implementation.

**Example:**
You ask Claude to plan adding real-time updates to the app. Instead of approving the first plan (WebSockets), you ask: "Show me three approaches: WebSockets, Server-Sent Events, and polling. For each, explain the tradeoffs in terms of server load, client complexity, and browser support."

**How to apply it:**
When the task has multiple viable solutions, explicitly request alternatives in your initial prompt: "Propose 2-3 approaches and explain the tradeoffs." Claude will explore each option and present them for comparison. This works best for architectural decisions, not implementation details.

**Pitfalls:**
Don't ask for alternatives on trivial decisions — it wastes time. Reserve this for choices that are expensive to reverse or have significant tradeoffs.

---

### Make Claude justify its architectural choices

**Why & when it matters:**
Understanding why Claude chose an approach helps you evaluate whether it's right for your context.

**Example:**
Claude's plan proposes adding a message queue. Before approving, you ask: "Why a queue instead of direct API calls? What problem does the queue solve?" Claude explains: "The external API has rate limits and occasional downtime — the queue provides retry logic and prevents request loss." Now you can evaluate whether that's worth the added complexity.

**How to apply it:**
When a plan includes a significant architectural decision (new database, new service, new pattern), ask "why this over the simpler alternative?" Claude should be able to articulate the problem being solved and why the added complexity is justified.

**Pitfalls:**
Don't ask for justification on every small decision — focus on the load-bearing choices. If Claude proposes a helper function, that doesn't need justification. If it proposes a new microservice, it does.

---

### Anchor the plan to existing patterns in the codebase

**Why & when it matters:**
Consistency with existing code is often more valuable than theoretical best practices.

**Example:**
```
❌ "Add error handling"
✅ "Add error handling following the same pattern as src/api/users.ts — wrap in 
   try/catch, use our custom ApiError class, and let the error middleware handle 
   the response formatting"
```

**How to apply it:**
When your codebase has established patterns for common concerns (error handling, validation, logging, data access), reference them explicitly. Claude will read the example and replicate its structure. This produces plans that feel native to your codebase.

**Pitfalls:**
Don't anchor to bad patterns. If the existing code has known issues, say so: "Follow the pattern in users.ts but use async/await instead of callbacks — we're migrating away from callbacks."

---

### Demand exact file paths and line numbers in plans

**Why & when it matters:**
Vague plans ("update the auth logic") are hard to review and often miss edge cases — specific plans are reviewable and executable.

**Example:**
```
❌ Plan says: "Update the authentication middleware to check for API keys"
✅ Plan says: "Update src/middleware/auth.ts:validateRequest (lines 23-45) to 
   check for API keys in the Authorization header before checking for session 
   cookies. Add new function extractApiKey at line 15."
```

**How to apply it:**
When reviewing a plan, check: does it name specific files, functions, and line ranges? If not, ask Claude to be more specific before approving. A good plan should let you mentally walk through the changes without reading the code.

**Pitfalls:**
Don't expect line-number precision for new code — Claude can't predict where new functions will land. But for modifications to existing code, exact line ranges should be specified.

---

### Get Claude to identify the full blast radius

**Why & when it matters:**
Changes have ripple effects — identifying all affected files upfront prevents mid-implementation surprises.

**Example:**
You ask Claude to rename a function. The plan says "rename `getUserData` to `fetchUserProfile` in user-service.ts." You ask: "What else calls this function?" Claude searches and finds 8 call sites across 5 files, plus 3 test files. Now the plan includes updating all call sites.

**How to apply it:**
For any change that modifies a public interface (function signature, API endpoint, database schema), explicitly ask: "What else depends on this? What breaks if we change it?" Claude will search for usages and include them in the plan.

**Pitfalls:**
Don't assume Claude automatically finds all dependencies — it needs to be asked. Exploration focuses on the primary task, not exhaustive dependency analysis.

---

### Know when to accept Claude's architecture vs. when to push back

**Why & when it matters:**
Claude's proposals are informed by general best practices but may not fit your specific constraints — you need to evaluate critically.

**Example:**
Claude proposes adding Redis for caching. You know your app runs on a single server with low traffic. You push back: "Redis is overkill for our scale — propose an in-memory cache instead." Claude revises to use a simple Map with TTL logic. Much simpler, fits your actual needs.

**How to apply it:**
Evaluate proposals against your actual constraints: team size, traffic scale, deployment model, maintenance burden. If a proposal feels over-engineered for your context, say so and ask for a simpler alternative. If it feels under-engineered, ask what happens at higher scale.

**Pitfalls:**
Don't reject proposals just because they're unfamiliar — sometimes Claude suggests better patterns you haven't used. Push back on complexity, not novelty.

---

### Include verification steps in the plan itself

**Why & when it matters:**
A plan without a testing strategy often produces code that works in theory but fails in practice.

**Example:**
A plan to add rate limiting should include: "Verification: 1) Send 100 requests in 10 seconds from the same IP, confirm the 11th is rate-limited. 2) Send requests from different IPs, confirm each gets their own limit. 3) Wait for the time window to reset, confirm requests succeed again."

**How to apply it:**
Before approving a plan, check: does it explain how to verify the changes work? If not, ask: "How do I test this end-to-end?" Claude should specify manual testing steps, automated test additions, or both.

**Pitfalls:**
Don't confuse verification with unit tests — verification is "how do I know this works in the real system?" Unit tests are part of implementation, not the plan.

---

## Evaluating & Iterating on the Plan

### Read plans critically, not optimistically

**Why & when it matters:**
Plans that look reasonable at first glance often have subtle flaws that become expensive problems during implementation.

**Example:**
Claude's plan says "Add a new `roles` column to the users table." Sounds fine. But you read critically and realize: this is a production database with 2M users, adding a column requires a table lock, and your deployment window is 5 minutes. The plan doesn't mention migration strategy. You push back: "How do we add this column without downtime?" Claude revises to use a multi-step migration with a shadow column.

**How to apply it:**
When reading a plan, actively look for problems. Ask: What's missing? What assumptions is this making? What could go wrong? What's the rollback strategy? Don't approve because it sounds plausible — approve because you've thought through the failure modes and they're acceptable.

**Pitfalls:**
Don't be so critical that you reject every plan. The goal is to find real problems, not to prove you're smarter than Claude. If you can't articulate a specific concern, the plan is probably fine.

---

### Apply the "does this tell me something I didn't know?" test

**Why & when it matters:**
Plans that just restate your request in different words aren't adding value — they're wasting time.

**Example:**
You ask Claude to "add input validation to the API." The plan says: "Step 1: Add validation. Step 2: Return errors for invalid input. Step 3: Add tests." This tells you nothing you didn't already know. You reject it: "Be specific — what fields need validation? What are the validation rules? Where does the validation logic live?" The revised plan names specific fields, cites existing validation patterns, and proposes exact error messages.

**How to apply it:**
After reading a plan, ask yourself: "Did I learn anything?" A good plan should surface details you hadn't considered: edge cases, existing code that needs updating, tradeoffs between approaches, or constraints you didn't know about. If the plan is just your request rephrased, it's not ready.

**Pitfalls:**
Don't expect plans to teach you your own domain — the "something new" might be about the codebase structure, not the business logic. A plan that says "reuse the existing validator in src/lib/validate.ts" is valuable even if the validation rules themselves are obvious.

---

### Make Claude call out its assumptions explicitly

**Why & when it matters:**
Unstated assumptions become bugs when they turn out to be wrong.

**Example:**
Claude's plan for adding a feature says "update the API endpoint." You ask: "What assumptions are you making about the API?" Claude responds: "I'm assuming the endpoint is synchronous and returns immediately. If it's async or queues work, this approach won't work." You realize the endpoint IS async. The plan gets revised before any code is written.

**How to apply it:**
When a plan makes a decision that depends on how existing code works, ask: "What are you assuming about X?" Common assumptions to surface: synchronous vs. async, single-server vs. distributed, SQL vs. NoSQL, authentication requirements, data volume, and API contracts.

**Pitfalls:**
Don't ask about assumptions on trivial decisions. If Claude proposes a helper function, you don't need to interrogate its assumptions. Focus on load-bearing decisions where wrong assumptions break things.

---

### Know how to reject and redirect effectively

**Why & when it matters:**
A vague rejection ("this isn't right") forces Claude to guess what you want — a specific redirect produces a better plan faster.

**Example:**
```
❌ "This plan won't work, try again"
✅ "This plan assumes we can modify the User table schema, but that table is in a 
   shared database we don't control. Revise to use a separate UserRoles table 
   with a foreign key to User.id instead."
```

**How to apply it:**
When rejecting a plan, explain: 1) What's wrong (the specific problem), 2) Why it's wrong (the constraint or context Claude missed), and 3) What direction to go instead (the alternative approach or additional context to consider). This gives Claude everything needed to revise without another round of guessing.

**Pitfalls:**
Don't over-specify the solution when redirecting. "Use a separate table" is enough — you don't need to dictate the exact schema. Give Claude room to find the best implementation of your direction.

---

### Iterate incrementally: approve parts, refine the rest

**Why & when it matters:**
Large plans often have some parts that are solid and others that need work — you don't have to approve or reject the whole thing.

**Example:**
Claude's plan has three parts: 1) Database schema changes (looks good), 2) API endpoint updates (needs revision), 3) Frontend changes (looks good). Instead of rejecting the whole plan, you say: "Parts 1 and 3 are approved. For part 2, revise to handle the async case I mentioned. Keep parts 1 and 3 as-is."

**How to apply it:**
When reviewing a multi-part plan, evaluate each part independently. Approve the parts that are ready, and ask for revisions only on the parts that need work. This speeds up iteration and prevents Claude from unnecessarily revising things that were already correct.

**Pitfalls:**
Don't approve parts that depend on the parts you're rejecting. If the API changes depend on the database schema, and you're rejecting the schema, reject both and ask for a revised approach to the whole thing.

---

### Understand when to call `ExitPlanMode` vs. keep refining

**Why & when it matters:**
Calling `ExitPlanMode` too early produces misaligned implementations; iterating too long wastes time on diminishing returns.

**Example:**
After two rounds of iteration, the plan is 90% there but you're still uncertain about one edge case. You have two options: 1) Approve and handle the edge case during implementation, or 2) Iterate one more time to nail down the edge case in the plan. If the edge case is complex and affects multiple files, iterate. If it's a small detail that won't ripple, approve and handle it during implementation.

**How to apply it:**
Call `ExitPlanMode` when: the plan is specific enough to execute, you understand the approach and agree with it, and any remaining uncertainties are small enough to resolve during implementation. Keep refining when: the plan is vague, you don't understand the approach, or there are architectural decisions still unresolved.

**Pitfalls:**
Don't use "I'll figure it out during implementation" as an excuse to approve a vague plan. If you can't mentally walk through the implementation based on the plan, it's not ready.

---

### Use `AskUserQuestion` responses to steer toward your intent

**Why & when it matters:**
Claude's questions during planning reveal what it's uncertain about — your answers shape the plan more than your initial prompt.

**Example:**
You ask Claude to add caching. Claude asks: "Should the cache be per-user or global? What's the expected cache hit rate? Are there any endpoints that should never be cached?" Your answers to these questions determine the entire caching strategy. If you answer vaguely ("whatever makes sense"), the plan will be generic. If you answer specifically ("per-user, 80% hit rate expected, never cache /admin endpoints"), the plan will be tailored.

**How to apply it:**
When Claude asks questions during planning, treat them as opportunities to inject critical context. Don't give minimal answers — explain the reasoning behind your answer so Claude can apply that reasoning to related decisions. "Per-user because different users see different data" is better than just "per-user."

**Pitfalls:**
Don't answer questions you're not sure about. "I don't know, what do you recommend?" is a valid answer. Claude will propose options and explain tradeoffs, then you can decide.

---

## Workflow & Collaboration

### Use `CLAUDE.md` to pre-load constraints

**Why & when it matters:**
Every planning session starts fresh — `CLAUDE.md` ensures critical constraints are always in context without repeating yourself.

**Example:**
You create `CLAUDE.md` in your repo root with: "We're on Postgres 12. The User table is in a shared schema we can't modify. All API changes must maintain backwards compatibility with mobile app v2.1+." Now every time you invoke planning mode, Claude reads this file first and factors these constraints into every plan automatically.

**How to apply it:**
Create a `CLAUDE.md` file at your repo root. Document: deployment constraints, database/framework versions, API contracts that can't change, architectural decisions, and any "we tried X and it didn't work" history. Keep it under 500 words — it's loaded into every conversation, so brevity matters.

**Pitfalls:**
Don't put temporary project state in `CLAUDE.md` — it's for durable constraints, not current work. Don't let it grow stale — outdated constraints are worse than no constraints.

---

### Treat the plan file as a shareable artifact

**Why & when it matters:**
The plan file is a markdown document you can share with teammates for async review before implementation starts.

**Example:**
Claude writes a plan to `.claude/plans/add-caching.md`. Before approving, you send the file to a senior engineer on Slack: "Does this caching approach make sense?" They review, suggest using Redis instead of in-memory cache, you redirect Claude, and the revised plan gets approved. The plan file served as a design doc without you writing one.

**How to apply it:**
After Claude writes a plan, the file lives in `.claude/plans/`. You can read it, share it, or commit it to git. Treat it like a lightweight design doc — it's more concrete than a proposal but less final than merged code. Use it to get feedback before implementation.

**Pitfalls:**
Don't commit plan files to main unless they're genuinely useful as documentation. Most plans are ephemeral — they're valuable during planning, not after implementation.

---

### Review plans with teammates before approving

**Why & when it matters:**
Complex architectural decisions benefit from multiple perspectives — planning mode makes this easy because the plan exists before any code is written.

**Example:**
You're planning a database migration. The plan looks reasonable to you, but you're not a database expert. You paste the plan into your team's Slack channel: "Claude proposed this migration strategy — does it handle the zero-downtime requirement correctly?" Your DBA spots that the plan doesn't account for long-running transactions and suggests a different approach. You redirect Claude before any code is written.

**How to apply it:**
For plans that involve: database changes, API contract changes, security-sensitive code, or unfamiliar domains, share the plan file with relevant experts before approving. Ask specific questions: "Does this handle X?" not "Does this look okay?"

**Pitfalls:**
Don't ask for review on trivial plans — you'll train your team to ignore your requests. Reserve collaborative review for decisions that are expensive to reverse or outside your expertise.

---

### Understand how plan mode interacts with tasks

**Why & when it matters:**
Planning mode can create tasks for implementation steps — this helps you track progress on complex plans.

**Example:**
Claude's plan has 5 distinct steps: schema migration, API updates, cache layer, tests, and documentation. During planning, you ask: "Create tasks for each of these steps." Claude uses `TaskCreate` to break the plan into trackable tasks. As you implement, you mark each task complete. If you get interrupted, the task list shows exactly where you left off.

**How to apply it:**
For multi-step plans, ask Claude to create tasks during or after planning. Each task should be independently completable. As you work through implementation, use `TaskUpdate` to mark progress. This is especially valuable for plans that span multiple sessions.

**Pitfalls:**
Don't create tasks for every tiny detail — tasks are for meaningful milestones, not individual function edits. A 5-step plan should have 5 tasks, not 20.

---

### Chain planning sessions for large features

**Why & when it matters:**
Features too large for one plan can be broken into a sequence of planning sessions, each building on the previous.

**Example:**
You're adding a new payment provider. Session 1: "Plan the provider interface and how it integrates with existing payment code." Approve and implement. Session 2: "Plan the Stripe-specific implementation using the interface from session 1." Approve and implement. Session 3: "Plan the migration strategy for moving existing customers to the new provider." Each session produces a focused, reviewable plan.

**How to apply it:**
When a feature has multiple independent decisions (interface design, implementation, migration), plan them sequentially. Implement each plan before starting the next planning session. This prevents plans from becoming too abstract and ensures each decision is informed by working code.

**Pitfalls:**
Don't chain sessions for tightly coupled work — if decision B depends on decision A, plan them together. Only chain when the decisions are genuinely separable.

---

### Use plan files as PR description templates

**Why & when it matters:**
The plan already explains what's changing and why — it's a ready-made PR description.

**Example:**
After implementing a plan, you open a PR. Instead of writing a description from scratch, you copy the plan file's summary section: "This PR adds rate limiting to the API. Approach: per-IP limits using Redis, with a global fallback. Files changed: middleware/rateLimit.ts (new), api/server.ts (updated to use middleware)." The plan becomes your PR description with minimal editing.

**How to apply it:**
When opening a PR for planned work, start with the plan file. Copy the approach summary, the list of files changed, and the verification steps. Edit for brevity — remove planning-specific details, keep the what and why. This saves time and ensures your PR description matches what was actually implemented.

**Pitfalls:**
Don't copy the entire plan verbatim — PR descriptions should be concise. Extract the summary, not the full exploration findings and iteration history.

---

### Commit plan files to git for durable decision records

**Why & when it matters:**
Plan files are ephemeral by default — committing them before approval creates a permanent record of what was decided and why, useful for post-mortems and reverting misaligned implementations.

**Example:**
You approve a plan to refactor the auth system. Three weeks later, a bug surfaces and you need to understand why the current approach was chosen over alternatives. The plan file (committed before implementation) shows the tradeoffs discussed, the constraints that drove the decision, and what was explicitly ruled out. Without it, you're reverse-engineering decisions from code and commit messages.

**How to apply it:**
After Claude writes a plan and you've iterated to approval, commit the plan file from `.claude/plans/` to git before starting implementation. Use a commit message like "Plan: refactor auth system" so it's findable in git log. This creates a durable record that survives beyond the conversation. If the implementation diverges from the plan, the diff between plan and reality reveals where and why.

**Pitfalls:**
Don't commit every draft plan — only commit plans you've approved and intend to implement. Don't commit plans for trivial tasks (typo fixes, dependency updates) — the overhead isn't worth it. Do commit plans for architectural decisions, multi-file refactors, or anything where future-you will ask "why did we do it this way?"

---

## Anti-Patterns & Traps

### Don't approve plans you haven't read

**Why & when it matters:**
Approving unread plans is the fastest way to waste hours on misaligned implementations.

**Example:**
You ask Claude to "add user roles." Claude writes a plan. You see it's 200 lines and think "looks thorough" without reading. You approve. Implementation starts. 30 minutes later you realize the plan added roles to the wrong table and now you're unwinding changes. If you'd spent 3 minutes reading the plan, you'd have caught this before any code was written.

**How to apply it:**
Treat plan approval like code review — you wouldn't merge a PR without reading the diff. Read the plan file top to bottom. Check: does it match what you asked for? Does it handle edge cases? Are the file paths correct? If you don't have time to read the plan, you don't have time to fix a bad implementation.

**Pitfalls:**
Don't skim and approve. The most expensive mistakes hide in details you skip. If the plan is too long to read carefully, it's too complex — ask Claude to break it into smaller planning sessions.

---

### Avoid underspecified prompts that produce vague plans

**Why & when it matters:**
"Add feature X" without context forces Claude to guess your requirements — vague input produces vague plans.

**Example:**
```
❌ "Add caching"
Result: Plan says "add a cache layer" with no specifics about what to cache, TTL, invalidation strategy, or where the cache lives. You approve because it sounds reasonable. Implementation is generic and doesn't fit your actual needs.

✅ "Add caching to GET /api/users with 5-minute TTL using the existing Redis client in src/cache.ts. Don't cache authenticated requests."
Result: Plan is specific, reviewable, and produces exactly what you need.
```

**How to apply it:**
Before invoking plan mode, answer: what exactly am I asking for? What constraints matter? What should the end state look like? Put those answers in your prompt. Specific prompts produce specific plans.

**Pitfalls:**
Don't confuse "specific" with "prescriptive." Specific means clear requirements and constraints. Prescriptive means dictating implementation steps. Be specific about what, not how.

---

### Don't over-constrain and miss better solutions

**Why & when it matters:**
Locking Claude into your exact approach defeats the purpose of planning — you lose the chance to discover better solutions.

**Example:**
You tell Claude: "Add a cache using a Map with manual TTL tracking." Claude implements exactly that. Later you realize Redis was already in the project and would have been simpler. You over-constrained the solution and missed the better option. If you'd said "add a cache, we have Redis available" Claude would have proposed using Redis.

**How to apply it:**
State requirements and constraints, not implementation details. "The cache needs to work across multiple servers" is a constraint. "Use Redis with a 5-minute TTL" is over-constraining. Let Claude explore and propose solutions, then evaluate them.

**Pitfalls:**
Don't swing too far the other way — some constraints are necessary. "Don't add new dependencies" or "keep the API contract unchanged" are valid constraints that prevent bad solutions.

---

### Understand that plan mode is not a one-way door

**Why & when it matters:**
Many users approve bad plans because they think rejecting means starting over — iteration is expected and cheap.

**Example:**
Claude's plan proposes a database migration that requires downtime. You're uncomfortable but approve anyway because you think "I already asked for this, I can't change my mind now." Wrong. You can and should push back: "This downtime isn't acceptable — revise to use a zero-downtime migration strategy." Claude will revise the plan. Iteration is the intended workflow.

**How to apply it:**
Treat every plan as a draft. If something feels wrong, say so. If you realize mid-review that your original request was wrong, redirect. The cost of iteration is minutes. The cost of implementing a bad plan is hours.

**Pitfalls:**
Don't iterate forever — if you're on round 5 and still not happy, the problem might be that the task is too large or your requirements are unclear. Break it down or clarify what you actually want.

---

### Avoid analysis paralysis: know when to stop refining

**Why & when it matters:**
Perfect plans don't exist — at some point you need to approve and start implementing.

**Example:**
You're on the third iteration of a plan. It's 95% right but you keep finding small things to refine. You ask for a fourth revision to handle an edge case that affects 0.1% of users. You're now 20 minutes into planning a 30-minute task. You've crossed into analysis paralysis.

**How to apply it:**
Ask: "Is this uncertainty blocking implementation, or can I handle it during implementation?" If the plan is specific enough to start coding and any remaining questions are small details, approve and move forward. Save iteration for architectural decisions, not minor details.

**Pitfalls:**
Don't use "good enough" as an excuse to approve vague plans. The threshold is "specific enough to execute," not "perfect in every detail."

---

### Don't confuse plans with specs

**Why & when it matters:**
Plans are for the current task — they're not documentation for future work or comprehensive design specs.

**Example:**
You ask Claude to add a feature. The plan is good. You approve. Then you ask: "What about internationalization? What about mobile support? What about the admin UI?" Claude's plan didn't cover these because you didn't ask for them. Plans scope to the request, not to every possible future extension.

**How to apply it:**
Plans answer "how do I implement what I asked for?" not "how does this fit into the entire system?" If you need a comprehensive design doc, ask for that explicitly. If you need a plan for a specific task, don't expect it to cover unrelated work.

**Pitfalls:**
Don't approve a plan and then immediately ask for unrelated features. If you realize mid-planning that the scope should be larger, redirect before approving — don't approve a narrow plan and then expand scope during implementation.

---

### Avoid re-entering plan mode mid-implementation

**Why & when it matters:**
Starting a new planning session while implementing a previous plan creates confusion and abandoned work.

**Example:**
You're halfway through implementing a plan to add caching. You realize you also need rate limiting. You invoke plan mode again: "add rate limiting." Claude starts a new planning session, explores the codebase, and writes a new plan — but now you have half-implemented caching and a new plan for rate limiting. The two plans don't coordinate. You end up with conflicts and wasted work.

**How to apply it:**
Finish implementing the current plan before starting a new planning session. If you discover new work mid-implementation, write it down and plan it after the current task is complete. If the new work is genuinely blocking, explicitly tell Claude: "Pause the caching implementation, we need to plan rate limiting first because it affects the caching design."

**Pitfalls:**
Don't use this as a reason to never plan — if you discover a genuinely large new task mid-implementation, it's fine to pause and plan it. Just be explicit about pausing the current work.

---

### Don't approve plans with vague verification steps

**Why & when it matters:**
Plans without concrete testing criteria often produce code that passes tests but fails in production.

**Example:**
Claude's plan for adding rate limiting ends with "Verification: test that it works." You approve. Implementation completes. You manually test with a few requests and it seems fine. You deploy. Production traffic reveals the rate limiter doesn't reset properly across server restarts, causing false positives. A concrete verification step would have caught this: "Send 100 requests, restart the server, send 100 more — confirm limits reset correctly."

**How to apply it:**
Before approving a plan, check the verification section. Does it specify exact steps you can execute? Does it cover edge cases (restarts, concurrent requests, error conditions)? If the verification is generic ("test it", "make sure it works"), ask Claude to be specific: "How exactly do I verify this? What are the test cases?"

**Pitfalls:**
Don't confuse verification steps with unit tests. Verification is "how do I know this works in the real system?" Unit tests are part of implementation. Both matter, but they're different.

---

### Don't use plan mode for exploratory coding

**Why & when it matters:**
If you're spiking to understand a problem, direct prompting is faster — plan mode is for implementation, not discovery.

**Example:**
You're investigating why API latency spiked. You don't know if it's the database, the cache, or the external service. You invoke plan mode: "figure out why the API is slow." Claude explores, writes a plan to add logging. But you don't need a plan — you need to poke around, run queries, check metrics. Direct prompts like "show me recent slow queries" or "check Redis hit rates" would have been faster.

**How to apply it:**
Use plan mode when you know what needs to change but not how to change it safely. Use direct prompts when you don't know what's wrong yet. Exploration is iterative and non-linear — planning mode's structured workflow adds overhead without benefit. Once exploration reveals the problem, then plan the fix.

**Pitfalls:**
Don't avoid planning legitimate fixes just because you discovered them through exploration. "Explore with prompts, plan the fix" is the right sequence.

---

### Avoid planning when you're unclear on requirements

**Why & when it matters:**
Vague requests produce vague plans — planning mode amplifies clarity, it doesn't create it.

**Example:**
Your PM says "we need better error handling" without specifics. You invoke plan mode with that exact phrase. Claude explores, finds 50 places errors are handled differently, and writes a vague plan to "standardize error handling." You approve because it sounds reasonable. Implementation starts and you realize you don't know what "standardized" means — should errors be logged? Returned to the client? Sent to Sentry? The plan didn't clarify because your request didn't clarify.

**How to apply it:**
Before invoking plan mode, answer: what does success look like? What specific problem am I solving? If you can't answer these, clarify requirements first — talk to your PM, check the ticket, look at user reports. Once you know what you want, then plan how to build it. Planning mode is not a requirements-gathering tool.

**Pitfalls:**
Don't use "I'll figure it out during planning" as an excuse to skip requirements work. Claude can't read your PM's mind any better than you can.

---

## Glossary

**Planning mode** — A workflow in Claude Code where Claude explores your codebase in read-only mode, writes a plan to a file, and requests approval before making any changes.

**`ExitPlanMode`** — The tool call Claude uses to signal that planning is complete and the plan is ready for your review and approval.

**Explore agents** — Parallel subagents Claude spawns during Phase 1 to read files, search for patterns, and gather context about your codebase before writing a plan.

**Plan file** — The markdown document (stored in `.claude/plans/`) where Claude writes its implementation plan, including approach, file changes, and verification steps.

**Phase 1 (Exploration)** — The read-only phase of planning mode where Claude uses Explore agents to understand your codebase before proposing changes.

**`AskUserQuestion`** — The tool Claude uses to ask clarifying questions during planning, helping refine requirements and resolve uncertainties before implementation.

**`CLAUDE.md`** — A file in your repo root that Claude reads automatically at the start of every conversation, used to document durable constraints and project context.

**Blast radius** — The full set of files, functions, and systems affected by a proposed change, including indirect dependencies and downstream impacts.

---

## Contributing

Contributions that improve or expand this playbook are welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) for the quality bar, submission process, and copy-paste template.

---

## License

MIT
