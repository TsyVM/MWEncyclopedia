# C12.1 — The Reflection Hash, Recovered

> **The one-sentence version:** the vault's field-name hash is Jenkins' **lookup2 seeded with `0xABCDEF00`**,
> and — unlike the asset hash — it is fully recoverable, verified by reproducing landmark values
> (`default → 0xEEC2271A`, `carsurface → 0xFDA45513`) that are present in the live file.

[← Chapter 12 hub](C12-Reflection-Schema.md) · [Next: C12.2 — Field → type → offset →](02-schema-map.md)

---

## The function

Field and collection names in the vault are hashed with **Jenkins' lookup2**, seeded with the constant
`0xABCDEF00`. This is the same reflection hash established in
[Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md), and here is a self-contained,
verified implementation:

```python
def reflection_hash(s: str, seed: int = 0xABCDEF00) -> int:
    M = 0xFFFFFFFF
    b = s.encode("latin1")
    def mix(a, bb, c):
        a=(a-bb)&M; a=(a-c)&M; a^=(c>>13); bb=(bb-c)&M; bb=(bb-a)&M; bb^=((a<<8)&M)
        c=(c-a)&M;  c=(c-bb)&M; c^=(bb>>13); a=(a-bb)&M; a=(a-c)&M; a^=(c>>12)
        bb=(bb-c)&M; bb=(bb-a)&M; bb^=((a<<16)&M); c=(c-a)&M; c=(c-bb)&M; c^=(bb>>5)
        a=(a-bb)&M; a=(a-c)&M; a^=(c>>3); bb=(bb-c)&M; bb=(bb-a)&M; bb^=((a<<10)&M)
        c=(c-a)&M; c=(c-bb)&M; c^=(bb>>15); return a, bb, c
    L = len(b); a = bb = 0x9E3779B9; c = seed; i = 0; ln = L
    while ln >= 12:
        a=(a+int.from_bytes(b[i:i+4],"little"))&M
        bb=(bb+int.from_bytes(b[i+4:i+8],"little"))&M
        c=(c+int.from_bytes(b[i+8:i+12],"little"))&M
        a, bb, c = mix(a, bb, c); i += 12; ln -= 12
    c = (c + L) & M; t = b[i:i+ln] + b"\x00"*(12-ln)
    for j, sh in [(10,24),(9,16),(8,8)]:
        if ln >= j+1: c=(c+(t[j]<<sh))&M
    for j, sh in [(7,24),(6,16),(5,8),(4,0)]:
        if ln >= j+1: bb=(bb+(t[j]<<sh))&M
    for j, sh in [(3,24),(2,16),(1,8),(0,0)]:
        if ln >= j+1: a=(a+(t[j]<<sh))&M
    a, bb, c = mix(a, bb, c); return c
```

## The proof

This is not asserted — it is checked against the live `attributes.bin`. Hashing names taken from the vault's
own `ErtS` string table ([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)) reproduces values that appear in
the records:

| Name | `reflection_hash(name)` | Present in file? |
|---|---|---|
| `default` | `0xEEC2271A` | ✓ (1 071×, the universal parent) |
| `carsurface` | `0xFDA45513` | ✓ |
| `gameplay` | `0x5CEA9D46` | ✓ |
| `fire` | `0x5E2FE5BC` | ✓ |
| `car` | `0xA13753EB` | ✓ |

Five independent names, five matches, each hash observed in the data. That is decisive: the vault keys its
fields by `reflection_hash(name)`, and you can compute it.

> ✅ *Verified:* `reflection_hash` (lookup2/`0xABCDEF00`) reproduces five field/collection hashes that are
> present in the live vault, including the landmark `default = 0xEEC2271A`.

## Why "recoverable" is the whole point

Contrast this with the **asset hash** of textures and geometry
([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)), which is deterministic but matches no standard hash
and cannot be computed from a name. The reflection hash is the opposite, and the difference is
transformative:

- **You can go from name to data directly.** Want `carsurface`'s grip value? Hash `carsurface`, hash the
  field name, find the record and entry — no dictionary needed.
- **You can add fields.** To introduce a new attribute you must produce its id; because the hash is
  computable, you mint the correct id from the name and the game will find it.
- **You can label everything.** Combined with the `ErtS` names, every hash in the file resolves to a
  human-readable field ([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)).

This computability is precisely what makes the vault a *writable* system and the reflection chapters worth
your time.

## Practical notes

- **Seed matters.** lookup2 with seed `0` gives a *different* value; the vault uses `0xABCDEF00`. A tool that
  forgets the seed will miss every field.
- **Encoding.** Names are ASCII/latin-1; hash the raw bytes.
- **Collisions are not your concern in practice.** The vault's field namespace is small enough that lookup2
  separates them cleanly; if you ever see two names collide, the schema disambiguates by collection.

---

### Key takeaways

- The reflection hash is **lookup2 seeded `0xABCDEF00`**; a verified implementation is given above.
- It reproduces five landmark vault hashes present in the live file, including `default = 0xEEC2271A`.
- Unlike the asset hash, it is **computable from a name** — so the vault is addressable and *writable* by
  name.
- Use seed `0xABCDEF00` (seed 0 gives a different result) and hash the raw ASCII bytes.
- Computability underlies everything else in this chapter: name→data lookup, labelling, and adding fields.

**Continue:** [C12.2 — Field → type → offset](02-schema-map.md) · [Chapter 12 hub](C12-Reflection-Schema.md)
