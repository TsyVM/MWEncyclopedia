# C15.2 — The Streaming Section Table

> **The one-sentence version:** chunk `0x00034110` is 720 sections of 92 bytes each; every entry carries a
> section id (`+0x00`), a byte offset into the stream file (`+0x14`), payload size fields
> (`+0x18`/`+0x1C`/`+0x20`), a count (`+0x24`), and a section hash (`+0x34`).

[← C15.1 — The master track file layout](01-master-layout.md) · [Chapter 15 hub](C15-Track-Streaming.md) ·
[Next: C15.3 — Section residency & the stream file →](03-residency.md)

---

## The 92-byte entry

Decoded from the real section table, each entry has this shape (offsets within the 92-byte record):

| Offset | Field | Section 0 | Section 2 | Role |
|---|---|---|---|---|
| `+0x00` | section id | `0x3058` | `0x3059` | the section's identifier |
| `+0x14` | stream offset | (0) | `0x011D8000` | byte offset in `STREAML2RA.BUN` |
| `+0x18` | size A | `0x011D73D0` | `0x064E2124` | payload size / memory size |
| `+0x1C` | size B | `0x011D73D0` | `0x064E2124` | payload size (often = A) |
| `+0x20` | size C | `0x011D73D0` | `0x0004AAFC` | resident/compressed size |
| `+0x24` | count | `0x0A` | `0x1E` | per-section count/flags |
| `+0x34` | section hash | `0x0B1A6276` | `0x34B72AD7` | the section's asset hash |
| `+0x38`…`+0x58` | (reserved/zero) | 0 | 0 | padding/unused |

720 × 92 = 66 240 bytes, matching the chunk exactly — the divide-to-N check
([C15.1](01-master-layout.md)) that confirms the parse.

## The load-bearing fields

Three fields do the streaming work:

- **`+0x14` stream offset** — where the section's payload begins in `STREAML2RA.BUN`
  ([C15.3](03-residency.md)). This is the pointer that turns a section id into actual bytes to read.
- **`+0x18`/`+0x1C`/`+0x20` sizes** — how many bytes to read. Multiple size fields appear because a section
  has more than one measure: an in-memory size and a stored (possibly compressed) size, and per-stream sizes.
  In some sections all three are equal (uncompressed, single stream); in others `+0x20` is much smaller than
  `+0x18` (compressed on disk, larger in memory).
- **`+0x00` id / `+0x34` hash** — the section's identity, used to reference it from position data
  ([C15.4](04-world-grid.md)) and elsewhere.

> ✅ *Verified:* the table is 720 × 92; ids at `+0x00` (`0x3058`, `0x3059`, `0x305A`, …), a stream offset at
> `+0x14`, three size fields at `+0x18`/`+0x1C`/`+0x20`, and a hash at `+0x34`, all decoded from real entries.
> 🟡 *Reasoned:* the precise distinction among the three size fields (memory vs compressed vs per-stream) is
> inferred from their relative magnitudes across sections; the field *positions* are verified.

## Reading the table

```python
def read_section_table(chunk):                 # chunk = 0x00034110 payload, 720 × 92
    sections = []
    for i in range(len(chunk) // 92):
        e = chunk[i*92 : (i+1)*92]
        sections.append({
            "id":        u32(e, 0x00),
            "offset":    u32(e, 0x14),          # into STREAML2RA.BUN
            "size_mem":  u32(e, 0x18),
            "size_alt":  u32(e, 0x1C),
            "size_disk": u32(e, 0x20),
            "count":     u32(e, 0x24),
            "hash":      u32(e, 0x34),
        })
    return sections
```

## The sizes tell you about compression

The relationship among the three size fields is informative:

- **All three equal** → the section is stored as-is (its disk size equals its memory size); a straight read.
- **`size_disk` < `size_mem`** → the section is compressed on disk (JDLZ, [Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)); read `size_disk` bytes and inflate to `size_mem`.

So the section table doubles as a compression manifest: comparing the disk and memory sizes tells the streamer
whether to decompress after reading, and how much memory to reserve. This is the streaming analogue of the
TPK's compressed-variant handling ([C5.3](../C5-Textures-TPK/03-two-variants.md)).

## The hash identifies the section's content

The `+0x34` hash is an **asset hash** ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)) naming the
section's content, letting other systems reference a section by identity rather than table index. Like all
asset hashes it is read, not recomputed ([C8.3](../C8-Geometry-Solids/03-object-hash.md)); treat it as the
section's stable name.

---

### Key takeaways

- `0x00034110` = **720 × 92**; each entry: id (`+0x00`), stream offset (`+0x14`), three sizes
  (`+0x18`/`+0x1C`/`+0x20`), count (`+0x24`), hash (`+0x34`).
- The stream offset + a size field locate and bound a section's payload in `STREAML2RA.BUN`.
- Three sizes distinguish memory vs on-disk (compressed) sizes — the table is also a compression manifest.
- `size_disk < size_mem` means JDLZ-compressed; inflate after reading.
- The `+0x34` asset hash is the section's stable identity; read it, don't recompute it.

**Continue:** [C15.3 — Section residency & the stream file](03-residency.md) · [Chapter 15 hub](C15-Track-Streaming.md)
