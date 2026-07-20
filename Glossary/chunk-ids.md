# Glossary — master chunk-ID table

Every chunk identifier known in the retail PC data set, grouped by subsystem. Container chunks (bit
31 set) are marked **C**; leaf chunks hold data. IDs were verified by scanning the retail world
bundles, the global compressed data, and the attribute vaults, and cross-checked against the chunk
handlers registered in the executable.

> **Reading tip.** When you dump an unknown file and see one of these IDs, you immediately know what
> subsystem you're in. When you see an ID *not* in this table, note whether bit 31 is set (container
> vs. leaf) and record its size — that alone tells you whether to recurse. The book's universal
> opener ([C1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) automates exactly this lookup.

The right-hand family digits are worth learning: `0x_0134___` is geometry, `0x_331____`/`0xB33_____`
is textures, `0x_0034___`/`0x_003____` is the world/streaming layer, and `0x_0E34___` is cutscene
animation. The high nibble is just the container bit riding on top of the family.

---

## Geometry / solids ([C8](../C8-Geometry-Solids/C8-Geometry-Solids.md), [C9](../C9-Meshes-FVF/C9-Meshes-FVF.md))

| ID | C | Name | Payload |
|---|:--:|---|---|
| `0x80134000` | C | GeometryContainer | One solid list |
| `0x80134001` | C | GeometryInfo | List-level info wrapper |
| `0x00134002` | | GeometryInfoHeader | `u32[4]`, then class/file name (C string @ +16) |
| `0x00134003` | | GeometryHashTable | `{u32 hash, u32 pad}` per object |
| `0x80134008` | C | GeometryEmpty | Empty placeholder |
| `0x80134010` | C | GeometryObject | One solid object |
| `0x00134011` | | GeometryObjectHeader | 160-byte header (nameHash, numTris, bbox, transform) + name |
| `0x00134012` | | GeometryTextureRefs | `{u32 hash, u32 pad}` per texture |
| `0x00134013` | | GeometryLightMaterial | Light / material info |
| `0x80134100` | C | GeometryMesh | Mesh container |
| `0x00134900` | | MeshHeader | Counts + FVF flags |
| `0x00134B01` | | MeshVertices | Vertex buffer(s) |
| `0x00134B02` | | MeshShadingGroups | 104 bytes per group |
| `0x00134B03` | | MeshIndices | `u16` triangle list |
| `0x00134C02` | | MeshMaterialName | Material / texture-usage C strings |

## Textures / TPK ([C5](../C5-Textures-TPK/C5-Textures-TPK.md), [C6](../C6-Texture-Codecs/C6-Texture-Codecs.md))

| ID | C | Name | Payload |
|---|:--:|---|---|
| `0xB3300000` | C | TPKContainer | Texture pack |
| `0xB3310000` | C | TPKInfo | Metadata wrapper |
| `0x33310001` | | TPKInfoHeader | version, 28-byte pack name @+4, 64-byte path @+32, hash @+96 |
| `0x33310002` | | TPKHashTable | `{u32 hash, u32 pad}` per texture |
| `0x33310003` | | TPKCompEntries | 24-byte descriptors (compressed variant) |
| `0x33310004` | | TPKEntries | 124-byte entries (standard variant) |
| `0x33310005` | | TPKCompInfo | 32-byte records, FourCC @ +20 |
| `0xB3320000` | C | TPKData | Pixel-data wrapper |
| `0x33320001` | | TPKDataHeader | Data header |
| `0x33320002` | | TPKDataRaw | Raw DXT/ARGB blob, or per-texture JDLZ blobs |
| `0xB0300100` | C | TPKAnimBlock | Texture-animation block ([C7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) |

## World / streaming / scenery ([C15](../C15-World-Streaming/C15-World-Streaming.md)–[C18](../C18-Road-Network/C18-Road-Network.md))

| ID | C | Name | Payload |
|---|:--:|---|---|
| `0x00034112` | | StreamingFileHeader | Master-file header |
| `0x00034110` | | StreamingSections | 92-byte section entries (the streaming table) |
| `0x00034191` | | StreamingBarrierInfo | Barrier / cull info |
| `0x00034146` | | TrackPositionMarkers | Pad + 48-byte point markers |
| `0x80034147` | C | TriggerRegionParent | Wraps one `0x0003414A` |
| `0x0003414A` | | TriggerRegions | Typed 2-D trigger regions |
| `0x80034150` | C | TrackPathManager | Path manager |
| `0x80034100` | C | ScenerySection | Prop-placement section |
| `0x00034101` | | SceneryHeader | Section number @ +12 |
| `0x00034102` | | SceneryInfos | 72-byte model definitions |
| `0x00034103` | | SceneryInstances | 64-byte placements |
| `0x00034105` | | SceneryTreeNodes | Spatial cull tree — 36-byte AABB nodes, fanout ≤5 (✅ decoded) |
| `0x00034106` | | SceneryGroupLinks | `{masterRecordIndex, instanceIndex}` u16 pairs; re-applies group state at section load |
| `0x00034107` | | SceneryOverrideHooks | Override hooks (128 B/record; preserve raw) |
| `0x00034108` | | SceneryGroupRecords | Master table: 6-byte `{i16 sectionNumber, i16 instanceIndex, u16 flags}` |
| `0x00034109` | | SceneryGroups | Named groups: BinHash-keyed lists of master-record indices |
| `0x8003B900` | C | VisibleSectionManager | Visibility manager |
| `0x0003B901` | | VisibleSectionBounds | Visible-section bounds |
| `0x0003B800` | | WorldMapData | `CARP` AI/GPS road network — 32-B nodes / 22-B segments (✅); cost grid raw (⏳) |
| `0x8003B810` | C | EventSequencePack | Holds `0x0003B811` |
| `0x0003B811` | | EventSequenceChunk | One event-script blob — `CARP` road scripts in track bundles; in NIS bundles, the scene's `ENIS*` timeline (✅ decoded) |
| `0x80036000` | C | LightSections | Lighting |
| `0x00037080` | | WeathermanChunk | Weather / time of day |
| `0x0003A100` | | MiniMapTile | JDLZ-wrapped minimap TPK tile ([C29](../C29-Minimap/C29-Minimap.md)) |

## Cutscene animation ([C24](../C24-NIS-Object/C24-NIS-Object.md)–[C26](../C26-Ambient-Animation/C26-Ambient-Animation.md))

| ID | C | Name | Payload |
|---|:--:|---|---|
| `0x00E34009` | | NisAnimation | 8-byte `0x11` sentinel + MIPS ELF32 animation object |
| `0x80E34000` | C | EASoundContainer | Embedded audio container |
| `0x073C1203` | | AudioEventRecord | Record ID leading `NISAudio.evt` / `copspeech.evt` (layout ⏳) |

## Common / misc

| ID | C | Name | Payload |
|---|:--:|---|---|
| `0x00000000` | | Null / Pad | Alignment filler between chunks |

---

## Non-chunk magics (a *different* container model)

These appear at file offset 0 and identify a container model that is **not** the chunk tree. Branch
on them *before* trying to read a chunk header.

| Magic | Meaning | Chapter |
|---|---|---|
| `'JDLZ'` | JDLZ-compressed stream (decompress, then re-identify) | [C3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md) |
| `'VPAK'` | Attribute vault | [C11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md) |
| `'DDS '` | Standalone DDS texture | [C6](../C6-Texture-Codecs/C6-Texture-Codecs.md) |
| `'ABKC'` / `'BNKl'` | Sound bank | [C19](../C19-Audio-Banks/C19-Audio-Banks.md) |
| `'SCHl'` | MUS / EA audio stream | [C21](../C21-Music-MUS/C21-Music-MUS.md) |
| `'MPFF'` | Music routing graph | [C21](../C21-Music-MUS/C21-Music-MUS.md) |
| `'LOCH'` | Memory-card / localized container | [C31](../C31-Save-Data/C31-Save-Data.md) |
| `'MZ'` | Windows executable / DLL | — |

---

## On the runtime side: the chunk-handler registry

The generic EAGL loader does not hardcode what each ID *means*. Instead the engine builds a **chunk
handler registry** at startup: a table mapping recognised chunk IDs to the subsystem callback that
consumes their payload. When the loader walks a bundle, it looks each leaf ID up in this table and
dispatches the payload to the matching handler; unrecognised IDs are skipped by `8 + size` with no
harm. This is the runtime counterpart of the container-bit rule — the *bit* decides "recurse or not,"
the *registry* decides "who gets this leaf." ✅ The registry's existence and dispatch behaviour are
confirmed in the executable; the full ID→handler mapping is a subset of the table above and is
detailed in [C36](../C36-Archives-VFS/C36-Archives-VFS.md).

> The container-bit rule (`id & 0x80000000`) remains the single most useful heuristic when you meet a
> new ID: it tells a parser whether to descend or to treat the payload as data before you know
> anything else about the file. See [C1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md).
