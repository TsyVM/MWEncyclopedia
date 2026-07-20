# C63.8 — Wall & Object Collision (`WCollisionPack`, `0x0003B801`)

> **The one-sentence version:** `WCollisionPack` is the collision the car is *stopped* by — a `CARP` article
> container (loader `0x64AD80` → manager `0x9B3890`) whose table-of-contents lists `ca` collision-geometry articles
> (local-space corridor-edge vertex strips) and `ci` instance lists, with cumulative-offset spans that make
> size-neutral editing possible.

[← C63.7 — Terrain collision mesh](07-terrain-collision.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: C63.9 — Smackables & FX placements →](09-smackables-emitters.md)

---

## What stops the car

The terrain mesh ([C63.7](07-terrain-collision.md)) is the floor; **`WCollisionPack`** (chunk `0x0003B801`) is the
*walls*. It holds the horizontal collision geometry — guardrails, building faces, barriers, the edges that keep the
car on the road — everything a body is *stopped by* rather than *rests on*. This is the geometry the narrow-phase
([C63.3](03-narrow-phase.md)) tests a car against to produce a contact ([C43.1](../C43-Collision-Contacts/01-detection.md))
when it hits a wall. Its loader at `0x64AD80` routes the loaded pack to the collision manager object at `0x9B3890`.

The name is a verified engine string, and it travels with a family of them: `WCollisionPack` (the container),
`WCollisionAssets` (the geometry assets), and `CollisionInstanceList` / `CollisionObjectList` (the object lists,
[C43.1](../C43-Collision-Contacts/01-detection.md)) — the same object-list names the runtime collision world
([C63.1](01-collision-world.md)) is built on. So `WCollisionPack` is the *on-disk form* of the `CollisionObjectList`
the runtime queries: the file holds the pack, the loader hands it to the manager, and the manager populates the
object lists the broad-phase ([C63.2](02-broad-phase.md)) then culls against.

> ✅ *Verified:* `WCollisionPack`, `WCollisionAssets`, `CollisionInstanceList`, `CollisionObjectList` are strings in
> `speed.exe`; the loader at `0x64AD80` tests `cmp dword [eax], 0x0003B801` and references the manager at
> `0x9B3890`. The retail world carries **390 `WCollisionPack` containers, 7,934 collision articles**, rebuilding
> byte-for-byte.

## A CARP article container

`WCollisionPack` is not a bespoke format — it's built on the **`CARP` article container**, the same table-of-contents
container structure the road network uses ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)). (The
two chunk IDs are neighbours: `0x0003B800` is the CARP road network, `0x0003B801` is the CARP collision pack — the
same container, different article payloads.) After the `0x11` alignment padding ([C63.6](06-ondisk-collision-data.md)),
the layout is:

```
16-byte inner header:
  +0x00  u32  innerTotal    (bytes after this 16-byte header)
  +0x04  u32  sectionId
  +0x08  u64  0
CARP container:
  0x28-byte header: 'PRAC', 0x2A, 0, 1, 'itrA', 0x12, articleRowCount, 1, 'emaN', 0x0402
  16-byte TOC rows (one per article):
    +0x00  u32  flags               (0 on the first row, 1 after)
    +0x04  u32  cumulativeEndOffset (END of this article in the data region)
    +0x08  u16  index               (per-tag article index)
    +0x0A  u8   tagLo               ('a' or 'i')
    +0x0B  u8   tagHi               ('c')
    +0x0C  u32  sizeCode
  terminator row:
    { u32 caCount, u32 runtimeSizeHint, 0xAA x8 }
  0xAA fill, section-name string, 0xAA fill,
  article data region  (article i spans [end[i-1], end[i]) ),
  undecoded footer/index + 0xAA fill to the end of the payload.
```

The fourcc `'PRAC'` is `CARP` byte-reversed; `'itrA'` reverses to `Arti`(cle) and `'emaN'` to `Name` — the tell-tale
little-endian fourcc storage ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)). The tag bytes are stored
low-then-high, so a row reads `"ca<index>"` or `"ci<index>"` when you concatenate `tagLo`/`tagHi` with the index.

## `ca` articles and `ci` instances

The TOC's two tag types split the collision data into geometry and placement:

- **`ca` — collision article.** Local-space collision *geometry*: vertex strips of `{vec3, flags 0x200}` and
  `{vec3, f32 ≈ 0.1}` pairs — the **left/right edges of the drivable corridor**. These are the barrier surfaces the
  car slides along and bounces off: the road's containing walls, expressed as edge strips in the section's local
  frame.
- **`ci` — collision instance list.** The *placement* layer — which collision articles are instanced where. Where
  `ca` is the shape, `ci` is the "put this shape at these transforms," the same shape/instance split the render side
  uses ([C8.2](../C8-Geometry-Solids/02-object-header.md)).

So a `WCollisionPack` is a small library of corridor-edge shapes (`ca`) plus a list of where to place them (`ci`) —
the section's walls, factored into reusable geometry and instances. That the collision walls are stored as
*corridor edges* is telling: MW's world is fundamentally a *road network* ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)),
and the collision that matters most is *staying on the road* — so the wall geometry is literally the road's left and
right boundaries.

> 🟡 *Reasoned:* the `ca` = local-space corridor-edge geometry / `ci` = instance-list reading follows from the tag
> names, the observed vertex-strip payloads (`{vec3, 0x200}` / `{vec3, ~0.1}` left/right pairs), and the
> shape/instance split; the exact strip semantics (the `0x200` flag, the `~0.1` scalar) are partly undecoded. The
> `CARP` container, the TOC row layout, the tag encoding, and the article spans are verified (byte-for-byte
> rebuild).

## Cumulative offsets and size-neutral editing

The TOC stores each article's **cumulative *end* offset**, not its start — article `i` occupies the data-region span
`[end[i-1], end[i])`. This chained-offset design has a useful property: an article can be made **zero-size** by
setting its end equal to the previous end, which *empties its span without removing its row*. Retail data already
contains zero-size articles, proving the loader tolerates them.

That's what makes collision editing **size-neutral** ([C63.6](06-ondisk-collision-data.md)): to remove an article's
geometry, empty its data span, keep its TOC row (as a zero-length entry), and grow the trailing `0xAA` fill by the
freed bytes — so the chunk's *total size never changes* and nothing downstream misaligns. Two fields are deliberately
**preserved verbatim** rather than recomputed, because their semantics aren't fully decoded and getting them wrong
corrupts the pack: the terminator row's second word (a runtime size *hint* that does **not** equal any in-file byte
count on several retail sections) and the undecoded footer/index after the last article (which may reference article
offsets). Respecting "preserve what you don't fully understand" is the difference between a byte-exact rebuild and a
subtly broken one.

> ✅ *Verified:* article spans are cumulative-end offsets (`article i` = `[end[i-1], end[i])`); zero-size articles
> occur in retail data; the pack rebuilds byte-for-byte when the terminator's size-hint word and the trailing
> footer are preserved rather than recomputed.

## RE implications

- **`WCollisionPack`** (`0x0003B801`, loader `0x64AD80` → manager `0x9B3890`) is the **wall/object collision** — what
  stops the car, the on-disk form of the runtime `CollisionObjectList`.
- **A CARP article container** — same TOC-of-articles structure as the road network (`0x0003B800`,
  [Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)); `'PRAC'` = `CARP` reversed.
- **`ca` = collision geometry** (local-space corridor-edge strips), **`ci` = instance list** (placement) — the
  shape/instance split.
- **Cumulative-end offsets** enable **size-neutral editing** — empty an article's span, keep its row, grow the tail
  fill; preserve the terminator hint and footer verbatim.

---

### Key takeaways

- `WCollisionPack` (`0x0003B801`) is **what stops the car** — the horizontal wall/object collision, loaded by
  `0x64AD80` into the manager at `0x9B3890`; it's the **on-disk form** of the runtime `CollisionObjectList`
  ([C63.1](01-collision-world.md)).
- It's a **`CARP` article container** — the *same* container the road network uses
  ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)); the two are neighbouring chunk IDs
  (`0x0003B800` road, `0x0003B801` collision). `'PRAC'`/`'itrA'`/`'emaN'` are `CARP`/`Arti`/`Name` reversed.
- The TOC lists **`ca` collision articles** (local-space **corridor-edge** vertex strips — the road's containing
  walls) and **`ci` instance lists** (placement) — the classic **shape/instance split**.
- Article spans are **cumulative-end offsets** (`article i` = `[end[i-1], end[i])`), which makes **size-neutral
  editing** possible — empty a span, keep its row, grow the tail fill so nothing downstream misaligns.
- Two fields are **preserved verbatim** (the terminator's runtime size-*hint* — not an in-file byte count — and the
  undecoded footer); byte-for-byte rebuild across **390 packs / 7,934 articles** depends on not "correcting" them.

**Continue:** [C63.9 — Smackables & FX placements](09-smackables-emitters.md) · [Chapter 63 hub](C63-Collision-World.md)
