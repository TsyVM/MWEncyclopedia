# C2.4 — Hash Resolution & Dictionaries

> **The one-sentence version:** you can't invert a hash, but you can hash a dictionary of candidate names
> *forward* and build a reverse map — and because the same names recur across files, a resolver seeded
> from the vault string table pays compounding dividends across the whole game.

[← C2.3 — The Bin sum hash](03-bin-sum-hash.md) · [Chapter 2 hub](C2-Identifiers-And-Hashing.md) ·
[Next: C2.5 — Collisions & renaming →](05-collisions-and-renaming.md)

---

## What it is

A cryptographic-strength inverse is impossible and unnecessary. Name recovery is a **forward** problem:
hash every string you can plausibly guess, store `hash → name`, and look your unknown hash up in that
map. The art is in assembling a good candidate dictionary, and the leverage comes from the fact that the
game reuses names heavily — resolve a name once and it stays resolved everywhere it appears.

## A three-hash resolver

Because the game uses three hashes ([hub](C2-Identifiers-And-Hashing.md)), a practical resolver registers
each candidate under all three and answers with whichever matches:

```python
class HashResolver:
    def __init__(self):
        self.joaat  = {}   # hash -> name (asset names)
        self.reflect = {}  # hash -> name (vault/reflection keys)
        self.bin    = {}   # hash -> name (secondary keys)
    def add(self, name):
        n = name.lower()
        self.joaat[joaat(n)]     = name
        self.reflect[lookup2(n)] = name
        self.bin[binhash(n)]     = name
    def resolve(self, h, domain=None):
        if domain == 'asset':   return self.joaat.get(h)   or f"0x{h:08X}"
        if domain == 'reflect': return self.reflect.get(h) or f"0x{h:08X}"
        if domain == 'bin':     return self.bin.get(h)     or f"0x{h:08X}"
        # unknown domain: try all, most-trustworthy first
        return (self.joaat.get(h) or self.reflect.get(h)
                or self.bin.get(h) or f"0x{h:08X}")
```

Passing the `domain` when you know it (you usually do — the structure you read the hash from tells you)
avoids false positives, especially against the weak Bin table ([C2.3](03-bin-sum-hash.md)).

## Where candidate strings come from

In rough order of yield:

1. **The vault string table.** `attributes.bin` carries its own strings — surface/material classes,
   VFX/particle names, AI/pursuit system names, and the tuning class names (`EngineRacer`,
   `SuspensionRacer`, and their kin). Hashing every string in that table with lookup2 immediately
   resolves a large fraction of all reflection keys in the game for free. This is the single highest-value
   seed you have; do it first. (Structure: [Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md).)
2. **Names stored as text elsewhere.** Object names in geometry object headers, texture names in standard
   TPK entries, material-usage strings in meshes. Every one you parse, you `add()` — and its bare-hash
   appearances everywhere else light up.
3. **Conventional / generated names.** The data is full of regular patterns: `BASE`, `KIT00`…`KIT05`,
   `WINDOW_FRONT`, `_A`/`_B`/`_C` LOD suffixes, `DAMAGE_*` zones, `RIM_*`. Generate candidate lists from
   these templates and hash them en masse.
4. **Brute force** for short alphanumeric names — enumerate short strings over a restricted alphabet and
   match. Feasible for names up to ~7–8 characters; the full treatment, including how to bound the search
   and prune the alphabet, is [Chapter 77](../C77-Hash-Recovery/C77-Hash-Recovery.md).

## Why coverage compounds

The key economic fact: **names are shared references.** A mesh stores the *hash* of its diffuse texture;
that texture is *named as text* in its TPK entry. Parse the TPK, `add()` the name, and now every mesh in
the game that references that texture displays a readable material name — with no extra work. Seed the
resolver once at startup from the vault string table, then keep `add()`-ing every text name you encounter
as you parse, and your resolution coverage climbs monotonically through a session. A browser that starts
showing `0x0743CFB1` ends up showing `carlot_pipe1_spec` for the same field an hour later, purely because
you walked more files.

```python
resolver = HashResolver()
for s in load_vault_strings('attributes.bin'):   # seed: Chapter 11
    resolver.add(s)
# ... then, as you parse assets:
for tex in tpk.entries:
    resolver.add(tex.name)                        # its bare-hash refs now resolve
```

## A note on confidence

A resolved name is a *hypothesis* confirmed by a hash match. For the well-avalanched hashes (Joaat,
lookup2) a match on a name longer than a few characters is essentially certain — the odds of a wrong
guess colliding are ~1 in 4 billion. For the Bin hash, and for very short names under any hash, treat a
match as strong-but-not-final and sanity-check it against context. When you publish a recovered name into
a shared dictionary, note which hash confirmed it.

## Bending it — building a durable dictionary

- **Persist your resolver.** Dump `hash → name` to a file and reload it; a name recovered once should
  never have to be recovered again. Community dictionaries are exactly this, shared.
- **Record the domain with each name.** Knowing a string is a *reflection* key vs. an *asset* name lets
  future lookups pass the right `domain` and avoids cross-hash false hits.
- **Prefer text sources over brute force.** Every name the data spells out is a certainty; every
  brute-forced name is a probability. Exhaust categories 1–3 before spending cycles on 4.
- **Never let an unresolved hash stop you.** `0x0743CFB1` is a perfectly usable identifier for editing —
  you can repoint or edit what it references without ever knowing its text ([C2.5](05-collisions-and-renaming.md)).

---

**Continue:** [C2.5 — Collisions, renaming & the modder's mental model](05-collisions-and-renaming.md) ·
[Chapter 2 hub](C2-Identifiers-And-Hashing.md)
