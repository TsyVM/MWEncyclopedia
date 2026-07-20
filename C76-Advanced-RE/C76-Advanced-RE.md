# Chapter 76 — Advanced Reverse Engineering

> **Goal of this chapter:** decode the *method* of hard RE — identifying unknown data (block markers, string tables,
> record structure), recovering a schema (the attribute vault's class-name table and code-driven registration),
> choosing static vs. dynamic recovery, and building readers you can *validate* — using the vault schema recon as a
> worked, honestly-tiered case study.

The book's format chapters present *finished* decodings; this chapter shows *how the hard ones are done* — and,
crucially, how to stay honest when they're only *partly* done. It's the advanced companion to the verification
methodology ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)): where that established
the *confidence tiers*, this shows the *techniques* that produce evidence for them — and uses the attribute vault
([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)), a genuinely *XL* target, as the running example,
dead-ends and all.

> **Verified against the real binaries.** The vault case study rests on independently-confirmed facts:
> `GLOBAL/attributes.bin` (689,728 B) opens with the block map **`ErtS`** (string table @ `0x80`, 1,961 names),
> **`NpeD`** (`0x55C00`), **`NrtS`** (`0x55C30`), **`NtaD`** (`0x55C40`, **4,732** typed records marked
> `0xEFFECADD`). `speed.exe` holds a **28-name reflection class table** at file offset **`0x4ADD1C`** (inline
> NUL-padded strings: `pvehicle`, `chopperspecs`, `RBVehicle`, `RBCop`, `EngineRacer`, …), and the vault's record
> keys resolve as **`lookup2` hashes (seed `0xABCDEF00`)** — the reflection hash
> ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)), whose seed appears ×50 in the executable.

---

## Deep-dive pages

- [C76.1 — Identifying unknown data](01-identifying-data.md): block markers, string tables, and record structure —
  reading a format you've never seen.
- [C76.2 — Recovering a schema](02-recovering-schema.md): the vault's class-name table and code-driven registration.
- [C76.3 — Static vs. dynamic recovery](03-static-vs-dynamic.md): disassembly vs. diff-a-known-change, and when to
  use each.
- [C76.4 — Building & validating readers](04-building-readers.md): from hypothesis to a reader you can trust
  (round-trip + statistics).
- [C76.5 — The advanced-RE method](05-advanced-method.md): the whole loop, honest tiering, and learning from
  dead-ends.

---

## 76.1 Identifying unknown data

The first task on an unknown format is *classification* ([C76.1](01-identifying-data.md)): is this region a string
table, a record array, code, or floats? The tells are structural — **fourcc block markers** (`ErtS`/`NtaD`), a
repeated **record marker** (`0xEFFECADD` every record), a fixed or variable **stride**, and string tables you can
read directly. The `attributes.bin` block map is the model: four tagged blocks, each a different role, legible before
any field is decoded.

## 76.2 Recovering a schema

A *schema* — the field→offset→type map — is the hard part ([C76.2](02-recovering-schema.md)). The vault's is
**code-driven**: `speed.exe` holds a 28-name class table (`0x4ADD1C`), and each class *self-registers* at startup via
a static-init thunk that pushes itself onto a global list (interned through `0x5CC240`). So the schema isn't a static
table to parse — it's *built by registration code*, which makes static recovery genuinely hard and is why the vault
took a hash breakthrough ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) to crack.

## 76.3 Static vs. dynamic recovery

When static analysis is XL, there's another path ([C76.3](03-static-vs-dynamic.md)): **dynamic diff**. Change *one
known value* in-game (raise a car's top speed via an upgrade), dump the data before and after, and diff — the changed
bytes *localise the field*. Static (disassemble the registration) and dynamic (diff a known change) are complementary:
static gives structure, dynamic gives ground-truth field locations without a disassembler.

## 76.4 Building & validating readers

A decoding is only as good as its *validation* ([C76.4](04-building-readers.md)): a reader is trustworthy when it
**round-trips** ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) and its results hold
*statistically* — e.g. the vault's keys resolving **66.8%** under `lookup2` (vs. <0.2% noise under the wrong hash) is
what turned a guess into a fact. Recording *dead-ends* and *false negatives* is part of the method, not a footnote.

---

### Key takeaways

- Advanced RE is the **method** behind the book's finished decodings — the **techniques** that produce evidence for
  the confidence tiers ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)).
- **Identify unknown data** by structure — **block markers** (`ErtS`/`NtaD`), **record markers** (`0xEFFECADD`),
  **stride**, and **string tables** — classify before decoding ([C76.1](01-identifying-data.md)).
- **Schemas can be code-driven** — the vault's field map is **built by registration code** (self-registering classes,
  `0x4ADD1C` table), not a static table, which is why it's *XL* to recover statically
  ([C76.2](02-recovering-schema.md)).
- **Static and dynamic recovery are complementary** — disassemble the registration *or* **diff a known in-game
  change** to localise fields ([C76.3](03-static-vs-dynamic.md)).
- **Validate by round-trip and statistics** — the vault keys resolving **66.8%** under `lookup2`/`0xABCDEF00` (vs.
  <0.2% under the wrong hash) is the difference between a guess and a fact ([C76.4](04-building-readers.md)); record
  the **dead-ends** too.

**Next:** [C76.1 — Identifying unknown data](01-identifying-data.md).
