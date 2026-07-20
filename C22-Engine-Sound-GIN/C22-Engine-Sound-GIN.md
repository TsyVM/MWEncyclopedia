# Chapter 22 — Engine Sound (GIN) & the RPM→Synth Bridge

> **Goal of this chapter:** decode the format behind a car's engine note — the `Gnsu` granular-synthesis file
> that stores an RPM range and a table of waveform grains — and follow the runtime bridge that turns the
> current engine RPM into a continuous, reactive engine sound.

A racing game lives or dies on its engine sound, and Most Wanted's is not a recording — it is **synthesised**.
Each car's engine note is a **GIN** file: a granular-synthesis unit that stores short waveform **grains** and
an **RPM range**, and the runtime blends and pitches those grains according to how hard you're revving. This
chapter decodes the GIN format from the retail files and explains the RPM→sound bridge.

> **Verified against retail data.** Of the 160 GIN files in `SOUND/`, **159 carry the magic `Gnsu`**
> (granular-synthesis unit); the one exception is a tool-exported RIFF/WAVE. Decoding `GIN_Acura_ITR.gin`
> directly gives magic `Gnsu`, version `"20"`, **rpmMin = 2267.0 at `+0x08`** and **rpmMax = 8638.1 at
> `+0x0C`** (f32), count fields at `+0x10`/`+0x14`, a data offset at `+0x18`, and a **table of increasing grain
> offsets** beginning at `+0x20`. (This corrects an older community-sourced header layout that placed the RPM
> fields at `+0x20`/`+0x24`.)

---

## Deep-dive pages

- [C22.1 — GIN is granular synthesis](01-granular-synthesis.md): why the engine note is synthesised from grains
  rather than recorded.
- [C22.2 — The Gnsu header](02-gnsu-header.md): magic, version, RPM range, counts, and the grain table —
  decoded from the retail file.
- [C22.3 — Grains & the grain table](03-grains.md): the waveform snippets and the offset table that indexes
  them.
- [C22.4 — The RPM→synth bridge](04-rpm-bridge.md): how the runtime maps current RPM to grains, pitch, and
  crossfade.
- [C22.5 — Accel, decel & load](05-accel-decel.md): the `_DCL` deceleration files and the throttle dimension.
- [C22.6 — Editing engine sound](06-editing-gin.md): swapping the note while keeping the RPM map intact.

---

## 22.1 The engine note is synthesised

No recording can cover a car sweeping continuously from idle to redline under varying load — so Most Wanted
doesn't try. GIN is **granular synthesis**: it stores many short waveform **grains**, each characteristic of
part of the rev range, and the runtime **blends and pitches** them to produce a continuous note that tracks the
engine exactly ([C22.1](01-granular-synthesis.md)). Rev higher and the synth shifts toward higher-RPM grains
and pitches them up; the sound follows the tachometer.

## 22.2 The Gnsu header, decoded

The header — verified on `GIN_Acura_ITR.gin` — is compact and legible:

| Offset | Type | Value | Field |
|---|---|---|---|
| `+0x00` | char[4] | `Gnsu` | magic (granular-synthesis unit) |
| `+0x04` | char[4] | `"20"` | version |
| `+0x08` | `f32` | **2267.0** | **rpmMin** — lowest RPM this file covers |
| `+0x0C` | `f32` | **8638.1** | **rpmMax** — highest RPM (near redline) |
| `+0x10` | `u32` | 50 | a count |
| `+0x14` | `u32` | 229 | grain count |
| `+0x18` | `u32` | 162344 | offset to sample/grain data |
| `+0x20` | `u32[]` | 0x1E0, 0xE5D, 0x19AB, … | **grain-offset table** (increasing) |

The RPM range at `+0x08`/`+0x0C` is the heart of it: this engine's synthesised note is defined between 2267 and
8638 RPM ([C22.2](02-gnsu-header.md)).

## 22.3 Grains indexed by offset

Following the header, the **grain-offset table** at `+0x20` lists the start of each waveform grain in the sample
data — the offsets increase monotonically (0x1E0, 0xE5D, 0x19AB, …), roughly evenly spaced, one entry per
grain. Each grain is a short snippet of engine waveform; together they span the RPM range
([C22.3](03-grains.md)). The synth picks grains by RPM and stitches them.

## 22.4 RPM drives the synth

The **bridge** from simulation to sound is RPM: each frame, the engine's current RPM (from the vehicle
simulation — [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) is mapped into the file's
[rpmMin, rpmMax] range, selecting the grain(s) for that rev level, **pitch-shifting** them to hit the exact RPM,
and **crossfading** between adjacent grains for a seamless sweep ([C22.4](04-rpm-bridge.md)). That is why the
engine sound rises and falls perfectly with the tachometer — it is *computed* from the RPM, not triggered.

## 22.5 Accel and decel

The engine sounds different accelerating (on-throttle, loaded) than decelerating (off-throttle, overrun), so
each car ships two GINs: the main file (acceleration) and a **`_DCL`** deceleration file — 73 of the 160 GINs
are `_DCL`. The synth blends between them based on **throttle/load**, adding a second dimension to the RPM sweep
([C22.5](05-accel-decel.md)).

---

### Key takeaways

- Engine sound is **GIN** — granular synthesis (`Gnsu`), not a recording; 159/160 retail GINs use the format.
- The header (verified): magic `Gnsu`, version `"20"`, **rpmMin `+0x08`**, **rpmMax `+0x0C`**, counts, data
  offset, and a **grain-offset table at `+0x20`**.
- Grains are short waveform snippets indexed by the offset table, spanning the RPM range.
- The **RPM→synth bridge** maps current RPM into the range, selects and pitch-shifts grains, and crossfades —
  the sound is computed from revs.
- Accel and `_DCL` decel GINs blend by throttle/load, adding the load dimension.

**Next:** [Chapter 23 — Video (EA Multimedia / VP6)](../C23-Video-VP6/C23-Video-VP6.md): the game's movies.
