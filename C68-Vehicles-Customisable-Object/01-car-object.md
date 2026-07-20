# C68.1 — The Car as an Object

> **The one-sentence version:** a car is a runtime object — `PlayerCar` (the owned instance) of a `CarType` (the
> model) — that holds **slots** (`CarSlot`) filled by **parts** (`CarPart`); customising it changes which parts fill
> the slots, and the vehicle sim reads the resulting configuration from the vault.

[← Chapter 68 hub](C68-Vehicles-Customisable-Object.md) · [Next: C68.2 — The shop's categories →](02-shop-categories.md)

---

## Two words for "car"

The engine distinguishes two ideas the player conflates as "my car":

- **`CarType`** — the *model*: the BMW M3 GTR, the Porsche 911, the Lexus IS300. It names the mesh
  ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), the base tuning ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)),
  and the customization options the model supports. There is one `CarType` per car in the game.
- **`PlayerCar`** — the *owned instance*: *your* M3 GTR, with *your* installed parts, paint, and tuning. It's a
  `CarType` plus the specific configuration you've built and bought.

This is the classic **type/instance** split ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)):
`CarType` is the class, `PlayerCar` is the object. Customization operates on the `PlayerCar` — it never changes the
`CarType` (the model definition is read-only game data) — so building a car is *configuring an instance*, and the
shared model data stays untouched.

> ✅ *Verified:* `PlayerCar` (×4), `CarType`, `CarSlot`, `CarPart`, and `CustomizePart` (×4) are strings in
> `speed.exe` — the owned-car object, the model, the slot, the part, and the customization operation.

## Slots and parts

The customisable structure of a `PlayerCar` is a set of **slots** (`CarSlot`), each holding one **part**
(`CarPart`):

- A **slot** is a customization point — the engine slot, the front brake slot, the spoiler slot, the paint slot.
- A **part** (`CarPart`) is what fills it — a specific cold-air intake, a specific spoiler, a specific colour
  ([C68.3](03-part-catalog.md)).
- **Installing** a part means *assigning it to its slot*; the car's configuration is the set of currently-installed
  parts across all slots.

So a `PlayerCar` is, structurally, a `CarType` plus a **slot→part map**. This is why customization is *reversible*
and *composable*: swapping a part is reassigning one slot, and the car is always the sum of its slots' current
parts. The performance slots feed the sim ([below](#the-object-feeds-the-sim)); the visual slots feed the renderer
([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)).

> 🟡 *Reasoned:* the slot→part model (a `PlayerCar` as a `CarType` plus a map of `CarSlot`→`CarPart`) is the natural
> reading of the verified `CarSlot`/`CarPart`/`CustomizePart` strings and the customization screens
> ([C68.2](02-shop-categories.md)); the exact slot table is per-`CarType` vault data
> ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)). The strings and the type/instance split are verified.

## The object feeds the sim

The reason the object model matters is that the **vehicle simulation reads it**. The car the physics simulates
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) is a rigid body
([Chapter 41](../C41-Physics-RigidBody/01-rigidbody-tree.md)) whose parameters — engine torque, gearing, grip,
brake force — come from the vault ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), *selected by the
installed parts*. Install a better turbo and the vault lookup returns more boost; the sim
([C40.1](../C40-Eight-Mechanics/01-the-mechanic-model.md)) then produces more power. So:

```
PlayerCar (CarType + installed parts)  ->  vault lookup  ->  sim parameters  ->  driving behaviour
```

The car object is the *bridge* between what you buy and how the car drives. Customization
([Chapter 56](../C56-Customization/C56-Customization.md)) is meaningful precisely because the object's
configuration is *read by the sim every time it needs a parameter* — there is no separate "apply upgrades" step, the
sim just reads whatever parts are currently installed. This is also why the two customizations
([C56.1](../C56-Customization/01-two-customizations.md)) differ: performance parts change vault values the sim reads;
visual parts change meshes/textures the renderer reads — same object, two consumers.

## RE implications

- **`CarType` vs `PlayerCar`** — the model (class) vs the owned instance (object); customization operates on the
  instance.
- **Slots and parts** — a `PlayerCar` is a `CarType` plus a `CarSlot`→`CarPart` map; installing = assigning a slot.
- **The object feeds the sim** — installed parts select vault parameters
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) the physics reads
  ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)).
- **Two consumers** — performance parts → sim (vault); visual parts → renderer (mesh/texture).

---

### Key takeaways

- The engine splits "car" into **`CarType`** (the model/class — one per car) and **`PlayerCar`** (the owned
  instance/object — your build) — the classic **type/instance** split
  ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)); customization only ever touches the
  instance.
- A `PlayerCar` is structurally a `CarType` plus a **`CarSlot`→`CarPart` map** — installing a part **assigns it to
  its slot**, so the car is always the sum of its slots' current parts (reversible, composable).
- The object is the **bridge** between buying and driving: installed parts **select vault parameters**
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) the sim reads every tick — no separate "apply" step.
- **Two consumers** read the object: the **sim** reads performance parts (via the vault), the **renderer** reads
  visual parts (meshes/textures, [Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)).
- Verified: `PlayerCar`, `CarType`, `CarSlot`, `CarPart`, `CustomizePart` — the object, model, slot, part, and
  customization operation.

**Continue:** [C68.2 — The shop's categories](02-shop-categories.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md)
