# C30.3 — Labels.bin

> **The one-sentence version:** `Labels.bin` is the shared, language-independent table of label names — the
> stable keys (`ACCEPT`, `ACCELERATION_HEADER`, `ACH_1000RACES_DESC`) that every UI string references and that
> map to the string ids the language files resolve.

[← C30.2 — The string-table format](02-table-format.md) · [Chapter 30 hub](C30-Localization-Labels.md) ·
[Next: C30.4 — The language files →](04-language-files.md)

---

## The label table

`Labels.bin` holds the **label names** — the readable identifiers the UI uses for every piece of text. Verified
sample from the retail file: `GUIDE`, `ACCELERATION_HEADER`, `ACCEPT`, `ACCEPT_DATE`, `ACH_1000RACES_DESC`,
`ACH_1000RACES_DS`, `ACH_1000RACES_HT`, `ACH_2CARS_DESC`, … These are the keys stage-1 of the lookup produces
([C30.1](01-label-system.md)): a screen says "show `ACCEPT`," and `Labels.bin` is where `ACCEPT` becomes a
string id.

The naming is systematic and self-documenting, like the geometry usage names
([C7.4](../C7-Materials-TexAnim/04-usage-names.md)):

- **Screen/context prefix** — `ACH_` (achievement), menu prefixes, car-name prefixes.
- **Suffix roles** — `_DESC` (description), `_DS`/`_HT` (variants), `_HEADER`, `_DATE`.

So `ACH_1000RACES_DESC` is "the description text for the 1000-races achievement." Reading the label tells you
what the string is *for*, before you see the string.

## Language-independent

The defining property of `Labels.bin` is that it is **shared across all languages** — there is one `Labels.bin`,
not one per language. The labels (and the ids they map to) are the same for English, French, Chinese, and every
other language; only the *strings* those ids resolve to differ, in the per-language files
([C30.4](04-language-files.md)). So `Labels.bin` is the **schema** of the game's text — "what text exists" —
and the language files are the **data** — "what it says in each language."

This split is what makes the label system localizable ([C30.1](01-label-system.md)): a label means the same
*concept* everywhere, and each language provides the words.

## The label → id map

Mechanically, `Labels.bin` maps a **label name** to a **string id** ([C30.2](02-table-format.md)). The UI
resolves a label to its id via `Labels.bin`, then resolves the id to text via the active language file. So
`Labels.bin` is the *join* between the human-readable label the UI uses and the compact id the language files
key on:

```
label name (ACCEPT)  ──Labels.bin──▶  string id  ──language file──▶  "Accept"
```

Whether the lookup keys on the label *string* or a *hash* of it ([C30.2](02-table-format.md)), the result is the
string id that every language file shares.

> ✅ *Verified:* `Labels.bin` is a `0x00039000` table containing the label names (`ACCEPT`,
> `ACCELERATION_HEADER`, achievement labels, …), shared across languages, mapping labels to string ids.
> 🟡 *Reasoned:* whether the key is the label string or its hash is the format's detail; the label-name content
> and the label→id role are verified.

## Why keep the labels at all

Most binary formats discard names and keep only ids ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)).
Keeping the label names in `Labels.bin` is deliberate:

- **Authoring by name.** Screens and tools reference labels by readable name, not opaque id — far more
  maintainable.
- **Self-documenting.** The label table *is* a catalogue of the game's text, readable directly.
- **Stable keys.** A label name is a durable identity; the underlying id can be regenerated, but the label
  stays.

It's the same choice the vault ([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)) makes — ship the names,
not just the hashes — for the same maintainability reasons.

## Editing implications

- **Adding text needs a label here.** A new UI string requires a new entry in `Labels.bin` (label → id) plus a
  string per language ([C30.6](06-editing-text.md)).
- **Don't rename labels casually.** Screens reference labels by name/id; renaming orphans the references.
- **Labels are shared** — an edit to `Labels.bin` affects all languages (it's the language-independent table).
- **Keep ids consistent** — the id a label maps to must exist in every language file.

---

### Key takeaways

- `Labels.bin` is the **label-name table** — the readable keys (`ACCEPT`, `ACH_*`, …) every UI string
  references.
- It's **language-independent** — one shared file; labels/ids are the same across languages, only strings
  differ.
- It maps **label name → string id**, the join between the UI's readable label and the language files' compact
  id.
- Keeping label names (not just ids) makes the UI authorable-by-name, self-documenting, and stably keyed.
- Adding text needs a label here + a string per language; don't rename labels; edits affect all languages.

**Continue:** [C30.4 — The language files](04-language-files.md) · [Chapter 30 hub](C30-Localization-Labels.md)
