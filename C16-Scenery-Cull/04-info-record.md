# C16.4 — The 72-byte SceneryInfo

> **The one-sentence version:** each model definition is 72 bytes — the link to the solid to draw, the draw
> distance / LOD rules, and flags — and it applies to *every* instance of that model at once, so it's the
> record you edit to change how a prop type looks everywhere.

[← C16.3 — The 64-byte SceneryInstance](03-instance-record.md) · [Chapter 16 hub](C16-Scenery-Cull.md) ·
[Next: C16.5 — The cull tree →](05-cull-tree.md)

---

## The record

A `SceneryInfo` (72 B) is the *model* side of the scenery split ([C16.2](02-models-instances.md)). Each one
defines one prop type:

- **The solid link** — which mesh to draw ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md)),
  referenced by the same asset-hash mechanism as everything geometric
  ([C8.3](../C8-Geometry-Solids/03-object-hash.md)).
- **Draw distance / LOD** — how far away the prop is still drawn, and how it fades or swaps to lower detail
  with distance.
- **Flags** — per-model behaviour bits (collision, shadow, etc.).

The 64-byte instances ([C16.3](03-instance-record.md)) reference these by index; one info serves all the
placements that point at it.

## One info, many instances

The defining property of the info is its *reach*: editing it changes every instance of that model. This is
usually what you want — and occasionally a trap:

- **Want:** a section's lampposts should be visible farther away → raise the info's draw distance once, and
  all of them fade at the new range.
- **Trap:** you meant to change *one* lamppost → editing the info changes *all* of them. To change a single
  prop you must give it its own info (a new model definition it alone references), not edit the shared one.

So the info is the "all of this type" lever; the instance is the "this one" lever
([C16.2](02-models-instances.md)). Choosing the right one is the scenery analogue of the vault's
override-vs-`default` scope decision ([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)).

## Draw distance is the headline field

Of the info's fields, **draw distance** is the one modders touch most, because it directly trades visual
richness for performance:

- **Longer draw distance** → props visible farther, richer skyline, more to render (slower).
- **Shorter draw distance** → props pop in closer, cheaper to render (faster), but more visible pop-in.

Because the draw distance is per-model, you can tune the *expensive* props (dense foliage, detailed clutter)
to fade sooner while keeping *cheap, iconic* props (signs, landmarks) visible far — a targeted
performance/appearance balance rather than a global slider.

> ✅ *Verified:* the 72-byte info stride and its role as the per-model definition referenced by instances,
> decoded against retail data.
> 🟡 *Reasoned:* the exact byte offsets of the solid link, draw distance, and each flag within the 72 bytes are
> identified by role; the stride and the model/instance join are verified.

## Keep the info table honest

The info table is a shared resource, and edits to it ripple through every instance:

- **Adding an info** is how you introduce a new prop type ([C16.2](02-models-instances.md)); append it so
  existing indices don't shift.
- **Never insert mid-table.** Inserting an info shifts every later index, silently re-modelling every instance
  whose index moved. Append, don't insert.
- **Don't delete an info that instances still reference.** Remove the referencing instances first, or the
  index dangles.
- **Match draw distance to the model's cost and importance.** A per-model decision, not a global one.

## Reading infos

```python
def read_infos(chunk):                 # SceneryInfos, 72 B stride
    return [chunk[i*72:(i+1)*72] for i in range(len(chunk) // 72)]
# decode the solid link + draw distance from each; preserve unknown bytes raw
```

As with instances, the robust approach is to decode the fields you act on (solid link, draw distance) and
preserve the rest byte-exact — you never need to fully model the flags to safely retarget a prop or extend its
draw distance.

---

### Key takeaways

- The 72-byte `SceneryInfo` defines a model: solid link, draw distance / LOD, and flags.
- It reaches *every* instance of that model — the "all of this type" lever; use a per-instance approach (a new
  info) to change one prop.
- Draw distance is the key tunable, trading richness for performance per-model.
- Append infos (never insert mid-table, which shifts indices); don't delete infos instances still reference.
- Decode the solid link and draw distance; preserve unknown bytes raw.

**Continue:** [C16.5 — The cull tree](05-cull-tree.md) · [Chapter 16 hub](C16-Scenery-Cull.md)
