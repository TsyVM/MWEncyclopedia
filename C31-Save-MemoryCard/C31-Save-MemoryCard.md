# Chapter 31 — Save Data & Memory-Card Containers

> **Goal of this chapter:** decode where the player's progress lives — the `LOCH`/`LOCI` `.loc` containers and
> the career save payload inside them — so you can read and edit a save, and understand the container that
> wraps it.

Everything the player earns — the Blacklist progress, the cars owned, the money, the settings — is stored in a
**save**. On disk (and memory card) the save is a **`.loc` container**: a `LOCH` header wrapping `LOCI` items,
with the career payload inside. This chapter decodes that container and situates the save data.

> **Verified against retail data.** `MEMCARD/LOCALE_*.loc` files open with the magic **`LOCH`** (`0x4C4F4348`)
> followed by a header (`+0x04 = 0x14` header size, version `1`, count `1`, `+0x10 = 0x5C`), then a **`LOCI`**
> record (`+0x14 = "LOCI"`, `+0x18 = 0x48` size) — a container of inner items. The file carries a `LOCIH`
> marker, confirming the `LOCH`→`LOCI` nesting.

---

## Deep-dive pages

- [C31.1 — The LOCH container](01-loch.md): the outer header and what it wraps.
- [C31.2 — LOCI items](02-loci.md): the inner records inside a `LOCH`.
- [C31.3 — The save payload](03-save-payload.md): the career/progress data.
- [C31.4 — Integrity: checksums & validation](04-integrity.md): how a save is protected against corruption.
- [C31.5 — Platforms & memory cards](05-platforms.md): the `.loc` per-platform and per-locale story.
- [C31.6 — Editing saves](06-editing-saves.md): reading and modifying a save safely.

---

## 31.1 The LOCH container

A `.loc` file is a **`LOCH`** container ([C31.1](01-loch.md)) — a header (`LOCH` magic, header size, version,
item count, size) followed by its items. It is the save's outer wrapper: a small, self-describing header that
says how big the save is, what version it is, and how many items it holds. The `LOCH`/`LOCI` pairing is a
classic header/item container, like the ABK's `ABKC`/`BNKl` ([C19.3](../C19-Audio-Banks/03-abk-bnkl.md)) or the
event pack/chunk ([C25.1](../C25-NIS-Events/01-carp-scripts.md)).

## 31.2 LOCI items

Inside the `LOCH` are **`LOCI`** records ([C31.2](02-loci.md)) — the container's items, each with its own header
(`LOCI` tag, size) and payload. A `LOCI` holds a piece of the save; the `LOCH` counts and locates them. The
verified file has one `LOCI` of size `0x48`, with a `LOCIH` sub-marker — so the container nests: `LOCH` →
`LOCI` → (item header + data).

## 31.3 The save payload

Within the items is the **career save payload** ([C31.3](03-save-payload.md)) — the player's progress: Blacklist
position ([plan Chapter 58](../README.md)), cars owned and their tuning ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)),
money/bounty, unlocks, and settings. This is the *content* the container protects; reading a save is opening the
container to reach this payload.

## 31.4 Integrity

Saves are protected against corruption and tampering by **integrity data** — a checksum (and possibly a
signature) so the game can detect a damaged or edited save ([C31.4](04-integrity.md)). This matters for editing:
change the payload and you must recompute the integrity data, or the game rejects the save. It's the save's
version of the size-tree discipline — an invariant that must be repaired after an edit.

## 31.5 Platforms & locales

The `.loc` files are **per-locale** (`LOCALE_ENGLISH.loc`, `LOCALE_GERMAN.loc`, …) and, more broadly, saves are
**per-platform** — the PC save and a console memory-card save differ in container framing even where the payload
is similar ([C31.5](05-platforms.md)). The memory-card context (console) adds its own wrapping (card blocks,
icons); the `.loc` is the game's portion. So "the save" is the payload; the `.loc`/memory-card container is how
each platform stores it.

---

### Key takeaways

- The save is a **`.loc`** file: a **`LOCH`** container (magic, header, item count) wrapping **`LOCI`** items —
  verified.
- `LOCH`/`LOCI` is a header/item container (like `ABKC`/`BNKl`); items hold the save's pieces.
- Inside is the **career payload** — Blacklist progress, cars, money, unlocks, settings.
- Saves carry **integrity data** (checksum/signature) — recompute it after any edit or the game rejects the
  save.
- `.loc` files are per-locale and saves per-platform; the container is how each platform stores the payload.

**Next:** [Chapter 32 — The Runtime Class System & Object Model](../C32-Runtime-Class-System/C32-Runtime-Class-System.md):
the book pivots from files on disk to the running game.
