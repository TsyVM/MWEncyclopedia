# C13.4 — Tuning Value → Simulation Knob

> **The one-sentence version:** a stored `Float` only becomes behaviour through the solver that reads it —
> power-type fields scale acceleration, grip-type fields scale cornering force, gearing/top-speed fields cap
> velocity — so predicting a change means knowing which knob a field turns.

[← C13.3 — Reading a car's performance](03-reading-performance.md) · [Chapter 13 hub](C13-Vault-CarTuning.md) ·
[Next: C13.5 — The performance bars →](05-performance-bars.md)

---

## A value is an input to a solver

The vault stores numbers; the driving *feel* comes from the physics solvers that consume them
([C13.2](02-behavior-classes.md)). The `EngineRacer` fields feed the powertrain solver; the `SuspensionRacer`
fields feed the tire/chassis solver. So to reason about a change you ask: **which solver reads this field, and
how does it use it?** The categories below group the tuning knobs by the behaviour they produce.

## The main knob categories

- **Powertrain → acceleration.** Fields governing engine force/power and its delivery scale how quickly the
  car gains speed. Increasing them makes the car pull harder; the effect is strongest where the power curve is
  richest. These live in the engine behavior.
- **Gearing / drivetrain → top speed & the accel–topspeed trade.** Gear-ratio and final-drive fields set how
  engine speed maps to road speed. Taller gearing raises top speed but softens acceleration; shorter gearing
  does the reverse. This is why two cars with equal power can top out differently.
- **Grip / tire → cornering.** Suspension/tire fields scale the lateral force the car can generate, i.e. how
  hard it can corner before sliding. Raising grip tightens handling; lowering it induces slides.
- **Mass / weight transfer → responsiveness & stability.** Mass and its distribution affect how quickly the
  car changes direction and how load shifts under braking/acceleration — the difference between darty and
  planted.
- **Braking → deceleration.** Brake-force fields set stopping power and brake balance.
- **Nitrous / boost → burst acceleration.** NOS fields add a temporary force multiplier (the `GAME_ACTION_NOS`
  binding triggers it; `carexhaustnos` is its effect).

> 🟡 *Reasoned:* these category→behaviour mappings describe how a racing sim of MW's design consumes such
> fields and match the observed field families (engine, suspension, gearing, NOS). The ✅ verified facts are
> that these behavior collections exist, inherit from `default`, and carry `Float` fields; the precise solver
> equations are internal to the physics code and not reproduced byte-for-byte here.

## Why equal numbers feel different

Because behaviour is the *product* of several knobs, a single value rarely tells the whole story:

- **Power without gearing is capped.** Raising engine power with unchanged gearing improves acceleration but
  not top speed — the car hits its gear-limited ceiling sooner and harder.
- **Grip without mass control is nervous.** More grip with a twitchy mass setup makes a car that corners hard
  but is hard to place.
- **Top speed without power is theoretical.** A tall top-speed setting the engine can't reach is just a number.

So tuning is about *balance* across knobs, which is exactly why MW summarises the mess into three bars
([C13.5](05-performance-bars.md)) rather than exposing every field.

## Predicting a change

A practical way to reason before testing:

1. **Identify the knob category** the field belongs to (power, gearing, grip, mass, brake, NOS).
2. **Predict the first-order effect** (more power → more accel).
3. **Predict the trade** (taller gearing → more top speed, less accel).
4. **Check the balance** — does another knob cap or amplify the change?

This turns editing from trial-and-error into hypothesis-and-test, and it is why understanding the
value→simulation mapping ([C13.3](03-reading-performance.md) for the values, this page for the mapping) is
worth more than a table of magic numbers.

## Editing implications

- **Edit the knob that matches your goal.** Want more top speed? Gearing/top-speed fields, not raw power.
  Want quicker corners? Grip, not power.
- **Expect trades.** Acceleration and top speed pull against each other through gearing; account for it.
- **Change one category at a time** when tuning, so you can attribute the effect — the same discipline as any
  experiment.

---

### Key takeaways

- Stored `Float`s become behaviour only through the solver that reads them; know which knob a field turns.
- Knob categories: powertrain→accel, gearing→top speed (and the accel trade), grip→cornering, mass→response,
  braking→decel, NOS→burst.
- Behaviour is the product of several knobs, so equal single values can feel very different.
- Predict a change by category, first-order effect, trade, and balance — then test.
- Edit the knob category that matches your goal, expect the accel↔top-speed trade, and change one thing at a
  time.

**Continue:** [C13.5 — The performance bars](05-performance-bars.md) · [Chapter 13 hub](C13-Vault-CarTuning.md)
