# C31.3 — The Save Payload

> **The one-sentence version:** inside the `LOCI` items is the career payload — the player's Blacklist progress,
> cars and their tuning, money and bounty, unlocks, and settings — the content the container exists to protect.

[← C31.2 — LOCI items](02-loci.md) · [Chapter 31 hub](C31-Save-MemoryCard.md) ·
[Next: C31.4 — Integrity: checksums & validation →](04-integrity.md)

---

## What a save holds

The payload inside the `LOCI` items ([C31.2](02-loci.md)) is the player's **progress** — everything a save must
remember so the game resumes where it left off:

- **Career / Blacklist** — position on the Blacklist ladder ([plan Chapter 58](../README.md)), events won,
  milestones, story progress.
- **Cars** — which cars are owned, and each car's **tuning** and customization
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — parts, paint, performance.
- **Economy** — money, bounty, and the currency of progression
  ([C14.2](../C14-Vault-Pursuit-Surfaces/02-heat-bounty.md)).
- **Unlocks** — cars, parts, markers, and content the player has earned.
- **Settings** — options, controls ([plan Chapter 53](../README.md)), and preferences.
- **Statistics** — races, pursuits, achievements ([the `ACH_*` labels](../C30-Localization-Labels/03-labels.md)).

This is the *state* the game accumulates as you play — the difference between a fresh start and a 60%-complete
career.

## References into the game's data

Much of the save is **references**, not copies — it names game content by the same ids/keys the rest of the
engine uses ([the "reference resolves to data" pattern](../C7-Materials-TexAnim/03-texture-binding.md)):

- A owned car is a **reference** to a car definition ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)),
  plus the player's tuning deltas — not a copy of the car.
- An unlock is a **flag/id** referencing the unlockable content.
- Blacklist progress is **indices/flags** into the career structure.

So a save is compact: it stores *what the player has done and chosen*, referencing the game's fixed content
rather than duplicating it. Reading a save means resolving those references against the game's data
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md), [C30](../C30-Localization-Labels/C30-Localization-Labels.md)).

> 🟡 *Reasoned:* the payload's content (career/cars/economy/unlocks/settings) is the standard racing-career save
> and consistent with the game's systems (Blacklist, car tuning, economy); the ✅ verified facts are the
> `LOCH`/`LOCI` container ([C31.1](01-loch.md)–[C31.2](02-loci.md)) that holds it. The precise payload field
> layout is the save format's detail.

## The player's overrides

A useful way to think of a save is as the player's **overrides** on a fresh game — the same override model as
the vault ([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)):

- A **fresh game** is the baseline (no cars, Blacklist #15, no money).
- A **save** records the deltas — the cars bought, the events won, the money earned, the options changed.

So the save is sparse in spirit: it stores what *differs* from a new game, referencing the game's fixed content
for everything unchanged. This is why saves are small relative to the game's data — they're the player's diff.

## Editing implications

- **Edit progress by editing the payload** — money, cars owned, Blacklist position, unlocks
  ([C31.6](06-editing-saves.md)).
- **References must be valid** — an owned-car reference must name a real car
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); an unlock id must exist.
- **Consistency matters** — the save's pieces should be coherent (e.g., a car referenced as owned should have
  valid tuning); inconsistent state can confuse the game.
- **Integrity must be recomputed** ([C31.4](04-integrity.md)) — any payload edit invalidates the checksum.

---

### Key takeaways

- The payload is the **career save**: Blacklist progress, cars + tuning, money/bounty, unlocks, settings,
  stats.
- It's mostly **references** (car ids, unlock flags, indices) into the game's fixed content — not copies — so
  it's compact.
- Think of a save as the player's **overrides** on a fresh game — the deltas from baseline.
- Editing progress edits the payload; keep references valid and state coherent.
- Any payload edit requires recomputing the save's integrity data (C31.4).

**Continue:** [C31.4 — Integrity: checksums & validation](04-integrity.md) · [Chapter 31 hub](C31-Save-MemoryCard.md)
