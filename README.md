# dev-muscle-memory

A personal collection of AI agent skills focused on Rust development. Designed to work with any agent that supports the skill format (Kiro, Claude, etc.).

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

## Installation

Clone and reference locally, or install via your agent's skill mechanism:

```bash
git clone https://github.com/hoangvo/dev-muscle-memory
```

## License

MIT
