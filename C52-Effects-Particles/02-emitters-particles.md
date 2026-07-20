# C52.2 — Emitters & Particles

> **The one-sentence version:** the particle system is an emitter→particle pipeline, all pooled — an emitter (from
> `EmitterSlotPool`) at a position (`EmitterPositionSlotPool`) spawns particles (from `ParticleSlotPool`) that are
> drawn by `ParticlesShader`, gated by `ParticleSystemEnable`.

[← C52.1 — The two effect worlds](01-two-worlds.md) · [Chapter 52 hub](C52-Effects-Particles.md) ·
[Next: C52.3 — The FX catalogue →](03-fx-catalogue.md)

---

## The emitter → particle pipeline

Scene particles ([C52.1](01-two-worlds.md)) are produced by a two-level system — **emitters** that spawn
**particles**:

- **An emitter** is the *source* — a thing that spawns particles over time at a rate, with initial properties
  (velocity, size, colour, lifetime). A tyre's smoke emitter, an impact's spark emitter.
- **A particle** is a *single sprite* — one puff of smoke, one spark — with a position, velocity, age, and
  appearance, spawned by an emitter and simulated until it dies.
- **An emitter group** bundles related emitters — a complete effect (a "drift smoke" effect might be several
  emitters: the smoke, the tyre marks, the heat shimmer).

So the hierarchy is *emitter group → emitters → particles*: an effect is a group of emitters, each spawning a
stream of particles. This maps to the verified pools ([below](#everything-is-pooled)).

> ✅ *Verified:* the particle system's pools are `EmitterSlotPool`, `EmitterGroupSlotPool`,
> `EmitterPositionSlotPool`, and `ParticleSlotPool`; `ParticlesShader` draws the particles and
> `ParticleSystemEnable` gates the system — all present in `speed.exe`.

## Everything is pooled

The particle system is **thoroughly pooled** ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) —
*four* verified slot pools:

| Pool | Holds |
|---|---|
| `EmitterGroupSlotPool` | the effect instances (groups of emitters) |
| `EmitterSlotPool` | the individual emitters |
| `EmitterPositionSlotPool` | the emitters' world positions |
| `ParticleSlotPool` | the individual particles |

Why so much pooling? Because particles are the *most churning* objects in the game — every drift spawns hundreds of
smoke particles that live a second and die; every impact throws sparks; every dirt road kicks dust. If each particle
were heap-allocated, the allocator would thrash constantly ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)).
Pooling gives *fixed slots*: a particle is born by claiming a free slot and dies by releasing it — no allocation, no
fragmentation, bounded memory. The four-level pooling (groups, emitters, positions, particles) means *every* level
of the system is slot-allocated. This is the pool allocator pattern
([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) at its most necessary — high-churn objects demand
it.

## The particle lifecycle

Each particle runs a short lifecycle each frame it's alive:

```
spawn (claim a ParticleSlotPool slot) — set position, velocity, size, colour, lifetime
   ↓ each frame:
update — age += dt; position += velocity·dt; fade size/alpha over lifetime
   ↓ when age ≥ lifetime:
die (release the slot)
```

So a particle is born at the emitter, drifts and fades over its short life, and returns its slot. The *emitter*
controls the *distribution* — how many particles, how fast, in what cone, with what variation — so a "smoke"
emitter and a "spark" emitter differ in their spawn parameters, producing soft rising smoke vs. fast falling
sparks from the same particle machinery. This is data-over-code again ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)):
one particle system, parameterised by emitter data into every visual effect.

> 🟡 *Reasoned:* the spawn→update→die particle lifecycle and the emitter-controls-distribution model are the
> standard particle-system design, consistent with the verified four-pool structure and `ParticlesShader`; the
> exact particle attributes and update math are per-effect data. The pools and shader are verified.

## ParticlesShader: drawing many sprites

The particles are drawn by **`ParticlesShader`** — a shader specialised for rendering *many small sprites*
efficiently ([C51.3](../C51-Render-Pipeline/03-effect-system.md)):

- **Billboarding** — each particle is a camera-facing quad (a sprite), so smoke and sparks always face the viewer.
- **Additive/alpha blending** — sparks and fire blend *additively* (brightening), smoke blends with *alpha*
  (translucent) — the shader handles the blend modes that make each effect read right.
- **Soft particles / depth** — particles fade where they intersect geometry (so smoke doesn't hard-edge into the
  ground), using the depth buffer.

So `ParticlesShader` is the render path for the particle world — batching thousands of sprites into efficient draws
([C51.5](../C51-Render-Pipeline/05-render-frame.md)), with the blending that distinguishes smoke from sparks from
dust. It's the counterpart, for particles, of the material effects ([C51.3](../C51-Render-Pipeline/03-effect-system.md))
for surfaces — one specialised shader for the whole particle world.

## RE implications

- **Emitter → particle pipeline** — emitter groups spawn emitters spawn particles; drawn by `ParticlesShader`.
- **Four-level pooling** — groups, emitters, positions, particles all slot-allocated (high-churn objects demand
  it).
- **Particle lifecycle** — spawn → update (age/move/fade) → die; the emitter's data controls the distribution.
- **`ParticlesShader`** draws many billboarded, blended sprites efficiently.

---

### Key takeaways

- Scene particles come from an **emitter → particle pipeline** — emitter *groups* hold emitters, emitters spawn
  particles, `ParticlesShader` draws them (`ParticleSystemEnable` gates it).
- The system is **four-level pooled** (`EmitterGroupSlotPool`/`EmitterSlotPool`/`EmitterPositionSlotPool`/
  `ParticleSlotPool`) — because particles are the **most churning** objects in the game (every drift, spark, and
  dust cloud).
- Each particle runs a short **spawn → update (age/move/fade) → die** lifecycle, returning its slot — no allocation
  churn.
- The **emitter's data controls the distribution** — one particle system, parameterised into smoke vs. sparks vs.
  dust (data-over-code).
- **`ParticlesShader`** billboards and blends thousands of sprites efficiently — the particle world's specialised
  render path.

**Continue:** [C52.3 — The FX catalogue](03-fx-catalogue.md) · [Chapter 52 hub](C52-Effects-Particles.md)
