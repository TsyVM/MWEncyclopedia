# C58.1 — The Shipping Executable

> **The one-sentence version:** `speed.exe` carries its own build metadata — a PE32 x86 image, linked with MSVC
> 7.10 (Visual Studio .NET 2003), built 2005-12-01 (retail v1.3), with 5 sections (`.text`/`.rdata`/`.data`/
> `.rsrc`/trailing) — the anatomy the whole book has read.

[← Chapter 58 hub](C58-Build-Pipeline.md) · [Next: C58.2 — The EAGL engine →](02-eagl-engine.md)

---

## The PE header tells its own story

`speed.exe` is a Windows **Portable Executable**, and its header records exactly how it was built
([Chapter 50](../C50-Verification-Methodology/02-byte-verification.md)):

- **PE32, x86** — machine `0x14C` (32-bit Intel), optional-header magic `0x10B` (PE32). A 32-bit Windows
  application, as all 2005 PC games were.
- **MSVC linker 7.10** — the linker version field is `7.10`, i.e. **Microsoft Visual C++ .NET 2003** (Visual Studio
  2003, "Everett"). This is the compiler/linker MW's PC build used.
- **Built 2005-12-01 01:06:20 UTC** — the PE timestamp is `1133399180` = 1 December 2005. Most Wanted launched in
  November 2005; this is the **retail v1.3** build, a month after launch (a patched/final release).
- **ImageBase `0x400000`** — the standard load address ([Chapter 50](../C50-Verification-Methodology/02-byte-verification.md)),
  the reason file-offset = VA − `0x400000`.

So the executable is *self-documenting* about its build: read the PE header and you know the platform (x86), the
toolchain (MSVC 2003), and the date (Dec 2005). This is the first thing to read in any RE
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) — the header orients you before a
single instruction is decoded.

> ✅ *Verified:* `speed.exe` — machine `0x14C` (x86), PE32 magic `0x10B`, **linker version 7.10** (MSVC .NET 2003),
> **timestamp 1133399180 = 2005-12-01 01:06:20 UTC**, ImageBase `0x400000`.

## The five sections

The executable has **5 sections** — the standard PE layout the whole book has navigated:

| Section | VA | Contents | Read in |
|---|---|---|---|
| `.text` | `0x401000` | executable code (~4.6 MB) | [Chapter 50](../C50-Verification-Methodology/02-byte-verification.md) (byte verification) |
| `.rdata` | `0x890000` | read-only data — strings, vtables | [Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md) (vtables) |
| `.data` | `0x8EA000` | read-write data — globals, list-heads | [Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md) (class families) |
| `.rsrc` | `0x9C7000` | Windows resources (icon, version) | — |
| (trailing) | `0xA38000` | trailing section | — |

Every address the book cited lives in one of these: the physics functions ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md))
in `.text`, the class-name strings and vtables ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) in
`.rdata`, the list-heads ([Chapter 32](../C32-Runtime-Class-System/03-eleven-families.md)) and globals (the `dt` at
`[0x9259BC]`, [Chapter 37](../C37-Frame-Spine-Modules/04-frametick.md)) in `.data`. So the section table is the
*map* of the executable — where the code, the constants, and the mutable state live. Knowing which section an
address is in tells you what kind of thing it is (code vs. read-only data vs. mutable global).

## The 4.6 MB of code

The `.text` section is **~4.6 MB** of x86 code — the entire game logic
([Chapters 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)–[57](../C57-World-Systems/C57-World-Systems.md)):
the class system, the physics, the AI, the pursuit, the renderer, the game flow. That the *whole game* fits in 4.6
MB of code (plus the data bundles, [C58.3](03-bundle-pipeline.md)) is a mark of the era — 2005 games were compact
by modern standards, the logic hand-optimised C++ ([C58.2](02-eagl-engine.md)). The book's work was, essentially,
mapping this 4.6 MB: finding the functions ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)), the
classes ([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)), and the data structures within
it, and explaining what each does. The executable is the *primary source* — 4.6 MB of code and the strings/data
around it, from which everything in this book was verified ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)).

## Why the build metadata matters

Reading the build metadata ([above](#the-pe-header-tells-its-own-story)) grounds the whole RE effort:

- **It dates and versions the target.** Knowing it's the 2005-12-01 retail v1.3 build means the addresses
  ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) are for *that specific build* — a
  different version would have different addresses. The book is pinned to this build.
- **It identifies the toolchain.** MSVC 7.1 explains the code's *idioms* — the `__thiscall` convention
  ([C41.4](../C41-Physics-RigidBody/04-simulate-thiscall.md)), the SEH prologues
  ([C41.2](../C41-Physics-RigidBody/02-physics-base.md)), the vtable layout
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) are all MSVC 2003 patterns. Recognising the compiler
  helps read its output.
- **It sets the context.** A 2005 x86 game built with VS2003 is a known quantity — the era's techniques (D3D9,
  [Chapter 51](../C51-Render-Pipeline/C51-Render-Pipeline.md); fixed pools,
  [Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) are what you'd expect, and finding them confirms
  the reading.

So the build metadata is the *frame* around the whole book — it tells you exactly *what* you're reverse-engineering
(this build, this compiler, this era), which is the essential first step of grounding every subsequent claim
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)). Read the header first; everything
else follows.

## RE implications

- **`speed.exe` is PE32 x86, MSVC 7.10, built 2005-12-01** — the retail v1.3 build, self-documented in the PE
  header.
- **5 sections** — `.text` (code), `.rdata` (strings/vtables), `.data` (globals/list-heads), `.rsrc`, trailing —
  the map of the executable.
- **~4.6 MB of code** in `.text` — the whole game logic, the book's primary source.
- **Build metadata frames the RE** — dates/versions the target, identifies the toolchain (MSVC idioms), sets the
  era context.

---

### Key takeaways

- `speed.exe` records its own build in the **PE header** — **PE32 x86**, **MSVC linker 7.10** (Visual Studio .NET
  2003), **built 2005-12-01** (retail v1.3), ImageBase `0x400000`.
- It has **5 sections** — `.text` (~4.6 MB code), `.rdata` (strings/vtables), `.data` (globals/list-heads),
  `.rsrc`, and a trailing section — the **map** of where code, constants, and mutable state live.
- The **~4.6 MB `.text`** holds the entire game logic (class system → world systems) — the book's **primary
  source**, mapped function by function.
- The **build metadata frames the whole RE** — pinning the target build/version, identifying the MSVC toolchain
  (explaining the code idioms), and setting the 2005 D3D9 context.
- Reading the PE header **first** is the essential grounding step ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md))
  — it tells you exactly what you're reverse-engineering.

**Continue:** [C58.2 — The EAGL engine](02-eagl-engine.md) · [Chapter 58 hub](C58-Build-Pipeline.md)
