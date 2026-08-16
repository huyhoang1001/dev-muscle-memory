# Muscle Memory for AI Agents: How I Stopped Repeating Myself Across Every Rust Project

Every senior engineer has a checklist in their head.

When you review code, you're not reading it fresh — you're pattern-matching. You've seen the `Arc<Mutex<T>>` where a single owner would've been fine. You've seen the `unwrap()` quietly panic in production at 3am. You've seen the async function blocking the runtime thread because someone reached for `std::fs` instead of `tokio::fs`. You know these things because you've been burned by them, or you've watched someone else get burned, and now the pattern lives in your muscle memory.

The problem is, AI agents don't have that muscle memory. They're fluent in Rust. They know the syntax, the standard library, the common patterns. But they don't carry your hard-won opinions about *this* codebase, *this* team's conventions, *this* hot path that cannot take a lock. Every new session, you're back to briefing a capable engineer who just walked in the door.

This is what `dev-muscle-memory` tries to solve.

---

## The Problem With Context Windows as Knowledge Storage

The naive approach is to dump everything into the system prompt. Your architecture docs, your coding conventions, your review checklist, your crate preferences. It works, sort of — until your context fills up with boilerplate that's irrelevant to the current task, the model starts ignoring the middle of long prompts, and you're paying for tokens that aren't helping.

The slightly better approach is to paste relevant docs manually before each session. This is what most people do. It's tedious, it breaks flow, and it relies on you remembering what to include.

What we actually want is something more like how a good engineer onboards. You don't hand them a 200-page document. You give them a map of the codebase, some conventions to follow, and pointers to deeper references they can pull when they need them. The rest they learn by doing.

That's the model Agent Skills are built on: **progressive disclosure**. Only the name and description of a skill loads at startup. The full instructions load when the agent decides they're relevant. Reference files — the detailed checklists, the pattern catalogs, the edge case guides — only load when the agent explicitly reaches for them.

This keeps context lean. It makes agents faster and more focused. And it means you can pack a lot of specialized knowledge into a skill without paying for it on every turn.

---

## What dev-muscle-memory Actually Is

`dev-muscle-memory` is a collection of agent skills and agent configs built specifically for Rust development. It lives at [github.com/huyhoang1001/dev-muscle-memory](https://github.com/huyhoang1001/dev-muscle-memory) and follows the open [Agent Skills](https://agentskills.io/) standard, which means it works with Kiro, Claude Code, Cursor, Codex, and about 30 other agentic tools.

There are three skills:

**`rust-code-review`** — a structured code review skill that focuses on what the Rust compiler *cannot* catch. Rustc already handles memory safety; this is everything else. Unsafe soundness (because `// SAFETY:` comments are load-bearing documentation, not optional noise). Async hazards — holding a `std::sync::Mutex` across an `.await` point is legal code that will deadlock under load. Cancellation safety in `select!` — whether a `Future` dropped mid-execution loses data is not obvious and rarely documented. Ownership patterns — when a `.clone()` is the right call versus when it's papering over a design problem. Error handling — the library/application split between `thiserror` and `anyhow`, and why `unwrap()` in production library code is an API contract violation. Performance — why `collect()` followed immediately by iteration is a code smell, and when `Box<dyn Trait>` in a hot path should make you pause.

The review follows a severity model: P0 blocks merge, P1 should fix before merge, P2 is a follow-up, P3 is optional. It never implements fixes without being asked. It presents findings and waits.

**`rust-dev`** — a development assistant oriented toward action rather than critique. Compiler error triage that explains the *root cause*, not just "try adding `.clone()`." Pattern selection — typestate, state machine, sealed traits, retry with backoff, cancellation tokens — with full working examples. A curated crate list organized by use case (async runtime, error handling, HTTP, database, observability, testing, parsing, security). Project structure guidance covering workspace layout, feature flag design, and CI configuration.

**`repo-explorer`** — maps an unfamiliar codebase. Module responsibilities, architectural patterns (layered, event-driven, actor model, middleware pipeline), data flow from entry point to output, conventions, and the landmines that aren't documented anywhere. Useful before making a large change to a codebase you haven't touched in months, or before onboarding a new contributor.

Each skill comes with `references/` — detailed Markdown files that get loaded on demand. `rust-code-review` has seven: ownership, unsafe, async, errors, performance, API design, and a removal plan template. The main `SKILL.md` tells the agent *what* to do; the references tell it *how* in detail.

---

## The Structure: Skills vs Agents

Here's where it gets interesting, and where most similar projects make a mistake.

```
dev-muscle-memory/
├── skills/          ← portable, platform-agnostic
│   ├── rust-code-review/
│   ├── rust-dev/
│   └── repo-explorer/
└── agents/
    └── kiro/        ← platform-specific
        ├── rust-code-review.json
        └── rust-dev.json
```

Skills and agents solve different problems.

A **skill** is a portable instruction package. It contains the knowledge — the checklist, the patterns, the reference guides. It's a `SKILL.md` file with YAML frontmatter and Markdown content, plus a `references/` folder for detailed material. It works anywhere that supports the Agent Skills standard. You install it once and it's available as a slash command (`/rust-code-review`) across any compatible tool.

An **agent** is a platform-specific configuration. It defines which model to use (or defers to the user's default), which tools are available, what permissions those tools have, what context loads automatically, and what greeting appears when you switch to it. Agent configs are JSON files that live in `.kiro/agents/` and are Kiro-specific. They're optional — you can use the skills without them — but they make the experience significantly better because you're not setting up context manually every session.

The reason to keep them separate is longevity. The skills are knowledge. They don't change when Kiro updates its agent config schema, when you add a new tool to your workflow, or when you want to use the same review checklist in Claude Code. The agent configs are wiring. They change when the platform changes, when your permissions model evolves, when you want to add a keyboard shortcut or a postToolUse hook. Keeping them in separate directories means you can evolve both independently without tangling them together.

If you want to add Cursor support tomorrow: add `agents/cursor/`. The skills stay untouched.

---

## Installing It: One Command

Because the skills follow the Agent Skills standard, they work with `npx skills` — the community CLI for the open agent skills ecosystem:

```bash
npx skills add huyhoang1001/dev-muscle-memory
```

The CLI clones the repo, finds the skills in `skills/`, detects which agents you have installed, and drops the skill files in the right place. For Kiro, that's `.kiro/skills/`. For Claude Code, it's `.claude/skills/`. It handles the path differences so you don't have to.

```bash
# See what's available before installing
npx skills add huyhoang1001/dev-muscle-memory --list

# Install only what you need
npx skills add huyhoang1001/dev-muscle-memory --skill rust-code-review --skill rust-dev

# Install globally, available in all projects
npx skills add huyhoang1001/dev-muscle-memory -g

# Update when the skills evolve
npx skills update
```

Once installed, type `/rust-code-review` in Kiro and the skill loads. Or, if you've installed the agent configs, switch to the `rust-code-review` agent from the agent picker in the chat input bar — and it comes pre-loaded with the skill, your steering files, and the right tool permissions already set.

---

## Using It in a Real Rust Project

Here's how the setup looks in `heap-sentry`, a Rust library for runtime heap monitoring.

`heap-sentry` has some specific constraints that make it a good test case. The allocator hot path must be lock-free — any `Mutex` in `alloc()` or `dealloc()` is a deadlock waiting to happen. There's a `SafeMutex<T>` wrapper that handles poisoning recovery; using raw `std::sync::Mutex` directly is a convention violation. The codebase is a library crate, so `anyhow` is wrong here and typed errors are required. It has five optional feature flags that must all compile independently.

None of this is in the Rust book. It's project-specific knowledge that would otherwise get re-explained every session or silently violated by an agent that didn't know.

The solution is a steering file at `.kiro/steering/conventions.md`:

```markdown
---
inclusion: manual
---

## Hot path rules
- Any change to allocator.rs that introduces a Mutex, heap allocation,
  or blocking call on alloc/dealloc is an automatic P0
- IN_TRACKING thread-local guard prevents re-entrant tracking
- should_sample_allocation() gates the Mutex-guarded maps — never bypass it

## Conventions
- SafeMutex<T> not raw std::sync::Mutex
- MSRV: 1.75.0
- Library crate — no anyhow, no unwrap() on the hot path
- Feature flags: backtrace, jemalloc, mimalloc, tracing — must all compile independently
```

And the agent configs load both the skill and this steering file:

```json
{
  "name": "rust-code-review",
  "resources": [
    "skill://.kiro/skills/rust-code-review/SKILL.md",
    "file://.kiro/steering/structure.md",
    "file://.kiro/steering/conventions.md"
  ],
  "tools": ["read", "shell"],
  "permissions": {
    "rules": [
      { "capability": "shell", "match": ["git *", "cargo *"], "effect": "allow" },
      { "capability": "shell", "match": ["rm *", "sudo *"], "effect": "deny" }
    ]
  }
}
```

The `inclusion: manual` in the steering file's frontmatter means it doesn't bloat every default session — it only loads when explicitly referenced. The agent config references it, so it loads every time you switch to the `rust-code-review` agent. Default Kiro sessions don't see it.

The review agent gets: the full `rust-code-review` checklist (loaded on demand from the skill), the codebase architecture (from `structure.md`), and the project-specific rules (from `conventions.md`). Shell access is pre-approved for `git *` and `cargo *` so it can run `git diff` and `cargo clippy` without prompting. `rm` and `sudo` are explicitly denied.

Switch to `rust-code-review`, say "review", and it runs `git diff`, reads the changed files, and produces a structured P0–P3 report without you setting up anything.

---

## What It Looks Like in Practice

Here's what a review of a real commit to `heap-sentry` looked like. The commit added backpressure to the allocation tracking system — a new `active_allocation_count` atomic counter, a new `is_at_capacity()` check, and a split of `record_allocation()` into `record_global_allocation()` and `record_sampled_allocation()`.

The review caught:

**P1 — TOCTOU in the backpressure gate.** `is_at_capacity()` reads `active_allocation_count` with `Ordering::Relaxed`, then `should_sample_allocation()` returns `true`, then `store_allocation_metadata()` checks the actual map length under a Mutex. Two separate checks, no coordination. Under concurrent load, the counter and the map can disagree. The fix isn't complicated — document the approximation explicitly and explain that the hard cap is still enforced inside the mutex — but it's exactly the kind of subtle concurrency issue that passes review when the reviewer is tired.

**P2 — `capture_stack_id()` acquires a Mutex outside the `IN_TRACKING` guard in the original code.** Backtraces allocate. `format!("{:?}", bt)` allocates. These allocations recursively re-enter `track_alloc()`. The guard at the top of `track_alloc()` prevents infinite recursion, but only for the sampled path — `record_global_allocation()` is called before the guard. This is actually correct behavior, but it's not documented, which means the next person to touch this code might break it without realizing.

**P2 — `record_allocation()` and `record_deallocation()` are now dead code that double-counts if called.** The refactor split them into global and sampled variants, but left the original wrappers as `pub fn`. Any external caller or test that calls `record_allocation()` will get double-counted metrics. Making them `pub(crate)` or adding a deprecation notice costs nothing.

These are exactly the findings that a competent Rust reviewer would flag and a solid AI assistant without the right context would miss entirely.

---

## What Makes a Good Skill

Writing skills is closer to writing documentation than writing code. The same principles apply.

**Be specific about when to load references.** Don't say "check the checklist." Say "load `references/rust-unsafe.md` for detailed SAFETY comment requirements." The agent needs an explicit signal to pull in more context, and vague instructions produce vague behavior.

**Structure for progressive disclosure.** The `SKILL.md` should be actionable on its own — a developer could read it and know what to do. The `references/` files should be the deep dives, the examples, the edge cases. Keep `SKILL.md` under 500 lines; move the rest to references.

**Tell the agent what NOT to do.** The `rust-code-review` skill has one prominent instruction: "Do NOT implement any changes until the user explicitly confirms." Without this, agents are eager to fix things. Sometimes you want a review, not a rewrite. The skill enforces the workflow.

**Write severity so the agent can triage, not just list.** A flat list of issues isn't useful. A P0/P1/P2/P3 breakdown with explicit "blocks merge" vs "optional improvement" gives the agent a decision framework, not just a checklist.

**Separate knowledge from project context.** The skill contains universal Rust knowledge. Project-specific rules go in steering files. This is what makes the skill reusable — you can install `rust-code-review` in any Rust project and then layer your project's conventions on top via a steering file without modifying the skill itself.

---

## Where This Goes From Here

The current skills cover Rust, but the approach is language-agnostic. The interesting extensions are obvious once you've built the first one.

**More languages.** The same structure — code review + dev assistant + reference files — works for Go, TypeScript, Python. The specific checklists are different (goroutine leaks instead of async hazards, `any` vs `unknown` in TypeScript, GIL considerations in Python) but the architecture is identical.

**Domain-specific skills.** Beyond language, there are domains with hard-won patterns: database schema migrations, API versioning, cryptography, infrastructure-as-code. A `safe-migrations` skill that checks for backward compatibility, column renames that break rolling deploys, missing indexes on new columns. A `secrets-hygiene` skill that knows the difference between a local dev credential and something that shouldn't be in version control.

**Project archetype templates.** A skill that knows how to scaffold a new service that matches your team's conventions — not generic boilerplate, but your conventions. `rust-new-service` that generates a `Cargo.toml` with your standard dependencies pinned, a `src/error.rs` with your error type pattern, a GitHub Actions workflow that matches what you're already running.

**Skills that call tools.** The Agent Skills spec supports a `scripts/` directory for executable code. A skill can instruct the agent to run `cargo semver-checks` before calling a release, or run your test suite with specific flags, or query a JIRA board for the acceptance criteria of the ticket being implemented. Skills become orchestration.

**Cross-agent consistency.** The real payoff is when your entire team installs the same skills. Code reviews use the same criteria. New code follows the same patterns. The conventions encoded in the skills become the conventions the whole team follows, enforced not by a linter but by an agent that understands context.

---

## The Bigger Idea

There's a shift happening in how we think about AI assistants for development. The early framing was: give the AI a problem, get a solution. The current framing is more interesting: give the AI the right context, and it applies your existing expertise at machine speed.

The expertise already exists. Senior engineers have it. Teams have it, encoded in code review comments, in ADRs, in the muscle memory of people who've been burned before. The challenge is making that expertise portable and accessible to the agent.

Skills are one answer to that. Not the only one, but a good one: small, composable, version-controlled, shareable, and built on an open standard that works across the increasingly fragmented landscape of agentic tools.

`dev-muscle-memory` is a starting point. Fork it, adapt it, break it apart and reassemble it for your stack. The goal isn't to have everyone use the same skills — it's for everyone to have *their* skills, not start from scratch in every new session.

The muscle memory you've built up over years of writing Rust shouldn't live only in your head. It should travel with you.

---

*`dev-muscle-memory` is open source at [github.com/huyhoang1001/dev-muscle-memory](https://github.com/huyhoang1001/dev-muscle-memory). Install with `npx skills add huyhoang1001/dev-muscle-memory`. Contributions welcome.*
