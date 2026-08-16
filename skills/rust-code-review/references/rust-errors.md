# Rust Error Handling Review Guide

---

## Library vs Application Error Types

This is one of the most common errors in Rust codebases.

```rust
// ❌ Library using anyhow — callers cannot match on error variants
pub fn parse_config(s: &str) -> anyhow::Result<Config> { ... }

// ✅ Library uses thiserror — callers can match, errors are structured
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    #[error("invalid syntax at line {line}: {message}")]
    Syntax { line: usize, message: String },

    #[error("missing required field: {field}")]
    MissingField { field: &'static str },

    #[error(transparent)]
    Io(#[from] std::io::Error),
}

pub fn parse_config(s: &str) -> Result<Config, ConfigError> { ... }

// ✅ Application binary can use anyhow for convenience
fn main() -> anyhow::Result<()> { ... }
```

---

## Preserving Error Context

```rust
// ❌ Original error is discarded — impossible to debug
fn bad() -> Result<()> {
    operation().map_err(|_| anyhow!("operation failed"))?;
    Ok(())
}

// ✅ Use .context() to add context while preserving the original error chain
fn good() -> Result<()> {
    operation().context("failed to initialize storage")?;
    Ok(())
}

// ✅ Use .with_context() for lazy formatting (avoids allocation if no error)
fn good_lazy(path: &Path) -> Result<()> {
    std::fs::read(path)
        .with_context(|| format!("failed to read file: {}", path.display()))?;
    Ok(())
}
```

---

## `unwrap()` and `expect()`

Every `unwrap()`/`expect()` in non-test code is a potential panic in production.

**Acceptable uses:**
- Tests: `assert`, `unwrap` in `#[test]` functions
- Infallible operations with proof: `"127.0.0.1".parse::<IpAddr>().unwrap()` — provably can't fail
- `Mutex::lock().unwrap()` — acceptable only if poisoning is truly impossible

**Flag for replacement:**
- `unwrap()` on I/O, parsing, or any user-influenced data
- `expect()` with a vague message like `"should work"` — the message should explain the invariant

```rust
// ❌ Panic on user input
let port: u16 = args[1].parse().unwrap();

// ✅ Proper error propagation
let port: u16 = args[1].parse().context("invalid port number")?;
```

---

## `#[source]` for Error Chains

```rust
// ❌ Wrapping an error without #[source] — chain is lost in Display
#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("database error: {0}")]
    Database(sqlx::Error),  // not #[source] — chain broken for .source() callers
}

// ✅ Use #[source] (or #[from]) to preserve the error chain
#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("database error")]
    Database(#[source] sqlx::Error),

    #[error("network request failed")]
    Network {
        #[source]
        cause: reqwest::Error,
        url: String,
    },
}
```

---

## `#[must_use]` Enforcement

Flag silently discarded `Result` and `Option` returns:

```rust
// ❌ Result silently dropped — error ignored
conn.execute(query);

// ✅ Explicitly handle or acknowledge
conn.execute(query)?;
// or if intentional:
let _ = conn.execute(query);  // explicitly discarded
```

---

## Error Message Style

- Lowercase, no trailing period: `"failed to open file"` not `"Failed to open file."`
- Actionable context: `"failed to bind to port 8080: address already in use"` not `"bind failed"`
- No internal implementation details exposed to end users

---

## Checklist

- [ ] Library crates use `thiserror` with typed, matchable errors
- [ ] Application crates use `anyhow` + `.context()` / `.with_context()`
- [ ] No `unwrap()`/`expect()` in non-test, non-infallible production code
- [ ] `#[source]` (or `#[from]`) used on error variants that wrap other errors
- [ ] `Result` and `Option` returns are not silently discarded
- [ ] Error messages are lowercase, no period, and include actionable context
- [ ] Error types implement `Display` (via thiserror) and `Debug`
- [ ] `#[must_use]` on public functions returning `Result` or `Option`
