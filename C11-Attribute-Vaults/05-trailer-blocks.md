# C11.5 — The Trailer Blocks

> **The one-sentence version:** after the data records come three small trailer blocks — `NpeD`
> (dependencies), `NrtS` (string references), and `NtaD` (the data directory, carrying the `0xEFFECADD`
> sentinel and record counts) — that index and cross-link the vault for the loader.

[← C11.4 — The data records](04-data-records.md) · [Chapter 11 hub](C11-Attribute-Vaults.md) ·
[Next: C11.6 — Navigating & editing the vault →](06-navigating-editing.md)

---

## The three blocks

At the trailer offset from the header (`0x55C00` in the worked file) sit three tagged blocks in sequence,
each a small fixed record:

```
NpeD @0x55C00   "NpeD"  size 0x30  count 2   … 0x18228ADE 0x610CE10B …  "bd.v…"
NrtS @0x55C30   "NrtS"  size 0x10  …
NtaD @0x55C40   "NtaD"  0x00027B20  0x58  0x11D(=285)  0x5E(=94)  0xEFFECADD  4 8 8 2 …
```

These are the vault's **indexes** — where the header block table and the record region are tied together.
Their four-character tags are the reversed spellings of their roles (a common EA convention): read `NpeD`,
`NrtS`, `NtaD` as **Dep-N**, **Str-N**, **Dat-N** — dependencies, strings, data.

## NpeD — dependencies

`NpeD` (dependency block, size 0x30, count 2 in the worked file) records the vault's **external
dependencies**: other data files or vault modules this one references, identified by hash and carrying small
descriptors (the trailing bytes include a version-like `bd.v…` string). Its purpose is load-time linkage — it
tells the engine what else must be present for this vault's references to resolve. For a reader, `NpeD` is
where you learn whether a vault is self-contained or expects companions.

## NrtS — string references

`NrtS` (string-reference block, size 0x10) is the smallest, an index that ties record fields to entries in
the string tables ([C11.2](02-erts-strings.md)) — the bridge that lets a `Text`/`Reference` value resolve to
a name. Where the data records hold hashes and the `ErtS` block holds names, `NrtS` is part of the machinery
that maps between them at load time.

## NtaD — the data directory

`NtaD` (data block/directory) is the most informative trailer, and it carries the numbers that describe the
record region:

```
+0x00  "NtaD"
+0x04  0x00027B20   offset/size of the data region (162 592)
+0x08  0x00000058   (88)
+0x0C  0x0000011D   count = 285
+0x10  0x0000005E   (94)
+0x14  0xEFFECADD   sentinel/magic
+0x18  0x00000004   \
+0x1C  0x00000008    } type/size descriptors (4, 8, 8, 2 …)
+0x20  0x00000008   /
+0x24  0x00000002
```

The `count = 285` and the following size descriptors (`4, 8, 8, 2`) index the data records
([C11.4](04-data-records.md)) — how many, and the widths involved. The **`0xEFFECADD`** sentinel here is the
same marker that punctuates the data region, doubling as a "this is the real data directory" signature.

> ✅ *Verified:* the three trailer tags (`NpeD`/`NrtS`/`NtaD`) exist in sequence at the header's trailer
> offset, with the sizes/counts shown and the `0xEFFECADD` sentinel in `NtaD`.
> 🟡 *Reasoned:* the precise role of each trailer field (dependency vs string-ref vs directory) is inferred
> from the tags, sizes, and contents; the tags, offsets, counts, and sentinel are verified.

## Why trailers, not a header index

Putting the indexes at the **end** is a build-pipeline convenience: the packer writes the string tables and
the records first (their sizes only known as they are emitted), then appends the trailer blocks once the final
offsets and counts are known. The fixed header ([C11.1](01-vpak-header.md)) points at the trailer region, so
the loader reads the header, jumps to the trailers to learn the record count and dependencies, and then walks
the data. It is the same "write-forward, index-backward" pattern many archive formats use.

## Editing implications

- **Keep counts truthful.** If you add or remove records, `NtaD`'s count (`285`) and its size descriptors must
  be updated, or the loader mis-reads the record region.
- **Preserve dependencies.** Don't drop an `NpeD` entry a reference relies on, or that reference dangles at
  load.
- **Don't relocate blocks without fixing the header.** The header's trailer offset must point at `NpeD`;
  moving the trailer means updating `0x20` in the header.
- **Treat `0xEFFECADD` as reserved.** It marks structure; never write it as data.

---

### Key takeaways

- Three trailer blocks index the vault: `NpeD` (dependencies), `NrtS` (string references), `NtaD` (data
  directory).
- Read the reversed tags as Dep-N / Str-N / Dat-N (an EA naming convention).
- `NtaD` carries the record **count** (285), size descriptors, and the `0xEFFECADD` sentinel.
- Trailers sit at the end (write-forward, index-backward); the fixed header points at them.
- On edits, keep `NtaD` counts and `NpeD` dependencies truthful and the header's trailer offset correct.

**Continue:** [C11.6 — Navigating & editing the vault](06-navigating-editing.md) · [Chapter 11 hub](C11-Attribute-Vaults.md)
