# C26.1 — One Format, Many Users

> **The one-sentence version:** the `__AnimationBank` MIPS-ELF format is a general animation container — a
> skeleton plus keyframed motion — so the same decoding covers cutscenes, world ambience, and gameplay
> animation; only what *drives* the bank changes.

[← Chapter 26 hub](C26-World-Ambient-Animation.md) · [Next: C26.2 — World-ambient animation →](02-world-ambient.md)

---

## A container, not a use-case

The animation bank decoded in [Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md) is a general
**animation container**: a MIPS ELF32 object whose symbol table names a skeleton
([C24.3](../C24-NIS-Animation/03-skeleton.md)) and whose `__AnimationBank` sections hold keyframed motion
([C24.4](../C24-NIS-Animation/04-skeletons-banks.md)). Nothing in that structure says "cutscene." A cutscene is
one *user* of the format — the one driven by an event script
([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) — but the same object type serves the world and gameplay.

So the right mental model is: **one format, three users.**

| User | What it animates | Driven by |
|---|---|---|
| Cutscene (NIS) | characters in a scene | scripted timeline (`ENIS` verbs) |
| World-ambient | cranes, ships, blimp | a continuous loop ([C26.2](02-world-ambient.md)) |
| Gameplay | animated gameplay objects | triggers / state ([C26.3](03-gameplay-animation.md)) |

## The decoding transfers directly

Because it's the same format, everything you learned in Chapter 24 applies unchanged:

- **Parse it as ELF** ([C24.2](../C24-NIS-Animation/02-parsing-elf.md)) — header → sections → `.symtab` →
  `.data`.
- **Recover the skeleton** from the symbol table ([C24.3](../C24-NIS-Animation/03-skeleton.md)) — the rig of
  the crane/ship/blimp/gameplay object falls out of the symbols.
- **The keyframes are in the banks** ([C24.4](../C24-NIS-Animation/04-skeletons-banks.md)) — and remain the
  frontier ([C24.5](../C24-NIS-Animation/05-keyframe-problem.md), [C26.5](05-shared-frontier.md)).

You don't relearn a format; you apply a decoded one to new objects. That's the whole point of this chapter —
recognising the reuse saves you from treating ambient animation as a mystery.

## Why EA reused the format

Using one animation format across cutscenes, ambience, and gameplay is the same economy seen everywhere in the
engine (the shared codec layer, [C20.1](../C20-Audio-Codecs/01-codec-set.md); CARP for both roads and scripts,
[C25.1](../C25-NIS-Events/01-carp-scripts.md)):

- **One runtime.** The engine ships one animation system (skeleton + keyframe evaluation); every animated
  thing routes through it.
- **One toolchain.** The animation tool ([C24.1](../C24-NIS-Animation/01-nis-bundle.md)) emits the same ELF
  banks whether the animator built a cutscene, a swinging crane, or a gameplay flourish.
- **One decode.** For you, decoding the format once covers all of it.

## What changes between users

The format is constant; three things vary by user:

- **The driver** — a script (cutscene), a loop (ambient), or a trigger (gameplay) advances the bank's clock.
- **The target** — a character rig, a world object (crane), or a gameplay entity.
- **The lifetime** — one-shot (cutscene), continuous (ambient), or event-scoped (gameplay).

Understanding a given animation is therefore mostly about identifying *which driver* plays *which bank* on
*which object* — the format underneath is the same.

---

### Key takeaways

- `__AnimationBank` is a **general** animation container (skeleton + keyframes), not cutscene-specific.
- One format, three users: cutscene (script-driven), world-ambient (loop-driven), gameplay (trigger-driven).
- Chapter 24's decoding transfers directly — parse ELF, recover the skeleton, keyframes remain the frontier.
- EA reused it for one runtime, one toolchain, one decode — the engine's usual economy.
- What varies by user is the **driver**, **target**, and **lifetime**, not the format.

**Continue:** [C26.2 — World-ambient animation](02-world-ambient.md) · [Chapter 26 hub](C26-World-Ambient-Animation.md)
