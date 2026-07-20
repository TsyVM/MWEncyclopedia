# C75.2 — In-Place vs. Repack

> **The one-sentence version:** the central modding choice is whether your edit changes a chunk's *size* — a
> size-neutral edit (same-size texture, a vault value, an emptied-but-kept record) is safe in place, while a
> size-changing edit shifts everything after it and demands a full repack of the container.

[← C75.1 — Back up first, write atomically](01-backups-atomic.md) · [Chapter 75 hub](C75-Modding-Workflow.md) ·
[Next: C75.3 — Ancestor-size fixups →](03-ancestor-fixups.md)

---

## The one question that decides everything

Before making an edit, ask: **does this change the size of any chunk?** The answer determines the entire approach,
because EAGL files are packed with internal sizes and offsets ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md))
— so *size* is the hazard, not *content*.

- **No size change → edit in place.** Overwrite the bytes; nothing downstream moves; the file's structure is
  untouched. Safe by construction.
- **Size change → repack.** Everything after the edit shifts, so the container's sizes, offsets, and alignment must
  all be recomputed — a full rebuild ([C75.3](03-ancestor-fixups.md)).

This single question is the spine of safe modding. Size-neutral edits are *easy and safe*; size-changing edits are
*possible but demanding*. The skilled modder's instinct is to **find the size-neutral way** to achieve a change
whenever one exists.

## The safe zone: size-neutral edits

A surprising amount can be changed *without* changing any size — these are the safe, in-place edits:

- **Same-size texture replace** — swap a `DXT` block for one of identical dimensions and format
  ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)). A re-skin ([C71.4](../C71-Cars-End-To-End/04-modding-files.md))
  that keeps the texture's byte size touches nothing else. The most common car and world mod.
- **Vault value edits** — change a tuning number in place ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md));
  the record is fixed-size, so a different value is size-neutral by construction. The safest performance mod.
- **Transform edits** — move or rotate a scenery instance ([C16.3](../C16-Scenery-Cull/03-instance-record.md)) or a
  smackable ([C63.9](../C63-Collision-World/09-smackables-emitters.md)): you rewrite the position/rotation fields *in
  the same-size record*. Relocating a prop is size-neutral.
- **Emptied-but-kept records** — delete an item's *content* while keeping its slot, growing tail padding to absorb
  the freed bytes ([C63.8](../C63-Collision-World/08-wcollisionpack.md)): a *size-neutral deletion*. Retail data
  already contains zero-size entries, so the loader tolerates them.

The pattern across all of these: **change the values, not the layout**. If you can express your edit as "different
bytes in the same-size slot," it's an in-place edit — no size-tree fixup, no alignment risk, no repack. This is why
the book's editing pages keep returning to size-neutrality ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md))
— it's the safe path, and it's wider than it first appears.

## When you must repack

Some edits *cannot* be size-neutral — they genuinely add or remove bytes:

- **A different-size mesh** — a new body kit or prop geometry ([Chapter 10](../C10-Geometry-IO/C10-Geometry-IO.md))
  is almost never the same byte size as the original.
- **Added records** — a *new* scenery instance, a *new* smackable, an *extra* texture — grows its chunk.
- **Different-size textures** — a higher-resolution skin changes the `DXT` block size.

These require a **repack**: rebuild the whole container, writing each chunk at its new position, recomputing every
size field ([C75.3](03-ancestor-fixups.md)) and re-establishing alignment
([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)). A repack is *not* a byte-patch — it's a
parse-everything, change-what-you-want, write-everything-fresh operation. It's more work and more risk (every size
and offset must be right), which is exactly why size-neutral edits are preferred when they'll do.

## Choosing the approach

The decision in practice:

```
edit changes a size?
├─ no  → IN-PLACE: overwrite the bytes, done (verify by round-trip, C75.4)
└─ yes → can it be made size-neutral?
         ├─ yes → do that instead (e.g. same-size texture, emptied-but-kept record)
         └─ no  → REPACK: full container rebuild with ancestor-size fixups (C75.3)
                  + re-alignment, then verify (C75.4)
```

The middle branch is the craft: *often* a size-changing edit can be *reformulated* as size-neutral — resize the new
texture to match, empty a record instead of removing it, reuse an existing slot. Reach for that before repacking.
When you genuinely can't, repack carefully ([C75.3](03-ancestor-fixups.md)) and verify hard
([C75.4](04-verify-test.md)). The whole art of safe modding is *staying in the size-neutral zone as long as
possible*, and repacking correctly when you must leave it.

## RE implications

- **Size is the hazard** — the one question is "does this change a chunk's size?"; content edits are safe, size
  edits aren't.
- **Size-neutral in-place edits** — same-size texture, vault value, transform, emptied-but-kept record — safe by
  construction.
- **Repack when size changes** — different mesh/texture, added record — a full container rebuild
  ([C75.3](03-ancestor-fixups.md)).
- **Prefer reformulation** — often a size-changing edit can be made size-neutral; do that before repacking.

---

### Key takeaways

- The **one question** that decides the approach: **does the edit change a chunk's size?** — because EAGL files are
  packed with internal **sizes and offsets** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)),
  *size* is the hazard, not content.
- **Size-neutral in-place edits are safe by construction** — same-size **texture replace**
  ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), **vault value** edits
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), **transform** edits
  ([C16.3](../C16-Scenery-Cull/03-instance-record.md)), **emptied-but-kept** records
  ([C63.8](../C63-Collision-World/08-wcollisionpack.md)) — change the **values, not the layout**.
- **Size-changing edits demand a repack** — a different-size mesh/texture or an added record shifts everything after
  it, so the whole container must be **rebuilt** with corrected sizes and alignment.
- The **craft** is **reformulating** a size-changing edit as size-neutral where possible (resize to match, empty a
  slot, reuse a record) — stay in the safe zone as long as you can.
- Prefer **in-place**; **repack** only when unavoidable, and then **verify hard** ([C75.4](04-verify-test.md)).

**Continue:** [C75.3 — Ancestor-size fixups](03-ancestor-fixups.md) · [Chapter 75 hub](C75-Modding-Workflow.md)
