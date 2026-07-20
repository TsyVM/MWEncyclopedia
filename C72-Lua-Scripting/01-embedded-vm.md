# C72.1 — The Embedded Lua 5.0.1 VM

> **The one-sentence version:** MW embeds a Lua 5.0.1 virtual machine to run *flow logic* — the flexible "what
> happens when" policy — over the fixed, fast C++ engine that provides the "how," keeping the front-end and event
> glue scriptable without recompiling the game.

[← Chapter 72 hub](C72-Lua-Scripting.md) · [Next: C72.2 — `LuaRuntime` & the bytecode chunks →](02-runtime-bytecode.md)

---

## Why embed a scripting language

A game engine faces two kinds of code. The **mechanism** — physics, rendering, collision — must be *fast* and rarely
changes, so it's C++ compiled into `speed.exe`. The **policy** — which screen follows which, what a cutscene triggers,
how a menu flows — changes *constantly* during development and benefits from being *flexible*, not fast. Hard-coding
policy in C++ means recompiling the whole game for every flow tweak. So MW does what most engines of its era did:
embed a **scripting language** for the policy layer.

The choice is **Lua 5.0.1** — a small, fast, embeddable language purpose-built for exactly this. Lua is tiny (the VM
is a few hundred KB), has a clean C API for binding engine functions, and runs precompiled bytecode
([C72.2](02-runtime-bytecode.md)). It's the canonical "embed a scripting layer in a C++ game" choice, and MW's use of
it is textbook: the compiled engine does the heavy work, Lua orchestrates.

> ✅ *Verified:* the version string `Lua 5.0.1` is present in `speed.exe`, along with the Lua debug API (`lua_getinfo`)
> and Lua runtime error text (`attempt to`, `stack overflow`) — the embedded VM is Lua 5.0.1.

## Policy over mechanism

The division of labour is the key idea:

- **C++ (mechanism)** — the sim ([Chapters 39–42](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), renderer
  ([Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md)), collision
  ([Chapter 63](../C63-Collision-World/C63-Collision-World.md)) — *how* things happen, fast and fixed.
- **Lua (policy)** — front-end flow ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)), event sequences, glue —
  *what* happens *when*, flexible and scripted.

Lua doesn't simulate a car or draw a frame; it *decides* — show this screen, then that one; when the race starts,
trigger this; when the player wins, go here. It's the *conductor*, not the *orchestra*. This is why the scripting
layer is thin in the executable ([C72.5](05-reading-lua.md)): it's not doing the compute-heavy work, so it needs only
a small VM and a few bridges ([C72.3](03-postoffice.md)–[C72.4](04-attributes-bridge.md)) to the C++ that does.

## Where Lua shows up

The most visible Lua is the **front-end** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)). The FEng front-end
engine runs a Lua *flow layer* over its screens: the logic that moves you from the main menu to the car lot to the
customization shop ([Chapter 68](../C68-Vehicles-Customisable-Object/C68-Vehicles-Customisable-Object.md)), reacting
to button presses and posting the messages that drive transitions. Menu flow is *pure policy* — exactly what
scripting is for — so it's the natural home of the Lua layer.

Beyond the front-end, Lua glues **event logic** ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)):
the sequences that fire when gameplay conditions are met, orchestrated through the message system
([C72.3](03-postoffice.md)). Wherever the game needs *flexible, changeable decision logic* rather than *fixed
compute*, that's where Lua sits.

> 🟡 *Reasoned:* that Lua drives the front-end flow and event glue (rather than gameplay compute) is the standard role
> of an embedded Lua layer and is consistent with the FEng flow layer ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md))
> and the message bridge ([C72.3](03-postoffice.md)); the exact set of scripted behaviours is bytecode-level detail
> ([C72.5](05-reading-lua.md)). The Lua 5.0.1 VM and its presence are verified.

## Why 5.0.1 specifically

The version matters for RE ([C72.2](02-runtime-bytecode.md)): **Lua 5.0.1** has a *specific bytecode format*
different from later 5.1+. A tool reading MW's Lua chunks must speak 5.0.1 bytecode — the header, the instruction
encoding, the constant table layout are all version-specific. 5.0.1 was current in 2004–2005 when MW was built, so
the choice simply reflects the era. Knowing the exact version ([verified above](#why-embed-a-scripting-language)) is
what makes the bytecode ([C72.2](02-runtime-bytecode.md)) *decodable* — you know which VM spec to apply.

## RE implications

- **Embedded scripting** — Lua 5.0.1 runs the *policy* layer over the C++ *mechanism*; the standard engine pattern.
- **Policy over mechanism** — Lua decides *what/when*, C++ does *how*; Lua is the conductor, not the orchestra.
- **Where it shows up** — front-end flow ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) and event glue
  ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) — flexible decision logic.
- **Version 5.0.1** — a specific bytecode format; knowing it is what makes the chunks decodable.

---

### Key takeaways

- MW embeds a **Lua 5.0.1** VM (verified string) to run the **policy** layer — the flexible "what happens when" —
  over the fixed, fast **C++ mechanism**, so flow can change without recompiling the game.
- Lua is the **conductor, not the orchestra** — it decides (show this screen, trigger that event), while C++
  simulates, renders, and collides; hence the scripting layer is **thin in the executable**.
- Lua's most visible home is the **front-end flow** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) — menu and
  shop navigation is pure policy — plus **event glue**
  ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)).
- The exact version **5.0.1** matters — it fixes the **bytecode format** ([C72.2](02-runtime-bytecode.md)), so a
  reader knows which VM spec to apply.
- The layer needs only a small VM plus a few **bridges** ([C72.3](03-postoffice.md)–[C72.4](04-attributes-bridge.md))
  to the C++ that does the real work.

**Continue:** [C72.2 — `LuaRuntime` & the bytecode chunks](02-runtime-bytecode.md) · [Chapter 72 hub](C72-Lua-Scripting.md)
