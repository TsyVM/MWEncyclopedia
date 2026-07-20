# C42.2 — The Engine & Drivetrain

> **The one-sentence version:** `EngineRacer` (vtable `0x008AB6A0`, **123 methods** — the most of the engine
> family) is the full drivetrain sim: throttle + RPM → torque curve → gearbox → clutch → drive split → wheel
> torque, plus NOS — the model behind the garage's performance bars.

[← C42.1 — Fidelity tiers](01-fidelity-tiers.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md) ·
[Next: C42.3 — Suspension →](03-suspension.md)

---

## The full drivetrain

`EngineRacer` is the hero-car powertrain, and its **123 methods** (verified at vtable `0x008AB6A0`) make it one
of the largest mechanic classes — because turning throttle into wheel force involves the whole drivetrain:

```
throttle + engine RPM
   → torque curve        (how much torque the engine makes at this RPM)
   → clutch              (engaged? slipping? — during shifts and launches)
   → gearbox             (× current gear ratio)
   → final drive         (× the axle ratio)
   → drive split         (front/rear/all-wheel distribution)
   → wheel torque        (delivered to each driven wheel → tyre force, C42.4)
   + NOS                 (nitrous boost added to engine torque)
```

Each stage is real drivetrain physics, and `EngineRacer`'s method count reflects the depth: the torque curve
lookup, the RPM integration, the shift logic, the clutch model, the NOS state — all live here. This is *where
throttle becomes force* ([C39.6](../C39-Vehicle-Simulation/06-input-to-tyres.md)).

> ✅ *Verified:* `EngineRacer` is a real vtable at `0x008AB6A0` with **123 methods** (confirmed by counting code
> pointers); its reflection hash `0xB2809518` appears ×4 as a vault key in `attributes.bin`. The engine family
> also includes `EngineTraffic` (67, `0x008AB8F8`), `EngineDragster` (132, `0x008ABF34`), `EngineSpline` (56,
> `0x008AB7AC`).

## The RPM loop and gearbox

The engine's core is the **RPM loop** ([C40.4](../C40-Eight-Mechanics/04-engine.md)): the driven wheels' speed,
back through the drive split and gearbox, sets the engine RPM; the RPM indexes the torque curve; the torque drives
the wheels faster; RPM rises. The **gearbox** is the discrete element in this continuous loop:

- **Each gear** is a ratio multiplying engine torque (and dividing wheel speed → engine RPM). Lower gears =
  more torque, higher RPM per wheel-speed = more acceleration; higher gears = less torque, more top speed.
- **Shifting** changes the active ratio, which drops the RPM (a shift up moves to a taller ratio, so the same
  wheel speed maps to lower RPM) — the momentary torque dip between gears.
- **The clutch** disengages briefly during a shift (interrupting torque), then re-engages — modelled so a
  mistimed shift or a launch (clutch slip off the line) behaves correctly.

`EngineDragster` (132 methods, the most) specialises exactly this: the launch (clutch dump), the perfect-shift
window, the redline management that the drag minigame ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md))
lives and dies on. So the engine family's fidelity is mostly about *how much drivetrain detail* — traffic (67)
skips the tuning surface, dragster (132) adds launch mechanics.

> 🟡 *Reasoned:* the specific drivetrain stages (torque curve → clutch → gearbox → final drive → split) are the
> standard powertrain model, consistent with `EngineRacer`'s verified 123-method size, the RPM→sound binding
> ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)), and the car-tuning vault
> ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); the exact per-method breakdown is deeper RE.

## NOS

Nitrous (NOS) is part of the engine mechanic — a boost that adds torque when activated
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)):

- **Activation** adds a torque bonus to the engine output (a big, temporary power increase).
- **A finite tank** depletes while active and refills over time (or per race, by tuning).
- **The surge** is what makes NOS a tactical tool — a burst of acceleration for a straight or an overtake.

That NOS lives in the engine mechanic (rather than a separate system) is consistent with it being a torque
modifier — it's just more engine power, gated by the tank. Its parameters (bonus, tank size, refill) are vault
data ([C42.5](05-tuning-surface.md)), tuned per car.

## The garage bars

`EngineRacer` is the model behind the garage's **performance display** — the TOPSPEED and ACCELERATION bars
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md), [Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)):

- **Top speed** is where drivetrain force (falling with RPM in top gear) balances drag — a function of the power
  curve and gearing.
- **Acceleration** is the integral of wheel force over speed — a function of torque, gearing, weight, and grip.
- **The bars** are computed from the engine (and car) parameters, so tuning the engine
  ([C42.5](05-tuning-surface.md)) moves the bars — the garage shows you the drivetrain's numbers.

So `EngineRacer` isn't just the in-race power model; it's the source of the performance metrics the player sees
and tunes against. The garage and the road are the same drivetrain, shown two ways.

## RE implications

- **`EngineRacer` (123 methods, `0x008AB6A0`)** is the full drivetrain — throttle → torque curve → clutch →
  gearbox → drive split → wheel torque, plus NOS.
- **The gearbox** is the discrete element in the RPM loop — gears trade torque for top speed; shifting dips
  torque.
- **`EngineDragster` (132)** specialises the launch/perfect-shift; **`EngineTraffic` (67)** skips the tuning
  surface.
- **NOS is a torque modifier** in the engine mechanic; the garage's performance bars are computed from this model.

---

### Key takeaways

- `EngineRacer` (**verified** vtable `0x008AB6A0`, **123 methods**) is the **full drivetrain** — where throttle
  becomes wheel force.
- The pipeline is **torque curve → clutch → gearbox → final drive → drive split → wheel torque**, driven by the
  RPM loop.
- The **gearbox** trades torque (low gears) for top speed (high gears); **shifting** dips torque; the **clutch**
  models launches and shifts.
- **`EngineDragster` (132)** adds launch/perfect-shift mechanics; **`EngineTraffic` (67)** is the cheap tier
  without the tuning surface.
- **NOS** is a torque boost in the engine mechanic; `EngineRacer` is the model behind the garage's top-speed and
  acceleration bars.

**Continue:** [C42.3 — Suspension](03-suspension.md) · [Chapter 42 hub](C42-Suspension-Tyres-Drivetrain.md)
