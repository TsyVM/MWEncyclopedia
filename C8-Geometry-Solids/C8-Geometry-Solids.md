# Chapter 8 — 3D Geometry: Solid Lists & Objects

> **Goal of this chapter:** open any geometry bundle, enumerate the objects (solids) inside it, and decode
> each object's header — its name, its asset-hash id, its triangle count, its bounding box, and its placement
> matrix — well enough to locate, identify, and navigate a model without yet touching a single triangle.

A **SolidList** is the engine's container of 3D objects: cars, props, world chunks, wheels, everything with
geometry. It is a directory of **solids**, each solid a named mesh with a bounding box and a transform. This
chapter is the *container and the object header* — the map of what is inside and where. The triangles and
vertex formats are [Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md); the materials that skin them were
[Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md); exporting and rebuilding them is
[Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md).

> **Verified against retail data.** Every structure here was parsed from real car geometry, principally
> `CARS/COBALTSS/GEOMETRY.BIN` — a SolidList of **316** objects whose first solid, `COBALTSS_BASE_A`, has
> **1460 triangles** and **1440 vertices** and a bounding box that matches the real dimensions of a compact
> car. The asset-hash findings draw on **1,179+** object names gathered across sixteen car bundles.

---

## Deep-dive pages

- [C8.1 — The SolidList container & directory](01-solidlist-container.md): `0x80134000`, its header block, and
  the three tables (`0x00134002` list header, `0x00134003` sorted hash table, `0x00134004` object directory).
- [C8.2 — The object header, field by field](02-object-header.md): the 176-byte `0x00134011` — flags,
  name-hash, triangle count, bounding box, transform, and the embedded name.
- [C8.3 — Object names & the asset hash](03-object-hash.md): the tail-additive property (sequential names →
  sequential ids), what you can and can't predict, and the sorted-table lookup.
- [C8.4 — Bounding boxes & the Z-up world](04-bounding-boxes.md): reading the AABB, confirming the coordinate
  system from a car's real proportions, and using boxes to identify parts.
- [C8.5 — The placement transform](05-transform.md): the 4×4 matrix, what "identity" means for a car base,
  and how objects are positioned.
- [C8.6 — Finding an object: binary search](06-lookup.md): using the sorted hash table and directory to jump
  straight to a solid by name-hash.
- [C8.7 — Editing solids safely](07-editing.md): the size-tree consequences of changing an object, and what
  must stay consistent across header, tables, and buffers.

---

## 8.1 The container at a glance

A SolidList is chunk `0x80134000`, and its shape is a header block followed by the objects:

```
0x80134000  SolidList
├── 0x80134001  header block
│   ├── 0x00134002  list header   (object count @ +0x0C; source filename, e.g. "GEOMETRY.BIN")
│   ├── 0x00134003  hash table    (N × 8 bytes: {name-hash, pad}; sorted ascending for binary search)
│   └── 0x00134004  object directory (N × 24 bytes: {name-hash, file-offset, size, size, 0, 0})
├── 0x80134010  Solid object #0
│   ├── 0x00134011  object header (176 bytes)
│   ├── 0x00134012 / 0x00134013 / 0x0013401A  aux tables
│   └── 0x80134100  mesh container  (materials + buffers — Chapters 7 & 9)
├── 0x80134010  Solid object #1
│   └── …
```

For the worked bundle the header's object count is `0x13C = 316`, the hash table is `316 × 8 = 2528` bytes,
and the directory is `316 × 24 = 7584` bytes — three independent structures that all agree on 316, your
instant sanity check ([C8.1](01-solidlist-container.md)).

## 8.2 The object header

Each solid opens with a 176-byte header (`0x00134011`) whose layout this book decoded field by field and
verified against `COBALTSS_BASE_A`:

| Offset | Field | Value (worked) |
|---|---|---|
| `+0x0C` | flags / version | `0x00400016` |
| `+0x10` | **name-hash** (asset hash) | `0x54DF8EF4` |
| `+0x14` | **num triangles** | `0x5B4` = 1460 |
| `+0x20` | **bbox min** (x,y,z) | (−2.20, −0.798, 0.530) |
| `+0x30` | **bbox max** (x,y,z) | (+1.27, +0.798, 1.222) |
| `+0x40` | **4×4 transform** | identity (car base) |
| `+0xA0` | **name** (ASCII) | `COBALTSS_BASE_A` |

The triangle count is not taken on faith: `1460 × 6 + 8 = 8768` is exactly the index-buffer size
([C7.1](../C7-Materials-TexAnim/01-mesh-container.md)). Full breakdown: [C8.2](02-object-header.md).

## 8.3 Names, hashes, and the coordinate system

Two facts from the header ripple through everything else. First, the **name-hash** is an *asset-hash* value
([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)) — and this chapter pins down a property of it that no
prior record documented: it is **tail-additive**. `COBALTSS_BASE_A/B/C` hash to
`0x54DF8EF4/F5/F6` — incrementing the last character by one increments the hash by one, confirmed in 97 % of
same-length last-character pairs. That gives a real prediction trick and a real limit
([C8.3](03-object-hash.md)). Second, the **bounding box** proves the world is **Z-up**: the car's box spans
3.47 m in X (length), a symmetric ±0.798 m in Y (width), and 0.53–1.22 m in Z (height above the pivot) —
only a Z-up reading makes those numbers a car ([C8.4](04-bounding-boxes.md)).

---

### Key takeaways

- A SolidList (`0x80134000`) is a header block (list header + sorted hash table + object directory) plus N
  solids (`0x80134010`).
- The object count appears in three places (header field, `hashtable/8`, `directory/24`) — cross-check them.
- The 176-byte object header carries name-hash (`+0x10`), triangle count (`+0x14`), bbox (`+0x20`/`+0x30`),
  transform (`+0x40`), and name (`+0xA0`); the triangle count is confirmed by the index buffer size.
- Object name-hashes are asset-hash values with a verified **tail-additive** property.
- The bounding box confirms the **Z-up** coordinate system from real car proportions.

**Next:** [Chapter 9 — Meshes, FVF & Vertex Formats](../C9-Meshes-FVF/C9-Meshes-FVF.md): decoding the vertex
and index buffers into triangles.
