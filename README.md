# The Most Wanted Encyclopedia

### A complete engine-level reference for *Need for Speed: Most Wanted* (2005, PC — retail v1.3)

This is a self-contained, byte-level and runtime-level guide to **every** file format, data
structure, class, and subsystem in the game built on EA Black Box's **EAGL** engine. It exists so
that a person with a hex editor, a disassembler, and patience can understand the game the way its
authors did — and change it with confidence, from swapping a single texture to rebuilding geometry,
retuning the pursuit AI, replacing the soundtrack, or tracing a value from a menu screen all the way
down to the float it reads out of a vault record.

Everything here is grounded in the retail PC data set and in the code of the shipped executable. It
is written **format-first and mechanism-first**: byte layouts, the algorithms that read and write
them, the runtime classes that consume them, worked examples in portable C++/Python, and — always —
the *reasoning* behind why each thing is built the way it is. Code is deliberately
**toolkit-agnostic**: it reads and writes raw bytes so it works in any language or environment, and
so the knowledge outlives any single program.

This edition is a ground-up rewrite of the original encyclopedia. It is deeper, wider, and more
precise: the old omnibus chapters have been split so that each mechanism gets the room it deserves,
the runtime class model is now first-class material rather than an appendix, and every finding that
was previously stated as a bare fact now carries its evidence and its confidence.

---

## How to read this book

The work is split into a **Glossary** (terminology, the master chunk-ID table, an extension index,
and a file-by-file catalogue of the entire game) followed by **78 chapters** that build from first
principles — a single chunk header — all the way up to the running game observed frame by frame.

> **Every chapter is a hub plus focused deep-dive pages.** The chapter file (e.g.
> `C11-Attribute-Vaults/C11-Attribute-Vaults.md`) is the overview and the map; inside the same
> folder, numbered pages (`01-…md`, `02-…md`, …) each take a *single* mechanism and answer four
> questions about it: **what it is, how it works, why it's built that way, and what happens if you
> bend it — the right way or the wrong way.** A chapter runs 8–12 such pages. The hub links to its
> deep-dive pages near the top.

### Confidence markers

Because much of this is reverse-engineered, every non-obvious claim is tagged so you always know how
much weight it bears:

- ✅ **Verified** — confirmed against retail bytes or the shipped executable's code; reproducible.
- 🟡 **Reasoned** — a well-supported inference about *intent* or *mechanism* that fits all observed
  evidence but is not a direct byte-for-byte proof.
- ⏳ **Open** — known to exist, not yet fully decoded; the boundary of current knowledge is stated
  explicitly rather than hidden.

A claim with no marker is either self-evidently structural (a stated struct offset you can see in a
hex editor) or established earlier in the same chapter.

### A note on evidence and provenance

Findings that describe the *running* game — singleton addresses, object sizes, interned string
tables, vtable layouts — are presented here as **properties of the engine**: facts about
`speed.exe` and the data it loads, established by static disassembly and by observing the process as
it runs. Where an address or a size matters, it is given; where a layout is only partially recovered,
that is said plainly. The emphasis throughout is on the *fact* and its *confidence*, not on tooling.

---

## Glossary

| Page | Contents |
|---|---|
| [Glossary/README.md](Glossary/README.md) | How the glossary is organised; the identification workflow |
| [Glossary/terminology.md](Glossary/terminology.md) | Every acronym and concept: EAGL, chunk, container bit, JDLZ, Joaat, Bin hash, FVF, DXT, TPK, VPAK, FourCC, mip, ADPCM, vtable, singleton, reflection schema… |
| [Glossary/chunk-ids.md](Glossary/chunk-ids.md) | Master table of every known chunk identifier, its container/leaf role, its payload layout, and where it appears |
| [Glossary/extensions.md](Glossary/extensions.md) | Extension → format → chapter map, plus the "identify an unknown file" decision tree and a portable identifier |
| **File catalogue** | A file-by-file listing of the entire retail data set (built out chapter-group by chapter-group): `files-TRACKS`, `files-GLOBAL`, `files-CARS`, `files-SOUND`, `files-NIS`, `files-MOVIES`, `files-FRONTEND`, `files-LANGUAGES`, `files-CREDITS`, `files-MEMCARD`, `files-SUBTITLES`, `files-ROOT` |

---

## Chapters

### Part I — Foundations & the container model

Everything downstream depends on Part I. Read C1–C4 in order; the rest of the book assumes them.

1. [C1 — The EAGL Container Model](C1-EAGL-Container-Model/C1-EAGL-Container-Model.md): chunks, the container bit, the size tree, alignment, endianness, the coordinate system, the non-chunk container models, and how to walk, dump, edit, and repack *any* file.
2. **C2 — Identifiers & Hashing**: Joaat (Jenkins one-at-a-time) and the Bin `33·h+c` hash, why names are stored as 32-bit numbers, collision handling, and name recovery from dictionaries.
3. **C3 — Compression (JDLZ / `.lzc`)**: the two-flag-stream LZ77 variant, a complete decompressor and compressor, and the compression boundary every opener must test first.
4. **C4 — Byte-Level Toolcraft**: bounded readers/writers, tree dumpers, hex-diffing, the disassembly-and-analysis workflow, and how to build a reader for a format nobody has documented.

### Part II — Textures & geometry

5. **C5 — Textures: the TPK Container Model**: both TPK variants (standard 124-byte entries vs. compressed 24-byte descriptors), the metadata tables, extraction and replacement.
6. **C6 — Texture Codecs**: DXT1/3/5 (BC1/2/3) block math, ARGB32, PAL8 palettes, the DDS interchange container, and mip chains.
7. **C7 — Materials, Texture References & Animation**: how solids bind textures, the light/material chunk, texture-usage strings, and the texture-animation block.
8. **C8 — 3D Geometry: Solid Lists & Objects**: the geometry container, the object header (name-hash, tri count, bbox, transform), and hash tables.
9. **C9 — Meshes, FVF & Vertex Formats**: shading groups, the flexible vertex format flags, strides (24/36/60), group-relative index buffers, and how a solid becomes triangles.
10. **C10 — Geometry Import/Export & Mesh Rebuilding**: exporting to OBJ/glTF at the Y-up boundary, re-importing, rebuilding vertex/index buffers, and the size-tree consequences.

### Part III — Attribute vaults

11. **C11 — Attribute Vaults: VPAK Structure**: the `VPAK` container, the `ErtS`/`NpeD`/`NrtS`/`NtaD` blocks, the string table, and typed records.
12. **C12 — The Reflection Schema & Resolved-Value Model**: the compiled field-name → offset → type map, the inline `{field, f32, type}` triple model, and `default` inheritance — the key to reading *and writing* vault records.
13. **C13 — Vault Categories: Car Tuning**: the per-car performance schema, how a tuning value maps to a simulation knob, and the UI performance bars.
14. **C14 — Vault Categories: Pursuit, Surfaces & Gameplay**: the pursuit/AI vault, surface/effect/destructible records, and the `FE_ATTRIB`/`gameplay` vaults.

### Part IV — The world

15. **C15 — The Master Track File & Streaming Sections**: the streaming table (92-byte section entries), section residency, and the world's top-level layout.
16. **C16 — Scenery, Props & the Cull Tree**: model definitions, 64-byte instance placements, the decoded 36-byte AABB cull tree, and scenery groups.
17. **C17 — Trigger Regions & Barriers**: typed 2-D world-plane volumes, the even-odd runtime test, barriers, and the events they fire.
18. **C18 — The Road Network (CARP)**: the AI/GPS graph — 32-byte nodes, 22-byte segments, the cost grid — and how cops and GPS route on it.

### Part V — Audio, video & animation

19. **C19 — Audio: Banks (SNR/SPT/ABK)**: bank routing and payload, the big-endian `PT` TLV records, and the SFX vocabulary.
20. **C20 — Audio Codecs**: EA-XA v2, EA-XAS v0, IMA-ADPCM, and EA-MP3 — the decode math and a portable decoder.
21. **C21 — Music (MUS/MPF) & the Routing Graph**: the `SCHl`/`SCDl`/`SCEl` stream blocks and the PathFinder music graph.
22. **C22 — Engine Sound (GIN) & the RPM→Synth Bridge**: the RPM-curve audio format and the runtime engine-synth pipeline.
23. **C23 — Video (EA Multimedia / VP6)**: the container, the On2 VP6 codec, EA-ADPCM audio, transcoding, and the movie player as a flow state.
24. **C24 — Animations & Cutscenes: the NIS Object**: the MIPS-ELF animation object, skeletons, and the keyframe quantisation problem.
25. **C25 — NIS Event Timelines, Scripts & Playback**: the `ENIS*` verb vocabulary, the decoded `EventSequenceChunk` script format, and the three-source playback assembly.
26. **C26 — World-Ambient & Gameplay Animation Banks**: the other users of the animation-bank format — the world-ambient banks (cranes, ships, blimp) and gameplay animation.

### Part VI — Front-end, text & save data

27. **C27 — Front-End: Shell Scenes & UI Atlases**: the menu scene assets, HUD layout, and how the shell is drawn.
28. **C28 — Fonts & Glyph Tables**: font atlases, glyph tables, and text rendering.
29. **C29 — Minimap: Tiles & Map Data**: JDLZ-wrapped minimap TPK tiles, `TrackMaps.bin`, and the map overlay.
30. **C30 — Localization: String Tables & the Label System**: per-language `Labels.bin`/`English.bin`… tables and the ID-based label lookup.
31. **C31 — Save Data & Memory-Card Containers**: the `LOCH`/`LOCI` `.loc` containers and the career save payload.

### Part VII — The runtime class system & substrate

The book pivots here from *files on disk* to the *running game*: the class model that ties the
formats together, and the substrates every subsystem sits on.

32. **C32 — The Runtime Class System & Object Model**: the class roles, the object model, and how loaded data becomes live objects.
33. **C33 — The Class Registry, Factories & the Class Reference**: the runtime registry, object construction, and a family-by-family catalogue of the game's runtime classes (verified sizes, vtable addresses, method counts).
34. **C34 — VTable Anatomy & Method Roles**: how to read a vtable, classify its slots (method/getter/thunk/stub/destructor), and identify a class from its vtable alone.
35. **C35 — Memory Management & Allocation**: the real allocator vs. the demoted stub, the `SlotPool` family, particle pools, the pre-sized A* pools, and the debug-fill sentinels that let you fingerprint globals.
36. **C36 — Archives & the Virtual File System**: the `.BUN` bundle model, the path→BinHash VFS, the `MemoryFile` intercept, the refcounted `StreamMgr`, and the A/B/C bundle loading scheme.
37. **C37 — The Frame Spine & Engine Modules**: `Engine::PumpFrame`/`FrameTick`, the module update order, and where each subsystem plugs into the frame.
38. **C38 — The Resource Streaming Manager & Residency**: GameFlow phases, the four preload manifests, refcounted residency, and load-time budgets.

### Part VIII — Simulation & physics

39. **C39 — Vehicle Simulation, End to End**: the per-frame vehicle pipeline from input to tyres, and the one-way connector boundary.
40. **C40 — The Eight Vehicle Mechanics**: the mechanic component model — engine, transmission, induction, drivetrain, suspension, steering, tyres, and the body — and how they connect.
41. **C41 — Physics & Rigid-Body Dynamics**: `Physics_Base`, the `Simulate → IntegrateMotion` chain, the `IRigidBody`/`RigidBody` abstraction, and time-stepping.
42. **C42 — Suspension, Tyres & Drivetrain**: the `Suspension*` class family, tyre grip, the drivetrain classes, and the tuning knobs that reach them.
43. **C43 — Collision Detection & Contact Records**: the 40-byte contact record, `DispatchCollisionEvents`, and the collision-event fan-out.
44. **C44 — Surfaces: Grip, Sound & Effects**: the surface taxonomy and how one surface tag fans out to grip + `RoadNoiseRecord` sound + `TireEffectRecord` effects.
45. **C45 — Damage & Deformation**: the `DAMAGE_*` zone model, the split between visual and functional damage, and how damage wrecks cops.

### Part IX — AI & gameplay systems

46. **C46 — AI Architecture: Goals & Actions**: the `AIGoal`/`AIAction` two-level planner and the mechanic component model that drives it.
47. **C47 — The AI Driver Brain & Vehicle Hierarchy**: the racer brain, skill scaling, and the AI vehicle class hierarchy.
48. **C48 — Pursuit & Heat: the State Machine**: the pursuit states and the two parallel Heat ladders (`heat_*` free-roam vs. `race_*` in-race).
49. **C49 — Cops: Dispatch, Formations, Roadblocks & the Bust Envelope**: cop management, formations, roadblocks, and the perception model (an envelope, not vision).
50. **C50 — Traffic & Population**: the spawn pipeline, the population band, traffic routing on the road rails, and the density/budget system.
51. **C51 — Racer AI & Rubber-Banding**: the two rubber-band systems — session `CatchUp` and the `aivehicle` multiplier table.
52. **C52 — Camera & the Cinematic Director**: the gameplay camera and the scripted director that frames intros, busts, and showcases.
53. **C53 — The Input System**: the device → mechanic → `GAME_ACTION_*` → driver pipeline, bindings, assists, and force-feedback.
54. **C54 — Speedbreaker & Sim-Rate Control**: the `GAME_ACTION_GAMEBREAKER` → `ESetSimRate` slow-motion hook.
55. **C55 — Points of Interest**: the marker + engagable trigger + minimap icon + career gate behind every map blip.
56. **C56 — Speedtraps, Pursuit Breakers & Hide-Spots**: the event POIs, the speed→bounty path, and the chase POIs with their cooldowns.

### Part X — Progression & the game in motion

57. **C57 — Game Session & Modes**: the session/mode objects and the begin/update/end lifecycle.
58. **C58 — Career & the Blacklist**: the Blacklist progression, players, and unlock gating.
59. **C59 — The Race Rulebook**: event = type + venue + checkpoint route + `gameplay`-class rules.
60. **C60 — In Motion: the Game in Playing Order**: boot → front-end → career → free-roam → a race end to end, every step grounded in a verified state/screen/string.

### Part XI — Rendering, effects & environment

61. **C61 — Rendering: the DX9 Frame Order**: world → road → vehicles → shadows → reflections → particles → speed-FX → bloom → motion-blur → HUD → Present, and the cheap-world/glossy-road/hero-car budget.
62. **C62 — Reflections, Shadows & the Car-Paint Material**: cubemap reflections, projected shadows, and the clearcoat car-paint shader.
63. **C63 — Speed Effects, Bloom, Motion Blur & Colour Grade**: the game's visual identity and the post chain.
64. **C64 — Effects & Particles: the Data-Driven System**: emitters, particle textures, linkage, and one-shots — the safest subsystem to retune.
65. **C65 — The FX-Bank Catalogue**: `fxenv`/`fxcar`/`fxtd`/`fxgame`/`fxnis`/`fxex` — cop lights, NOS, birds, leaves, flares, and why brake lights aren't emitters.
66. **C66 — Environment: Sky, Time-of-Day & Lighting**: the layered textured sky sets, baked world lighting, and authored time-of-day presets.
67. **C67 — Weather, Fog, Wind & Vegetation**: stock `RAINDROP` rain, `SFX_Weather` audio, `wetpaved` roads, fog/haze, and ambient wind — all driven by `WeathermanChunk`/`LightSections`.

### Part XII — Vehicles as customisable objects

68. **C68 — Vehicles: the Customisable Object**: the car as an object, the five shop categories, and what "buying" an item actually does.
69. **C69 — Performance Upgrades & Tuning Bars**: the engine/transmission/induction/tyre upgrade classes and the `TOPSPEED`/`ACCEL`/`HANDLING` bars.
70. **C70 — Visual Customisation**: body kits/aero, wheels/brakes, paint & colour, and the per-car `VINYLS.BIN` decal masks.
71. **C71 — Cars, End to End**: a complete car-modding walkthrough that ties every system in the book together.

### Part XIII — Scripting & networking

72. **C72 — Lua Scripting: the Embedded 5.0.1 Layer**: the bytecode chunks, the `LuaPostOffice`/`LuaRuntime`/`LuaAttributes` bridge, and what is moddable.
73. **C73 — The Message Vocabulary & Stategraph**: the `M*`/`E*` message verbs and the event-logic "stategraph."
74. **C74 — Multiplayer & Online Services**: the decoded-but-mostly-dead EA online surface — the product/auth tags, the lobby lifecycle, the wire protocol, and an honest accounting of what the map does and does not prove.

### Part XIV — Reverse engineering & workflow

75. **C75 — Modding Workflow & Safety**: backups, in-place vs. repack, ancestor-size fixups, atomic writes, and distribution.
76. **C76 — Advanced Reverse Engineering**: identifying unknown data, recovering the attribute schema, building your own readers, and validating them.
77. **C77 — Hash Recovery & Name Dictionaries**: brute-forcing Joaat/Bin hashes, building name dictionaries, and closing the "0x0743CFB1" gaps.
78. **C78 — Runtime Observation & Live State**: how to read values that only exist while the game runs — interned name tables, named singletons, live object sizes and upgrade deltas — and how to fold them back into a static findings log.

---

## Appendices

- **Code Library** — every worked reader/writer in the book, collected and portable.
- **Struct Reference** — every documented struct in one place, with offsets and confidence.
- **Command Vocabulary** — the full `GAME_ACTION_*` / `M*` / `E*` / `ENIS*` / `CDAction*` verb lists.
- **Coverage Audit** — what is ✅ verified, 🟡 reasoned, and ⏳ open, subsystem by subsystem.

---

## Scope, sources, and honesty

This encyclopedia describes the **retail PC release, patched to v1.3**. Console builds differ in
byte order, in some codecs, and in a handful of animations that are static on PC; those differences
are called out where they matter.

Two disciplines run through every page. First, **the world is Z-up and stays Z-up** — no parser in
this book converts axes; conversion happens only at an explicit import/export boundary. Second,
**every claim carries its confidence.** Where the evidence is a byte you can see, it says so; where
it is an inference about intent, it says that instead; and where knowledge runs out, the ⏳ marks the
edge rather than papering over it. The goal is a reference you can trust precisely because it tells
you how much to trust each line.
