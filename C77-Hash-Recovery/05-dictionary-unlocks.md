# C77.5 — The Name Dictionary Unlocks the Book

> **The one-sentence version:** the recovered name dictionary is the master key — it turns the opaque hashes that
> pervade *every* chapter into legible names, so a single cross-cutting artifact makes the whole book's data
> readable, and the honest frontier (66.8% and climbing) is the last thing the book maps.

[← C77.4 — Brute-forcing & the frontier](04-bruteforce-frontier.md) · [Chapter 77 hub](C77-Hash-Recovery.md) ·
[Book index →](../README.md)

---

## One key for every chapter

Hashes are not confined to one system — they are *everywhere* in the book, because the whole engine identifies things
by hash ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)):

- **Assets** — textures, solids, models, referenced by `JOAAT` ([Chapters 5](../C5-Textures-TPK/C5-Textures-TPK.md)–[10](../C10-Geometry-IO/C10-Geometry-IO.md)).
- **Vault keys** — every tuning and gameplay field, by `lookup2` ([Chapters 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)–[14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)).
- **Smackable/emitter/collision keys** — `assetHash`, `paramHash`, `emitterHash` ([C63.9](../C63-Collision-World/09-smackables-emitters.md)).
- **Section, scenery, and event identifiers** — throughout the world data
  ([Chapters 15](../C15-Track-Streaming/C15-Track-Streaming.md)–[18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)).

So a *single* artifact — the name dictionary ([C77.3](03-name-dictionary.md)) — resolves identifiers *across the
entire book*. Recover a name once and it lights up wherever its hash appears: a `paramHash` in a smackable record
([C63.9](../C63-Collision-World/09-smackables-emitters.md)), the same hash as a vault key
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), the same name harvested from the exe
([C50.2](../C50-Verification-Methodology/02-byte-verification.md)). The dictionary is the *cross-cutting key* that the
per-chapter decodings all draw on — which is why hash recovery is the fitting *finale*: it's the one technique whose
payoff is *universal*, turning numbers into names everywhere at once.

## The recursive payoff

Hash recovery has a uniquely *recursive* reward ([C77.3](03-name-dictionary.md)): recovered names *improve recovery
itself*. A name resolved in one chapter (a surface `wetpaved`, a part `PART_EN_*`) enters the dictionary and both
*labels its own hash* and *seeds new candidates* (the `PART_EN_*` convention, related surfaces) that resolve hashes in
*other* chapters. So progress compounds across the whole book: the more of the book you've decoded, the more names you
have; the more names you have, the more of the book you can decode. The vault's **66.8%**
([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md)) and the exe's harvested strings and the cars' `PART_*`/`KIT*`
conventions all feed *one* growing dictionary, and each feeds the others.

This is the deep reason the book's chapters *reinforce* rather than merely *sit beside* each other: they share the
name dictionary. A convention found in the cars cluster ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md))
resolves a hash in the world data; a class name from the vault recon ([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md))
labels a physics object ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)). The dictionary is the
connective tissue of the book's *evidence* — the reason decoding one system helps decode the next.

## From bytes to names to understanding

Zoom all the way out and the book is one long arc, and hash recovery is its final step:

```
BYTES        →  STRUCTURE     →  NAMES         →  UNDERSTANDING
(raw files)     (Ch.1 chunks,    (Ch.77 hash      (the whole book:
                 formats)         recovery)        what it all means)
```

- **Bytes → structure** — the container model ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) and
  the format chapters turn raw files into *chunks and records*.
- **Structure → names** — hash recovery (this chapter) turns the *opaque identifiers* in those records into *legible
  names*.
- **Names → understanding** — a named, structured dataset is *readable*: `PART_EN_COLD_AIR_INTAKE_SYSTEM` in a car's
  vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) is *understood*, not just *located*.

So hash recovery is the step that turns a *parsed* game into an *understood* one. Structure tells you *where* things
are; names tell you *what* they are; together they're comprehension. This is why the chapter closes the book: with the
formats decoded ([Chapters 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)–[76](../C76-Advanced-RE/C76-Advanced-RE.md))
and the names recovered (this chapter), Most Wanted's data is not just *readable* but *legible* — the goal the whole
book has been building toward.

## The last frontier

And yet — honestly ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)) — the dictionary is *never quite
complete*. There will always be a residue: the long, arbitrary names ([C77.4](04-bruteforce-frontier.md)) that no
source has yet yielded, hashes like `0x0743CFB1` still labelled only by their number. So the book ends not with a
claim of *completeness* but with a *precisely-mapped frontier*: the large majority of identifiers named, the exact
remainder bounded and characterised, and the concrete paths to close each gap stated
([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md)). That is the truest possible ending for a
*verification-first* book — not "we decoded everything" (a claim no honest RE can make) but "here is what is known, to
what confidence, and exactly what remains." The name dictionary is the master key that opens most of the doors; the
frontier is the honest record of the few still locked, and the map to their keys.

## RE implications

- **One key for every chapter** — the name dictionary resolves hashes across the whole book (assets, vault, world,
  events).
- **Recursive payoff** — recovered names seed conventions that resolve hashes elsewhere; progress compounds book-wide.
- **Bytes → structure → names → understanding** — hash recovery is the step from *parsed* to *understood*.
- **The last frontier** — the dictionary is never complete; the book ends with a mapped frontier, not a completeness
  claim.

---

### Key takeaways

- The name dictionary is the **master key** — hashes pervade **every** chapter (assets `JOAAT`, vault `lookup2`,
  smackable/emitter/section keys), so one cross-cutting artifact makes the **whole book's data legible**.
- Hash recovery's payoff is **recursive** — a name recovered in one chapter **seeds conventions** that resolve hashes
  in others — so the chapters share **one growing dictionary**, the connective tissue of the book's evidence.
- The book is one arc — **bytes → structure → names → understanding** — and hash recovery is the step from
  **parsed** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) to **understood**: structure says
  *where*, names say *what*, together comprehension.
- The book ends not with **completeness** but with a **precisely-mapped frontier** — most identifiers named, the exact
  remainder (`0x0743CFB1`-class hashes) bounded, and the paths to close each gap stated — the truest ending for a
  **verification-first** work ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)).
- The dictionary opens most of the doors; the **honest frontier** is the map to the few still locked.

**This completes Chapter 77 and the C68–C77 batch.** See the [book index](../README.md) for the full chapter map.

**Sources:** the three hash functions verified in `speed.exe` — `JOAAT` ([C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)),
the bin-sum hash ([C2.3](../C2-Identifiers-And-Hashing/03-bin-sum-hash.md)), and `lookup2` (seed `0xABCDEF00` ×50, mix
`0x9E3779B9` ×4, [C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md)); ~66.8% of vault keys resolved under
`lookup2` ([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md)). Candidate names harvested from the executable's
strings ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)) and MW's naming conventions
([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)). Method: [Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md).
