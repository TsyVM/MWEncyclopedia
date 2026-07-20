# C40.5 — SUSPENSION: Loads & Weight Transfer

> **The one-sentence version:** `BEHAVIOR_MECHANIC_SUSPENSION` models the spring/damper at each wheel — computing
> the load on each tyre and the weight transfer under acceleration, braking, and cornering — which is what gives
> a car its handling character.

[← C40.4 — ENGINE](04-engine.md) · [Chapter 40 hub](C40-Eight-Mechanics.md) ·
[Next: C40.6 — DAMAGE, DRAW & SOUND →](06-damage-draw-sound.md)

---

## The wheel loads

`BEHAVIOR_MECHANIC_SUSPENSION` is the **grip** mechanic — not because it produces grip directly (the tyres do,
[Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)), but because it computes the
**load on each tyre**, and tyre grip scales with load. Each wheel ([C39.3](../C39-Vehicle-Simulation/03-part-array.md))
has a suspension — a spring and damper — that compresses against the ground:

- **Spring force.** The more the suspension compresses, the harder the spring pushes back — holding the car up and
  setting the wheel's vertical load.
- **Damper force.** The faster the suspension moves, the more the damper resists — controlling how quickly load
  changes (the car's "settling").
- **The normal force** at each wheel is the sum — the load that presses the tyre onto the road, and thus how much
  grip that tyre can generate ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)).

So the suspension mechanic's output is four wheel loads, each frame — the foundation of the car's grip
distribution.

## Weight transfer

The crucial behaviour the suspension produces is **weight transfer** — the redistribution of load between wheels
as the car accelerates, brakes, and corners:

- **Braking → forward transfer.** Deceleration shifts load onto the front wheels (they compress, the rear
  extends) — more front grip, less rear.
- **Acceleration → rearward transfer.** The opposite — load shifts rearward, helping a rear-drive car put power
  down.
- **Cornering → lateral transfer.** Load shifts onto the outside wheels — which is why a car leans (rolls) in a
  turn, and why the outside tyres do most of the cornering work.

Weight transfer is what makes the car feel like it has *mass* and *balance*. It's the reason trail-braking works
(braking into a corner loads the front for turn-in), the reason a car can spin (too much rear weight transfer
under power breaks the rear tyres loose), and the reason suspension tuning matters. The suspension mechanic
computes it every frame from the body's acceleration and the geometry.

> 🟡 *Reasoned:* the spring/damper wheel-load and weight-transfer model is the standard vehicle-suspension
> simulation, consistent with the verified per-wheel part array ([C39.3](../C39-Vehicle-Simulation/03-part-array.md))
> and the suspension mechanic's presence in `attributes.bin` ([C40.1](01-the-mechanic-model.md)); the exact
> spring/damper parameters are per-car vault data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).

## Handling character

The suspension mechanic's parameters ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) define a
car's **handling character**:

- **Spring stiffness** — stiff springs = flat, responsive, less weight transfer (a race car); soft springs =
  more roll, more transfer, more forgiving (a cruiser).
- **Damper rates** — how quickly the car settles; too soft floats, too stiff skips over bumps.
- **Ride height and geometry** — where the mass sits, how much it rolls.
- **Front/rear balance** — stiffer front vs. rear shifts the car toward understeer or oversteer.

Tuning these is what a suspension upgrade does, and it's how two cars with identical engines can feel completely
different to drive — one darty and nervous, one planted and smooth. The suspension mechanic is the *handling*
half of a car's identity, as the engine ([C40.4](04-engine.md)) is the *performance* half.

## Suspension feeds the tyres

The suspension doesn't act alone — it feeds the **tyres** ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)):
the wheel load the suspension computes is the tyre's grip budget, and the tyre turns that budget (plus slip)
into the actual lateral and longitudinal forces on the body. So the chain is *suspension (load) → tyre (force) →
rigid body (motion)*. This is why the suspension is the grip mechanic even though it produces no force on the
road itself: it sets the conditions under which the tyres grip. Understanding a car's handling means
understanding this suspension-tyre pair together ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).

## RE implications

- **`BEHAVIOR_MECHANIC_SUSPENSION`** computes each wheel's **load** (spring + damper) — the tyre's grip budget.
- **It produces weight transfer** — forward (braking), rearward (accel), lateral (cornering) — the car's balance.
- **Its parameters make handling character** — spring/damper rates, ride height, front/rear balance.
- **It feeds the tyres** — load → tyre force → motion; the grip mechanic sets the conditions the tyres grip
  under.

---

### Key takeaways

- `BEHAVIOR_MECHANIC_SUSPENSION` models each wheel's **spring + damper**, computing the **load** on each tyre —
  the grip budget.
- Its central behaviour is **weight transfer** — load shifting forward (braking), rearward (accel), and laterally
  (cornering) — which gives the car mass and balance.
- Its **parameters** (stiffness, damping, ride height, front/rear balance) define a car's **handling character**
  — darty vs. planted.
- The suspension is the **handling** half of a car's identity, as the engine is the **performance** half.
- It **feeds the tyres**: load → tyre force → motion — understand the suspension-tyre pair together
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).

**Continue:** [C40.6 — DAMAGE, DRAW & SOUND](06-damage-draw-sound.md) · [Chapter 40 hub](C40-Eight-Mechanics.md)
