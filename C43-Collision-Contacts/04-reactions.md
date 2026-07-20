# C43.4 — Reaction Records

> **The one-sentence version:** the physical response to a contact is data-driven by two verified vault records —
> `CollisionReactionRecord` (×35 in `attributes.bin`, the player/generic car's push-back/slow-down/spin) and
> `AICollisionReactionRecord` (×14, the AI car's response) — split so a crash feels right for the player and
> behaves right for the cop AI.

[← C43.3 — Classification](03-classification.md) · [Chapter 43 hub](C43-Collision-Contacts.md) ·
[Next: C43.5 — Smackables →](05-smackables.md)

---

## Two reaction records

Once a contact is classified ([C43.3](03-classification.md)), the **physical response** — how the car actually
moves in reaction — is chosen from a vault record. There are **two**, and which one applies depends on who's
driving:

| Record | Hash | Occurrences | Governs |
|---|---|---|---|
| `CollisionReactionRecord` | `0x63E3B021` | **35** | the player / generic car's response |
| `AICollisionReactionRecord` | `0xAA229CD7` | **14** | the AI car's response (cops, traffic, racers) |

Both are heavily used — `CollisionReactionRecord` appears **35 times** in `attributes.bin`, one of the
most-referenced vault records in the physics data. This reflects how central crash response is to the driving
feel: there are many reaction tunings (per collision type, per context), and they're referenced throughout the
car data.

> ✅ *Verified:* `rh("CollisionReactionRecord")=0x63E3B021` appears **×35** and
> `rh("AICollisionReactionRecord")=0xAA229CD7` appears **×14** as vault keys in `GLOBAL/attributes.bin` — the
> data-driven collision responses for player and AI.

## What a reaction record governs

A reaction record is the tuning for **how a contact shoves the car** — turning the contact's force and normal
([C43.2](02-contact-records.md)) into the car's motion change:

- **Push-back.** How hard the collision pushes the car away from what it hit — a wall bounce, a car-to-car shove.
- **Slow-down.** How much speed the collision scrubs — a head-on wall hit kills speed; a glancing brush barely
  does.
- **Spin.** How much the collision rotates the car — an off-centre hit yaws it; enough spins it out.

These are the "how dramatic is the crash" knobs ([C43.3](03-classification.md)) — the record scales the raw
contact impulse into the *felt* reaction. A more forgiving record lets you bounce off walls and keep going; a
harsher one punishes contact with big speed loss and spin. The record is where crash *feel* is authored,
separately from crash *damage* ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)) — how the car
*moves* vs. how it's *hurt* are two different records.

> 🟡 *Reasoned:* the specific fields of the reaction records (impulse scale, speed-loss, spin factor, clamps) are
> the natural parameters of a data-driven collision response, consistent with the verified records and observed
> crash behaviour; the exact field layout isn't byte-dumped here. The records' presence and heavy use are
> verified.

## Why split player and AI

Splitting the response into a player record and an AI record ([C43.3](03-classification.md)) is a deliberate,
important design choice:

- **The crash must feel right from the player's seat.** `CollisionReactionRecord` tunes the *player's* experience
  — how it feels to clip a wall or trade paint. This is about game feel, tuned for the human at the controls.
- **The cop AI needs predictable collision behaviour.** `AICollisionReactionRecord` tunes how *AI cars* respond —
  and the cops' pursuit tactics ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) depend on it. A **PIT
  manoeuvre** (a cop tapping your rear quarter to spin you) only works if the cop's own collision response is
  controlled and predictable — otherwise the cop would spin out too, or bounce off unpredictably.
- **Different roles, different feel.** A cop ramming you should behave differently than you ramming a wall; a
  traffic car you clip should react as a traffic car, not as the player would. Separate records let each role feel
  and behave correctly.

So the player/AI split is what lets Most Wanted's crashes serve two masters at once: satisfying game feel for the
player, and reliable tactical behaviour for the cop AI. The same collision, two reaction tunings, chosen by who's
in the seat ([C43.3](03-classification.md)).

## The reaction is one of three reads

The reaction is only the *physics* consequence of a contact — one of three ([C43.3](03-classification.md)):

- **Reaction** (this page) — `CollisionReactionRecord`/`AICollisionReactionRecord` → the car's motion change.
- **Damage** ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)) — the `Damage*` mechanic → zone
  damage. Separate record: how the car *moves* vs. how it's *hurt*
  ([C43.3](03-classification.md)).
- **Presentation** — effects + sound → the sparks and the crunch.

All three read the same contact ([C43.2](02-contact-records.md)), branching on the same tag
([C43.3](03-classification.md)), but through *independent* records — so crash feel, crash damage, and crash
spectacle can each be tuned without disturbing the others (the one-way boundary,
[C39.5](../C39-Vehicle-Simulation/05-connectors.md), again: presentation can't perturb the reaction). The reaction
record is the physics slice of that fan-out.

## RE implications

- **Two reaction records** — `CollisionReactionRecord` (×35, player/generic) and `AICollisionReactionRecord` (×14,
  AI) — verified vault keys.
- **They govern how a contact shoves the car** — push-back, slow-down, spin — the "how dramatic" knobs.
- **Player/AI split** lets crashes feel right for the player and behave predictably for the cop AI (PIT
  manoeuvres).
- **Reaction is one of three reads** — separate from damage and presentation; independent tuning of the same
  contact.

---

### Key takeaways

- A contact's **physical response** is a vault record — **`CollisionReactionRecord`** (×35, player/generic) or
  **`AICollisionReactionRecord`** (×14, AI) — both **verified**, heavily used.
- They govern **push-back, slow-down, and spin** — scaling the raw contact impulse into the *felt* reaction (crash
  feel), separately from **damage** (how the car is hurt).
- The **player/AI split** is deliberate: the crash must **feel right** for the player and **behave predictably**
  for the cop AI (whose PIT manoeuvres rely on controlled collision response).
- The reaction is the **physics slice** of the three-way fan-out (reaction, damage, presentation) — all reading
  one contact, through independent records.
- `CollisionReactionRecord`'s **×35** references make it one of the most central physics vault records — crash
  response is core to the driving feel.

**Continue:** [C43.5 — Smackables](05-smackables.md) · [Chapter 43 hub](C43-Collision-Contacts.md)
