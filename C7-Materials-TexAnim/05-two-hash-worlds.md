# C7.5 — The Two Hash Worlds

> **The one-sentence version:** MW hashes names with two different functions for two different jobs — a
> **reflection hash** (lookup2 with a fixed seed, fully recoverable) for attribute-vault field names, and an
> **asset hash** (deterministic but matching no standard hash, minted by the packer) for texture keys and
> geometry object names — and telling them apart saves you from hours of chasing the wrong algorithm.

[← C7.4 — Material usage-name strings](04-usage-names.md) · [Chapter 7 hub](C7-Materials-TexAnim.md) ·
[Next: C7.6 — Texture animation →](06-texture-animation.md)

---

## Two systems, two hashes

Every 32-bit hash in Most Wanted looks identical on the page — eight hex digits — but there are two distinct
hashing systems, and a value only makes sense inside its own world:

| | Reflection hash | Asset hash |
|---|---|---|
| Identifies | attribute-vault **field names** (`TopSpeed`, `default`) | **content**: texture keys, geometry object names, world asset IDs |
| Function | **lookup2 / Jenkins, seed `0xABCDEF00`** | non-standard; minted by the offline packer |
| Recoverable? | **Yes** — recomputable from the name | **No** — matches no standard string hash |
| Where verified | disassembly of the hash routine ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)) | 2,012-name cross-pack determinism ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)) |

## The reflection hash is known

The vault system ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) identifies each field by
hashing its name, and that function is **known and reproducible**: it is Jenkins' lookup2 seeded with
`0xABCDEF00`, confirmed by disassembling the routine and by reproducing landmark values (for example the
pervasive `default` key `0xEEC2271A`) from the name alone
([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)). Because it is recoverable, you can
*compute* a field's id from its name, which is what makes writing new vault records — not just reading
them — possible.

## The asset hash is not

The content world is different. Texture keys and geometry object names are just as deterministic — the same
name always yields the same value — but they reproduce **none** of the standard hashes. Two independent
proofs, from two different subsystems, converge on this:

- **Texture keys.** 2,012 texture names that recur across packs all carry byte-identical keys (deterministic),
  yet not one matches Joaat, Bin/FNV, CRC-32 across eight polynomials, djb2/sdbm/ELF, or lookup2/3 on any
  input form ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)).
- **Object names.** The solid `COBALTSS_BASE_A` stores the name-hash `0x54DF8EF4`; computing Joaat
  (`0x16EC635A`), Bin (`0x8A995669`), FNV-1a (`0x13A913B4`), CRC-32 (`0xF03B362D`), and lookup2
  (`0xBFE3945B`) of the name all miss.

The consistent explanation is that the asset hash is produced by EA's **offline build pipeline**, whose
routine did not ship in the executable; the runtime only ever *reads and compares* these values, never
regenerates them. So the asset hash is a stable identifier you take from the file, not a formula you apply.

## Why keeping them separate matters

Confusing the two worlds is a classic time-sink:

- If you try to **recompute a texture key or object name** with lookup2/`0xABCDEF00` (the reflection hash),
  you will fail, because content is asset-hashed — and you might wrongly conclude the reflection hash is
  "wrong."
- If you try to **treat an attribute-field id as opaque** and look it up by stored value only, you miss that
  you *can* compute it and therefore *can* author new fields.

The rule is simple: **field names → reflection hash (compute it); content names → asset hash (read it).** When
you meet a 32-bit value, ask which system it belongs to before deciding whether to compute or to look it up.

## Practical guidance

- **Vault/attribute work** ([C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)–[C14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)):
  compute field ids with lookup2/`0xABCDEF00`; you can add and target fields by name.
- **Texture and geometry work** (this chapter, [C5](../C5-Textures-TPK/C5-Textures-TPK.md),
  [C8](../C8-Geometry-Solids/C8-Geometry-Solids.md)): read asset keys/name-hashes from the file and preserve
  them; do not attempt to regenerate them from names.
- **When a value won't reproduce,** that is usually a signal you are in the asset-hash world, not that your
  hash code is broken.

> ✅ *Verified:* reflection hash = lookup2 seed `0xABCDEF00` (disassembly + `0xEEC2271A` reproduction); asset
> hash is deterministic yet unmatched by any standard hash for both texture keys (2,012 names) and object
> names (`COBALTSS_BASE_A` → `0x54DF8EF4`).

---

### Key takeaways

- MW has two hash worlds: **reflection hash** (attribute fields) and **asset hash** (content).
- Reflection hash is lookup2/`0xABCDEF00` — **known and recomputable**; asset hash is packer-minted and
  **not recoverable**.
- Both proofs are data-backed: `0xEEC2271A` reproduced for the reflection hash; 2,012 texture names and
  `COBALTSS_BASE_A`'s `0x54DF8EF4` for the asset hash.
- Rule: compute field ids from names; read content keys from the file. Ask which world a value is in first.
- A value that "won't reproduce" is a hint you're in the asset-hash world.

**Continue:** [C7.6 — Texture animation](06-texture-animation.md) · [Chapter 7 hub](C7-Materials-TexAnim.md)
