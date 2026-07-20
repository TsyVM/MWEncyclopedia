# C75.1 — Back Up First, Write Atomically

> **The one-sentence version:** never edit the only copy — back up the original before touching it, and write every
> change atomically (to a temp file, then rename) so a crash mid-write can never leave you with a corrupt game file
> and no way back.

[← Chapter 75 hub](C75-Modding-Workflow.md) · [Next: C75.2 — In-place vs. repack →](02-inplace-vs-repack.md)

---

## Never edit the only copy

The first and most important rule of modding is the cheapest: **keep the original**. Game data files are the product
of a build pipeline ([Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md)) you can't re-run; if you corrupt a
`.BUN` or a car `BIN` and have no backup, the only recovery is reinstalling. So before *any* edit:

- **Copy the original** to a `.bak` (or a versioned backup) — a known-good file you can always restore.
- **Keep it out of the edit path** — the backup must not be what your tool writes to.
- **Consider a manifest** — a small record of what you changed, from which original, so a mod is *reversible* and
  *auditable*.

This costs a file copy and saves the install. It's the safety net under everything else in this chapter: with a
recoverable original, *no* edit is fatal — the worst case is "restore the backup and try again." Modding without a
backup is editing without an undo, and the format's hazards ([C75.3](03-ancestor-fixups.md)) make an undo essential.

## Atomic writes

The second rule protects against a *different* failure — a crash *during* the write. If your tool writes changes
directly into the game file and dies halfway (a crash, a full disk, a killed process), you're left with a
**half-written, corrupt file** — worse than either the original or the intended result. The fix is the **atomic
write** pattern:

```
1. write the full new file to a temp file   (game.bun.tmp)
2. fsync / flush it to disk
3. rename the temp over the target           (game.bun.tmp → game.bun)
```

The **rename** is the trick: on essentially every filesystem, renaming a file over another is *atomic* — it either
fully happens or doesn't, never half. So the target is *always* either the complete old file or the complete new
one, never a torn mixture. A crash before the rename leaves the original intact (and a stray `.tmp` to delete); a
crash after leaves the finished new file. There is no corrupt-in-between state.

This matters especially for the big files ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — a
500 MB stream file written in place is a long window to crash in; written atomically, the window is just the
instant rename. Atomic writes turn "a crash could corrupt my game" into "a crash costs me a temp file."

## The two together

Backups and atomic writes cover the two ways an edit can go wrong:

- **A bad edit** (wrong bytes, broken format) → the **backup** restores the original.
- **A failed write** (crash mid-save) → the **atomic rename** guarantees the target is never torn.

Together they make modding *safe to attempt*: you can try an edit knowing that a mistake is recoverable and a crash
is survivable. This is the foundation the rest of the workflow builds on — the in-place/repack decision
([C75.2](02-inplace-vs-repack.md)), the size-tree fixups ([C75.3](03-ancestor-fixups.md)), and the verification
([C75.4](04-verify-test.md)) all assume you *can* fail safely, because a backup and an atomic write mean failure is
never permanent. Get these two habits first, and everything else is just precision.

## RE implications

- **Back up the original** — game data is build-pipeline output you can't re-run; a `.bak` is the only cheap undo.
- **Write atomically** — temp file + rename; the target is always the complete old or new file, never torn.
- **Big files especially** — atomic writes shrink the crash window from the whole write to the instant rename.
- **Together** — a backup covers bad edits, an atomic write covers failed saves; failure becomes recoverable.

---

### Key takeaways

- **Never edit the only copy** — back up the original to a `.bak` (game data is **build-pipeline output**,
  [Chapter 58](../C58-Build-Pipeline/C58-Build-Pipeline.md), you can't regenerate) so **every mistake is
  recoverable**.
- **Write atomically** — write the new file to a **temp** file, flush, then **rename** it over the target; the rename
  is atomic, so the game file is **never left half-written** by a crash.
- This matters most for the **large stream files** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) —
  atomic writes shrink the crash-corruption window to the **instant of the rename**.
- Backups cover **bad edits** (restore), atomic writes cover **failed writes** (all-or-nothing) — together they make
  modding **safe to attempt**, the foundation the rest of the workflow assumes.
- Keeping a **manifest** of what changed (from which original) makes a mod **reversible and auditable**.

**Continue:** [C75.2 — In-place vs. repack](02-inplace-vs-repack.md) · [Chapter 75 hub](C75-Modding-Workflow.md)
