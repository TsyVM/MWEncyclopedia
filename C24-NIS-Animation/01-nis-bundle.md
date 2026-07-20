# C24.1 — The NIS Bundle & the ELF Payload

> **The one-sentence version:** a NIS cutscene is an EAGL bundle holding textures and several animation
> payloads, and each animation payload (chunk `0x00E34009`) is an 8-byte `0x11` sentinel followed by a
> little-endian MIPS ELF32 object — a compiler artifact, not a bespoke struct.

[← Chapter 24 hub](C24-NIS-Animation.md) · [Next: C24.2 — Parsing the MIPS ELF32 →](02-parsing-elf.md)

---

## The bundle

A NIS scene file (`Scene_ArrestF02_BundleB.bun`) is an ordinary **EAGL container**
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — verified magic `0xB3300000` — holding a
mix of chunks:

```
Scene bundle (0xB3300000 …)
├── TPKs (0xB3300000 / 0xB3310000 / 0xB3320000)   textures for the scene (Chapter 5)
├── 0x00E34009  animation payload   ← MIPS ELF32 object (×3 in this bundle)
├── __AnimationBank ELF             keyframe banks (C24.4)
└── 0x00134C02 …                    scene data
```

So a cutscene bundles everything the scene needs: its textures ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)),
its skeletons, and its animation banks. The chunk that makes NIS unusual is `0x00E34009` — the animation
payload.

## The 0x00E34009 payload

Verified on the retail bundle, the `0x00E34009` chunk at `0xEE340` contains:

```
+0x00  11 11 11 11 11 11 11 11   8-byte 0x11 sentinel (MW's fill marker)
+0x08  7F 45 4C 46 01 01 01 00   "\x7fELF", ELFCLASS32, ELFDATA2LSB …
+0x0C  …                          the rest of a MIPS ELF32 relocatable object
```

After the 8-byte sentinel — the same `0x11` fill marker seen in geometry buffers
([C9.1](../C9-Meshes-FVF/01-vertex-buffer.md)) and CARP ([C18.1](../C18-Road-Network-CARP/01-carp-format.md)) —
the payload is a genuine **ELF object file**: `\x7fELF`, 32-bit (`01`), little-endian (`01`). This bundle holds
**three** such payloads.

## Why an ELF object is remarkable

Almost every format in this book is a custom EA layout. This one is a **standard compiler artifact** — a
relocatable object file, the kind `gcc`/`ld` produce. That's a genuine surprise, and it tells you how EA's
animation tool worked: it *compiled* rigs and animation into object files, presumably to link them into the
game's runtime the way code is linked. The practical consequence is wonderful — you parse it with **ELF
tooling** (or the ELF knowledge everyone already has), not a reverse-engineered reader
([C24.2](02-parsing-elf.md)).

> ✅ *Verified:* the NIS bundle is EAGL (`0xB3300000`); the `0x00E34009` payload is an 8-byte `0x11` sentinel +
> `\x7fELF` MIPS ELF32; a `__AnimationBank` ELF also appears in the bundle.

## MIPS, and why it matters

The ELF's target architecture is **MIPS** — the CPU family of the PlayStation 2 (and PSP). This is a strong
clue to the format's origin and to the PS2/PC split ([C24.6](06-ps2-vs-pc.md)): the animation objects were
built for the MIPS-based console, and the PC build carries the same objects even though it renders the scenes
differently. The MIPS target also means any relocations/addresses in the object are MIPS ones — you read the
data and symbols, not execute the code.

## Reading the payload

```python
def get_nis_elf(bundle):
    for chunk in walk_eagl(bundle):
        if chunk.id == 0x00E34009:
            body = chunk.payload[8:]         # skip the 8-byte 0x11 sentinel
            assert body[:4] == b"\x7fELF"
            yield Elf32(body)                # parse as ELF (C24.2)
```

Skip the sentinel, confirm the ELF magic, and hand the rest to an ELF parser. From there, the skeleton comes
out of the symbol table ([C24.3](03-skeleton.md)).

## Editing implications

- **Treat it as an ELF, not bytes.** Edits go through ELF structure (sections, symbols), not raw offsets
  ([C24.2](02-parsing-elf.md)).
- **Preserve the sentinel.** The 8-byte `0x11` prefix is part of the chunk; keep it.
- **Skeletons and keyframes are separate objects.** Editing the rig (`0x00E34009`) is distinct from editing the
  motion (`__AnimationBank`, [C24.4](04-skeletons-banks.md)).
- **Mind the frontier.** The keyframes are not fully decoded ([C24.5](05-keyframe-problem.md)) — skeleton edits
  are more tractable than motion edits.

---

### Key takeaways

- A NIS cutscene is an **EAGL bundle** with textures, animation payloads, and keyframe banks.
- The animation payload (`0x00E34009`) is an 8-byte `0x11` sentinel + a **MIPS ELF32** relocatable object —
  verified.
- It's a **standard compiler artifact**, so you parse it with ELF tooling, not a bespoke reader.
- The **MIPS** target ties it to the PS2 lineage and the PS2/PC rendering split (C24.6).
- Skip the sentinel, parse as ELF, and recover the skeleton from the symbol table; keyframes are a separate,
  harder object.

**Continue:** [C24.2 — Parsing the MIPS ELF32](02-parsing-elf.md) · [Chapter 24 hub](C24-NIS-Animation.md)
