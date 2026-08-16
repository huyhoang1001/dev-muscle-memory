# Rust Ownership & Borrowing Patterns

## Unnecessary `clone()`

`clone()` is often "Rust duct tape" — used to silence the borrow checker rather than fix the real design issue.

```rust
// ❌ Cloning to avoid borrow — ask: is this really needed?
fn process(data: &Data) -> Result<()> {
    let owned = data.clone();
    expensive_operation(owned)
}

// ✅ Pass by reference if the callee only reads
fn process(data: &Data) -> Result<()> {
    expensive_operation(data)
}

// ✅ If clone IS needed, document why
fn spawn_task(data: &Data) {
    // Clone needed: data must be 'static to move into spawned task
    let owned = data.clone();
    tokio::spawn(async move { process(owned).await });
}
```

**Review question**: "Can this be a borrow instead of a clone?"

---

## `Arc<Mutex<T>>` Over-use

Shared mutable state via `Arc<Mutex<T>>` is often a sign of an unexamined design, not a necessity.

```rust
// ❌ Is this shared state really required?
struct Service {
    cache: Arc<Mutex<HashMap<String, Data>>>,
}

// ✅ If only one owner — no Arc needed
struct Service {
    cache: HashMap<String, Data>,
}

// ✅ If concurrent reads only — RwLock or lock-free structure
struct Service {
    cache: Arc<RwLock<HashMap<String, Data>>>,
}

// ✅ For high-concurrency maps — DashMap avoids global lock
use dashmap::DashMap;
struct Service {
    cache: Arc<DashMap<String, Data>>,
}
```

**Review questions**:
- "Does this state need to be shared, or can ownership be restructured?"
- "Are writes rare? Consider `RwLock` or `DashMap`."
- "Is `Mutex` held across `.await`? That's an async hazard — see rust-async.md."

---

## `RefCell` Usage

`RefCell` enables interior mutability with runtime borrow checking. It's a valid tool but should be deliberate.

**Flag when:**
- Used in multi-threaded context (use `Mutex` instead)
- Used to work around a design problem that could be solved structurally
- Not documented why static borrowing doesn't work

---

## `Cow<'_, T>` Opportunities

`Cow` (Clone-on-Write) avoids allocation when a function sometimes returns borrowed data and sometimes owned.

```rust
use std::borrow::Cow;

// ❌ Always allocates, even when input is already valid
fn normalize(name: &str) -> String {
    if name.is_empty() { "Unknown".to_string() } else { name.to_string() }
}

// ✅ Allocates only when mutation is needed
fn normalize(name: &str) -> Cow<'_, str> {
    if name.is_empty() {
        Cow::Borrowed("Unknown")
    } else if name.chars().any(|c| c.is_uppercase()) {
        Cow::Owned(name.to_lowercase())
    } else {
        Cow::Borrowed(name)
    }
}
```

---

## Lifetime Complexity

Complex lifetime annotations are often a symptom of a structural design issue.

**Flag when:**
- More than 2 lifetime parameters on a single function
- Lifetime bounds cascade through multiple layers
- `'static` bounds used to work around borrowing difficulties

**Suggest**: Consider restructuring data ownership, using `Arc`, or splitting the function.

---

## Checklist

- [ ] Every `clone()` is justified or replaced with a borrow
- [ ] `Arc<Mutex<T>>` is truly needed for shared state
- [ ] `RefCell` is documented and thread-safe alternatives considered
- [ ] `Cow` used where functions sometimes allocate, sometimes borrow
- [ ] Lifetime annotations are minimal and necessary
