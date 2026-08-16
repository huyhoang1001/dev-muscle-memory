---
name: rust-code-review
description: "Senior-level code review for Rust codebases. Covers ownership/borrowing, unsafe correctness, async/cancellation safety, error handling, performance, and API design — the things the compiler can't catch."
---

# Rust Code Review

## Overview

Perform a structured review of current git changes with a Rust expert lens. The compiler already handles memory safety — this review focuses on what `rustc` cannot catch: business logic, API design, async hazards, unsafe soundness, performance footguns, and maintainability.

Default to **review-only** output. Do not implement fixes until the user explicitly confirms.

---

## Severity Levels

| Level | Name | Description | Action |
|-------|------|-------------|--------|
| **P0** | Critical | Unsound unsafe, data race, correctness bug, security issue | Must block merge |
| **P1** | High | Logic error, async hazard, API design flaw, significant perf regression | Should fix before merge |
| **P2** | Medium | Code smell, maintainability concern, unnecessary allocation | Fix in this PR or follow-up |
| **P3** | Low | Style, naming, minor clippy-level suggestion | Optional improvement |

---

## Workflow

### 1) Preflight

- Run `git status -sb`, `git diff --stat`, `git diff` to scope the changes.
- Identify crate type: `[lib]`, `[[bin]]`, or both — different review priorities apply.
- Check `Cargo.toml` for new/changed dependencies — flag unpinned or unusual crates.
- Run `cargo clippy -- -D warnings` and `cargo fmt --check` mentally; flag obvious violations.

**Edge cases:**
- **No changes**: Ask if the user wants to review staged changes or a specific commit/branch range.
- **Large diff (>500 lines)**: Summarize by module first, then review in batches by concern area.
- **Mixed concerns**: Group findings by feature area, not just file order.

---

### 2) Ownership & borrowing audit

Load `references/rust-ownership.md`.

Check for:
- **Unnecessary `clone()`**: Every clone should be justified. Flag clones that could be borrows or `Cow`.
- **`Arc<Mutex<T>>` over-use**: Is shared state really necessary? Could the design eliminate it?
- **`RefCell` misuse**: Dynamic borrow checking should be a last resort; flag without justification.
- **Lifetime complexity**: Overly complex lifetimes often signal a design problem upstream.
- **`Cow<'_, T>` opportunities**: Functions that sometimes allocate and sometimes borrow.

---

### 3) Unsafe code audit (highest priority)

Load `references/rust-unsafe.md`.

Every `unsafe` block and `unsafe fn` must be reviewed with maximum scrutiny:

- **`SAFETY` comment required**: Every `unsafe` block must have a `// SAFETY:` comment explaining *why* it is sound.
- **`# Safety` doc section required**: Every `unsafe fn` must document preconditions in `# Safety`.
- **Minimal unsafe surface**: The `unsafe` block should be as small as possible.
- **Invariants documented**: What must the caller guarantee? What does the implementation guarantee?
- **Safe alternative exists?**: Flag if a safe API (`get`, `checked_*`) could replace the unsafe call.
- **FFI boundaries**: Verify pointer validity, alignment, lifetime, and aliasing rules at FFI edges.
- **`transmute` red flag**: Almost always wrong; demand justification and size/alignment proof.

---

### 4) Async & concurrency audit

Load `references/rust-async.md`.

Check for:
- **Blocking in async context**: `std::fs`, `std::net`, `thread::sleep`, CPU-heavy loops without `spawn_blocking`.
- **`std::sync::Mutex` held across `.await`**: Will deadlock or cause subtle starvation; use `tokio::sync::Mutex` or restructure.
- **Cancellation safety in `select!`**: Is every `Future` in `select!` cancel-safe? `read_exact` is not; `read` is.
- **Unnecessary `spawn`**: `tokio::spawn` for a task that's immediately `.await`ed; prefer direct await or `join!`.
- **`JoinHandle` result ignored**: A panicking task silently disappears; always handle `JoinHandle` errors.
- **`'static` requirement for `spawn`**: Data passed to spawned tasks must be `'static`; flag hidden clones or `Arc` wrapping.
- **Structured concurrency preference**: `try_join!` / `join!` preferred over ad-hoc spawning when tasks are scoped.

---

### 5) Error handling audit

Load `references/rust-errors.md`.

Check for:
- **`unwrap()`/`expect()` in non-test, non-prototype code**: Each one is a potential panic; demand justification or replacement.
- **Library crate using `anyhow`**: Libraries should use `thiserror` with typed errors so callers can match.
- **Application crate using bare `?` without context**: Use `.context("...")` or `.with_context(|| ...)` to preserve the error chain.
- **`#[source]` missing**: Error variants wrapping other errors should use `#[source]` to preserve the chain.
- **`must_use` results ignored**: Check that `Result` and `Option` returns aren't silently discarded.
- **Error messages**: Should be lowercase, no trailing period, actionable.

---

### 6) Performance audit

Load `references/rust-perf.md`.

Check for:
- **`collect()` followed immediately by iteration**: Almost always removable; use iterator chains.
- **String building in loops**: Use `String::with_capacity` + `push_str`, `write!`, or `join`.
- **`Box<dyn Trait>` in hot paths**: Static dispatch (`impl Trait` / generics) is faster; flag without justification.
- **`to_string()` / `format!()` for allocation-free operations**: Unnecessary heap allocation.
- **Missing `with_capacity`**: Whenever the final size is known upfront.
- **`HashMap` with `String` keys when `&str` suffices**: Consider `HashMap<&str, _>` or `IndexMap`.
- **Cloning large structs**: Should be behind `Arc` or passed by reference.

---

### 7) API design & idiomatic Rust

Load `references/rust-api-design.md`.

Check for:
- **Accept `&str` not `String`, `&[T]` not `Vec<T>`**: Functions should accept the most general form.
- **Return `impl Trait` not `Box<dyn Trait>`** where possible.
- **Builder pattern correctness**: Consuming vs. `&mut self` builders; prefer consuming for immutable final objects.
- **`Default` trait implementation**: Types with sensible defaults should implement `Default`.
- **`Display` vs `Debug`**: `Display` for user-facing messages, `Debug` for developer/log output.
- **`#[non_exhaustive]`**: Public enums and structs that may grow should be marked non-exhaustive.
- **`#[must_use]`**: Functions returning `Result`, `Option`, or meaningful values should be marked.
- **Trait object safety**: `dyn Trait` requires object-safe traits; check for `Self` returns or generic methods.

---

### 8) Removal candidates

Load `references/removal-plan.md`.

- Identify dead code (`#[allow(dead_code)]` is a red flag), feature-flagged-off paths, deprecated functions.
- Distinguish **safe to remove now** vs **needs migration plan**.

---

### 9) Output format

```markdown
## Rust Code Review Summary

**Files reviewed**: X files, Y lines changed
**Crate type**: lib / bin / workspace
**Overall assessment**: [APPROVE / REQUEST_CHANGES / COMMENT]

---

## Findings

### P0 - Critical
(none or list)

### P1 - High
1. **[src/file.rs:42]** Brief title
   - What the issue is
   - Why it matters (soundness / correctness / hazard)
   - Suggested fix with code example

### P2 - Medium
2. ...

### P3 - Low
...

---

## Removal / Cleanup Plan
(if applicable)

## Additional Suggestions
(non-blocking improvements)
```

**Inline comment format:**
```
::code-comment{file="src/allocator.rs" line="42" severity="P1"}
Holding std::sync::Mutex guard across .await point — will deadlock under contention.
Use tokio::sync::Mutex or drop the guard before awaiting.
::
```

---

### 10) Next steps confirmation

```markdown
---

## Next Steps

Found X issues (P0: _, P1: _, P2: _, P3: _).

**How would you like to proceed?**

1. **Fix all** — I'll implement all suggested fixes
2. **Fix P0/P1 only** — Critical and high priority only
3. **Fix specific items** — Tell me which ones
4. **No changes** — Review complete, no implementation needed

Please choose an option or give specific instructions.
```

**Do NOT implement any changes until the user explicitly confirms.**

---

## Resources

| File | Purpose |
|------|---------|
| `references/rust-ownership.md` | Clone, Arc, Cow, lifetime patterns |
| `references/rust-unsafe.md` | SAFETY comments, FFI, transmute, invariants |
| `references/rust-async.md` | Blocking, Mutex across await, cancellation safety, spawn |
| `references/rust-errors.md` | thiserror, anyhow, context, unwrap, must_use |
| `references/rust-perf.md` | collect, allocations, Box<dyn>, with_capacity |
| `references/rust-api-design.md` | Idiomatic APIs, trait design, must_use, non_exhaustive |
| `references/removal-plan.md` | Dead code, safe delete vs deferred plan |
