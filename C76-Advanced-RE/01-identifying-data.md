# C76.1 — Identifying Unknown Data

> **The one-sentence version:** the first task on an unfamiliar format is *classification* — is this region a string
> table, a record array, code, or floats? — answered by structural tells: fourcc block markers, repeated record
> markers, a consistent stride, and readable string tables, as the `attributes.bin` block map shows.

[← Chapter 76 hub](C76-Advanced-RE.md) · [Next: C76.2 — Recovering a schema →](02-recovering-schema.md)

---

## The classification question

Faced with a region of unknown bytes, the first question is not "what does this field mean?" but "**what *kind* of
data is this?**" — because the answer dictates every subsequent technique:

- **A string table** — read it directly; the names are often the Rosetta stone for everything else.
- **A record array** — find the stride and the record structure; the count and marker come next.
- **Code** — disassemble it ([C76.3](03-static-vs-dynamic.md)); look for the functions that read the data.
- **Floats / packed numbers** — check for plausible value ranges and known constants.

Getting the *kind* right first is what turns a wall of hex into a tractable problem. Guess wrong — treat a record
array as a string blob — and every later step is confused. So classification is the foundation, and it's done by
*structure*, before meaning.

## The structural tells

Formats announce their structure through a handful of reliable tells ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)):

- **Fourcc block markers** — four-byte ASCII tags (`ErtS`, `NtaD`) at block boundaries
  ([C2.1](../C2-Identifiers-And-Hashing/01-joaat-asset-hash.md)). Scanning for printable-ASCII 4-tuples on aligned
  offsets reveals the top-level layout.
- **Repeated record markers** — a constant value at a fixed stride (`0xEFFECADD` at the head of every vault record)
  marks *record boundaries*: find the marker's period and you've found the record size.
- **Consistent stride** — records of fixed size repeat at a fixed interval; the gap between markers *is* the stride.
- **Readable string tables** — runs of NUL-terminated printable text, often length-prefixed or counted, are usually
  the *names* that key everything else.
- **Entropy** — compressed/encrypted data looks random (high entropy); code and structured data don't. A JDLZ
  boundary ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) shows as an entropy cliff.

These tells are *format-agnostic* — they work on a file you've never seen, because they're properties of *how binary
data is organised*, not of the specific game. Reading them is the craft of "seeing structure in hex."

## The vault block map: a worked classification

The `attributes.bin` file ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) is the model of
classification-by-structure. Before decoding a single field, its *shape* is legible from the tells:

```
offset      tag     size        classified as
0x80        ErtS    31,136 B     string table — 1,961 names (classes, fields, surfaces, VFX, AI)
0x55C00     NpeD    48 B         small index/dependency table
0x55C30     NrtS    16 B         small symbol table
0x55C40     NtaD    162,592 B    typed records — 4,732 markers (0xEFFECADD), 12–392 B each
```

Every classification here comes from a tell: the **fourcc tags** (`ErtS`/`NtaD`) mark the blocks; the **`ErtS` block**
is a string table (readable names); the **`NtaD` block** is a record array (the `0xEFFECADD` marker repeats 4,732
times, giving the record count and, by the gaps, the 12–392 B variable stride). So *before* knowing what any field
*means*, you know the file is **a string table + small index tables + a big typed-record array** — the skeleton on
which the schema ([C76.2](02-recovering-schema.md)) hangs.

> ✅ *Verified:* `GLOBAL/attributes.bin` (689,728 B) opens with `ErtS` @ `0x80`, `NpeD` @ `0x55C00`, `NrtS` @
> `0x55C30`, `NtaD` @ `0x55C40`; the `NtaD` block contains **4,732** records each marked `0xEFFECADD` — all confirmed
> by direct read.

## Names first

A recurring lesson: **read the string table first**. The `ErtS` block's 1,961 names — classes, fields, surfaces, VFX,
AI ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) — are the *vocabulary* of everything else in
the file. Once you can read the names, the records ([C76.2](02-recovering-schema.md)) become *nameable*: a record's
key can be matched to a name ([C76.4](04-building-readers.md)), a field's hash resolved to a string
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)). The string table is the format's *self-description*
([C50.2](../C50-Verification-Methodology/02-byte-verification.md)) — the same reason the executable's own strings make
the engine legible. So the identification order is: find the blocks, read the strings, *then* attack the records —
names before meaning.

## RE implications

- **Classify first** — string table, record array, code, or floats? — the *kind* dictates every later technique.
- **Structural tells** — fourcc markers, repeated record markers, stride, string tables, entropy — format-agnostic.
- **The vault block map** — `ErtS` (strings) + small index tables + `NtaD` (4,732 records) — shape legible before
  meaning.
- **Names first** — read the string table; it's the vocabulary that makes records nameable.

---

### Key takeaways

- The first question on unknown data is **"what *kind* is this?"** — string table, record array, code, or floats —
  because the *kind* dictates every subsequent technique; classify by **structure, before meaning**.
- The **structural tells** are format-agnostic: **fourcc block markers** (`ErtS`/`NtaD`), **repeated record markers**
  (`0xEFFECADD` → record boundaries & count), **stride**, **string tables**, and **entropy** (compression shows as an
  entropy cliff).
- The **`attributes.bin` block map** is the worked model — its shape (**string table + index tables + 4,732-record
  array**) is legible from the tells *before* any field is decoded.
- **Read the string table first** — the `ErtS` names are the format's **vocabulary**, making records nameable and
  hashes resolvable ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)); names before meaning.
- Verified: the `attributes.bin` block map and its record marker count, by direct read of the retail file.

**Continue:** [C76.2 — Recovering a schema](02-recovering-schema.md) · [Chapter 76 hub](C76-Advanced-RE.md)
