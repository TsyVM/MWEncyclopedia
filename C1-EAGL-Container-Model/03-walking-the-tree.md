# C1.3 — Walking the Tree: Iteration, Recursion & Safe Readers

> **The one-sentence version:** the whole engine's file layer reduces to two loops — a flat sibling
> walk and a recursive descent — plus a bounds-checked reader that never trusts a length; get those
> three right once and you can open every file in the game.

[← Chapter 1 hub](C1-EAGL-Container-Model.md) · [C1.2 — Chunk header & sizes](02-chunk-header-and-sizes.md) ·
[Next: C1.4 — Alignment & padding →](04-alignment-and-padding.md)

---

## What it is

"Walking the tree" is the act of visiting chunks in order. There are exactly two shapes of walk:

1. **Flat / sibling walk** — visit the chunks at one level, stepping `8 + size` between them.
2. **Recursive / tree walk** — the flat walk, plus: whenever the container bit is set, descend into the
   payload and flat-walk *that*, and so on to the leaves.

Everything you will ever do to a file — dumping, searching, extracting, patching — is one of these two
walks with a different `visit()` body. Write them once, correctly, and never again.

## The flat walk

```python
import struct

def walk(buf):
    """Yield (id, payload_offset, size) for each top-level chunk in buf."""
    off = 0
    while off + 8 <= len(buf):
        cid, size = struct.unpack_from('<II', buf, off)
        if off + 8 + size > len(buf):
            break                         # truncated or not a chunk boundary
        yield cid, off + 8, size
        off += 8 + size
```

Two invariants make this safe. First, the loop condition `off + 8 <= len(buf)` guarantees there are
always eight bytes to read a header from. Second, the `off + 8 + size > len(buf)` check guarantees the
payload actually fits before you trust `size`. Drop either check and a malformed file walks you off the
end of the buffer.

## The recursive walk

```python
def walk_tree(buf, base=0, depth=0):
    """Yield (depth, absolute_offset, id, size, payload_memoryview) for every chunk, nested."""
    off = 0
    while off + 8 <= len(buf):
        cid, size = struct.unpack_from('<II', buf, off)
        if off + 8 + size > len(buf):
            break
        payload = memoryview(buf)[off + 8: off + 8 + size]
        yield depth, base + off, cid, size, payload
        if cid & 0x80000000:                      # container → recurse
            yield from walk_tree(payload, base + off + 8, depth + 1)
        off += 8 + size
```

The `base` parameter is the piece people forget, and the piece that matters most for editing: it
accumulates the **absolute file offset** of each chunk. When you later patch a byte, you need to know
exactly where it lives on disk, and `base + off` gives you that even ten levels deep. Without it you
know a chunk's offset *within its parent* but not *within the file*, which is useless for a writer.

The C equivalent, using an explicit stack instead of recursion so a pathological file can't blow your
call stack:

```c
typedef struct { const uint8_t* p; size_t len, off; uint32_t base; int depth; } Frame;

void walk_tree(const uint8_t* data, size_t len) {
    Frame stack[64]; int sp = 0;
    stack[sp++] = (Frame){ data, len, 0, 0, 0 };
    while (sp) {
        Frame* f = &stack[sp - 1];
        if (f->off + 8 > f->len) { sp--; continue; }
        uint32_t id, size;
        memcpy(&id,   f->p + f->off,     4);
        memcpy(&size, f->p + f->off + 4, 4);
        if (f->off + 8 + (size_t)size > f->len) { sp--; continue; }
        visit(f->depth, f->base + (uint32_t)f->off, id, f->p + f->off + 8, size);
        const uint8_t* payload = f->p + f->off + 8;
        uint32_t child_base = f->base + (uint32_t)f->off + 8;
        int      child_depth = f->depth + 1;
        size_t   psize = size;
        f->off += 8 + (size_t)size;              // advance THIS level first
        if ((id & 0x80000000u) && sp < 64)       // then descend
            stack[sp++] = (Frame){ payload, psize, 0, child_base, child_depth };
    }
}
```

Note the ordering: advance the current frame's cursor *before* pushing the child frame, so that when
the child frame pops you resume at the correct sibling. Getting that order wrong is a classic
re-descend-forever bug.

## The dumper — your first instrument

Before writing any format-specific parser, print the tree:

```python
for depth, abs_off, cid, size, _ in walk_tree(open(path, 'rb').read()):
    print(f"{'  ' * depth}0x{cid:08X}  size={size:<8}  @0x{abs_off:X}")
```

Run against an unknown file, this tells you what subsystem you're looking at — match the ids against
[the master table](../Glossary/chunk-ids.md). It is the most useful twelve lines of code in your
toolbox, and [C1.9](09-universal-opener.md) wraps it into a reusable tool.

## Reading primitives safely

Inside a leaf payload you read fields — ints, floats, strings. A bounds-checked sequential reader
prevents a malformed file from crashing your tool. The pattern: clamp at end-of-buffer and return zero
on overrun rather than reading out of bounds, but **advance the cursor even on overrun** so callers
stay aligned.

```cpp
struct Reader {
    const uint8_t* p; size_t n, cur = 0;
    Reader(const uint8_t* d, size_t len): p(d), n(len) {}
    template<class T> T read() {
        T v{};
        if (cur + sizeof(T) <= n) memcpy(&v, p + cur, sizeof(T));
        cur += sizeof(T);                  // advance even on overrun
        return v;
    }
    void skip(size_t k){ cur += k; }
    void skip_pad(){ while (cur < n && p[cur] == 0x11) ++cur; }   // strip 0x11 alignment
    std::string cstr(size_t maxlen = 256){                        // null-terminated
        std::string s;
        for (size_t i = 0; i < maxlen && cur < n; i++){ char c = (char)p[cur++]; if (!c) break; s += c; }
        return s;
    }
    std::string fixed(size_t len){                                // fixed-width field
        std::string s;
        for (size_t i = 0; i < len && cur + i < n; i++){ char c = (char)p[cur + i]; if (!c) break; s += c; }
        cur += len; return s;
    }
};
```

Two string conventions recur throughout the formats, and mixing them up shifts every field after the
string:

- **Fixed-width fields** (e.g. a 24-byte name slot): always consume the full width, even if the string
  is shorter. Use `fixed(width)`.
- **Null-terminated strings**: consume up to and including the terminator. Use `cstr()`.

## Why it is designed this way

The engine's own loader is exactly this recursive walk with a `visit()` that dispatches to registered
handlers ([C1.12](12-runtime-view.md)). By writing your reader as the *same* walk, you guarantee that
anything the game can load, you can parse — and anything your walk chokes on, the game would choke on
too. That equivalence is the quiet superpower of the chunk model: your tools and the engine agree on
the shape of every file, for free.

## Bending it — robustness choices that pay off

- **Always bounds-check, always.** The two checks in the flat walk are not optional politeness; they
  are the difference between a tool that reports "truncated chunk at 0x4A20" and one that segfaults on
  a file you were about to fix.
- **Prefer the explicit stack for untrusted input.** Recursion is cleaner to read, but a maliciously or
  accidentally deep file can overflow the call stack. For a batch tool that eats thousands of files,
  the iterative version never surprises you.
- **Keep `visit()` pure.** Have the walk *yield* chunks and do your logic outside it. A walk that also
  edits, prints, and extracts in one tangled function is where subtle cursor bugs hide.

---

**Continue:** [C1.4 — Alignment padding & null chunks](04-alignment-and-padding.md) ·
[Chapter 1 hub](C1-EAGL-Container-Model.md)
