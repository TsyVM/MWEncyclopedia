# Chapter 77 — Hash Recovery & Name Dictionaries

> **Goal of this chapter:** decode how *lost names are recovered* — the three one-way hashes (`JOAAT`, the bin-sum
> hash, and the `lookup2`/`0xABCDEF00` reflection hash) that turned names into opaque IDs at build time, and the
> techniques that turn them back: harvesting candidate names, building a name dictionary, hashing-and-matching, and
> brute-forcing what's left — closing gaps like the unresolved `0x0743CFB1`.

Throughout the book, identifiers are *hashes* — an asset is a `JOAAT` of its name
([C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)), a vault key a `lookup2` of its field name
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)). The build pipeline
([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)) hashed the names and mostly discarded them, so the shipped
game refers to things by number. This final chapter is about *getting the names back* — because a recovered name
turns an opaque `0x0743CFB1` into a legible label, and the **name dictionary** that results is the master key that
makes the whole book's data readable.

> **Verified against the executable.** MW uses three hash functions ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)):
> **`JOAAT`** (Jenkins one-at-a-time — asset/bin names, [C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)),
> the **bin-sum hash** ([C2.3](../C2-Identifiers-And-Hashing/03-bin-sum-hash.md)), and the **reflection hash** —
> Jenkins **`lookup2`, seed `0xABCDEF00`** ([C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md)), whose seed
> appears ×50 and whose golden-ratio mix constant `0x9E3779B9` appears ×4 in `speed.exe`. Under `lookup2`, **66.8%**
> of vault keys already resolve to names ([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md)); recovering the rest is
> this chapter's problem.

---

## Deep-dive pages

- [C77.1 — The recovery problem](01-recovery-problem.md): why names are lost, and why hashes can't simply be
  inverted.
- [C77.2 — The three functions & their reversibility](02-three-functions.md): `JOAAT`, bin-sum, `lookup2` — and how
  each is (only) brute-forceable.
- [C77.3 — Building a name dictionary](03-name-dictionary.md): harvesting candidates, hash-and-match, growing the
  dictionary.
- [C77.4 — Brute-forcing & the frontier](04-bruteforce-frontier.md): exhaustive search, its limits, and the
  `0x0743CFB1` gaps.
- [C77.5 — The name dictionary unlocks the book](05-dictionary-unlocks.md): the master key, and the final frontier.

---

## 77.1 The recovery problem

Names become hashes at build time ([C77.1](01-recovery-problem.md)) and the names are *gone* — the game ships the
numbers. And a hash is **one-way**: you can't invert `JOAAT` or `lookup2` to get the name back, because many names
map to each hash and the function destroys information. So recovery isn't *inversion* — it's *search*: guess names,
hash them, and match.

## 77.2 The three functions & their reversibility

MW's three hashes ([C77.2](02-three-functions.md)) — `JOAAT`, bin-sum, and `lookup2`/`0xABCDEF00` — are all
*non-invertible* but all *forward-cheap*: hashing a candidate is fast, so you can test millions of guesses per second.
Their differences (seed, mixing, output) matter only for *how you generate candidates*; the recovery *strategy*
(hash-and-match) is the same for all three.

## 77.3 Building a name dictionary

The core technique ([C77.3](03-name-dictionary.md)) is a **name dictionary**: a growing list of candidate names, each
pre-hashed under all three functions, so any hash you meet can be looked up. Candidates come from *harvesting* — the
executable's own strings ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)), naming conventions
(`PART_<FAMILY>_*`, [C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)), and wordlists — so the
dictionary compounds: every name found helps find the next.

## 77.4 Brute-forcing & the frontier

For hashes no dictionary resolves ([C77.4](04-bruteforce-frontier.md)), there's **brute force**: exhaustively
generate short strings (up to some length/charset) and hash each. It's decisive for *short* names but explodes for
long ones, so a frontier remains — unresolved hashes like `0x0743CFB1` that are neither in the dictionary nor short
enough to brute-force. Mapping that frontier honestly ([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md)) is the
chapter's close.

---

### Key takeaways

- Identifiers are **hashes** — names were hashed at build time ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md))
  and mostly **discarded**, so recovery means **getting the names back** to make the data legible.
- A hash is **one-way** — you can't invert it; recovery is **search** (guess names, hash, match), not inversion.
- The three functions — **`JOAAT`**, **bin-sum**, **`lookup2`/`0xABCDEF00`** — are all **non-invertible but
  forward-cheap**, so the strategy (**hash-and-match** against a **name dictionary**) is the same for all
  ([C77.2](02-three-functions.md)–[C77.3](03-name-dictionary.md)).
- The **name dictionary** compounds — harvested from the exe's strings, naming conventions, and wordlists — and
  **brute force** closes short-name gaps, leaving an honest **frontier** of unresolved hashes (`0x0743CFB1`)
  ([C77.4](04-bruteforce-frontier.md)).
- The recovered dictionary is the **master key** ([C77.5](05-dictionary-unlocks.md)) — it turns opaque IDs across the
  *whole book* into names.

**Next:** [C77.1 — The recovery problem](01-recovery-problem.md).
