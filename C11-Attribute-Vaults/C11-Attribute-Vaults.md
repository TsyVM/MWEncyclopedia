# Chapter 11 — Attribute Vaults: VPAK Structure

> **Goal of this chapter:** open `attributes.bin`, recognise it as a **VPAK** container, and map its whole
> anatomy — the header, the `ErtS` string table, the reflection type-name table, the typed data records, and
> the `NpeD`/`NrtS`/`NtaD` trailer blocks — well enough to locate any collection and field before we resolve
> their values in Chapter 12.

Most Wanted is a heavily **data-driven** game: how fast a car accelerates, how aggressive the police are, how
a surface sounds and shatters, what an effect looks like — none of it is hard-coded, all of it lives in
**attribute vaults**. The master vault is `GLOBAL/attributes.bin`, a **VPAK** file, and this chapter is its
container format. Chapter 12 decodes the reflection schema that gives the records meaning; Chapters 13 and 14
tour the actual content (car tuning; pursuit, surfaces, gameplay).

> **Verified against retail data.** Every structure here was parsed from the real `GLOBAL/attributes.bin`
> (689 728 bytes): magic `VPAK`, version 1, an `ErtS` string table of **1 960** names, **16** reflection
> primitive types, and trailer blocks `NpeD`/`NrtS`/`NtaD` carrying the sentinel `0xEFFECADD`. The reflection
> hash is confirmed live: `lookup2("default", seed 0xABCDEF00) = 0xEEC2271A`, and that value appears **1 071**
> times as the universal parent reference.

---

## Deep-dive pages

- [C11.1 — The VPAK header](01-vpak-header.md): magic, version, block count, and the offset/size table that
  locates every section.
- [C11.2 — The ErtS string table](02-erts-strings.md): the 1 960 collection/field/instance names, how they're
  packed, and how they map to hashes.
- [C11.3 — The reflection type-name table](03-type-names.md): the 16 `EA::Reflection::*` primitive types that
  give every field a data type.
- [C11.4 — The data records](04-data-records.md): the collection record — class hash, parent reference, and
  `{field-hash, value}` entries — decoded from real bytes.
- [C11.5 — The trailer blocks](05-trailer-blocks.md): `NpeD`, `NrtS`, `NtaD` and the `0xEFFECADD` sentinel —
  dependencies, string references, and the data directory.
- [C11.6 — Navigating & editing the vault](06-navigating-editing.md): finding a collection, changing a value
  safely, and what must stay consistent.

---

## 11.1 VPAK at a glance

`attributes.bin` opens with the ASCII magic **`VPAK`**, then a header that declares the format version and
the blocks that follow:

```
0x00  "VPAK"                     magic
0x04  u32   version = 1
0x08  u32   0x40                 (header/section table size)
0x0C  u32   3                    block count
0x14  u32   …                    block offsets / sizes
0x1C  u32   0x80                 → offset of the first block (ErtS)
0x80  "ErtS" …                   string table
...   reflection type names, then typed data records
~0x55C00  "NpeD" / "NrtS" / "NtaD"  trailer blocks
```

Unlike the EAGL chunk files ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)), VPAK is
**not** a generic `{id, size}` chunk tree — it is a purpose-built container with a fixed header and named
blocks. Its layout is [C11.1](01-vpak-header.md).

## 11.2 Two string tables

The vault is unusually legible because it ships its names. The **`ErtS` block** (at 0x80) is a packed table of
1 960 null-terminated strings — the collection, field, and instance names: `default`, `carsurface`,
`gameplay`, `collisionworld`, and hundreds of effect/instance names like `fxcar_impactdebris`
([C11.2](02-erts-strings.md)). A second string region holds the **reflection type names** —
`EA::Reflection::Float`, `::UInt16`, `::Double`, `::TextureRef`, `::Reference`, and twelve more
([C11.3](03-type-names.md)). Together they mean you rarely have to guess what a value *is* or *means*.

## 11.3 Records, classes, and the universal parent

The bulk of the file is **typed records**. A collection record carries a class/collection hash, a **parent
reference**, and a list of `{field-hash, value}` entries. The parent reference is the key to the whole
system: almost every collection's parent is **`default`** (`0xEEC2271A`), which appears 1 071 times. A
collection stores only the fields that *differ* from `default` and inherits the rest — the resolved-value
model that Chapter 12 formalises. The record byte layout is [C11.4](04-data-records.md).

## 11.4 The reflection hash is live and known

Unlike the asset hash of the geometry world ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)), the
vault's field-name hash is **recoverable**: it is Jenkins' lookup2 seeded with `0xABCDEF00`, and this book
confirmed it against the live file — `default → 0xEEC2271A`, `carsurface → 0xFDA45513`,
`gameplay → 0x5CEA9D46`, `fire → 0x5E2FE5BC`, `car → 0xA13753EB`, every one present in the data. Because you
can *compute* a field's hash from its name, you can find and even add fields by name — the foundation of
writing to the vault, not just reading it ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).

---

### Key takeaways

- `attributes.bin` is a **VPAK** container: `VPAK` magic, version 1, a header with a block table, then named
  blocks — not an EAGL chunk tree.
- It ships two string tables: `ErtS` (1 960 collection/field/instance names) and the 16 `EA::Reflection::*`
  type names.
- The body is **typed records**: class hash + parent reference + `{field-hash, value}` entries.
- Nearly every collection inherits from **`default`** (`0xEEC2271A`, seen 1 071×) — storing only overrides.
- The field-name hash is the **reflection hash** (lookup2/`0xABCDEF00`), verified live and *computable* — so
  the vault is addressable by name.

**Next:** [Chapter 12 — The Reflection Schema & Resolved-Value Model](../C12-Reflection-Schema/C12-Reflection-Schema.md):
how field, type, and inheritance combine into a value you can read and write.
