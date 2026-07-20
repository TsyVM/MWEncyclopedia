# C58.5 — Reading the Build & the Book's Method

> **The one-sentence version:** the executable's own build metadata (PE32, MSVC 7.1, 2005-12-01, EAGL/Black Box)
> is the frame around the whole book — and reading it closes the encyclopedia: 58 chapters that reverse-engineered
> a shipped 2005 game from its bytes, verification-first, from the chunk container to the build pipeline.

[← C58.4 — The asset pipeline](04-asset-pipeline.md) · [Chapter 58 hub](C58-Build-Pipeline.md)

---

## Anchors for build RE

The build is anchored on verified metadata and strings:

- **The PE header** — PE32 x86, MSVC linker 7.10, timestamp 2005-12-01, 5 sections
  ([C58.1](01-shipping-exe.md)).
- **The engine lineage** — `EAGL`, `BlackBox`, `nfs_mostwanted` ([C58.2](02-eagl-engine.md)).
- **The bundle format** — `BCHUNK_`, the chunk container, `GLOBAL`/`GLOBALB`, JDLZ/LZC
  ([C58.3](03-bundle-pipeline.md)).
- **The pipeline** — authoring → pack → compress → stream ([C58.4](04-asset-pipeline.md)).

From these, the build is navigable: the executable, the engine, the format, and the pipeline.

## Reading the build first (and last)

The build metadata is what you read *first* in any RE ([C58.1](01-shipping-exe.md)) — and, fittingly, *last* in
this book. First, because the PE header ([C58.1](01-shipping-exe.md)) frames everything: it tells you the platform,
compiler, and version, so every subsequent address ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md))
is grounded in *this specific build*. Last, because understanding the *pipeline* ([C58.4](04-asset-pipeline.md)) and
*engine* ([C58.2](02-eagl-engine.md)) requires having decoded the *content* first — you appreciate the pipeline
only once you know what it produces. So the build chapter bookends the RE: the header orients you at the start, and
the pipeline contextualises you at the end. Reading the build is both the *entry* and the *conclusion* of
understanding a shipped game.

## The whole book in one arc

This chapter completes the encyclopedia's arc — 58 chapters that decoded Most Wanted from its bytes:

- **Part I–II — Formats** ([Chapters 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)–[14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md))
  — the chunk container, compression, textures, geometry, and the vault: *the data*.
- **Part III–VI — Content** ([Chapters 15](../C15-Track-Streaming/C15-Track-Streaming.md)–[31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md))
  — the world, audio, video, animation, UI, and save: *the assets*.
- **Part VII — Substrate** ([Chapters 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)–[38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md))
  — the class system, memory, VFS, frame, and streaming: *the machine*.
- **Part VIII — Simulation** ([Chapters 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
  — the vehicle, physics, collision, AI, and pursuit: *the gameplay*.
- **Part IX — Presentation & Structure** ([Chapters 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)–58)
  — the method, renderer, effects, cameras, career, events, customization, world, and build: *the experience and
  the making*.

So the book runs from the *smallest unit* (a `{id, size}` chunk, [Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md))
to the *largest context* (the build pipeline, this chapter) — a complete traversal of Most Wanted, from its data
format to its production. Every chapter verified its claims against `speed.exe` and `attributes.bin`
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)), building a *textbook* — not lore,
but a reference grounded in the bytes.

## The method, proven

The book's thesis ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) — *verification-first
RE* — is proven by the whole. Across 58 chapters, the same discipline held:

- **Every claim reduced to a check** — a byte pattern ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)),
  a hash ([C50.3](../C50-Verification-Methodology/03-hash-verification.md)), a vtable count
  ([C50.4](../C50-Verification-Methodology/04-vtable-verification.md)), or a constant.
- **Confidence was marked** — ✅ verified, 🟡 reasoned, ⚪ open ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md))
  — never letting inference pass as fact.
- **Received wisdom was tested** — and corrected where wrong ([C50.5](../C50-Verification-Methodology/05-cross-checking.md),
  the GIN offsets, the TPK hash).
- **The engine's patterns emerged** — data-over-code
  ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)), composition
  ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)), reusable architectures
  ([C53.3](../C53-Cameras-Director/03-cinematic-director.md)), pools
  ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — recognised once, then everywhere.

So the method scaled from one function ([C41.5](../C41-Physics-RigidBody/05-integrate-math.md), `Math::Sqrt`) to a
whole game — which is the deepest lesson of the book. Reverse-engineering isn't guessing; it's *reducing claims to
checks against the artifact*, marking what you can't check, and letting the patterns reveal the architecture. That
discipline, applied 58 times, turned 4.6 MB of anonymous code ([C58.1](01-shipping-exe.md)) into a legible,
verified account of how Most Wanted works.

## What the book leaves you with

Beyond the specific facts, the encyclopedia leaves a *way of seeing*:

- **A shipped game is knowable.** With the artifact (the executable and its data) and the discipline
  ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)), even a closed-source 2005 game
  is *fully legible* — its formats, its classes, its systems, its build.
- **Great engines are coherent.** MW's EAGL ([C58.2](02-eagl-engine.md)) reuses a few good ideas
  ([C53.5](../C53-Cameras-Director/05-reading-cameras.md)) — data-over-code, composition, pools, connectors —
  *everywhere*, so learning one system teaches many. Coherence is the mark of maturity.
- **The method transfers.** The techniques ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md))
  — byte, hash, vtable, cross-check — work on *any* binary. This book is a worked example of RE as a practice, not
  just a MW reference.

So the encyclopedia ends where it began — at the executable ([C58.1](01-shipping-exe.md)) — but now *understood*:
from the build metadata that frames it, through the 4.6 MB of code and the compressed bundles, to the game it runs.
Most Wanted, built by EA Black Box on EAGL in 2005, reverse-engineered verification-first across 58 chapters — a
complete, grounded account of how a great game works, and a demonstration of how to *know* a binary. The bytes were
the source; the discipline was the method; the understanding is the result.

## RE implications

- **The build metadata frames the whole RE** — read first (to orient), understood last (to contextualise).
- **The book runs from chunk to pipeline** — the smallest data unit to the largest production context.
- **The method is proven** — verification-first RE scaled from one function to a whole game.
- **The way of seeing transfers** — a shipped game is knowable; great engines are coherent; the method works on any
  binary.

---

### Key takeaways

- The **build metadata** (PE32, MSVC 7.1, 2005-12-01, EAGL/Black Box) is the **frame** around the book — read
  *first* to orient the RE, understood *last* to contextualise the pipeline.
- The encyclopedia runs a **complete arc** — from the smallest unit (a `{id, size}` chunk,
  [Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) to the largest context (the build pipeline) —
  formats → content → substrate → simulation → presentation → build.
- The **method is proven** across 58 chapters — every claim reduced to a **check**, confidence **marked**, received
  wisdom **corrected**, and the engine's **patterns** recognised everywhere.
- The book leaves a **way of seeing** — a shipped game is **knowable**, great engines are **coherent** (reuse a few
  ideas everywhere), and the **method transfers** to any binary.
- Most Wanted — built by **EA Black Box on EAGL in 2005** — is now **fully, verifiably legible**: the bytes were the
  source, the discipline the method, the understanding the result.

**This completes the MW05 Encyclopedia** — 58 chapters, verification-first, from the chunk container to the build
pipeline.

**Sources:** `speed.exe` (verified: PE32 x86, MSVC linker 7.10, timestamp 2005-12-01 01:06:20 UTC, ImageBase
`0x400000`, 5 sections; `EAGL`, `BlackBox`, `nfs_mostwanted`/`nfsmw`; `BCHUNK_`/`BCHUNK_NULL`, `GLOBAL`/`GLOBALB`).
