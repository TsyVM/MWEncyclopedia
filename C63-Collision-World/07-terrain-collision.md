# C63.7 — Terrain Collision Mesh (`0x00034159`)

> **The one-sentence version:** the terrain collision chunk is a quantized triangle *soup* — `s16` vertices
> dequantized by fixed `/4, /4, /16` — that answers the wheels' ground-height query (`0x74B180`) by 2D
> point-in-triangle; it's the *floor* the car rides on, and it holds *only* the ground (props and buildings live in
> the other collision families).

[← C63.6 — The on-disk collision data](06-ondisk-collision-data.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: C63.8 — Wall & object collision →](08-wcollisionpack.md)

---

## The floor of the world

The single most-queried piece of collision data is the **ground height** — every wheel, every frame, asks *"how
high is the terrain under me?"* to place the suspension ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)).
That question is answered by the **terrain collision mesh**, chunk `0x00034159`: a triangle mesh of the drivable
ground, one per stream section, loaded by the handler at `0x74B3A0` and queried by the height function at `0x74B180`.

It is deliberately *not* the render terrain ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)). The visual
ground has thousands of triangles, textures, and detail; the *collision* ground is a coarse soup of **117–165
triangles per section** — just enough to define a height field the wheels can sample cheaply. Separating collision
geometry from render geometry ([C63.1](01-collision-world.md)) is the whole reason a car can sample the ground at
1000 Hz without touching the GPU's meshes.

> ✅ *Verified:* the loader at `0x74B3A0` tests `cmp dword [eax], 0x00034159` and links the payload into the runtime
> list at `0x9B0BC0`; the ground-height query is at `0x74B180`. The retail world carries **435 terrain meshes,
> 51,504 triangles total**, and they rebuild byte-for-byte.

## The chunk layout

After the 8-byte chunk header and the `0x11` alignment padding ([C63.6](06-ondisk-collision-data.md)), the payload
is a 24-byte header then a run of 18-byte triangles:

```
24-byte header:
  +0x00  u32  versionA      (0x0B)
  +0x04  u32  versionB      (0x0B)
  +0x08  u32  sectionId
  +0x0C  u32  triangleCount
  +0x10  u64  0             (runtime list links — zero on disk)
then triangleCount x 18-byte triangles:
  { s16 x[3], s16 y[3], s16 z[3] }     // three verts, SoA per-axis
then 2 pad bytes.
```

Note the layout is **structure-of-arrays per triangle**: the three X's, then the three Y's, then the three Z's —
not three interleaved `(x,y,z)` vertices. Each triangle is a self-contained 18 bytes (9 × `s16`); there is no shared
vertex/index buffer, which is why it's a *soup* rather than an indexed mesh. The `+0x10` qword is zeroed on disk and
overwritten at load time with the runtime linked-list pointers that thread the section's mesh into the global list
at `0x9B0BC0`.

## Quantization: s16 and fixed dequant

Vertices are stored as **16-bit signed integers** and dequantized to world units by *fixed* divisors:

```
worldX = x / 4.0        worldY = y / 4.0        worldZ = z / 16.0
```

So X and Y have a resolution of **0.25 m** (`s16` range ±32767 → ±8191 m, comfortably spanning a section) and Z has
a *finer* **0.0625 m** resolution — height needs more precision than horizontal position, because the wheels are
sensitive to small bumps. The engine stores the *reciprocals* as constants and multiplies:

```
0x890E98 = 4.0        0x8910F0 = 0.25 (= 1/4)        0x8B493C = 0.0625 (= 1/16)
```

Quantizing to `s16` halves the storage versus `f32` and — because the divisors are powers-of-two-friendly — the
dequant is exact for representable values. This is a classic space/precision trade: coarse where it doesn't matter
(horizontal), fine where it does (vertical).

> ✅ *Verified:* the three dequant constants read `4.0`, `0.25`, `0.0625` at file offsets `0x490E98`, `0x4910F0`,
> `0x4B493C` (VA − `0x400000`) in retail `speed.exe` v1.3 — exactly the `/4, /4, /16` mapping.

## The height query: 2D point-in-triangle

The wheels don't want a *mesh* — they want a *number*: the ground height `z` at a given `(x, y)`. The query at
`0x74B180` treats the triangle soup as a **height field**: it finds the triangle whose 2D projection (ignoring `z`)
*contains* the query point, then interpolates the three vertices' `z` by barycentric weight to get the exact ground
height there. Because it's a 2D containment test, the terrain must be **roughly single-valued in z** — one ground
height per horizontal point — which is exactly what a drivable road surface is. Overhangs, tunnels, and walls (which
are *multi-valued* in z) therefore can't live here; they're `WCollisionPack` geometry ([C63.8](08-wcollisionpack.md)).

This is why the terrain mesh is a *soup* and not a general collision mesh: it only needs to answer a 2D-indexed
height lookup, so it can skip adjacency, normals, and indexing. The query walks the section's triangles (a few
hundred at most, already narrowed by the streaming residency to the player's neighbourhood,
[C38.1](../C38-Resource-Streaming-Residency/01-streammgr.md)) and returns the interpolated height — cheap
enough to run per wheel per tick.

> 🟡 *Reasoned:* the 2D point-in-triangle + z-interpolation reading follows from the query function's job (ground
> height from a triangle soup) and the single-valued-in-z structure of the data; the precise walk/acceleration
> inside `0x74B180` is per-function RE. The loader, the list at `0x9B0BC0`, the query address, and the layout are
> verified.

## Ground only — props are elsewhere

A crucial fact for understanding the collision world: the terrain mesh holds **only the ground**. Individual props,
buildings, walls, and street furniture have **zero coverage** in the terrain soup — a lamppost is not a bump in the
height field. This is by design and it *partitions the collision problem* cleanly:

- **The floor** — terrain mesh (this page): where the wheels rest, the drivable surface.
- **The walls/objects** — `WCollisionPack` ([C63.8](08-wcollisionpack.md)): what stops the car horizontally.
- **The breakables** — smackables ([C63.9](09-smackables-emitters.md)): what the car knocks down.

So a car driving into a wall is *not* handled by the terrain mesh at all — the wheels ride the terrain height while
the body is stopped by the `WCollisionPack` geometry. This separation is why the height query can stay a simple 2D
lookup: it never has to reason about vertical obstacles, because those aren't its job. Reading the terrain mesh
correctly means understanding what it *excludes* as much as what it includes — it is the floor, and *only* the floor.

## RE implications

- **Terrain collision** (`0x00034159`, loader `0x74B3A0`) is the **ground-height field** the wheels sample — the
  floor, not the walls.
- **Layout** — 24-byte header (`triangleCount` at `+0x0C`), then 18-byte `s16` triangles (SoA per-axis), 2 pad
  bytes.
- **Quantized** — `s16` dequantized `/4, /4, /16` (constants `4.0 / 0.25 / 0.0625` verified) — coarse XY, fine Z.
- **Height query** (`0x74B180`) — 2D point-in-triangle + z-interpolation; requires single-valued-in-z (drivable
  ground).
- **Ground only** — props/walls have zero coverage here; they're `WCollisionPack`/smackables.

---

### Key takeaways

- The terrain collision mesh (`0x00034159`) is the **floor of the world** — the ground-height field every wheel
  samples each tick ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)), **separate from** the render terrain
  and far coarser (**117–165 triangles per section**).
- **Layout:** 24-byte header (`triangleCount` at `+0x0C`, runtime links zeroed at `+0x10`) then 18-byte triangles
  stored **structure-of-arrays** — `s16 x[3], y[3], z[3]` — a *soup*, not an indexed mesh.
- **Quantized to `s16`** and dequantized by **fixed `/4, /4, /16`** (constants `4.0 / 0.25 / 0.0625`, verified in
  `speed.exe`) — 0.25 m horizontal, 0.0625 m vertical resolution: coarse where it doesn't matter, fine where it
  does.
- The **height query** (`0x74B180`) treats the soup as a height field — **2D point-in-triangle then z-interpolate** —
  which is why terrain must be **single-valued in z** (drivable ground; overhangs/walls can't live here).
- The mesh holds **only the ground** — props, walls, and buildings have **zero coverage** and belong to
  `WCollisionPack` ([C63.8](08-wcollisionpack.md)) and smackables ([C63.9](09-smackables-emitters.md)); the clean
  partition is what keeps the height query a simple 2D lookup.

**Continue:** [C63.8 — Wall & object collision (`WCollisionPack`)](08-wcollisionpack.md) ·
[Chapter 63 hub](C63-Collision-World.md)
