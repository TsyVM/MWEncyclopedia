# C22.5 — Accel, Decel & Load

> **The one-sentence version:** an engine sounds different on-throttle than off, so each car ships two GINs —
> the main (acceleration) file and a `_DCL` deceleration file — and the synth blends between them by
> throttle/load, adding a second dimension to the RPM sweep.

[← C22.4 — The RPM→synth bridge](04-rpm-bridge.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md) ·
[Next: C22.6 — Editing engine sound →](06-editing-gin.md)

---

## Two files per car

Engine sound has two dimensions, not one. RPM ([C22.4](04-rpm-bridge.md)) is the primary axis, but **load** —
whether the engine is pulling (on-throttle) or overrunning (off-throttle) — changes the note dramatically at
the *same* RPM. So each car carries two GINs:

- **The main GIN** (e.g. `GIN_Acura_ITR.gin`) — the **acceleration** / on-load note.
- **The `_DCL` GIN** (e.g. `GIN_Acura_ITR_DCL.gin`) — the **deceleration** / overrun note.

Verified: of the 160 GIN files, **73 are `_DCL`** — roughly one deceleration file per car alongside its main
file. `DCL` = deceleration.

## Why load needs its own grains

On-throttle and off-throttle are acoustically different engines:

- **Accelerating** — the engine is under load, combustion is strong, the note is full and aggressive.
- **Decelerating** — the throttle is closed, the engine is being driven by the wheels (overrun), producing a
  lighter, sometimes popping/backfiring character.

A single set of grains can't capture both, because they differ in *timbre* at the same RPM, not just pitch. So
the deceleration character gets its own grain set in the `_DCL` file, and the synth chooses/blends between the
two based on load.

## Blending by throttle/load

The synth runs the RPM bridge ([C22.4](04-rpm-bridge.md)) on *both* files and **crossfades between them by
throttle/load**:

```
           full throttle ───────────────────────────── closed throttle
main GIN:  ████████████████▓▓▓▓▓░░░░░░
_DCL GIN:  ░░░░░░░░░░░▓▓▓▓▓████████████████
                     ↑ blend point moves with throttle/load
```

- **On throttle** → mostly the main (accel) GIN.
- **Off throttle** → mostly the `_DCL` (decel) GIN.
- **Part throttle / transitions** → a blend, so lifting off produces a smooth timbral shift into overrun rather
  than a hard switch.

So the engine note is a 2-D synthesis: **RPM** selects the pitch/timbre within each file, and **load** blends
between the accel and decel files. Both axes are live, so the sound reacts to how you drive — flooring it vs
coasting sound different at the same speed.

> ✅ *Verified:* two GINs per car (main + `_DCL`); 73 `_DCL` files among 160 GINs, one deceleration file per car
> alongside its main file.
> 🟡 *Reasoned:* the throttle/load crossfade between the two files is the synth's behaviour, described from the
> accel/decel file pairing; the pairing itself is verified.

## The load signal

Like RPM, the load/throttle signal comes from the simulation and the player's input
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — throttle position and engine load. So the
engine sound reflects both *what the car is doing* (RPM) and *what you're asking of it* (throttle), making it a
direct audio readout of the driving. This coupling is why a well-tuned engine sound feels connected to the
controls — it is literally driven by them.

## Editing implications

- **Edit both files together.** To re-voice a car's engine, replace *both* the main and `_DCL` GINs, or accel
  and decel won't match ([C22.6](06-editing-gin.md)).
- **Keep the accel/decel character distinct.** The `_DCL` should sound like overrun (lighter/popping), the main
  like load — swapping or blurring them makes the engine feel wrong on lift-off.
- **Match RPM ranges across the pair.** Both files should cover the same rev band
  ([C22.2](02-gnsu-header.md)) so the load crossfade stays aligned across revs.
- **Test on- and off-throttle.** Rev under load *and* coast down; the edit is only right if both — and the
  transition between them — sound correct.

---

### Key takeaways

- Engine sound has two axes: **RPM** (pitch/timbre) and **load** (accel vs decel).
- Each car ships two GINs — the main (acceleration) and a **`_DCL`** (deceleration); 73 `_DCL` files in the game.
- Accel and decel are acoustically different engines (load vs overrun), so decel gets its own grains.
- The synth crossfades between the two files by throttle/load — a live 2-D synthesis driven by the controls.
- Edit both files together, keep their characters distinct and RPM ranges matched, and test on- and
  off-throttle.

**Continue:** [C22.6 — Editing engine sound](06-editing-gin.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md)
