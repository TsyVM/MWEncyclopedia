# C4.4 — Static Analysis: the Executable as Ground Truth

> **The one-sentence version:** `speed.exe` contains the exact code that reads every structure in the
> game; with a virtual-address-to-file-offset map and a disassembler you can read that code and settle any
> question the data alone leaves ambiguous.

[← C4.3 — Hex-diffing](03-hex-diffing.md) · [Chapter 4 hub](C4-Byte-Level-Toolcraft.md) ·
[Next: C4.5 — Validation harnesses →](05-validation.md)

---

## Why the executable is the authority

Every fact in this book ultimately answers to one source: the code that runs. A struct layout is whatever
the routine that reads it says it is; a hash is whatever the function that computes it does; a default is
whatever the immediate compiled into the branch holds. The shipped data is downstream of the code, and
community lore is downstream of both. When you can read the code, you are reading the specification with no
intermediary — which is why the hash function of
[C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md) is ✅ verified rather than merely reproduced:
it was read out of the instruction stream.

`speed.exe` is a 32-bit PE (`PE32`, Intel 80386) with a standard image base of `0x00400000` and these
sections:

```
.text    VA 0x00401000  vsize 0x48E2A5   (code)
.rdata   VA 0x00890000  vsize 0x05971D   (const data, vtables, strings)
.data    VA 0x008EA000  vsize 0x0DCE10   (globals, singletons)
.rsrc    VA 0x009C7000                   (resources)
```

Knowing where `.text`, `.rdata`, and `.data` live is half the battle: code lives in `.text`, vtables and
string literals in `.rdata`, and the named singletons and global flags in `.data`
([C35](../C35-Memory-Management/C35-Memory-Management.md)).

## The one piece of scaffolding you need: VA → file offset

Addresses in the disassembly are *virtual* addresses (VAs) — where the loader maps bytes at runtime. To
read those bytes out of the file on disk you convert a VA to a file offset using the section table: find
the section whose virtual range contains the VA, then rebase into its raw data.

```python
import pefile, capstone

class Exe:
    def __init__(self, path):
        self.pe = pefile.PE(path, fast_load=True)
        self.base = self.pe.OPTIONAL_HEADER.ImageBase
        self.data = self.pe.__data__
    def va2off(self, va):
        rva = va - self.base
        for s in self.pe.sections:
            size = max(s.Misc_VirtualSize, s.SizeOfRawData)
            if s.VirtualAddress <= rva < s.VirtualAddress + size:
                return s.PointerToRawData + (rva - s.VirtualAddress)
        return None
    def read(self, va, n):
        off = self.va2off(va); return self.data[off:off+n]
    def dis(self, va, n=64):
        md = capstone.Cs(capstone.CS_ARCH_X86, capstone.CS_MODE_32)
        for ins in md.disasm(bytes(self.read(va, n)), va):
            print(f"{ins.address:08x}: {ins.bytes.hex():<16} {ins.mnemonic} {ins.op_str}")
```

That is the entire bridge. `Exe('speed.exe').dis(0x5CC240)` prints the reflection-hash wrapper exactly as
[C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md) shows it, because the disassembly *is* the
source of that page.

## Patterns that pin a fact down

You do not need to read assembly fluently to extract facts — you need to recognise a handful of shapes:

- **A struct field access.** `mov eax, [esi + 0x10]` says "the object in `esi` has a dword at offset
  0x10." A cluster of such loads maps a struct's layout directly; the offsets are the field offsets, and
  the register widths (`movzx` for byte/word, `movss` for float) hint the types.
- **An array loop.** A loop that advances a pointer by a constant each iteration (`add esi, 0x40`) reveals
  a **record stride** — `0x40` here means 64-byte records, corroborating a data-derived stride
  ([C4.2](02-decoding-unknowns.md)).
- **A vtable.** A `.rdata` table of code pointers, loaded into an object's first dword at construction, is
  a vtable; its address identifies the class and its slots are the virtual methods
  ([C34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
- **A magic or seed immediate.** `mov edx, 0xABCDEF00` or `mov eax, 0xEEC2271A` — a constant moved into a
  register is often a seed, a sentinel, or a FourCC, and it is a load-bearing fact you can quote directly.
- **A known algorithm's signature.** The golden-ratio constant `0x9E3779B9` and a twelve-byte block loop
  are Jenkins lookup2; the `<<10 / >>6` per-byte loop is one-at-a-time. Recognising library algorithms by
  their constants is one of the highest-leverage skills.

## When to reach for it

Static analysis is the tie-breaker and the *why*-machine. Reach for it when:

- Two data-derived hypotheses both "fit" and you need the reader routine to decide
  ([C4.2](02-decoding-unknowns.md)).
- You need a value that never appears in data — a default baked into a branch, a hardcoded limit, a pool
  size ([C35](../C35-Memory-Management/C35-Memory-Management.md)).
- You want to know not just a field's offset but *what the game does with it* — which subsystem reads it,
  what it gates, whether it's even used.

It is more effort than hex-diffing for merely *locating* a field, so use diffing to locate and
disassembly to *explain* — they are complementary, not competing.

## Bending it — reading code responsibly

- **Quote addresses so others can check.** A disassembly claim without its VA is unverifiable; with it,
  anyone can reproduce ([C4.5](05-validation.md)).
- **Corroborate code against data.** The strongest facts are where a code-derived offset and a data-derived
  stride agree; when they do, mark it ✅. When only one is available, mark accordingly.
- **Don't over-read a single instruction.** One `mov` is a data point; a *cluster* of consistent accesses
  is a layout. Build the picture from several confirming reads, not one suggestive line.
- **Mind the difference between "present" and "used."** Code that references a field proves the field
  exists; it takes reading the surrounding logic to know whether that path is live in retail play (some
  mapped systems are dormant — [C74](../C74-Multiplayer-Online/C74-Multiplayer-Online.md)).

---

**Continue:** [C4.5 — Validation harnesses](05-validation.md) · [Chapter 4 hub](C4-Byte-Level-Toolcraft.md)
