# C16.3 — The 64-byte SceneryInstance

> **The one-sentence version:** each placement is 64 bytes — the prop's world-space AABB (min `+0x00`, max `+0x0C`),
> two flag words, the world position (`+0x20`), a quantized `int16` 3×3 rotation matrix (`+0x2C`, ÷ 8192), and the
> `u16` info index (`+0x3E`) — a fully-decoded record, so an edit shifts the AABB *and* the transform together.

[← C16.2 — Models vs instances](02-models-instances.md) · [Chapter 16 hub](C16-Scenery-Cull.md) ·
[Next: C16.4 — The 72-byte SceneryInfo →](04-info-record.md)

---

## The record

`SceneryInstance` is the `0x00034103` chunk's payload (16-byte-aligned via the `0x11` pad,
[C63.6](../C63-Collision-World/06-ondisk-collision-data.md)), 64 bytes per record, decoded against the full retail
set (77,783 instances):

| Offset | Type | Field |
|---|---|---|
| `+0x00` | `f32[3]` | **AABB min** (world, Z-up) |
| `+0x0C` | `f32[3]` | **AABB max** (world, Z-up) |
| `+0x18` | `u32` | **flags A** |
| `+0x1C` | `u32` | **flags B** |
| `+0x20` | `f32[3]` | **position** (world, Z-up) |
| `+0x2C` | `int16[9]` | **rotation** — 3×3 matrix, each element ÷ 8192 |
| `+0x3E` | `u16` | **info index** → `SceneryInfos` ([C16.4](04-info-record.md)) |

The record opens with the same axis-aligned bounding box the cull tree indexes ([C16.5](05-cull-tree.md)) — which is
why the tree can partition instances without dereferencing their models: the box it needs is right there in the
instance. The middle bytes are the full pose — two flag words, the world position, and the orientation — and the
`u16` info index at the end selects the model ([C16.2](02-models-instances.md)).

> ✅ *Verified:* the chunk is `0x00034103` (loader dispatch `cmp dword [edx], 0x00034103` in `speed.exe`, alongside
> `SceneryHeader` `0x00034101` and `SceneryInfos` `0x00034102`); the 64-byte stride, leading AABB, flag words,
> position, `int16` 3×3 rotation, and `u16` info index (77,783/77,783 in range) all round-trip against retail data.

## The quantized rotation

The orientation packs a **3×3 rotation matrix as nine `int16`s**, each divided by **8192** (= 2¹³) to recover the
matrix element — so the stored range ±32768 maps to ±4.0, ample headroom for the ±1 components of a rotation matrix.
Nine 16-bit values (18 bytes) hold the full orientation at ~0.0001 precision, a third the size of nine `f32`s.

The matrix is stored **row-major (D3D row-vector layout)** and must be read in **straight memory order** — the *k*-th
`int16` fills matrix slot `[k/3][k%3]` — the same convention the object transforms use
([C1.6](../C1-EAGL-Container-Model/06-matrices-and-coordinates.md)). This is not a detail to skip: reading it
element-transposed yields the *inverse* rotation, which turns every rotated prop the wrong way. The memory copy *is*
the convention conversion — copy the nine values straight across, divide by 8192, done.

```python
def read_rotation(rec):                    # rec = 64-byte instance
    q = struct.unpack_from("<9h", rec, 0x2C)   # nine int16, memory order
    return [[q[0]/8192, q[1]/8192, q[2]/8192],
            [q[3]/8192, q[4]/8192, q[5]/8192],
            [q[6]/8192, q[7]/8192, q[8]/8192]]
```

> ✅ *Verified:* the rotation is `int16[9]` at `+0x2C` with scale 8192, read in straight memory order (row-major);
> the round-trip (read ÷ 8192, write × 8192 with clamp) reproduces retail bytes exactly.

## The AABB is load-bearing

The leading AABB is not a cache — it is the instance's spatial identity:

- **The cull tree indexes it.** A node's box must contain its instances' boxes ([C16.5](05-cull-tree.md)); the tree
  reads `+0x00`/`+0x0C` directly.
- **It must enclose the placed model.** The box is the model's bounds *at this instance's transform* — the
  world-space extent of this placed copy (position `+0x20` + rotation `+0x2C`).
- **It moves with the prop.** Shift the position without shifting the AABB and the culler tests the old location —
  the prop pops in and out incorrectly.

So the box and the transform are a matched pair: they describe the same placement from two angles (extent and pose),
and an edit must keep them consistent.

## Moving, rotating, scaling

Now that the transform is fully decoded, each operation has an exact rule ([C16.6](06-editing-scenery.md)):

- **Move** (translate by Δ): add Δ to the **position** (`+0x20`) *and* to both AABB corners (`+0x00`, `+0x0C`). The
  box shifts with the prop; extent is unchanged.
- **Rotate**: rewrite the **`int16` 3×3** (`+0x2C`, × 8192), then **recompute** the AABB from the model's local
  bounds under the new rotation — a rotation changes the world-space extent, so the old box no longer fits.

```python
def move_instance(rec, delta):             # delta = (dx,dy,dz), Z-up
    px,py,pz = struct.unpack_from("<3f", rec, 0x20)
    struct.pack_into("<3f", rec, 0x20, px+delta[0], py+delta[1], pz+delta[2])
    mn = struct.unpack_from("<3f", rec, 0x00)
    mx = struct.unpack_from("<3f", rec, 0x0C)
    struct.pack_into("<3f", rec, 0x00, *(a+b for a,b in zip(mn,delta)))
    struct.pack_into("<3f", rec, 0x0C, *(a+b for a,b in zip(mx,delta)))
```

Translation is the safe, common case — the extent is unchanged, s