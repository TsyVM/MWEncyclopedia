# Chapter 24 — Animations & Cutscenes: the NIS Object

> **Goal of this chapter:** open a NIS cutscene bundle and decode its animation payload — a MIPS ELF32
> relocatable object carrying the skeleton and keyframes — understand the skeleton from its symbol table, and
> confront the keyframe-quantisation problem that keeps full playback out of reach.

Not every cutscene is a video ([Chapter 23](../C23-Video-VP6/C23-Video-VP6.md)). The **NIS** ("Non-Interactive
Sequence") cutscenes — an arrest, a story beat — are *rendered in-engine* from scripted camera and character
animation, not pre-recorded film. Their animation data has a striking form: an **ELF object file**, the kind a
compiler emits. This chapter decodes that payload, recovers the skeleton, and is honest about where the
keyframes remain unsolved.

> **Verified against retail data.** `NIS/Scene_ArrestF02_BundleB.bun` is an EAGL bundle (magic `0xB3300000`)
> holding TPKs and **three `0x00E34009` animation payloads**; the first payload at `0xEE340` is an **8-byte
> `0x11` sentinel followed by `\x7fELF`** — a little-endian MIPS **ELF32** relocatable object — and a separate
> **`__AnimationBank`** ELF appears later in the bundle. Both facts were read directly from the file.

---

## Deep-dive pages

- [C24.1 — The NIS bundle & the ELF payload](01-nis-bundle.md): the bundle's chunks and the `0x00E34009`
  animation object.
- [C24.2 — Parsing the MIPS ELF32](02-parsing-elf.md): reading the payload like any ELF — header, sections,
  symbol table, data.
- [C24.3 — The bind-pose skeleton](03-skeleton.md): recovering the rig from the ELF symbol table.
- [C24.4 — Skeletons vs animation banks](04-skeletons-banks.md): the `0x00E34009` skeleton ELFs vs the
  `__AnimationBank` keyframe ELFs.
- [C24.5 — The keyframe-quantisation problem](05-keyframe-problem.md): why the keyframes remain unsolved, and
  what is known.
- [C24.6 — Animated on PS2, static on PC](06-ps2-vs-pc.md): the platform split in how NIS animation plays.

---

## 24.1 An ELF object as animation data

The animation payload — chunk `0x00E34009` — is not a flat struct. After an **8-byte `0x11` sentinel** (the
same fill marker as the geometry buffers, [C9.1](../C9-Meshes-FVF/01-vertex-buffer.md)) it is a **little-endian
MIPS ELF32 relocatable object** — the exact artifact a compiler/linker toolchain produces, complete with
section headers and a symbol table ([C24.1](01-nis-bundle.md)). This is unusual and revealing: EA's animation
tool emitted rigs and animation as *object files*, so you parse them with ELF tooling, not a bespoke reader.

## 24.2 Parse it like any ELF

Because it's a genuine ELF32, decoding is standard:

```python
def parse_nis_anim(payload):
    body = payload[8:]                    # skip the 8-byte 0x11 sentinel
    assert body[:4] == b"\x7fELF"
    elf = Elf32(body)                     # header → section headers → .symtab → .data
    return elf
```

Header → section headers → symbol table (`.symtab`) → the `.data` section — the same steps as reading any
relocatable object ([C24.2](02-parsing-elf.md)). The `EAGL4::SymbolPool` evidence in the symbols confirms this
came from EA's animation toolchain.

## 24.3 The skeleton falls out of the symbol table

An ELF symbol table is exactly the right shape for a **rig**: it names every bone and skeleton and points each
at its data. So the **bind-pose skeleton** — the hierarchy of bones and their rest transforms — is recoverable
directly from `.symtab`: the symbols name the bones, and their data gives the pose ([C24.3](03-skeleton.md)).
The rig hierarchy comes for free from the object's own symbols.

## 24.4 Skeletons here, keyframes there

A NIS bundle holds **several** ELF objects with a division of labour:

- **`0x00E34009` payloads** carry the **skeletons** (the rigs).
- Separate **`__AnimationBank`** ELFs carry the **keyframes** (the motion over time).

So the rig and its animation are separate objects, joined by the bones they share
([C24.4](04-skeletons-banks.md)). The skeleton is decoded; the keyframes are where the trouble is.

## 24.5 The keyframes are unsolved

The honest frontier of this chapter: the **keyframe data is not fully decoded.** The animation banks hold the
per-bone motion as **quantised** keyframes, and the quantisation scheme — how the compact stored values map
back to rotations/translations over time — has resisted decoding ([C24.5](05-keyframe-problem.md)). So you can
recover the *rig* (skeleton) but not yet reliably *play back* the *motion* from the file alone. This is stated
plainly rather than glossed.

## 24.6 A platform split

A final wrinkle: NIS animation **plays on PS2 but is static on PC** — the PC build renders the NIS scenes
without the character animation the PS2 version shows ([C24.6](06-ps2-vs-pc.md)). This platform difference
explains some of why the PC keyframe path is under-exercised and under-documented, and it's a caution when
reasoning about what a NIS scene *should* look like.

---

### Key takeaways

- NIS cutscenes are **rendered in-engine**, not video; their animation is a **MIPS ELF32** object (chunk
  `0x00E34009`, after an 8-byte `0x11` sentinel) — verified.
- Parse it like any **ELF**: header → sections → `.symtab` → `.data`; `EAGL4::SymbolPool` marks EA's toolchain.
- The **bind-pose skeleton** is recoverable from the symbol table — the rig falls out of the ELF symbols.
- A bundle splits **skeletons** (`0x00E34009`) from **keyframes** (`__AnimationBank` ELFs).
- The **keyframe quantisation is unsolved** (the frontier), and NIS animation is **animated on PS2, static on
  PC**.

**Next:** [Chapter 25 — NIS Event Timelines, Scripts & Playback](../C25-NIS-Events/C25-NIS-Events.md): the
scripts that drive a cutscene.
