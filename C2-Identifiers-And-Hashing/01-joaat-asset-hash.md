# C2.1 — Joaat: the Asset-Name Hash

> **The one-sentence version:** the names of textures, solids, geometry objects and effects are stored
> as a 32-bit Jenkins one-at-a-time hash of the lower-cased string — a tiny, well-avalanched function
> you can reproduce in nine lines and must match byte-for-byte.

[← Chapter 2 hub](C2-Identifiers-And-Hashing.md) · [Next: C2.2 — The reflection hash →](02-reflection-hash.md)

---

## What it is

Joaat is Bob Jenkins's "one-at-a-time" hash, published in 1997. It consumes one input byte per
iteration and finishes with a three-step avalanche:

```c
uint32_t joaat(const char* s) {
    uint32_t h = 0;
    for (; *s; ++s) {
        h += (uint8_t)*s;      // absorb the byte
        h += h << 10;          // mix upward
        h ^= h >>  6;          // mix downward
    }
    h += h <<  3;              // final avalanche
    h ^= h >> 11;
    h += h << 15;
    return h;
}
```

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
```

Every addition and shift is modulo 2³². The C version gets this for free from `uint32_t` overflow; the
Python version needs the explicit `& M` after each widening step, or the intermediate `h << 10` grows
without bound and the whole hash diverges after a few characters. This masking discipline is the single
most common bug when porting the function to a big-integer language.

## How it behaves — avalanche and why it matters

A good name hash must **avalanche**: flipping one input bit should flip about half the output bits, so
that similar names (`CARLOT_PIPE1` vs `CARLOT_PIPE2`) land far apart and collisions are rare. The two
in-loop mix steps (`<< 10` up, `>> 6` down) spread each absorbed byte across the whole 32-bit state
before the next byte arrives; the three-step tail finishes the diffusion so the last byte avalanches as
thoroughly as the first. The practical upshot for you: across the game's tens of thousands of asset
names, natural collisions essentially do not happen — a hash that fails to resolve is almost always a
name you haven't guessed yet, not two names colliding.

## Case sensitivity

Asset names are generally lower-cased before hashing. When you match user input against a stored asset
hash, lower-case first:

```python
def joaat_ci(s):
    return joaat(s.lower())
```

The convention is not perfectly uniform across every subsystem, so if a lower-cased match fails and you
are *sure* the name is right, try the raw case too. But lower-case is the correct first assumption.

## Why this hash, here

One-at-a-time is cheap (a handful of ALU ops per byte, no tables, no seed), has good avalanche for short
identifiers, and was already the house name-hash across EA Black Box's engine generation. For asset
names — short ASCII strings compared constantly at load time — it is an ideal fit: fast to compute when
the loader interns a name, and collision-free in practice across the shipped set. Note the deliberate
contrast with the *reflection* system ([C2.2](02-reflection-hash.md)), which needed a hash over
potentially longer, structured keys and used lookup2 instead — a reminder that "which Jenkins hash" is a
real distinction, not pedantry.

## Reproducing it against real data

The way to trust your port is to reproduce a hash the game actually stores. Read a texture name that is
present as text (standard TPK entries store the name string; [Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)),
hash it, and confirm it equals the `nameHash` field in the same entry and the corresponding entry in the
pack's hash table. When your `joaat("some_texture_name")` equals the stored 32-bit value, your port is
byte-correct and you can trust it everywhere else names appear as bare hashes.

```python
# schematic; see C5 for the exact TPK entry layout
for tex in tpk.entries:
    assert joaat(tex.name.lower()) == tex.name_hash, tex.name
```

## Bending it — the do's and don'ts

- **Do reproduce a known stored hash before trusting your port.** One correct round-trip against real
  data is worth any amount of "looks right."
- **Do lower-case by default**, and keep the raw-case fallback for the rare subsystem that doesn't.
- **Don't confuse it with the reflection hash.** If you are hashing a vault class name, a car name, a
  field name, or an enum value, Joaat is the *wrong* function — use lookup2/`0xABCDEF00`
  ([C2.2](02-reflection-hash.md)). Mixing them is the number-one source of "my hash matches nothing."
- **Don't try to invert it.** It is one-way; recovery is by forward-hashing a dictionary
  ([C2.4](04-hash-resolution.md)), never by "reversing" the arithmetic.

---

**Continue:** [C2.2 — The reflection hash: lookup2 with seed `0xABCDEF00`](02-reflection-hash.md) ·
[Chapter 2 hub](C2-Identifiers-And-Hashing.md)
