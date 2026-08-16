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

### 1. Declare the skills (`skills-lock.json`)

Create a `skills-lock.json` at the root of your repo. List whichever skills you want:

```json
{
  "skills": [
    {
      "name": "rust-code-review",
      "source": "huyhoang1001/dev-muscle-memory",
      "path": "rust-code-review",
      "description": "Senior Rust code review: unsafe soundness, async hazards, ownership, error handling, performance, API design."
    },
    {
      "name": "rust-dev",
      "source": "huyhoang1001/dev-muscle-memory",
      "path": "rust-dev",
      "description": "Hands-on Rust assistant: compiler errors, API design, pattern selection, crate recommendations, tooling setup."
    },
    {
      "name": "repo-explorer",
      "source": "huyhoang1001/dev-muscle-memory",
      "path": "repo-explorer",
      "description": "Map any repository: architecture, modules, data flow, dependencies, and conventions."
    }
  ]
}
```

### 2. Add a project context steering file (recommended)

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

To load your project context alongside a skill:

```
#skills /rust-code-review
```

---

## Customizing for Your Project

Each skill's references are plain Markdown files — you can fork this repo and edit them to match your team's conventions. Common things to customize:

- **`rust-code-review/references/rust-unsafe.md`** — add project-specific unsafe invariants
- **`rust-dev/references/rust-crates.md`** — add or remove crates you've standardized on
- **`rust-dev/references/rust-project-structure.md`** — match your workspace layout

---

## License

MIT
