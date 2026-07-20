# C22.4 — The RPM→Synth Bridge

> **The one-sentence version:** each frame the runtime takes the engine's current RPM from the vehicle
> simulation, maps it into the file's [rpmMin, rpmMax] range to pick the grain(s), pitch-shifts them to the
> exact rev, and crossfades adjacent grains — so the engine note is computed from revs, not triggered.

[← C22.3 — Grains & the grain table](03-grains.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md) ·
[Next: C22.5 — Accel, decel & load →](05-accel-decel.md)

---

## The bridge

The "RPM→synth bridge" connects the *simulation* to the *sound*. Every audio frame:

```
vehicle simulation → current RPM (e.g. 5200)
   → normalise into [rpmMin, rpmMax]      (2267…8638 → position in the rev band)
   → select grain(s) for that position     (grain table, C22.3)
   → pitch-shift the grain to hit exactly 5200 RPM
   → crossfade adjacent grains for a seamless sweep
   → engine note out (mixed with decel/load — C22.5)
```

The current RPM is the input, the engine note is the output, and the GIN file's grains + RPM range are the
synthesis parameters. Because it runs every frame, the note tracks the tachometer continuously — rev up and the
pitch rises smoothly through the grains; lift off and it falls.

## Select, pitch, crossfade

Three operations turn a rev number into sound:

- **Select.** The normalised RPM position indexes into the grains ([C22.3](03-grains.md)) — which grain(s)
  characterise this part of the rev range.
- **Pitch-shift.** A grain captures *near* a rev level, not exactly it; the synth resamples (pitch-shifts) the
  grain so its fundamental matches the *exact* current RPM. This is what lets a few hundred grains cover every
  RPM continuously — you pitch between them.
- **Crossfade.** As RPM moves from one grain's region to the next, the synth blends the outgoing and incoming
  grains so there's no audible step — a smooth timbral morph across the rev range.

Together these produce a note that is both *pitch-accurate* (matches the tach) and *timbre-continuous* (no
seams between grains).

## Why this beats a sampler

A naive engine sound plays a fixed clip per gear or rev band and pitch-shifts one clip — which sounds
artificial at the band edges and can't morph timbre. The granular bridge is better because:

- **Pitch and timbre are decoupled.** Pitch comes from resampling to the exact RPM; timbre comes from *which*
  grains — so a high rev sounds both higher *and* different (harsher), as real engines do, not just a sped-up
  low rev.
- **Continuous, not stepped.** Crossfading grains avoids the "gear-change click" of clip-based engines.
- **Reactive.** Because it's recomputed from live RPM, it responds instantly to throttle blips, gearshifts, and
  redline — the sound *is* the simulation's RPM.

> ✅ *Verified:* the file provides the RPM range ([C22.2](02-gnsu-header.md)) and grain table
> ([C22.3](03-grains.md)) the bridge consumes; the RPM is sourced from the vehicle simulation
> ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).
> 🟡 *Reasoned:* the exact select/pitch/crossfade algorithm is the runtime synth's behaviour, described from the
> granular-synthesis model and the file's parameters; the file's inputs (range, grains) are verified.

## RPM comes from the simulation

The bridge's input — engine RPM — is produced by the vehicle physics
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)): the engine behavior and gearing compute the
current revs from throttle, speed, and gear. So the audio and the simulation are tightly coupled: change the
car's gearing (revs per speed) and the engine *sounds* different at a given speed, because the RPM feeding the
synth changed. This is why tuning and sound feel connected — they share the RPM signal.

## Editing implications

- **The bridge is fixed; you edit the file.** You can't change how RPM maps to grains at runtime — you change
  the *grains* (timbre) and the *RPM range* (where the timbre sits) in the GIN
  ([C22.6](06-editing-gin.md)).
- **Keep the RPM range matched to the car.** If a car revs to 8600, its GIN's `rpmMax` should too, or the synth
  runs off the end of the grains ([C22.2](02-gnsu-header.md)).
- **Grain ordering is the rev map.** Because grains map to rev positions by index/order, preserve ordering
  unless you intend to re-map ([C22.3](03-grains.md)).
- **Test across the rev range.** A GIN edit is only right if it sounds correct from idle to redline and on
  overrun — rev it through the whole band ([C22.6](06-editing-gin.md)).

---

### Key takeaways

- The bridge maps live **RPM → grain selection → pitch-shift → crossfade → note**, every frame.
- Select picks grains by rev position; pitch-shift hits the exact RPM; crossfade blends adjacent grains
  seamlessly.
- It beats a sampler by decoupling pitch and timbre, staying continuous, and reacting instantly.
- RPM comes from the vehicle simulation, so gearing/tuning and engine sound are coupled through the rev signal.
- You edit the file (grains, RPM range), not the fixed bridge; keep the range matched to the car and test across
  all revs.

**Continue:** [C22.5 — Accel, decel & load](05-accel-decel.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md)
