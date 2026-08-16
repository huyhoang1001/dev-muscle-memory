# Rust Dev

A hands-on Rust development assistant. Helps you write idiomatic code, debug compiler errors, design APIs, pick the right crates, and structure your project — action-oriented, not just review.

## Usage

```
/rust-dev
```

Then describe what you're building or paste the error you're hitting.

## Capabilities

| Task | Example prompt |
|------|---------------|
| **Compiler error triage** | "Getting `cannot borrow as mutable` — here's the error..." |
| **API design** | "How should I design this error type for a library?" |
| **Pattern selection** | "What's the right pattern for a state machine in Rust?" |
| **Crate recommendations** | "What should I use for async HTTP requests?" |
| **Project structure** | "How should I split this into a workspace?" |
| **Tooling setup** | "How do I set up `cargo-nextest` and coverage?" |
| **Debugging** | "How do I use `tracing` to instrument this async function?" |

## Structure

```
rust-dev/
├── SKILL.md
├── agents/
│   └── agent.yaml
└── references/
    ├── rust-patterns.md          # typestate, newtype, sealed trait, retry, etc.
    ├── rust-crates.md            # curated crate list by use case
    └── rust-project-structure.md # workspace layout, features, CI config
```

## License

MIT
