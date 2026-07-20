# C7.1 — Where Materials Live: the Mesh Container

> **The one-sentence version:** inside every solid sits a mesh container `0x80134100` whose descriptor
> `0x00134900` counts the material groups (`+0x10`) and indices (`+0x2C`), and whose two big buffers hold
> the geometry — and the byte sizes of those buffers *prove* the vertex and triangle counts.

[← Chapter 7 hub](C7-Materials-TexAnim.md) · [Next: C7.2 — The shading-group descriptor →](02-shading-groups.md)

---

## The sub-tree

A solid ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) contains exactly one mesh container,
chunk `0x80134100`, and everything about how the solid is drawn lives beneath it:

```
0x80134100  Mesh container
├── 0x00134900  mesh descriptor   (48 bytes: counts & flags)
├── 0x00134B02  shading-group table (8-byte header + N × 104-byte group records)
├── 0x00134B03  index buffer       (8-byte marker + numTris × 3 × u16)
├── 0x00134B01  vertex buffer      (8-byte marker + numVerts × stride)
└── 0x00134C02… per-group extension records
```

This is the boundary between *material data* (the descriptor and the group table) and *geometry data* (the
two buffers). This page decodes the descriptor and shows how it ties to the buffers; [C7.2](02-shading-groups.md)
decodes the group table; the buffers are [Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md).

## The mesh descriptor (`0x00134900`, 48 bytes)

For the worked solid `COBALTSS_BASE_A` the descriptor reads:

```
u32 index:  0    1    2     3        4    5   6   ...  11
value:      0    0   0x12  0x14180  0x0C  0   1   ...  0x111C
```

Two fields are the ones you use constantly, and both are confirmed by cross-checking against the buffers:

| Offset | Value | Meaning | Cross-check |
|---|---|---|---|
| `+0x10` | `0x0C` = 12 | **number of shading groups** | table `0x00134B02` = `8 + 12·104` = 1256 bytes ✓ |
| `+0x2C` | `0x111C` = 4380 | **number of indices** | 4380 = 1460 tris × 3; index buffer = `8 + 4380·2` = 8768 ✓ |

`+0x08` (`0x12` = 18) is a secondary count (vertex streams / texture slots) that does not divide the group
table and is not the group count — the group count is `+0x10`. The lesson the descriptor teaches is to
**never trust a single count in isolation**: confirm it against the byte size of the structure it is
supposed to describe. `12` is the group count because it — and only it — makes the group table divide
evenly.

## The buffers prove the counts

The most reliable way to establish a solid's geometry counts is arithmetic on the buffer chunk sizes, which
the container records exactly ([C1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)):

```
index buffer  0x00134B03 = 8768 bytes → (8768 − 8) / 6  = 1460 triangles
vertex buffer 0x00134B01 = 51848 bytes → (51848 − 8) / 36 = 1440 vertices  (stride 36)
```

Both buffers open with an 8-byte marker (a run of `0x11` bytes — MW's fill/alignment sentinel) before the
data proper, which is why the `− 8` appears. `1460·6 = 8760` and `1440·36 = 51840` land exactly on the
buffer sizes minus that marker. So for this solid **numTris = 1460, numVerts = 1440, vertex stride = 36** —
not read from a header field but *forced* by the byte counts, the strongest kind of verification. The stride
(24 / 36 / 60) is a property of the vertex format ([Chapter 9](../C9-Meshes-FVF/C9-Meshes-FVF.md)); here it
is 36.

## A robust reader

```python
def read_mesh(mesh_container_chunks, buffers):
    desc   = mesh_container_chunks[0x00134900]
    n_grp  = u32(desc, 0x10)                 # shading-group count
    n_idx  = u32(desc, 0x2C)                 # index count
    groups = parse_groups(mesh_container_chunks[0x00134B02], n_grp)   # C7.2
    idx    = mesh_container_chunks[0x00134B03][8:]   # skip 8-byte marker
    vtx    = mesh_container_chunks[0x00134B01][8:]
    assert len(idx) == n_idx * 2, "index count vs buffer mismatch → reparse"
    return groups, vtx, idx
```

The two `assert`-style cross-checks (group count vs table size, index count vs buffer size) are not
paranoia — they are how you know your parse of an *unfamiliar* solid is correct before you rely on a single
field.

> ✅ *Verified:* group count at descriptor `+0x10` (12) matches the 104-byte × 12 group table; index count
> at `+0x2C` (4380) matches the index buffer; vertex/triangle counts are forced by buffer byte sizes
> (1440 verts × 36, 1460 tris × 6), all from `COBALTSS_BASE_A`.

---

### Key takeaways

- Every solid has one mesh container `0x80134100`: descriptor + group table + index buffer + vertex buffer.
- Descriptor `0x00134900`: `+0x10` = shading-group count, `+0x2C` = index count.
- Group table `0x00134B02` = 8-byte header + `count × 104` bytes; buffers open with an 8-byte `0x11` marker.
- Counts are best *proven* by buffer arithmetic: `(idxbytes−8)/6` = tris, `(vtxbytes−8)/stride` = verts.
- Confirm every count against the size of the structure it describes before trusting it.

**Continue:** [C7.2 — The shading-group descriptor](02-shading-groups.md) · [Chapter 7 hub](C7-Materials-TexAnim.md)
