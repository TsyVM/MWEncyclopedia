# C72.2 — `LuaRuntime` & the Bytecode Chunks

> **The one-sentence version:** `LuaRuntime` is the VM host — it creates the Lua state, loads the precompiled
> **bytecode** chunks (marked by the `ESC"Lua"` signature), and runs them — so the game ships *compiled* Lua the VM
> executes directly, not source it parses at load.

[← C72.1 — The embedded Lua 5.0.1 VM](01-embedded-vm.md) · [Chapter 72 hub](C72-Lua-Scripting.md) ·
[Next: C72.3 — `LuaPostOffice`: the message bridge →](03-postoffice.md)

---

## The runtime host

**`LuaRuntime`** is the engine's wrapper around the Lua VM — the object that owns the Lua *state* and drives it. Its
job is the standard embed lifecycle:

- **Create** the Lua state (the `lua_State`, the VM's world of globals, stack, and registry).
- **Register** the C bindings — the engine functions Lua can call, and the bridges
  ([C72.3](03-postoffice.md)–[C72.4](04-attributes-bridge.md)).
- **Load** bytecode chunks ([below](#bytecode-not-source)) into the state.
- **Run** them, and field their calls back into the engine.
- **Report** errors — the Lua error text in the executable (`attempt to`, `stack overflow`) surfaces through here,
  and the debug API (`lua_getinfo`) lets it attach *where* a script failed.

So `LuaRuntime` is the *boundary* between C++ and Lua: everything the script world touches in the engine goes through
it. It's the single host the rest of the engine talks to when it wants to run or message a script.

> ✅ *Verified:* `LuaRuntime` is a string in `speed.exe`; the linked Lua debug API (`lua_getinfo`) and error text
> (`attempt to`, `stack overflow`) confirm a full VM with error reporting, not a stripped subset.

## Bytecode, not source

MW ships Lua as **precompiled bytecode**, not `.lua` source. The tell is the **Lua chunk signature** — every compiled
Lua chunk begins with `ESC"Lua"` (`1B 4C 75 61`), and that byte sequence is present in `speed.exe`. A precompiled
chunk is the output of Lua's compiler (`luac` or `lua_dump`): the source has already been parsed and compiled to VM
instructions, so the game *loads* instructions rather than *compiling* text.

The header of a Lua 5.0.1 chunk is a small, fixed preamble followed by the top-level function:

```
1B 4C 75 61   ESC "Lua"          — the signature
50            version 0x50       — Lua 5.0
<format>      format byte
<endian>      0=big, 1=little
<sizes>       sizeof(int), size_t, Instruction, lua_Number
<test number> a known constant to verify the number format
then: the top-level function prototype
      — its constants, instructions, and (if kept) debug line info
```

So a bytecode chunk is a *function prototype tree*: instructions, a constant table, nested function prototypes, and
optional debug info (source name + line numbers). The VM loads this directly into runnable form — no parser needed at
runtime.

> ✅ *Verified:* the Lua chunk signature `ESC"Lua"` (`1B 4C 75 61`) is present in `speed.exe` (Lua 5.0 = version byte
> `0x50`); MW's scripts are precompiled bytecode.
> 🟡 *Reasoned:* the exact header field layout above is the documented Lua 5.0.1 chunk format; the count and content
> of MW's shipped chunks are bytecode-level detail ([C72.5](05-reading-lua.md)).

## Why bytecode

Shipping bytecode instead of source has concrete advantages the choice reflects:

- **No runtime parser** — loading skips lexing and compiling; the VM reads instructions straight in. Faster load,
  smaller runtime (the compiler needn't ship).
- **Compact** — bytecode is denser than source text.
- **Mild obfuscation** — bytecode isn't human-readable, so the scripting logic isn't casually editable (though it's
  *decompilable*, [C72.5](05-reading-lua.md)).

The trade is that bytecode is **version-locked** ([C72.1](01-embedded-vm.md)): a 5.0.1 chunk only loads in a 5.0.1
VM, because the instruction encoding and header are version-specific. This is why the version pin matters — the
shipped bytecode and the embedded VM must match exactly, and any tool reading the chunks must speak the same 5.0.1
dialect.

## Debug info and error reporting

That `lua_getinfo` is linked is a small but telling detail. `lua_getinfo` is Lua's **debug interface** — it retrieves
information about a running function (its name, the current line, the source). Its presence means the VM can produce
*located* errors: not just "attempt to index a nil value" but *where* it happened. Combined with the error strings
(`attempt to`, `stack overflow`), this shows MW's Lua build kept **debug information** — the bytecode chunks likely
retain source names and line numbers, and the runtime can report them. For RE, that's a gift
([C72.5](05-reading-lua.md)): debug info in the chunks means decompiled scripts can carry their original line
structure, and error paths name their scripts.

## RE implications

- **`LuaRuntime` is the host** — creates the state, registers bindings, loads/runs chunks, reports errors; the C++/Lua
  boundary.
- **Bytecode, not source** — the `ESC"Lua"` signature marks precompiled chunks; the VM loads instructions directly.
- **Chunk format** — signature + version + sizes, then a function-prototype tree (instructions, constants, debug
  info).
- **Debug info kept** — `lua_getinfo` + error text = located errors and (likely) source/line info in the chunks.

---

### Key takeaways

- **`LuaRuntime`** is the VM **host** — it owns the Lua state, registers the engine bindings (including the bridges,
  [C72.3](03-postoffice.md)–[C72.4](04-attributes-bridge.md)), loads and runs chunks, and reports errors — the single
  C++/Lua boundary.
- MW ships **precompiled bytecode**, not source — the **`ESC"Lua"`** signature (`1B 4C 75 61`, version `0x50`) marks
  each chunk; the VM loads **instructions** directly, no runtime parser.
- A chunk is a **function-prototype tree** — signature/version/sizes preamble, then instructions, a constant table,
  nested prototypes, and optional debug info.
- Bytecode is **version-locked to 5.0.1** — chunk and VM must match, which is why the version pin
  ([C72.1](01-embedded-vm.md)) matters for reading it.
- The linked **`lua_getinfo`** debug API + error strings show MW kept **debug info** — located errors, and likely
  source/line data in the chunks (a boon to RE, [C72.5](05-reading-lua.md)).

**Continue:** [C72.3 — `LuaPostOffice`: the message bridge](03-postoffice.md) · [Chapter 72 hub](C72-Lua-Scripting.md)
