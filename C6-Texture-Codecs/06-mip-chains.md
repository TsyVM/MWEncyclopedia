# C6.6 — Mip Chains, Sizing & the Block Rule

> **The one-sentence version:** a texture's byte slice is its levels stored largest-first with no padding;
> each level halves `w` and `h`, linear formats size a level as `w·h·bpp` and block formats as
> `⌈w/4⌉·⌈h/4⌉·blockBytes` — and block levels **never shrink below one 4×4 block**, which is the rule people
> forget.

[← C6.5 — Palettized P8](05-pal8.md) · [Chapter 6 hub](C6-Texture-Codecs.md) ·
[Next: C6.7 — The DDS interchange container →](07-dds-interchange.md)

---

## The chain, physically

Every format in MW stores its mip chain the same way: **level 0 (full size) first**, then each successive
level at half the width and half the height (rounding down, floored at 1), packed back-to-back with no
inter-level padding, all inside the single byte slice the entry's `total_size` covers
([C5.4](../C5-Textures-TPK/04-finding-pixels.md)). `base_size` (`+0x40`) is level 0 alone; `total_size`
(`+0x38`) is the whole chain; `mip_count` (`+0x4E`) is how many levels there are.

```
[ level 0 ][ level 1 ][ level 2 ] ... [ level N-1 ]
  w×h        w/2×h/2    w/4×h/4          1×1-ish
```

## Level dimensions

Dimensions halve with a floor at 1, independently per axis:

```python
def level_dims(w, h, level):
    return (max(1, w >> level), max(1, h >> level))
```

A non-square texture reaches 1 on its shorter axis first and then continues halving only the longer axis
(e.g. 512×128 → 256×64 → … → 4×1 → 2×1 → 1×1).

## Level byte sizes — the two rules

**Linear formats** (ARGB32 `bpp=4`, P8 `bpp=1`) size a level by texel count:

```
size(level) = level_w * level_h * bpp
```

**Block formats** (DXT1 `blockBytes=8`; DXT3/DXT5 `blockBytes=16`) size a level by *block* count, and this
is where the floor lives:

```
size(level) = max(1, ceil(level_w / 4)) * max(1, ceil(level_h / 4)) * blockBytes
```

A 2×2 or 1×1 DXT level is **not** a fraction of a block — it is a full 4×4 block (8 or 16 bytes), because
BC compression has no sub-block representation. Sizing small mips as `w·h·bpp` instead of by whole blocks
undercounts every level below 4×4 and desynchronises the walk. This single mistake accounts for most
"my mip offsets drift near the end of the chain" bugs.

```python
def level_size(fmt, w, h):
    if fmt in ("DXT1",):          return max(1,(w+3)//4)*max(1,(h+3)//4)*8
    if fmt in ("DXT3","DXT5"):    return max(1,(w+3)//4)*max(1,(h+3)//4)*16
    if fmt == 0x15:               return w*h*4      # ARGB32
    if fmt == 0x29:               return w*h*1      # P8 indices
```

## Seeking to a level

To jump to mip *k*, sum the sizes of levels `0 … k-1` and add to `data_offset`:

```python
def mip_offset(fmt, w, h, k):
    off = 0
    for level in range(k):
        lw, lh = level_dims(w, h, level)
        off += level_size(fmt, lw, lh)
    return off        # relative to this texture's data_offset in the blob

def total_size(fmt, w, h, mip_count):
    return sum(level_size(fmt, *level_dims(w, h, l)) for l in range(mip_count))
```

`total_size(...)` computed this way must equal the entry's `total_size` field. That equality is your
per-texture correctness check — if they disagree, either the format, the dimensions, or the mip count was
misread, or the texture is not the format you think.

## Worked checks against real textures

Every one of these is a texture parsed from retail data ([C5.4](../C5-Textures-TPK/04-finding-pixels.md)),
and the formula reproduces its stored size:

| Texture | W×H | Format | Mips | Formula | Stored `total_size` |
|---|---|---|---|---|---|
| `MW_LOGO` | 512×128 | ARGB32 | 1 | 512·128·4 = 262 144 | 262 144 ✓ |
| `FONT_MW_BODY` | 256×256 | DXT3 | 1 | 64·64·16 = 65 536 | 65 536 ✓ |
| `BASEPOLY` | 32×32 | DXT3 | 1 | 8·8·16 = 1024 | 1024 ✓ |
| `TREAD` | 128×64 | P8 | chain | 8192 + 2048 + 512 + … | 10 880 ✓ |

The DXT3 rows show the block rule in action: 256×256 is 64×64 blocks, not 256·256 texels — off by the
`blockBytes/16`-per-texel factor if you forget.

## Do textures always carry full chains?

No — `mip_count` varies per texture and is frequently **1** (base level only), as with `MW_LOGO`,
`FONT_MW_BODY`, and `BASEPOLY`. UI and font art often ship without mips because they are drawn at a fixed
size; world surfaces that are viewed at range carry full chains. Always trust the stored `mip_count`
(`+0x4E`) rather than assuming `⌊log2(max(w,h))⌋+1`; the pipeline does not always generate the full chain.

---

### Key takeaways

- Levels are stored largest-first, no padding; `base_size` = level 0, `total_size` = whole chain,
  `mip_count` = level count.
- Dimensions halve with a per-axis floor of 1.
- Linear level size = `w·h·bpp`; block level size = `⌈w/4⌉·⌈h/4⌉·blockBytes` with a **one-block floor**.
- `Σ level_size == total_size` is the per-texture correctness check.
- `mip_count` is often 1 (UI/fonts); trust the stored value, never assume a full chain.

**Continue:** [C6.7 — The DDS interchange container](07-dds-interchange.md) · [Chapter 6 hub](C6-Texture-Codecs.md)
