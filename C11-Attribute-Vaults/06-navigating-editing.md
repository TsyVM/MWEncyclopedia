# C11.6 — Navigating & Editing the Vault

> **The one-sentence version:** to change a tuning value you find its collection and field by name (via the
> `ErtS` map and the reflection hash), overwrite the value's bytes **in its declared type and width**, and —
> because that changes nothing's size — write the file back with no repack at all.

[← C11.5 — The trailer blocks](05-trailer-blocks.md) · [Chapter 11 hub](C11-Attribute-Vaults.md) ·
[Next: Chapter 12 — The Reflection Schema →](../C12-Reflection-Schema/C12-Reflection-Schema.md)

---

## Finding a value

The vault is addressable by name, which makes navigation a three-step lookup:

1. **Name → hash.** Compute the reflection hash of the collection and field names
   (`lookup2(name, 0xABCDEF00)` — [C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)), or read them from
   the `ErtS` map ([C11.2](02-erts-strings.md)).
2. **Find the collection record.** Locate the record whose collection hash matches
   ([C11.4](04-data-records.md)).
3. **Find the field entry.** Within that record, find the `{field-hash, value}` entry whose field hash
   matches; if it isn't there, the value is inherited from the parent (`default`) —
   [Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md).

```python
def find_field(vault, collection_name, field_name):
    ch = reflection_hash(collection_name)
    fh = reflection_hash(field_name)
    rec = vault.record_by_hash(ch)
    for (h, value, off, ftype) in rec.fields:
        if h == fh:
            return off, ftype, value      # located: byte offset + type
    return None                            # inherited from parent (C12)
```

## The safe edit: overwrite in place

The overwhelmingly common vault edit — retuning a value — changes no sizes and is therefore the safe,
no-repack path ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)):

```python
def set_value(buf, off, field_type, new_value):
    packed = encode_by_type(new_value, field_type)   # Float→<f, UInt32→<I, …
    assert len(packed) == type_size(field_type), "width must match the declared type"
    buf[off : off+len(packed)] = packed              # overwrite the value bytes only
    return buf
```

Two rules make this bulletproof:

- **Match the type.** Write a `Float` field as four float bytes, a `UInt32` as four integer bytes
  ([C11.3](03-type-names.md)); never write a float pattern into an integer field.
- **Match the width.** A fixed-width value keeps its width, so the record size — and every offset,
  count, and trailer — is unchanged. Nothing downstream needs fixing.

Because the value bytes sit at a fixed offset and keep their length, this is the vault equivalent of a
same-size texture overwrite ([C5.5](../C5-Textures-TPK/05-extract-replace.md)): the most reliable mod you can
make.

## The repack edit: when size changes

Some changes do alter size and pull in the full bookkeeping:

- **Adding a field** to a collection — a new `{hash, value}` entry grows the record; update its field count
  and size, shift following records, and fix the `NtaD` count ([C11.5](05-trailer-blocks.md)).
- **Editing a `Text` value** to a different length — variable-width, so the record grows/shrinks; repack.
- **Adding a collection** — a new record plus updates to the directory counts and any references.

For all of these, the discipline mirrors the geometry directory fixups
([C8.7](../C8-Geometry-Solids/07-editing.md)): change the bytes, then repair every count and offset that
described them, then verify.

## Change one thing, or everything

Inheritance gives you a powerful choice of scope ([C11.4](04-data-records.md)):

- **Change one collection** — override the field in that record. Only that car / surface / event changes.
- **Change the baseline for all** — edit the field in **`default`**. Every collection that inherits (does not
  override) the field picks up the new value. This is how you make a sweeping balance change in one edit.

Knowing which to use is half of effective vault editing: retuning one car is a per-record override; rebalancing
the whole game is a `default` edit.

## Verify by re-reading

After any edit, re-open the vault and confirm:

1. the `VPAK` header still parses and its offsets are consistent ([C11.1](01-vpak-header.md));
2. the edited field reads back as the value you wrote, in the right type;
3. for a repack, `NtaD`'s record count matches the actual records and no offset dangles
   ([C11.5](05-trailer-blocks.md)).

For an in-place value change, check (2) alone is enough — nothing else moved. That asymmetry is exactly why
you prefer the in-place edit whenever the change allows it.

---

### Key takeaways

- Navigate by name: reflection-hash the collection and field, find the record, find the `{field-hash, value}`
  entry (or inherit from `default`).
- The safe edit overwrites the value bytes **in the declared type and width** — no size change, no repack.
- Size-changing edits (add field, resize `Text`, add collection) require record/count/offset fixups and
  verification, like the geometry directory.
- Inheritance sets scope: override one record to change one thing; edit `default` to change everything.
- Verify by re-reading; an in-place value edit only needs the value to read back correctly.

**Continue:** [Chapter 12 — The Reflection Schema & Resolved-Value Model](../C12-Reflection-Schema/C12-Reflection-Schema.md) ·
[Chapter 11 hub](C11-Attribute-Vaults.md)
