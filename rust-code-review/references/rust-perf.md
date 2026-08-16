# Rust Performance Review Guide

---

## Unnecessary `collect()`

`collect()` materializes an iterator into a heap allocation. If you only need to iterate, sum, or check the result, skip it.

```rust
// ❌ Collects to Vec, then re-iterates — double work
fn bad(items: &[i32]) -> i32 {
    items.iter()
        .filter(|x| **x > 0)
        .collect::<Vec<_>>()   // unnecessary allocation
        .iter()
        .sum()
}

// ✅ Keep the chain lazy
fn good(items: &[i32]) -> i32 {
    items.iter().filter(|x| **x > 0).copied().sum()
}

// ❌ Collect just to check emptiness
fn bad_any(items: &[Item]) -> bool {
    !items.iter().filter(|i| i.is_valid()).collect::<Vec<_>>().is_empty()
}

// ✅ Use Iterator::any
fn good_any(items: &[Item]) -> bool {
    items.iter().any(|i| i.is_valid())
}
```

---

## String Building

```rust
// ❌ + operator reallocates on every concatenation
fn bad(parts: &[&str]) -> String {
    let mut s = String::new();
    for p in parts { s = s + p; }
    s
}

// ✅ join for simple cases
fn good_join(parts: &[&str]) -> String {
    parts.join("")
}

// ✅ with_capacity + push_str when size is known
fn good_capacity(parts: &[&str]) -> String {
    let total: usize = parts.iter().map(|s| s.len()).sum();
    let mut s = String::with_capacity(total);
    for p in parts { s.push_str(p); }
    s
}

// ✅ write! macro for formatted building
use std::fmt::Write;
fn good_write(items: &[u32]) -> String {
    let mut s = String::new();
    for (i, item) in items.iter().enumerate() {
        if i > 0 { s.push(','); }
        write!(s, "{item}").unwrap();  // write! on String is infallible
    }
    s
}
```

---

## `Box<dyn Trait>` in Hot Paths

Dynamic dispatch (`dyn Trait`) prevents inlining and adds an indirection. Flag in hot paths.

```rust
// ❌ Dynamic dispatch — virtual table call, no inlining
fn bad(handler: &dyn Handler) { handler.handle(); }

// ✅ Static dispatch — monomorphized, inlinable
fn good<H: Handler>(handler: &H) { handler.handle(); }

// ✅ impl Trait for return position
fn make_handler() -> impl Handler { ConcreteHandler::new() }
```

`Box<dyn Trait>` is appropriate when:
- Storing heterogeneous types in a collection
- Returning different types from a function that callers don't know at compile time
- When the monomorphization cost would be excessive

---

## Missing `with_capacity`

When the final size of a `Vec`, `String`, or `HashMap` is known or estimable upfront, pre-allocate.

```rust
// ❌ Multiple reallocations as the Vec grows
let mut results = Vec::new();
for item in items { results.push(transform(item)); }

// ✅ Single allocation
let mut results = Vec::with_capacity(items.len());
for item in items { results.push(transform(item)); }

// ✅ Even better: use iterator map + collect (compiler can often optimize)
let results: Vec<_> = items.iter().map(transform).collect();
```

---

## Cloning Large Structs

```rust
// ❌ Passing large struct by value — copied on every call
fn process(data: LargeStruct) { ... }

// ✅ Pass by reference
fn process(data: &LargeStruct) { ... }

// ✅ For shared ownership across threads, use Arc (one allocation, cheap clone)
fn process(data: Arc<LargeStruct>) { ... }
```

---

## Allocation in Hot Paths

Patterns to flag in frequently-called code:

| Pattern | Cheaper Alternative |
|---------|-------------------|
| `format!("...")` for a key/label | `&'static str` or a pre-built string |
| `Vec::new()` per request | Pre-allocated pool or `smallvec` |
| `HashMap::new()` per request | Reuse or use `BTreeMap` for small maps |
| `String::from(literal)` | `&'static str` |
| `.to_owned()` for comparison | Compare with `==` on `&str` directly |

---

## Iterator Adapter Ordering

```rust
// ❌ filter after map — map runs on items that will be filtered
items.iter().map(|x| expensive(x)).filter(|x| x.is_valid())

// ✅ filter first — map only runs on items that pass
items.iter().filter(|x| x.is_valid()).map(|x| expensive(x))
```

---

## Checklist

- [ ] No `collect()` followed immediately by another iteration
- [ ] `Iterator::any`, `all`, `count`, `sum`, `find` used instead of `collect` + check
- [ ] String building uses `join`, `with_capacity + push_str`, or `write!`
- [ ] `Box<dyn Trait>` in hot paths is flagged; generic alternatives considered
- [ ] `Vec`, `String`, `HashMap` pre-allocated with `with_capacity` when size is known
- [ ] Large structs passed by reference or wrapped in `Arc`
- [ ] `filter` placed before `map` in iterator chains
- [ ] No unnecessary `format!` / `to_string()` / `to_owned()` in hot paths
