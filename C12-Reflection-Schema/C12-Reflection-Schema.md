# Chapter 12 — The Reflection Schema & Resolved-Value Model

> **Goal of this chapter:** turn the raw records of Chapter 11 into *meaning* — compute a field's id from its
> name, read its bytes as the right type, and resolve its value through the inheritance chain up to
> `default` — so you can both read any attribute correctly and write new ones with confidence.

Chapter 11 gave you the vault's container: string tables, typed records, `{field-hash, value}` entries, and a
`default` parent. This chapter is the **semantics** — the reflection schema that maps a human name to a
32-bit id, ties that id to a type, and defines how a value is *resolved* when a collection inherits most of
its fields. It is the difference between seeing bytes and understanding a car's top speed.

> **Verified against retail data.** The reflection hash is confirmed live against `GLOBAL/attributes.bin`:
> `lookup2(name, seed 0xABCDEF00)` reproduces `default → 0xEEC2271A`, `carsurface → 0xFDA45513`,
> `gameplay → 0x5CEA9D46`, `fire → 0x5E2FE5BC`, `car → 0xA13753EB` — every hash present in the file. The 16
> `EA::Reflection::*` types and the 1 071-fold `default` parent reference anchor the type and inheritance
> models.

---

## Deep-dive pages

- [C12.1 — The reflection hash, recovered](01-reflection-hash.md): lookup2 with seed `0xABCDEF00`, why it is
  *computable* (unlike the asset hash), and the landmark values that prove it.
- [C12.2 — Field → type → offset](02-schema-map.md): the compiled schema that binds each field id to a type
  and a place, and why the type is mandatory to read a value.
- [C12.3 — The inline value triple](03-value-triple.md): the `{field, value, type}` unit as it appears in
  records, decoded from real bytes.
- [C12.4 — `default` inheritance](04-default-inheritance.md): the parent chain, sparse overrides, and why
  `default` appears 1 071 times.
- [C12.5 — Resolving a value](05-resolving-values.md): the full read algorithm — own value, else inherit,
  else schema default — with worked results.
- [C12.6 — Writing to the vault](06-writing-values.md): overriding, adding fields, and editing `default`, all
  keeping the schema honest.

---

## 12.1 The hash you can compute

The vault's field-name hash is the **reflection hash** — Jenkins' lookup2 seeded with `0xABCDEF00` — and its
defining virtue is that, unlike the asset hash of textures and geometry
([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)), it is **recoverable**. You can compute a field's id
from its name and go straight to it, and you can mint the id for a *new* field you want to add. This book
verified it against the live file: hashing `default`, `carsurface`, `gameplay`, `fire`, and `car` all produce
values that are present in the records ([C12.1](01-reflection-hash.md)). Computability is the property that
makes the vault *writable*, not just readable.

## 12.2 Field, type, place

A value is meaningless without three facts, and the schema supplies all three:

- **Which field** — the reflection hash of the name ([C12.1](01-reflection-hash.md)).
- **What type** — one of the 16 `EA::Reflection::*` primitives ([C11.3](../C11-Attribute-Vaults/03-type-names.md)),
  which says how many bytes and how to read them.
- **Where** — the byte offset of the value within the record.

Get the type wrong and `0x40A00000` is `5.0` as a `Float` or a billion-plus as a `UInt32`
([C12.2](02-schema-map.md)). The schema is the compiled binding of `field id → (type, place)` that keeps these
straight, and it is why a vault reader is a *schema-driven* reader, not a fixed struct.

## 12.3 The value triple

In the records, an attribute appears as an inline unit that pairs the field id with its value, interpreted by
type — the **`{field, value, type}` triple** ([C12.3](03-value-triple.md)). Decoded from real bytes, a triple
reads as, for example, `{0xEBCEE74C, 5.0, Float}` — field id, the float `5.0`, read as a `Float`. Walking a
record is walking its triples; the type in each tells you how far to advance to the next.

## 12.4 Inheritance: sparse by design

Almost every collection names **`default`** as its parent (`0xEEC2271A`, seen 1 071 times) and stores **only
the fields it overrides** ([C12.4](04-default-inheritance.md)). This makes records small and makes global
tuning trivial. It also means reading a value is not "look it up in this record" but "resolve it": use the
collection's own value if present, else the parent's, up the chain to `default`
([C12.5](05-resolving-values.md)). The resolution is the model's heart, and it is what a tool must implement
to report the value the *game* actually uses.

## 12.5 Read and write

Because the hash is computable, the type set is known, and the inheritance rule is simple, you can do both
halves of the job. **Reading** is resolve-then-decode ([C12.5](05-resolving-values.md)). **Writing** is the
inverse with discipline: override a field in one collection, add a new field, or change `default` for a
sweeping effect — always writing the value in its declared type and keeping the record consistent
([C12.6](06-writing-values.md)). This chapter is the toolkit for both.

---

### Key takeaways

- The reflection hash is **lookup2 seeded `0xABCDEF00`** — verified live and, crucially, **computable** from
  a name.
- A value needs three facts: which field (hash), what type (one of 16), and where (offset) — the schema binds
  them.
- Attributes appear as `{field, value, type}` triples; the type sets how to read and how far to advance.
- Inheritance is sparse: collections override a few fields and inherit the rest from **`default`** (1 071×).
- Reading is resolve-then-decode; writing is override / add / edit-`default`, keeping the schema honest.

**Next:** [Chapter 13 — Vault Categories: Car Tuning](../C13-Vault-CarTuning/C13-Vault-CarTuning.md): the
schema applied to the performance data that defines every car.
