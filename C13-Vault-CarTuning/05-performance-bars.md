# C13.5 — The Performance Bars

> **The one-sentence version:** the garage's top-speed / acceleration / handling bars are a *summary* of the
> underlying tuning fields, not stored ratings — which is why upgrading a part moves a bar: the part changes
> the values, and the bar re-summarises them.

[← C13.4 — Tuning value → simulation knob](04-value-to-sim.md) · [Chapter 13 hub](C13-Vault-CarTuning.md) ·
[Next: C13.6 — Retuning a car safely →](06-retuning.md)

---

## The bars summarise, they don't define

Most Wanted's garage shows a car's performance as three bars — **top speed**, **acceleration**, **handling**
(and often **nitrous**). These are a *presentation* of the underlying behavior fields
([C13.2](02-behavior-classes.md)), computed to give the player a legible, comparable rating. The car's actual
behaviour comes from the many `Float` fields the solvers read ([C13.4](04-value-to-sim.md)); the bars compress
those into three numbers.

The direction of causality matters: **fields → bars**, never the reverse. You do not tune a car by setting a
bar; you tune the fields, and the bar reflects the result. Editing the bars' *appearance* without changing the
fields would make the garage lie about a car that still drives the old way.

## Why upgrades move the bars

The upgrade/parts system is the clearest illustration. Installing a better engine part changes the powertrain
fields; installing better tires changes grip fields. Because the bars are computed from those fields, the
relevant bar rises when the part is fitted. So an upgrade is, under the hood, an **override of tuning fields**
([C12.6](../C12-Reflection-Schema/06-writing-values.md)) — the parts system writes new values, and the bars
re-summarise. This is the parts side of the same vault you have been reading.

> 🟡 *Reasoned:* that the bars are computed from the behavior fields (rather than stored as independent
> ratings) is inferred from how upgrades move them and from MW's data-driven design; the ✅ verified fact is
> that the tuning fields these bars summarise are real `Float` vault fields under the behavior collections.

## Reading a bar back to its fields

To understand or reproduce a bar you work backwards to the fields that feed it:

- **Top speed bar** ← gearing/top-speed and power fields ([C13.4](04-value-to-sim.md)): the terminal velocity
  the drivetrain allows.
- **Acceleration bar** ← engine power/response and gearing: how fast the car reaches speed.
- **Handling bar** ← suspension/grip and mass fields: cornering ability and responsiveness.
- **Nitrous bar** ← NOS fields: strength/duration of the boost.

A tool that wants to *predict* a bar computes a rating from these fields; a tool that only wants to *change* a
bar edits the fields and lets the game recompute the display. The latter is almost always what a modder wants.

## Bars vs simulation

There is a subtlety worth internalising: the bar is a **rating**, the simulation is the **truth**. Two cars
can share a top-speed bar yet reach different real velocities if their gearing differs
([C13.4](04-value-to-sim.md)), because the bar is a compressed summary and the sim is the full computation. So
trust the fields for behaviour and the bars for comparison, and don't be surprised when a bar and a stopwatch
disagree slightly — they measure different things.

## Editing implications

- **Edit fields, not bars.** Change the behavior tuning ([C13.6](06-retuning.md)); let the bar recompute.
- **Expect coupled movement.** Raising power moves the acceleration bar and possibly top speed; raising grip
  moves handling. One field can nudge more than one bar.
- **Use the bars for comparison, the fields for tuning.** The bar tells you roughly where a car sits; the
  fields tell you exactly why.

---

### Key takeaways

- The garage bars (top speed, acceleration, handling, nitrous) **summarise** the behavior fields; they are not
  stored ratings.
- Causality is fields → bars; upgrades move bars by overriding tuning fields.
- Map each bar back to its feeding fields (gearing/power → top speed & accel; grip/mass → handling; NOS →
  nitrous).
- The bar is a rating, the simulation is the truth — equal bars can drive differently.
- Edit the fields and let bars recompute; use bars to compare, fields to tune.

**Continue:** [C13.6 — Retuning a car safely](06-retuning.md) · [Chapter 13 hub](C13-Vault-CarTuning.md)
