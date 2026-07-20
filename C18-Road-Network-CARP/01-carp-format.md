# C18.1 — CARP Is a TLV Blob, Not a Chunk Tree

> **The one-sentence version:** the road network's `0x0003B800` payload is a `CARP` blob with the magic stored
> reversed (`PRAC`) and a directory of four-character tags (`RNnd`, `RNsg`, …) pointing at counts and offsets —
> so you must branch it *out* of the universal chunk walker and parse it as a tag directory.

[← Chapter 18 hub](C18-Road-Network-CARP.md) · [Next: C18.2 — The RNnd node →](02-node-record.md)

---

## The critical structural fact

Almost everything in this book is an EAGL `{id, size}` chunk tree
([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)). `CARP` is **not**. Inside the
`0x0003B800` chunk (`WorldMapData`), the payload is a self-describing **TLV** (tag / length / value) blob with
its own directory. If you point the universal chunk reader at it, the reader will interpret `RNnd`, `RNsg`, and
their sizes as chunk headers, walk into the middle of node data, "parse" nonsense, and — worst of all — if you
write the result back, corrupt the graph. The road network is the one place the book's default tools do the
wrong thing, so a correct reader **detects `CARP` and switches parsers**.

```python
def read_worldmapdata(chunk_0003B800):
    payload = chunk_0003B800.body
    assert payload[0:4] == b"CARP" or payload[8:12] == b"CARP"  # magic (may follow a marker)
    return parse_carp(payload)      # a tag directory, NOT walk_eagl()
```

## The magic is reversed

The magic reads **`PRAC`** in the raw bytes — `CARP` spelled backwards — because CARP's four-character tags are
stored in the opposite byte order from EAGL chunk ids. Every tag follows the same convention, so you read them
reversed:

| Raw bytes | Tag | Meaning |
|---|---|---|
| `PRAC` | `CARP` | magic |
| `dnNR` | `RNnd` | **road nodes** (32-byte) |
| `gsNR` | `RNsg` | **road segments** (22-byte) |
| `drNR` | `RNrd` | roads (named streets) |
| `fpNR` | `RNpf` | pathfind data |
| `pgNR` | `RNgp` | node groups |
| `dhNR` | `RNhd` | header |
| `drGC` | `CGrd` | cost grid |
| `ncGC` | `CGcn` | cost-grid connections |

Verified: all of these tags are present in the retail `L2RA.BUN` road blob, in reversed form.

## The tag directory

CARP opens with a directory: each entry names a tag, a **count**, and a **data offset** into the blob. Reading
the directory tells you how many nodes, segments, and roads there are and where their arrays live:

```python
def parse_carp_directory(payload):
    tags = {}
    p = find_directory_start(payload)          # after the CARP/PRAC magic + header
    while more_entries(payload, p):
        tag   = payload[p:p+4][::-1].decode()   # reverse the bytes → "RNnd", "RNsg", …
        count = u32(payload, p + 8)
        off   = u32(payload, p + 12)
        tags[tag] = (count, off)
        p += ENTRY_SIZE
    return tags
```

From the worked track this yields `RNnd → (4385, 0x338A0)`, `RNsg → (6538, 0x636F0)`, `RNrd → (1308, …)`.
Those counts are your ground truth for the arrays that follow ([C18.2](02-node-record.md), [C18.3](03-segment-record.md)).

> ✅ *Verified:* `0x0003B800` carries the `CARP` magic (`PRAC` reversed) and a tag directory with all the
> `RN*`/`CG*` tags; the directory gives 4,385 nodes, 6,538 segments, 1,308 roads.

## Why CARP is different

`CARP` ("Car Attribute Package") is EA's attribute-blob format — the same family idea as the VPAK vault
([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)): a self-describing container of typed data,
not a recursive chunk tree. The road network is stored this way because it is *tabular graph data* (arrays of
fixed-size records indexed by the directory), which a flat tag-directory format serves better than a size-tree.
Recognising "this is CARP, parse it as a directory" is the whole trick.

## CARP beyond the road network

The road network is not the only user of the `CARP` directory container — the *same* structure (the `'PRAC'`
reversed magic, the tag directory of typed entries) carries several unrelated systems, distinguished by their chunk
ID and their entry tags:

- **`0x0003B800` — the road network** (this chapter): tags `RNnd` (nodes) / `RNsg` (segments) — the graph.
- **`0x0003B801` — `WCollisionPack`** ([C63.8](../C63-Collision-World/08-wcollisionpack.md)): tags `ca` (collision
  geometry articles) / `ci` (instance lists) — the walls that stop the car. The neighbouring chunk ID is no
  coincidence: the world's collision walls are the road's corridor edges, stored in the same container as the road
  graph they parallel.
- **`0x0003B811` — event sequences** ([C17.6](../C17-Triggers-Barriers/06-events-editing.md)): a `CARP` directory
  whose `Ni` entry holds the sequence's name hash — the scripted-event blobs.

So `CARP` is a *reusable directory format*, not a road-network-specific one — recognising it is a skill that pays off
across the road graph, the collision packs, and the event scripts alike. Each reverses its tags the same way
([the magic is reversed](#the-magic-is-reversed)), counts its entries from the directory, and treats the blob as one
unit. Learn to read one CARP and you can read them all; the only difference is which typed records the directory
points at.

> ✅ *Verified:* `0x0003B801` (`WCollisionPack`) and `0x0003B811` (event sequence) both carry the `CARP`/`'PRAC'`
> directory, confirmed by their `speed.exe` loader dispatch and by byte-for-byte parse of their directories in the
> retail data — the same container family as the `0x0003B800` road network.

## Editing implications

- **Never walk CARP with the chunk reader.** Branch it out; parse the directory
  ([C18.6](06-frontier-editing.md)). Walking it with the EAGL reader and writing back corrupts the graph.
- **Trust the directory counts.** The `RNnd`/`RNsg` counts tell you the array lengths; use them, don't guess by
  dividing sizes.
- **Read tags reversed.** A tag that looks like `dnNR` is `RNnd`.
- **The blob is one unit.** Its offsets are internal; editing anything shifts them, so treat CARP as a whole to
  rewrite, not a tree to patch.

---

### Key takeaways

- `0x0003B800` is a **`CARP` TLV blob**, not an EAGL chunk tree — the universal reader must branch around it.
- The magic and all tags are stored **reversed** (`PRAC` = `CARP`, `dnNR` = `RNnd`).
- A tag directory gives each array's **count** and **offset**: 4,385 nodes, 6,538 segments, 1,308 roads
  (verified).
- CARP is EA's attribute-blob format — tabular graph data in a self-describing directory, not a size-tree.
- Parse the directory, trust its counts, read tags reversed, and never walk CARP with the chunk reader.

**Continue:** [C18.2 — The RNnd node](02-node-record.md) · [Chapter 18 hub](C18-Road-Network-CARP.md)
