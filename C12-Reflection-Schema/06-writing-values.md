# C12.6 — Writing to the Vault

> **The one-sentence version:** because the reflection hash is computable and the types are known, you can
> *write* — override a field in one collection, edit `default` to move everything, or add a new field —
> always writing the value in its declared type and keeping the record and its indexes consistent.

[← C12.5 — Resolving a value](05-resolving-values.md) · [Chapter 12 hub](C12-Reflection-Schema.md) ·
[Next: Chapter 13 — Vault Categories: Car Tuning →](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)

---

## Three kinds of write

Editing the vault is choosing among three operations, in increasing invasiveness:

1. **Override an existing value** — the field already has a triple in the collection; overwrite its bytes.
2. **Change the baseline** — edit the field in `default` so every non-overriding collection follows.
3. **Add a new override** — the field isn't in this collection yet; insert a triple.

The first two are **in-place** (no size change); the third is a **repack**. Prefer the first two whenever the
change allows.

## Override in place (the safe write)

If `resolve` reports the field is `defined_in` the very collection you're editing
([C12.5](05-resolving-values.md)), its value bytes are already there at a fixed offset — overwrite them in the
declared type and width and you are done, with nothing else to fix:

```python
def override_value(buf, value_off, ftype, new_value):
    packed = encode_by_type(new_value, ftype)          # Float→<f, UInt32→<I, Bool→byte, …
    assert len(packed) == type_size(ftype)             # width must match
    buf[value_off : value_off + len(packed)] = packed  # in-place, no size change
```

This is the vault's equivalent of a same-size texture overwrite ([C5.5](../C5-Textures-TPK/05-extract-replace.md))
or an in-place geometry value edit ([C8.7](../C8-Geometry-Solids/07-editing.md)): the record size, the field
count, `NtaD`'s directory ([C11.5](../C11-Attribute-Vaults/05-trailer-blocks.md)), and every offset are
untouched.

## Edit `default` for a global change

To rebalance the whole game in one edit, resolve the field in **`default`** and override it there
([C12.4](04-default-inheritance.md)). Every collection that inherits (does not override) the field picks up the
new baseline automatically. This is enormously powerful and enormously blunt — use it for genuine global
changes (e.g. a physics constant), and remember that any collection with its *own* override is unaffected.

## Add a field (the repack write)

If the collection doesn't yet define the field (it's inherited), adding an override means **inserting a
triple**, which grows the record and triggers bookkeeping:

1. **Mint the field id.** `reflection_hash(field_name)` ([C12.1](01-reflection-hash.md)) — computable, which
   is the whole reason this is possible.
2. **Insert the triple** `{field_id, value(typed)}` into the record ([C12.3](03-value-triple.md)).
3. **Bump the record's field count and size.**
4. **Shift following records** and fix `NtaD`'s count and any offsets
   ([C11.5](../C11-Attribute-Vaults/05-trailer-blocks.md)).
5. **Update the `ErtS` string table** if the field name is genuinely new, and any header offsets after it
   ([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)).

This is the vault analogue of a geometry repack: change the bytes, then repair every count and offset that
described them.

## Rules that keep writes safe

- **Write in the declared type.** A `Float` field takes float bytes; never write a float into a `UInt32`
  field or vice versa ([C12.2](02-schema-map.md)).
- **Match width for in-place edits.** Fixed-width values keep their width; `Text` edits change length and force
  a repack ([C11.3](../C11-Attribute-Vaults/03-type-names.md)).
- **Don't emit the sentinel.** `0xEFFECADD` marks structure — never a value
  ([C11.4](../C11-Attribute-Vaults/04-data-records.md)).
- **Pick the right scope.** Override for one collection; edit `default` for all; use an intermediate parent for
  a family ([C12.4](04-default-inheritance.md)).

## Verify by resolving

After any write, re-run resolution ([C12.5](05-resolving-values.md)) on the affected field and confirm it
returns the value you intended, from the collection you intended (`defined_in`). For a repack, also re-check
the `VPAK` header parses and `NtaD`'s count matches the records
([C11.6](../C11-Attribute-Vaults/06-navigating-editing.md)). An in-place override only needs the resolution to
read back correctly — the same asymmetry that makes it the preferred edit.

---

### Key takeaways

- Writing has three modes: **override** (in place), **edit `default`** (global, in place), **add a field**
  (repack).
- In-place overrides change value bytes in the declared type/width — no size change, nothing downstream to
  fix.
- Editing `default` moves every non-overriding collection at once; use it for genuine global changes.
- Adding a field mints the id via the reflection hash, inserts a triple, and requires record/count/offset
  fixups.
- Write in the declared type, match width for in-place edits, never emit `0xEFFECADD`, and verify by
  re-resolving.

**Continue:** [Chapter 13 — Vault Categories: Car Tuning](../C13-Vault-CarTuning/C13-Vault-CarTuning.md) ·
[Chapter 12 hub](C12-Reflection-Schema.md)
