# C77.3 — Building a Name Dictionary

> **The one-sentence version:** the core recovery tool is a name dictionary — a growing list of candidate names,
> each pre-hashed under all three functions — fed by harvesting the executable's own strings, exploiting naming
> conventions, and adding wordlists, so that every name found reveals conventions that generate the next.

[← C77.2 — The three functions & their reversibility](02-three-functions.md) · [Chapter 77 hub](C77-Hash-Recovery.md) ·
[Next: C77.4 — Brute-forcing & the frontier →](04-bruteforce-frontier.md)

---

## The dictionary is a table

A **name dictionary** is the workhorse of hash recovery ([C77.2](02-three-functions.md)) — conceptually a table of
candidate names, each pre-hashed under all three functions:

```
name                              JOAAT       bin-sum     lookup2(0xABCDEF00)
TOPSPEED                          0x……        0x……        0x0743CF..(example)
PART_EN_COLD_AIR_INTAKE_SYSTEM    0x……        0x……        0x……
wetpaved                          0x……        0x……        0x……
BMWM3GTR                          0x……        0x……        0x……
```

To resolve an opaque ID, you *look it up* — scan the column for its hash function ([C77.2](02-three-functions.md)) for
a matching value; if found, you have the name. Because hashing is forward-cheap ([C77.2](02-three-functions.md)),
building the table (hash every candidate ×3) and querying it (a lookup) are both fast. The dictionary is thus a
*precomputed inverse* — not a true inverse ([C77.1](01-recovery-problem.md)), but a lookup that resolves any hash
whose name you've *thought to include*. The whole art is **getting the right names into it**.

## Harvesting candidates

Candidate names come from three wells, in rough order of yield:

- **The executable's own strings** ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)) — the richest
  source. `speed.exe` is *full* of names: `PART_*`, `OL_*`, the `M*`/`E*` messages, the class table
  ([C76.2](../C76-Advanced-RE/02-recovering-schema.md)), surface names, effect names. Every string in the binary is a
  candidate — and many hashes in the *data* are hashes of strings in the *code*. Harvesting the exe's strings alone
  resolves a large fraction of IDs, because the game kept the names it needed as strings even as it hashed others.
- **Naming conventions** — MW's names are *systematic* ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)):
  `PART_<FAMILY>_<DESC>`, `<CAR>_KIT00_<PART>`, `EShow<Screen>`, `MEnter<State>`. Once you know a convention, you
  *generate* candidates — every `PART_<FAMILY>_*` combination, every `<CAR>_KIT00_*` part — and hash them all.
  Conventions turn a few known names into *thousands* of generated candidates.
- **Wordlists** — general English, car-domain vocabulary (makes, models, part names), and EA/NFS-specific terms.
  These catch the names that *don't* follow a convention — a one-off surface or effect name.

The best dictionaries combine all three: harvested exe strings (high-confidence real names), convention-generated
candidates (systematic coverage), and wordlists (the long tail).

> ✅ *Verified:* the executable is dense with candidate names — the `PART_*`, `OL_*`, `M*`/`E*`, `Customize*`, class,
> and surface strings decoded across the book are all in `speed.exe`, and each is a dictionary candidate
> ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).

## The dictionary compounds

The dictionary's power is that it **compounds** — every name found makes the *next* easier:

- **Names reveal conventions.** Finding `PART_EN_COLD_AIR_INTAKE_SYSTEM` reveals the `PART_EN_*` pattern
  ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)); now you generate *every* engine-part candidate.
- **Conventions reveal names.** Generating `PART_TU_STAGE_1/2/3` from the turbo convention
  ([C69.1](../C69-Performance-Upgrades-Tuning/01-classes-tiers.md)) resolves three more hashes — which may reveal a
  *stage* convention used elsewhere.
- **Names reveal vocabulary.** A recovered surface name (`wetpaved`) suggests related terms (`drypaved`, `wetgrass`)
  to try.

So recovery is a *virtuous cycle*: harvested names → inferred conventions → generated candidates → more recovered
names → more conventions. Each turn of the cycle grows the dictionary, which resolves more IDs, which reveals more
patterns. This is why the vault went from opaque to **66.8%** resolved ([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md))
once the right hash ([C77.2](02-three-functions.md)) unlocked the `ErtS` names — those names *were* the dictionary,
and matching them resolved two-thirds of the keys at a stroke. The remaining third
([C77.4](04-bruteforce-frontier.md)) is names *not yet in the dictionary* — the next turns of the cycle.

## Sharing dictionaries

A practical multiplier: **dictionaries are shareable**. A name recovered once — by anyone — can be added to a common
dictionary that resolves that hash for *everyone*, forever. So hash recovery is naturally a *collective* effort: the
community's pooled dictionary is far larger than any one person's, and it only grows. This is why long-decoded games
have near-complete name coverage — decades of pooled harvesting — and why contributing a recovered name is durable
work ([C76.4](../C76-Advanced-RE/04-building-readers.md)): unlike a one-off decode, a dictionary entry keeps resolving
its hash indefinitely, for anyone who uses the dictionary.

## RE implications

- **The dictionary is a table** — candidate names × three hash columns; resolve an ID by looking up its column.
- **Harvest from three wells** — the exe's own strings (richest), naming conventions (systematic), wordlists (the
  tail).
- **It compounds** — names → conventions → generated candidates → more names — a virtuous cycle.
- **Shareable** — a name recovered once resolves that hash for everyone; recovery is collective and cumulative.

---

### Key takeaways

- A **name dictionary** — candidate names each **pre-hashed under all three functions** — is a **precomputed inverse**
  ([C77.1](01-recovery-problem.md)): resolve an opaque ID by **looking it up** in the column for its hash function.
- Candidates come from three wells: **the executable's own strings** (richest — `speed.exe` is dense with names),
  **naming conventions** (systematic — generate every `PART_<FAMILY>_*`), and **wordlists** (the long tail).
- The dictionary **compounds** — **names reveal conventions, conventions generate candidates, candidates recover more
  names** — the virtuous cycle that took the vault to **66.8%** resolved once the right hash unlocked the `ErtS` names
  ([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md)).
- Dictionaries are **shareable and cumulative** — a name recovered **once** resolves that hash for **everyone,
  forever** — so recovery is a **collective** effort whose pooled dictionary only grows.
- The unresolved remainder is simply **names not yet in the dictionary** — the next turns of the cycle, or brute force
  ([C77.4](04-bruteforce-frontier.md)).

**Continue:** [C77.4 — Brute-forcing & the frontier](04-bruteforce-frontier.md) · [Chapter 77 hub](C77-Hash-Recovery.md)
