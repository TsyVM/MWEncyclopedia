# C71.4 — Modding a Car's Files

> **The one-sentence version:** modding a car means editing its `BIN`s directly — `GEOMETRY.BIN` for meshes, the
> three `TPK`s for textures, the vault for tuning — with the same size-neutral discipline the world data needs, so
> that offsets stay put and the game still loads.

[← C71.3 — The visual build](03-visual-build.md) · [Chapter 71 hub](C71-Cars-End-To-End.md) ·
[Next: C71.5 — The complete car →](05-complete-car.md)

---

## Which file holds what

In-game customization ([C71.2](02-performance-build.md)–[C71.3](03-visual-build.md)) *selects* within the shipped
data; *modding* changes the data itself. A car's data is a small, well-defined file set
([C70.5](../C70-Visual-Customisation/05-reading-visual.md)):

| To change… | Edit | Format |
|---|---|---|
| body/kit/wheel **meshes** | `GEOMETRY.BIN` | solid objects ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) |
| the base **skin** / materials | `TEXTURES.BIN` | `TPK` ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) |
| the **vinyl** artwork | `VINYLS.BIN` | `TPK` ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) |
| the **performance/handling** | the car's vault entries | vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) |

So a car mod is a *targeted* edit: a new body mesh is a `GEOMETRY.BIN` change, a new paint scheme a `TEXTURES.BIN`
change, a faster car a vault change. Knowing which file holds what — the map above — is the first half of car modding;
the second half is editing it *safely*.

## The size-neutral discipline

Car `BIN`s are chunk containers ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) with the same
hazard as the world sections ([C15.7](../C15-Track-Streaming/07-section-contents.md)): chunks are packed with
internal offsets, and **changing a chunk's size shifts everything after it**. The safe edits are therefore
*size-neutral*:

- **Same-size texture replace** — swap a `DXT` block for one of identical dimensions/format
  ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)); the `TPK` layout is unchanged, so nothing shifts. The safest,
  most common car mod (a re-skin).
- **Vault value edits** — change a tuning number in place ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md));
  the record size is fixed, so it's size-neutral by construction. The safest performance mod.
- **Geometry edits** — riskier: a different mesh is a different size, which *does* shift the `GEOMETRY.BIN` layout, so
  it requires a full rebuild that recomputes the container's sizes and offsets
  ([C10](../C10-Geometry-IO/C10-Geometry-IO.md)), not an in-place patch.

The rule mirrors the world-data rule exactly ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)): edit in
place when the size is unchanged; rebuild the whole container when it isn't. Never let a chunk grow and slide the ones
after it.

## Verify by round-trip

The discipline that makes car modding safe is the same as the book's method
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)): **round-trip verification**. Before
trusting an edit, confirm that *reading then re-writing the file unchanged reproduces it byte-for-byte* — if it does,
your reader/writer understands the format, and a real edit is trustworthy. A car mod that round-trips clean (parse →
rebuild → identical bytes) is a car mod that will load; one that doesn't has a format gap you haven't closed. This is
why the safest mods (texture replace, vault edit) are the *size-neutral* ones — they touch the least and round-trip
most easily.

## The full workflow

This page is the *car-specific* view of modding; the *general* workflow — backups, in-place vs. repack,
ancestor-size fixups, atomic writes, distribution — is [Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md).
The two compose: [Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md) gives the safety procedure, this page
gives the car-data map it applies to. A complete car mod is: back up the `BIN`, make a size-neutral edit (or a full
rebuild if not), round-trip-verify, and test in-game — the general procedure over the car file set. With that, the
car cluster's decoding becomes *actionable*: you can not only read a car but change it, safely.

## RE implications

- **The file map** — `GEOMETRY.BIN` (meshes), `TEXTURES.BIN`/`VINYLS.BIN` (`TPK`), the vault (tuning) — which file
  holds what.
- **Size-neutral edits** — same-size texture replace and in-place vault edits are safe; geometry changes need a full
  rebuild.
- **Round-trip verify** — parse → rebuild → identical bytes proves the edit will load
  ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)).
- **Full workflow** — [Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md) for backups/atomic writes/
  distribution.

---

### Key takeaways

- A car mod is a **targeted file edit** — meshes in `GEOMETRY.BIN` ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)),
  textures in the `TPK`s ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), tuning in the vault
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — so the first step is the **which-file-holds-what**
  map.
- Car `BIN`s have the **same size hazard** as world sections ([C15.7](../C15-Track-Streaming/07-section-contents.md)):
  the safe edits are **size-neutral** — same-size texture replace and in-place vault edits — while **geometry changes
  need a full rebuild** ([C10](../C10-Geometry-IO/C10-Geometry-IO.md)).
- **Round-trip verification** ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) — parse
  → rebuild → identical bytes — proves an edit will load; the size-neutral mods round-trip most easily.
- This is the **car-specific** modding view; the **general** workflow (backups, atomic writes, distribution) is
  [Chapter 75](../C75-Modding-Workflow/C75-Modding-Workflow.md) — together they make the car decoding **actionable**.

**Continue:** [C71.5 — The complete car](05-complete-car.md) · [Chapter 71 hub](C71-Cars-End-To-End.md)
