# Rust Patterns Reference

## Typestate Pattern

Encode state in the type system so invalid transitions are compile-time errors.

```rust
// States as zero-sized types
struct Disconnected;
struct Connected;
struct Authenticated;

struct Client<State> {
    inner: InnerClient,
    _state: std::marker::PhantomData<State>,
}

impl Client<Disconnected> {
    pub fn new() -> Self { ... }
    pub fn connect(self, addr: &str) -> Result<Client<Connected>> { ... }
}

impl Client<Connected> {
    pub fn authenticate(self, token: &str) -> Result<Client<Authenticated>> { ... }
}

impl Client<Authenticated> {
    pub fn send(&self, msg: &str) -> Result<()> { ... }
}

// send() is unreachable unless connect() and authenticate() were called — enforced by the compiler
```

---

## Newtype for Domain Safety

```rust
pub struct UserId(u64);
pub struct PostId(u64);

// Compiler rejects: fn get_post(id: UserId) called with PostId
fn get_post(id: PostId) -> Option<Post> { ... }

// Add useful impls without boilerplate
impl UserId {
    pub fn new(id: u64) -> Self { Self(id) }
    pub fn value(&self) -> u64 { self.0 }
}

impl std::fmt::Display for UserId {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "user:{}", self.0)
    }
}
```

---

## Sealed Trait (Prevent External Implementations)

```rust
mod private {
    pub trait Sealed {}
}

pub trait MyTrait: private::Sealed {
    fn do_thing(&self);
}

// Only types in this crate can implement MyTrait
impl private::Sealed for MyType {}
impl MyTrait for MyType {
    fn do_thing(&self) { ... }
}
```

---

## Extension Trait

Add methods to types you don't own:

```rust
pub trait ResultExt<T> {
    fn log_err(self, msg: &str) -> Option<T>;
}

impl<T, E: std::fmt::Display> ResultExt<T> for Result<T, E> {
    fn log_err(self, msg: &str) -> Option<T> {
        self.map_err(|e| tracing::error!("{msg}: {e}")).ok()
    }
}

// Usage
let value = fallible_op().log_err("operation failed")?;
```

---

## `once_cell` / `std::sync::OnceLock` for Lazy Statics

```rust
use std::sync::OnceLock;

static CONFIG: OnceLock<AppConfig> = OnceLock::new();

fn get_config() -> &'static AppConfig {
    CONFIG.get_or_init(|| AppConfig::load_from_env())
}

// For computed values in std (1.70+):
static COMPILED_REGEX: OnceLock<regex::Regex> = OnceLock::new();

fn get_regex() -> &'static regex::Regex {
    COMPILED_REGEX.get_or_init(|| regex::Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap())
}
```

---

## State Machine with Enum

```rust
#[derive(Debug)]
enum ConnectionState {
    Idle,
    Connecting { attempt: u32 },
    Connected { session_id: String },
    Failed { reason: String },
}

impl ConnectionState {
    fn transition(self, event: Event) -> Self {
        match (self, event) {
            (Self::Idle, Event::Connect) => Self::Connecting { attempt: 1 },
            (Self::Connecting { attempt }, Event::Retry) if attempt < 3 =>
                Self::Connecting { attempt: attempt + 1 },
            (Self::Connecting { .. }, Event::Success(id)) =>
                Self::Connected { session_id: id },
            (Self::Connecting { .. }, Event::Failure(reason)) =>
                Self::Failed { reason },
            (state, _) => state,  // Invalid transitions are no-ops
        }
    }
}
```

---

## Cancellation Token

```rust
use tokio_util::sync::CancellationToken;

async fn run_with_cancellation() {
    let token = CancellationToken::new();
    let child_token = token.child_token();

    let worker = tokio::spawn(async move {
        loop {
            tokio::select! {
                _ = child_token.cancelled() => {
                    tracing::info!("worker shutting down");
                    break;
                }
                result = do_work() => {
                    handle(result).await;
                }
            }
        }
    });

    // Later: signal shutdown
    token.cancel();
    worker.await.unwrap();
}
```

---

## Structured Concurrency with `FuturesUnordered`

```rust
use futures::stream::{FuturesUnordered, StreamExt};

async fn process_all(items: Vec<Item>) -> Vec<Result<Output>> {
    let mut futures: FuturesUnordered<_> = items
        .into_iter()
        .map(|item| process_item(item))
        .collect();

    let mut results = Vec::new();
    while let Some(result) = futures.next().await {
        results.push(result);
    }
    results
}
```

---

## Zero-Copy Parsing with `nom` / `winnow`

```rust
use winnow::{PResult, Parser, token::take_while, ascii::digit1};

fn parse_header(input: &mut &str) -> PResult<(&str, u32)> {
    let name = take_while(1.., |c: char| c.is_alphanumeric() || c == '-').parse_next(input)?;
    ":".parse_next(input)?;
    " ".parse_next(input)?;
    let value: u32 = digit1.parse_to().parse_next(input)?;
    Ok((name, value))
}
```

---

## Retry with Exponential Backoff

```rust
use std::time::Duration;

async fn with_retry<F, Fut, T, E>(
    max_attempts: u32,
    mut f: F,
) -> Result<T, E>
where
    F: FnMut() -> Fut,
    Fut: std::future::Future<Output = Result<T, E>>,
    E: std::fmt::Display,
{
    let mut delay = Duration::from_millis(100);
    for attempt in 1..=max_attempts {
        match f().await {
            Ok(val) => return Ok(val),
            Err(e) if attempt == max_attempts => return Err(e),
            Err(e) => {
                tracing::warn!("attempt {attempt}/{max_attempts} failed: {e}");
                tokio::time::sleep(delay).await;
                delay = (delay * 2).min(Duration::from_secs(30));
            }
        }
    }
    unreachable!()
}
```
