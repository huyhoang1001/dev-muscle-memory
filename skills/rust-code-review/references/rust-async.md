# Rust Async & Concurrency Review Guide

---

## Blocking in Async Context

The async runtime uses a small thread pool. Blocking inside an async task starves other tasks.

```rust
// ❌ Blocks the runtime thread — starves other tasks
async fn bad() {
    let data = std::fs::read_to_string("file.txt").unwrap();  // blocking I/O
    std::thread::sleep(Duration::from_secs(1));               // blocking sleep
}

// ✅ Use async equivalents
async fn good() -> Result<String> {
    let data = tokio::fs::read_to_string("file.txt").await?;
    tokio::time::sleep(Duration::from_secs(1)).await;
    Ok(data)
}

// ✅ For unavoidable blocking (CPU-heavy, sync-only libraries)
async fn with_blocking() -> Result<Output> {
    tokio::task::spawn_blocking(|| {
        expensive_cpu_computation()  // runs on dedicated blocking thread pool
    }).await?
}
```

**Flag**: any `std::fs`, `std::net`, `thread::sleep`, heavy loops, or sync-only library calls in async functions.

---

## `std::sync::Mutex` Held Across `.await`

Holding a `std::sync::Mutex` guard across an `.await` point will cause deadlocks or thread starvation, because the guard is not `Send`.

```rust
// ❌ Guard held across .await — will either fail to compile or deadlock
async fn bad(mutex: &std::sync::Mutex<Data>) {
    let guard = mutex.lock().unwrap();
    async_operation().await;  // guard still held here!
    process(&guard);
}

// ✅ Option 1: Drop guard before awaiting
async fn good_scoped(mutex: &std::sync::Mutex<Data>) {
    let data = {
        let guard = mutex.lock().unwrap();
        guard.clone()  // extract what you need, drop guard immediately
    };
    async_operation().await;
    process(&data);
}

// ✅ Option 2: Use tokio::sync::Mutex (designed for async)
async fn good_tokio(mutex: &tokio::sync::Mutex<Data>) {
    let guard = mutex.lock().await;
    async_operation().await;  // OK: tokio Mutex is designed for this
    process(&guard);
}
```

**Guidance**:
- `std::sync::Mutex` → short critical sections, never cross `.await`
- `tokio::sync::Mutex` → when you need to hold across `.await`
- `RwLock` → when reads vastly outnumber writes

---

## Cancellation Safety in `select!`

A `Future` in `select!` is cancelled (dropped) when another branch completes first. If the future was mid-operation, its state is lost.

```rust
// ❌ read_exact is NOT cancel-safe: bytes already read are lost on cancellation
async fn bad(stream: &mut TcpStream) {
    let mut buf = vec![0u8; 1024];
    loop {
        select! {
            result = stream.read_exact(&mut buf) => { handle(&buf); }
            _ = timeout()                         => { println!("timeout"); }
        }
    }
}

// ✅ read IS cancel-safe: unread data stays in the stream
async fn good(stream: &mut TcpStream) {
    let mut buf = vec![0u8; 1024];
    loop {
        select! {
            result = stream.read(&mut buf) => {
                match result {
                    Ok(0)  => break,
                    Ok(n)  => handle(&buf[..n]),
                    Err(e) => return Err(e),
                }
            }
            _ = timeout() => { println!("timeout"); }
        }
    }
}

// ✅ When you need read_exact semantics: wrap in a spawned task
async fn exact_with_cancel(stream: TcpStream) {
    let handle = tokio::spawn(async move {
        let mut buf = vec![0u8; 1024];
        stream.read_exact(&mut buf).await?;  // cancel-safe at the JoinHandle level
        Ok::<_, io::Error>(buf)
    });
    select! {
        result = handle => { /* handle result */ }
        _ = timeout()  => { handle.abort(); }
    }
}
```

**Cancel-safe**: `tokio::io::AsyncReadExt::read`, channel `recv`, `sleep`
**NOT cancel-safe**: `read_exact`, `read_to_end`, `write_all`, most multi-step operations

**Document it**:
```rust
/// # Cancel Safety
/// This method is **not** cancel safe. If cancelled mid-read, partial data is lost.
async fn read_message(stream: &mut TcpStream) -> Result<Message> { ... }
```

---

## `tokio::pin!` for Reused Futures

```rust
// ❌ Creates a new sleep future each iteration — timer resets every loop
async fn bad_loop() {
    loop {
        select! {
            _ = tokio::time::sleep(Duration::from_secs(10)) => break,
            data = receive() => process(data).await,
        }
    }
}

// ✅ Pin the future so it persists across loop iterations
async fn good_loop() {
    let sleep = tokio::time::sleep(Duration::from_secs(10));
    tokio::pin!(sleep);
    loop {
        select! {
            _ = &mut sleep => break,
            data = receive() => process(data).await,
        }
    }
}
```

---

## `spawn` vs Direct `.await`

```rust
// ❌ Unnecessary spawn — adds overhead, loses error context
async fn bad() {
    let handle = tokio::spawn(async { simple_op().await });
    handle.await.unwrap();
}

// ✅ Direct await for sequential operations
async fn good() {
    simple_op().await;
}

// ✅ spawn for true parallelism
async fn parallel() -> Result<(A, B)> {
    let t1 = tokio::spawn(fetch_a());
    let t2 = tokio::spawn(fetch_b());
    Ok((t1.await??, t2.await??))
}

// ✅ Prefer structured concurrency with join!/try_join! when tasks are scoped
async fn structured() -> Result<(A, B, C)> {
    tokio::try_join!(fetch_a(), fetch_b(), fetch_c())
}
```

---

## `JoinHandle` Error Handling

A panicking spawned task silently disappears unless you handle the `JoinHandle`.

```rust
// ❌ Panic is silently swallowed
tokio::spawn(async { risky().await });

// ✅ Handle both task errors and panics
match handle.await {
    Ok(Ok(result))  => process(result),
    Ok(Err(e))      => error!("task error: {e}"),
    Err(join_err) if join_err.is_panic() => error!("task panicked: {join_err:?}"),
    Err(join_err)   => error!("task cancelled: {join_err}"),
}
```

---

## Channel Usage

- **`mpsc`**: Multiple producers, single consumer. Use `try_send` for non-blocking; size the buffer appropriately.
- **`oneshot`**: Single request/response pairs.
- **`broadcast`**: Fan-out to multiple consumers. Check receiver lag — a slow consumer causes channel overflow.
- **`watch`**: Latest-value semantics. Good for config or state propagation.

**Flag**: Unbounded channels (`channel()` with no bound) in hot paths — potential memory exhaustion.

---

## Checklist

- [ ] No blocking I/O or `thread::sleep` in async functions
- [ ] `std::sync::Mutex` not held across `.await`
- [ ] All `Future`s in `select!` are cancel-safe, or non-cancel-safe ones are documented
- [ ] `tokio::pin!` used for futures that must persist across loop iterations
- [ ] `spawn` is used for true parallelism, not as a reflex
- [ ] All `JoinHandle` results are handled (not ignored)
- [ ] Spawned tasks satisfy `'static` bound — no hidden borrowed data
- [ ] `try_join!` / `join!` preferred for scoped concurrent tasks
- [ ] Channel buffer sizes are bounded and reasoned about
- [ ] Async functions that are not cancel-safe are documented as such
