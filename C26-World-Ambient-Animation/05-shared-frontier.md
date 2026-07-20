# C26.5 — The Shared Frontier

> **The one-sentence version:** because ambient and gameplay animation use the same format as cutscenes, they
> inherit the same open problem — the keyframe quantisation is undecoded — so their rigs are recoverable but
> their motion is not, and the banks are preserve-raw.

[← C26.4 — Ambient vs cutscene playback](04-ambient-vs-cutscene.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md) ·
[Next: C26.6 — Working with ambient banks →](06-working-with-banks.md)

---

## The frontier travels with the format

The keyframe-quantisation problem ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)) is a property of the
**format**, not of cutscenes specifically. So every user of the animation-bank format inherits it: the crane's
swing, the ship's drift, the blimp's traverse, and gameplay animations all store their motion as the same
undecoded quantised keyframes. The decoded/undecoded split is identical to Chapter 24:

- ✅ **Skeletons** — recoverable from the ELF symbol table ([C24.3](../C24-NIS-Animation/03-skeleton.md)) for
  every animated object.
- ⏳ **Keyframes** — quantised in the unresolved scheme ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)),
  for every animated object.

So nothing about ambient or gameplay animation is *more* decoded than cutscene animation — it's the same
format at the same frontier.

## What this means in practice

For anyone working with animated world/gameplay objects:

- **You can identify and rebuild the rigs.** The crane, ship, and blimp skeletons come out of their symbol
  tables — you know their bone structure and bind pose.
- **You cannot yet reproduce their motion** from the banks alone — the swing, drift, and traverse are opaque
  quantised keyframes.
- **On PC, the character-animation caveat applies** ([C24.6](../C24-NIS-Animation/06-ps2-vs-pc.md)) to the NIS
  path; world-ambient motion that the PC build *does* play is observable in-game even if the bank data isn't
  decoded.

So the practical position is the same as cutscenes: rigs open, motion closed.

## One decode would unlock everything

The upside of the shared frontier is that it's a **single** problem: crack the keyframe quantisation once, and
it unlocks motion for *all* users — cutscenes, ambience, and gameplay — because they share the format. This is
the leverage of format reuse working in the reverse-engineer's favour: the effort isn't per-user, it's
per-format, and the format is one.

> ⏳ *Open (shared):* the keyframe quantisation is undecoded across all animation-bank users. The skeletons and
> the ELF/bank structure are ✅ verified ([Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md)).

## Preserve-raw, everywhere

The editing rule follows the frontier: treat animation banks — cutscene, ambient, or gameplay — as
**preserve-raw** ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)). Read and round-trip them
byte-exact; don't attempt to interpret or edit the quantised keyframes, because you'd be editing data whose
meaning you don't have. The tractable edits are the **rig** (skeleton) and the **driver**
([C26.4](04-ambient-vs-cutscene.md)) — loop ranges, cutscene timing, gameplay cues — not the motion itself
([C26.6](06-working-with-banks.md)).

## Where contribution helps most

Because the frontier is shared and singular, it's the highest-leverage place to contribute: a decode of the
keyframe quantisation ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)) would, in one stroke, make *all*
animation editable — every cutscene, every ambient mover, every gameplay animation. The PS2 build (which plays
the motion, [C24.6](../C24-NIS-Animation/06-ps2-vs-pc.md)) is the reference to validate against. Until then,
the honest position holds across the whole animation system: rigs yes, motion not yet.

---

### Key takeaways

- The keyframe-quantisation frontier is a property of the **format**, so ambient and gameplay animation inherit
  it.
- Split is identical to cutscenes: skeletons ✅ recoverable, keyframes ⏳ undecoded — for every animated object.
- You can rebuild rigs (crane, ship, blimp, gameplay objects) but not reproduce their motion from the banks.
- One decode unlocks **all** users — the frontier is a single, shared, per-format problem.
- Treat all animation banks as **preserve-raw**; edit rigs and drivers, not the quantised keyframes.

**Continue:** [C26.6 — Working with ambient banks](06-working-with-banks.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md)
