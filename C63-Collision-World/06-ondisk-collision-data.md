# C63.6 — The On-Disk Collision Data

> **The one-sentence version:** the runtime collision world ([C63.1](01-collision-world.md)–[C63.5](05-reading-collision-world.md))
> is *fed* by collision data baked into every stream section — three chunk families (terrain mesh `0x00034159`,
> wall/object `WCollisionPack` `0x0003B801`, smackable spawners `0x00034027`) dispatched through the loader at
> `0x45D600`, each aligning its records to a 16-byte boundary by the rule `(ptr + 0x17) & ~0xF`.

[← C63.5 — Reading the collision world](05-reading-collision-world.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: C63.7 — Terrain collision mesh →](07-terrain-collision.md)

---

## Runtime vs. data

Pages [C63.1](01-collision-world.md)–[C63.5](05-reading-collision-world.md) decoded the collision world as a
*runtime* machine — the broad-phase `Grid`, the narrow-phase test, the `CollisionCache`. But that machine is *empty*
until it's *loaded*: the geometry a car actually collides against is **baked into the stream sections**
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) and streamed in as the player drives
([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)). This page and the next
three decode that *on-disk collision data* — the actual bytes that become the collision world.

The distinction matters: a chapter that only describes the runtime (the machine) without the data (what it eats) is
half a chapter. The runtime is *the same code* for every section; the *data* is what makes Rockport's specific walls,
hills, and lampposts collidable. Reading the collision world completely means reading *both* — and the data side is
where byte-level verification ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) bites
hardest, because it's a concrete format that either round-trips byte-for-byte or doesn't.

> ✅ *Verified against retail v1.3.* The collision data was decoded against the retail world stream
> (`TRACKS/L2RA.BUN` + `STREAML2RA.BUN`, the Rockport free-roam world) and the `speed.exe` chunk loaders, and
> **rebuilds byte-for-byte identical** across the world's **720 stream sections** — 435 terrain meshes (51,504
> triangles), 390 `WCollisionPack` containers (7,934 collision articles), and 366 smackable packs (13,484 records).
> A format that rebuilds bit-exact is a format that is *fully* understood.

## Three collision chunk families

Each stream section ([C15.2](../C15-Track-Streaming/02-section-table.md)) is a run of tagged chunks
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — an 8-byte header (`u32 id`, `u32 payloadSize`)
then the payload. Three chunk IDs carry the section's collision data:

| Chunk ID | Family | What it is | Loader |
|---|---|---|---|
| `0x00034159` | Terrain collision mesh | Ground-height triangle soup ([C63.7](07-terrain-collision.md)) | `0x74B3A0` |
| `0x0003B801` | `WCollisionPack` | Wall/object collision the car is stopped by ([C63.8](08-wcollisionpack.md)) | `0x64AD80` |
| `0x00034027` | Smackable spawners | Knock-down props — lampposts, signs ([C63.9](09-smackables-emitters.md)) | `0x6829D0` |
| `0x0003BC00` | FX-emitter placements | Emitter transforms (adjacent, [C63.9](09-smackables-emitters.md)) | — |

Each family answers a *different* physics question. The terrain mesh answers **"how high is the ground here?"** —
the surface the wheels rest on ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)). The `WCollisionPack`
answers **"what walls/objects stop the car?"** — the barriers a body bounces off
([C43.1](../C43-Collision-Contacts/01-detection.md)). The smackables answer **"what can I knock down?"** — the
destructible props ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)). Together they *are* the collidable
world: the floor, the walls, and the breakables.

> ✅ *Verified:* the strings `WCollisionPack`, `WCollisionAssets`, `CollisionInstanceList`, `CollisionObjectList`,
> `Smackable` (×9), and `RBSmackable` are present in `speed.exe`; the three loaders are real functions at the
> addresses above.

## The dispatcher at 0x45D600

When a section streams in, its chunks are walked and each is handed to the **chunk dispatcher at `0x45D600`**, which
routes the chunk to the loader registered for its ID. Each collision loader begins by *testing the chunk ID* — and
the test instruction literally encodes the ID:

```
0x6829D0  smackable loader:  8B 44 24 04   mov  eax,[esp+4]     ; chunk ptr
                             81 38 27 40 03 00  cmp  dword [eax],0x00034027
0x74B3A0  terrain  loader:  8B 44 24 04   mov  eax,[esp+4]
                             81 38 59 41 03 00  cmp  dword [eax],0x00034159
0x64AD80  WCollisionPack:   8B 44 24 04   mov  eax,[esp+4]
                             81 38 01 B8 03 00  cmp  dword [eax],0x0003B801
```

The `cmp dword [eax], imm32` immediate *is* the chunk ID, little-endian: `27 40 03 00` = `0x00034027`,
`59 41 03 00` = `0x00034159`, `01 B8 03 00` = `0x0003B801`. This is the strongest kind of verification
([C50.2](../C50-Verification-Methodology/02-byte-verification.md)): the loader's own dispatch test names the chunk it
handles, in the shipped code. There is no ambiguity about which loader owns which chunk — the byte pattern says so.

> ✅ *Verified:* at file offsets `0x2829D0`, `0x34B3A0`, `0x24AD80` (VA − `0x400000`), each loader's prologue is
> `mov eax,[esp+4]` then `cmp dword [eax], <chunkID>` with the immediate equal to the chunk ID above — the loader
> tests for exactly the chunk it parses.

## The alignment invariant

Every record-bearing collision chunk shares one structural rule — the **16-byte alignment invariant**. After the
8-byte chunk header, the payload begins with a run of `0x11` filler bytes, just enough to push the *first record
header* onto a 16-byte boundary. The loader computes this with:

```
alignedStart = (payloadPtr + 0x17) & ~0xF
```

`0x17` = `8` (chunk header) + `0xF` (round-up bias); `& ~0xF` clears the low 4 bits. The effect: the record data
always lands 16-byte-aligned relative to the section start — and because sections are themselves **2048-aligned** in
the stream ([C15.2](../C15-Track-Streaming/02-section-table.md)), section-relative alignment equals absolute
alignment, so the aligned loads the physics code does (SIMD vector reads) never fault.

This invariant is also the collision data's **crash-class trap**. Because chunks are packed back-to-back, *any* edit
that shifts a chunk by a non-multiple of 16 silently misaligns *every following chunk's records* — producing corrupt
props, ghost collision, and load crashes. The safe way to change collision data is therefore **size-neutral**: keep
every chunk's start offset fixed mod 16 (mod 128 for texture bases), absorbing any size delta into null-padding
chunks rather than letting downstream chunks slide. The three collision families each have their own size-neutral
edit path ([C63.7](07-terrain-collision.md)–[C63.9](09-smackables-emitters.md)) built around this rule.

> 🟡 *Reasoned:* the `(ptr + 0x17) & ~0xF` alignment is read directly from the loaders and holds for every
> record-bearing chunk across the retail world's 720 sections; the "shift-by-non-16 corrupts downstream" failure
> mode is the observed consequence of that invariant, confirmed by byte-for-byte rebuild testing. The alignment
> arithmetic and the 2048-section stride are verified.

## RE implications

- **Runtime vs. data** — [C63.1](01-collision-world.md)–[C63.5](05-reading-collision-world.md) are the machine;
  this cluster ([C63.6](06-ondisk-collision-data.md)–[C63.9](09-smackables-emitters.md)) is the data it loads.
- **Three families** — terrain mesh (`0x00034159`, floor), `WCollisionPack` (`0x0003B801`, walls), smackables
  (`0x00034027`, breakables).
- **The dispatcher** at `0x45D600` routes by chunk ID; each loader's `cmp` immediate *is* the ID.
- **The alignment invariant** — `(ptr + 0x17) & ~0xF`; records are 16-byte aligned, sections 2048-aligned; edits
  must be size-neutral.

---

### Key takeaways

- The runtime collision world is **empty until loaded** — the geometry a car collides against is **baked into the
  stream sections** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) as on-disk collision data; this
  cluster decodes those bytes.
- **Three chunk families** carry it: **terrain mesh** (`0x00034159`, the floor), **`WCollisionPack`** (`0x0003B801`,
  the walls/objects), and **smackable spawners** (`0x00034027`, the breakables) — plus adjacent FX-emitter
  placements (`0x0003BC00`).
- Chunks are routed through the **dispatcher at `0x45D600`**; each loader begins `cmp dword [eax], <chunkID>` — the
  immediate **literally encodes the chunk ID** (`27 40 03 00` = `0x00034027`), the strongest verification there is.
- Every record-bearing chunk obeys the **16-byte alignment invariant** `(ptr + 0x17) & ~0xF`; sections are
  **2048-aligned**, so edits must be **size-neutral** or every downstream chunk misaligns (corrupt props, crashes).
- The format **rebuilds byte-for-byte** across the retail world's **720 sections** — the proof it is *fully*
  understood, not merely sketched.

**Continue:** [C63.7 — Terrain collision mesh](07-terrain-collision.md) · [Chapter 63 hub](C63-Collision-World.md)
