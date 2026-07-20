# C76.3 — Static vs. Dynamic Recovery

> **The one-sentence version:** when static disassembly is XL, there's a second path — *dynamic diff*: change one
> known value in-game, dump the data before and after, and diff, so the changed bytes *localise the field* — and the
> two approaches are complementary, static giving structure and dynamic giving ground-truth field locations without
> a disassembler.

[← C76.2 — Recovering a schema](02-recovering-schema.md) · [Chapter 76 hub](C76-Advanced-RE.md) ·
[Next: C76.4 — Building & validating readers →](04-building-readers.md)

---

## Two ways to find a field

Once you know a format holds records ([C76.1](01-identifying-data.md)) but not *what each field is or where*, there
are two fundamentally different ways to find out:

- **Static** — read the *code* that reads the data. Disassemble the loader, the registration
  ([C76.2](02-recovering-schema.md)), the constructors; recover the field map from how the program *lays out and
  accesses* the bytes. Needs a disassembler and patience; gives *structure* and *types*.
- **Dynamic** — watch the *data change*. Make one known change in the running game, capture the bytes before and
  after, and see *what moved*. Needs a live game session, not a disassembler; gives *ground-truth field locations*.

Neither is strictly better — they answer different questions and fail in different ways. The skill is knowing which
to reach for, and using them *together* ([below](#complementary-not-competing)).

## The dynamic-diff technique

Dynamic diff is the workhorse when static is XL ([C76.2](02-recovering-schema.md)), and it's beautifully simple:

```
1. dump the data file            (attributes.bin, "before")
2. make ONE known change in-game  (e.g. raise a car's top speed via an upgrade part)
3. dump the data file again       ("after")
4. diff before vs. after          → the changed bytes ARE that field
```

The power is in the *isolation*: because you changed *exactly one known thing*, the bytes that differ *must* be that
field. No disassembly, no hypothesis about layout — the game itself tells you where the field lives by *writing it*.
Change a car's top-speed rating and the differing bytes localise the top-speed field in the vault
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); repeat for each field, one known change at a time, and
you build the field map *empirically*. Each find carries its own provenance — "this byte range changed when I did
X" — which is *verification built into the discovery* ([C76.4](04-building-readers.md)).

> 🟡 *Reasoned:* the dynamic-diff method (dump → one known change → dump → diff localises the field) is a standard RE
> technique and the practical path for the code-driven vault schema ([C76.2](02-recovering-schema.md)); the specific
> fields it recovers are per-change results. The vault's structure and hash ([C76.2](02-recovering-schema.md)) are
> verified.

## The cost of each

The trade-offs are sharp, and dictate when to use which:

| | Static | Dynamic |
|---|---|---|
| **Needs** | a disassembler + skill | a live game + a way to make known changes |
| **Gives** | structure, types, *completeness* (all fields, from the code) | ground-truth *locations* (only the fields you change) |
| **Fails when** | the schema is code-driven/XL ([C76.2](02-recovering-schema.md)) | you can't make a *clean, isolated* change |
| **Provenance** | "the code reads it this way" | "these bytes changed when I did X" |

Static gives you *everything at once* if you can read the code — but a code-driven schema
([C76.2](02-recovering-schema.md)) makes that a huge effort. Dynamic gives you *exactly what you probe* — reliable
but incremental, one field per experiment, and only for values you can *change* in-game. So static suits
*structure-heavy, code-readable* formats; dynamic suits *hard-to-disassemble* ones where you can drive known changes.

## Complementary, not competing

The two are best *combined* ([C76.5](05-advanced-method.md)):

- **Static for the skeleton** — the block map ([C76.1](01-identifying-data.md)), the record marker, the stride, the
  key hash ([C76.2](02-recovering-schema.md)). Structure first.
- **Dynamic for the flesh** — the actual field locations, probed one known change at a time, hung on the static
  skeleton.

The vault is the model: *static* gave the shape (blocks, records, keys resolving under `lookup2`,
[C76.2](02-recovering-schema.md)), and *dynamic diff* is the practical way to fill the field map without fully tracing
the XL registration code. Neither alone was enough — static couldn't cheaply reach the field offsets, dynamic couldn't
give the whole structure — but *together* they crack it: structure from static, locations from dynamic, each
verifying the other where they overlap.

## A static dead-end (why you cross-check)

A cautionary tale for *pure* static work ([C76.4](04-building-readers.md)): searching the executable for field-like
strings turned up `TOPSPEED`, `ACCELERATION`, `HANDLING` with a clean registration pattern — a fixed 68-byte stride,
each field a wrapper object with a min/max range of `1.0..160.0`. It *looked* like the vault's field registrar. It
wasn't: the identical `1.0..160.0` range across all three (regardless of physical meaning) is the tell of a **generic
UI slider/star-rating widget** — the garage's top-speed/acceleration/handling *bars*
([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)), **not** the reflection field
map that serialises to the vault. A plausible static lead that answered a *different question*. The lesson: a static
finding needs *cross-checking* (here, dynamic diff would have shown these don't map to vault records) — pattern-match
is a hypothesis, not a conclusion ([C76.4](04-building-readers.md)).

## RE implications

- **Two paths** — static (read the code) gives structure/types; dynamic (diff a known change) gives ground-truth
  locations.
- **Dynamic diff** — dump → one known change → dump → diff localises the field; verification built into discovery.
- **Complementary** — static for the skeleton (blocks, keys), dynamic for the flesh (field offsets); combine them.
- **Cross-check static leads** — the `TOPSPEED`/`ACCEL`/`HANDLING` "registrar" was a UI widget
  ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)), not the vault schema.

---

### Key takeaways

- There are **two ways to recover a field**: **static** (disassemble the code that reads it — structure & types, but
  XL when the schema is code-driven, [C76.2](02-recovering-schema.md)) and **dynamic** (diff the data around a known
  change — ground-truth locations, no disassembler).
- **Dynamic diff** is the workhorse: **dump → make one known in-game change → dump → diff** — the changed bytes **are**
  that field, with **provenance built into the discovery**.
- The two are **complementary** — **static for the skeleton** (block map, record marker, key hash) and **dynamic for
  the flesh** (field offsets, one probe at a time) — the vault needs both.
- **Cross-check static leads** — the `TOPSPEED`/`ACCELERATION`/`HANDLING` "registrar" (68-byte stride, `1.0..160.0`
  range) *looked* like the vault schema but was the garage **UI bars**
  ([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)) — a pattern-match is a
  **hypothesis, not a conclusion**.

**Continue:** [C76.4 — Building & validating readers](04-building-readers.md) · [Chapter 76 hub](C76-Advanced-RE.md)
