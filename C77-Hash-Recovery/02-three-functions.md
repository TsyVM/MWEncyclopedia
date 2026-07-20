# C77.2 — The Three Functions & Their Reversibility

> **The one-sentence version:** MW's three hashes — `JOAAT`, the bin-sum hash, and `lookup2` (seed `0xABCDEF00`) —
> are all non-invertible but all *forward-cheap*, so the same recovery strategy (hash a candidate, compare) works for
> every one; their differences only change *which* function you hash a candidate under.

[← C77.1 — The recovery problem](01-recovery-problem.md) · [Chapter 77 hub](C77-Hash-Recovery.md) ·
[Next: C77.3 — Building a name dictionary →](03-name-dictionary.md)

---

## Three functions, three domains

Recovery starts with knowing *which* function produced an identifier, because MW uses **three**
([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)), each for a domain:

- **`JOAAT`** (Jenkins one-at-a-time, [C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)) — **asset and bin
  names**: textures, solids, the general string hash. A per-character `add / shift / xor` loop with a finalising mix.
- **The bin-sum hash** ([C2.3](../C2-Identifiers-And-Hashing/03-bin-sum-hash.md)) — a distinct hash used for certain
  bin identifiers.
- **`lookup2`** (Jenkins lookup2, seed `0xABCDEF00`, [C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md)) —
  the **reflection/vault keys**: the field and class names ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).
  It mixes 12 bytes at a time using the golden-ratio constant `0x9E3779B9`.

So the *first* recovery question for any opaque ID is "which function made it?" — a `JOAAT` asset reference, a
`lookup2` vault key, or a bin-sum. Often the *context* tells you ([C2.6](../C2-Identifiers-And-Hashing/06-identifier-map.md)):
a vault record key is `lookup2`, an asset reference is `JOAAT`. When context is ambiguous, you simply *try all three*
([below](#the-strategy-is-the-same)).

> ✅ *Verified:* the `lookup2` seed `0xABCDEF00` appears ×50 and its mix constant `0x9E3779B9` ×4 in `speed.exe` —
> confirming the reflection hash is Jenkins `lookup2` ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md));
> `JOAAT` and the bin-sum hash are decoded in [C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)/[C2.3](../C2-Identifiers-And-Hashing/03-bin-sum-hash.md).

## Non-invertible, but forward-cheap

All three share the two properties that shape recovery ([C77.1](01-recovery-problem.md)):

- **Non-invertible** — each is one-way ([C77.1](01-recovery-problem.md)); none can be run backward. This is *by
  design* — they're designed to *scatter* names across the hash space, not to be reversible.
- **Forward-cheap** — each is *trivial to compute forward*. `JOAAT` is a few operations per character; `lookup2` a
  few more per 12-byte block. A modern CPU hashes *tens of millions* of candidate strings per second under any of
  them.

This combination is *exactly* what makes search-based recovery ([C77.1](01-recovery-problem.md)) feasible. If hashing
were *slow*, testing millions of candidates would be impractical; because it's *fast*, you can hash an entire
dictionary ([C77.3](03-name-dictionary.md)) or brute-force a huge candidate space ([C77.4](04-bruteforce-frontier.md))
in seconds. The one-wayness blocks inversion; the forward-cheapness enables search. Recovery lives in that gap.

## The strategy is the same

The crucial simplification: **the recovery *strategy* is identical for all three functions**. Whether an ID is
`JOAAT`, bin-sum, or `lookup2`, you recover it the same way — hash candidates *under that function* and compare:

```
for each candidate name:
    if JOAAT(name)   == target: match  (asset/bin)
    if binsum(name)  == target: match  (bin)
    if lookup2(name, 0xABCDEF00) == target: match  (reflection/vault)
```

So a recovery tool implements all three hashes and, for any target, tests candidates under whichever function(s)
apply. The functions' *differences* — seed, block size, mixing — matter *only* to the hash computation itself, not to
the strategy: you plug in the right function and the hash-and-match loop is unchanged. This is why a single **name
dictionary** ([C77.3](03-name-dictionary.md)), pre-hashed under all three, resolves identifiers of *any* type — one
dictionary, three hash columns, universal lookup.

The practical consequence: you never need to know *in advance* which function made an ID. Pre-hash every candidate
under all three; then any target hash you meet is a lookup in whichever column it lands in. The three-function
complexity collapses into a three-column table.

## RE implications

- **Three functions, three domains** — `JOAAT` (assets/bins), bin-sum (bins), `lookup2`/`0xABCDEF00` (reflection/vault).
- **Which one?** — context usually tells you ([C2.6](../C2-Identifiers-And-Hashing/06-identifier-map.md)); when unsure,
  try all three.
- **Non-invertible but forward-cheap** — no inversion, but tens of millions of forward hashes/sec — the gap search
  lives in.
- **One strategy** — hash-and-match under the right function; differences affect only the hash, not the method — so a
  dictionary pre-hashed under all three is universal.

---

### Key takeaways

- MW uses **three hashes** for three domains — **`JOAAT`** (assets/bins), the **bin-sum hash** (bins), and
  **`lookup2`/`0xABCDEF00`** (reflection/vault keys) — so the first recovery question is **"which function made this
  ID?"** (context usually answers; else try all three).
- All three are **non-invertible but forward-cheap** — one-way by design, yet computable **tens of millions of
  times per second** — which is *exactly* the property that makes **search-based recovery** feasible
  ([C77.1](01-recovery-problem.md)).
- The recovery **strategy is identical** for all three — **hash candidates under that function and compare** — the
  functions' differences (seed, block size, mixing) change only the *hash*, never the *method*.
- So a single **name dictionary** ([C77.3](03-name-dictionary.md)) **pre-hashed under all three functions** is a
  **universal lookup** — three hash columns, any target ID resolved by the column it lands in.
- Verified: the `lookup2` seed `0xABCDEF00` (×50) and mix constant `0x9E3779B9` (×4) in `speed.exe`
  ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).

**Continue:** [C77.3 — Building a name dictionary](03-name-dictionary.md) · [Chapter 77 hub](C77-Hash-Recovery.md)
