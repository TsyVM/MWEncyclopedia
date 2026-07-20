# Chapter 2 — Identifiers & Hashing

> **Goal of this chapter:** understand why the game stores almost no readable names, know *exactly*
> which of three distinct hash functions produces each identifier you meet, turn those hashes back into
> text, and plan edits in a world where there is usually no string to change — only a number.

When you dump a chunk tree ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) you
will constantly see values like `0x0743CFB1` sitting exactly where a name should be. This is not
obfuscation; it is a deliberate performance decision. The engine compares, stores, and looks up names
as fixed-width 32-bit integers, because comparing two integers is a single instruction and comparing
two strings is a loop. Every texture reference, every solid name, every attribute key, every enum value
is hashed at build time and lives in the shipped data as a number.

The catch — and the reason this chapter is long — is that Most Wanted does **not** use one hash. It uses
**three**, each in a different subsystem, and they are mutually incompatible. Hash a name with the wrong
one and you will get a number that matches nothing, conclude the data is corrupt, and waste an
afternoon. The single most valuable thing this chapter gives you is a precise map of *which hash keys
which subsystem*, verified against the executable's own code.

> **This chapter is verified against `speed.exe` (retail PC v1.3).** The reflection hash below is not
> inferred from matching outputs — it was read directly out of the instruction stream, and its identity
> is proven three independent ways. Where a claim rests on disassembly, the address is given so you can
> confirm it yourself.

---

## Deep-dive pages

- [C2.1 — Joaat: the asset-name hash](01-joaat-asset-hash.md): Jenkins one-at-a-time, the hash that names textures, solids, objects and effects; its avalanche, its case rule, and how to reproduce it exactly.
- [C2.2 — The reflection hash: lookup2 with seed `0xABCDEF00`](02-reflection-hash.md): the **separate, verified** hash that keys the entire attribute/vault system — read directly from the executable, proven by three coincident sources. Read this before you touch any vault data.
- [C2.3 — The Bin sum hash](03-bin-sum-hash.md): the weak `33·h + c` sum, where it genuinely appears, and why its collision-proneness is a design smell you must respect.
- [C2.4 — Hash resolution & dictionaries](04-hash-resolution.md): inverting a one-way function by forward-hashing a dictionary, seeding from the vault string table, and why coverage compounds.
- [C2.5 — Collisions, renaming & the modder's mental model](05-collisions-and-renaming.md): why you repoint rather than rename, how to introduce a new name safely, and the birthday-bound math behind "rare but real."
- [C2.6 — The identifier map: where every hash lives](06-identifier-map.md): a field-by-field table of which hash appears in which structure, with a worked cross-reference from a mesh to its texture.

---

## 2.1 Three hashes, three domains

Here is the map. Internalise it before anything else in this chapter:

| Hash | Algorithm | Seed / init | Keys… | Verified by |
|---|---|---|---|---|
| **Joaat** | Jenkins one-at-a-time | `h = 0` | asset & bundle names: textures, solids, geometry objects, effects, animation nodes | reproduction against stored asset hashes |
| **Reflection hash** | Jenkins **lookup2** (1996) | `0xABCDEF00` | the *entire* attribute/reflection system: class names, collection (car) names, field names, enum values | ✅ read from `speed.exe` code at `0x005CC240`/`0x005CC090`; proven by `lookup2("default") == 0xEEC2271A` |
| **Bin sum** | `h = c + 33·h` | `h = 0` | a handful of secondary keyings (e.g. some scenery-group keys) | reproduction; weak, collision-prone |

The two Jenkins hashes are the ones people confuse, because both are "a Jenkins hash" and both are 32
bits. They are *not* interchangeable: one-at-a-time processes one byte per round with shift-add-xor;
lookup2 processes twelve bytes per round through a very different mixing function and is seeded with a
non-zero constant. Their outputs for the same string are unrelated (for `"default"`: Joaat gives
`0xE4DF46D5`, lookup2 gives `0xEEC2271A`).

## 2.2 Why hashes at all

The design buys three things:

1. **Constant-time identity.** A name comparison becomes a 32-bit integer compare. In hot paths — the
   loader dispatching chunks, the reflection system resolving a field, a mesh finding its texture — this
   matters.
2. **Fixed-width storage.** A hash is always four bytes. Structures that reference names by hash have a
   fixed stride, which is exactly what the in-place residency model of
   [C1.12](../C1-EAGL-Container-Model/12-runtime-view.md) needs.
3. **Build-time name stripping.** The shipped data need not carry most strings at all, shrinking it and
   incidentally making casual inspection harder.

The cost is borne entirely by *you*, the person reading the data years later: a hash is one-way, so the
names are gone unless you can guess them back. [C2.4](04-hash-resolution.md) is how you win most of them
back anyway.

## 2.3 The reflection hash, established from the binary

The most important single fact in this chapter is that the attribute/vault system's keys are **lookup2
seeded with `0xABCDEF00`**, and this is not a guess. The public entry point is a small wrapper that
computes the string length and tail-calls the mixing core with the seed in `edx`:

```
005cc261: ba00efcdab       mov edx, 0xabcdef00        ; <<< the seed
005cc266: e925feffff       jmp 0x5cc090               ; tail-call the lookup2 core
```

and the core at `0x005CC090` opens with the golden-ratio constant and a twelve-bytes-at-a-time loop —
the unmistakable fingerprint of Jenkins lookup2:

```
005cc09c: b9b979379e       mov ecx, 0x9e3779b9        ; a = b = 0x9E3779B9 (golden ratio)
005cc099: 83f80c           cmp eax, 0xc               ; process 12 bytes per block
```

The proof it is exactly this function and not merely something lookup2-shaped: `lookup2("default")`
computes to `0xEEC2271A`; that same constant is compiled into the executable as an immediate
(`006a5baf: b8 1a 27 c2 ee  mov eax, 0xeec2271a`) as the "default key" sentinel; and it occurs **1071
times** literally in `attributes.bin` as the parent/default key of records. A value we computed, a value
the compiler emitted, and a value the data ships with — three independent sources, one number. Full
derivation, disassembly, and a byte-faithful port are in [C2.2](02-reflection-hash.md).

## 2.4 Reproducing the hashes

You need all three in your toolkit. The one-at-a-time and Bin sum are tiny; lookup2 is longer and lives
in [C2.2](02-reflection-hash.md). All arithmetic is modulo 2³² — in Python you **must** mask after each
step or the result diverges.

```python
M = 0xFFFFFFFF

def joaat(s):
    if isinstance(s, str): s = s.encode('latin1')
    h = 0
    for c in s:
        h = (h + c) & M
        h = (h + ((h << 10) & M)) & M
        h ^= h >> 6
    h = (h + ((h << 3) & M)) & M
    h ^= h >> 11
    h = (h + ((h << 15) & M)) & M
    return h

def binhash(s):
    if isinstance(s, str): s = s.encode('latin1')
    h = 0
    for c in s: h = (c + 33 * h) & M
    return h
```

## 2.5 The case rule

The reflection system interns keys from lower-cased strings. When you hash a name to match a stored
reflection key, hash the **lower-cased** form:

```python
def reflect_key(name):        # lookup2 lives in C2.2
    return lookup2(name.lower())
```

If a name you are *certain* exists doesn't match a stored hash, try the lower-cased form before
concluding the algorithm differs. This is the single most common false alarm when people first work with
the vault. (Joaat asset names are also generally lower-cased in the data, but the convention is looser;
try both cases.)

## 2.6 The modder's consequence, in one sentence

Because identity is a hash and hashing is one-way, **you almost never rename anything — you change what a
name already points at.** You edit the pixels a texture hash points to, the mesh a solid hash points to,
the bytes of the vault record a reflection key points to. The hash stays exactly where it is. The rare
exception — introducing a genuinely new name — means choosing a string whose hash does not collide in
the relevant table and then writing that hash into every place the engine will look for it.
[C2.5](05-collisions-and-renaming.md) is the discipline for doing that safely.

---

### Key takeaways

- Three incompatible hashes: **Joaat** (assets), **lookup2/`0xABCDEF00`** (the whole attribute/vault
  system), and the weak **Bin sum** (a few secondary keys). Never cross them.
- The reflection hash is ✅ verified from `speed.exe` itself, proven by `lookup2("default") ==
  0xEEC2271A` appearing as computed value, compiled immediate, and shipped data.
- All hashing is mod 2³²; the reflection system hashes lower-cased keys.
- Hashing is one-way; you recover names by forward-hashing a dictionary ([C2.4](04-hash-resolution.md)).
- You edit what a hash points to, not the hash.

**Next:** [Chapter 3 — Compression (JDLZ)](../C3-Compression-JDLZ/C3-Compression-JDLZ.md): many of the
files whose names you want to resolve are compressed before you can even see their chunks.
