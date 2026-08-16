# Curated Rust Crate Recommendations

> Focused on actively maintained, widely used crates. Versions noted as of mid-2025 — check crates.io for latest.

---

## Async Runtime

| Crate | Use when |
|-------|----------|
| `tokio` | Default choice for most async applications; rich ecosystem |
| `async-std` | Mirrors std API; smaller ecosystem |
| `smol` | Lightweight, `no_std`-compatible option |

**Recommendation**: Use `tokio` unless you have a specific reason not to.

---

## Error Handling

| Crate | Use when |
|-------|----------|
| `thiserror` | Library crates — typed, matchable errors |
| `anyhow` | Application binaries — convenience, context chains |
| `miette` | CLI tools — pretty error output with source spans |
| `color-eyre` | Applications wanting rich backtraces and pretty output |

---

## Serialization

| Crate | Use when |
|-------|----------|
| `serde` + `serde_json` | JSON — the default choice |
| `serde` + `toml` | Config files |
| `serde` + `serde_yaml` | YAML (use `serde_yml` — maintained fork) |
| `bincode` | Binary format, same-system communication |
| `postcard` | Binary format, `no_std`, embedded/network |
| `rkyv` | Zero-copy deserialization for large data |
| `prost` | Protocol Buffers |

---

## HTTP

| Crate | Use when |
|-------|----------|
| `reqwest` | HTTP client — feature-rich, async, batteries included |
| `hyper` | Low-level HTTP, building frameworks |
| `axum` | HTTP server — ergonomic, tower-based |
| `actix-web` | HTTP server — high performance, mature |
| `warp` | HTTP server — filter-based composition |
| `tower` | Middleware abstraction layer |
| `tower-http` | Common HTTP middleware (tracing, CORS, compression) |

---

## Database

| Crate | Use when |
|-------|----------|
| `sqlx` | Async SQL with compile-time query checking (PostgreSQL, MySQL, SQLite) |
| `diesel` | Sync ORM with strong type guarantees |
| `sea-orm` | Async ORM built on `sqlx` |
| `rusqlite` | SQLite, sync |
| `redis` | Redis client (async and sync) |
| `mongodb` | MongoDB official async driver |

---

## Tracing & Observability

| Crate | Use when |
|-------|----------|
| `tracing` | Structured, async-aware logging — prefer over `log` |
| `tracing-subscriber` | Configure tracing output (stdout, JSON) |
| `tracing-opentelemetry` | Export traces to OpenTelemetry collectors |
| `metrics` | Application metrics facade |
| `opentelemetry` | OpenTelemetry SDK |

**Minimal setup**:
```rust
// Cargo.toml
// tracing = "0.1"
// tracing-subscriber = { version = "0.3", features = ["env-filter"] }

fn main() {
    tracing_subscriber::fmt()
        .with_env_filter(tracing_subscriber::EnvFilter::from_default_env())
        .init();
}
```

---

## CLI

| Crate | Use when |
|-------|----------|
| `clap` | Full-featured CLI arg parsing (derive API recommended) |
| `argh` | Lightweight, Google-style |
| `indicatif` | Progress bars and spinners |
| `console` | Terminal colors and styles |
| `dialoguer` | Interactive prompts |
| `tui-rs` / `ratatui` | Terminal UI apps |

---

## Concurrency & Parallelism

| Crate | Use when |
|-------|----------|
| `rayon` | Data parallelism — parallel iterators |
| `crossbeam` | Lock-free data structures, scoped threads, channels |
| `dashmap` | Concurrent `HashMap` — better than `Mutex<HashMap>` for high contention |
| `tokio-util` | `CancellationToken`, codec utilities, `TaskTracker` |
| `futures` | Extra async combinators (`FuturesUnordered`, `StreamExt`) |

---

## Testing

| Crate | Use when |
|-------|----------|
| `tokio::test` | Async test macro |
| `mockall` | Auto-generate mock implementations of traits |
| `wiremock` | HTTP mock server for integration tests |
| `proptest` | Property-based testing |
| `rstest` | Parameterized tests and fixtures |
| `insta` | Snapshot testing |
| `criterion` | Benchmarks with statistical analysis |
| `cargo-nextest` | Faster test runner (install as cargo subcommand) |

---

## Parsing

| Crate | Use when |
|-------|----------|
| `winnow` | Parser combinators — modern, zero-copy, excellent errors |
| `nom` | Parser combinators — established, huge ecosystem |
| `pest` | PEG grammar files |
| `logos` | Lexer generator via derive macros |
| `chumsky` | Recursive descent with great error recovery |

---

## Utilities

| Crate | Use when |
|-------|----------|
| `once_cell` | Lazy statics (pre-`std::sync::OnceLock` compatibility) |
| `itertools` | Extra iterator adapters |
| `indexmap` | `HashMap` that preserves insertion order |
| `smallvec` | Stack-allocated Vec for small collections |
| `bytes` | Efficient byte buffer management (used by tokio/hyper) |
| `uuid` | UUID generation and parsing |
| `chrono` / `time` | Date/time handling (`time` is more `no_std` friendly) |
| `regex` | Regular expressions |
| `bytemuck` | Safe byte reinterpretation (alternative to `transmute`) |

---

## Security

| Crate | Use when |
|-------|----------|
| `ring` | Crypto primitives — well-audited |
| `rustls` | TLS — pure Rust, no OpenSSL |
| `argon2` | Password hashing |
| `jsonwebtoken` | JWT encoding/decoding |
| `secrecy` | Zero-on-drop secret wrappers |
| `zeroize` | Securely zero memory containing secrets |
