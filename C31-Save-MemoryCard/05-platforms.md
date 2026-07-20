# C31.5 — Platforms & Memory Cards

> **The one-sentence version:** the `.loc` files are per-locale, and saves are per-platform — the PC save and a
> console memory-card save wrap similar payloads differently, with the memory card adding its own block/icon
> framing around the game's `.loc` portion.

[← C31.4 — Integrity: checksums & validation](04-integrity.md) · [Chapter 31 hub](C31-Save-MemoryCard.md) ·
[Next: C31.6 — Editing saves →](06-editing-saves.md)

---

## Per-locale `.loc` files

The `MEMCARD/` directory holds `.loc` files **per locale** — `LOCALE_ENGLISH.loc`, `LOCALE_GERMAN.loc`,
`LOCALE_FRENCH.loc`, `LOCALE_BRAZILIAN_PORTUGUESE.loc`, `LOCALE_GREEK.loc`, and more. These are locale-specific
save-related data (the naming mirrors the localization files, [Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) —
the `LOCH`/`LOCI` containers ([C31.1](01-loch.md)) tailored per locale, likely holding localized save metadata
(save-slot descriptions, formatted text) alongside the payload structure.

## Per-platform saves

More broadly, saves are **per-platform**. Most Wanted shipped on PC, PS2, Xbox, GameCube, and more, and each
platform stores saves differently:

- **PC** — a save file on disk (the `.loc`/`LOCH` container).
- **Console memory cards** — the save wrapped in the platform's memory-card format (blocks, an icon, platform
  metadata) around the game's `.loc` portion.

The **payload** ([C31.3](03-save-payload.md)) — the career progress — is similar across platforms (same game,
same progress), but the **container** differs: the memory-card framing is platform-specific, and the `.loc`
inside is the game's portion. So "the save" splits into the game's data (portable) and the platform's wrapper
(not).

## The memory-card wrapper

On consoles, a memory card adds its own layer around the game's save
([C31.1](01-loch.md)):

- **Card blocks** — the memory card allocates the save in blocks; the save occupies some number of them.
- **An icon and title** — the card shows a save icon and description (often localized —
  [C31.5 locale files](05-platforms.md)).
- **Card metadata** — platform-specific headers the console's save manager reads.

The game's `LOCH` container sits inside this card wrapper. So reading a console save means peeling the card
format first, then the `LOCH` — two nested containers, the outer platform-owned, the inner game-owned.

> ✅ *Verified:* `.loc` files are per-locale (`LOCALE_*.loc`) `LOCH` containers; saves live under `MEMCARD/`.
> 🟡 *Reasoned:* the per-platform container differences and memory-card wrapping are the standard multi-platform
> save story; the `.loc`/`LOCH` game portion is verified.

## Portability

The split matters for anyone moving saves between platforms or tools:

- **The payload is (largely) portable** — the career progress is the same game state.
- **The container is not** — a PC `.loc` and a PS2 memory-card save wrap it differently, with different
  integrity ([C31.4](04-integrity.md)) and framing.

So converting a save between platforms is repackaging the payload into the target's container (and recomputing
its integrity), not a byte copy. The `LOCH`/`LOCI` structure is the game's consistent core; the platform layer
around it varies.

## Editing implications

- **Identify the container layers** — on console, peel the memory-card wrapper before the `LOCH`
  ([C31.1](01-loch.md)).
- **The payload is the portable part** — progress transfers; the container/integrity is per-platform.
- **Respect locale files** — `LOCALE_*.loc` are per-locale; edit the one matching the target locale.
- **Recompute the right integrity** — each platform's container may protect the save differently
  ([C31.4](04-integrity.md)).

---

### Key takeaways

- `.loc` files are **per-locale** (`LOCALE_*.loc`) `LOCH` containers under `MEMCARD/`.
- Saves are **per-platform**: similar payload, different container (PC file vs console memory-card framing).
- Consoles add a **memory-card wrapper** (blocks, icon, metadata) around the game's `LOCH` — two nested
  containers.
- The **payload** is largely portable; the **container** and integrity are platform-specific.
- Peel the platform wrapper first, treat the payload as the portable core, and recompute the correct
  per-platform integrity.

**Continue:** [C31.6 — Editing saves](06-editing-saves.md) · [Chapter 31 hub](C31-Save-MemoryCard.md)
