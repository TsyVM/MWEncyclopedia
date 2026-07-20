# Chapter 75 — Modding Workflow & Safety

> **Goal of this chapter:** turn the book's format knowledge into a *safe editing practice* — always back up, prefer
> size-neutral in-place edits, repack (with ancestor-size fixups) when you can't, verify every change by round-trip,
> write atomically, and distribute cleanly — so a mod changes what you intend and nothing else.

Every format chapter ends with an "editing" note; this chapter gathers those into one **discipline**. It's not new
bytes — it's the *method* for changing bytes without breaking the game: the container's size-tree
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)), the 16-byte alignment invariant
([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)), and round-trip verification
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)), assembled into a workflow. It's the
practical companion to every "reading in RE" page — where those decode *what the data is*, this decodes *how to
change it safely*.

> **Built on verified structure.** The workflow rests on facts established across the book: EAGL files are **nested
> chunk containers** with **size fields at every level** ([C1.2](../C1-EAGL-Container-Model/02-chunk-header-and-sizes.md)),
> record data is **16-byte aligned** (`(ptr + 0x17) & ~0xF`, [C63.6](../C63-Collision-World/06-ondisk-collision-data.md)),
> sections are **2048-aligned** ([C15.7](../C15-Track-Streaming/07-section-contents.md)), and a correct reader/writer
> **round-trips byte-for-byte** ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)). The workflow is
> what those facts *imply* for safe editing.

---

## Deep-dive pages

- [C75.1 — Back up first, write atomically](01-backups-atomic.md): never lose the original; never leave a
  half-written file.
- [C75.2 — In-place vs. repack](02-inplace-vs-repack.md): the size-neutral edit and when you must rebuild the
  container.
- [C75.3 — Ancestor-size fixups](03-ancestor-fixups.md): the size-tree — when a chunk changes size, every parent's
  size must follow.
- [C75.4 — Verify by round-trip, then test](04-verify-test.md): parse → rebuild → identical bytes, then in-game.
- [C75.5 — Distribution & the modding method](05-distribution.md): packaging mods, and the whole safe workflow.

---

## 75.1 Back up first, write atomically

The first rule ([C75.1](01-backups-atomic.md)): **never edit the only copy**. Back up the original before touching
it, and write changes **atomically** (to a temp file, then rename) so a crash mid-write never leaves a corrupt game
file. A recoverable original and an all-or-nothing write are the safety net under everything else.

## 75.2 In-place vs. repack

The central choice ([C75.2](02-inplace-vs-repack.md)): a **size-neutral in-place** edit (same-size texture, a vault
value, an emptied-but-kept record) touches only the bytes it changes and is *safe by construction*
([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)); a **size-changing** edit (a bigger mesh, an added
record) shifts everything after it and demands a **repack** — a full rebuild of the container with corrected sizes
and alignment ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)).

## 75.3 Ancestor-size fixups

When a chunk *does* change size ([C75.3](03-ancestor-fixups.md)), the container's **size-tree**
([C1.2](../C1-EAGL-Container-Model/02-chunk-header-and-sizes.md)) must be repaired: every **ancestor** chunk's size
field includes the changed chunk, so each parent up to the root must be adjusted by the same delta. Miss one and the
walker ([C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md)) desynchronises — the classic corruption.

## 75.4 Verify by round-trip, then test

Before trusting a mod ([C75.4](04-verify-test.md)): **round-trip** it — parse the edited file and rebuild it; if the
bytes match your intended output and an *unchanged* file round-trips identically
([C50.2](../C50-Verification-Methodology/02-byte-verification.md)), your tooling understands the format. Then **test
in-game** — the final proof that the edit loads and behaves.

---

### Key takeaways

- The workflow turns the book's **format facts** — the container **size-tree**
  ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)), the **16-byte alignment** invariant
  ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)), and **round-trip verification**
  ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) — into a **safe editing practice**.
- **Back up first, write atomically** ([C75.1](01-backups-atomic.md)) — a recoverable original and an all-or-nothing
  write underpin everything.
- **Prefer size-neutral in-place edits** ([C75.2](02-inplace-vs-repack.md)) — safe by construction; **repack** only
  when the size changes, with **ancestor-size fixups** ([C75.3](03-ancestor-fixups.md)) up the size-tree.
- **Verify by round-trip, then test in-game** ([C75.4](04-verify-test.md)) — parse → rebuild → identical bytes proves
  the tooling; the game proves the mod.
- This is the **practical companion** to every "reading in RE" page — how to *change* the data those pages decode,
  without breaking the game.

**Next:** [C75.1 — Back up first, write atomically](01-backups-atomic.md).
