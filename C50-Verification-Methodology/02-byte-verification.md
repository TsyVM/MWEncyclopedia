# C50.2 — Byte Verification

> **The one-sentence version:** the strongest proof is reading the machine code itself — file-offset = VA −
> `0x400000`, then decode: opcodes like `fsqrt` prove a function's identity beyond doubt, and prologue signatures
> (`mov esi,ecx`, `sub esp,0xNNN`, SEH setup) identify and characterise every function.

[← C50.1 — The confidence tiers](01-confidence-tiers.md) · [Chapter 50 hub](C50-Verification-Methodology.md) ·
[Next: C50.3 — Hash verification →](03-hash-verification.md)

---

## The address-to-bytes mapping

`speed.exe` is a PE32 executable with **ImageBase `0x400000`**, and — the key fact that makes byte verification
cheap — for this image **the file offset of a virtual address is `VA − 0x400000`**. So to read the code at VA
`0x5C5E80`, you read the file at offset `0x1C5E80`. No disassembler framework is needed: a raw byte read plus
hand-decoding of x86 is enough to verify a function.

```python
exe = open('speed.exe','rb').read()
IB  = 0x400000
def bytes_at(va, n): return exe[va-IB : va-IB+n]
```

This simplicity is deliberate in the book's method: every byte-verified claim can be reproduced with four lines of
Python and an x86 opcode reference. There's no tooling barrier between the reader and the proof.

> ✅ *Verified:* ImageBase is `0x400000`; file-offset = VA − `0x400000`. Confirmed by every byte-verified function
> in the book resolving correctly at that mapping (e.g. `Physics::Simulate 0x6BB4D0` → file `0x2BB4D0` =
> `56 8B F1 …`).

## The gold standard: opcodes that prove identity

The strongest possible byte verification is a function whose **opcodes alone prove what it does**. The exemplar is
`Math::Sqrt (0x5C5E80)`:

```asm
D9 44 24 04     fld   dword ptr [esp+4]   ; load the float argument
D9 FA           fsqrt                      ; ← the x87 square-root opcode
C3              ret
```

The `fsqrt` opcode (`D9 FA`) is *unambiguously* the FPU square root. A function that loads a float, executes
`fsqrt`, and returns *is* a square root — there is no interpretation, no inference, no room for doubt. This is the
gold standard: when the opcodes themselves name the operation, the claim is as certain as anything in computing.

Not every function is this clean, but many carry opcodes that strongly constrain their identity — `imul`/`idiv`
for arithmetic, `fmul`/`fadd` for float math, `rep movs` for memory copies, `call [eax+N]` for virtual dispatch.
Reading these tells you *what a function does* from its instructions, not from a guess.

## Prologue signatures

Even without a defining opcode, a function's **prologue** identifies and characterises it — the first few bytes
encode its calling convention and stack usage:

| Prologue bytes | Meaning |
|---|---|
| `8B FF` / `55 8B EC` | standard function entry (`mov edi,edi` hotpatch / `push ebp; mov ebp,esp`) |
| `56 8B F1` | `push esi; mov esi,ecx` — a **`__thiscall`** (`this` in `ECX`) |
| `53 8B D9` | `push ebx; mov ebx,ecx` — `__thiscall`, `this` → `EBX` |
| `81 EC NN NN 00 00` | `sub esp, 0xNNNN` — a large **stack frame** (heavy local math) |
| `6A FF 68 ?? ?? ?? 00` | `push -1; push <handler>` — an **SEH** scope (constructor with sub-objects) |

These signatures let you verify claims like "`Physics::Simulate` is a `__thiscall`" (its `8B F1` = `mov esi,ecx`),
"`IntegrateMotion` has a big math frame" (its `81 EC 30 05 00 00` = `sub esp,0x530`), or "`Physics_Base::ctor`
constructs sub-objects" (its `6A FF 68 …` SEH prologue). The prologue is a fingerprint — cheap to read, and
strongly identifying.

> ✅ *Verified:* prologue-based claims throughout the book — `Physics::Simulate 0x6BB4D0` = `56 8B F1 …`
> (`__thiscall`), `IntegrateMotion 0x6BA510` = `81 EC 30 05 00 00` (`sub esp,0x530`), `Physics_Base::ctor 0x6B9920`
> = `6A FF 68 12 DF 87 00` (SEH) — each read directly from the bytes.

## Following the call chain

Byte verification also *traces structure* — the `call` and `mov [reg+off]` instructions reveal how functions and
data connect ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)):

- **`call <target>`** — a direct call names the callee's address, so you follow the chain (sim driver → step →
  `Simulate` → `IntegrateMotion`, [C39.1](../C39-Vehicle-Simulation/01-pipeline.md)).
- **`mov eax,[esi+0xEC]`** — a field access names an offset, so you recover struct layout (the part array at
  `[this+0xEC]`, [C39.3](../C39-Vehicle-Simulation/03-part-array.md); the spawn queue at `[esi+0x6C]`,
  [C49.1](../C49-Cops-Dispatch-Roadblocks/01-fleet-manager.md)).
- **`call [eax+0x4C]`** — a virtual dispatch names a vtable slot ([C41.4](../C41-Physics-RigidBody/04-simulate-thiscall.md)).

So reading bytes recovers not just *what a function is* but *what it calls and touches* — the call graph and the
struct offsets, all from the instructions. This is how the book anchored the sim pipeline, the collision-reads-wheels
fact, and the cop spawn queue: by decoding the exact instructions at the exact addresses.

## RE implications

- **File-offset = VA − `0x400000`** — byte verification needs only a raw read and x86 decoding, no framework.
- **Opcodes prove identity** — `fsqrt` (`D9 FA`) makes `Math::Sqrt` certain; defining opcodes constrain a
  function's meaning.
- **Prologues fingerprint functions** — `__thiscall` (`mov esi,ecx`), stack frames (`sub esp`), SEH (`push -1;
  push h`).
- **`call`/`mov [reg+off]` trace structure** — the call graph and struct offsets, read from the instructions.

---

### Key takeaways

- The strongest proof is **reading the machine code** — for `speed.exe`, **file-offset = VA − `0x400000`**, so any
  byte claim is reproducible with a four-line read + an opcode reference.
- The **gold standard** is opcodes that prove identity — `Math::Sqrt` is `fld; fsqrt; ret`, and the **`fsqrt`
  opcode leaves no doubt**.
- **Prologue signatures** fingerprint functions — `__thiscall` (`mov esi,ecx`), large stack frames (`sub
  esp,0xNNN`), SEH constructors (`push -1; push <handler>`).
- **`call` and `mov [reg+offset]`** instructions trace the **call graph** and **struct layout** — how the book
  recovered the sim pipeline and struct offsets.
- Byte verification is the book's **bedrock tier** — a claim backed by an opcode sequence is as solid as RE gets.

**Continue:** [C50.3 — Hash verification](03-hash-verification.md) · [Chapter 50 hub](C50-Verification-Methodology.md)
