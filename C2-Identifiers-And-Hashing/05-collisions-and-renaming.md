# C2.5 — Collisions, Renaming & the Modder's Mental Model

> **The one-sentence version:** in a hash-keyed world you repoint instead of rename, you add new names by
> choosing collision-free strings, and you respect the birthday bound — "rare but real" is a
> quantifiable risk, not a hand-wave.

[← C2.4 — Hash resolution](04-hash-resolution.md) · [Chapter 2 hub](C2-Identifiers-And-Hashing.md) ·
[Next: C2.6 — The identifier map →](06-identifier-map.md)

---

## The core mental shift

In a string-keyed system you change a name by editing the string. In Most Wanted there is no string —
only a hash — so the operations available to you are different:

- **Repoint:** change what an existing hash *refers to*. Edit the pixels behind a texture hash, the mesh
  behind a solid hash, the bytes of a vault record behind a reflection key. The hash never moves. This is
  the overwhelmingly common, safe operation.
- **Reference-swap:** change a *reference* to point at a different existing hash. E.g. make a mesh use a
  different (already-present) texture by writing that texture's hash into the mesh's texture-ref slot.
  Also safe, because both hashes already exist and resolve.
- **Introduce:** add a genuinely new named asset, which means inventing a name, hashing it, adding the
  asset under that hash, and writing the hash into every place the engine will look. This is the only
  operation where collisions matter, and it needs care.

Ninety percent of practical modding is *repoint*. Reach for *introduce* only when you truly need a new
named entity, because it is where the sharp edges are.

## Why you (almost) never rename

Renaming would mean changing a hash — and a hash appears in *many* places: the asset's own header, a
hash table, and every reference to it across every file. To "rename" consistently you would have to find
and rewrite all of them, and miss one and you get a dangling reference the engine can't resolve. Since
the *text* has no runtime meaning anyway (the engine only ever compares hashes), renaming buys you
nothing the engine cares about. The only reason to want a text name is human readability, and that is
solved by the resolver ([C2.4](04-hash-resolution.md)), not by editing the data.

## Collisions: the birthday-bound reality

Two different strings *can* hash to the same 32-bit value. How worried should you be? Use the birthday
approximation: the chance that a set of `n` random 32-bit hashes contains at least one collision is
roughly `1 − e^(−n²/2·2³²)`. That gives:

| Distinct names in one table | Approx. collision probability |
|---|---|
| 1,000 | ~0.012 % |
| 10,000 | ~1.1 % |
| 100,000 | ~68 % |

Two lessons. First, **within a single small table** (a car's texture set, a scenery group list — dozens
to hundreds of entries), collisions are astronomically unlikely, which is why the game's own names don't
collide. Second, the number that matters is names *in the same table*, not names in the whole game — the
engine only ever compares hashes within one lookup context, so a texture and a vault field sharing a
hash is harmless because they are never compared. Your collision risk is therefore local and small,
*unless* you are adding many new names to one table.

For the weak Bin hash the effective space is far smaller than 2³² because of poor avalanche, so treat its
collision odds as much worse than the table above and check exhaustively ([C2.3](03-bin-sum-hash.md)).

## Introducing a new name safely

When you must add a new named asset:

1. **Enumerate the target table's existing hashes.** The set of hashes the engine will compare your new
   name against is one specific table (a TPK's hash table, a vault collection, a scenery group set).
2. **Choose a name and hash it** with the *correct* hash for that subsystem (Joaat for assets,
   lookup2/`0xABCDEF00` for reflection keys, Bin for the few Bin-keyed tables).
3. **Check for collision against that table only.** If your new hash is already present, pick a different
   name and retry. A one-character change fully re-avalanches a good hash, so collisions are trivially
   avoided.
4. **Write the hash everywhere the engine expects it** — the asset's own header/hash table *and* every
   reference. Missing one leaves a dangling reference.
5. **Register the name in your resolver** so future dumps show it as text.

```python
def safe_new_name(candidate, existing_hashes, hashfn):
    h = hashfn(candidate.lower())
    if h in existing_hashes:
        raise ValueError(f"{candidate!r} collides in this table; choose another")
    return h
```

## Bending it — the discipline in one place

- **Prefer repoint over introduce.** Editing what a hash points to has zero collision surface.
- **Use the right hash for the subsystem, always.** A new vault key hashed with Joaat will never be found
  by the engine; it must be lookup2/`0xABCDEF00`.
- **Collision-check against the specific table, not the whole game.** Local is what matters, and local is
  where the check is cheap and exact.
- **Keep the original bytes.** Every operation here is reversible if you kept a backup; introducing names
  in particular can leave dangling references if you slip, and the fastest fix is revert-and-retry
  ([Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md)).

---

**Continue:** [C2.6 — The identifier map: where every hash lives](06-identifier-map.md) ·
[Chapter 2 hub](C2-Identifiers-And-Hashing.md)
