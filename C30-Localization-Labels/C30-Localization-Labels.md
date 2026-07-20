# Chapter 30 — Localization: String Tables & the Label System

> **Goal of this chapter:** decode how the game shows the right text in the right language — a per-language
> string table plus a shared `Labels.bin`, joined by an ID-based lookup — so you can read, translate, and add
> UI text without touching a single screen.

The UI ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) never stores literal words. It
references a **label**; the label resolves to a **string id**; the **active language file** maps that id to the
actual string; and the fonts ([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) render it. This
indirection is what lets one set of screens serve every language. This chapter decodes the tables and the
lookup.

> **Verified against retail data.** `LANGUAGES/` holds a per-language string table each — `English.bin`,
> `French.bin`, `German.bin`, `Chinese.bin`, `Korean.bin`, … — plus a shared `Labels.bin`. All are the **same
> chunk format** (chunk id `0x00039000`, with a header of table offsets): verified, `English.bin` and
> `Labels.bin` open with identical header words (`0x10, 0x11A8, 0x1814, 0xA554, 0x100`). `Labels.bin` contains
> the **label names** (`ACCELERATION_HEADER`, `ACCEPT`, `ACH_1000RACES_DESC`, …); the language files contain
> the **translated strings**.

---

## Deep-dive pages

- [C30.1 — The label system](01-label-system.md): the ID-based lookup that makes the UI language-neutral.
- [C30.2 — The string-table format](02-table-format.md): the `0x00039000` chunk — header, hash/index, string
  pool.
- [C30.3 — Labels.bin](03-labels.md): the label-name table every UI string references.
- [C30.4 — The language files](04-language-files.md): the per-language string tables and text encoding.
- [C30.5 — Image text & fonts](05-image-text.md): text that's a picture, and the font link.
- [C30.6 — Translating & adding text](06-editing-text.md): editing strings, adding labels, keeping the lookup
  valid.

---

## 30.1 Text is by ID, not literal

The core of localization is a three-stage, ID-based lookup ([C30.1](01-label-system.md)):

```
UI screen (.fng)         Labels.bin              English.bin (active language)
  "show label ACCEPT"  →  ACCEPT = string-id  →  string-id = "Accept"
                                                 French.bin: string-id = "Accepter"
```

The screen names a **label**; `Labels.bin` maps the label to a **string id**; the **active language file** maps
the id to the string. Change the active language file and every string changes at once, with no change to
screens or labels — the whole point of the system.

## 30.2 One table format, three offsets

Every localization file — the language tables and `Labels.bin` — is the **same** chunk (`0x00039000`) with a
header of **offsets to sub-tables**: an index/hash structure keyed by id, and a **string pool** holding the
actual text ([C30.2](02-table-format.md)). Verified, `English.bin` and `Labels.bin` share identical header
words, confirming one format. Reading a string is: find its id in the index, follow the offset into the string
pool.

## 30.3 Labels.bin: the shared keys

`Labels.bin` is the **label table** — the stable names every UI string references (`ACCEPT`,
`ACCELERATION_HEADER`, achievement labels, car-name labels, menu labels). It is **language-independent**: the
labels are the same for every language; only the strings they resolve to differ. So `Labels.bin` is the schema
of "what text the game has," and the language files are the per-language data ([C30.3](03-labels.md)).

## 30.4 The language files: the strings

Each `LANGUAGES/*.bin` is one language's **string table** — the same `0x00039000` format, mapping string ids to
that language's text ([C30.4](04-language-files.md)). Swapping which one is active is what changes the game's
language. The text is encoded to cover the language's script (Latin, CJK, etc.,
[C28.5](../C28-Fonts-Glyphs/05-codepoints.md)), and the fonts must have glyphs for it
([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)).

## 30.5 Some text is a picture

Where a word is **art** — a logo, a stylised graphic — it isn't a string but an **image** in a per-language TPK
(`LanguageTextures.bin`), localized by swapping the picture ([C30.5](05-image-text.md)). So the UI has two
kinds of localizable text: **glyph text** (strings + fonts) and **image text** (a localized atlas), covered
here and in [Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md).

---

### Key takeaways

- Text is **ID-based**: screen → label → string id → string (active language) → glyphs — the UI is
  language-neutral.
- All localization files share one format: chunk **`0x00039000`** with header offsets to an index and a string
  pool (verified identical headers).
- `Labels.bin` holds the **label names** (language-independent keys); the `LANGUAGES/*.bin` files hold the
  per-language **strings**.
- Swapping the active language file changes every string at once, with no screen changes.
- Some "text" is **image text** (a localized atlas), the other kind of localizable text.

**Next:** [Chapter 31 — Save Data & Memory-Card Containers](../C31-Save-MemoryCard/C31-Save-MemoryCard.md): where
the player's progress is stored.
