# Chapter 58 — The Build: Toolchain, Bundles & Asset Pipeline

> **Goal of this chapter:** decode how Most Wanted was *made* — the shipping executable (PE32 x86, MSVC linker
> 7.10, built 2005-12-01), the EA Black Box **EAGL** engine it's built on, the **BCHUNK** bundle pipeline that
> packs the assets, and the authoring→pack→compress→stream content pipeline — the industry-development side of the
> game.

Every earlier chapter decoded a *runtime* system; this one steps back to the *development* side — how the game was
*built and shipped*. It decodes the executable's own build metadata (compiler, date, structure), the EAGL engine
lineage, the bundle/chunk pipeline that turns authored assets into the shipped data files, and the whole content
pipeline from art tools to the streaming disc. It's the capstone: after 57 chapters on *what the game is*, this is
*how it came to be* — the toolchain and pipeline behind Most Wanted.

> **Verified against the executable.** `speed.exe` is a **PE32 x86** image (machine `0x14C`, PE32 magic `0x10B`),
> **linked with MSVC linker 7.10** (Visual Studio .NET 2003), **built 2005-12-01 01:06:20 UTC** (PE timestamp
> `0x4..` = 1133399180). It has **5 sections**: `.text` (code, VA `0x401000`, ~4.6 MB), `.rdata` (`0x890000`),
> `.data` (`0x8EA000`), `.rsrc` (`0x9C7000`), and a trailing section (`0xA38000`). Verified strings name the
> lineage: **`EAGL`** (the engine), **`BlackBox`** (the studio), **`nfs_mostwanted`**/`nfsmw` (the project), and
> the **`BCHUNK_`** bundle-chunk system (`BCHUNK_NULL`), with the **`GLOBAL`**/`GLOBALB` bundle.

---

## Deep-dive pages

- [C58.1 — The shipping executable](01-shipping-exe.md): PE32, MSVC 7.1, the build date and sections.
- [C58.2 — The EAGL engine](02-eagl-engine.md): EA Black Box's engine and its lineage.
- [C58.3 — The bundle pipeline](03-bundle-pipeline.md): BCHUNK chunks, bundles, and compression.
- [C58.4 — The asset pipeline](04-asset-pipeline.md): authoring → pack → compress → stream.
- [C58.5 — Reading the build & the book's method](05-reading-build.md): PE analysis and a retrospective.

---

## 58.1 The shipping executable

`speed.exe` carries its own build metadata ([C58.1](01-shipping-exe.md)): it's a **PE32 x86** image, **linked with
MSVC 7.10** (Visual Studio .NET 2003 — the compiler of its era), and **built 2005-12-01** — the retail v1.3 build,
a month after the game's launch. Its **5 sections** (`.text` code, `.rdata` read-only, `.data`, `.rsrc` resources,
plus a trailing section) are the anatomy the whole book has read — `.text` for the code
([Chapter 50](../C50-Verification-Methodology/02-byte-verification.md)), `.rdata` for the strings and vtables
([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).

## 58.2 The EAGL engine

Most Wanted is built on **EAGL** ([C58.2](02-eagl-engine.md)) — EA's graphics/game library, the engine of **EA
Black Box** (the studio, verified string `BlackBox`). The project codename is **`nfs_mostwanted`**/`nfsmw`. EAGL is
a *shared* engine across EA titles ([C41.6](../C41-Physics-RigidBody/06-vehicle-types.md) noted the vehicle-class
breadth; [C57.3](../C57-World-Systems/03-weather-rain.md) the weather support) — MW is one *configuration* of a
broader technology, which is why the executable carries capabilities beyond what MW uses.

## 58.3 The bundle pipeline

The game's assets ship as **bundles** of **chunks** ([C58.3](03-bundle-pipeline.md)) — the verified `BCHUNK_`
system (`BCHUNK_NULL`) built on the EAGL chunk container ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)).
A bundle (like `GLOBAL`/`GLOBALB`) is a tree of `{id, size}` chunks ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md))
holding textures, geometry, vault data — compressed with JDLZ/LZC
([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)). This is the *shipping format* — the container the
whole book has been parsing.

## 58.4 The asset pipeline

Behind the bundles is the **content pipeline** ([C58.4](04-asset-pipeline.md)): artists and designers author assets
in tools (models, textures, the vault, [Chapter 56](../C56-Customization/C56-Customization.md)), which are *packed*
into chunks ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)), *compressed*
([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)), and *streamed* at runtime
([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)). This authoring →
pack → compress → stream chain is how authored content becomes the shipped, streamed game — the pipeline the whole
runtime consumes.

---

### Key takeaways

- `speed.exe` is a **PE32 x86** image, **linked with MSVC 7.10** (Visual Studio .NET 2003), **built 2005-12-01**
  (retail v1.3) — 5 sections (`.text`/`.rdata`/`.data`/`.rsrc`/trailing).
- It's built on **EAGL** — EA **Black Box**'s shared engine (project `nfs_mostwanted`) — MW is one *configuration*
  of a broader technology.
- Assets ship as **bundles of `BCHUNK` chunks** (the EAGL chunk container,
  [Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — `{id, size}` trees, JDLZ/LZC-compressed
  ([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)).
- The **content pipeline** is authoring → pack → compress → stream — how authored assets become the shipped,
  streamed game.
- This chapter is the **development-side capstone** — after 57 chapters on what the game *is*, this is how it was
  *built and shipped*.

**This completes the encyclopedia** — see [C58.5](05-reading-build.md) for the retrospective on the whole book's
method.
