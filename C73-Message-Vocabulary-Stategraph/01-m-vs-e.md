# C73.1 — Messages vs Events: the M/E Split

> **The one-sentence version:** the vocabulary has two levels — `M*` *flow messages* (high-level policy, the language
> of the gameflow and Lua) and `E*` *engine events* (low-level mechanism, what the C++ systems do) — connected by
> `M`→`E` pairs where a flow message translates into an engine event.

[← Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md) · [Next: C73.2 — The `M*` stategraph →](02-stategraph.md)

---

## Two prefixes, two levels

Scanning the message vocabulary in `speed.exe`, two prefixes dominate — and they mark two *levels* of the same
system:

- **`M*` — flow messages.** High-level *policy*: `MEnterFreeRoam`, `MEnterSafeHouse`, `MSetCopsEnabled`,
  `MOnlineQuickRace`. These read like *intentions* — "enter free roam," "enable cops," "start a quick race." They're
  the language of the gameflow ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)) and the Lua policy
  layer ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)).
- **`E*` — engine events.** Lower-level *mechanism*: `ESpawnExplosion`, `EPlayRaceNIS`, `EResetPlayerCar`,
  `ESetSimRate`. These read like *actions* — "spawn an explosion," "play the race intro," "reset the car." They're
  what the C++ systems concretely *do*.

So `M*` is *what should happen* (policy) and `E*` is *how it happens* (mechanism) — the same policy/mechanism split
the Lua layer embodies ([C72.1](../C72-Lua-Scripting/01-embedded-vm.md)), here expressed in the message vocabulary
itself. The prefix tells you which level a message lives at: `M` for the flow, `E` for the engine.

> ✅ *Verified:* the `M*` family (`MEnterFreeRoam`, `MEnterSafeHouse`, `MEnterPostRaceFlow`, `MEnteringGameplay`,
> `MLoadingComplete`, `MSetTrafficSpeed`, `MSetCopsEnabled`, `MSetCopAutoSpawnMode`, `MOnline*`) and the `E*` family
> (`EShow*`, `ESpawn*`, `EPlay*`, `ETrigger*`, `EPlayer*`, `EReset*`, `ESet*`) are strings in `speed.exe`.

## The M→E pairs

The clearest evidence that these are *two levels of one system* is the **matched pairs** — the same verb with both
prefixes:

```
MSetCopAutoSpawnMode   (flow message)   →   ESetCopAutoSpawnMode   (engine event)
```

A flow message (`M`) and an engine event (`E`) for the *same operation*. This shows the two-level flow directly: the
policy layer posts `MSetCopAutoSpawnMode` ("cops should auto-spawn"), and that resolves into the engine event
`ESetCopAutoSpawnMode` that actually reconfigures the cop system ([Chapter 49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md)).
The `M` is the *request*, the `E` is the *execution* — a translation from intent to action across the flow/engine
boundary.

Not every message pairs (many `E*` events have no `M*`, because they're purely internal engine actions), but where a
pair exists, it maps a policy decision to its mechanism. Reading the pairs is reading the *seams* between the
scripted flow and the compiled engine.

> 🟡 *Reasoned:* that `M`→`E` pairs map a flow-level request to an engine-level action (policy → mechanism) is the
> natural reading of the matched-verb naming and the two-level architecture; the exact translation site is
> per-message RE. The `M*`/`E*` strings and the paired names are verified.

## Why two levels

Splitting the vocabulary into flow and engine levels buys the same thing the Lua/C++ split buys
([C72.1](../C72-Lua-Scripting/01-embedded-vm.md)):

- **The flow can be simple.** A game-flow script ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md))
  reasons in `M*` intentions — "enter the safe house" — without knowing the dozens of engine events that entails.
- **The engine can be precise.** The C++ systems handle `E*` events — spawn this, reset that, play the other — each a
  concrete, testable action.
- **The boundary is legible.** The `M`→`E` translation is where policy meets mechanism, a clean seam you can find and
  reason about.

So the M/E split is *architecture expressed as naming*: the prefix tells you whether a message is a high-level intent
or a low-level action, and the pairs mark where one becomes the other. This is why the vocabulary is so *readable*
([C73.5](05-reading-messages.md)) — the names carry the layer structure, so scanning them reveals the engine's
policy/mechanism organisation directly.

## RE implications

- **Two prefixes, two levels** — `M*` flow messages (policy) and `E*` engine events (mechanism).
- **`M`→`E` pairs** — the same verb at both levels maps a flow *request* to an engine *action* (policy → mechanism).
- **Not all pair** — many `E*` are internal-only; pairs mark the policy/mechanism seam.
- **Architecture as naming** — the prefix carries the layer, so the vocabulary reveals the engine's organisation.

---

### Key takeaways

- The message vocabulary has **two levels**: **`M*` flow messages** (high-level *policy* — `MEnterFreeRoam`,
  `MSetCopsEnabled`) and **`E*` engine events** (low-level *mechanism* — `ESpawnExplosion`, `EResetPlayerCar`).
- **`M`→`E` pairs** (e.g. `MSetCopAutoSpawnMode`→`ESetCopAutoSpawnMode`) show the levels connect — the `M` is the
  **request** (intent), the `E` is the **execution** (action) — a policy-to-mechanism translation.
- The split is the **same policy/mechanism division** as the Lua/C++ layers
  ([C72.1](../C72-Lua-Scripting/01-embedded-vm.md)), expressed in the naming: `M` for the flow, `E` for the engine.
- It lets the **flow stay simple** (reason in intentions) and the **engine stay precise** (concrete actions), with a
  **legible seam** at the `M`→`E` translation.
- Verified: the `M*` and `E*` families and the matched-verb pairs in `speed.exe`.

**Continue:** [C73.2 — The `M*` stategraph](02-stategraph.md) · [Chapter 73 hub](C73-Message-Vocabulary-Stategraph.md)
