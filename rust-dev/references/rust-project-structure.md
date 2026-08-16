# Rust Project Structure Guide

---

## Single Crate Layout

```
my-crate/
├── Cargo.toml
├── Cargo.lock          # commit for binaries, gitignore for libraries
├── clippy.toml         # custom clippy lints
├── rustfmt.toml        # formatting config
├── src/
│   ├── lib.rs          # public API surface — re-exports and top-level docs
│   ├── error.rs        # error types (thiserror)
│   ├── config.rs       # configuration types
│   └── core/
│       ├── mod.rs
│       └── engine.rs
├── tests/
│   ├── helpers/
│   │   └── mod.rs      # shared test utilities
│   └── integration.rs  # integration tests
├── examples/
│   └── basic.rs        # runnable examples (also serve as docs)
├── benches/
│   └── throughput.rs   # criterion benchmarks
└── docs/
    └── architecture.md
```

---

## Workspace Layout (Multi-Crate)

```
my-workspace/
├── Cargo.toml          # [workspace] manifest
├── Cargo.lock          # single lock file for entire workspace
├── crates/
│   ├── my-core/        # core types, traits, no I/O
│   │   └── Cargo.toml
│   ├── my-storage/     # storage implementations
│   │   └── Cargo.toml
│   ├── my-api/         # HTTP API layer
│   │   └── Cargo.toml
│   └── my-cli/         # CLI binary
│       └── Cargo.toml
└── tests/
    └── e2e/            # end-to-end tests across crates
```

**Workspace Cargo.toml**:
```toml
[workspace]
members = ["crates/*"]
resolver = "2"

[workspace.dependencies]
# Pin shared deps once, inherit with { workspace = true }
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
thiserror = "1"
anyhow = "1"

[workspace.lints.rust]
unsafe_code = "forbid"  # or "warn" if you need unsafe

[workspace.lints.clippy]
all = "warn"
pedantic = "warn"
```

**Member Cargo.toml** (inheriting workspace deps):
```toml
[package]
name = "my-core"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { workspace = true }
serde = { workspace = true }
thiserror = { workspace = true }
```

---

## `lib.rs` Re-export Strategy

```rust
// src/lib.rs — defines the public API surface

// Re-export what users need, hide implementation details
pub use self::client::Client;
pub use self::config::Config;
pub use self::error::{Error, Result};

// Keep internal modules private
mod client;
mod config;
mod error;
mod internal;  // not pub — internal only

// Feature-gated optional modules
#[cfg(feature = "async")]
pub use self::async_client::AsyncClient;
#[cfg(feature = "async")]
mod async_client;
```

---

## Feature Flag Design

```toml
[features]
default = ["std"]

# Core features
std = []                         # std support (disable for no_std)
alloc = []                       # alloc without full std

# Optional integrations
async = ["dep:tokio", "dep:futures"]
tls = ["dep:rustls", "async"]   # tls implies async
serde = ["dep:serde"]

# Full-featured build
full = ["async", "tls", "serde"]

[dependencies]
tokio = { version = "1", optional = true, features = ["rt", "net"] }
rustls = { version = "0.23", optional = true }
serde = { version = "1", optional = true, features = ["derive"] }
```

**In code**:
```rust
#[cfg(feature = "serde")]
use serde::{Deserialize, Serialize};

#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
pub struct Config {
    pub timeout_ms: u64,
}
```

---

## Test Organization

```rust
// src/module.rs — unit tests live alongside the code
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_basic_case() { ... }

    #[test]
    fn test_edge_case_empty() { ... }
}

// tests/integration.rs — integration tests use the public API only
use my_crate::{Client, Config};

#[test]
fn test_full_workflow() { ... }

// tests/helpers/mod.rs — shared test utilities
pub fn build_test_client() -> Client { ... }
pub fn fixture_config() -> Config { ... }
```

**Async tests with tokio**:
```rust
#[tokio::test]
async fn test_async_operation() {
    let result = async_fn().await;
    assert_eq!(result, expected);
}

// Multi-threaded runtime for concurrency tests
#[tokio::test(flavor = "multi_thread", worker_threads = 2)]
async fn test_concurrent_access() { ... }
```

---

## CI Configuration (`.github/workflows/ci.yml`)

```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with:
          components: clippy, rustfmt

      - uses: Swatinem/rust-cache@v2  # cache build artifacts

      - name: fmt
        run: cargo fmt --all --check

      - name: clippy
        run: cargo clippy --all-targets --all-features -- -D warnings

      - name: test
        run: cargo test --all-features

      - name: doc
        run: cargo doc --no-deps --all-features
        env:
          RUSTDOCFLAGS: "-D warnings"
```

---

## `clippy.toml` Recommended Config

```toml
# Enforce SAFETY comments on unsafe blocks
# (requires clippy::undocumented_unsafe_blocks)
msrv = "1.75"  # minimum supported Rust version

# Adjust per project:
# too-many-arguments threshold
too-many-arguments-threshold = 6
```

In `Cargo.toml` or `.cargo/config.toml`:
```toml
[workspace.lints.clippy]
undocumented_unsafe_blocks = "warn"
pedantic = "warn"
# Disable noisy pedantic lints you don't care about:
module_name_repetitions = "allow"
must_use_candidate = "allow"
```
