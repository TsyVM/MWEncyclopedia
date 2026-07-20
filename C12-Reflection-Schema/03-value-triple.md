# C12.3 — The Inline Value Triple

> **The one-sentence version:** an attribute appears in a record as a `{field, value, type}` unit — the
> reflection-hash field id, the value bytes, and the type that reads them — and walking a record is walking
> its triples, each type telling you how far to the next.

[← C12.2 — Field → type → offset](02-schema-map.md) · [Chapter 12 hub](C12-Reflection-Schema.md) ·
[Next: C12.4 — `default` inheritance →](04-default-inheritance.md)

---

## The unit of the vault

The vault's atom is the triple: **a field id, a value, and the type to read it**. Everything else — records,
collections, inheritance — is built from triples. Decoded from the live file, a record's entries read as:

```
{ 0xEBCEE74C , 5.0  , Float }        # field id, value, type
{ 0xF2DE3891 , 30.0 , Float }
```

The field id ([C12.1](01-reflection-hash.md)) says *which* attribute; the type
([C11.3](../C11-Attribute-Vaults/03-type-names.md)) says *how to read* the bytes and *how wide* they are; the
value is the payload in between. Label the field ids via the `ErtS` name map
([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)) and the triples become human-readable attributes.

## How triples pack in a record

Within a collection record ([C11.4](../C11-Attribute-Vaults/04-data-records.md)) the field ids and their
values are laid out so that, given the schema (`field id → type`), you can walk them in order: read a field
id, look up its type, read that many value bytes, advance, repeat until the record's field count is exhausted.

```python
def walk_triples(buf, rec, schema):
    off = rec.first_field_off
    triples = []
    for _ in range(rec.field_count):
        fid   = u32(buf, off);          off += 4
        ftype = schema[fid]
        w     = type_size(ftype)
        value = decode_by_type(buf, off, ftype); off += w
        triples.append((fid, value, ftype))
    return triples
```

The type in each triple is what makes the walk self-describing: it advances the cursor by exactly the value's
width, so the next field id lands where expected. Miss a type (or get one wrong) and every subsequent triple
is read from the wrong offset — the desynchronisation failure common to all typed streams
([C12.2](02-schema-map.md)).

## Reading each type

`decode_by_type` is the small switch that turns value bytes into a Python value per the 16 primitives:

```python
def decode_by_type(buf, off, ftype):
    if ftype == "Float":     return struct.unpack_from("<f", buf, off)[0]
    if ftype == "Double":    return struct.unpack_from("<d", buf, off)[0]
    if ftype in ("UInt32","Type"):     return u32(buf, off)
    if ftype == "Int32":     return struct.unpack_from("<i", buf, off)[0]
    if ftype in ("UInt16",): return struct.unpack_from("<H", buf, off)[0]
    if ftype in ("Int16",):  return struct.unpack_from("<h", buf, off)[0]
    if ftype in ("UInt8","Char"): return buf[off]
    if ftype == "Int8":      return struct.unpack_from("<b", buf, off)[0]
    if ftype == "Bool":      return buf[off] != 0
    if ftype == "Reference": return u32(buf, off)      # a hash → another collection
    if ftype == "Text":      return read_text(buf, off)      # variable width
    # Int64/UInt64/Double are 8-byte; KeyValueAttrib is nested
    ...
```

The `Reference` case is special and important: its value is itself a hash pointing at *another* collection,
which is exactly how the parent reference (`default`) and cross-collection links are stored
([C12.4](04-default-inheritance.md)). So a triple can hold a plain number *or* a pointer into the vault graph,
and the type is what tells them apart.

## Why the triple is the right abstraction

Modelling the vault as a stream of `{field, value, type}` triples — rather than fixed C structs — is what
makes it flexible and, for you, tractable:

- **Sparse records work naturally.** A collection lists only the triples it overrides
  ([C12.4](04-default-inheritance.md)); the absent ones simply aren't in the stream.
- **The schema can evolve.** Fields can be added without breaking readers that walk triples by type.
- **Tools stay simple.** One `walk_triples` + `decode_by_type` reads *every* collection, whatever its fields —
  no per-collection struct definitions.

This is the same schema-driven philosophy that makes the whole reflection system general; the triple is its
in-record expression.

---

### Key takeaways

- The vault's atom is the `{field, value, type}` triple: field id (hash), value bytes, and the type that
  reads them.
- Walking a record = reading triples in order; each type advances the cursor by the value's width.
- `decode_by_type` maps the 16 primitives to values; `Reference` yields a hash pointing at another
  collection.
- A wrong type desynchronises the walk from that field on — honour the type at every step.
- Triples make sparse records and schema evolution natural and let one reader handle every collection.

**Continue:** [C12.4 — `default` inheritance](04-default-inheritance.md) · [Chapter 12 hub](C12-Reflection-Schema.md)
