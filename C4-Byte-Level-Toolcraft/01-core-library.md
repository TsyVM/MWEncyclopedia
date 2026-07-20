# C4.1 — The Reusable Core Library

> **The one-sentence version:** collect the readers, walker, hashes, and codec from C1–C3 into one small,
> tested module so every experiment starts three lines in instead of re-deriving plumbing you already got
> right once.

[← Chapter 4 hub](C4-Byte-Level-Toolcraft.md) · [Next: C4.2 — Decoding unknowns →](02-decoding-unknowns.md)

---

## What it is

A single module — call it `mw` — that exports the primitives the rest of the book assumes. Nothing here
is new; it is C1–C3 packaged so that a fresh experiment imports a solved foundation rather than copy-pasting
a walker for the hundredth time. The value is not cleverness, it is *reuse with confidence*: this module
is tested once against retail data and then trusted everywhere.

## What it exports

```python
# mw.py — the workshop core. Import this in every experiment.
import struct

M = 0xFFFFFFFF

# ---- hashes (C2) ----
def joaat(s):
    if isinstance(s, str): s = s.encode('latin1')
    h = 0
    for c in s:
        h = (h + c) & M; h = (h + ((h << 10) & M)) & M; h ^= h >> 6
    h = (h + ((h << 3) & M)) & M; h ^= h >> 11; h = (h + ((h << 15) & M)) & M
    return h

def binhash(s):
    if isinstance(s, str): s = s.encode('latin1')
    h = 0
    for c in s: h = (c + 33 * h) & M
    return h

# lookup2 / 0xABCDEF00 (C2.2) — full body omitted here; paste from C2.2
from reflection_hash import lookup2   # or inline it

# ---- chunk walk (C1) ----
def walk_tree(buf, base=0, depth=0):
    off = 0
    while off + 8 <= len(buf):
        cid, size = struct.unpack_from('<II', buf, off)
        if off + 8 + size > len(buf): break
        yield depth, base + off, cid, size, memoryview(buf)[off+8:off+8+size]
        if cid & 0x80000000:
            yield from walk_tree(buf[off+8:off+8+size], base + off + 8, depth + 1)
        off += 8 + size

# ---- compression (C3) ----
def jdlz_decompress(data):
    assert data[:4] == b'JDLZ'
    out_len = int.from_bytes(data[8:12], 'little')
    out = bytearray(out_len); pos, op, f1, f2, n = 16, 0, 1, 1, len(data)
    while pos < n and op < out_len:
        if f1 == 1: f1 = data[pos] | 0x100; pos += 1
        if f2 == 1: f2 = data[pos] | 0x100; pos += 1
        if f1 & 1:
            b0, b1 = data[pos], data[pos+1]; pos += 2
            if f2 & 1: length = ((b0 & 0xF0) << 4 | b1) + 3; dist = (b0 & 0x0F) + 1
            else:      length =  (b0 & 0x1F) + 3;            dist = ((b0 & 0xE0) << 3 | b1) + 17
            for _ in range(length):
                if op >= out_len: break
                out[op] = out[op - dist]; op += 1
            f2 >>= 1
        else:
            out[op] = data[pos]; op += 1; pos += 1
        f1 >>= 1
    return bytes(out)

# ---- bounded reader (C1.3) ----
class Reader:
    def __init__(self, data, off=0): self.d = data; self.p = off
    def u8(self):  v = self.d[self.p] if self.p < len(self.d) else 0; self.p += 1; return v
    def u16(self): v = struct.unpack_from('<H', self.d, self.p)[0] if self.p+2 <= len(self.d) else 0; self.p += 2; return v
    def u32(self): v = struct.unpack_from('<I', self.d, self.p)[0] if self.p+4 <= len(self.d) else 0; self.p += 4; return v
    def f32(self): v = struct.unpack_from('<f', self.d, self.p)[0] if self.p+4 <= len(self.d) else 0.0; self.p += 4; return v
    def skip(self, k): self.p += k
    def skip_pad(self):
        while self.p < len(self.d) and self.d[self.p] == 0x11: self.p += 1
    def cstr(self, maxlen=256):
        s = bytearray()
        for _ in range(maxlen):
            if self.p >= len(self.d): break
            c = self.d[self.p]; self.p += 1
            if c == 0: break
            s.append(c)
        return s.decode('latin1')
    def fixed(self, n):
        raw = bytes(self.d[self.p:self.p+n]); self.p += n
        return raw.split(b'\0', 1)[0].decode('latin1')

# ---- universal opener (C1.9) ----
MAGICS = {b'VPAK':'vault', b'DDS ':'dds', b'ABKC':'bank', b'BNKl':'bank',
          b'SCHl':'mus', b'MPFF':'music_graph', b'LOCH':'loc'}
def open_eagl(path):
    buf = open(path, 'rb').read()
    if buf[:4] == b'JDLZ': buf = jdlz_decompress(buf)
    if buf[:4] in MAGICS:  return MAGICS[buf[:4]], buf
    if buf[:2] == b'MZ':   return 'pe', buf
    return 'chunks', buf
```

## Why keep it in one module

Three reasons, all about trust and speed:

1. **Test surface converges.** When the walker, the reader, and the codec live in one place, they are
   tested in one place. A bug fixed once is fixed everywhere. Scattered copies drift and rot.
2. **Experiments become short.** A new investigation is `import mw`, `open_eagl`, `walk_tree`, and a few
   lines of the actual question. The plumbing is not on screen because it is not in doubt.
3. **The confidence is transitive.** Because `mw` round-trips the retail files
   ([C4.5](05-validation.md)), any experiment built on it inherits that foundation. You are never
   simultaneously debugging your parser *and* your hypothesis.

## Grow it deliberately

As you decode formats in later chapters, promote each *verified* parser into `mw` — the TPK reader
([C5](../C5-Textures-TPK/C5-Textures-TPK.md)), the solid reader ([C8](../C8-Geometry-Solids/C8-Geometry-Solids.md)),
the vault reader ([C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)). The rule for promotion is
strict: a parser earns its place in `mw` only after it round-trips or reproduces known values across the
data set. Unverified code stays in the experiment where its uncertainty is visible; verified code moves
to the library where it is trusted.

## Bending it — keep the core honest

- **Never let unverified code into the core.** The whole value of `mw` is that importing it means "this is
  known good." One shaky function poisons that guarantee.
- **Keep it dependency-light and language-portable.** The book's code is deliberately raw-bytes and
  standard-library only, so the knowledge survives any toolkit ([README](../README.md)). Resist pulling in
  heavy frameworks that tie the core to one environment.
- **Version your dictionaries alongside it.** The resolver ([C2.4](../C2-Identifiers-And-Hashing/04-hash-resolution.md))
  is part of the workshop; persist and version its `hash → name` map so recovered names accumulate rather
  than evaporate between sessions.

---

**Continue:** [C4.2 — Reading an unknown structure](02-decoding-unknowns.md) ·
[Chapter 4 hub](C4-Byte-Level-Toolcraft.md)
