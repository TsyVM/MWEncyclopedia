# Chapter 7 — Materials, Texture References & Animation

> **Goal of this chapter:** follow the thread from a piece of geometry to the pixels on its surface — how a
> solid's mesh is split into material groups, how each group names the texture it wants, and how that name
> becomes one of the keyed textures Chapter 5 taught you to find.

Chapters 5 and 6 gave you textures as self-contained objects: a keyed pack of images you can locate, decode,
and edit. This chapter is the **binding layer** — the data that says "this triangle is drawn with *that*
texture, using *these* material parameters." It lives not in the TPK but inside the geometry, in the mesh
container of every solid, and it is the reason a car's windows are glass and its body is paint.

> **Verified against retail data.** The structures here were parsed from `CARS/COBALTSS/GEOMETRY.BIN`, a
> single-`SolidList` car bundle. Its first solid, `COBALTSS_BASE_A`, has 1460 triangles and 1440 vertices,
> and carries readable material names — `DEFAULT`, and usage strings such as `BODY_ALUMINUM`,
> `BODY_DULL_PLASTIC`, `BODY_MOLDING`, `WINDOW_FRONT`, `WINDOW_LEFT_REAR`, `WINDOW_RIGHT_REAR` — which anchor
> every claim below.

---

## Deep-dive pages

- [C7.1 — Where materials live: the mesh container](01-mesh-container.md): the `0x80134100` sub-tree of a
  solid, and the descriptor (`0x00134900`) that counts its material groups.
- [C7.2 — The shading-group descriptor](02-shading-groups.md): the per-group record in `0x00134B02` — its
  color/material parameters and the vertex/triangle range it draws.
- [C7.3 — Binding a texture by asset key](03-texture-binding.md): how a group names its texture, why that
  name is the **asset hash** (not the reflection hash), and how the engine resolves it against a bound TPK.
- [C7.4 — Material usage-name strings](04-usage-names.md): the human-readable slot names (`BODY_*`,
  `WINDOW_*`) carried in the geometry, what they mean, and how they map to shaders and paint.
- [C7.5 — The two hash worlds](05-two-hash-worlds.md): the crucial distinction — the asset hash (texture
  keys, object names) versus the reflection hash (attribute fields) — proven by data.
- [C7.6 — Texture animation](06-texture-animation.md): how scrolling/flipbook textures are expressed, the
  mechanism, and what is and isn't recoverable from the file.

---

## 7.1 The binding lives in the geometry

A solid ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) is not one uniform surface — it is a set
of triangles partitioned into **material groups** (often called shading groups), each drawn with one texture
and one set of material parameters. That partition, and the texture each partition wants, is stored inside
the solid's mesh container `0x80134100`:

```
0x80134100  Mesh container (one per solid)
├── 0x00134900  mesh descriptor   (counts: material groups, index count, flags)
├── 0x00134B02  shading-group table (per-group material params + texture reference)
├── 0x00134B03  index buffer       (8-byte marker + numTris × 3 × u16)
├── 0x00134B01  vertex buffer      (8-byte marker + numVerts × stride)
└── 0x00134C02… per-group extension records (bounds/params/names)
```

The descriptor and the shading-group table are the material data; the two buffers are the geometry
([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)). Verified sizes for `COBALTSS_BASE_A` tie the two together
exactly: the index buffer is `8 + 1460·6 = 8768` bytes and the vertex buffer is `8 + 1440·36 = 51848` bytes,
so `numTris = 1460` and `numVerts = 1440` are not guesses — they are forced by the byte counts
([C7.1](01-mesh-container.md)).

## 7.2 A material group is a range plus a look

Each shading group says two things: **which triangles** it owns (a contiguous range in the index buffer)
and **how they look** (a texture reference plus material parameters — diffuse color, specular, flags). The
engine draws the solid group by group: bind the group's texture, set its parameters, draw its index range.
That is why a car body and its windows can share one mesh yet look completely different — they are separate
groups with separate textures. The per-group record and its fields are [C7.2](02-shading-groups.md).

## 7.3 Textures are named by asset key

A group does not embed its texture; it **references** it by the same 32-bit key Chapter 5 established
([C5.6](../C5-Textures-TPK/06-the-texture-key.md)). At load time the engine has a TPK bound (the car's
texture pack, or a shared world pack), and it resolves the group's key against that pack's hash table to get
the actual texture. This indirection is what lets the same geometry be re-skinned: swap the bound pack, keep
the keys, and the car changes livery without touching a triangle. The resolution model is
[C7.3](03-texture-binding.md).

## 7.4 The names are meaningful

Alongside the numeric keys, the geometry carries **human-readable usage names** — `BODY_ALUMINUM`,
`BODY_DULL_PLASTIC`, `BODY_MOLDING`, `WINDOW_FRONT`, and so on. These are the art pipeline's semantic slot
names: they tell the shader system what *kind* of surface a group is (painted metal, dull plastic, glass),
which drives how the car's chosen paint color and environment reflections are applied. `DEFAULT` is the
fallback material. These strings are your Rosetta Stone when reverse-engineering an unfamiliar solid —
[C7.4](04-usage-names.md).

## 7.5 Two different hashes, and why it matters

The single most important thing to carry out of this chapter is that MW has **two distinct hash worlds**,
and confusing them wastes hours:

- The **asset hash** identifies content — texture keys, geometry object names (`COBALTSS_BASE_A`), world
  asset IDs. It is deterministic per name but matches no standard string hash, because it is minted by the
  offline packer ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)).
- The **reflection hash** identifies *attribute fields* in the vault system — and it *is* a known function
  (lookup2 with a fixed seed), recoverable and re-computable
  ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)).

A texture key and an attribute-field id look identical on the page — both are 32-bit hex — but they come
from different functions and index different systems. [C7.5](05-two-hash-worlds.md) proves the split with
data and tells you which is which.

## 7.6 Animated textures

Some surfaces animate — scrolling road-edge lights, flipbook effects, shimmering signage. MW expresses this
by animating the *reference* (which texture, or which UV offset, a group samples over time) rather than by
rewriting pixels. [C7.6](06-texture-animation.md) covers the mechanism and is candid about which parts are
byte-verified and which are reasoned from behaviour.

---

### Key takeaways

- Material/texture binding lives inside the solid's mesh container `0x80134100`, not in the TPK.
- A solid's triangles are partitioned into **shading groups**; each group = an index range + a texture
  reference + material parameters.
- Textures are referenced by the 32-bit **asset key** and resolved against a bound TPK at load — the basis
  of re-skinning.
- Geometry carries readable **usage names** (`BODY_*`, `WINDOW_*`, `DEFAULT`) that drive shader/paint
  behaviour.
- MW has two hash worlds — asset hash (content) vs reflection hash (attribute fields); never conflate them.

**Next:** [Chapter 8 — 3D Geometry: Solid Lists & Objects](../C8-Geometry-Solids/C8-Geometry-Solids.md): the
container these materials hang inside, decoded object by object.
