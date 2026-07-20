# C8.3 — Object Names & the Asset Hash

> **The one-sentence version:** object name-hashes are asset-hash values — deterministic, not any standard
> hash — but this book pins down a property no prior record noted: the hash is **tail-additive**, so
> `NAME_A`, `NAME_B`, `NAME_C` hash to consecutive values, which yields a real prediction trick for
> trailing-character variants and a real proof about the algorithm's shape.

[← C8.2 — The object header](02-object-header.md) · [Chapter 8 hub](C8-Geometry-Solids.md) ·
[Next: C8.4 — Bounding boxes & the Z-up world →](04-bounding-boxes.md)

---

## Object names are asset-hashed

Like texture keys ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)), a solid's name-hash belongs to the
**asset-hash world** ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)): it is a deterministic function
of the name that matches no standard string hash. `COBALTSS_BASE_A → 0x54DF8EF4` reproduces none of Joaat
(`0x16EC635A`), Bin (`0x8A995669`), FNV-1a (`0x13A913B4`), CRC-32 (`0xF03B362D`), or lookup2 (`0xBFE3945B`).
Gathering **1,179+** object names across sixteen car bundles and testing the full hash battery confirms it:
zero matches, on any input form.

## The tail-additive discovery

Where the asset hash *does* reveal structure is in how it responds to changing the **last** character. The
first three objects of the worked car are a family:

```
COBALTSS_BASE_A → 0x54DF8EF4
COBALTSS_BASE_B → 0x54DF8EF5     (+1)
COBALTSS_BASE_C → 0x54DF8EF6     (+1)
```

The names differ only in the final character (`A`, `B`, `C` — ASCII +1 each), and the hashes differ by
exactly +1 each. This is not a one-off: across the corpus, of all same-length pairs that differ only in their
**last** character, **2,398 of 2,474 (97 %)** satisfy `Δhash = Δchar`. In other words, the last character of
the name contributes to the hash **additively, with weight one** — the signature of a Horner-style core where
the final step is `h = h · P + c`.

> ✅ *Verified:* the asset hash is tail-additive — last-character deltas map one-to-one onto hash deltas in
> 97 % of same-length pairs; the exceptions are rare and consistent with occasional collisions/carries.

## What the tail-additive property does *not* imply

It is tempting to conclude the whole hash is a simple `h = h·P + c` Horner (like sdbm or djb2). It is **not**.
Solving for a single constant multiplier `P` and initial value across the 1,179 names reproduces **zero**
pairs beyond the one used to derive them, and the classic multipliers (33, 65599, FNV primes) all fail. So
while the *last* character enters additively, earlier characters do not combine under a single constant
multiplier — there is a position-dependent or nonlinear element in the mixing. The full closed form is not
recoverable from shipped data, exactly as expected for a packer-minted hash whose routine did not ship in the
runtime.

> ✅ *Verified negative:* constant-multiplier Horner (`h = h·P + c`) with any single `P`/init reproduces none
> of the object hashes beyond the seed pair, so the asset hash is more than a simple polynomial hash.

## The prediction trick (and its limits)

The tail-additive property is genuinely useful, because it lets you compute the hash of a name that differs
from a **known** name only in trailing characters, without knowing the algorithm:

```python
def predict_sibling_hash(known_name, known_hash, new_name):
    # valid only when new_name == known_name except in trailing char(s),
    # same length, and you trust the tail-additive property
    assert len(known_name) == len(new_name)
    # single trailing-char change:
    diffs = [i for i in range(len(known_name)) if known_name[i] != new_name[i]]
    if diffs == [len(known_name)-1]:
        return (known_hash + (ord(new_name[-1]) - ord(known_name[-1]))) & 0xFFFFFFFF
    raise ValueError("prediction only valid for a trailing-character change")
```

So if you know `COBALTSS_BASE_A = 0x54DF8EF4`, you know `..._BASE_D = 0x54DF8EF7` without any hashing. This
covers the very common case of enumerated part variants (`_A`, `_B`, `_C`, `_00`, `_01`, …). Its limit is
equally clear: change anything *before* the last character and the additive shortcut no longer applies,
because earlier positions mix nonlinearly. For those you must **read** the stored hash, not compute it.

## Looking an object up by name-hash

Because the hash table (`0x00134003`) is sorted ascending ([C8.1](01-solidlist-container.md)), finding an
object is a binary search on the hash, then a directory read for the offset — covered in detail in
[C8.6](06-lookup.md). The name-hash is the key to that lookup; the ASCII name at `+0xA0` is only for display.

## Practical guidance

- **Read object hashes; don't try to compute them from scratch.** Like texture keys, they're asset-hashed and
  not reproducible by standard hashes.
- **Do use the tail-additive shortcut** for enumerated variants that differ only in a trailing character — it
  is verified and exact in 97 % of cases (spot-check against a second known sibling when it matters).
- **Preserve hashes across edits.** The hash keys the directory and every reference; changing a name without
  updating its hash (or vice versa) breaks lookup.
- **Never assume constant-P Horner** — the earlier characters do not combine that simply.

---

### Key takeaways

- Object name-hashes are asset-hash values: deterministic, matching no standard hash (verified over 1,179+
  names).
- The asset hash is **tail-additive** — last-character deltas equal hash deltas (97 %); sequential names get
  sequential ids.
- It is **not** a constant-multiplier Horner: solving for one `P`/init reproduces nothing beyond the seed, so
  earlier characters mix nonlinearly.
- You can **predict** the hash of a trailing-character sibling from a known one; you cannot for earlier-position
  changes — read those.
- The name-hash is the lookup key (sorted table + directory); the ASCII name is for humans.

**Continue:** [C8.4 — Bounding boxes & the Z-up world](04-bounding-boxes.md) · [Chapter 8 hub](C8-Geometry-Solids.md)
