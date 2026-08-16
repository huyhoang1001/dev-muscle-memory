# dev-muscle-memory

A personal collection of AI agent skills focused on Rust development. Works with [Kiro](https://kiro.dev) and any agent that supports the skill format.

## Skills

### [`rust-code-review`](./rust-code-review/)

Senior-level Rust code review. Covers what the compiler **cannot** catch:

- Unsafe soundness (SAFETY comments, FFI, transmute)
- Async hazards (blocking, Mutex across `.await`, cancellation safety)
- Ownership misuse (unnecessary clones, Arc overuse, Cow opportunities)
- Error handling (thiserror/anyhow split, unwrap in production, error chains)
- Performance (collect(), allocations, Box<dyn> in hot paths)
- API design (idiomatic Rust, #[must_use], #[non_exhaustive], newtype pattern)

### [`rust-dev`](./rust-dev/)

Hands-on Rust development assistant:

- Compiler error triage — explains root cause, not just the fix
- API design guidance — types, error design, builder patterns
- Pattern selection — typestate, state machine, sealed traits, retry, etc.
- Crate recommendations — curated list by use case
- Project structure — workspace layout, feature flags, CI config

### [`repo-explorer`](./repo-explorer/)

Maps any repository's architecture, modules, data flow, and conventions. Produces a structured repo map to navigate an unfamiliar codebase confidently.

---

## Setup in Your Repo

### 1. Import the skills into Kiro

Skills must live in `.kiro/skills/` (project) or `~/.kiro/skills/` (global) to appear as slash commands. The easiest way is to import directly from this repo via the Kiro UI:

1. Open the **Kiro panel** in the IDE sidebar
2. Go to **Agent Steering & Skills**
3. Click **+** → **Import a skill** → **GitHub**
4. Paste the URL for each skill you want:

| Skill | GitHub URL |
|-------|-----------|
| `rust-code-review` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/rust-code-review` |
| `rust-dev` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/rust-dev` |
| `repo-explorer` | `https://github.com/huyhoang1001/dev-muscle-memory/tree/main/repo-explorer` |

Kiro copies the skill into `.kiro/skills/` and it becomes a slash command immediately.

**For global install** (available in all your projects): same flow — Kiro will place it in `~/.kiro/skills/` instead.

**For Kiro CLI**: same import flow, or copy the skill folder directly into `~/.kiro/skills/skill-name/`.

### 2. Invoke

In Kiro chat, type `/` to see available skills, then select:

```
/rust-code-review
```
```
/rust-dev  how should I design this error type?
```
```
/repo-explorer
```

Kiro also auto-activates skills when your request matches the skill description — so asking *"review my latest git changes"* may trigger `rust-code-review` automatically.

### 3. Add a project context steering file (recommended)

The skills work out of the box, but they get significantly better when you give them project-specific context upfront — things like your hot path rules, MSRV, conventions, and module responsibilities that the agent would otherwise have to rediscover every session.

Create `.kiro/steering/skills.md`:

```markdown
---
inclusion: manual
---

# Agent Skills — <your-project>

## `/rust-code-review` conventions

- Crate type: lib / bin
- MSRV: x.xx
- Hot path rules: (e.g. "no allocations in `allocator.rs`")
- Concurrency model: (e.g. "use SafeMutex, not raw Mutex")
- Error handling: (e.g. "library crate — thiserror only, no unwrap on hot path")
- Feature flags: (list optional features that must compile independently)

## `/rust-dev` context

- Where to add new features: (e.g. "new metric → metrics.rs + analysis.rs + lib.rs")
- Patterns to follow: (e.g. "atomic counters for hot path, lazy_static for globals")
- What to avoid: (e.g. "no new global statics without justification")
```

The `inclusion: manual` front matter means this file is only loaded when you explicitly reference it with `#skills` in chat — keeping it out of every prompt but available when you need it.

### 3. Use the skills

In Kiro chat, invoke a skill by name:

```
/rust-code-review
```
```
/rust-dev  how should I design this error type?
```
```
/repo-explorer
```

---

## Going further: Custom Agents

Skills are instructions the agent loads on demand. **Custom agents** go one step further — they pre-wire the skills, project context, tool permissions, and a system prompt into a dedicated agent you switch to by name.

Create `.kiro/agents/rust-code-review.md` in your project:

```markdown
---
name: rust-code-review
description: Senior Rust code reviewer. Reviews git changes for unsafe soundness, async hazards, ownership issues, and performance.
tools:
  - read
  - shell
permissions:
  rules:
    - capability: shell
      match: ["git *", "cargo clippy *", "cargo check *"]
      effect: allow
resources:
  - skill://.kiro/skills/rust-code-review/SKILL.md
  - file://.kiro/steering/skills.md
welcomeMessage: "Ready to review. Say **review** or give me a commit range."
---

You are a senior Rust engineer performing code review.
Add any project-specific rules here — hot path constraints, MSRV, naming conventions, etc.
```

Then switch to the agent in Kiro and it comes pre-loaded with the skill, your project context, and shell access scoped to `git` and `cargo` only — no extra setup per session.

---

## Customizing for Your Project

Each skill's references are plain Markdown files — you can fork this repo and edit them to match your team's conventions. Common things to customize:

- **`rust-code-review/references/rust-unsafe.md`** — add project-specific unsafe invariants
- **`rust-dev/references/rust-crates.md`** — add or remove crates you've standardized on
- **`rust-dev/references/rust-project-structure.md`** — match your workspace layout

---

## License

MIT
