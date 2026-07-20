# C69.3 — The Three Tuning Bars

> **The one-sentence version:** the garage shows three bars — `TOPSPEED`, `ACCELERATION`, `HANDLING` — that are
> *summaries* of the car's underlying tuning fields, not stored ratings, so installing a part moves a bar because
> the part changes the fields and the bar re-summarises them.

[← C69.2 — The `PERF_` rating system](02-perf-ratings.md) · [Chapter 69 hub](C69-Performance-Upgrades-Tuning.md) ·
[Next: C69.4 — From upgrade to bar to behaviour →](04-upgrade-to-behaviour.md)

---

## Three numbers for a whole car

Where the `PERF_` ratings ([C69.2](02-perf-ratings.md)) score each *class*, the garage boils the whole car down to
**three bars**:

- **`TOPSPEED`** — how fast the car will ultimately go.
- **`ACCELERATION`** — how hard it pulls to get there.
- **`HANDLING`** — how well it corners and stops.

These three are the player's mental model of a build — you glance at the bars to see if a car is a straight-line
missile or a corner-carver. They compress the nine class ratings ([C69.1](01-classes-tiers.md)) into the three
dimensions that matter for *driving*: go, gather speed, and turn.

> ✅ *Verified:* `TOPSPEED`, `ACCELERATION` (`ACCEL`), and `HANDLING` are strings in `speed.exe` — the three garage
> bars.

## Summaries, not stored ratings

The crucial fact ([C13.5](../C13-Vault-CarTuning/05-performance-bars.md)) is that a bar is **not a stored number**.
There is no "top speed = 7/10" field saved on the car; the bar is *computed* from the car's actual tuning fields
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — the torque curve, the gearing, the grip, the mass —
every time the garage draws it. This is *why upgrading works*:

```
install part -> tuning fields change -> bar recomputed from fields -> bar moves
```

If bars were stored ratings, an upgrade would have to *also* write the bar, and the two could drift out of sync. By
making the bar a **live summary of the fields**, the game guarantees the bar always reflects the real build: change
the fields and the bar *necessarily* follows. This is the same principle as the HUD ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md))
— show a computed view of live state, never a cached copy — applied to the garage.

## Which upgrades move which bar

Because each bar summarises a *set* of fields, each upgrade class ([C69.1](01-classes-tiers.md)) moves the bars its
fields feed:

| Bar | Fed mainly by |
|---|---|
| `TOPSPEED` | engine (`EN`), turbo (`TU`), nitrous (`NO`), transmission gearing (`TR`), weight (`WT`) |
| `ACCELERATION` | engine (`EN`), turbo (`TU`), nitrous (`NO`), tyres launch grip (`TI`), weight (`WT`) |
| `HANDLING` | tyres (`TI`), suspension (`SU`), brakes (`BR`), weight (`WT`) |

Note **weight** (`WT`) moves *all three* — lighter is faster, quicker, and nimbler — which is why weight reduction
feels universally good. Note too that a class can feed *two* bars: the engine raises both top speed and acceleration
(more power does both), tyres raise both acceleration (launch grip) and handling (cornering grip). So the bars are
*not independent* — one upgrade can nudge two — which is the whole reason the three-bar summary is more informative
than a single "performance" number: it shows the *shape* of the gain, not just its size.

> 🟡 *Reasoned:* the class→bar mapping (which upgrades feed which bar) follows from what each class changes
> ([C68.3](../C68-Vehicles-Customisable-Object/03-part-catalog.md)) and what each bar means; the exact fields each
> bar sums are vault/UI internals ([C13.5](../C13-Vault-CarTuning/05-performance-bars.md)). The three bar strings and
> the summary-not-stored model are verified.

## The bar is a preview of the drive

The bars matter because they *predict the driving* ([C69.4](04-upgrade-to-behaviour.md)). Since the same tuning
fields the bar summarises are the fields the sim reads ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)),
a higher `TOPSPEED` bar *means* a higher actual top speed — the bar and the behaviour can't disagree, because
they're the same numbers viewed two ways. This is the payoff of the summary design: the garage bar is a *faithful
preview* of what the car will do, not a marketing approximation. Reading the bars is reading a compressed forecast
of the sim ([C69.4](04-upgrade-to-behaviour.md)) — which is exactly what a player wants before spending cash.

## RE implications

- **Three bars** — `TOPSPEED`, `ACCELERATION`, `HANDLING` — compress the nine class ratings into the driving
  dimensions.
- **Summaries, not stored** — computed live from the tuning fields ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md));
  upgrading moves a bar because the fields change.
- **Class → bar mapping** — each class feeds the bars its fields drive; weight feeds all three; classes can feed
  two.
- **A faithful preview** — the bar summarises the same fields the sim reads, so it can't disagree with the drive.

---

### Key takeaways

- The garage shows **three bars** — **`TOPSPEED`**, **`ACCELERATION`**, **`HANDLING`** — compressing the nine class
  ratings into the three dimensions that matter for driving.
- A bar is a **live summary of the tuning fields, not a stored rating** ([C13.5](../C13-Vault-CarTuning/05-performance-bars.md))
  — so **installing a part moves a bar** because the part changes the fields and the bar re-summarises them (the same
  show-computed-state principle as the HUD).
- **Each class feeds the bars its fields drive** — engine/turbo → speed + accel, tyres → accel + handling,
  suspension/brakes → handling, and **weight feeds all three**; classes can feed two bars, so the *shape* of a gain
  is visible.
- The bar is a **faithful preview of the drive** — it summarises the same fields the sim reads
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)), so bar and behaviour can't
  disagree.
- Verified: `TOPSPEED`, `ACCELERATION`, `HANDLING`, and the summary-not-stored model.

**Continue:** [C69.4 — From upgrade to bar to behaviour](04-upgrade-to-behaviour.md) ·
[Chapter 69 hub](C69-Performance-Upgrades-Tuning.md)
