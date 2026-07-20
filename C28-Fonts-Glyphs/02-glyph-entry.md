# C28.2 — The Glyph Entry

> **The one-sentence version:** each glyph is a ≈20-byte record — a `u16` codepoint, a float UV rectangle
> `(u0, v0, u1, v1)` locating its image on the atlas, and a float `advance` for how far the cursor moves after
> drawing it.

[← C28.1 — The atlas + glyph table](01-atlas-glyph-table.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md) ·
[Next: C28.3 — The atlas →](03-atlas.md)

---

## The record

A glyph table ([C28.1](01-atlas-glyph-table.md)) is an array of ≈20-byte glyph entries:

| Offset | Type | Field |
|---|---|---|
| `+0x00` | `u16` | **codepoint** — the character |
| `+0x??` | `f32` | **u0** — left edge on the atlas |
| … | `f32` | **v0** — top edge |
| … | `f32` | **u1** — right edge |
| … | `f32` | **v1** — bottom edge |
| `+0x??` | `f32` | **advance** — cursor movement after drawing |

Three facts, everything a renderer needs ([C28.4](04-rendering.md)): **which** character (codepoint), **where**
its picture is (the UV rectangle), and **how wide** it lays out (advance).

## Codepoint: the character's identity

The `u16` codepoint keys the glyph by character — the Unicode/codepage value of the letter. A renderer looks up
a glyph by the character it wants to draw ([C28.4](04-rendering.md)), so the codepoint is the table's search
key. Sixteen bits covers the Basic Multilingual Plane, enough for the game's Latin, Greek, Cyrillic, and CJK
scripts ([C28.5](05-codepoints.md)) — each language's font carries the codepoints its script needs.

## UV rectangle: where the image is

The four floats `(u0, v0, u1, v1)` are the glyph's **rectangle on the atlas** ([C28.3](03-atlas.md)) in
normalized texture coordinates (0–1). `(u0, v0)` is the top-left corner, `(u1, v1)` the bottom-right, so the
glyph's image is the sub-region `[u0,u1] × [v0,v1]` of the atlas texture. Drawing the glyph samples exactly
this rectangle. Normalized coordinates mean the same table works regardless of the atlas's pixel dimensions —
the rectangle scales with the texture.

## Advance: how wide it lays out

The `advance` float is the glyph's **layout width** — how far the text cursor moves to the right after drawing
this character, before the next one ([C28.4](04-rendering.md)). Advance is a *metric*, distinct from the
glyph's *image width*: a narrow letter like `i` has a small advance, a wide one like `W` a large advance, and a
space has an advance but no image. Advance is what makes text **proportional** (variable-width) rather than
monospaced — each character occupies its natural width.

## The metric caveat

Because the glyph-table chunk id is **unconfirmed** ([C28.1](01-atlas-glyph-table.md)), the exact field order
within the ≈20-byte record (and the precise chunk that holds it) is decoded **heuristically** — a payload that
divides evenly into ≈20-byte records with plausible codepoints, UVs in [0,1], and sensible advances. So:

> 🟡 *Reasoned:* the glyph-entry fields (codepoint, UV rectangle, advance) and ≈20-byte size are a heuristic
> decode, reliable enough to *read* metrics but not fully chunk-confirmed. The atlas
> ([C28.3](03-atlas.md)) is unambiguous and fully editable.

This is why the chapter distinguishes **re-skinning** (edit the atlas — safe) from **metric editing** (edit the
table — heuristic, [C28.6](06-editing-fonts.md)).

## Reading glyphs

```python
def parse_glyphs(table_payload, REC=20):
    glyphs = {}
    for i in range(len(table_payload) // REC):
        r = table_payload[i*REC:(i+1)*REC]
        cp = struct.unpack_from("<H", r, 0)[0]
        u0, v0, u1, v1, adv = struct.unpack_from("<5f", r, 4)   # heuristic field order
        glyphs[cp] = {"uv": (u0, v0, u1, v1), "advance": adv}
    return glyphs
```

## Editing implications

- **Read metrics, edit the atlas.** Reading glyph rectangles/advances is reliable; editing them is heuristic —
  prefer atlas re-skins ([C28.6](06-editing-fonts.md)).
- **Keep UVs in [0,1]** — the rectangle is normalized; values outside the range sample off-atlas.
- **Advance ≠ image width** — changing advance re-spaces text without changing the glyph image.
- **Codepoint is the key** — a glyph must key on the character it draws, or lookups miss.

---

### Key takeaways

- A glyph entry is ≈20 bytes: `codepoint (u16)`, a float UV rectangle `(u0,v0,u1,v1)`, and a float `advance`.
- Codepoint is the character identity/search key; UV rectangle is the atlas sub-region; advance is the layout
  width.
- Normalized UVs make the table atlas-size-independent; advance gives proportional (variable-width) text.
- The field layout is a **heuristic** decode (chunk id unconfirmed) — reliable to read, cautious to edit.
- Read metrics freely, edit the atlas for re-skins, keep UVs normalized, and key glyphs by codepoint.

**Continue:** [C28.3 — The atlas](03-atlas.md) · [Chapter 28 hub](C28-Fonts-Glyphs.md)
