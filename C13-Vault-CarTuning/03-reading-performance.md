# C13.3 — Reading a Car's Performance

> **The one-sentence version:** reading a car's real numbers is the resolve-then-decode operation of Chapter
> 12 aimed at behavior fields — hash the collection and field, walk the parent chain to the value, decode it
> as `Float`, and label it with its name from the string table.

[← C13.2 — Physics behavior classes](02-behavior-classes.md) · [Chapter 13 hub](C13-Vault-CarTuning.md) ·
[Next: C13.4 — Tuning value → simulation knob →](04-value-to-sim.md)

---

## The procedure

Nothing new is needed — car performance uses the exact machinery of Chapters 11–12:

```python
def read_car_performance(vault, behavior, fields, schema, name_map):
    ch = reflection_hash(behavior)                      # C12.1: "EngineRacer" → 0xB2809518
    report = {}
    for field in fields:
        fh = reflection_hash(field)
        value, ftype, src = resolve(vault, ch, fh, schema)   # C12.5: walk to default if needed
        report[name_map.get(fh, hex(fh))] = {
            "value": value,                              # decoded per type (usually Float)
            "type":  ftype,
            "source": "override" if src == ch else "inherited",
        }
    return report
```

Three steps, each from a prior chapter: **hash** the names ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)),
**resolve** through inheritance ([C12.5](../C12-Reflection-Schema/05-resolving-values.md)), **decode** by type
([C12.3](../C12-Reflection-Schema/03-value-triple.md)). Car data is not a special format — it is the vault,
read carefully.

## Label everything with the string table

Raw field hashes are unreadable; the `ErtS` string table
([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)) turns them into names. Build the `hash → name` map once
and every performance field becomes legible:

```python
name_map = read_erts(vault.buf)                 # hash → name (C11.2)
# now report keys are "topSpeed", "gripScale", … not 0x-hashes
```

This is the difference between a dump of numbers and a *readable* performance sheet. Because MW ships its
field names, you can produce the latter directly from the file.

## Floats, mostly — but check

Tuning fields are overwhelmingly `Float` ([C13.2](02-behavior-classes.md)), so float-first decoding reads most
of a car's data correctly, and human-scale results (a grip of `1.2`, a scale of `0.85`) confirm it. But not
everything is a float:

- **Flags** are `Bool`/`UInt` — a "has NOS" switch is not a float.
- **References** point at other collections (a car → its engine behavior).
- **Counts** (gear count) are integers.

So decode as `Float` by default, but treat a value that reads as a billion-plus or a tiny denormal as a sign
you have the wrong type ([C12.2](../C12-Reflection-Schema/02-schema-map.md)) — it is probably an integer or a
reference. Cross-check against sibling collections: the same field across many cars must share a type.

## Override vs inherited tells the story

The `source` tag on each field is genuinely informative, not decoration:

- A field marked **override** is where this car/behavior *intentionally differs* from baseline — the
  designer's deliberate tuning.
- A field marked **inherited** came from `default`; the car simply doesn't customise it.

So a car's *identity* is its overrides. If you want to understand what makes the M3 GTR special, list its
overridden fields — those are the deliberate choices; everything else is baseline. This is the fastest way to
read a car's design intent from the data.

## A readable performance sheet

Putting it together, you can generate, for any behavior, a table of `name → value (source)` — the vault's
answer to "what are this car's numbers?":

```
EngineRacer (0xB2809518), parent = default
  <fieldName>   = 5.0    (override)
  <fieldName>   = 30.0   (override)
  <fieldName>   = …      (inherited from default)
  …
```

The values are the tuning; the names come from `ErtS`; the sources come from resolution. Everything on the
sheet is derived from the file with no external data.

---

### Key takeaways

- Reading performance = hash → resolve → decode, the Chapter 12 procedure aimed at behavior fields.
- Label field hashes with the `ErtS` `hash → name` map to get a readable sheet.
- Decode as `Float` by default, but watch for `Bool`/`UInt` flags, integer counts, and `Reference` links.
- The **override/inherited** source tag reveals a car's design intent — its overrides are what make it
  distinct.
- A full performance sheet (name, value, source) is derivable entirely from the file.

**Continue:** [C13.4 — Tuning value → simulation knob](04-value-to-sim.md) · [Chapter 13 hub](C13-Vault-CarTuning.md)
