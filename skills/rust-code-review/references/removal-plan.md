# Removal and Cleanup Plan Template

## Priority Levels

- **P0**: Remove immediately (unsound, security risk, or blocks other work)
- **P1**: Remove in current PR or next sprint
- **P2**: Backlog — tracked with a follow-up issue

---

## Rust-Specific Dead Code Signals

| Signal | What it means |
|--------|--------------|
| `#[allow(dead_code)]` | Explicitly suppressed warning — is this still needed? |
| `#[cfg(feature = "...")]` block, feature removed from Cargo.toml | Dead code behind gone feature |
| `pub` item with zero external uses (`rg` finds no callers) | Possibly unused public API |
| `todo!()` / `unimplemented!()` in non-test code | Incomplete implementation or planned removal |
| Deprecated `#[deprecated]` items with no migration path noted | Needs removal plan |

---

## Safe to Remove Now

### Item: [Name/Description]

| Field | Details |
|-------|---------|
| **Location** | `src/module.rs:line` |
| **Rationale** | Why this should be removed |
| **Evidence** | `rg "symbol_name"` returns no callers; `#[allow(dead_code)]` present |
| **Impact** | None — no active consumers |
| **Steps** | 1. Remove code  2. Remove tests  3. Remove from `pub use` in `lib.rs`  4. `cargo test` |
| **Verification** | `cargo build --all-targets` passes; no new warnings |

---

## Defer Removal (Plan Required)

### Item: [Name/Description]

| Field | Details |
|-------|---------|
| **Location** | `src/module.rs:line` |
| **Why defer** | Active callers in external crates, needs semver deprecation cycle |
| **Preconditions** | Feature flag off for 2 releases, telemetry shows 0 usage |
| **Breaking changes** | List any public API / trait changes |
| **Migration path** | What callers should use instead; document in `#[deprecated(since = "...", note = "...")]` |
| **Timeline** | Target version or date |
| **Owner** | Person responsible for the removal PR |
| **Validation** | `cargo semver-checks`; check downstream crates |
| **Rollback plan** | Revert the deprecation commit |

---

## Pre-Removal Checklist

- [ ] `rg "symbol_name"` — confirmed no callers in this codebase
- [ ] Checked `pub use` re-exports in `lib.rs` and any facade modules
- [ ] Checked for dynamic usage (trait objects, reflection patterns)
- [ ] Feature flag telemetry reviewed (if applicable)
- [ ] Tests referencing the removed item updated or removed
- [ ] `CHANGELOG.md` updated
- [ ] `#[deprecated]` annotation added one release before hard removal (for public API)
- [ ] `cargo semver-checks` run (for library crates)
