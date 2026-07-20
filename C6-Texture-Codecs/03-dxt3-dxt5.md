# C6.3 — DXT3 & DXT5 Alpha (BC2 / BC3)

> **The one-sentence version:** DXT3 and DXT5 are just DXT1 color with an 8-byte alpha block bolted on the
> front — DXT3's alpha is sixteen literal 4-bit values (crisp edges, MW's favourite), DXT5's is two 8-bit
> endpoints and a 3-bit ramp (smooth gradients) — making both 16 bytes per 4×4 block.

[← C6.2 — DXT1 / BC1](02-dxt1-bc1.md) · [Chapter 6 hub](C6-Texture-Codecs.md) ·
[Next: C6.4 — Uncompressed ARGB32 →](04-argb32.md)

---

## The shared shape

Both formats store each 4×4 block as **16 bytes**: an 8-byte alpha block followed by an 8-byte color block.
The color block is **bit-for-bit the DXT1 color block** from [C6.2](02-dxt1-bc1.md) — same 565 endpoints,
same 2-bit selectors — with one exception: in DXT3/DXT5 the color block is **always in four-color mode**
(the `color0 <= color1` punch-through case is not used, because alpha is carried separately). So if you
already decode DXT1 color, you are three-quarters done; only the alpha block differs.

```
DXT3 / DXT5 block (16 bytes):
+0   alpha block   (8 bytes)  ← the only difference between the two
+8   color block   (8 bytes)  ← identical to DXT1, always 4-color mode
```

## DXT3 (BC2): explicit 4-bit alpha

The alpha block is the simplest possible: **sixteen 4-bit alpha values**, one per texel, packed little-endian
with texel (0,0) in the lowest nibble.

```python
def decode_dxt3_alpha(block8):
    bits = int.from_bytes(block8, "little")
    a = []
    for texel in range(16):
        v = (bits >> (4*texel)) & 0xF
        a.append(v * 17)          # 4-bit → 8-bit: ×17 maps 0..15 to 0..255 exactly
    return a                       # row-major, 16 values
```

The `×17` (i.e. `v*255//15`, and `15*17 = 255`) expansion is exact. Because each texel's alpha is stored
**literally**, there is no interpolation and therefore **no gradient artifacts on hard edges** — a cutout's
silhouette stays crisp. That is precisely why the open-world art pipeline reaches for DXT3 by default: the
game is full of alpha-tested decals, signs, fences, and foliage where a clean edge matters more than smooth
falloff, and DXT3's 4-bit alpha gives 16 clean levels with zero ringing.

The trade is precision: 16 alpha levels only. For most cutouts that is invisible; for a soft glow it bands,
which is where DXT5 earns its place.

## DXT5 (BC3): interpolated alpha ramp

DXT5 spends its 8 alpha bytes very differently: two 8-bit alpha **endpoints** and sixteen **3-bit
selectors** into an 8-value ramp built from them.

```
alpha block (8 bytes):
+0  u8   alpha0
+1  u8   alpha1
+2  6 bytes = 16 texels × 3 bits (little-endian), texel (0,0) lowest
```

The 8-entry alpha palette is built by comparing the two endpoints — the same "ordering picks the mode"
trick DXT1 uses for color:

```python
def build_alpha_ramp(a0, a1):
    a = [a0, a1]
    if a0 > a1:                       # 8 interpolated values, no explicit 0/255
        for i in range(1, 7):
            a.append(((7-i)*a0 + i*a1) // 7)
    else:                             # 6 interpolated + explicit 0 and 255
        for i in range(1, 5):
            a.append(((5-i)*a0 + i*a1) // 5)
        a += [0, 255]
    return a                          # 8 entries

def decode_dxt5_alpha(block8):
    a0, a1 = block8[0], block8[1]
    ramp = build_alpha_ramp(a0, a1)
    bits = int.from_bytes(block8[2:8], "little")
    return [ramp[(bits >> (3*t)) & 0x7] for t in range(16)]
```

The `a0 > a1` branch gives eight interpolated steps for pure smooth alpha; the `a0 <= a1` branch trades two
of those steps for guaranteed exact 0 and 255, useful when a texture mixes fully-transparent and
fully-opaque regions with a soft transition between. Either way DXT5 delivers smoother alpha than DXT3, at
the cost of the interpolation ringing DXT3 avoids.

## Putting a block together

```python
def decode_dxt3_block(block16):
    alpha = decode_dxt3_alpha(block16[0:8])
    color = decode_dxt1_color(block16[8:16])   # always 4-color mode
    return combine(color, alpha)               # 4×4 RGBA

def decode_dxt5_block(block16):
    alpha = decode_dxt5_alpha(block16[0:8])
    color = decode_dxt1_color(block16[8:16])
    return combine(color, alpha)
```

Surface decoding is identical to DXT1 ([C6.2](02-dxt1-bc1.md)) but with a **16-byte** stride, so a level is
`⌈w/4⌉·⌈h/4⌉·16` bytes ([C6.6](06-mip-chains.md)).

## Choosing between them when you edit

When you replace a texture ([C5.5](../C5-Textures-TPK/05-extract-replace.md)) keep the **same format** so
the byte size is unchanged and the edit stays in-place. If you are authoring new art and get to choose:

- **Hard-edged alpha (cutouts, decals, UI masks)** → DXT3. Crisp, no ringing, and it is what the rest of the
  world uses so it matches.
- **Smooth alpha (glows, soft shadows, gradient fades)** → DXT5. The interpolated ramp avoids DXT3's 16-level
  banding.
- **No alpha at all** → DXT1, at half the size — don't pay 16 bytes/block for an alpha you don't use.

> ✅ *Verified:* DXT3 and DXT5 blocks are 16 bytes and size every mip level as `⌈w/4⌉·⌈h/4⌉·16` — confirmed
> against real DXT3 textures (e.g. `FONT_MW_BODY`, 256×256 DXT3 = 65 536 bytes = 64·64·16). The alpha-block
> math is the standard BC2/BC3 definition.

---

### Key takeaways

- DXT3 and DXT5 = 16 B/block: an 8-byte alpha block + the DXT1 color block (always 4-color mode).
- DXT3 alpha is sixteen literal 4-bit values (×17 to 8-bit) — crisp edges, no interpolation; MW's default.
- DXT5 alpha is two 8-bit endpoints + a 3-bit ramp; `a0>a1` ⇒ 8 interpolated, else 6 + exact 0/255 — smooth.
- Color decoding is shared with DXT1; only the alpha block and the 16-byte stride differ.
- Edit rule: preserve the format for in-place swaps; author DXT3 for hard alpha, DXT5 for soft, DXT1 for none.

**Continue:** [C6.4 — Uncompressed ARGB32](04-argb32.md) · [Chapter 6 hub](C6-Texture-Codecs.md)
