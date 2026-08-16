# Unsafe Code Review Guide

> Unsafe code is the highest-priority section of any Rust review. The compiler cannot verify soundness here — the reviewer is the last line of defense.

---

## The Golden Rules

1. **Every `unsafe` block must have a `// SAFETY:` comment** explaining *why* it is sound — not what it does.
2. **Every `unsafe fn` must have a `# Safety` doc section** documenting preconditions the caller must uphold.
3. **Minimize the unsafe surface** — the `unsafe` block should wrap only the single operation that requires it.
4. **Document the invariants** — what must be true before, during, and after.
5. **Prefer safe alternatives** — if a safe API exists (`get`, `checked_add`, etc.), use it.

---

## Required Comment Formats

```rust
// ❌ Unsafe with no explanation — instant P0
unsafe fn transmute_bad<T, U>(t: T) -> U {
    std::mem::transmute(t)
}

// ✅ unsafe fn: # Safety doc section
/// Reinterprets the bits of `T` as `U`.
///
/// # Safety
///
/// - `T` and `U` must have identical size and alignment.
/// - The bit pattern of `t` must be a valid value for `U`.
/// - No references to `t` may exist after this call.
unsafe fn transmute_ok<T, U>(t: T) -> U {
    // SAFETY: Caller guarantees size/alignment match and bit validity per doc.
    std::mem::transmute(t)
}

// ✅ unsafe block inside safe fn
fn get_unchecked(slice: &[u8], index: usize) -> u8 {
    debug_assert!(index < slice.len());
    // SAFETY: index < slice.len() verified by debug_assert above.
    // In release builds, callers must guarantee valid index.
    unsafe { *slice.get_unchecked(index) }
}
```

---

## Common Unsafe Patterns — What to Check

### Raw pointer dereference
- Is the pointer non-null? (use `NonNull` where possible)
- Is the pointer properly aligned for `T`?
- Is the pointed-to memory initialized?
- Is the lifetime valid — not dangling?
- Is there any aliasing that violates Rust's aliasing rules?

### `slice::from_raw_parts` / `slice::from_raw_parts_mut`
- Is `data` valid for `len * size_of::<T>()` bytes?
- Is `data` properly aligned for `T`?
- Does the slice live no longer than the backing allocation?
- For `_mut`: no other references to this memory exist simultaneously?

### `std::mem::transmute`
- Are `T` and `U` the same size? (`assert_eq!(size_of::<T>(), size_of::<U>())`)
- Is the bit pattern of the source a valid value for the target type?
- Are there hidden padding bytes that could be uninitialized?
- Consider using `bytemuck` crate for safe, checked byte transmutation.

### `ptr::copy` / `ptr::copy_nonoverlapping`
- Source and destination are valid for the byte count.
- For `_nonoverlapping`: ranges truly do not overlap.
- Source memory is initialized.

### FFI boundaries
- Pointer is valid, aligned, and live for the duration of the call.
- String pointers are null-terminated (for C strings).
- No Rust references alias the C-side data during the call.
- Return values from C are validated before use.

```rust
// ✅ FFI wrapper pattern
extern "C" {
    fn c_process(ptr: *const u8, len: usize) -> i32;
}

pub fn safe_process(data: &[u8]) -> Result<i32, Error> {
    // SAFETY: data.as_ptr() is valid for data.len() bytes (slice invariant).
    // c_process only reads from the buffer and does not retain the pointer.
    let result = unsafe { c_process(data.as_ptr(), data.len()) };
    if result < 0 { Err(Error::from_code(result)) } else { Ok(result) }
}
```

---

## Red Flags — Automatic P0

| Pattern | Why it's a P0 |
|---------|--------------|
| `unsafe` block with no `SAFETY:` comment | Unverifiable soundness |
| `unsafe fn` with no `# Safety` doc | Caller has no contract to uphold |
| `transmute` without size/alignment proof | UB if sizes differ |
| Mutable raw pointer aliasing | Undefined behavior, can corrupt memory |
| `get_unchecked` without a bounds proof | Out-of-bounds access |
| FFI pointer with no lifetime justification | Use-after-free potential |
| `unsafe impl Send` / `unsafe impl Sync` without reasoning | Data race potential |

---

## Checklist

- [ ] Every `unsafe` block has a `// SAFETY:` comment
- [ ] Every `unsafe fn` has a `# Safety` documentation section
- [ ] The unsafe block is as small as possible
- [ ] Invariants are explicitly stated, not implied
- [ ] A safe alternative was considered and rejected with justification
- [ ] Raw pointers: validity, alignment, lifetime, aliasing verified
- [ ] FFI: pointer validity and lifetime at the call boundary verified
- [ ] `transmute`: same size and valid bit pattern proven
- [ ] `unsafe impl Send/Sync`: thread-safety invariants documented
