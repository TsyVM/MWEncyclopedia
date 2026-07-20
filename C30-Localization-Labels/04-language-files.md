# C30.4 — The Language Files

> **The one-sentence version:** each `LANGUAGES/*.bin` is one language's string table — the same `0x00039000`
> format, mapping string ids to that language's text — so swapping which file is active changes the game's
> language, and each must be encoded for its script.

[← C30.3 — Labels.bin](03-labels.md) · [Chapter 30 hub](C30-Localization-Labels.md) ·
[Next: C30.5 — Image text & fonts →](05-image-text.md)

---

## One file per language

`LANGUAGES/` holds a string table per language — `English.bin`, `French.bin`, `German.bin`, `Italian.bin`,
`Dutch.bin`, `Danish.bin`, `Finnish.bin`, `Chinese.bin`, `Korean.bin`, and more. Each is the **same
`0x00039000` format** as `Labels.bin` ([C30.2](02-table-format.md)) — verified identical headers — but its
string pool holds *that language's* text. So the files are parallel: same ids (from `Labels.bin`,
[C30.3](03-labels.md)), different strings.

```
string id 0x…  →  English.bin: "Accept"
               →  French.bin:  "Accepter"
               →  German.bin:  "Annehmen"
               →  Chinese.bin: "接受"
```

The **active** language file ([C30.1](01-label-system.md)) is the one the game resolves ids through; the others
sit unused until the player switches language.

## Same ids, different strings

Because every language file keys on the **same string ids** (the ones `Labels.bin` maps labels to), the id is
the language-independent handle and the string is the per-language data:

- `Labels.bin` (shared) — label → id.
- `English.bin` (per language) — id → "Accept".
- `French.bin` (per language) — id → "Accepter".

This parallelism is what makes switching languages a single-file swap: the ids don't move, so resolving the
same label through a different language file yields that language's string, with no other change
([C30.1](01-label-system.md)).

## Encoding for the script

A language file's strings must be encoded to represent its **script** ([C28.5](../C28-Fonts-Glyphs/05-codepoints.md)):

- **Latin languages** (English, French, German, …) — extended-ASCII / UTF-8 / UTF-16 covering accented Latin.
- **CJK** (Chinese, Korean) — a wide encoding (UTF-16 or a multibyte codepage) for thousands of ideographs.

The `LANGUAGES/` directory even shows encoding variants in the auxiliary text (e.g. `agree-ucs2.chi` is UCS-2/
UTF-16 Chinese, `agree.chi` a UTF-8 variant) — a hint that the localization pipeline handles multiple encodings
per script. The strings' encoding must match what the renderer and fonts expect
([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)), so a translation edit must produce correctly-encoded
bytes.

> ✅ *Verified:* `LANGUAGES/` holds per-language `*.bin` string tables (Latin and CJK), all in the `0x00039000`
> format matching `Labels.bin`; encoding variants (UCS-2 vs UTF-8) appear for CJK.
> 🟡 *Reasoned:* the exact per-language string encoding within the pool is the format's detail; the one-file-per-
> language, shared-id structure is verified.

## Fonts must match

A language file provides the *characters*; the **fonts** must have *glyphs* for them
([C28.5](../C28-Fonts-Glyphs/05-codepoints.md)). So localization is two coordinated parts:

- **Strings** (this chapter) — the words, per language.
- **Fonts** ([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) — the letterforms to draw them.

Both must cover the language: a Chinese string is useless if the active font has no CJK glyphs, and a CJK font
is wasted without Chinese strings. Editing text in a language may therefore require the matching font glyphs
([C30.6](06-editing-text.md)).

## Editing implications

- **Translate by editing the language file's string pool** ([C30.2](02-table-format.md)) — same ids, new text.
- **Encode correctly** — produce bytes in the language's expected encoding, or text renders as garbage.
- **Match the fonts** — the active font must have glyphs for the characters used
  ([C28.5](../C28-Fonts-Glyphs/05-codepoints.md)).
- **Keep ids aligned across languages** — a string added to one language file needs the same id in the others
  (and in `Labels.bin`).

---

### Key takeaways

- Each `LANGUAGES/*.bin` is one language's string table, same `0x00039000` format, different string pool.
- All language files key on the **same string ids** (from `Labels.bin`) — id is language-independent, string is
  per-language.
- Switching language is a single-file swap because ids don't move.
- Strings are **encoded per script** (Latin, CJK); encoding variants (UCS-2/UTF-8) exist for CJK.
- Fonts must have glyphs for the language; translate the pool with correct encoding and matching font glyphs.

**Continue:** [C30.5 — Image text & fonts](05-image-text.md) · [Chapter 30 hub](C30-Localization-Labels.md)
