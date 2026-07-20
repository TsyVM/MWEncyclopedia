# C11.2 — The ErtS String Table

> **The one-sentence version:** the `ErtS` block at 0x80 is a packed table of 1 960 null-terminated names —
> collections, fields, and instances like `default`, `carsurface`, `gameplay`, `collisionworld`,
> `fxcar_impactdebris` — and each of those names, hashed with the reflection hash, is how records key on it.

[← C11.1 — The VPAK header](01-vpak-header.md) · [Chapter 11 hub](C11-Attribute-Vaults.md) ·
[Next: C11.3 — The reflection type-name table →](03-type-names.md)

---

## The block

`ErtS` begins at 0x80 with its tag and a size field, then a run of concatenated, null-terminated ASCII
strings:

```
+0x80  "ErtS"       tag
+0x84  u32  0x79A0  block size (31 136 bytes)
+0x88  "default\0" "destruction\0" "carsurface\0" "car\0" "gameplay\0" …
```

Parsed from the real file, the block holds **1 960** strings. A sample from the front:

```
default        destruction     carsurface      car         gameplay
terraindriving environmental   fire            nis         fxcar_impactdebris
fxcar_coplightblue  fxenv_leaffall_hvy  fxenv_smokestack   fxgame_flare_red
collisionworld  fxtd_dr_asphalt_leaves  fxnis_extradust1   fxenv_fountain1
```

These are exactly the human-readable identifiers of the vault: the **collections** (`carsurface`, `gameplay`,
`collisionworld`), the reserved **`default`** base ([C11.4](04-data-records.md)), and the many **instances**
(effects like `fxcar_impactdebris`, `fxgame_flare_red`). The vault ships its own vocabulary.

## Why the names are here at all

Most binary databases discard names and keep only hashes. VPAK keeps both, and that is a gift:

- **You can read the vault without a name dictionary.** The strings are right there, so you can enumerate
  every collection and instance by name directly from the file.
- **You can bridge names to hashes.** Records key fields by the reflection hash of the name
  ([C11.4](04-data-records.md)); with the string table you can build the `name → hash` map yourself and label
  every record.
- **You can add fields meaningfully.** To add a field you need its name *and* its hash; the string table
  shows you the naming conventions to follow.

## Names → hashes: the reflection hash

Every string here hashes to a reflection-hash value ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md))
using Jenkins' lookup2 seeded with `0xABCDEF00`. This is verified live against the file — the hashes of
strings from this very table appear in the records:

| Name (from `ErtS`) | `lookup2(name, 0xABCDEF00)` | In file? |
|---|---|---|
| `default` | `0xEEC2271A` | ✓ (1 071×) |
| `carsurface` | `0xFDA45513` | ✓ |
| `gameplay` | `0x5CEA9D46` | ✓ |
| `fire` | `0x5E2FE5BC` | ✓ |
| `car` | `0xA13753EB` | ✓ |

So the string table and the record hashes are two views of the same identifiers, joined by a hash you can
compute ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).

## Building the name map

```python
def read_erts(buf, off=0x80):
    assert buf[off:off+4] == b"ErtS"
    size = u32(buf, off+4)
    blob = buf[off+8 : off+8+size]
    names = [s.decode("latin1") for s in blob.split(b"\x00") if s]
    # join to hashes so records become human-readable
    return {reflection_hash(n): n for n in names}   # hash → name
```

With this `hash → name` map, every `{field-hash, value}` entry in the records
([C11.4](04-data-records.md)) can be labelled with its real field name — turning an opaque record into a
readable list of named attributes.

## Instances and namespacing

Some names are **dotted** — `collisionworld.interrupts`, `collworld_spin.anytimeevent` — indicating a
hierarchical namespace: a collection and a sub-key. The `fx…` prefixes group effect instances by domain
(`fxcar_*` car effects, `fxenv_*` environment, `fxgame_*` gameplay, `fxnis_*` cutscene). Learning the prefix
vocabulary lets you find related content quickly: every car impact effect shares `fxcar_`, every surface its
`carsurface`/`terraindriving` collection.

## Editing implications

- **Don't rename casually.** A name's hash is what records reference; changing a string without updating every
  referencing hash orphans the record.
- **Add names in the same style.** New instances should follow the prefix conventions so they slot into the
  right domain.
- **Keep the block size honest.** If you add or remove strings, update the `ErtS` size at `+0x84` and any
  header offsets that follow the block ([C11.1](01-vpak-header.md)).

---

### Key takeaways

- `ErtS` (0x80) is a packed table of **1 960** null-terminated names: collections, fields, and instances.
- The names are the vault's vocabulary — `default`, `carsurface`, `gameplay`, `collisionworld`, `fx*`
  instances.
- Each name hashes (reflection hash, lookup2/`0xABCDEF00`) to the value records key on — verified live.
- Build a `hash → name` map from the block to label every record's fields.
- Dotted names and `fx*` prefixes are a namespace; don't rename without fixing references, and keep the block
  size honest on edits.

**Continue:** [C11.3 — The reflection type-name table](03-type-names.md) · [Chapter 11 hub](C11-Attribute-Vaults.md)
