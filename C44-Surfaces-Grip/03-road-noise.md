# C44.3 — RoadNoiseRecord: the Sound

> **The one-sentence version:** `RoadNoiseRecord` (verified vault key, ×48 in `attributes.bin`) selects the tyre
> roll/skid sample set for the surface — asphalt hums, gravel rattles, grass goes soft — so you can tell what
> you're driving on with your eyes closed.

[← C44.2 — Grip](02-grip.md) · [Chapter 44 hub](C44-Surfaces-Grip.md) ·
[Next: C44.4 — TireEffectRecord →](04-tire-effects.md)

---

## The audio read

The **sound** read of the surface tag is governed by **`RoadNoiseRecord`** — a vault record that maps a surface
(and tyre mode, [C44.4](04-tire-effects.md)) to the tyre audio for that condition. It's heavily used: verified,
its reflection hash `0xFFDB013B` appears **48 times** in `attributes.bin`, one of the most-referenced audio
records — because there's an entry per surface (× a few modes), and surfaces are many
([C44.1](01-surface-taxonomy.md)).

`RoadNoiseRecord` selects the **tyre roll/skid sample set** for the surface:

- **`asphalt`** — a smooth hum, rising with speed.
- **`gravel`** / **`dirt`** — a rattle, loose stones under the tyres.
- **`grass`** — a soft, muffled roll.
- **`concrete`** — a harder, more resonant hum than asphalt (expansion joints, texture).
- **`water`** / **`sand`** — a hiss or a drag.

So the tyre's voice tracks the ground: the same car sounds different on each surface, because `RoadNoiseRecord`
points the audio system ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)) at the right samples for
the tag.

> ✅ *Verified:* `rh("RoadNoiseRecord")=0xFFDB013B` appears **×48** as a vault key in `GLOBAL/attributes.bin` — the
> tyre-audio record per surface (× mode). It is the audio counterpart of `TireEffectRecord`
> ([C44.4](04-tire-effects.md), ×50).

## Roll and skid

`RoadNoiseRecord` covers two regimes of tyre sound, matching the tyre mode
([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)):

- **Roll** — the tyre rolling normally on the surface (the driving mode). A continuous roll sound, pitched/leveled
  by speed — the baseline "driving on X" hum.
- **Skid** — the tyre sliding (skid/slide modes) — the screech of a locked or drifting tyre, which *also* depends
  on the surface: a skid on asphalt shrieks, a slide on gravel scrabbles, a slide on grass barely sounds.

So the record isn't just "the sound of surface X" — it's "the sound of surface X being driven on *this way*." A
hard braking skid and a gentle roll on the same asphalt are different samples, both from the asphalt
`RoadNoiseRecord`. This is why the audio feels responsive: it tracks not just where you are but what your tyres are
doing ([C44.4](04-tire-effects.md)).

## Sound completes the surface

`RoadNoiseRecord` is what makes a surface *audibly* real. Together with grip ([C44.2](02-grip.md)) and effects
([C44.4](04-tire-effects.md)), it's why every surface is a complete sensory experience:

- **You hear the transition.** Driving off asphalt onto grass, the hum drops to a soft roll — an *audible* cue
  that you've left the road, reinforcing the grip loss ([C44.2](02-grip.md)) you feel.
- **You can drive by ear.** In a drift or a night chase, the tyre sound tells you the surface and the slip — you
  can sense a skid starting before you see it.
- **The world sounds textured.** A city of concrete, asphalt, and grass ([C44.1](01-surface-taxonomy.md)) has an
  audio texture that matches its look — coherent because both key off the same tag ([C44.5](05-three-reads.md)).

So the sound read is a full third of the surface experience — not a garnish, but a core feedback channel. That
`RoadNoiseRecord` is referenced 48 times reflects how much care went into making each surface sound like itself
across all its modes.

## Why a separate record for audio

Splitting the surface's sound into its own record (rather than bundling it with grip or effects) follows the
engine's medium-ownership pattern ([C44.5](05-three-reads.md)):

- **Audio designers own it.** `RoadNoiseRecord` is authored by the people who make the sound
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)), independently of the physics (grip) and the
  visuals (effects) teams.
- **It can be retuned alone.** Change how gravel sounds without touching how gravel grips or smokes — the reads
  are independent ([C44.5](05-three-reads.md)).
- **It keys off the shared tag.** Because it uses the same surface tag as grip and effects, the sound stays
  consistent with them as long as all three records are filled in ([C44.5](05-three-reads.md)).

So `RoadNoiseRecord` is the audio slice of the surface fan-out — its own record, its own owners, keyed by the
shared tag. Edit it to restyle a surface's sound; leave the grip and effects to their records.

## RE implications

- **`RoadNoiseRecord` (×48)** is the surface's **audio** read — the tyre roll/skid samples per surface.
- **It covers roll and skid** — keyed by surface *and* tyre mode ([C44.4](04-tire-effects.md)); a skid and a roll
  on the same surface differ.
- **Sound completes the surface** — audible transitions, driving by ear, a textured world.
- **A separate record** for audio — owned by sound designers, retunable alone, keyed by the shared tag.

---

### Key takeaways

- **`RoadNoiseRecord`** (verified vault key, **×48**) is the surface's **audio** read — selecting the tyre
  roll/skid sample set per surface.
- Each surface sounds like itself: **asphalt hums, gravel rattles, grass goes soft, concrete resonates** — you can
  tell the surface by ear.
- It covers **roll and skid** — keyed by surface *and* tyre mode, so a braking skid and a gentle roll on the same
  asphalt are different samples.
- **Sound completes the surface** — you *hear* the transition off the road, can drive by ear, and the world has an
  audio texture matching its look.
- It's a **separate record** owned by audio designers, retunable alone, keyed by the shared surface tag — the
  audio slice of the fan-out.

**Continue:** [C44.4 — TireEffectRecord: the visual](04-tire-effects.md) · [Chapter 44 hub](C44-Surfaces-Grip.md)
