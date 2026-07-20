# C75.4 — Verify by Round-Trip, then Test

> **The one-sentence version:** before trusting a mod, prove it two ways — round-trip it (an *unchanged* file must
> rebuild byte-for-byte, and your edited file must rebuild to exactly your intended bytes) to prove the *format* is
> right, then test in-game to prove the *behaviour* is right.

[← C75.3 — Ancestor-size fixups](03-ancestor-fixups.md) · [Chapter 75 hub](C75-Modding-Workflow.md) ·
[Next: C75.5 — Distribution & the modding method →](05-distribution.md)

---

## Two kinds of correct

A mod can be wrong in two independent ways, and each needs its own check:

- **Format-wrong** — the file no longer parses correctly: a bad size ([C75.3](03-ancestor-fixups.md)), a misalignment
  ([C63.6](../C63-Collision-World/06-ondisk-collision-data.md)), a broken chunk. The game may crash on load or read
  garbage.
- **Behaviour-wrong** — the file parses fine but does the wrong thing: the car's too fast, the texture's the wrong
  colour, the prop's in the floor. The format is valid; the *content* is off.

These are *orthogonal* — a mod can be format-perfect but behaviour-wrong, or (rarely) behave right while being
subtly format-broken. So verification is **two stages**: prove the format by **round-trip**, then prove the behaviour
by **testing in-game**. Skip either and you're shipping on faith.

## The round-trip proof

The format check is the book's own method ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)) turned on
your edit: **parse the file, rebuild it, and compare**. It comes in two forms, both essential:

- **The identity round-trip** — take an *unchanged* original, parse it, and rebuild it. The output must be
  **byte-for-byte identical** to the input. If it isn't, your reader/writer *doesn't fully understand the format* —
  and you must not trust it to make edits, because it'll corrupt something it doesn't model. This is the *precondition*
  for any editing tool ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).
- **The edit round-trip** — parse your *edited* file and rebuild it; the result must match *exactly the bytes you
  intended*. Every size fixed ([C75.3](03-ancestor-fixups.md)), every alignment preserved, only the intended bytes
  changed. This proves the *edit* is structurally clean.

```
IDENTITY:  original ──parse──▶ model ──rebuild──▶ bytes ;  assert bytes == original
EDIT:      edited   ──parse──▶ model ──rebuild──▶ bytes ;  assert bytes == intended
```

The identity round-trip is the one modders skip and shouldn't: it's the proof that your *tool* is trustworthy *before*
you rely on it. A tool that can't reproduce an unchanged file has a format gap, and that gap will silently damage
your mod. Round-tripping is *cheap* (parse + rebuild + compare) and *decisive* — run it every time
([C71.4](../C71-Cars-End-To-End/04-modding-files.md)).

## Then test in-game

Round-trip proves the file is *structurally* correct; it says nothing about whether the change *does what you wanted*.
For that, there's only one test: **load it in the game**. In-game testing catches:

- **Format errors the round-trip missed** — if the game crashes on load or shows corruption, some structure is wrong
  in a way your model didn't capture (a reason to *improve the round-trip*, [C50.2](../C50-Verification-Methodology/02-byte-verification.md)).
- **Behaviour errors** — the car handles wrong, the skin looks off, the prop floats. The file's valid; the *values*
  need tuning.

So testing is both the *final* format check (the game is the ultimate parser) and the *only* behaviour check. A mod
that round-trips clean *and* loads and behaves correctly in-game is *verified* in both senses — format and behaviour.
That's the bar to clear before distribution ([C75.5](05-distribution.md)): not "it probably works," but "it round-trips
and it runs."

## The verification loop

In practice the two checks form a loop with the edit:

```
edit (C75.2/3) → round-trip (format) ──fail──▶ fix the structure, repeat
                        │pass
                        ▼
               test in-game (behaviour) ──fail──▶ adjust the values, repeat
                        │pass
                        ▼
                   verified — ready to distribute (C75.5)
```

A format failure sends you back to the *structure* (sizes, alignment, [C75.3](03-ancestor-fixups.md)); a behaviour
failure sends you back to the *values* (the actual edit). Only when *both* pass is the mod done. This loop is the
same verification-first discipline the whole book runs on ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)),
applied to *making* changes rather than *reading* them: never trust a change you haven't proven, and prove it two
ways — the tool (round-trip) and the game (test).

## RE implications

- **Two kinds of correct** — format (parses right) and behaviour (does the right thing) — orthogonal, each needs its
  own check.
- **Round-trip** — the *identity* round-trip proves the tool; the *edit* round-trip proves the edit; both compare
  bytes ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).
- **Test in-game** — the ultimate parser (final format check) and the only behaviour check.
- **The loop** — format fail → fix structure; behaviour fail → fix values; both pass → verified.

---

### Key takeaways

- A mod can be **format-wrong** (won't parse — bad size/alignment) or **behaviour-wrong** (parses, does the wrong
  thing) — **orthogonal** failures, so verification is **two stages**.
- **Round-trip proves the format** ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)): the **identity**
  round-trip (unchanged file rebuilds **byte-for-byte**) proves your **tool** is trustworthy; the **edit** round-trip
  (edited file rebuilds to **exactly your intended bytes**) proves the **edit** is clean.
- **Don't skip the identity round-trip** — a tool that can't reproduce an unchanged file has a **format gap** that will
  silently corrupt your mod; it's cheap and decisive, so run it every time.
- **Then test in-game** — the game is the **ultimate parser** (final format check) and the **only** behaviour check;
  the bar is "round-trips *and* runs," not "probably works."
- The **edit → round-trip → test** loop is the book's **verification-first discipline**
  ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) applied to *making* changes — prove
  every change two ways before you trust it.

**Continue:** [C75.5 — Distribution & the modding method](05-distribution.md) · [Chapter 75 hub](C75-Modding-Workflow.md)
