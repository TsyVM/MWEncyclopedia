# Chapter 16 — Scenery, Props & the Cull Tree

> **Goal of this chapter:** open a streamed world section and decode what fills it — the model definitions,
> the placed prop instances, and the spatial cull tree that decides which of them are potentially visible —
> so you can move, add, remove, and cull scenery correctly.

Chapter 15 delivered sections as bytes; this chapter decodes their **contents**. A section's world is built
from *scenery*: a small set of **model definitions** (which meshes exist here) and a large set of
**instances** (each a placed copy with its own transform), organised under a per-section **cull tree** that
turns "test every prop" into "descend a few nodes." Together they are how a city block of thousands of props
is stored compactly and drawn cheaply.

> **Verified against retail data.** The scenery structures are confirmed at two levels: the master file's
> per-section scenery organisation (container `0x80034150` with a header of count 373, `SceneryInfos`, group
> lists, and preculler data) parsed directly from `TRACKS/L2RA.BUN`, and the per-section instance/info/tree
> record formats — **64-byte `SceneryInstance`**, **72-byte `SceneryInfo`**, **36-byte tree node** — decoded
> against the full retail data set (947 sections, 19,578 tree nodes, 77,783 instances) with the cull-tree
> leaves forming an exact partition of the instances. The three per-section chunk IDs are confirmed by their
> loader dispatch in `speed.exe`: `SceneryHeader` **`0x00034101`**, `SceneryInfos` **`0x00034102`**, and
> `SceneryInstances` **`0x00034103`** (each tested as a `cmp dword [reg], <id>` immediate). The instance's
> orientation is a quantized `int16` 3×3 matrix (÷ 8192) at `+0x2C` ([C16.3](03-instance-record.md)).

---

## Deep-dive pages

- [C16.1 — Scenery in a section](01-scenery-section.md): the `ScenerySection` container and the chunks that
  hold models, instances, and the tree.
- [C16.2 — Models vs instances](02-models-instances.md): why scenery splits the *what* from the *where*, and
  which record you edit.
- [C16.3 — The 64-byte SceneryInstance](03-instance-record.md): the placement record — AABB, transform, and
  the info index — decoded field by field.
- [C16.4 — The 72-byte SceneryInfo](04-info-record.md): the model definition — the solid link, draw distance,
  and flags.
- [C16.5 — The cull tree](05-cull-tree.md): the 36-byte node, the ≤ 5 fanout, and the partition invariant that
  keeps every prop drawn exactly once.
- [C16.6 — Editing scenery safely](06-editing-scenery.md): moving, duplicating, and adding props while keeping
  the info table, indices, and cull tree honest.

---

## 16.1 A section is models plus placements

Inside a streamed section ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)), scenery is a
`ScenerySection` container whose chunks separate model definitions from placements from the visibility index:

```
ScenerySection
├── SceneryInfos       72 bytes per model definition   (what can be placed here)
├── SceneryInstances   64 bytes per placed instance    (each copy's transform + info index)
├── SceneryTreeNodes   36 bytes per cull-tree node      (which instances are visible from where)
└── SceneryOverrideHooks  128-byte records (override data, preserve-raw)
```

The three-way division is the whole design: a handful of *infos* (a lamppost model), thousands of *instances*
(every lamppost in the block), and a *tree* that indexes them spatially ([C16.1](01-scenery-section.md)). The
master track file additionally carries a section-level scenery organisation (`0x80034150` — verified header
count 373, model-index group lists), but the actual placements live in the streamed section payloads.

## 16.2 The model/instance split

A **SceneryInfo** is a model *definition* — it names the solid to draw ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md))
and its draw-distance/LOD rules. A **SceneryInstance** is a *placement* — a transform plus an index into the
info table. Drawing a prop is `instance → info → solid`, with the instance's transform positioning it. This
split is why a mesh is stored once but placed thousands of times, and why the instance is the friendly edit:
move a prop by editing its transform, not its geometry ([C16.2](02-models-instances.md)).

## 16.3 The instance is an AABB + transform + index

The 64-byte `SceneryInstance` leads with the prop's world-space **bounding box** (min at `+0x00`, max at
`+0x0C`, both `f32[3]`, Z-up) — the same AABB the cull tree indexes — followed by transform data and, at
`+0x3E`, the **`u16` info index** that selects the model. Verified: all 77,783 retail instances carry an info
index in range. Moving a prop means shifting *both* the transform and the AABB by the same delta
([C16.3](03-instance-record.md)).

## 16.4 The info defines the model

The 72-byte `SceneryInfo` is the model side: the link to the solid, the draw distance / LOD fade, and flags
that apply to every instance of that model at once. Change an info's draw distance and every lamppost in the
block fades at the new range ([C16.4](04-info-record.md)).

## 16.5 The cull tree partitions the props

`SceneryTreeNodes` is a per-section AABB tree with **fanout ≤ 5**. Each 36-byte node holds an AABB (min
`+0x00`, max `+0x0C`) and five `i16` entries at `+0x1A`: a non-negative entry is an **instance index**, a
negative entry is a **child node** (`−value`); node 0 is the root. The leaves form an **exact partition** of
the section's instances — every prop referenced once, verified across all 947 sections. That partition is what
makes visibility "descend a few nodes" instead of "test 77,783 boxes" ([C16.5](05-cull-tree.md)).

---

### Key takeaways

- A section's scenery is `SceneryInfos` (72 B model defs) + `SceneryInstances` (64 B placements) +
  `SceneryTreeNodes` (36 B cull nodes).
- Models and instances are split: one info, many instances — edit instances to move/add/remove props.
- The 64-byte instance is an AABB (`+0x00`/`+0x0C`) + transform + a `u16` info index (`+0x3E`); all 77,783
  retail instances are in range.
- The 72-byte info defines the model (solid link, draw distance, flags) for all its instances.
- The 36-byte cull-tree node (fanout ≤ 5, node 0 = root) partitions instances into leaves exactly once —
  verified across 947 sections.

**Next:** [Chapter 17 — Trigger Regions & Barriers](../C17-Triggers-Barriers/C17-Triggers-Barriers.md): the
gameplay volumes layered over the same world.
