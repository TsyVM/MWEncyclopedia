# C43.5 — Smackables

> **The one-sentence version:** the world's knock-over objects — cones, signs, fences — are `RBSmackable` bodies
> tuned by `SmackableParams`, the cheapest physics bodies, sitting inert until a `carhitsmackable` contact wakes
> them to scatter.

[← C43.4 — Reaction records](04-reactions.md) · [Chapter 43 hub](C43-Collision-Contacts.md) ·
[Next: C43.6 — Reading collision in RE →](06-reading-collision.md)

---

## The knock-over objects

Driving through Rockport, you plough through cones, clip signs, and smash fences — and they scatter satisfyingly.
Those are **smackables**: light physics objects that dress the world and react when hit. They're implemented as
**`RBSmackable`** bodies ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) — a rigid-body class in the
physics tree — tuned by **`SmackableParams`**, and the verified string set around them is rich:

- **`RBSmackable`** — the smackable rigid body.
- **`SmackableParams`** — the tuning class (mass, how it scatters).
- **`EffectsSmackable`** — the effects when smacked (debris, dust).
- **`SmackableRenderConn`** — the render connector ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) drawing
  it.
- **`ESpawnSmackable`** — the event that spawns one.

So a smackable is a full little object — body + params + effects + render connector — just a very lightweight one.

> ✅ *Verified:* `RBSmackable`, `SmackableParams`, `EffectsSmackable`, `SmackableRenderConn`, and `ESpawnSmackable`
> are present as strings in `speed.exe`. The `carhitsmackable` classification (`rh=0xA906E973`) is a vault key in
> `attributes.bin` ([C43.3](03-classification.md)).

## Where the smackables come from

This page is the *runtime* half of a smackable — the `RBSmackable` body and its params. The *placement* half is on
disk: each stream section carries a **smackable spawner chunk** (`0x00034027`,
[C63.9](../C63-Collision-World/09-smackables-emitters.md)) — a table of 64-byte records that says *which* prop
(`assetHash`), *where* (a swizzled world position), and with *which* tuning (`paramHash`, a `SmackableParams` vault
key). When a section streams in ([C15.7](../C15-Track-Streaming/07-section-contents.md)), those records spawn the
`RBSmackable` bodies this page describes — so `ESpawnSmackable` is the runtime event, and `0x00034027` is the
on-disk data that drives it.

The two halves interlock through the `paramHash`: the spawner record names a `SmackableParams` set
([C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md)), and the runtime body reads that set to know its
mass and how it scatters. A section's props also carry a **per-section 2048-bit "already smashed" mask**
([C63.9](../C63-Collision-World/09-smackables-emitters.md)) so a smackable you've knocked down stays down — the
runtime state keyed on the on-disk record order. Reading smackables completely means reading *both* halves: the body
here, the spawner data in [C63.9](../C63-Collision-World/09-smackables-emitters.md).

## Inert until smacked

A smackable's defining behaviour is that it's **inert until hit**. Unlike a car (always simulating,
[Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), a smackable sleeps:

- **At rest**, it's a static-looking prop — no per-frame physics, just a drawn object
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)). It costs almost nothing.
- **On contact** (`carhitsmackable`, [C43.3](03-classification.md)), it *wakes* — becomes an active rigid body,
  takes the impact impulse, and reacts (topples, flies, rolls).
- **After settling**, it sleeps again (or despawns) — back to costing nothing.

This sleep/wake pattern is why a level can have *thousands* of smackables (every cone, bollard, and sign) without
a physics cost — they only simulate in the instant they're struck. It's the physics equivalent of the render cull
tree ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)): pay only for what's active. The
`RBSmackable` being the *cheapest* body in the tree ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) is
what makes this affordable.

> 🟡 *Reasoned:* the sleep-until-hit lifecycle is the standard smackable/breakable-prop design, consistent with
> the verified `RBSmackable` class, the `ESpawnSmackable` event, and the `carhitsmackable` tag; the exact wake
> trigger and sleep threshold are deeper RE. The class/param/event strings and the tag hash are verified.

## Why smackables are their own class

Making smackables a distinct rigid-body class (`RBSmackable`) rather than reusing `SimpleRigidBody`
([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)) reflects their special lifecycle and role:

- **The sleep/wake behaviour** is specific — a smackable needs the "inert until hit, then react, then settle"
  logic, distinct from a body that's always simulating.
- **The spawn/despawn** (`ESpawnSmackable`) ties them to the world streaming
  ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — smackables come and go with the sections around
  the player, replenishing as you drive.
- **The presentation** (`EffectsSmackable`, `SmackableRenderConn`) is their own — the debris and the (often
  simple) render path, separate from a car's.

So `RBSmackable` is the class that makes the world feel *destructible on a budget*: a dedicated, cheap, sleep-until-hit
body with its own spawn and effects, deployed in bulk to dress the streets. They're a small but characteristic
piece of Most Wanted's feel — the game *wants* you to smash through them, and the physics is built to let you, at
scale.

## Smackables in the fan-out

When you hit a smackable, the collision fan-out ([C43.3](03-classification.md)) plays out in its mildest form:

- **Reaction** ([C43.4](04-reactions.md)) — minimal on the *car* (a cone barely slows you); the *smackable* takes
  the impulse and scatters.
- **Damage** ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)) — typically none to the car (a
  cone doesn't dent you) — one reason `carhitsmackable` is distinct from `carhitwall`
  ([C43.3](03-classification.md)).
- **Presentation** — `EffectsSmackable` debris + a light impact sound.

So the smackable case is the "harmless spectacle" end of the collision spectrum: lots of visual payoff, negligible
cost to the car. Contrast `carhitwall` (big reaction, big damage) — the classification ([C43.3](03-classification.md))
is exactly what routes a cone-hit to "scatter, no harm" and a wall-hit to "stop, damage." Same machinery, opposite
ends.

## RE implications

- **Smackables are `RBSmackable` bodies** with `SmackableParams` tuning, `EffectsSmackable`, and
  `SmackableRenderConn` — verified strings.
- **They're inert until hit** — sleep/wake, so thousands can dress a level at near-zero cost.
- **Their own class** for the sleep/wake lifecycle, spawn/despawn with streaming, and light presentation.
- **The mildest fan-out** — `carhitsmackable` → scatter, no car damage — the harmless-spectacle end of collision.

---

### Key takeaways

- The world's **knock-over objects** are **`RBSmackable`** bodies (`SmackableParams`, `EffectsSmackable`,
  `SmackableRenderConn`, `ESpawnSmackable`) — verified strings, the **cheapest** physics bodies.
- They're **inert until smacked** — sleep at rest, wake on a `carhitsmackable` contact, react, settle — so
  thousands can dress a level at near-zero cost.
- They're their **own class** for the sleep/wake lifecycle, streaming-tied spawn/despawn, and light presentation.
- Hitting one is the **mildest collision fan-out** — scatter and debris, negligible car reaction or damage — the
  opposite end from `carhitwall`.
- Smackables are a small but characteristic piece of MW's feel — the game is **built to let you smash through
  them at scale**.

**Continue:** [C43.6 — Reading collision in RE](06-reading-collision.md) · [Chapter 43 hub](C43-Collision-Contacts.md)
