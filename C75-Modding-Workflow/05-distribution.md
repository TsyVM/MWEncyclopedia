# C75.5 — Distribution & the Modding Method

> **The one-sentence version:** distribute a mod as either the finished replacement files or a patch that applies the
> changes, always with a manifest and the untouched original recoverable — and understand the whole workflow (back
> up → edit → fix sizes → verify → test → ship) as the mirror image of the book's reading method.

[← C75.4 — Verify by round-trip, then test](04-verify-test.md) · [Chapter 75 hub](C75-Modding-Workflow.md) ·
[Book index →](../README.md)

---

## Two ways to distribute

Once a mod is verified ([C75.4](04-verify-test.md)), you ship it — in one of two forms:

- **Full-file replacement** — distribute the *finished* modified file (the whole `.BUN` or car `BIN`). The user backs
  up their original ([C75.1](01-backups-atomic.md)) and drops yours in. Simple and robust — the user gets exactly
  the bytes you tested — but large (a whole stream file) and version-specific (it only fits the exact original you
  edited).
- **Patch / diff** — distribute just the *changes* (a binary diff, or a script that re-applies your size-neutral
  edits, [C75.2](02-inplace-vs-repack.md)). Small and precise, but fragile — a patch assumes the user's original is
  *exactly* the one you diffed against; a different game version or a prior mod breaks it.

For **size-neutral** edits ([C75.2](02-inplace-vs-repack.md)) a patch is attractive — the changes are small and
localised, so a diff is tiny. For **repacked** files ([C75.3](03-ancestor-fixups.md)), full replacement is usually
safer — the whole container was rebuilt, so there's no clean "diff" anyway. Either way, ship a **manifest**
([C75.1](01-backups-atomic.md)) — what changed, from which original (version/checksum), and how to install — so the
user can verify they're applying it to the right base and can undo it.

## Respect the format

The deepest principle of good modding is **respect the format** — change only what you mean to, and leave everything
else byte-identical ([C75.4](04-verify-test.md)):

- **Preserve what you don't understand** — an undecoded footer, a mystery field, a runtime-only value: copy it
  *verbatim* ([C63.8](../C63-Collision-World/08-wcollisionpack.md)). Don't "clean up" bytes you can't explain; they
  may matter.
- **Minimise the blast radius** — a size-neutral edit ([C75.2](02-inplace-vs-repack.md)) touches the fewest bytes;
  prefer it, so a mod is a *scalpel*, not a *sledgehammer*.
- **Keep it verifiable** — a mod that round-trips ([C75.4](04-verify-test.md)) is one others can *check*, *rebase*,
  and *build on*. A mod that only "works on my machine" is a dead end.

A mod that respects the format is *stable* (it won't crash on edge cases the author didn't hit), *compatible* (it
changes only its lane, so it coexists with other mods), and *maintainable* (its changes are legible). This is the
difference between a mod that survives and one that breaks the next time the game does something the author didn't
test — the same *preserve-what-you-don't-understand* discipline the book's own decoders follow
([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)).

## The modding method mirrors the RE method

Step back and the whole workflow is the **inverse** of the book's reading method
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)):

```
READING (the book):   bytes → parse → model → understand          (decode what it is)
MODDING (this chapter): understand → change model → rebuild → bytes  (encode what you want)
```

Reading goes *bytes → meaning*; modding goes *meaning → bytes*. And they share the same spine: the **round-trip**
([C75.4](04-verify-test.md)) is where they meet — reading proves you understand the format by rebuilding it
unchanged, and modding uses that same rebuild to write your change. So the ability to *mod* is *exactly* the ability
to *read*, proven: a format you can round-trip is a format you can safely edit, and a format you can't isn't ready to
mod. This is why the book's "reading in RE" pages and this chapter are two halves of one skill — decode to
understand, encode to change, verified by the same round-trip.

## The whole workflow

The safe modding workflow, end to end:

```
1. BACK UP the original, plan an ATOMIC write            (C75.1)
2. decide IN-PLACE vs REPACK — prefer size-neutral       (C75.2)
3. if repacking: ANCESTOR-SIZE FIXUPS + re-alignment     (C75.3)
4. ROUND-TRIP verify (identity + edit), then TEST in-game (C75.4)
5. DISTRIBUTE (replacement or patch) with a MANIFEST     (this page)
```

Every step guards against a specific failure — losing the original, shifting offsets, desyncing the size-tree,
shipping a broken file, or an unapplyable patch. Follow them and modding is *safe and repeatable*: the format
knowledge the book decoded becomes a *practice* you can trust. That's the point of the whole book — not just to
*understand* Most Wanted's data, but to be able to *change* it, correctly, on purpose.

## RE implications

- **Two distribution forms** — full-file replacement (robust, large, version-specific) or patch/diff (small, fragile);
  ship a manifest either way.
- **Respect the format** — preserve what you don't understand, minimise the blast radius, keep it verifiable.
- **Modding mirrors reading** — bytes→meaning (read) vs meaning→bytes (mod), meeting at the round-trip; modding is
  reading, proven.
- **The whole workflow** — back up → in-place/repack → fixups → verify/test → distribute.

---

### Key takeaways

- Distribute as a **full-file replacement** (robust, large, tied to one original) or a **patch/diff** (small,
  fragile, assumes the exact base) — with a **manifest** ([C75.1](01-backups-atomic.md)) so users apply it to the
  right base and can undo it; patches suit **size-neutral** edits, replacement suits **repacks**.
- **Respect the format** — **preserve what you don't understand** ([C63.8](../C63-Collision-World/08-wcollisionpack.md)),
  **minimise the blast radius** (size-neutral scalpel, not sledgehammer), and **keep it verifiable** (round-trippable)
  — the marks of a stable, compatible, maintainable mod.
- The modding method is the **inverse of the reading method** — reading goes **bytes → meaning**, modding goes
  **meaning → bytes** — meeting at the **round-trip** ([C75.4](04-verify-test.md)): the ability to mod *is* the
  ability to read, proven.
- The **whole workflow** — back up + atomic write → in-place vs. repack → ancestor-size fixups + re-alignment →
  round-trip + in-game test → distribute with a manifest — makes modding **safe and repeatable**.
- This chapter turns the book's **format knowledge into a practice** — the point isn't only to *understand* MW's data
  but to **change it, correctly, on purpose**.

**This completes the modding-workflow chapter.** See the [book index](../README.md) for the full chapter map.

**Sources:** synthesis of verified structure — the EAGL **size-tree** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md),
esp. [C1.2](../C1-EAGL-Container-Model/02-chunk-header-and-sizes.md)/[C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md)/[C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)/[C1.11](../C1-EAGL-Container-Model/11-failure-modes.md)),
the **16-byte alignment** invariant and null-padding absorption ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md),
[C15.7](../C15-Track-Streaming/07-section-contents.md)), **round-trip verification**
([C50.2](../C50-Verification-Methodology/02-byte-verification.md)), and the per-format editing pages (e.g.
[C71.4](../C71-Cars-End-To-End/04-modding-files.md), [C63.8](../C63-Collision-World/08-wcollisionpack.md)).
