# C22.1 — GIN Is Granular Synthesis

> **The one-sentence version:** a car's engine note can't be one recording because the engine sweeps
> continuously through revs and load — so GIN stores short waveform **grains** and the runtime blends and
> pitches them into a continuous note that tracks the tachometer exactly.

[← Chapter 22 hub](C22-Engine-Sound-GIN.md) · [Next: C22.2 — The Gnsu header →](02-gnsu-header.md)

---

## Why not just record the engine?

Consider what a recording would have to cover: an engine idling, accelerating through every RPM to redline,
holding at part-throttle, decelerating on overrun — in every combination, seamlessly, forever. No finite
recording can do this. Play a fixed clip and it either loops audibly or fails to match the car's actual revs.
The engine note has to be **generated** to follow the simulation.

So GIN uses **granular synthesis**: store many short, characteristic **grains** of engine waveform, and
**assemble** a continuous note from them in real time based on the engine's state ([C22.4](04-rpm-bridge.md)).
The magic `Gnsu` — "granular-synthesis unit" — names exactly this.

## Grains + a rule = a continuous note

The two ingredients of granular synthesis, both in the GIN file:

- **Grains** — short waveform snippets, each characteristic of some part of the rev range
  ([C22.3](03-grains.md)).
- **A mapping rule** — the RPM range ([C22.2](02-gnsu-header.md)) that says which grains correspond to which
  revs, so the synth knows what to play when.

At runtime, the current RPM selects grains, the synth pitch-shifts them to the exact rev, and crossfades
between adjacent grains — producing a note that rises smoothly from idle to redline and back, matching the
tachometer sample-for-sample ([C22.4](04-rpm-bridge.md)). The engine sound is *computed*, not triggered.

## Why granular synthesis suits engines

Engines are the ideal case for granular synthesis, which is why EA (and racing games generally) use it:

- **Continuous parameter (RPM).** An engine's sound is dominated by one continuous variable — revs — that maps
  naturally onto grain selection + pitch. Granular synthesis excels when a continuous control drives the sound.
- **Periodic waveform.** An engine note is quasi-periodic (firing pulses), so short grains capture its
  character and stitch without obvious seams.
- **Compact.** A handful of grains covering a rev range is far smaller than recordings of every RPM, and covers
  *every* RPM including ones never explicitly recorded (by pitching between grains).
- **Reactive.** Because the note is generated from live RPM, it reacts instantly to throttle, gearshifts, and
  load — impossible with fixed clips.

## The result: a synth, not a sampler

The distinction matters for understanding (and editing) engine sound: GIN is a **synthesiser** parameterised by
the file's grains and RPM map, not a **sampler** playing clips. So:

- Changing the *grains* changes the engine's *timbre* (what it sounds like).
- Changing the *RPM map* changes *how the timbre maps to revs* (where it sounds high or low).
- The *runtime bridge* ([C22.4](04-rpm-bridge.md)) is fixed — it always drives the synth from RPM.

This framing — grains + map + bridge — is the mental model for everything in this chapter.

---

### Key takeaways

- No recording can cover an engine's continuous rev/load sweep — the note must be **synthesised**.
- GIN (`Gnsu`) is **granular synthesis**: short waveform **grains** + an **RPM map**, assembled live.
- The runtime selects grains by RPM, pitch-shifts to the exact rev, and crossfades — a note that tracks the
  tachometer.
- Engines suit granular synthesis: one continuous control (RPM), quasi-periodic waveform, compact, reactive.
- GIN is a **synth** (grains + map + bridge), not a sampler — the model for decoding and editing it.

**Continue:** [C22.2 — The Gnsu header](02-gnsu-header.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md)
