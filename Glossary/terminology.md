# Glossary — terminology & concepts

The vocabulary you need to read the rest of the encyclopedia, grouped by the layer it belongs to —
from the raw container model up to the running game. Skim it once; refer back as needed. Each term
points to the chapter that unpacks it.

---

## Engine & container model

**EAGL** — "EA Graphics Library," the EA Black Box engine technology underpinning Most Wanted and
its sibling titles (the *Underground* and *Most Wanted*/*Carbon* generation). Its defining trait for
a modder is the **chunk** container model: nearly every asset on disk is a tree of chunks.

**Chunk** — the universal building block. An 8-byte header `{ uint32 id; uint32 size; }` followed by
exactly `size` bytes of payload. Files are just chunks laid end to end. See
[C1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md).

**Container chunk** — a chunk whose payload is *itself* a sequence of chunks. Identified by **bit 31
of the id being set** (`id & 0x80000000`). You recurse into it.

**Leaf chunk** — a chunk whose bit 31 is clear. Its payload is raw, format-specific data. You stop
and parse it.

**Container bit** — the single flag (`0x80000000`) that distinguishes the two. It is the most useful
heuristic in the whole engine: it tells any walker whether to descend or to treat the payload as data,
with no schema and no per-format knowledge. See [C1.1](../C1-EAGL-Container-Model/01-the-container-bit.md).

**Payload** — the `size` bytes after a chunk header. For a container, a nested chunk stream; for a
leaf, the actual data.

**Size tree** — the invariant that a container's `size` equals the total on-disk footprint of its
children, each child contributing `8 + child.size`. Editing a payload's length invalidates every
ancestor's size, which must be fixed from the leaf up. See
[C1.2](../C1-EAGL-Container-Model/02-chunk-header-and-sizes.md).

**Off-by-eight** — the classic bug of stepping `size` instead of `8 + size` between siblings, landing
8 bytes early and desyncing the walk. The header is excluded from `size`, always.

**Alignment padding (`0x11`)** — runs of the literal byte `0x11` inserted before many payloads to
push meaningful data onto a memory boundary (commonly 16 bytes). Strip them before parsing; they
carry no information but they *do* count toward `size`. See
[C1.4](../C1-EAGL-Container-Model/04-alignment-and-padding.md).

**Null chunk** — a chunk with `id == 0x00000000`: pure filler between chunks, aligning the *next*
chunk's header. Step over it like any other chunk.

**BUN / BIN / LZC** — common extensions. `.BUN` and `.BIN` are usually raw chunk trees; `.LZC` is a
JDLZ-compressed chunk stream. The extension is a *hint*, not a contract — identify by content
([extensions.md](extensions.md)).

**FourCC** — a 4-byte ASCII tag used as a magic/identifier at file offset 0 for the *non-chunk*
container models: `'VPAK'`, `'JDLZ'`, `'DDS '`, `'ABKC'`, `'SCHl'`, `'MPFF'`, `'LOCH'`, `'MZ'`.

---

## Numbers & identifiers

**Little-endian (LE)** — least-significant byte first; the native order for almost everything.
`0x12345678` is stored on disk as `78 56 34 12`.

**Big-endian (BE)** — most-significant byte first. Used *only* inside the EA audio stream fields (the
`PT` TLV records of a bank, the `SCHl` header fields of a music stream) — a fossil of the format's
console heritage. Reading a 44.1 kHz rate as `0x44AC0000` instead of `0x0000AC44` is the tell you
read the wrong side of the border. See
[C1.5](../C1-EAGL-Container-Model/05-endianness-islands.md).

**Joaat (Jenkins one-at-a-time)** — the 32-bit string hash used pervasively for names (textures,
objects, classes, effects). Most identifiers are stored as a Joaat hash, *not* as text, which is why
a raw dump shows `0x0743CFB1`-style placeholders until you resolve them. See
[C2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md).

**Reflection hash (lookup2 / `0xABCDEF00`)** — the hash that keys the *entire* attribute/vault
reflection system: class names, collection (car) names, field names, enum values. It is Jenkins
**lookup2** (1996) seeded with `0xABCDEF00` — a *different* function from Joaat. ✅ Verified from
`speed.exe` (`0x005CC240`/`0x005CC090`) and proven by `lookup2("default") == 0xEEC2271A`. See
[C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md).

**Bin hash** — a weak `hash = c + 33·hash` sum used by a few *secondary* keyings (e.g. some scenery
group keys). It is **not** the attribute-system hash (that is the reflection hash above). Collision-prone;
use with care. See [C2.3](../C2-Identifiers-And-Hashing/03-bin-sum-hash.md).

**Hash resolution / name recovery** — recovering the original string for a hash by hashing a
dictionary of candidate names and matching. Without it, names display as numeric placeholders. See
[C77](../C77-Hash-Recovery/C77-Hash-Recovery.md).

---

## Compression

**JDLZ** — EA Black Box's LZ77 variant with two interleaved flag streams and a 16-byte header
beginning with the ASCII magic `'JDLZ'`. Must be decompressed before any chunk header is visible. See
[C3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md).

**`.lzc`** — a file that is JDLZ-compressed (often a global or font bundle). Decompress, then
re-identify the result.

---

## Textures & materials

**TPK (Texture PacK)** — a container (`0xB3300000`) bundling textures with a metadata table. Two
variants: **standard** (124-byte entries, one raw pixel blob) and **compressed** (24-byte
descriptors, per-texture JDLZ blobs). See [C5](../C5-Textures-TPK/C5-Textures-TPK.md).

**DXT1 / DXT3 / DXT5 (BC1 / BC2 / BC3)** — S3/DirectX block texture compression. 4×4-pixel blocks;
DXT1 is 8 bytes/block (0.5 B/px), DXT3 and DXT5 are 16 bytes/block (1 B/px). See
[C6](../C6-Texture-Codecs/C6-Texture-Codecs.md).

**ARGB32** — uncompressed 32-bit colour, 4 B/px. **PAL8** — 8-bit palettised pixels plus a colour
palette.

**DDS** — Microsoft DirectDraw Surface, the standard interchange container for DXT/ARGB textures: a
128-byte header (`'DDS '` magic) plus pixel data.

**Mip / mipmap** — successively half-sized copies of a texture for distance rendering. A "mip chain"
is the base image plus all smaller levels.

**FVF (Flexible Vertex Format)** — a flag set describing what each vertex contains (position, normal,
colour, UVs, tangents) and therefore the **stride** in bytes (commonly 24, 36, or 60). See
[C9](../C9-Meshes-FVF/C9-Meshes-FVF.md).

---

## Geometry

**Solid** — one renderable object (a body panel, a building, a prop). Grouped into a **solid list**.

**Mesh / shading group** — a renderable sub-mesh within a solid, each with one material and its own
slice of vertex and index data (104 bytes per group descriptor).

**Index buffer** — a list of `uint16` indices forming triangles, numbered **relative** to a group's
slice of the vertex buffer ("group-relative").

**AABB** — axis-aligned bounding box (`min`/`max` corner), used for culling and placement.

---

## World

**Section** — a streamed region of the world, loaded and unloaded by camera proximity. The streaming
table (92-byte entries) lists them. See [C15](../C15-World-Streaming/C15-World-Streaming.md).

**Scenery** — the prop layer: many transformed *instances* (64 bytes each) of solid models placed in
the world, culled through a spatial AABB tree. See [C16](../C16-Scenery-Cull-Tree/C16-Scenery-Cull-Tree.md).

**Cull tree** — the spatial acceleration structure over scenery: 36-byte AABB nodes with a fanout of
up to five children. ✅ decoded.

**Scenery group** — a named, BinHash-keyed set of scenery instances toggled together (event
barriers, doors, breakable props). Its state is re-applied at section load.

**Trigger region** — a typed 2-D world-plane volume that fires gameplay events (speedtrap, pursuit
boundary, tollbooth), tested at runtime by an even-odd point-in-polygon rule. See
[C17](../C17-Trigger-Regions/C17-Trigger-Regions.md).

**Road network (`CARP`)** — the AI/GPS graph: 32-byte nodes and 22-byte segments plus a cost grid.
Cops and GPS route on it; traffic rides its rails. ✅ graph core decoded. See
[C18](../C18-Road-Network/C18-Road-Network.md).

---

## Attributes

**VPAK / vault** — the EA "Attribulator" attribute container (`attributes.bin` and friends) where
gameplay, car, and pursuit tuning live. `'VPAK'` magic. See
[C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md).

**ErtS / NpeD / NrtS / NtaD** — the tagged blocks inside a vault: string table, dependency table,
symbol table, and typed data records, respectively.

**Reflection schema** — the field-name → offset → type map the engine uses to interpret vault
records. It is compiled into the executable, which is why raw records are hard to decode field by
field without recovering it. See [C12](../C12-Reflection-Schema/C12-Reflection-Schema.md).

**Resolved-value model** — the readable-and-writable view of a vault record as inline
`{field, f32, type}` triples with `default` inheritance, the practical basis for editing tuning
values.

---

## Audio & video

**ADPCM** — Adaptive Differential PCM, a family of 4-bit-per-sample compressors. Variants here:
IMA-ADPCM and EA-ADPCM.

**EA-XA / EA-XAS** — EA's ADPCM codecs. EA-XA v2 ("eaxa") is used by sound banks and music; EA-XAS
v0 ("xas0") by engine audio and VP6 movie audio. Both are mono per stream. See
[C20](../C20-Audio-Codecs/C20-Audio-Codecs.md).

**EA-MP3** — MP3 wrapped in an EA stream header.

**SCHl / SCCl / SCDl / SCEl** — the EA-stream block tags inside a `.MUS` file: header, continuation,
data, and end.

**PT chunk** — the big-endian tag-length-value records describing one sound inside an ABK/BNKl bank.

**VP6** — On2 VP6 video codec, carried in an EA Multimedia container (`.vp6`). See
[C23](../C23-Video-VP6/C23-Video-VP6.md).

---

## The runtime: classes, objects & the substrate

These terms belong to the *running* game rather than to files on disk. They become central from
[C32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md) onward.

**Class / object** — the engine is C++ at runtime; loaded data is turned into objects of concrete
classes. A class has a known **size** (bytes per instance) and a **vtable**.

**VTable** — the table of virtual-function pointers at the head of a polymorphic object. Its address
and the roles of its slots (method, getter, thunk, stub, destructor) identify the class. See
[C34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md).

**Singleton / manager** — a single global instance coordinating a subsystem (for example the cop
fleet manager, the stream manager, the road-network manager). Located at a fixed address in the
executable's data segment.

**Connector** — a one-way boundary object that lets one subsystem read another's state without a
back-reference, used heavily in the vehicle model to keep the simulation acyclic. See
[C39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md).

**Mechanic** — a component of the vehicle model (engine, transmission, induction, drivetrain,
suspension, steering, tyres, body). The "eight mechanics" compose a car's behaviour. See
[C40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md).

**StreamMgr** — the refcounted resource/residency manager that owns loaded bundles and hands out
`MemoryFile` views. See [C36](../C36-Archives-VFS/C36-Archives-VFS.md).

**MemoryFile** — an in-memory view of a file whose reads the engine can intercept, the seam between
the VFS and the loaders.

**SlotPool** — a fixed-size-block allocator; the engine uses a family of them (including three
dedicated particle pools and pre-sized pathfinding pools) alongside a general allocator. See
[C35](../C35-Memory-Management/C35-Memory-Management.md).

**GameFlow** — the boot/phase state machine that decides which manifests and bundles are resident at
each stage (front-end, career, in-race). See [C38](../C38-Streaming-Residency/C38-Streaming-Residency.md).

**Frame spine** — the top-level per-frame update chain (`Engine::PumpFrame`/`FrameTick`) into which
every subsystem's update hooks in a fixed order. See [C37](../C37-Frame-Spine/C37-Frame-Spine.md).

**Debug-fill sentinel** — a recognisable fill value (e.g. `0x0F0F0F0F`, `0xFFFF`, `0xAAAA`) left in
uninitialised or freed memory, useful for fingerprinting globals by their on-disk/initial value.

---

## Modding-process terms

**In-place patch** — overwriting data within its existing slot without changing file size. The
safest, fastest technique: change values, never counts. See [C75](../C75-Modding-Workflow/C75-Modding-Workflow.md).

**Repack** — rebuilding a block when new data doesn't fit, which shifts offsets and forces fixing the
size fields of all enclosing chunks.

**Ancestor-size fixup** — after growing or shrinking a nested chunk, adding the size delta to every
container that encloses it, all the way to the file root. Forgetting this is the most common cause of
a file the game refuses to load.

**Backup / revert** — keeping the original bytes so any change can be undone. The first rule of the
workflow chapter.
