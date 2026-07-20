# C16.1 — Scenery in a Section

> **The one-sentence version:** a streamed section's props live in a `ScenerySection` container that
> separates model definitions (`SceneryInfos`, 72 B), placements (`SceneryInstances`, 64 B), and the spatial
> index (`SceneryTreeNodes`, 36 B) — three parallel tables joined by index.

[← Chapter 16 hub](C16-Scenery-Cull.md) · [Next: C16.2 — Models vs instances →](02-models-instances.md)

---

## The container

When a section ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) is decompressed, its scenery is
an EAGL sub-tree ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)) — a `ScenerySection`
container holding a fixed set of chunks:

```
ScenerySection
├── SceneryInfos          72-byte model definitions
├── SceneryInstances      64-byte placements (transform + info index)
├── SceneryTreeNodes      36-byte cull-tree nodes
└── SceneryOverrideHooks  128-byte override records (preserve-raw)
```

Each chunk is a flat array of fixed-size records, so the record counts are `chunkSize / stride` — the
divide-to-N check you know from every table in this book (TPK entries, SolidList objects, streaming sections).
The infos and instances are joined by index; the tree references instances by index too. Nothing here embeds
geometry — the props' meshes are solids elsewhere ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)),
and the scenery only *references* them.

## Two levels of scenery organisation

Scenery is indexed at two levels, and it helps to keep them straight:

- **Per-section content** (this chapter): the `SceneryInfos`/`SceneryInstances`/`SceneryTreeNodes` inside each
  streamed section — the actual props and their placements.
- **Master-file organisation**: the track file (`L2RA.BUN`) carries a section-level scenery container
  (`0x80034150`) whose header — verified to hold a count of **373** and a `u16` remap/index table — plus
  group lists (`0x00034153`, blocks of model indices) and preculler data organise scenery *across* sections
  for the streamer.

The master file says which sections have what; the section payloads hold the placements. This chapter decodes
the payloads.

> ✅ *Verified:* the master file's scenery container `0x80034150` (header count 373, model-index group lists,
> preculler chunk) parsed from `L2RA.BUN`; the per-section record strides (72/64/36) decoded against the full
> retail set.

## Reading the section's scenery

```python
def read_scenery(section_chunks):
    infos     = parse_records(section_chunks["SceneryInfos"],     72)   # C16.4
    instances = parse_records(section_chunks["SceneryInstances"], 64)   # C16.3
    tree      = parse_records(section_chunks["SceneryTreeNodes"], 36)   # C16.5
    # joins: instance.info_index → infos[]; tree entries → instances[]
    return infos, instances, tree
```

The three tables are useless in isolation and powerful together: the tree tells you *which* instances to
consider, each instance names *which* info (model), and the info names *which* solid to draw. Decode all three
and you can reconstruct — and edit — a section's world.

## Counts and cross-checks

As always, cross-check the counts before trusting a parse:

- `instances = SceneryInstances.size / 64`; every instance's info index must be `< infoCount`.
- `infos = SceneryInfos.size / 72`.
- `nodes = SceneryTreeNodes.size / 36`; every tree leaf entry must reference a valid instance, and the leaves
  must **partition** the instances ([C16.5](05-cull-tree.md)).

These three checks — index-in-range, valid stride division, leaf partition — are what tell you the section is
internally consistent, and they are exactly the invariants an edit must preserve
([C16.6](06-editing-scenery.md)).

---

### Key takeaways

- A section's props are a `ScenerySection` container: `SceneryInfos` (72 B), `SceneryInstances` (64 B),
  `SceneryTreeNodes` (36 B), plus preserve-raw override hooks.
- Record counts are `chunkSize / stride`; the tables join by index and reference solids, not embedded
  geometry.
- Scenery is organised at two levels: master-file section organisation (`0x80034150`, count 373) and
  per-section placements (this chapter).
- Reconstruct a prop via `tree → instance → info → solid`.
- Cross-check: info index in range, clean stride division, and cull-tree leaves partitioning the instances.

**Continue:** [C16.2 — Models vs instances](02-models-instances.md) · [Chapter 16 hub](C16-Scenery-Cull.md)
