# C24.5 — The Keyframe-Quantisation Problem

> **The one-sentence version:** the animation banks store per-bone motion as *quantised* keyframes, and the
> quantisation scheme — how the compact stored values map back to rotations and translations over time — has
> not been decoded, so the rig is recoverable but full playback from the file is not.

[← C24.4 — Skeletons vs animation banks](04-skeletons-banks.md) · [Chapter 24 hub](C24-NIS-Animation.md) ·
[Next: C24.6 — Animated on PS2, static on PC →](06-ps2-vs-pc.md)

---

## The honest frontier

A reference earns trust by marking where knowledge ends. For NIS animation, the boundary is sharp: the
**skeleton is decoded** ([C24.3](03-skeleton.md)), but the **keyframes are not.** The `__AnimationBank` ELFs
([C24.4](04-skeletons-banks.md)) hold the per-bone motion as **quantised** values, and the quantisation scheme
— the mapping from those compact stored numbers back to real rotations/translations across time — has resisted
decoding. So you can recover *who moves* and *the rig they move*, but not reliably *the motion itself*, from the
file alone.

> ⏳ *Open:* the NIS keyframe quantisation is undecoded — the chapter's stated frontier. The skeleton
> ([C24.3](03-skeleton.md)) and the bundle/ELF structure ([C24.1](01-nis-bundle.md)–[C24.2](02-parsing-elf.md))
> are ✅ verified.

## What "quantised keyframes" means

Animation keyframes store, per bone per time, a **rotation** (and sometimes translation/scale). Stored naively
that's expensive — many bones × many frames × full-precision quaternions. So animation formats **quantise**:
they compress each value into a few bits using a scheme (fixed-point, a compressed quaternion representation,
per-channel ranges, delta coding, etc.). Decoding requires knowing *exactly* which scheme, because the same
bits mean different rotations under different quantisers.

The NIS banks are quantised this way, and the specific scheme — how many bits, which representation, what
ranges, how time is encoded — is what's unknown. Without it, the stored bytes are opaque numbers rather than
poses.

## Why it's hard

Keyframe quantisation is a notoriously fiddly thing to reverse without the code, for reasons that all apply
here:

- **Many plausible schemes.** Compressed quaternions alone have several common encodings (smallest-three,
  axis-angle fixed-point, per-component); guessing wrong yields plausible-but-wrong motion.
- **Ranges and reference frames.** Values are often relative to the bind pose ([C24.3](03-skeleton.md)) and
  scaled by per-channel ranges — you need those to reconstruct absolute rotations.
- **Time encoding.** Keyframes may be uniformly spaced or have their own time stamps, with interpolation between
  them; the timing model is part of the puzzle.
- **No ground truth on PC.** Because NIS animation is **static on PC** ([C24.6](06-ps2-vs-pc.md)), the PC build
  doesn't exercise the playback path, so there's less to compare against.

Cracking it would require either the decode code (from the MIPS `.text`, though it targets PS2) or a careful
statistical/structural analysis against known motions — neither trivial.

## What you can still do

The undecoded keyframes don't make NIS useless — the decoded half supports real work:

- **Recover and render the rig** in bind pose ([C24.3](03-skeleton.md)) — the characters, their skeletons, the
  scene's cast.
- **Extract the scene's assets** — textures ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) and skeletons
  from the bundle ([C24.1](01-nis-bundle.md)).
- **Read the event script** — the cutscene's *direction* (camera, timing, events) is a separate, decoded system
  ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)), so you know *what happens* even if the character motion
  is opaque.

So a NIS scene is partly open: its structure, cast, and script are readable; its character motion is the gap.

## Editing implications

- **Don't edit keyframes blind.** Without the quantisation scheme, editing bank motion data is guesswork that
  produces wrong poses ([C24.4](04-skeletons-banks.md)).
- **Skeleton and script edits are feasible.** The rig ([C24.3](03-skeleton.md)) and the event timeline
  ([Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)) are decoded — edit those instead.
- **Mark it in tooling.** A NIS tool should treat animation banks as **preserve-raw** — read/round-trip them
  byte-exact, don't attempt to interpret ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)).
- **This is where contribution matters.** If the quantisation is ever cracked, NIS motion becomes editable —
  it's the chapter's open invitation.

---

### Key takeaways

- The **keyframes are undecoded**: the `__AnimationBank` motion is quantised in a scheme that hasn't been
  cracked (⏳).
- Quantised keyframes compress per-bone rotations/translations over time; decoding needs the *exact* scheme.
- It's hard because of many plausible encodings, bind-pose-relative ranges, time encoding, and no PC ground
  truth ([C24.6](06-ps2-vs-pc.md)).
- You can still recover the **rig**, extract assets, and read the **event script** (Chapter 25) — the scene is
  partly open.
- Treat animation banks as **preserve-raw**; edit skeletons and scripts instead; cracking the quantisation is
  the open frontier.

**Continue:** [C24.6 — Animated on PS2, static on PC](06-ps2-vs-pc.md) · [Chapter 24 hub](C24-NIS-Animation.md)
