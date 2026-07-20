# C44.5 — The Three Synchronized Reads

> **The one-sentence version:** one surface tag drives three synchronized reads — grip (feel), `RoadNoiseRecord`
> (sound), and `TireEffectRecord` (look) — so a `sand` tag picks low grip, sandy tyre noise, and `fxtd_dr_sand`
> together, and every surface is coherent across all three senses.

[← C44.4 — TireEffectRecord](04-tire-effects.md) · [Chapter 44 hub](C44-Surfaces-Grip.md) ·
[Next: C44.6 — Reading surfaces in RE →](06-reading-surfaces.md)

---

## One tag, three reads

The organising idea of the whole surface system is that **one surface tag** ([C44.1](01-surface-taxonomy.md)) is
read **three ways at once**, continuously, every frame you're driving:

```
tyre on surface S, doing mode M
        │
        ├──▶ grip            (tyre model, C44.2)        →  hold vs. slide       [FEEL]
        ├──▶ RoadNoiseRecord (audio, C44.3, ×48)        →  roll/skid sound      [SOUND]
        └──▶ TireEffectRecord → fxtd_<M>_<S> (C44.4,×50)→  smoke/debris         [LOOK]
```

All three branch on the *same tag* (and, for sound and effects, the tyre mode
[C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)). So a `sand` surface simultaneously gives low grip,
sandy tyre noise, and a `fxtd_dr_sand` dust plume — the surface is *coherent* across feel, sound, and look. You
feel the slide, hear the hiss, and see the dust, all describing the same sand.

> ✅ *Verified:* the three reads are anchored on verified vault records — the surface tags
> ([C44.1](01-surface-taxonomy.md)), `RoadNoiseRecord` (×48, [C44.3](03-road-noise.md)), and `TireEffectRecord`
> (×50, [C44.4](04-tire-effects.md)) — all reflection-hash keys in `attributes.bin`. The grip coefficient is a
> per-surface tunable ([C44.2](02-grip.md)).

## Why coherence matters

That all three key off the same tag is what makes surfaces *believable*. Coherence — feel, sound, and look
agreeing — is the difference between a convincing surface and a broken one:

- **Agreement builds belief.** When grass grips softly, sounds soft, and throws up turf, your brain accepts it as
  grass. The three senses corroborate each other.
- **Disagreement breaks immersion.** A surface that *looks* like asphalt but *grips* like ice (or smokes like
  dirt) is the classic "feels wrong" bug — the senses contradict, and the illusion collapses.
- **Learnability.** Because a surface is always the same across all three reads, you *learn* it — you know grass is
  slippery (feel), and the soft sound (audio) and turf (visual) become reliable cues for that grip. The world
  teaches you its materials consistently.

So the single-tag design isn't just tidy engineering — it's what lets Rockport's surfaces feel real and be learned.
The tag is the guarantee that a surface behaves the same way to every sense, everywhere it appears.

## The continuous fan-out mirrors collision

This chapter's three-read fan-out is the **continuous** twin of the collision fan-out
([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)):

| | Collision ([Ch 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) | Surface (this chapter) |
|---|---|---|
| **Trigger** | a discrete contact | a continuous condition |
| **Classification** | `carhit*`/`carscrape*` tag | surface tag (`asphalt`, `grass`…) |
| **Physics read** | `CollisionReactionRecord` | grip |
| **Sound read** | `carhit*` sound | `RoadNoiseRecord` |
| **Visual read** | impact effect | `TireEffectRecord` |

Both take a **classification** and fan it out to **physics + sound + visuals** through independent records. Hitting
the world (discrete) and driving on it (continuous) are the same design idea applied twice
([C43.6](../C43-Collision-Contacts/06-reading-collision.md)). Recognising this shared shape is the key to the whole
"touching the world" system: classify, then fan out to three independent, tunable reads. It recurs because it
*works* — it keeps physics, audio, and visuals coherent while letting each be authored by its own team.

## Medium ownership

The three-read split embodies **medium ownership** — each read is owned and tuned by the team that owns that
medium ([C44.3](03-road-noise.md)):

- **Physics team** owns grip (the per-surface coefficients, [C44.2](02-grip.md)).
- **Audio team** owns `RoadNoiseRecord` (the tyre samples, [C44.3](03-road-noise.md)).
- **Effects/VFX team** owns `TireEffectRecord` (the `fxtd_*` particles, [C44.4](04-tire-effects.md)).

Each authors their record independently, all keyed by the shared surface tag. This is why a big team can build a
rich, coherent world: the tag is the *contract* between the disciplines, and each fills in its own record. To add a
new surface, all three fill in an entry ([C44.6](06-reading-surfaces.md)); to retune one aspect (say, make gravel
smoke more), only that team touches only its record. The single tag coordinates the three mediums without coupling
them.

## RE implications

- **One surface tag, three synchronized reads** — grip (feel), `RoadNoiseRecord` (sound), `TireEffectRecord`
  (look) — all keyed by the tag.
- **Coherence** across the three is what makes surfaces believable and learnable; disagreement is the "feels
  wrong" bug.
- **The continuous fan-out mirrors collision** — classify, then physics + sound + visuals through independent
  records.
- **Medium ownership** — each read owned by its discipline's team, coordinated by the shared tag.

---

### Key takeaways

- **One surface tag drives three synchronized reads**: **grip** (feel), **`RoadNoiseRecord`** (sound, ×48),
  **`TireEffectRecord`** (look, ×50) — a `sand` tag gives low grip + sandy noise + `fxtd_dr_sand` together.
- **Coherence** across feel, sound, and look makes a surface **believable and learnable**; contradiction is the
  classic "feels wrong" bug.
- The three-read fan-out is the **continuous twin of the collision fan-out** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md))
  — classify, then physics + sound + visuals through independent records.
- The design embodies **medium ownership** — physics, audio, and VFX teams each own one read, coordinated by the
  shared tag (a contract without coupling).
- This shared shape — **classify, then fan out to three tunable reads** — is the key to Most Wanted's whole
  "touching the world" system.

**Continue:** [C44.6 — Reading surfaces in RE](06-reading-surfaces.md) · [Chapter 44 hub](C44-Surfaces-Grip.md)
