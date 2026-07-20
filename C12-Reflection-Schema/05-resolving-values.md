# C12.5 — Resolving a Value

> **The one-sentence version:** to read the value the game actually uses, check the collection's own triple;
> if it isn't there, follow the parent reference up the chain to `default`, taking the first definition you
> find — resolve first, then decode by type.

[← C12.4 — `default` inheritance](04-default-inheritance.md) · [Chapter 12 hub](C12-Reflection-Schema.md) ·
[Next: C12.6 — Writing to the vault →](06-writing-values.md)

---

## The algorithm

Resolution is a walk up the parent chain, stopping at the first collection that defines the field:

```python
def resolve(vault, collection_hash, field_hash, schema):
    ch = collection_hash
    seen = set()
    while ch is not None and ch not in seen:
        seen.add(ch)
        rec = vault.record_by_hash(ch)
        if rec is None:
            break
        for (fid, value, ftype) in walk_triples(vault.buf, rec, schema):   # C12.3
            if fid == field_hash:
                return value, ftype, ch          # found here (ch = which collection defined it)
        ch = rec.parent_reference                # climb: usually → default
    return schema_default(field_hash)            # nothing overrode it; use the type/schema baseline
```

Three things make this correct and safe:

- **First definer wins.** The nearest collection in the chain that has the field supplies the value; ancestors
  are only consulted if closer ones don't define it.
- **`seen` guards cycles.** A malformed vault with a parent loop would otherwise hang; the visited-set stops
  it.
- **Return the *source* too.** Knowing *which* collection defined a value (`ch`) tells you whether it was an
  override or inherited — invaluable for editing decisions ([C12.6](06-writing-values.md)).

## A worked resolution

Take a car collection that overrides only a couple of fields. Its record holds, say,
`{topSpeed, 5.0, Float}` and `{someRate, 30.0, Float}` ([C12.3](03-value-triple.md)), and its parent
reference is `default` (`0xEEC2271A`):

- Resolving **`topSpeed`** finds it in the collection itself → `5.0` (an override).
- Resolving **`grip`** (not in the record) climbs to `default` and takes `default`'s `grip` → the inherited
  baseline.

So the *full* attribute set of the collection is its own two overrides plus every other field inherited from
`default`. Reporting that full set is what a faithful tool must do — the record alone shows only the two
overrides.

## Resolve, then decode

Order matters: **resolve which bytes, then decode by type** ([C12.2](02-schema-map.md)). Resolution finds the
right triple (own or inherited); decoding turns its value bytes into a number/flag/reference according to the
type. Doing it in the other order — decoding a field from the collection's record before checking whether it's
even defined there — is how tools report garbage for inherited fields.

```python
def get_attribute(vault, collection_name, field_name, schema):
    value, ftype, source = resolve(
        vault,
        reflection_hash(collection_name),
        reflection_hash(field_name),
        schema,
    )
    return {"value": value, "type": ftype, "defined_in": source}   # source: this collection or an ancestor
```

## References resolve too

When the resolved value is a `Reference` ([C12.3](03-value-triple.md)) — a hash pointing at another collection
— you may need to resolve *that* collection in turn (for example to follow a car → engine → some shared table
link). Treat references as edges in the vault graph and resolve transitively when the question demands it,
keeping the same `seen`-set discipline to avoid cycles across collections.

## Why this is the heart of the model

Everything in Chapters 11–12 exists to support this one operation. The string tables let you name fields; the
hash lets you find them; the types let you read them; inheritance lets records stay sparse — and **resolution
is what turns all of that into the single value the game uses at runtime.** Get resolution right and the vault
is an open book; get it wrong (read the record literally) and most of the data looks missing.

---

### Key takeaways

- Resolve a value by walking the parent chain, taking the **first collection** that defines the field, up to
  `default`.
- Guard against parent cycles with a visited-set; return the defining collection so you know override vs
  inherited.
- The full attribute set = a collection's overrides + everything inherited from `default`.
- **Resolve first, decode by type second** — decoding before resolving reports garbage for inherited fields.
- `Reference` values are graph edges; resolve them transitively when needed. Resolution is the operation the
  whole model serves.

**Continue:** [C12.6 — Writing to the vault](06-writing-values.md) · [Chapter 12 hub](C12-Reflection-Schema.md)
