# C56.2 — Performance Parts

> **The one-sentence version:** performance is upgraded by parts in categories — `PART_ENGINE`, `PART_SUSPENSION`,
> `PART_BRAKE`, `PART_TIRE`, `PART_TRANS`, `PART_TURBO` — each installable at a `PerformanceLevel` tier that raises
> the relevant vault parameters, with `PerformanceMatching` scaling events to your car's level.

[← C56.1 — The two customizations](01-two-customizations.md) · [Chapter 56 hub](C56-Customization.md) ·
[Next: C56.3 — Tuning sliders →](03-tuning-sliders.md)

---

## The part categories

Performance upgrades are organised into **part categories** — the verified `PART_*` slots, each a system of the car
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)):

| Part slot | Upgrades | Affects |
|---|---|---|
| `PART_ENGINE` | the engine | power ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `PART_TURBO` | forced induction | boost/power ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `PART_TRANS` | the transmission | gearing/shift ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) |
| `PART_SUSPENSION` | the suspension | handling ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)) |
| `PART_BRAKE` | the brakes | braking |
| `PART_TIRE` | the tyres | grip ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)) |

Each category maps to a *system* of the driving model ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)):
engine/turbo/trans to the powertrain, suspension/brakes/tyres to the handling. Upgrading a category improves *that
system*, so building a car is choosing which systems to upgrade toward a balanced (or specialised) whole.

> ✅ *Verified:* the `PART_*` categories — `PART_ENGINE`, `PART_SUSPENSION`, `PART_BRAKE`, `PART_TIRE`, `PART_TRANS`,
> `PART_TURBO` (and `PART_NAME`/`PART_NO` metadata) — are present in `speed.exe`, alongside `PerformanceLevel` and
> `PerformanceMatching`.

## Installing a part edits the vault

The core mechanism ([C56.1](01-two-customizations.md), [C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)):
**installing a part raises the relevant vault parameters** ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)):

```
install PART_ENGINE (pro tier) on the car:
   → the car's engine vault fields (power curve, redline, [C42.2])
     are set to the pro-engine values
   → the sim's EngineRacer ([C42.2]) reads the new values
   → the car is now more powerful
```

So a part is a *parameter set* — installing it writes those parameters into the car's vault data
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), and the sim
([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) immediately reflects them
(the `EngineRacer`/`SuspensionRacer` code reads the new numbers). This is the data-over-code payoff
([C56.1](01-two-customizations.md)): the part system doesn't add engine *code* — it swaps engine *data*. A "pro
turbo" is a set of higher boost numbers, applied by selecting it. Any part is applied uniformly: write its
parameters, and the car changes.

## PerformanceLevel: the tiers

Parts come in **tiers** — the verified `PerformanceLevel` — each a step up in the parameters:

- **Stock** — the car as bought; baseline parameters.
- **Street / Sport** — mid-tier upgrades; better parameters.
- **Pro / top tier** — the best parts; peak parameters.

Higher `PerformanceLevel` = better numbers. A car's overall performance level is (roughly) the sum of its installed
part tiers — a fully pro-upgraded car is far faster than a stock one. `PerformanceLevel` is thus the *progression
axis* of performance customization: as you earn money/unlocks ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)),
you buy higher-tier parts, raising your car's level. This ties performance to career progression — climbing the
Blacklist ([Chapter 54](../C54-GameFlow-Blacklist/03-the-blacklist.md)) both requires and rewards a better-upgraded
car.

## PerformanceMatching: scaling the challenge

A subtle but important system is **`PerformanceMatching`** (verified) — the game *scales events* to your car's
performance level:

- **Events match your level** — an AI racer's car ([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md))
  is tuned to be competitive with *your* car's performance level, so races stay challenging as you upgrade.
- **It prevents trivial wins** — without matching, a fully-upgraded car would trivialise early events; matching
  keeps the challenge proportional.
- **It's the "rubber-band" at the car level** — complementary to the AI's in-race catch-up
  ([C46.4](../C46-AI-Goals-Actions/04-override-goals.md)), matching balances the *starting* performance.

So `PerformanceMatching` is the difficulty-balancing counterpart to the upgrade system — as you make your car
faster, the opposition scales too, keeping races competitive. This is why you can't simply out-upgrade the game:
the events match your level. It's a deliberate design ([C48.2](../C48-Pursuit-Heat/02-heat.md) does the same for
Heat) — the challenge tracks your power, so progression *feels* earned rather than trivial. Performance
customization makes you stronger *relative to a fixed baseline*, but the game keeps the *relative* challenge steady.

> 🟡 *Reasoned:* the `PerformanceMatching` interpretation (scaling AI cars to the player's performance level) is
> the natural reading of the name and MW's documented difficulty design; the exact matching formula is per-config
> data. The `PART_*` categories, `PerformanceLevel`, and `PerformanceMatching` strings are verified.

## RE implications

- **Performance parts** are `PART_*` categories — engine/turbo/trans (power), suspension/brakes/tyres (handling).
- **Installing a part edits the vault** — writes its parameter set; the sim reads the new numbers (data-over-code).
- **`PerformanceLevel` tiers** (stock → pro) raise the parameters — the progression axis of performance.
- **`PerformanceMatching`** scales events to your car's level — keeping races competitive as you upgrade.

---

### Key takeaways

- Performance is upgraded by **parts in categories** — `PART_ENGINE`/`PART_TURBO`/`PART_TRANS` (powertrain),
  `PART_SUSPENSION`/`PART_BRAKE`/`PART_TIRE` (handling) — each mapping to a driving-model system.
- **Installing a part edits the vault** — it's a **parameter set** written into the car's data
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); the sim reads the new numbers (no new code).
- **`PerformanceLevel` tiers** (stock → street → pro) raise the parameters — the progression axis, tied to career
  earnings.
- **`PerformanceMatching`** scales AI cars to your performance level — you can't out-upgrade the game; the
  challenge stays proportional.
- Performance customization is the **data-over-code payoff** — parts swap numbers, and the car gets faster.

**Continue:** [C56.3 — Tuning sliders](03-tuning-sliders.md) · [Chapter 56 hub](C56-Customization.md)
