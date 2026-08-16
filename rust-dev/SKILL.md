---
name: rust-dev
description: "Rust development assistant. Helps design idiomatic APIs, structure crates, choose the right patterns, debug tricky compiler errors, and navigate the Rust ecosystem tooling."
---

# Rust Dev

## Overview

A hands-on Rust development companion. Use this when you're building something and need help choosing the right pattern, understanding a compiler error, designing an API, or picking the right crate. Unlike `rust-code-review`, this skill is **action-oriented** — it helps you write code, not just critique it.

---

## Capabilities

### 1) Compiler error triage

When you hit a compiler error, paste the full `rustc` output and this skill will:
- Explain the root cause clearly (not just repeat the error)
- Identify the underlying ownership/lifetime/type issue
- Propose the idiomatic fix, not just "add `.clone()`"
- Show before/after code

Common errors covered:
- Borrow checker conflicts (`cannot borrow ... as mutable`)
- Lifetime errors (`lifetime may not live long enough`)
- Trait bound failures (`the trait ... is not implemented`)
- Async-related (`future cannot be sent between threads safely`)
- Type inference failures

---

### 2) API design guidance

Ask: *"How should I design this API?"* and get:
- Type signature recommendations (accept `&str` not `String`, etc.)
- Error type design (`thiserror` vs `anyhow`, enum granularity)
- Builder pattern advice (consuming vs `&mut self`)
- Trait design (object safety, blanket impls, sealed traits)
- Visibility and module structure

Load `references/rust-api-design.md` for patterns.

---

### 3) Pattern selection

Ask: *"What's the right pattern for X?"* and get concrete guidance on:

| Use case | Pattern |
|----------|---------|
| Shared config across threads | `Arc<Config>` or `once_cell::sync::Lazy` |
| Optional dependency injection | `Option<Box<dyn Trait>>` or generic parameter |
| State machine | Enum with `impl` per variant, or typestate pattern |
| Async retry logic | `backoff` crate or manual exponential backoff |
| Runtime plugin loading | `libloading` + trait objects |
| Zero-copy parsing | `nom`, `winnow`, or manual `&[u8]` slicing |
| Structured concurrency | `tokio::try_join!`, `FuturesUnordered` |
| Cancellation tokens | `tokio_util::sync::CancellationToken` |

Load `references/rust-patterns.md` for detailed examples.

---

### 4) Crate ecosystem guidance

Ask: *"What crate should I use for X?"* and get curated recommendations with trade-offs.

Load `references/rust-crates.md` for the curated list.

---

### 5) Project structure guidance

Ask: *"How should I structure this crate/workspace?"* and get:
- Workspace layout for multi-crate projects
- When to split into sub-crates (`-core`, `-macros`, `-cli`)
- Feature flag design
- `lib.rs` re-export strategy
- Test organization: unit inline, integration in `tests/`, examples in `examples/`

Load `references/rust-project-structure.md` for patterns.

---

### 6) Tooling & workflow

Ask: *"How do I set up X?"* and get setup guides for:

| Tool | Purpose |
|------|---------|
| `cargo clippy` | Linting — run with `-D warnings` in CI |
| `cargo fmt` | Formatting — enforce via `--check` in CI |
| `cargo nextest` | Faster test runner |
| `cargo deny` | Dependency audit (licenses, advisories) |
| `cargo semver-checks` | Detect accidental breaking changes |
| `cargo expand` | Debug proc macros and derive |
| `cargo flamegraph` / `samply` | CPU profiling |
| `cargo criterion` | Benchmarking with `criterion` |
| `miri` | Run under interpreter to catch UB |
| `cargo llvm-cov` | Code coverage |

---

### 7) Debugging & profiling assistance

- Interpret `cargo flamegraph` or `samply` output
- Suggest `#[inline]`, `#[cold]`, `likely/unlikely` hints for hot paths
- Guide through `RUST_LOG` / `tracing` setup
- Explain how to use `dbg!`, `eprintln!`, and `tracing::instrument`

---

## Workflow

### For compiler errors:
1. Paste the full error output
2. Identify the root cause category
3. Explain the invariant being violated
4. Propose idiomatic fix with code

### For design questions:
1. Understand the use case and constraints
2. Load the relevant reference file
3. Present 1-3 options with trade-offs
4. Recommend the best fit for the stated constraints
5. Show concrete code for the recommended approach

### For "what crate should I use":
1. Clarify async vs sync, `std`-only vs `no_std`, stability requirements
2. Present 2-3 options from `references/rust-crates.md`
3. Give a concrete minimal example with the recommended choice

---

## Output Style

- Lead with the recommendation, not the theory
- Always show working code, not pseudocode
- For trade-off comparisons, use a table
- For multi-step implementations, number the steps
- End with: *"Want me to implement this in your codebase?"*

---

## Resources

| File | Purpose |
|------|---------|
| `references/rust-patterns.md` | Common Rust patterns with full examples |
| `references/rust-crates.md` | Curated crate recommendations by use case |
| `references/rust-project-structure.md` | Workspace, module, and feature flag layout |
