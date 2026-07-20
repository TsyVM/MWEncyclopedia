# C2.3 — The Bin Sum Hash

> **The one-sentence version:** a weak `h = c + 33·h` sum hash appears in a few secondary keyings; it is
> easy to compute, easy to collide, and you should treat any table keyed by it with extra suspicion.

[← C2.2 — The reflection hash](02-reflection-hash.md) · [Chapter 2 hub](C2-Identifiers-And-Hashing.md) ·
[Next: C2.4 — Hash resolution →](04-hash-resolution.md)

---

## What it is

The Bin hash is a classic "multiply-by-a-small-constant and add" string sum, the same family as the
well-known `djb2`:

```c
uint32_t binhash(const char* s) {
    uint32_t h = 0;
    for (; *s; ++s) h = (uint8_t)*s + 33u * h;
    return h;
}
```

```python
M = 0xFFFFFFFF
def binhash(s):
    if isinstance(s, str): s = s.encode('latin1')
    h = 0
    for c in s: h = (c + 33 * h) & M
    return h
```

Some variants of this family initialise `h` to a non-zero constant (djb2 uses `5381`); the game's usage
initialises to zero. If a Bin-keyed table won't resolve with the zero-init form, the non-zero-init
variant is the first thing to try — but in this data set, zero-init is the form that matches.

## Where it actually appears

Be disciplined about scope. The Bin hash is **not** the attribute-system hash — that is
lookup2/`0xABCDEF00` ([C2.2](02-reflection-hash.md)), a point the reflection-hash discovery settled
definitively. The Bin sum shows up in a smaller set of secondary keyings, most visibly the **scenery
group keys** in the world data ([C16](../C16-Scenery-Cull/C16-Scenery-Cull.md)), where named
groups of props are looked up by a BinHash of the group name. Treat its presence as the exception to
document, not the rule to assume: when you meet an unresolved 32-bit key, try Joaat and lookup2 first,
and reach for Bin only when those fail and the context is one of the known Bin-keyed subsystems.

> 🟡 *Reasoned scope:* the boundary of "which subsystems use Bin" is drawn from where zero-init `33·h`
> reproduces stored keys and where the better-avalanched hashes do not. The *algorithm* is ✅
> reproducible; the *exhaustive list* of Bin-keyed tables is not claimed to be complete.

## Why it's weak — and why that matters to you

A `33·h + c` sum has poor avalanche: short strings and anagram-like strings cluster, and the low bits
are barely mixed. Concretely, two different names collide far more readily than under Joaat or lookup2.
For the game this is tolerable because the Bin-keyed tables are small (a few dozen entries), so even a
weak hash rarely collides *within one small table*. For **you** it means two things:

1. **Reverse resolution is riskier.** When you forward-hash a dictionary to recover a Bin key, a match is
   weaker evidence than a Joaat/lookup2 match, because a wrong name is more likely to hash to the same
   value by accident. Corroborate a recovered Bin name against context (does this group name make sense
   here?) rather than trusting the hash alone.
2. **Introducing a new Bin-keyed name needs a collision check against the *whole table*, not just a
   spot check.** With poor avalanche, a new name is more likely to clash with an existing one; enumerate
   the table's keys and confirm your new name's Bin hash is absent before committing.

## Reproducing it

As with the other two hashes, validate your port against a stored key before trusting it: hash a scenery
group name you can read as text and confirm it equals the stored BinHash for that group
([C16](../C16-Scenery-Cull/C16-Scenery-Cull.md) gives the exact structure). One confirmed
round-trip against real data is the standard.

## Bending it — respect the weakness

- **Don't use Bin where the engine uses lookup2 or Joaat.** It is a niche hash; assuming it broadly is a
  fast route to wrong conclusions.
- **Corroborate recovered Bin names.** A single hash match is weaker evidence here than elsewhere.
- **Collision-check new Bin names against the entire target table**, because the birthday bound bites
  sooner with poor avalanche.
- **When in doubt about which hash a key uses, try all three and see which reproduces a key you can read
  as text.** The one that round-trips a known name is the one in use for that subsystem.

---

**Continue:** [C2.4 — Hash resolution & dictionaries](04-hash-resolution.md) ·
[Chapter 2 hub](C2-Identifiers-And-Hashing.md)
