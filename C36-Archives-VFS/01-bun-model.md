# C36.1 — The .BUN Bundle Model

> **The one-sentence version:** a `.BUN` bundle is an ordinary EAGL chunk file — verified, `GLOBALA.BUN` opens
> with `0xB3300000`, not a `'BUN '` magic — so the extension is a convention and the universal chunk tools open
> a bundle directly.

[← Chapter 36 hub](C36-Archives-VFS.md) · [Next: C36.2 — The virtual file system →](02-vfs.md)

---

## No archive layer

The shipped `.BUN` files that fill `GLOBAL\`, `TRACKS\`, `FRONTEND\`, and the per-car `CARS\` folders are, each
one, an **EAGL chunk file** ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — the same
`{id, size}` chunk tree as a TPK ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)), a SolidList
([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)), or a track file
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)). There is **no separate archive format** wrapping
the chunks — a bundle *is* the chunks.

Verified: `GLOBAL/GLOBALA.BUN` opens with bytes `00 00 30 B3` = the chunk id `0xB3300000` — an EAGL container —
**not** a `'BUN '` FourCC. So a bundle announces itself as a chunk file, not as a bundle.

## The extension is a convention

`.BUN` is a **naming convention**, not a magic number:

- **No `'BUN '` FourCC.** A bundle has no bundle-specific header tag; it's identified by *being an EAGL chunk
  file at a bundle path*, not by a signature.
- **The chunk id is the identity.** The first chunk (`0xB3300000` for GLOBALA, or whatever top-level container)
  tells you what the file is — a TPK, a SolidList, a mixed bundle.
- **`.BIN`/`.lzc` siblings** are the same idea under other extensions — the extension hints at content/
  compression, not format.

So "is this a bundle?" isn't answered by a magic check but by "is this an EAGL chunk file the VFS serves?" The
practical upshot: use "bundle" loosely for the `.BUN`/`.BIN`/`.lzc` files, and treat them all as chunk files.

> ✅ *Verified:* `GLOBALA.BUN` is an EAGL chunk file (`0xB3300000`, not `'BUN '`); the extension is a convention
> with no FourCC magic.

## The universal tools just work

Because a bundle is an EAGL chunk file, everything from Chapter 1 applies directly
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)):

- **Walk it** with the chunk tree walker ([C1.3](../C1-EAGL-Container-Model/03-walking-the-tree.md)) — no
  archive layer to strip.
- **Find its contents** — the TPKs, SolidLists, and other chunks inside are the bundle's resources.
- **Edit it** with the size-tree discipline ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)) —
  a bundle edit is a chunk-file edit.

So opening a car bundle to reach its geometry ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)) and
textures ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) is just walking its chunks — which is exactly what
the geometry and texture chapters did. The bundle *is* the chunk file they parsed.

## Compression siblings

Some bundles are compressed — the `.lzc` files (like `GlobalB.lzc`, `FrontB.lzc`) are **JDLZ-compressed**
([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) bundles: decompress to reveal the EAGL chunk file
inside ([C3.5](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)). So the loading path tests for JDLZ first
([C3.5](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)), decompresses if needed, then walks the chunks. The
compression is a transport wrapper; the content is the same EAGL bundle.

## Editing implications

- **Edit a bundle as a chunk file** — walk, edit, size-fix ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)).
- **Don't look for a `'BUN '` magic** — there isn't one; check the top-level chunk id.
- **Decompress `.lzc` first** ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)) — it's a JDLZ wrapper
  over the bundle.
- **The VFS finds bundles by path-hash** ([C36.2](02-vfs.md)) — not by scanning for a magic.

---

### Key takeaways

- A `.BUN` bundle is an **EAGL chunk file** (verified `0xB3300000`, no `'BUN '` magic) — no archive layer.
- The **extension is a convention**; identity comes from the top-level chunk id, not a signature.
- The **universal chunk tools** (Chapter 1) open, walk, and edit a bundle directly.
- `.lzc` siblings are **JDLZ-compressed** bundles — decompress to reveal the same EAGL chunk file.
- Treat `.BUN`/`.BIN`/`.lzc` as chunk files; the VFS finds them by path-hash, not magic.

**Continue:** [C36.2 — The virtual file system](02-vfs.md) · [Chapter 36 hub](C36-Archives-VFS.md)
