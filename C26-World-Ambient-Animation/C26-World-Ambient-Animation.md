# Chapter 26 — World-Ambient & Gameplay Animation Banks

> **Goal of this chapter:** show that the NIS animation-bank format is not cutscene-only — the same MIPS-ELF
> animation banks drive the world's ambient motion (cranes, ships, the blimp) and gameplay animation — so one
> decoded format explains animation across the whole game.

Chapter 24 decoded the NIS animation object as a MIPS ELF32 carrying skeletons and (quantised) keyframes. That
same format has **other users**. The moving cranes at the docks, the ships in the harbour, the blimp overhead,
and various gameplay animations are all driven by the **animation-bank** format — not scripted cutscenes, but
the same `__AnimationBank` ELF objects applied to world and gameplay objects. This short chapter maps those
other users and what changes (and doesn't) when the format leaves the cutscene.

> **Grounded in the verified format.** The animation-bank structure — an ELF object with skeletons in the
> symbol table and keyframes in `__AnimationBank` banks — is the one decoded in
> [Chapter 24](../C24-NIS-Animation/C24-NIS-Animation.md) against `NIS/Scene_ArrestF02_BundleB.bun` (verified
> `\x7fELF` payload, `__AnimationBank` present). This chapter applies that verified format to its non-cutscene
> users.

---

## Deep-dive pages

- [C26.1 — One format, many users](01-one-format.md): the animation-bank format beyond cutscenes.
- [C26.2 — World-ambient animation](02-world-ambient.md): cranes, ships, the blimp — the city's moving
  scenery.
- [C26.3 — Gameplay animation](03-gameplay-animation.md): animated gameplay objects driven by the same banks.
- [C26.4 — Ambient vs cutscene playback](04-ambient-vs-cutscene.md): looping ambient motion vs scripted
  timelines.
- [C26.5 — The shared frontier](05-shared-frontier.md): the keyframe-quantisation problem applies here too.
- [C26.6 — Working with ambient banks](06-working-with-banks.md): what's editable and what's preserve-raw.

---

## 26.1 The animation bank is a general format

The `__AnimationBank` ELF ([C24.4](../C24-NIS-Animation/04-skeletons-banks.md)) is a general **animation
container** — a skeleton plus keyframed motion — and nothing about it is cutscene-specific. A cutscene *uses*
it (driven by an event script, [Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)), but the world's ambient
objects and gameplay systems use the *same* format, driven differently ([C26.1](01-one-format.md)). So the
decoding you learned for NIS transfers directly: skeleton from the symbol table
([C24.3](../C24-NIS-Animation/03-skeleton.md)), keyframes in the banks (still the frontier,
[C24.5](../C24-NIS-Animation/05-keyframe-problem.md)).

## 26.2 The city moves

Most Wanted's world is not static scenery ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) alone — parts
of it **move**: dockside **cranes** swing, **ships** drift in the harbour, a **blimp** crosses the sky. These
are **world-ambient animations** — animation banks attached to world objects, playing continuously to make the
city feel alive ([C26.2](02-world-ambient.md)). They are the animated counterpart to the static props: same
world, but with motion baked into a bank.

## 26.3 Gameplay animation

Beyond ambience, various **gameplay** objects animate through the same format — animated elements tied to
events and systems rather than pure background ([C26.3](03-gameplay-animation.md)). The format is the shared
substrate; what differs is *what drives it* (an ambient loop, a gameplay trigger, or a cutscene script).

## 26.4 Driven differently

The interesting variation is playback, not format:

- **Cutscene** — driven by a scripted timeline (`ENIS` verbs, [Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)),
  one-shot, directed.
- **World-ambient** — driven by a **loop**, continuous, undirected — the crane just keeps swinging
  ([C26.4](04-ambient-vs-cutscene.md)).
- **Gameplay** — driven by **triggers/state**, playing when a gameplay condition calls for it.

Same skeleton + keyframes; three different clocks driving them.

## 26.5 The same frontier

Because it is the same format, it inherits the same open problem: the **keyframe quantisation is undecoded**
([C24.5](../C24-NIS-Animation/05-keyframe-problem.md)). So you can recover the *rigs* of the crane, ship, and
blimp (skeletons from symbols) but not yet reliably their *motion* from the banks alone
([C26.5](05-shared-frontier.md)). What's editable (skeletons, and the driving logic) and what's preserve-raw
(the quantised keyframes) is exactly as in Chapter 24.

---

### Key takeaways

- The NIS **animation-bank** format (MIPS ELF: skeleton + keyframes) is general — not cutscene-only.
- **World-ambient** animation (cranes, ships, blimp) uses it to make the city move — the animated counterpart
  to static scenery.
- **Gameplay** objects animate through the same banks, driven by triggers/state.
- The format is shared; only the **driver** differs — cutscene script vs ambient loop vs gameplay trigger.
- It inherits Chapter 24's frontier: skeletons recoverable, **keyframes still undecoded** — same editable/
  preserve-raw split.

**Next:** [Chapter 27 — Front-End: Shell Scenes & UI Atlases](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md):
leaving the world for the menus.
