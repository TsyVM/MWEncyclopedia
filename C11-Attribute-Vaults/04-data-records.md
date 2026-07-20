# C11.4 — The Data Records

> **The one-sentence version:** the body of the vault is collection records — each with a class/collection
> hash, a **parent reference** (almost always `default`, `0xEEC2271A`), and a list of `{field-hash, value}`
> entries — and a collection stores only the fields that differ from its parent.

[← C11.3 — The reflection type-name table](03-type-names.md) · [Chapter 11 hub](C11-Attribute-Vaults.md) ·
[Next: C11.5 — The trailer blocks →](05-trailer-blocks.md)

---

## A record, from real bytes

Here is a collection record decoded from the live `attributes.bin`, annotated:

```
+0x189EC  0x00000101   record header (flags / kind)
+0x189F0  0x00030003   counts (fields)
+0x189F4  0x00000020   record size (0x20)
+0x189F8  0xFB111FEF   class / collection hash
+0x189FC  0xEEC2271A   parent reference  → "default"
+0x18A00  0x00000000
+0x18A04  0xEBCEE74C   field-name hash
+0x18A08  0xF2DE3891   field-name hash
+0x18A0C  0x00000000
+0x18A10  0x40A00000   value = 5.0   (Float)
+0x18A14  0x41F00000   value = 30.0  (Float)
+0x18A18  0xFB111FEF   (next record begins — same class hash pattern)
```

The shape is consistent: a small header (flags, field count, size), the **collection hash** that names the
record, the **parent reference**, then the field entries pairing a **field-name hash** with a **typed value**.
The two floats `5.0` and `30.0` are the actual attribute values this collection sets.

## The parent reference is the whole idea

Look at `0xEEC2271A` at `+0x189FC`: it is `lookup2("default", 0xABCDEF00)`
([C11.2](02-erts-strings.md)), and it appears **1 071 times** across the file — once per collection that
inherits from `default`. This is the vault's central design:

- There is a base collection, **`default`**, that defines every field's baseline value.
- Every other collection names `default` (or another collection) as its **parent** and stores **only the
  fields it overrides**.
- Reading a field means: use this collection's value if present, else fall back to the parent's, recursively
  up to `default`.

So a record is sparse — it lists differences, not the full attribute set. A car that is standard in every
respect but top speed stores only top speed; everything else resolves from `default`. This keeps the vault
compact and makes global changes easy (edit `default`, everything inherits). The resolution algorithm is
[Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md).

## Field entries: hash + typed value

Each field entry joins a **field-name hash** (reflection hash of the field's name — computable from `ErtS`,
[C11.2](02-erts-strings.md)) to a **value** whose bytes are read according to the field's **type**
([C11.3](03-type-names.md)). Concretely:

```python
def read_field(buf, off, field_type):
    field_hash = u32(buf, off)
    value_bytes = buf[off+4 : off+4 + type_size(field_type)]
    value = decode_by_type(value_bytes, field_type)   # Float→f32, UInt32→u32, Reference→hash, …
    return field_hash, value, off + 4 + type_size(field_type)
```

The field hash tells you *which* attribute (label it via the `ErtS` name map); the type tells you *how to
read* the value. Both are needed, and the type is why you cannot decode a record from field hashes alone.

## The record header and the sentinel

Records carry a small header (flags, a field/entry count, and a size) so a reader can bound each one, and the
file uses the sentinel **`0xEFFECADD`** as a structural marker between/around record groups — you saw it after
the `carsurface` record and in the `NtaD` trailer ([C11.5](05-trailer-blocks.md)). Treat `0xEFFECADD` as a
"here is a boundary" beacon: it is a reliable anchor when scanning the data region, and a value that should
never appear as a legitimate field value.

> ✅ *Verified:* real records carry a header, a collection hash, a `default` (`0xEEC2271A`) parent reference,
> and `{field-hash, value}` entries with sensible float values (`5.0`, `30.0`); `0xEEC2271A` occurs 1 071× as
> the parent, and `0xEFFECADD` marks structural boundaries.
> 🟡 *Reasoned:* the exact bit meaning of the header flags/counts is identified by role (field count, size)
> rather than fully bit-proven; the collection/parent/field-entry structure is verified.

## Editing implications

- **Change values, keep structure.** Overwriting a field's value bytes in its declared type/width is the safe
  edit ([C11.6](06-navigating-editing.md)); the record size is unchanged.
- **Adding a field** means inserting a `{hash, value}` entry, bumping the record's field count and size, and
  fixing everything downstream — a repack.
- **Respect inheritance.** To change a value for *every* collection, edit `default`; to change one collection,
  add/override the field in that record.
- **Never emit `0xEFFECADD` as a value** — it collides with the structural sentinel.

---

### Key takeaways

- The vault body is **collection records**: header + collection hash + parent reference + `{field-hash,
  value}` entries.
- The parent reference is almost always **`default`** (`0xEEC2271A`, 1 071×); collections store only
  overrides and inherit the rest.
- A field entry pairs a reflection-hash **field id** with a **typed value** — you need both the hash (which
  field) and the type (how to read it).
- `0xEFFECADD` is a structural sentinel/boundary marker, never a legitimate value.
- Safe edits change value bytes in place; adding fields or editing `Text` is a repack; edit `default` to
  affect everything.

**Continue:** [C11.5 — The trailer blocks](05-trailer-blocks.md) · [Chapter 11 hub](C11-Attribute-Vaults.md)
