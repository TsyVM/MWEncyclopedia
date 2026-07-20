# C72.5 — What's Moddable & Reading Lua in RE

> **The one-sentence version:** the Lua layer is a modding surface for *policy* — flow and event logic — reached by
> decompiling the 5.0.1 bytecode chunks (aided by retained debug info) and understood through the two bridges;
> reading it means finding the chunks, the `LuaRuntime` host, and the `LuaPostOffice`/`LuaAttributes` bindings.

[← C72.4 — `LuaAttributes`: the attribute bridge](04-attributes-bridge.md) · [Chapter 72 hub](C72-Lua-Scripting.md) ·
[Next: Chapter 73 — The Message Vocabulary & Stategraph →](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)

---

## Anchors for Lua RE

The Lua layer is anchored on verified strings:

- **The VM** — `Lua 5.0.1`, `lua_getinfo`, the error text (`attempt to`, `stack overflow`) ([C72.1](01-embedded-vm.md)).
- **The host** — `LuaRuntime`, and the `ESC"Lua"` bytecode signature ([C72.2](02-runtime-bytecode.md)).
- **The bridges** — `LuaPostOffice` (messages, [C72.3](03-postoffice.md)), `LuaAttributes` (vault,
  [C72.4](04-attributes-bridge.md)).

From these, the whole layer is navigable: the VM, the chunks it runs, and the two ways scripts touch the engine.

## The RE workflow

Reading the Lua layer:

1. **Confirm the VM** — `Lua 5.0.1` ([C72.1](01-embedded-vm.md)); the bytecode dialect to apply.
2. **Find the chunks** — the `ESC"Lua"` signature ([C72.2](02-runtime-bytecode.md)); the precompiled scripts.
3. **Decompile** — a Lua 5.0.1 decompiler turns bytecode back to readable source, aided by any retained debug
   info ([C72.2](02-runtime-bytecode.md)).
4. **Trace the bridges** — `LuaPostOffice`/`LuaAttributes` ([C72.3](03-postoffice.md)–[C72.4](04-attributes-bridge.md));
   what a script can *do* and *know*.

The output is the scripting picture: which chunks exist, what messages they post/receive, and what attributes they
read.

## What's moddable

The Lua layer is a **policy** modding surface — it controls *flow and events*, not compute ([C72.1](01-embedded-vm.md)):

- **Front-end flow** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) — menu/shop navigation, screen
  transitions. Editing the flow scripts changes *how the UI moves*.
- **Event logic** ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) — the
  sequences that fire on gameplay conditions. Editing them changes *what triggers what*.

Modding it is harder than editing vault data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), because
the scripts ship as **bytecode** ([C72.2](02-runtime-bytecode.md)): you must decompile a chunk, edit the logic, and
recompile it to *matching 5.0.1 bytecode* for the VM to load it. The version lock is the main friction — the toolchain
must speak 5.0.1 exactly. But the payoff is real: Lua is *where the flexible logic lives*, so flow and event mods that
would be impossible in the compiled C++ are reachable through the script layer.

> 🟡 *Reasoned:* the moddable surface (front-end flow + event logic, via decompile→edit→recompile 5.0.1 bytecode) is
> the standard reach of an embedded Lua layer and follows from the bridges and the bytecode shipping model; the exact
> shipped chunk set and whether the runtime accepts re-inserted chunks are bytecode/runtime detail. The VM, bridges,
> and bytecode signature are verified.

## An honest accounting

What the evidence *proves*, and what it doesn't — the book's discipline
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) applied to a thin layer:

- **Proven** — MW embeds **Lua 5.0.1** ([C72.1](01-embedded-vm.md)); it runs **precompiled bytecode**
  ([C72.2](02-runtime-bytecode.md)); it exposes **three named components** — `LuaRuntime`, `LuaPostOffice`,
  `LuaAttributes`; the VM keeps **debug info** (`lua_getinfo`, error text).
- **Reasoned** — that Lua drives *front-end flow and event glue* (the standard role, consistent with FEng
  [Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md) and the message system
  [Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)); that the bridges are
  read (attributes) and act (messages) channels.
- **Not established here** — the *exact* shipped scripts, whether `LuaAttributes` permits writes, and the precise
  binding API — all bytecode-level, recoverable by decompiling the chunks ([above](#the-re-workflow)).

Stating the tiers plainly is the point: the layer's *architecture* is verified (VM + three bridges + bytecode), while
its *contents* (the specific scripts) are a decompilation task. This is the honest shape of the Lua layer's evidence —
a solid frame, with the interior recoverable but not yet enumerated.

## RE implications

- **Anchor on** `Lua 5.0.1`, the `ESC"Lua"` signature, `LuaRuntime`, `LuaPostOffice`, `LuaAttributes`.
- **The RE workflow** — confirm VM → find chunks → decompile → trace bridges.
- **Moddable** — front-end flow + event logic, via decompile→edit→recompile 5.0.1 bytecode.
- **Honest tiers** — architecture verified (VM + bridges + bytecode); specific scripts are a decompilation task.

---

### Key takeaways

- The Lua layer is anchored on **`Lua 5.0.1`**, the **`ESC"Lua"`** bytecode signature, **`LuaRuntime`**, and the two
  bridges **`LuaPostOffice`**/**`LuaAttributes`** — from which the whole layer is navigable.
- The RE workflow: **confirm the VM → find the `ESC"Lua"` chunks → decompile (aided by retained debug info) → trace
  the bridges** — yielding which scripts exist, what they post, and what they read.
- **Moddable surface** — **front-end flow** ([Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md)) and **event logic**
  ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) — reached by
  decompile→edit→recompile to **matching 5.0.1 bytecode** (the version lock is the friction).
- **Honest accounting** — the **architecture** is verified (Lua 5.0.1 + three bridges + bytecode + debug info); the
  **specific scripts** are a decompilation task, not yet enumerated.
- The layer is a **policy** surface — flexible flow/event logic above the fixed C++ mechanism — the flexible edge of
  an otherwise-compiled engine.

**Next:** [Chapter 73 — The Message Vocabulary & Stategraph](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md).

**Sources:** `speed.exe` (verified strings: `Lua 5.0.1`, `lua_getinfo`, `attempt to`, `stack overflow`; `LuaRuntime`,
`LuaPostOffice`, `LuaAttributes`; the `ESC"Lua"` = `1B 4C 75 61` bytecode signature). Message system:
[Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md). Vault:
[Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md). Front-end flow:
[Chapter 65](../C65-HUD-Runtime/C65-HUD-Runtime.md). The Lua 5.0.1 chunk format is the documented Lua spec.
