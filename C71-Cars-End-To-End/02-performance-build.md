# C71.2 — The Performance Build

> **The one-sentence version:** building performance is a loop repeated across the nine classes — pick a class, buy
> its next tier, watch its `PERF_` rating and the `TOPSPEED`/`ACCELERATION`/`HANDLING` bars rise, and feel the sim
> respond — with the class mix chosen for the build you want.

[← C71.1 — Anatomy of a car](01-anatomy.md) · [Chapter 71 hub](C71-Cars-End-To-End.md) ·
[Next: C71.3 — The visual build →](03-visual-build.md)

---

## The upgrade loop

Performance customization is one loop, run again and again
([Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)):

```
1. pick a class          (EN/EC/TU/NO/TR/SU/BR/TI/WT — C69.1)
2. buy its next tier     (add to cart, confirm — C68.4)
3. the part installs     (into its slot — C68.1)
4. tuning fields change  (the vault — Ch.13)
5. rating + bar rise     (PERF_<class> and the three bars — C69.2/C69.3)
6. the sim responds      (the car drives faster — Ch.42)
   ↺ repeat for the next class / tier
```

Each pass raises one class one tier. The feedback is immediate ([C69.4](../C69-Performance-Upgrades-Tuning/04-upgrade-to-behaviour.md)):
the bar moves as you buy, and the car changes the next time you drive — no separate apply step. A fully-built car is
the loop run to the top tier of all nine classes.

## A worked example: the engine

Take the engine class (`EN`, the deepest at ten parts,
[C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)). Building it is a sub-loop up its tiers:

```
cold air intake  ->  cat-back exhaust  ->  high-flow headers  ->  intake manifold  ->  ... -> blueprint the block
```

Each part raises the torque the sim samples ([C40.4](../C40-Eight-Mechanics/04-engine.md)), so `PERF_ENGINE`
([C69.2](../C69-Performance-Upgrades-Tuning/02-perf-ratings.md)) climbs, and because engine feeds both `TOPSPEED` and
`ACCELERATION` ([C69.3](../C69-Performance-Upgrades-Tuning/03-tuning-bars.md)), *both* those bars rise with each
engine part. Add a turbo on top (`TU`, staged 1→2→3, [C69.1](../C69-Performance-Upgrades-Tuning/01-classes-tiers.md))
and the power climbs further; add nitrous (`NO`) for the on-demand boost. The engine sub-loop alone can transform a
car's straight-line pace — and it's *visible* the whole way, because the bars preview each step
([C69.4](../C69-Performance-Upgrades-Tuning/04-upgrade-to-behaviour.md)).

## Building for a goal

Because each class feeds specific bars ([C69.3](../C69-Performance-Upgrades-Tuning/03-tuning-bars.md)), you build
*toward a goal* by choosing the class mix:

- **A top-speed car** — pour into engine (`EN`), turbo (`TU`), transmission gearing (`TR`), and weight (`WT`); these
  push `TOPSPEED`.
- **A drag/acceleration car** — engine + turbo + nitrous (`NO`) + tyres (`TI`, launch grip) + weight; these push
  `ACCELERATION`.
- **A handling car** — tyres (`TI`), suspension (`SU`), brakes (`BR`), and weight (`WT`); these push `HANDLING`.
- **A balanced car** — max everything; a maxed car tops all three bars.

Weight reduction (`WT`) is the universal good — it feeds *all three* bars
([C69.3](../C69-Performance-Upgrades-Tuning/03-tuning-bars.md)) — so it belongs in every build. This is the strategy
layer the `PERF_` ratings enable ([C69.2](../C69-Performance-Upgrades-Tuning/02-perf-ratings.md)): you can *see*
which class moves which bar, and spend accordingly.

## The economy of building

The build is gated by cash ([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)) — the career economy
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) — so the loop has a *budget*:

- You earn cash by winning ([Chapter 55](../C55-Race-Events/C55-Race-Events.md)) and evading
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- You spend it climbing tiers; **owned** parts stay yours ([C68.4](../C68-Vehicles-Customisable-Object/04-buying.md)),
  so the spend is cumulative — a maxed car is a running total of every tier bought.
- Re-tuning ([C13.6](../C13-Vault-CarTuning/06-retuning.md)) between owned parts is free — you pay to *acquire*, then
  reconfigure at no cost.

So the performance build is a *progression*: earn, buy the next tier, watch the bars rise, drive faster, earn more.
It's the loop that turns the career into a car — the mechanical spine of MW's "build your ride" fantasy, every step
of it visible in the bars and felt in the sim.

## RE implications

- **The upgrade loop** — class → buy tier → install → vault → rating + bar → sim, repeated.
- **A worked example** — the engine sub-loop climbs its ten parts, raising `PERF_ENGINE` and both speed bars.
- **Build for a goal** — class mix chooses which bars rise; weight feeds all three.
- **Economy** — cash-gated, owned-parts cumulative, re-tuning free; the career becomes the car.

---

### Key takeaways

- The performance build is **one loop** — pick a class, buy its next tier, the part installs and changes the vault,
  the **`PERF_` rating and bars rise**, the **sim responds** — repeated across the nine classes to a maxed car.
- The feedback is **immediate** ([C69.4](../C69-Performance-Upgrades-Tuning/04-upgrade-to-behaviour.md)) — the bar
  moves as you buy, the car changes as you drive — no apply step.
- You **build toward a goal** by class mix — engine/turbo/gearing/weight for top speed, tyres/suspension/brakes/
  weight for handling — with **weight (`WT`) a universal good** (all three bars).
- The build is **cash-gated** ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) but **owned parts
  are cumulative** and **re-tuning is free** ([C13.6](../C13-Vault-CarTuning/06-retuning.md)) — the career progression
  *is* the car build.

**Continue:** [C71.3 — The visual build](03-visual-build.md) · [Chapter 71 hub](C71-Cars-End-To-End.md)
