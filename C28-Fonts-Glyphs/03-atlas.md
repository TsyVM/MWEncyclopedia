# C28.3 — The Atlas

> **The one-sentence version:** the letterform sheet is an ordinary DXT1/DXT3 TPK (commonly 256×256 or
> 512×256), so it exports and re-skins with the standard texture tools — the font's *look* is fully editable.

[← C28.2 — The glyph entry](02-glyph-entry.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md) ·
[Next: C28.4 — Rendering text →](04-rendering.md)

---

## The atlas is a normal TPK

The font's imagery — the actual pictures of the letters — is a standard **TPK** texture
([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), typically DXT1 or DXT3
([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)), commonly 256×256 or 512×256. There is nothing
font-specific about the atlas itself: it's a texture sheet with the glyphs packed onto it, and the glyph table
([C28.2](02-glyph-entry.md)) is what turns "a texture of letters" into "a font" by saying which rectangle is
which character.

Because it's a plain TPK, the atlas is:

- **Exportable** — extract it to DDS/PNG with the texture path ([C5.5](../C5-Textures-TPK/05-extract-replace.md)).
- **Re-skinnable** — replace the pixels to restyle the font ([C28.6](06-editing-fonts.md)).
- **Decodable** — DXT/ARGB decode ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) like any texture.

## How glyphs pack onto it

The glyphs are arranged on the atlas as a packing of small images — often a rough grid, sometimes a tighter
pack — with each glyph's location given by its UV rectangle ([C28.2](02-glyph-entry.md)):

```
512×256 atlas
┌──┬──┬──┬──┬──┬──┐
│A │B │C │D │E │F │   each cell (or packed region) is one glyph;
├──┼──┼──┼──┼──┼──┤   its (u0,v0,u1,v1) rectangle locates it
│G │H │… │  │  │  │
└──┴──┴──┴──┴──┴──┘
```

The packing doesn't have to be uniform — the UV rectangles can describe arbitrary sub-regions — but a legible
grid is common for a font sheet. The advance ([C28.2](02-glyph-entry.md)) is separate from the cell size, so a
glyph's image can be narrower than its cell.

## DXT and text quality

Fonts are usually **DXT1 or DXT3** ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)), which is a
deliberate quality/size trade for text:

- **DXT3** (explicit 4-bit alpha, [C6.3](../C6-Texture-Codecs/03-dxt3-dxt5.md)) suits anti-aliased glyph edges
  with crisp alpha — the reason DXT3 dominates cutout art applies to letterforms too.
- **DXT1** (1-bit alpha) suits hard-edged fonts where partial transparency isn't needed.

The trade is the usual DXT one: 4× smaller than uncompressed, with block-compression artifacts that are mostly
invisible at text sizes. A re-skin should keep the same format ([C28.6](06-editing-fonts.md)).

> ✅ *Verified (archive):* the font atlas is a normal DXT1/DXT3 TPK (e.g. 256×256 / 512×256), exportable and
> re-skinnable with the texture tools.

## Mipmaps and scaling

A font atlas may carry mipmaps ([C6.6](../C6-Texture-Codecs/06-mip-chains.md)) so text stays clean when scaled
down (small UI text), or it may be drawn at a fixed size without them. The glyph UVs are normalized
([C28.2](02-glyph-entry.md)), so the same table works at any atlas resolution — which is what lets a font be
authored at one size and sampled at another. When re-skinning, preserve the mip structure the original used
([C6.6](../C6-Texture-Codecs/06-mip-chains.md)).

## Editing implications

- **Re-skin by editing the atlas TPK** — the fully-supported font edit ([C28.6](06-editing-fonts.md)).
- **Keep glyphs in their rectangles.** Redraw each letter within the UV region the glyph table expects
  ([C28.2](02-glyph-entry.md)), or the table samples the wrong pixels.
- **Preserve format and dimensions** — same-size DXT swap is in-place ([C5.5](../C5-Textures-TPK/05-extract-replace.md));
  resizing the atlas needs the UVs (heuristic table) adjusted.
- **Match mips** — keep the original's mip chain for clean scaling.

---

### Key takeaways

- The font atlas is a normal DXT1/DXT3 TPK (256×256 / 512×256) — export and re-skin it with the texture tools.
- The glyph table turns the texture-of-letters into a font by mapping rectangles to characters.
- Glyphs pack onto the atlas (often a grid); each is located by its UV rectangle, image width independent of
  advance.
- DXT3 suits anti-aliased glyph alpha; keep the format on re-skin; normalized UVs make the atlas
  size-independent.
- Re-skin the atlas keeping glyphs in their rectangles, preserving format, dimensions, and mips.

**Continue:** [C28.4 — Rendering text](04-rendering.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md)
