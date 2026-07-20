# Chapter 72 — Lua Scripting: the Embedded 5.0.1 Layer

> **Goal of this chapter:** decode the embedded scripting layer — the **Lua 5.0.1** VM, the precompiled **bytecode
> chunks** it runs (the `ESC"Lua"` signature), and the three bridges that wire Lua to the engine: `LuaRuntime` (the
> host), `LuaPostOffice` (the message bridge), and `LuaAttributes` (the attribute/vault bridge) — plus an honest
> account of what the layer makes moddable.

Most of the engine is C++ compiled into `speed.exe`; a thin layer on top is **scripted** in Lua. This chapter decodes
that layer: the embedded VM, how scripts ship (bytecode, not source), and the three named bridges that let a script
send engine messages ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) and
read/write attributes ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)). It's the *flow and glue*
layer — the flexible logic sitting above the fixed C++ systems, most visibly driving the front-end
([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)).

> **Verified against the executable.** The embedded VM is **Lua 5.0.1** (the version string is present in
> `speed.exe`). The three bridges are named: **`LuaRuntime`**, **`LuaPostOffice`**, **`LuaAttributes`**. The Lua
> debug API is linked (`lua_getinfo`), and Lua error text is present (`attempt to`, `stack overflow`). Precompiled
> **bytecode** is indicated by the Lua chunk signature `ESC"Lua"` (`1B 4C 75 61`).

---

## Deep-dive pages

- [C72.1 — The embedded Lua 5.0.1 VM](01-embedded-vm.md): why a scripting language is embedded, and what Lua does
  over the C++ engine.
- [C72.2 — `LuaRuntime` & the bytecode chunks](02-runtime-bytecode.md): the VM host and the precompiled chunks it
  runs.
- [C72.3 — `LuaPostOffice`: the message bridge](03-postoffice.md): how a script sends and receives engine messages.
- [C72.4 — `LuaAttributes`: the attribute bridge](04-attributes-bridge.md): how a script reads and writes the vault.
- [C72.5 — What's moddable & reading Lua in RE](05-reading-lua.md): the scripting surface and how to navigate it.

---

## 72.1 The embedded Lua 5.0.1 VM

MW embeds a **Lua 5.0.1** virtual machine ([C72.1](01-embedded-vm.md)) — a small, fast scripting language — to run
*flow logic* over the fixed C++ engine: front-end screen flow
([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)), event sequences, and glue that would be painful to hard-code.
Scripting keeps the *policy* (what happens when) flexible while the C++ keeps the *mechanism* (how it happens) fast.

## 72.2 `LuaRuntime` & the bytecode chunks

The **`LuaRuntime`** ([C72.2](02-runtime-bytecode.md)) is the VM host — it creates the Lua state, loads chunks, and
runs them. Scripts ship as **precompiled bytecode** (the `ESC"Lua"` signature), not source: the game loads compiled
Lua chunks, so the shipped data is bytecode the VM executes directly.

## 72.3 `LuaPostOffice`: the message bridge

The **`LuaPostOffice`** ([C72.3](03-postoffice.md)) bridges Lua to the engine's **message system**
([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)): a script can *post* engine
messages (the `M*`/`E*` verbs) and *receive* them, so Lua flow logic drives and reacts to the C++ systems through the
same message vocabulary the engine uses internally.

## 72.4 `LuaAttributes`: the attribute bridge

The **`LuaAttributes`** ([C72.4](04-attributes-bridge.md)) bridges Lua to the **attribute vault**
([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)): a script can *read* attribute values (tuning,
gameplay parameters) and act on them, so the data-driven vault and the script-driven flow meet — scripts reason about
the same attributes the sim consumes.

---

### Key takeaways

- MW embeds a **Lua 5.0.1** VM to run **flow logic** over the C++ engine — flexible *policy* above fixed *mechanism* —
  most visibly the front-end flow ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)).
- **`LuaRuntime`** hosts the VM and runs **precompiled bytecode chunks** (the `ESC"Lua"` signature) — scripts ship
  compiled, not as source.
- **`LuaPostOffice`** bridges Lua to the **message system**
  ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) — scripts post and
  receive the engine's `M*`/`E*` messages.
- **`LuaAttributes`** bridges Lua to the **vault** ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md))
  — scripts read the same attributes the sim consumes.
- The layer is **glue** — three bridges (runtime, messages, attributes) connecting a small scripting language to the
  big compiled engine.

**Next:** [C72.1 — The embedded Lua 5.0.1 VM](01-embedded-vm.md).
