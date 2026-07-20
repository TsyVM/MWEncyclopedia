# C26.6 — Working with Ambient Banks

> **The one-sentence version:** the tractable work on animated world/gameplay objects is extracting rigs,
> tracing drivers, and round-tripping banks byte-exact — the quantised keyframes stay preserve-raw until the
> format's motion decode is solved.

[← C26.5 — The shared frontier](05-shared-frontier.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md) ·
[Next: Chapter 27 — Front-End: Shell Scenes & UI Atlases →](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)

---

## What you can do today

Working with world-ambient and gameplay animation banks, the feasible tasks are the ones that don't depend on
the undecoded keyframes ([C26.5](05-shared-frontier.md)):

- **Extract the rig.** Parse the ELF ([C24.2](../C24-NIS-Animation/02-parsing-elf.md)) and recover the
  skeleton from the symbol table ([C24.3](../C24-NIS-Animation/03-skeleton.md)) — the crane/ship/blimp/gameplay
  object's bone hierarchy and bind pose.
- **Identify the object and its driver.** Determine whether a bank is loop-driven (ambient), cue-driven
  (gameplay), or script-driven (cutscene) ([C26.4](04-ambient-vs-cutscene.md)) — the animation's behaviour
  lives in the driver.
- **Trace the cue.** For gameplay animation, follow the trigger/event
  ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)) that starts it.
- **Round-trip the bank byte-exact.** Read and rewrite the animation bank without interpreting its keyframes —
  the preserve-raw discipline ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)).

## What to leave alone

Two things are not safe to edit until the format's motion decode lands:

- **The quantised keyframes.** Editing them blind produces wrong motion
  ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)); treat them as opaque.
- **Anything that assumes a keyframe interpretation.** Don't build tooling that "adjusts" motion values you
  can't read.

Treat the bank's motion data the way you'd treat any undecoded region: **preserve it, don't interpret it.**

## Editing behaviour via the driver

You *can* change how an ambient or gameplay animation behaves — by editing its **driver**, not its motion
([C26.4](04-ambient-vs-cutscene.md)):

- **Ambient loops** — adjust the loop range or rate to change how the crane swings or how fast the blimp moves
  (as far as the driver exposes these), without touching the keyframes' meaning.
- **Gameplay cues** — change the trigger/condition ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md))
  that starts a gameplay animation — when it plays, not what the motion is.
- **Enable/disable** — a mover can be added or removed as a placed object (like scenery,
  [C16.6](../C16-Scenery-Cull/06-editing-scenery.md)) even if its internal motion is opaque.

So behaviour is editable at the *system* level (when and whether it plays) even while the *motion* is not.

## A checklist for an unfamiliar animated object

When you meet an animated world/gameplay object:

1. **Find the bank.** Locate the `__AnimationBank` / `0x00E34009` ELF for the object
   ([C24.1](../C24-NIS-Animation/01-nis-bundle.md)).
2. **Recover the skeleton.** Parse the ELF symbols ([C24.3](../C24-NIS-Animation/03-skeleton.md)) — you now
   know the rig.
3. **Classify the driver.** Loop (ambient), cue (gameplay), or script (cutscene) — [C26.4](04-ambient-vs-cutscene.md).
4. **Trace the cue** if gameplay ([Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md)).
5. **Preserve the keyframes** ([C26.5](05-shared-frontier.md)) — don't interpret them yet.

This gets you a full structural understanding (rig + driver + cue) of any animated object, with the motion
honestly bracketed as the open piece.

## The through-line

Chapter 26 closes the animation story that Chapters 24–25 opened: one MIPS-ELF format, three drivers (script,
loop, cue), one shared frontier. Cutscenes, the living city, and gameplay animation are the same format wearing
different clocks — decode it once (rigs done, motion pending) and you understand animation across the whole
game.

---

### Key takeaways

- Feasible work: extract rigs, classify the driver, trace gameplay cues, and round-trip banks byte-exact.
- Leave the **quantised keyframes** alone (preserve-raw) until the format's motion decode is solved.
- You can edit **behaviour via the driver** (loop range/rate, gameplay cue, enable/disable) without touching
  motion.
- A checklist — find bank → recover skeleton → classify driver → trace cue → preserve keyframes — fully maps an
  animated object.
- The chapter closes the animation arc: one format, three drivers, one frontier — rigs decoded, motion pending.

**Continue:** [Chapter 27 — Front-End: Shell Scenes & UI Atlases](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md) ·
[Chapter 26 hub](C26-World-Ambient-Animation.md)
