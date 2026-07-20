# C63.9 — Smackables & FX Placements (`0x00034027`, `0x0003BC00`)

> **The one-sentence version:** smackable spawners (`0x00034027`, loader `0x6829D0`) are the *physics* records for
> knock-down props — 64-byte entries with a swizzled stored-frame position, an asset hash, and a vault parameter key,
> tracked by a per-section 2048-bit "already smashed" mask — while the neighbouring `0x0003BC00` chunk places FX
> emitters by a 4×4 world transform.

[← C63.8 — Wall & object collision](08-wcollisionpack.md) · [Chapter 63 hub](C63-Collision-World.md) ·
[Next: Chapter 64 — World Update →](../C64-World-Update/C64-World-Update.md)

---

## The physics of breakable props

Drive through a lamppost, a road sign, or a phone box and it topples — that's a **smackable**. The smackable
spawner chunk (`0x00034027`, loader `0x6829D0`, per-record processor `0x6828D0`) is the *physics* half of a
destructible prop. The *visual* half is an ordinary scenery instance
([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)); the smackable record is **what the physics hits and
breaks**. The two are paired — a lamppost you can knock down is a scenery instance (the mesh you see) plus a
smackable record (the collidable, breakable body) at the same spot.

This is why smackables live in the collision cluster and not with scenery: they're collision *bodies*, driven by the
rigid-body smackable code the engine names `Smackable` / `RBSmackable` ([C43.5](../C43-Collision-Contacts/05-smackables.md)).
When a car contacts one, the narrow-phase ([C63.3](03-narrow-phase.md)) produces the hit, and the smackable's
parameters decide how it yields — topple, shatter, or resist.

> ✅ *Verified:* the loader at `0x6829D0` tests `cmp dword [eax], 0x00034027` (per-record processor at `0x6828D0`);
> `Smackable` (×9) and `RBSmackable` are strings in `speed.exe`. The retail world carries **366 smackable packs,
> 13,484 records**, rebuilding byte-for-byte.

## The record layout

After the `0x11` padding ([C63.6](06-ondisk-collision-data.md)), a 16-byte header then 64-byte records:

```
16-byte header:
  +0x00  u32  versionA (0x0B)   +0x04  u32  versionB (0x0B)
  +0x08  s16  sectionId         +0x0A  s16  firstLocalIndex   (per-section base)
  +0x0C  s16  recordCount       +0x0E  u8   runtimeFlag, u8 0
then recordCount x 64-byte records:
  +0x00  u32  0
  +0x04  f32[3]  direction   (stored frame, unit length)
  +0x10  f32[3]  position    (stored frame)
  +0x1C  f32  1.0
  +0x20  u32  assetHash      +0x24  u32  assetHash   (repeated)
  +0x28  u32  paramHash      (vault smackable parameter key)
  +0x2C  u16  globalId       +0x2E  u16  0
  +0x30  u32  localIndex     (firstLocalIndex + n)
  +0x34  u32  0   +0x38  f32  scale/size   +0x3C  u32  0
```

Three fields tie the record into the rest of the engine. **`assetHash`** (stored twice) is the bin-name hash
([C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)) of the prop's asset — which mesh/model it is.
**`paramHash`** is a **vault key** into the smackable parameter category ([C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md))
— it names the *behaviour* (mass, break threshold, how it topples), so many lampposts can share one parameter set by
sharing a `paramHash`. **`localIndex`** = `firstLocalIndex + n` gives each record a stable per-section index — which
matters for the smashed-mask ([below](#the-already-smashed-mask)).

## The swizzled stored frame

Positions and directions are stored in a **permuted axis frame**, not world coordinates. The loader swizzles on
load:

```
stored -> world:   world  = (  s.z, -s.x,  s.y )
world -> stored:   stored = ( -w.y,  w.z,  w.x )
```

So a stored vector `(sx, sy, sz)` becomes world `(sz, -sx, sy)`. The book's world frame is **Z-up, game-native**
([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) and must not be re-oriented; the smackable *stored* frame is a
different axis convention that the loader maps into it. Anyone reading or editing smackable positions must apply this
swizzle — a raw read of `+0x10` is in the stored frame, not world space. (This was confirmed statistically: under
this mapping, every record's position lands inside its own section's scenery neighbourhood; under the identity
mapping it does not.)

> 🟡 *Reasoned:* the exact stored→world swizzle `(s.z, -s.x, s.y)` is verified by round-trip and by positions
> landing in-neighbourhood across all 13,484 records; the underlying reason (the frame the exporter emitted) is
> historical. The record layout, `assetHash`/`paramHash`/`localIndex` roles, and the byte-for-byte rebuild are
> verified.

## The already-smashed mask

The header's `firstLocalIndex` and the records' `localIndex` exist to serve a runtime structure: a **per-section
2048-bit mask** of which smackables have *already been smashed*, indexed by record order. When you topple a lamppost,
the game sets that record's bit; the prop stays down (and out of collision) for the rest of the session without
re-spawning. The mask is a fixed 2048 bits per section, which imposes a hard rule: **a section can hold at most 2048
smackables**, and record order must be preserved so the bits keep pointing at the right props.

This is why the on-disk `localIndex` is **preserved, never re-sequenced**, on a rebuild: the smashed-mask bit for
record *k* is meaningless if record *k* moves. It's a small but sharp example of the collision data encoding a
*runtime contract* — the file's index field exists because the runtime keeps a bitmask keyed on it.

> 🟡 *Reasoned:* the per-section 2048-bit smashed-mask indexed by record order (hence the ≤ 2048 record ceiling and
> the preserve-index rule) is read from the runtime flag/index fields and the loader's use of them; the mask's exact
> storage is per-function RE. The `firstLocalIndex`/`localIndex` fields and their preservation requirement are
> verified.

## The adjacent FX-emitter chunk (`0x0003BC00`)

Sharing the collision-data neighbourhood is chunk `0x0003BC00` — **FX-emitter placements**. It's not collision, but
it's decoded here because it rides alongside: a 16-byte header `{ u32 0, u32 5, u32 count, u32 sectionId }` then
`count` × 80-byte records:

```
+0x00  u32  emitterHash   (attributes.bin emitter key)
+0x04  u32  0
+0x08  u32  sectionId
+0x0C  u32  0
+0x10  f32[16]  row-major 4x4 world transform  (position in row 3, w = 1)
```

Each record places a world FX emitter ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)) — steam,
sparks, ambient particle sources — by an `emitterHash` (a vault emitter key) and a full 4×4 world transform (identity
rotation observed in retail, so effectively a positioned emitter). It's the *placement* layer for ambient world FX,
carried in the same section-side data as collision. (Present in 290 retail sections; 21 carry an extra undecoded
trailer, preserved verbatim.)

> ✅ *Verified:* `0x0003BC00` records are 80 bytes with `emitterHash` at `+0x00`, `sectionId` at `+0x08`, and a
> row-major 4×4 transform at `+0x10`, across the 290 retail sections that carry the chunk; positions land inside
> their section's neighbourhood.

## RE implications

- **Smackables** (`0x00034027`, loader `0x6829D0`) are the **physics records for knock-down props** — paired with a
  scenery instance ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) for the visual.
- **64-byte records** — `assetHash` (which model), `paramHash` (vault behaviour key,
  [C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md)), `localIndex` (stable per-section index).
- **Swizzled stored frame** — `world = (s.z, -s.x, s.y)`; raw reads are *not* world space.
- **2048-bit smashed mask** — per section, indexed by record order → ≤ 2048 records, preserve `localIndex`.
- **`0x0003BC00`** — adjacent FX-emitter placements: 80-byte records, `emitterHash` + 4×4 world transform.

---

### Key takeaways

- **Smackables** (`0x00034027`) are the **physics half of a destructible prop** — the visual is a scenery instance
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)), the smackable record is **what the car hits and breaks**,
  driven by the `Smackable`/`RBSmackable` code.
- Each **64-byte record** carries an **`assetHash`** (which model), a **`paramHash`** (a **vault key** naming the
  break behaviour, [C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md)), and a stable **`localIndex`**.
- Positions use a **swizzled stored frame** — `world = (s.z, -s.x, s.y)` — so a raw read of `+0x10` is *not* world
  space; the loader swizzles into the game's Z-up world.
- A **per-section 2048-bit "already smashed" mask** keyed on record order tracks toppled props — imposing a **≤ 2048
  smackables per section** ceiling and a **preserve-`localIndex`** rule on any rebuild (a runtime contract encoded in
  the file).
- The **adjacent `0x0003BC00`** chunk places **FX emitters** ([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md))
  — 80-byte records with an `emitterHash` and a **4×4 world transform** — ambient-FX placement carried in the same
  section-side data.

**Next:** [Chapter 64 — World Update: Bodies, Effects & the Active Lists](../C64-World-Update/C64-World-Update.md) —
how the loaded collision bodies are advanced each tick.

**Sources:** retail `speed.exe` v1.3 and the retail world stream (`TRACKS/L2RA.BUN` + `STREAML2RA.BUN`). Verified:
loader `0x6829D0` (`cmp [eax],0x00034027`), processor `0x6828D0`; `Smackable`/`RBSmackable` strings; 64-byte record
layout; stored↔world swizzle; `0x0003BC00` 80-byte emitter records. Byte-for-byte rebuild across 366 smackable packs
/ 13,484 records and 290 emitter sections. Vault parameter keys: [Chapter 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md).
