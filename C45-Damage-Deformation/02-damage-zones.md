# C45.2 — Damage Zones

> **The one-sentence version:** a car has two zone systems — six **coarse zones** (`DAMAGE0_FRONT`/`FRONTLEFT`/
> `FRONTRIGHT`/`REAR`/`REARLEFT`/`REARRIGHT`) that accumulate impact energy, and many **part-specific breakables**
> (`DAMAGE_HOOD`, `DAMAGE_LEFT_DOOR`, `DAMAGE_LEFT_HEADLIGHT`, …) that break individually — all verified strings.

[← C45.1 — The Damage family](01-damage-family.md) · [Chapter 45 hub](C45-Damage-Deformation.md) ·
[Next: C45.3 — Deformation & breakables →](03-deformation.md)

---

## Two zone systems

The car isn't one damage bucket — it's divided into **zones**, and the verified strings reveal *two* systems
working together:

**1. Coarse zones (`DAMAGE0_*`)** — six regions that accumulate impact energy:

```
DAMAGE0_FRONT   DAMAGE0_FRONTLEFT   DAMAGE0_FRONTRIGHT
DAMAGE0_REAR    DAMAGE0_REARLEFT    DAMAGE0_REARRIGHT
```

Front/rear × left/centre/right — a 6-cell grid covering the car's body. Each accumulates the damage from contacts
in its area, and drives the *panel deformation* ([C45.3](03-deformation.md)) for that region.

**2. Part-specific breakables (`DAMAGE_*`)** — individual parts that break/detach:

```
DAMAGE_HOOD   DAMAGE_TRUNK   DAMAGE_BODY   DAMAGE_BUSHGUARD
DAMAGE_FRONT_BUMPER   DAMAGE_REAR_BUMPER
DAMAGE_LEFT_DOOR   DAMAGE_RIGHT_DOOR   DAMAGE_LEFT_REAR_DOOR   DAMAGE_RIGHT_REAR_DOOR
DAMAGE_LEFT_HEADLIGHT   DAMAGE_RIGHT_HEADLIGHT
DAMAGE_LEFT_BRAKELIGHT   DAMAGE_RIGHT_BRAKELIGHT
DAMAGE_FRONT_WINDOW   DAMAGE_FRONT_LEFT_WINDOW   DAMAGE_FRONT_RIGHT_WINDOW
DAMAGE_REAR_LEFT_WINDOW   DAMAGE_REAR_RIGHT_WINDOW
DAMAGE_FRONT_WHEEL   DAMAGE_COP_LIGHTS   DAMAGE_COP_SPOILER
```

So the car is a *grid* of coarse zones (for crumple) *and* a *list* of nameable parts (for breakage). A contact
feeds both: the region's coarse zone accumulates energy, and any parts in the hit area may break.

> ✅ *Verified:* the coarse `DAMAGE0_*` zones (6) and the part-specific `DAMAGE_*` breakables (listed above) are all
> present as strings in `speed.exe`, alongside glass detail (`LEFT_HEADLIGHT_GLASS`, `RIGHT_HEADLIGHT_GLASS`,
> `WINDSHIELD`, `BREAK_HEADLIGHT_LEFT`/`_RIGHT`) and `BUMPER`/`BUMPER_DECAL`.

## Coarse zones: crumple regions

The six `DAMAGE0_*` zones are the **crumple regions** — the granularity at which the car's body deforms
([C45.3](03-deformation.md)). Each is a spatial region (a corner or end of the car) with its own accumulated
damage:

- **A front-left impact** ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) feeds
  `DAMAGE0_FRONTLEFT`, crumpling the front-left of the car.
- **A rear-end hit** feeds `DAMAGE0_REAR` (and/or the rear corners), crumpling the back.
- **Damage accumulates** — repeated hits to a zone deepen its deformation, up to a limit.

Six zones is a deliberate granularity: enough that damage is *directional* (you can see which corner you hit), few
enough to be cheap and to map cleanly to the car's body panels. It's the coarse "how bent is each corner" layer.

## Breakables: the detail layer

The `DAMAGE_*` parts are the **detail layer** — the individual bits that pop, shatter, and fall off
([C45.3](03-deformation.md)):

- **Glass** — windows and headlights shatter (`WINDSHIELD`, `DAMAGE_FRONT_WINDOW`, `LEFT_HEADLIGHT_GLASS`).
- **Panels** — the hood, trunk, doors, and bumpers deform and can detach (`DAMAGE_HOOD`, `DAMAGE_FRONT_BUMPER`,
  `DAMAGE_LEFT_DOOR`).
- **Lights** — headlights and brakelights break, going dark (`BREAK_HEADLIGHT_LEFT`, `DAMAGE_LEFT_BRAKELIGHT`).
- **Cop parts** — the light bar and spoiler (`DAMAGE_COP_LIGHTS`, `DAMAGE_COP_SPOILER`,
  [C45.5](05-cop-damage.md)).

Each breakable is a nameable part with intact/broken states, tied to a location on the car. When the coarse zone
covering its location takes enough damage, the part breaks. So the two systems are layered: the coarse zone is the
*energy accumulator*, and the breakables are the *discrete state changes* that fire as the zone's damage crosses
thresholds ([C45.3](03-deformation.md)).

## Why two systems

Splitting damage into coarse zones + named breakables is a clean division:

- **Coarse zones for continuous crumple.** The gradual, analog deformation of a corner is naturally a *scalar per
  region* — six accumulators, each driving a smooth mesh deformation ([C45.3](03-deformation.md)).
- **Named parts for discrete breakage.** A door falling off or a headlight shattering is a *discrete event* on a
  *specific part* — naturally a per-part state, not a region scalar.
- **Together they cover both.** A crash both crumples (coarse) and breaks (parts) — the two systems capture the
  two kinds of damage a real crash produces.

So the zone design matches the *physics of damage*: bulk deformation is regional and continuous; part failure is
specific and discrete. Most Wanted models both, which is why a damaged car both *sags* (crumpled panels) and
*sheds* (broken glass, lost bumpers) — the two verified systems working in concert.

> 🟡 *Reasoned:* the mapping of coarse zones to continuous crumple and named parts to discrete breakage is inferred
> from the two verified string sets and observed damage behaviour; the exact zone→part threshold wiring is deeper
> RE. The zone and part **strings** are verified.

## RE implications

- **Two zone systems** — six coarse `DAMAGE0_*` (crumple regions) + many part-specific `DAMAGE_*` breakables — all
  verified strings.
- **Coarse zones accumulate impact energy** per region (front/rear × left/centre/right) → panel deformation.
- **Breakables are named parts** (hood, doors, glass, lights, cop parts) with intact/broken states, firing at
  thresholds.
- **The split matches damage physics** — regional continuous crumple + specific discrete breakage.

---

### Key takeaways

- A car has **two damage-zone systems**: six **coarse zones** (`DAMAGE0_FRONT`/`FRONTLEFT`/`FRONTRIGHT`/`REAR`/
  `REARLEFT`/`REARRIGHT`) and many **part-specific breakables** (`DAMAGE_HOOD`, `DAMAGE_LEFT_DOOR`,
  `DAMAGE_LEFT_HEADLIGHT`, …) — all **verified strings**.
- **Coarse zones** accumulate impact energy per region and drive **panel crumple**; six is enough for directional
  damage, cheap enough to map to panels.
- **Breakables** are nameable parts (glass, panels, lights, cop light bar) with **intact/broken states** that fire
  when their zone crosses a threshold.
- The **two systems layer**: coarse zones are continuous energy accumulators; breakables are discrete state
  changes.
- The split **matches the physics of damage** — bulk crumple (regional, continuous) + part failure (specific,
  discrete).

**Continue:** [C45.3 — Deformation & breakables](03-deformation.md) · [Chapter 45 hub](C45-Damage-Deformation.md)
