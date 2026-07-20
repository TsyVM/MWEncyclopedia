# C30.1 — The Label System

> **The one-sentence version:** the UI shows text through a three-stage ID lookup — screen names a label,
> `Labels.bin` maps it to a string id, the active language file maps the id to the string — so the screens are
> language-neutral and swapping the language file re-translates everything at once.

[← Chapter 30 hub](C30-Localization-Labels.md) · [Next: C30.2 — The string-table format →](02-table-format.md)

---

## Three stages

Showing a piece of UI text is a chain of indirection, each stage decoupling the next:

```
1. UI screen        → label      ("show label ACCEPT")
2. Labels.bin       → string id  (ACCEPT = id 0x…)
3. active language  → string     (id 0x… = "Accept" / "Accepter" / "Annehmen")
   English.bin / French.bin / German.bin …
```

The screen ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) knows only a **label** — a stable
name like `ACCEPT`. `Labels.bin` ([C30.3](03-labels.md)) turns that label into a **string id**. The **active
language file** ([C30.4](04-language-files.md)) turns the id into the actual **string** for the current
language. The fonts ([Chapter 28](../C28-Fonts-Glyphs/C28-Fonts-Glyphs.md)) then render it.

## Why two levels of indirection

Screen→label→id→string looks like a lot of hops, but each earns its place:

- **Screen → label** decouples *layout* from *content*: a screen references `ACCEPT`, not the word "Accept", so
  re-wording never touches the screen ([C27.5](../C27-FrontEnd-Shell-UI/05-ui-text.md)).
- **Label → id** decouples the *human name* from the *storage key*: labels are readable (`ACH_1000RACES_DESC`),
  ids are compact — and `Labels.bin` is the shared join.
- **Id → string** decouples *language* from everything: only this stage differs per language, so translating is
  editing one file.

The payoff is **localization for free**: change the active language file and every string in the game changes,
with no change to screens, labels, or ids. This is the same "reference resolves to data" indirection as
textures ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)) and vault fields
([C12.5](../C12-Reflection-Schema/05-resolving-values.md)), applied to text.

## Language-neutral screens

The consequence worth internalising: the **screens are language-neutral.** A menu layout
([C27.3](../C27-FrontEnd-Shell-UI/03-layout.md)) is authored once, in no particular language — it references
labels. The game ships one set of screens and many language files
([C30.4](04-language-files.md)); the active language decides what the (identical) screens display. This is why
a localized build doesn't re-author menus per language — it swaps the string table.

## The active language

At runtime one language file is **active** (per the player's setting), and it's the one stage-3 lookup uses.
Switching language switches which `LANGUAGES/*.bin` is active, and every subsequent label lookup resolves
through it — so the whole UI re-translates. `Labels.bin` (the label→id map) and the screens (label references)
don't change; only the active string source does.

> ✅ *Verified:* `Labels.bin` (label names) and the per-language `*.bin` files (strings) are the two tables of
> this lookup, both in the `0x00039000` format ([C30.2](02-table-format.md)); the label names are present in
> `Labels.bin`.
> 🟡 *Reasoned:* the exact label→id and id→string record encoding is the table format's detail
> ([C30.2](02-table-format.md)); the three-stage model and the two files are verified.

## Editing implications

- **Translate by editing the language file** — change stage-3 strings; screens and labels untouched
  ([C30.6](06-editing-text.md)).
- **Change which text a screen shows** by re-labelling it ([C27.5](../C27-FrontEnd-Shell-UI/05-ui-text.md)) —
  stage-1.
- **Add new text** by adding a label (`Labels.bin`) and a string per language ([C30.6](06-editing-text.md)).
- **Keep the chain intact** — a label with no id, or an id with no string, shows nothing.

---

### Key takeaways

- UI text is a three-stage lookup: **screen → label → string id → string** (active language) → glyphs.
- Each hop decouples something: layout from content, human name from key, language from all else.
- The payoff is **language-neutral screens** — one set of menus, many language files, swapped by the active
  language.
- Switching language switches the active `*.bin`; labels and screens don't change.
- Translate via the language file, re-label to change which text, add label+strings for new text; keep the chain
  valid.

**Continue:** [C30.2 — The string-table format](02-table-format.md) · [Chapter 30 hub](C30-Localization-Labels.md)
