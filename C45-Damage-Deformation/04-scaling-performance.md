# C45.4 — Scaling & Performance Loss

> **The one-sentence version:** how much a contact hurts is set by `DamageScaleRecord` (verified vault key, ×24) —
> it scales the collision force into zone damage per collision type — and enough damage degrades performance,
> feeding back into the engine and handling; this is *how the car is hurt*, distinct from *how it moves*.

[← C45.3 — Deformation & breakables](03-deformation.md) · [Chapter 45 hub](C45-Damage-Deformation.md) ·
[Next: C45.5 — Cop damage & the bust →](05-cop-damage.md)

---

## DamageScaleRecord: force to damage

A collision produces a **force** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)); how much of
that force becomes **damage** is governed by **`DamageScaleRecord`** — a verified vault record whose reflection
hash `0xD99B853C` appears **×24** in `attributes.bin`. It's the scaling from impact to harm:

- **Per collision type.** A `carhitwall` ([C43.3](../C43-Collision-Contacts/03-classification.md)) scales
  differently than a `carhitcar` — hitting a wall at speed hurts more than trading paint. The ×24 references cover
  the different collision contexts.
- **Into the zone.** The scaled damage is added to the coarse zone ([C45.2](02-damage-zones.md)) in the impact
  area, driving crumple ([C45.3](03-deformation.md)) and breakage.
- **Tunable.** Because it's vault data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), designers
  tune how fragile or tough cars are without touching the damage code.

So `DamageScaleRecord` is the dial between "a big hit" and "a lot of damage" — the tuning that decides whether the
game is arcade-forgiving (little damage per hit) or punishing (lots).

> ✅ *Verified:* `rh("DamageScaleRecord")=0xD99B853C` appears **×24** as a vault key in `GLOBAL/attributes.bin` —
> the collision-force → zone-damage scaling record, per collision type.

## Hurt vs. move: two separate records

A crucial architectural point ([C43.4](../C43-Collision-Contacts/04-reactions.md)): the damage a contact does and
the *motion* it imparts are **two separate records**:

| Record | Hash | Governs |
|---|---|---|
| `CollisionReactionRecord` | `0x63E3B021` (×35) | how the car **moves** — push/slow/spin |
| `DamageScaleRecord` | `0xD99B853C` (×24) | how the car is **hurt** — zone damage |

These are deliberately decoupled ([C43.3](../C43-Collision-Contacts/03-classification.md)). A collision can shove
the car a lot but damage it little (a glancing high-speed brush), or damage it a lot but move it little (a hard hit
into an immovable wall). Splitting the two records lets designers tune crash *feel* (reaction) and crash *cost*
(damage) independently — the same contact, two knobs. This is why Most Wanted can make crashes feel dramatic
without necessarily destroying the car, or vice versa.

## Performance loss

Beyond looks ([C45.3](03-deformation.md)), enough accumulated damage causes **performance loss** — the car
mechanically drives worse, feeding back into the other mechanics
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)):

- **Engine damage → power loss.** A wrecked front end (where the engine sits) degrades the engine mechanic
  ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) — less power, worse acceleration.
- **Suspension/handling damage → worse handling.** Damage to the running gear degrades the suspension
  ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)) — the car pulls, handles loosely.
- **Tyre damage → grip loss.** Blown/punctured tyres ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md))
  cut grip — the spike-strip effect ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).

This is the DAMAGE mechanic's **feedback** into the sim ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md)) —
the one output mechanic that loops back and changes the physics. A car late in a hard pursuit
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) may be slower and looser than it started, because damage
has degraded its engine, suspension, and tyres. Damage is thus not just cosmetic — it's a *mechanical* cost that
mounts over a chase.

> 🟡 *Reasoned:* the performance-degradation feedback (damage → reduced engine/handling/grip) is the standard
> consequential-damage model, consistent with the DAMAGE mechanic's role ([C40.6](../C40-Eight-Mechanics/06-damage-draw-sound.md))
> and the verified damage classes/records; the exact degradation curves are vault tunables. The `DamageScaleRecord`
> ×24 and the damage classes are verified.

## Damage as a pursuit resource

The performance-loss loop makes damage a **resource** in the pursuit game
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)):

- **The player manages damage.** Taking hits (from cops, traffic, walls) degrades your car — so evading cleanly
  matters, and a long pursuit wears you down.
- **The cops inflict damage.** Ramming and PIT manoeuvres ([C43.4](../C43-Collision-Contacts/04-reactions.md))
  aren't just about position — they *damage* you, slowly crippling your escape.
- **Damage is (mostly) recoverable.** Between events, or via a pursuit breaker / reset
  ([ResetCar](../C42-Suspension-Tyres-Drivetrain/01-fidelity-tiers.md)), damage can be undone — so it's a
  per-event pressure, not permanent.

So damage ties the physics ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) to the pursuit stakes
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)): every collision is both a physics event and a chip at your
car's ability to escape. This is what makes cop contact *threatening* rather than merely annoying — it has a
mechanical, mounting cost, scaled by `DamageScaleRecord`.

## RE implications

- **`DamageScaleRecord` (×24)** scales collision force → zone damage, per collision type — the fragility dial.
- **Hurt vs. move are separate records** — `DamageScaleRecord` (damage) vs. `CollisionReactionRecord` (motion) —
  independently tunable.
- **Performance loss** — enough damage degrades engine/handling/grip, feeding back into the sim (the DAMAGE
  mechanic's feedback).
- **Damage is a pursuit resource** — cops damage you to cripple your escape; damage is mostly recoverable per
  event.

---

### Key takeaways

- **`DamageScaleRecord`** (verified vault key, **×24**) scales a collision's force into **zone damage**, per
  collision type — the dial between "a big hit" and "a lot of damage."
- **Hurt and move are two separate records** — `DamageScaleRecord` (how the car is hurt) vs.
  `CollisionReactionRecord` (how it moves) — decoupled so crash cost and crash feel tune independently.
- Enough damage causes **performance loss** — degraded engine (power), suspension (handling), and tyres (grip) —
  the DAMAGE mechanic's **feedback** into the sim.
- Damage is a **pursuit resource** — cops ram and PIT to *damage* you, crippling your escape; a long chase wears
  the car down.
- Damage is **mostly recoverable** per event (reset/breakers) — a mounting per-event pressure, not permanent.

**Continue:** [C45.5 — Cop damage & the bust](05-cop-damage.md) · [Chapter 45 hub](C45-Damage-Deformation.md)
