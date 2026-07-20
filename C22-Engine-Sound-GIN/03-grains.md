# C22.3 — Grains & the Grain Table

> **The one-sentence version:** after the header comes an offset table (at `+0x20`) whose monotonically
> increasing entries mark the start of each waveform grain in the sample data — short snippets of engine
> waveform that the synth stitches and pitches across the rev range.

[← C22.2 — The Gnsu header](02-gnsu-header.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md) ·
[Next: C22.4 — The RPM→synth bridge →](04-rpm-bridge.md)

---

## The grain-offset table

Beginning at `+0x20`, the GIN header carries a table of **increasing `u32` offsets** — verified on
`GIN_Acura_ITR.gin` as `0x1E0, 0xE5D, 0x19AB, 0x2581, 0x3386, 0x4143, …`. Each entry is the start of one
**grain** in the sample/grain data (which begins at the `+0x18` data offset,
[C22.2](02-gnsu-header.md)). The `+0x14` count (229) sizes the table, so grain *k* runs from
`offset[k]` to `offset[k+1]`:

```python
def read_grains(buf, header):
    grains = []
    offs = header["grain_offsets"] + [header["data_off_end"]]   # sentinel = end of data
    for k in range(len(offs) - 1):
        start, end = offs[k], offs[k+1]
        grains.append(buf[header["data_off"] + start : header["data_off"] + end])
    return grains
```

The offsets increasing roughly evenly (spacings ~3000) is the signature of fixed-ish-length grains laid out
back to back — a clean, table-indexed array, the same "offset table + payload" pattern as the ABK `PT` table
([C19.3](../C19-Audio-Banks/03-abk-bnkl.md)) or the SolidList directory
([C8.1](../C8-Geometry-Solids/01-solidlist-container.md)).

## What a grain is

Each grain is a **short snippet of engine waveform** — a few cycles of the engine's firing pulses,
characteristic of some point in the rev range. On its own a grain is a fraction of a second of engine sound; in
sequence, pitched and crossfaded, the grains become a continuous note. The grains span the file's RPM range
([C22.2](02-gnsu-header.md)), so lower-indexed grains tend to capture lower-RPM character and higher-indexed
grains higher-RPM character (the synth's mapping, [C22.4](04-rpm-bridge.md)).

The audio inside the grains is waveform data — decoded like the other engine/movie audio via the shared codec
layer (EA-XAS is the engine-audio codec, [C20.4](../C20-Audio-Codecs/04-xas-ima.md)), or PCM for the
tool-exported variants.

## Why grains, not one waveform

Storing the engine as indexed grains rather than one long waveform is what makes the synthesis work
([C22.1](01-granular-synthesis.md)):

- **Selectable by RPM.** The synth can jump to the grain(s) matching the current rev without playing through
  everything — the offset table is a random-access index into the rev range.
- **Crossfadeable.** Adjacent grains can be blended, so the transition between rev levels is seamless rather
  than stepped ([C22.4](04-rpm-bridge.md)).
- **Loopable.** A grain (or a small set) can loop to hold a steady RPM, and the synth moves to other grains as
  RPM changes.
- **Compact.** A few hundred short grains cover the whole rev range, versus recording every RPM.

The table is the crossfade map's backbone: it says where each grain is, so the synth can pick, pitch, and blend
them by rev.

> ✅ *Verified:* the grain-offset table at `+0x20` holds monotonically increasing offsets (229 of them per the
> `+0x14` count), indexing back-to-back grains in the data region.
> 🟡 *Reasoned:* that lower-index grains map to lower RPM (vs an explicit per-grain RPM field) is inferred from
> the range + ordering; the table structure and grain indexing are verified.

## Editing implications

- **Keep the table pointing at valid grain starts.** Replace grain audio and the offsets must still mark each
  grain's beginning; a mis-aligned offset plays a grain mid-waveform (a click or wrong pitch).
- **Preserve grain count and ordering.** The synth's RPM→grain mapping assumes the count and order
  ([C22.4](04-rpm-bridge.md)); reordering grains re-maps the rev range.
- **Fit grain lengths.** Wildly changing a grain's length shifts every later offset — re-stamp the table, and
  ideally keep lengths similar so the crossfade timing holds ([C22.6](06-editing-gin.md)).
- **Match the codec.** Encode replacement grains with the engine-audio codec (EA-XAS) at the file's rate.

---

### Key takeaways

- The grain-offset table (`+0x20`, sized by the `+0x14` count) marks each grain's start in the data region.
- Grain *k* spans `offset[k]…offset[k+1]`; the offsets increase roughly evenly (back-to-back grains).
- A grain is a short snippet of engine waveform; sequenced/pitched/crossfaded, grains make a continuous note.
- Grains enable RPM-selectable, crossfadeable, loopable, compact engine sound — the table is the crossfade
  map's backbone.
- Edit by keeping offsets on valid grain starts, preserving count/order, fitting lengths, and matching the
  codec.

**Continue:** [C22.4 — The RPM→synth bridge](04-rpm-bridge.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md)
