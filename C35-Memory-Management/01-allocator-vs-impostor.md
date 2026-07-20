# C35.1 — The Real Allocator vs the Impostor

> **The one-sentence version:** `0x5D29D0` is the real zeroing pool allocator (`__fastcall`, pool handle in
> `ECX`, size vs `0x400` threshold), while `0x6269B0` is a two-line getter stub (`mov eax,0x9205E0; ret`) that
> a naive scan mistakes for `operator new`.

[← Chapter 35 hub](C35-Memory-Management.md) · [Next: C35.2 — Pool allocators & SlotPools →](02-pools-slotpools.md)

---

## The real allocator: `0x5D29D0`

The runtime's real object allocator is **`0x5D29D0`**, and its opening bytes tell the story:

```
53             push ebx
8B 5C 24 08    mov  ebx, [esp+8]      ; the size argument
81 FB 00 04 00 00  cmp ebx, 0x400     ; size-class threshold (small vs large)
56             push esi
57             push edi
8B F9          mov  edi, ecx          ; ecx = pool handle (__fastcall)
76 …           jbe  …                 ; branch by size class
```

So it is a **`__fastcall` zeroing pool allocator**: the **pool handle** arrives in `ECX`, the **size** on the
stack; it compares the size against **`0x400`** to route small vs large allocations, allocates from the pool,
**zeroes** the block, and returns it. The zeroing is why freshly-allocated objects start blank
([C35.4](04-debug-fill.md)), and the `0x400` threshold is a small/large size class split common in pool
allocators.

> ✅ *Verified:* `0x5D29D0` reads a size, compares it to `0x400`, takes a pool handle in `ECX` (`__fastcall`),
> and allocates+zeroes — confirmed from its bytes.

## The impostor: `0x6269B0`

Right beside it in "looks like an allocator" space is **`0x6269B0`**, which a naive scan flags as the allocator
because call sites push a size before calling it — exactly the shape of an `operator new` call. But its bytes
are unambiguous:

```
B8 E0 05 92 00    mov eax, 0x009205E0   ; return a fixed global address
C3                ret                    ; …ignoring the pushed argument entirely
```

It is a **two-line getter stub**: it returns the fixed global `0x9205E0` and **ignores its argument**. It
allocates nothing. It was once mis-read as the allocator precisely because of the call-site coincidence — the
memory-system version of a mis-classified vtable getter ([C34.2](../C34-VTable-Anatomy/02-classifying-slots.md)).

> ✅ *Verified:* `0x6269B0` is exactly `mov eax, 0x009205E0; ret` — a getter returning a fixed global, not an
> allocator.

## The lesson: classify by behaviour

This pair is a compact lesson in reverse-engineering discipline
([Chapter 4](../C4-Byte-Level-Toolcraft/C4-Byte-Level-Toolcraft.md)):

- **Call-site shape lies.** Both functions have call sites that push a size — but only one *uses* it.
- **Read the callee, not the caller.** The impostor's two lines reveal it does nothing with the size; the real
  allocator's code shows it allocating.
- **Getter stubs are traps.** A `mov eax, X; ret` that returns a fixed value is a getter, whatever its callers
  push ([C34.2](../C34-VTable-Anatomy/02-classifying-slots.md)).

So the correct method is: when a function *looks* like a well-known primitive (allocator, constructor), confirm
by reading its body. The archive's symbol map names them `RealAllocator_0x5D29D0` and the impostor accordingly —
a correction worth carrying.

## Why it matters

Getting the allocator right is foundational for reasoning about memory:

- **Object construction** ([C33.3](../C33-Class-Registry-Factories/03-construction.md)) calls the *real*
  allocator (`0x5D29D0`) to get an object's `size` bytes — tracing construction means finding this call.
- **Memory tracing** ([C35.6](06-reading-memory.md)) keys on the real allocation path; keying on the impostor
  finds nothing.
- **Pool handles** ([C35.2](02-pools-slotpools.md)) are the `ECX` argument to `0x5D29D0` — identifying which pool
  an allocation uses starts here.

So `0x5D29D0` is the anchor for all memory RE, and `0x6269B0` is the decoy to avoid.

## RE implications

- **The real allocator is `0x5D29D0`** (`__fastcall`, pool in `ECX`, size vs `0x400`, zeroes) — the memory
  anchor.
- **`0x6269B0` is a getter stub** returning `0x9205E0` — not an allocator; ignore it for allocation tracing.
- **Classify by the callee's body**, not the call-site's pushed args — the core discipline.
- **Trace construction to `0x5D29D0`** to find where objects get their memory and which pool.

---

### Key takeaways

- **`0x5D29D0`** is the real allocator: `__fastcall` zeroing pool allocator (pool handle `ECX`, size vs `0x400`).
- **`0x6269B0`** is a getter stub (`mov eax,0x9205E0; ret`) mis-read as `operator new` — it allocates nothing.
- The lesson: **classify by the callee's behaviour**, not the call-site's pushed size — getter stubs are traps.
- The real allocator is the anchor for construction ([C33.3](../C33-Class-Registry-Factories/03-construction.md))
  and memory tracing; the impostor is the decoy.
- Trace object construction to `0x5D29D0` (and its `ECX` pool handle) to map allocations.

**Continue:** [C35.2 — Pool allocators & SlotPools](02-pools-slotpools.md) · [Chapter 35 hub](C35-Memory-Management.md)
