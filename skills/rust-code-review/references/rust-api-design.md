# Rust API Design Review Guide

---

## Accept the Most General Type

```rust
// ❌ Forces caller to allocate a String
fn greet(name: String) { println!("Hello, {name}"); }

// ✅ Accepts both &str and String (and anything that derefs to str)
fn greet(name: &str) { println!("Hello, {name}"); }

// ❌ Forces caller to have a Vec
fn process(items: Vec<u8>) { ... }

// ✅ Accepts slices, Vecs, arrays — anything contiguous
fn process(items: &[u8]) { ... }

// ✅ For paths: accept impl AsRef<Path>
fn read_file(path: impl AsRef<Path>) -> Result<String> {
    std::fs::read_to_string(path.as_ref()).context("failed to read file")
}
```

---

## Return `impl Trait` Over `Box<dyn Trait>`

```rust
// ❌ Heap allocation, dynamic dispatch, less ergonomic
fn make_iter() -> Box<dyn Iterator<Item = u32>> { ... }

// ✅ Zero-cost, inlinable, composable
fn make_iter() -> impl Iterator<Item = u32> { ... }

// Note: Box<dyn Trait> is still correct when:
// - The concrete type varies at runtime (enum dispatch or plugin system)
// - You need to store in a heterogeneous collection
// - The return type is part of a trait (impl Trait not allowed in trait methods before RPITIT)
```

---

## Builder Pattern

```rust
// ✅ Consuming builder (preferred for immutable final objects)
pub struct RequestBuilder {
    url: String,
    timeout: Duration,
    headers: Vec<(String, String)>,
}

impl RequestBuilder {
    pub fn new(url: impl Into<String>) -> Self {
        Self { url: url.into(), timeout: Duration::from_secs(30), headers: vec![] }
    }

    pub fn timeout(mut self, t: Duration) -> Self { self.timeout = t; self }
    pub fn header(mut self, k: impl Into<String>, v: impl Into<String>) -> Self {
        self.headers.push((k.into(), v.into())); self
    }
    pub fn build(self) -> Request { Request { ... } }
}

// ✅ &mut self builder (preferred when builder is reused)
impl ConfigBuilder {
    pub fn set_timeout(&mut self, t: Duration) -> &mut Self { ... }
}
```

**Flag**: mixed consuming/`&mut self` methods on the same builder — pick one style.

---

## `Default` Trait

Types with a sensible zero-state should implement `Default`:

```rust
// ✅ Derive when all fields implement Default
#[derive(Default)]
pub struct Config {
    pub timeout: Duration,      // Duration::default() == Duration::ZERO
    pub retries: u32,           // 0
    pub verbose: bool,          // false
}

// ✅ Custom Default when field defaults aren't zero-values
impl Default for Config {
    fn default() -> Self {
        Self {
            timeout: Duration::from_secs(30),
            retries: 3,
            verbose: false,
        }
    }
}
```

---

## `Display` vs `Debug`

| Trait | Purpose | Who sees it |
|-------|---------|-------------|
| `Display` | Human-readable message | End users, logs shown to operators |
| `Debug` | Developer-readable representation | `{:?}` in test output, internal logs |

```rust
// ❌ Exposing internal structure to users via Display
impl Display for Error {
    fn fmt(&self, f: &mut Formatter<'_>) -> fmt::Result {
        write!(f, "{self:?}")  // leaks internal struct names, field values
    }
}

// ✅ Display for users, Debug for developers
impl Display for ConfigError {
    fn fmt(&self, f: &mut Formatter<'_>) -> fmt::Result {
        match self {
            Self::MissingField { field } => write!(f, "missing required field: {field}"),
            Self::InvalidValue { field, value } =>
                write!(f, "invalid value '{value}' for field '{field}'"),
        }
    }
}
```

---

## `#[must_use]`

```rust
// ✅ Functions whose return value must not be ignored
#[must_use]
pub fn build(self) -> Config { ... }

#[must_use = "this Result must be checked"]
pub fn save(&self) -> Result<(), Error> { ... }

// ✅ On types — any value of this type should not be silently dropped
#[must_use]
pub struct Transaction { ... }
```

---

## `#[non_exhaustive]`

Add `#[non_exhaustive]` to public enums and structs that may grow in future versions. This prevents downstream callers from writing exhaustive match arms that would break on new variants.

```rust
// ✅ External crates cannot write exhaustive matches on this enum
#[non_exhaustive]
pub enum ErrorKind {
    NotFound,
    PermissionDenied,
    Timeout,
    // We can add more variants without breaking semver
}
```

**Flag missing `#[non_exhaustive]`** on public enums in library crates with many variants that are likely to grow.

---

## Trait Object Safety

A trait is object-safe (usable as `dyn Trait`) if:
- No methods return `Self`
- No methods have generic type parameters
- No associated constants (in some cases)

```rust
// ❌ Not object-safe — can't use as dyn Processor
trait Processor {
    fn clone(&self) -> Self;        // returns Self
    fn process<T: Item>(&self, t: T); // generic parameter
}

// ✅ Object-safe
trait Processor {
    fn process(&self, item: &dyn Item);
    fn name(&self) -> &str;
}
```

---

## Newtype Pattern for Type Safety

Prefer newtypes over raw primitives for domain values:

```rust
// ❌ Easy to swap arguments — same type
fn create_user(name: String, email: String) { ... }
create_user(email, name);  // compiles, wrong!

// ✅ Distinct types, compiler catches swaps
pub struct UserName(String);
pub struct Email(String);
fn create_user(name: UserName, email: Email) { ... }
```

---

## Checklist

- [ ] Functions accept `&str` not `String`, `&[T]` not `Vec<T>`, `impl AsRef<Path>` for paths
- [ ] Return `impl Iterator` / `impl Trait` instead of `Box<dyn Trait>` where possible
- [ ] Builder pattern is consistently consuming or `&mut self` — not mixed
- [ ] Types with sensible defaults implement `Default`
- [ ] `Display` shows user-friendly messages; `Debug` shows internal representation
- [ ] `#[must_use]` on functions returning `Result`, `Option`, or meaningful builders
- [ ] `#[non_exhaustive]` on public enums in library crates that may grow
- [ ] Traits intended for `dyn` use are object-safe
- [ ] Domain primitives use newtypes to prevent argument swaps
