# C24.6 — Animated on PS2, Static on PC

> **The one-sentence version:** the NIS animation objects are MIPS ELF built for the PS2, and the PS2 build
> plays them as full character animation — but the PC build renders the same scenes largely *static*, which is
> both a curiosity and part of why the PC keyframe path is under-exercised.

[← C24.5 — The keyframe-quantisation problem](05-keyframe-problem.md) · [Chapter 24 hub](C24-NIS-Animation.md) ·
[Next: Chapter 25 — NIS Event Timelines, Scripts & Playback →](../C25-NIS-Events/C25-NIS-Events.md)

---

## The platform split

The NIS animation data ships in the PC files, but the two platforms use it differently:

- **PS2** — plays the NIS scenes with **full character animation**: the characters move, driven by the
  keyframes ([C24.5](05-keyframe-problem.md)) through the skeleton ([C24.3](03-skeleton.md)).
- **PC** — renders the same scenes **largely static**: the scene, camera, and cast are there, but the character
  motion is not played the way it is on PS2.

So the data is present on PC, but the PC runtime doesn't drive it the same way. This is the kind of
platform-specific behaviour that only becomes clear when you compare builds — and it directly shapes what the
PC files can tell you.

## Why it makes sense

The split is consistent with the format's origins ([C24.1](01-nis-bundle.md)):

- **The objects are MIPS ELF.** MIPS is the PS2's CPU family; the animation objects (and any decode code in
  their `.text`) were built for that architecture. The PC port carries the data but doesn't necessarily run the
  same MIPS-targeted playback path.
- **NIS was a PS2-era feature.** Non-interactive rendered sequences were more central on the console; the PC
  build may have deprioritised the character-animation path (using video, static scenes, or reduced NIS instead).
- **Effort follows the lead platform.** Where a feature is fully realised on one platform and vestigial on
  another, the second platform's path is often under-tested — which is exactly the situation here.

> 🟡 *Reasoned:* the PS2-animated / PC-static behaviour is an observed platform difference; the ✅ verified facts
> are that the animation objects are **MIPS ELF** ([C24.1](01-nis-bundle.md)) and that the skeleton is decoded
> ([C24.3](03-skeleton.md)) while the keyframes are not ([C24.5](05-keyframe-problem.md)).

## Why it matters for decoding

The platform split is not just trivia — it explains the frontier:

- **No PC ground truth.** Because the PC build doesn't play the character animation, you can't watch the PC game
  to see what a correct keyframe decode *should* produce — removing the easiest validation for cracking the
  quantisation ([C24.5](05-keyframe-problem.md)).
- **The decode path may be PS2-side.** If the runtime code that interprets the quantised keyframes is
  MIPS-targeted and effectively unused on PC, reverse-engineering it means reading PS2 code, not PC code.
- **Expectations must be calibrated.** When reasoning about "what a NIS scene looks like," remember the PC
  reference is static — the animated version lives on PS2.

So the split both *causes* some of the difficulty (no PC ground truth, PS2-side decode) and *warns* you not to
assume the PC behaviour is the whole story.

## What to take from it

For anyone working with NIS on PC:

- **The data is richer than the PC playback.** The files carry full animation the PC build doesn't show — so
  the assets (skeletons, keyframes, script) are worth extracting even though PC renders them static.
- **Cross-platform comparison is the way in.** If the quantisation is ever cracked, the PS2 build is the
  reference to validate against, not the PC one.
- **Don't mistake static-on-PC for missing.** The animation *is* in the files ([C24.4](04-skeletons-banks.md));
  it's the PC playback path that doesn't drive it.

---

### Key takeaways

- NIS animation is **animated on PS2, largely static on PC** — the data ships on both, but only PS2 plays the
  character motion.
- This fits the format's **MIPS ELF** (PS2) origins and NIS being a console-era feature.
- It explains the frontier: **no PC ground truth** for validating a keyframe decode, and a likely **PS2-side
  decode path** ([C24.5](05-keyframe-problem.md)).
- The PC files are **richer than PC playback** — extract the skeletons, banks, and script regardless.
- Validate any future keyframe decode against the **PS2** build; don't mistake PC's static rendering for missing
  data.

**Continue:** [Chapter 25 — NIS Event Timelines, Scripts & Playback](../C25-NIS-Events/C25-NIS-Events.md) ·
[Chapter 24 hub](C24-NIS-Animation.md)
