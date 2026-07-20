# C6.2 — DXT1 / BC1, Block by Block

> **The one-sentence version:** DXT1 stores each 4×4 texel block in 8 bytes — two 16-bit 565 color
> endpoints and sixteen 2-bit selectors — and the *ordering* of the two endpoints secretly switches the
> block between a 4-color opaque mode and a 3-color-plus-transparent mode.

[← C6.1 — The format field](01-format-field.md) · [Chapter 6 hub](C6-Texture-Codecs.md) ·
[Next: C6.3 — DXT3 & DXT5 alpha →](03-dxt3-dxt5.md)

---

## The 8-byte block

A DXT1 image is a grid of independent 4×4 texel blocks, stored left-to-right, top-to-bottom. Each block is
exactly 8 bytes:

```
offset  size  field
+0      u16   color0   (RGB, 5-6-5, little-endian)
+2      u16   color1   (RGB, 5-6-5, little-endian)
+4      u32   indices  (16 texels × 2 bits, little-endian)
```

`color0` and `color1` are the block's two **endpoint** colors, each packed as 5 bits red, 6 bits green,
5 bits blue:

```python
def unpack565(c):
    r = (c >> 11) & 0x1F
    g = (c >>  5) & 0x3F
    b =  c        & 0x1F
    # expand to 8-bit by bit-replication (not just << ) for accurate midpoints
    return ((r*255+15)//31, (g*255+31)//63, (b*255+15)//31)
```

Bit-replication (or the rounding shown) matters: a lazy `r << 3` biases every channel dark and shifts the
interpolated midpoints, which is visible as banding on gradients.

## The four-color palette

From the two endpoints the decoder derives a **4-entry local palette**. Which two extra colors it computes
depends on comparing the raw 16-bit values `color0` and `color1`:

**Opaque mode — when `color0 > color1`:**

```
c0 = color0
c1 = color1
c2 = (2*c0 + c1) / 3      # ⅓ of the way from c0 to c1
c3 = (c0 + 2*c1) / 3      # ⅔ of the way
```

All four are fully opaque. Every texel picks one of `{c0, c1, c2, c3}`.

**Punch-through mode — when `color0 <= color1`:**

```
c0 = color0
c1 = color1
c2 = (c0 + c1) / 2        # midpoint
c3 = transparent black    # RGBA (0,0,0,0)
```

Now the block has only three colors plus one fully transparent slot. This is DXT1's single bit of alpha:
free, but binary — a texel is either opaque or fully see-through, with no partial values.

The interpolation is per-channel on the 8-bit expanded endpoints. Use integer math that matches D3D's
rounding (`(2*a + b + 1)/3` style) if you need bit-exact agreement with the GPU.

## The 2-bit selectors

The `indices` word holds 16 two-bit fields, one per texel, in row-major order with texel (0,0) in the
**lowest** two bits:

```python
def decode_dxt1_block(block):
    color0 = block[0] | (block[1] << 8)
    color1 = block[2] | (block[3] << 8)
    bits   = int.from_bytes(block[4:8], "little")
    pal    = build_palette(color0, color1)   # 4 × RGBA per the mode above
    out    = [[None]*4 for _ in range(4)]
    for texel in range(16):
        sel = (bits >> (2*texel)) & 0x3
        out[texel // 4][texel % 4] = pal[sel]
    return out                                # 4×4 RGBA
```

## Decoding a whole surface

Blocks tile the image in 4×4 steps. A surface whose dimensions are not multiples of four is padded up to
the next multiple (the extra texels exist in the block and are simply not sampled):

```python
def decode_dxt1(w, h, data):
    img = new_rgba(w, h)
    bx, by = (w + 3)//4, (h + 3)//4
    p = 0
    for gy in range(by):
        for gx in range(bx):
            blk = decode_dxt1_block(data[p:p+8]); p += 8
            blit_block(img, blk, gx*4, gy*4, w, h)   # clip at edges
    return img
```

`bx*by*8` is the exact byte size of one mip level — the identity you use to walk the mip chain
([C6.6](06-mip-chains.md)).

## Encoding back (for replacement)

To *write* DXT1 you must choose two endpoints per block that minimise error over its 16 texels, then assign
each texel the nearest palette entry. Good encoders search endpoint candidates along the block's principal
color axis; a serviceable one uses the block's min/max corners as endpoints. For MW replacement work you
rarely hand-roll this — you export to DDS, edit, and re-encode with an established DXT compressor
([C6.7](07-dds-interchange.md)) — but two rules keep the mode bit correct:

- If your texture needs **1-bit alpha**, ensure transparent texels land in punch-through mode
  (`color0 <= color1`) so the fourth slot is the transparent one.
- If it is **fully opaque**, keep `color0 > color1` so you get the extra interpolated color and better
  gradients.

> ✅ *Verified:* DXT1 blocks are 8 bytes and `(⌈w/4⌉·⌈h/4⌉·8)` sizes every level exactly — confirmed by the
> mip-chain byte totals of real DXT1 textures. The 565/interpolation/selector math is the standard BC1
> definition the GPU implements.

---

### Key takeaways

- DXT1 = 8 bytes per 4×4 block: two 565 endpoints + a 32-bit selector word.
- `color0 > color1` ⇒ four opaque colors; `color0 <= color1` ⇒ three colors + one transparent — DXT1's only
  alpha, and it is binary.
- Expand 565 by bit-replication, interpolate with D3D-style rounding, or gradients band.
- Texel (0,0) is the lowest 2 bits of the selector word; blocks tile in 4×4 steps with edge padding.
- Level size is `⌈w/4⌉·⌈h/4⌉·8` bytes — the mip-walk identity.

**Continue:** [C6.3 — DXT3 & DXT5 alpha (BC2 / BC3)](03-dxt3-dxt5.md) · [Chapter 6 hub](C6-Texture-Codecs.md)
