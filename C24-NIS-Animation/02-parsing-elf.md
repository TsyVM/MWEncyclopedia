# C24.2 — Parsing the MIPS ELF32

> **The one-sentence version:** the payload is a genuine ELF32, so you read it the standard way — ELF header →
> section headers → symbol table (`.symtab`) → the `.data` section — and the `EAGL4::SymbolPool` symbols
> confirm it came from EA's animation toolchain.

[← C24.1 — The NIS bundle & the ELF payload](01-nis-bundle.md) · [Chapter 24 hub](C24-NIS-Animation.md) ·
[Next: C24.3 — The bind-pose skeleton →](03-skeleton.md)

---

## Standard ELF, standard parse

Because the payload is a real ELF32 object ([C24.1](01-nis-bundle.md)), there is no reverse-engineering to do
on the *container* — you parse it exactly like any relocatable object:

```python
def parse_elf32(body):                         # body = payload after the 8-byte sentinel
    assert body[:4] == b"\x7fELF"
    ei_class, ei_data = body[4], body[5]       # 01 = ELFCLASS32, 01 = ELFDATA2LSB (little-endian)
    e_shoff   = u32(body, 0x20)                # section header table offset
    e_shentsize, e_shnum = u16(body, 0x2E), u16(body, 0x30)
    e_shstrndx = u16(body, 0x32)               # section-name string table index
    sections = [read_shdr(body, e_shoff + i*e_shentsize) for i in range(e_shnum)]
    return Elf32(body, sections)
```

The ELF header gives you the section table; the section table names each section (`.symtab`, `.strtab`,
`.data`, `.text`, `.rel*`); the symbol table names the bones and skeletons ([C24.3](03-skeleton.md)); the data
section holds their values. This is bread-and-butter ELF parsing — the payoff of EA having emitted an object
file instead of a custom blob.

## The sections that matter

For animation, three sections carry the content:

- **`.symtab`** (+ `.strtab`) — the **symbol table**: names every bone and skeleton and points each at its data
  (offset + size). This is where the rig lives ([C24.3](03-skeleton.md)).
- **`.data`** — the **payload**: the actual bind-pose transforms and (in the animation-bank ELFs) the keyframe
  data ([C24.4](04-skeletons-banks.md)).
- **`.rel` / `.rela`** — **relocations**: how symbol references are patched when the object is "linked" into the
  runtime. You mostly read through these rather than apply them.

Walking `.symtab`, resolving each symbol's name via `.strtab`, and reading its data from `.data` reconstructs
the rig.

## EAGL4::SymbolPool — the toolchain fingerprint

The symbols carry a telltale name: **`EAGL4::SymbolPool`**. This is the fingerprint of EA's EAGL animation
toolchain — the object was produced by EA's tools, which used an ELF `SymbolPool` to name and index animation
data. Seeing `EAGL4::SymbolPool` in the symbol table confirms both the format's origin and that you're reading
the right structure ([C24.1](01-nis-bundle.md)).

> ✅ *Verified:* the payload is `\x7fELF` ELFCLASS32/LSB; the archive confirms the standard ELF parse (header →
> sections → `.symtab` → `.data`) and the `EAGL4::SymbolPool` toolchain evidence.
> 🟡 *Reasoned:* the exact per-symbol data layout of the animation values is read from `.data` per symbol; the
> ELF structure and `SymbolPool` evidence are verified.

## Why ELF was a smart choice for EA

Emitting animation as ELF objects, odd as it looks, was pragmatic:

- **Symbols name the rig for free.** An ELF symbol table is *exactly* "name every entity and point it at its
  data" — a perfect fit for "name every bone/skeleton" ([C24.3](03-skeleton.md)). No custom index format
  needed.
- **Standard toolchain.** EA could use the compiler/linker toolchain (assemble, relocate, link) to build and
  combine animation objects, rather than maintaining a bespoke packer.
- **Relocatable = linkable.** As relocatable objects, animation data could be "linked" into the runtime like
  code, with references resolved at load.

So the ELF form isn't an accident — it reuses the toolchain's naming and linking for animation data.

## Editing implications

- **Use an ELF library.** Read/write via standard ELF tooling; don't poke raw offsets
  ([C24.1](01-nis-bundle.md)).
- **Resolve symbols via `.strtab`.** Bone/skeleton names come from the string table referenced by `.symtab`.
- **Read data per symbol.** Each symbol's `st_value`/`st_size` locate its data in `.data`.
- **Respect relocations.** If you move or add data, relocations referencing it must stay consistent — another
  reason to edit through ELF tooling rather than bytes.

---

### Key takeaways

- The payload is a standard **ELF32** (LSB) — parse it header → sections → `.symtab` → `.data`, no custom reader.
- `.symtab` (+ `.strtab`) names bones/skeletons; `.data` holds their values; `.rel*` are relocations.
- **`EAGL4::SymbolPool`** in the symbols is EA's animation-toolchain fingerprint, confirming the format.
- ELF was smart for EA: symbols name the rig for free, and relocatable objects link into the runtime like code.
- Edit through an ELF library, resolve names via `.strtab`, read data per symbol, and keep relocations
  consistent.

**Continue:** [C24.3 — The bind-pose skeleton](03-skeleton.md) · [Chapter 24 hub](C24-NIS-Animation.md)
