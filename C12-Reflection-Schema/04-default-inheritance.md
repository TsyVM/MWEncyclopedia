# C12.4 — `default` Inheritance

> **The one-sentence version:** nearly every collection names `default` (`0xEEC2271A`) as its parent and
> stores only the fields it changes, so the baseline for the whole vault lives in one place — which is why
> `default` is referenced 1 071 times and why editing it changes everything that doesn't override.

[← C12.3 — The inline value triple](03-value-triple.md) · [Chapter 12 hub](C12-Reflection-Schema.md) ·
[Next: C12.5 — Resolving a value →](05-resolving-values.md)

---

## One baseline, many overrides

The vault is built on inheritance. There is a base collection, **`default`**, that defines a value for every
field. Every other collection carries a **parent reference** — almost always `default` — and stores **only
the triples that differ** from its parent ([C12.3](03-value-triple.md)). A collection that is standard in all
but one respect stores exactly one triple; the rest resolve from `default`.

The evidence is emphatic: `reflection_hash("default") = 0xEEC2271A`
([C12.1](01-reflection-hash.md)), and that value appears **1 071 times** in `attributes.bin` — once as each
inheriting collection's parent reference. Inheritance is not an occasional feature; it is the pervasive
structure of the file.

## Why design it this way

Sparse, inheritance-based records buy three things at once:

- **Compactness.** Collections store differences, not full attribute sets — hundreds of cars/surfaces/effects
  fit in a small file because most fields are inherited.
- **Global tunability.** Change a field in `default` and every collection that doesn't override it moves in
  lockstep — a one-edit balance change ([C12.6](06-writing-values.md)).
- **Clear intent.** A collection's stored triples *are* its design: they say exactly how this car/surface
  differs from baseline, with no noise from inherited values.

It is the same idea as CSS cascade or object-oriented defaults: define once, override sparingly.

## The parent chain

The parent reference is a `Reference`-typed value ([C12.3](03-value-triple.md)) holding the parent
collection's hash. Usually that is `default`, but a collection can name a *different* parent, forming a chain:

```
some_specific_car → car → default
```

Resolution walks this chain: check the collection, then its parent, then the grandparent, up to `default`,
stopping at the first one that defines the field ([C12.5](05-resolving-values.md)). Most chains are one hop
(→ `default`), but the model supports deeper hierarchies where a family of collections shares an intermediate
base.

> ✅ *Verified:* `default` (`0xEEC2271A`) is the universal parent, referenced 1 071 times; collections carry a
> parent reference and store sparse overrides.
> 🟡 *Reasoned:* multi-level parent chains beyond `→ default` are inferred from the `Reference` mechanism and
> the presence of intermediate collections (`car`, etc.); single-level inheritance to `default` is the
> verified common case.

## Reading is resolving, not looking up

The consequence for anyone reading the vault is fundamental: **a collection's record does not contain its
values** — it contains its *overrides*. To report the value the game actually uses you must **resolve** —
follow the parent chain until the field is found ([C12.5](05-resolving-values.md)). A tool that reads only the
collection's own triples will report "field absent" for every inherited attribute, which is almost all of
them. Resolution is mandatory, not optional.

## Editing through inheritance

Inheritance turns "what do I edit?" into a scope decision:

- **Change one collection** → add or modify the override triple in that record. Only it changes.
- **Change the baseline** → edit the field in `default`. Every non-overriding collection follows.
- **Introduce a family default** → point several collections at a shared intermediate parent and set the
  value there.

Choosing the right level is the essence of vault editing ([C12.6](06-writing-values.md)): a surgical
per-collection override versus a sweeping `default` change are the same mechanic applied at different points
in the chain.

---

### Key takeaways

- Collections name a **parent** (almost always `default`, `0xEEC2271A`, 1 071×) and store only overrides.
- Inheritance gives compactness, global tunability, and records that state design intent.
- The parent is a `Reference` value; resolution walks the chain (usually one hop to `default`) to the first
  definer.
- A record holds overrides, **not** values — reading requires **resolution**, or inherited fields look
  absent.
- Edit scope by level: override one record, edit `default` for all, or use an intermediate parent for a
  family.

**Continue:** [C12.5 — Resolving a value](05-resolving-values.md) · [Chapter 12 hub](C12-Reflection-Schema.md)
