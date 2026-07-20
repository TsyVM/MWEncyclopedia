# C24.4 — Skeletons vs Animation Banks

> **The one-sentence version:** a NIS bundle separates the rig from the motion — the `0x00E34009` ELF payloads
> carry the **skeletons**, while separate `__AnimationBank` ELFs carry the **keyframes** — joined by the bones
> they share.

[← C24.3 — The bind-pose skeleton](03-skeleton.md) · [Chapter 24 hub](C24-NIS-Animation.md) ·
[Next: C24.5 — The keyframe-quantisation problem →](05-keyframe-problem.md)

---

## Two kinds of ELF object

A NIS bundle holds **several** ELF objects, and they divide into two roles:

| Object | Chunk / name | Carries |
|---|---|---|
| **Skeleton** | `0x00E34009` payloads | the rig — bones + bind-pose ([C24.3](03-skeleton.md)) |
| **Animation bank** | `__AnimationBank` ELFs | the keyframes — motion over time ([C24.5](05-keyframe-problem.md)) |

Verified: the retail bundle has three `0x00E34009` payloads and a separate `__AnimationBank` ELF. So the rig
and its animation are **different objects**, not one combined blob — the same separation-of-concerns as scenery
models vs instances ([C16.2](../C16-Scenery-Cull/02-models-instances.md)) or textures vs materials
([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)).

## Why split rig from motion

Separating the skeleton from the animation bank is standard animation-pipeline design, and it buys the usual
things:

- **Reuse.** One skeleton can be driven by many animation banks — a character rig plus a library of motions.
  Storing them separately means the rig isn't duplicated per animation.
- **Independent authoring.** Riggers build skeletons; animators build motion; the two are produced and edited
  separately and joined at runtime.
- **Clean binding.** The animation bank targets bones *by name/index* in the skeleton, so any compatible bank
  can drive a rig ([C24.3](03-skeleton.md)).

This is the animation analogue of the model/instance and texture/material splits: a stable definition (the rig)
and the thing that varies over it (the motion).

## The join is the bones

The animation bank and the skeleton connect through the **bones**: a keyframe in the bank targets a specific
bone, identified by the name/index the skeleton defines. So playback is:

```
skeleton (bind pose) + animation bank (keyframes) → per-frame bone transforms → posed character
   C24.3                    C24.5                        (compose down the hierarchy)
```

Each frame, the bank's keyframes give each bone a deviation from its bind pose, and composing those down the
hierarchy poses the character. The skeleton is the *what* (which bones), the bank is the *how* (their motion) —
and the bones are the shared vocabulary.

> ✅ *Verified:* the bundle contains `0x00E34009` skeleton payloads and a separate `__AnimationBank` ELF —
> confirming the rig/motion split.
> 🟡 *Reasoned:* that the bank targets skeleton bones by name/index is the standard binding and consistent with
> the ELF symbol design; the split into skeleton and bank objects is verified.

## Decoded and undecoded halves

The split maps neatly onto what is and isn't solved:

- **Skeletons (`0x00E34009`)** — **decoded**. The rig comes out of the symbol table
  ([C24.3](03-skeleton.md)).
- **Animation banks (`__AnimationBank`)** — **undecoded keyframes**. The motion data is quantised in a scheme
  that hasn't been cracked ([C24.5](05-keyframe-problem.md)).

So you can recover *who* the characters are and *how they're built*, but not yet reliably *how they move*, from
the file alone. The split makes the frontier precise: it's the bank, not the skeleton.

## Editing implications

- **Skeleton and bank are edited separately.** Rig changes go in the `0x00E34009` ELF; motion changes in the
  `__AnimationBank` ELF ([C24.2](02-parsing-elf.md)).
- **Keep the binding intact.** Rename or reindex a bone in the skeleton and every bank targeting it must follow
  ([C24.3](03-skeleton.md)).
- **Skeleton edits are feasible; bank edits are the frontier.** Because keyframes are undecoded
  ([C24.5](05-keyframe-problem.md)), editing motion is not yet reliable.
- **Reuse works.** A compatible bank can drive a rig — the split is what enables animation reuse.

---

### Key takeaways

- A NIS bundle splits **skeletons** (`0x00E34009` ELFs) from **animation banks** (`__AnimationBank` ELFs) —
  verified.
- The split enables rig reuse, independent authoring, and clean name/index binding — standard pipeline design.
- The two join through **bones**: the bank's keyframes target the skeleton's bones, composed to pose the
  character.
- Decoded half = skeletons (from symbols); undecoded half = **keyframes** in the banks (the frontier).
- Edit rig and bank separately, keep the bone binding intact; skeleton edits are feasible, motion edits are not
  yet.

**Continue:** [C24.5 — The keyframe-quantisation problem](05-keyframe-problem.md) · [Chapter 24 hub](C24-NIS-Animation.md)
