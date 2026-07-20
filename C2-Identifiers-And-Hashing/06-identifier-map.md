# C2.6 — The Identifier Map: Where Every Hash Lives

> **The one-sentence version:** a field-by-field table of which hash appears in which structure, so that
> when you read a bare 32-bit value you already know which of the three functions produced it — and a
> worked cross-reference from a mesh to its texture.

[← C2.5 — Collisions & renaming](05-collisions-and-renaming.md) · [Chapter 2 hub](C2-Identifiers-And-Hashing.md) ·
[Next chapter: C3 — Compression →](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)

---

## The map

When you pull a 32-bit value out of a structure, the *structure* tells you which hash made it. Keep this
table next to your dumper:

| Where you read it | Field | Hash | Notes |
|---|---|---|---|
| Geometry object header | `nameHash` (@ +16 in the 160-byte header) | Joaat | the solid's own name ([C8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) |
| Geometry texture-refs chunk | per-entry hash | Joaat | diffuse/normal/spec texture the solid uses |
| TPK entry | `nameHash` | Joaat | the texture's name ([C5](../C5-Textures-TPK/C5-Textures-TPK.md)) |
| TPK hash table | per-texture hash | Joaat | parallel index into the entry list |
| Vinyl / livery layer | `textureHash` | Joaat | decal/mask texture ([C70](../C70-Visual-Customisation/C70-Visual-Customisation.md)) |
| Attribute vault — class name | record class key | **lookup2/`0xABCDEF00`** | e.g. `EngineRacer` ([C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) |
| Attribute vault — collection key | car / collection name | **lookup2/`0xABCDEF00`** | e.g. a car's tuning collection |
| Attribute vault — field name | per-field key | **lookup2/`0xABCDEF00`** | the `{field, value, type}` triple's field ([C12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) |
| Attribute vault — enum value | enum key | **lookup2/`0xABCDEF00`** | symbolic enum member |
| Attribute vault — default/parent | `0xEEC2271A` | **lookup2/`0xABCDEF00`** | literally `lookup2("default")`; the inheritance sentinel |
| Scenery group | group key | **Bin sum** | named prop group ([C16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) |
| Sound routing | sound id | *(numeric id, not a name hash)* | an index/id, not a hashed string ([C19](../C19-Audio-Banks/C19-Audio-Banks.md)) |

The single most useful line here is the divide between rows: **everything in a geometry or texture
structure is Joaat; everything in the attribute vault is lookup2/`0xABCDEF00`.** If you remember only
that, you will pick the right hash almost every time. The `0xEEC2271A` row is a bonus — when you see that
exact constant in vault data, you are looking at a default/parent pointer, not a real field, which is a
handy landmark when navigating records ([C12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).

## Cross-referencing is hash-based

The engine wires structures together by hash. A shading group inside a mesh does not embed its texture;
it stores the texture's Joaat hash, and the runtime (or your tool) finds the matching texture by scanning
a TPK's entries for that hash. This indirection is why the resolver compounds ([C2.4](04-hash-resolution.md))
and why "repoint" is the natural edit ([C2.5](05-collisions-and-renaming.md)).

```python
def find_texture(tex_hash, tpks):
    for tpk in tpks:
        for tex in tpk.entries:
            if tex.name_hash == tex_hash:
                return tex
    return None
```

## Worked example — from a mesh to a readable material name

Put the pieces together. You are browsing a car's geometry and a shading group records diffuse texture
hash `0x0743CFB1`. You want to know what it is.

```python
# 1) Seed the resolver from the vault string table (Chapter 11) — one time.
resolver = HashResolver()
for s in load_vault_strings('attributes.bin'):
    resolver.add(s)

# 2) As you parse the car's TPK, register every texture name (they are stored as text).
for tex in tpk.entries:
    resolver.add(tex.name)

# 3) Now the bare hash resolves — as an ASSET/Joaat domain lookup.
print(resolver.resolve(0x0743CFB1, domain='asset'))
#    -> e.g. "carlot_pipe1_spec"  instead of a hex blob

# 4) And you can locate the actual pixels to edit:
tex = find_texture(0x0743CFB1, [tpk])
#    tex now points at the DXT/ARGB blob you would repoint (Chapter 5/6).
```

The moment the resolver is warm, the entire world's geometry reads in material names instead of hex,
which makes every subsequent editing decision legible — you can see that a group uses `..._spec` and
know it is a specular map before you touch a byte.

## Bending it — use the map as a guardrail

- **Let the structure choose the hash.** Never guess the hash family; read the field's row in the table
  above. Passing the wrong `domain` to the resolver is how a Joaat asset hash gets mis-resolved against
  the Bin table and yields a garbage "name."
- **Treat `0xEEC2271A` as a landmark, not data.** When it shows up in vault records it means "inherit
  from default," and skipping over it correctly is part of reading the resolved-value model
  ([C12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).
- **Register text names eagerly.** The map is only useful if your resolver is fed; every name spelled out
  in a header or entry should be `add()`-ed the moment you read it.

---

### Chapter 2 — where you've arrived

You can now tell, for any 32-bit identifier in the game, which of three verified hash functions produced
it; you can reproduce all three exactly (the reflection hash proven from `speed.exe` itself); you can
recover most names by forward-hashing dictionaries seeded from the vault; and you know why editing means
repointing a hash rather than renaming it. That is the vocabulary the rest of the book speaks.

**Back to:** [Chapter 2 hub](C2-Identifiers-And-Hashing.md) ·
**Next chapter:** [C3 — Compression (JDLZ)](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)
