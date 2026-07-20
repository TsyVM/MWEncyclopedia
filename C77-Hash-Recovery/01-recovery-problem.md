# C77.1 — The Recovery Problem

> **The one-sentence version:** names became hashes at build time and the names were discarded, and because a hash is
> one-way — many names map to each, destroying information — you can't invert it, so recovering a name is *search*
> (guess, hash, match), not inversion.

[← Chapter 77 hub](C77-Hash-Recovery.md) · [Next: C77.2 — The three functions & their reversibility →](02-three-functions.md)

---

## Names in, numbers out

During development, everything has a *name* — a car is `BMWM3GTR`, a field is `TOPSPEED`, a surface is `wetpaved`.
But the build pipeline ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)) **hashes** those names into fixed
numbers ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)) — a `JOAAT`
([C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)), a `lookup2`
([C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md)) — and the *shipped game refers to things by the number,
not the name*. Hashing is faster to compare and compact to store, so the runtime wants numbers; the names were a
development convenience, mostly **discarded** at build.

So the shipped data is full of *opaque identifiers*: a vault record keyed by `0x0743CFB1`, an asset referenced by a
`JOAAT`, with no name attached. The game doesn't need the name — it only ever compares hashes — but *we* do, because a
name is *legible* and a hash isn't. `0x0743CFB1` tells you nothing; `PART_EN_COLD_AIR_INTAKE_SYSTEM` tells you
everything. Recovering the names is what turns the book's data from *numbers* into *meaning*.

## A hash is one-way

The reason recovery is *hard* is that a hash function is **one-way** — deliberately non-invertible
([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)):

- **Many-to-one** — infinitely many strings map to each 32-bit hash (there are only 2³² hashes but unlimited
  strings), so a hash *cannot* uniquely identify its input. Given `0x0743CFB1`, there are *many* strings that produce
  it.
- **Information-destroying** — the function mixes and folds its input down to 32 bits, throwing away everything that
  distinguishes the specific name. The bits that made `TOPSPEED` different from `TOPSPEEE` are *gone* after hashing.

So there is **no inverse function** — no `unhash(0x0743CFB1) → "the name"`. This isn't a gap in our understanding of
the algorithm ([C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md)); it's a *mathematical property* of
hashing. You can compute the hash of a name in one direction, but you cannot compute the name of a hash in the other.
Any claim to "reverse a hash" that isn't search is wrong.

## Recovery is search, not inversion

Since you can't *invert* a hash, recovery works the *only* way it can — **forward search**:

```
guess a candidate name  →  hash it (JOAAT / bin-sum / lookup2)  →  does it match the target hash?
        │                                                              │yes → recovered!
        └──────────────────────── no → try the next candidate ────────┘
```

You *don't* run the hash backward; you run it *forward* on candidate after candidate until one matches. This flips the
problem from "invert a one-way function" (impossible) to "find a name that hashes to this" (a *search*, tractable if
your candidates are good). The whole chapter is about making that search *efficient*: a good source of candidates (the
name dictionary, [C77.3](03-name-dictionary.md)) and exhaustive generation where needed (brute force,
[C77.4](04-bruteforce-frontier.md)).

The catch is *collisions* ([C2.5](../C2-Identifiers-And-Hashing/05-collisions-and-renaming.md)): because hashing is
many-to-one, a matching candidate is *probably* the original name but not *provably* — another string could hash to
the same value. In practice a *plausible* name (a real word, a convention-following identifier,
[C77.3](03-name-dictionary.md)) that matches is almost certainly right, but recovery is *evidence*, not *proof* — a
tiering ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)) matter, like everything else in the book.

## RE implications

- **Names → numbers** — the build hashes names and discards them; shipped data refers to things by hash
  ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)).
- **One-way** — hashes are many-to-one and information-destroying, so there's **no inverse function** — a
  mathematical fact.
- **Recovery is forward search** — guess names, hash them, match; not inversion.
- **Matches are evidence** — collisions mean a match is *probably* right, not *provably*
  ([C2.5](../C2-Identifiers-And-Hashing/05-collisions-and-renaming.md)).

---

### Key takeaways

- Names are **hashed at build time** ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)) and **discarded** —
  the shipped game refers to everything by **number**, so recovering names is what turns opaque IDs (`0x0743CFB1`)
  into **legible labels** (`PART_EN_COLD_AIR_INTAKE_SYSTEM`).
- A hash is **one-way** — **many-to-one** (unlimited strings, 2³² hashes) and **information-destroying** — so there is
  **no inverse function**; "reversing a hash" that isn't search is a category error.
- Recovery is therefore **forward search** — guess a candidate name, **hash it forward**, and check for a match —
  flipping "invert a one-way function" (impossible) into "find a name that hashes to this" (tractable).
- A match is **evidence, not proof** — collisions ([C2.5](../C2-Identifiers-And-Hashing/05-collisions-and-renaming.md))
  mean a plausible matching name is *almost certainly* right but tiered, like all the book's claims
  ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)).
- The chapter's work is making the search **efficient** — good candidates (the **name dictionary**,
  [C77.3](03-name-dictionary.md)) and exhaustive **brute force** where needed ([C77.4](04-bruteforce-frontier.md)).

**Continue:** [C77.2 — The three functions & their reversibility](02-three-functions.md) · [Chapter 77 hub](C77-Hash-Recovery.md)
