# C35.4 — Debug-Fill Sentinels

> **The one-sentence version:** the allocator zeroes fresh memory and the free path writes recognisable fill
> bytes into released memory, so a block's contents fingerprint its state — freshly allocated (zeroed), freed
> (filled), or a live global — which lets you map a memory dump.

[← C35.3 — Pre-sized pools](03-presized-pools.md) · [Chapter 35 hub](C35-Memory-Management.md) ·
[Next: C35.5 — Allocation in the object lifecycle →](05-lifecycle.md)

---

## Memory carries its state in its bytes

The allocation system marks memory in ways you can read back:

- **Freshly allocated memory is zeroed** — the real allocator (`0x5D29D0`,
  [C35.1](01-allocator-vs-impostor.md)) zeroes the block it returns, so a just-allocated, not-yet-constructed
  object is all zeros.
- **Freed memory is filled** — the free path writes a **debug-fill sentinel** (a recognisable byte pattern) into
  released blocks, so freed memory is distinguishable from live memory.
- **Live memory holds real values** — constructed, in-use objects carry their fields' actual values.

So a region's byte pattern reveals its **state**: zeros (fresh), fill sentinel (freed), or meaningful data
(live). This is a gift for reverse-engineering a memory dump.

## Fingerprinting a dump

Because state is visible in the bytes, you can **fingerprint** regions of memory ([C35.6](06-reading-memory.md)):

```
region is all 0x00        → freshly allocated, not yet constructed
region is all <fill byte> → freed / uninitialised
region has a vtable ptr + varied data → a live object (identify by vtable, C34.3)
region at a known address with a fixed value → a global
```

- A block starting with a **vtable pointer** ([C34.1](../C34-VTable-Anatomy/01-what-is-a-vtable.md)) and varied
  fields is a **live object** — identify its class by the vtable ([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)).
- A block of **fill bytes** is **dead** memory — freed or never used.
- **Globals** sit at fixed addresses (like the `0x9205E0` the impostor returns,
  [C35.1](01-allocator-vs-impostor.md), or the family list-heads,
  [C32.3](../C32-Runtime-Class-System/03-eleven-families.md)) with recognisable contents.

So a memory dump isn't opaque — its patterns partition it into fresh, freed, live, and global, guiding your
analysis to the live objects that matter.

## The debug-fill byte

Debug allocators conventionally fill freed memory with a distinctive byte (values like `0xCC`, `0xCD`, `0xDD`,
`0xFE` are common in MSVC-family runtimes) so that:

- **Use-after-free is obvious** — code reading freed memory sees the fill pattern, not stale-but-plausible data,
  making the bug detectable.
- **Uninitialised reads are obvious** — reading before construction sees the fill/zeros.
- **Memory state is legible** — the fill byte marks dead memory in a dump.

The presence of these sentinels ([the archive notes debug-fill sentinels that fingerprint globals](../RE-Data-And-Discoveries))
is what makes memory RE tractable: you can tell what a region *is* from what it *contains*.

> 🟡 *Reasoned:* the specific fill byte(s) are the runtime's convention; the verified facts are that the real
> allocator **zeroes** fresh memory ([C35.1](01-allocator-vs-impostor.md)) and the archive documents debug-fill
> sentinels usable to fingerprint globals. The *technique* (byte pattern → memory state) follows from these.

## Sentinels as anchors

The fill patterns and zeroing double as **anchors** for locating structures:

- **Find live objects** by scanning for vtable pointers amid non-fill data
  ([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)).
- **Find globals** at known addresses with stable contents (list-heads, singletons like the StreamMgr at
  `0x91A098`, [Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
- **Bound live regions** by the fill-byte gaps between them.

So the sentinels are not just debugging aids — they're a map legend for the heap, telling you which regions to
read and which to skip.

## RE implications

- **Zeros = fresh, fill = dead, data = live** — read a region's state from its bytes.
- **Fingerprint a dump** into fresh/freed/live/global to guide analysis ([C35.6](06-reading-memory.md)).
- **Live objects** start with a vtable pointer ([C34.3](../C34-VTable-Anatomy/03-identify-by-vtable.md)) — the
  targets worth reading.
- **Fill sentinels anchor** the heap — find live objects and globals, skip dead regions.

---

### Key takeaways

- Fresh memory is **zeroed** (real allocator); freed memory is **fill-filled** (debug sentinel); live memory
  holds real values.
- A region's byte pattern **fingerprints its state** — zeros (fresh), fill (dead), vtable+data (live), stable
  global.
- Debug-fill bytes make **use-after-free and uninitialised reads obvious**, and mark dead memory in a dump.
- The sentinels/zeroing are **anchors** — find live objects (vtable ptrs) and globals, skip filled regions.
- Use the byte-pattern → state mapping to map a memory dump before deep analysis.

**Continue:** [C35.5 — Allocation in the object lifecycle](05-lifecycle.md) · [Chapter 35 hub](C35-Memory-Management.md)
