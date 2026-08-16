# Rust Code Review

A senior-level code review skill for Rust codebases. Covers what the compiler **cannot** catch: unsafe soundness, async hazards, cancellation safety, ownership misuse, error handling patterns, performance footguns, and idiomatic API design.

## Usage

```
/rust-code-review
```

Reviews current git changes against the Rust-specific checklist.

## What It Covers

| Area | Examples |
|------|---------|
| **Unsafe soundness** | Missing `SAFETY` comments, unsound transmute, FFI contract violations |
| **Async hazards** | Blocking in async, `Mutex` across `.await`, non-cancel-safe `select!` |
| **Ownership** | Unnecessary `clone()`, `Arc<Mutex<T>>` overuse, `Cow` opportunities |
| **Error handling** | `unwrap()` in production, library `anyhow`, missing error context |
| **Performance** | Needless `collect()`, string building, `Box<dyn>` in hot paths |
| **API design** | Accept `&str` not `String`, `#[must_use]`, `#[non_exhaustive]`, newtype pattern |
| **Dead code** | `#[allow(dead_code)]`, unused pub items, deprecated without plan |

## Severity Levels

| Level | Name | Action |
|-------|------|--------|
| P0 | Critical | Must block merge |
| P1 | High | Should fix before merge |
| P2 | Medium | Fix in PR or follow-up |
| P3 | Low | Optional improvement |

## Workflow

1. **Preflight** — `git diff`, check `Cargo.toml` for new deps
2. **Ownership audit** — clone, Arc, Cow, lifetimes
3. **Unsafe audit** — SAFETY comments, invariants, FFI (highest priority)
4. **Async audit** — blocking, mutex, cancellation, spawn
5. **Error handling** — thiserror/anyhow split, unwrap, context
6. **Performance** — collect, allocation, Box<dyn>
7. **API design** — idiomatic Rust patterns
8. **Output** — structured findings by severity
9. **Confirmation** — wait for user approval before implementing fixes

## Structure

```
rust-code-review/
├── SKILL.md
├── agents/
│   └── agent.yaml
└── references/
    ├── rust-ownership.md     # clone, Arc, Cow, lifetimes
    ├── rust-unsafe.md        # SAFETY comments, FFI, transmute
    ├── rust-async.md         # blocking, mutex, cancellation safety, spawn
    ├── rust-errors.md        # thiserror, anyhow, unwrap, context
    ├── rust-perf.md          # collect, allocation, Box<dyn>
    ├── rust-api-design.md    # idiomatic APIs, must_use, non_exhaustive
    └── removal-plan.md       # dead code, safe delete vs deferred plan
```

## License

MIT
