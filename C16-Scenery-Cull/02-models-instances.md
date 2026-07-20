# C16.2 — Models vs Instances

> **The one-sentence version:** scenery separates the *model* (a `SceneryInfo` naming a solid and its LOD
> rules) from each *placement* (a `SceneryInstance`: a transform plus an info index), so a mesh is stored once
> and placed thousands of times — and the instance is the cheap, local, reversible record you edit.

[← C16.1 — Scenery in a section](01-scenery-section.md) · [Chapter 16 hub](C16-Scenery-Cull.md) ·
[Next: C16.3 — The 64-byte SceneryInstance →](03-instance-record.md)

---

## The split

The single most important idea in scenery is the model/instance separation:

- A **`SceneryInfo`** (72 B, [C16.4](04-info-record.md)) is a *model definition* — one per distinct prop type.
  It names the solid to draw and carries draw-distance/LOD and flags.
- A **`SceneryInstance`** (64 B, [C16.3](03-instance-record.md)) is a *placement* — one per placed copy. It
  carries a transform (where) and an **index into the info table** (what).

Drawing a prop is a three-hop dereference:

```
SceneryInstance ──info_index──▶ SceneryInfo ──solid link──▶ Solid (Chapter 8)
      │
      └── transform: positions the solid in the world
```

The instance holds no geometry — only a reference and a placement. One lamppost *model*; a thousand lamppost
*instances*, each a 64-byte transform pointing at the same model.

## Why split it

The separation buys three things at once, which is why virtually every engine uses it:

- **Memory.** The lamppost mesh exists once as a solid; the thousand placements are 64-byte instances, not a
  thousand copies of the mesh. A city of props costs kilobytes of instances, not gigabytes of geometry.
- **Consistency.** Change the model (the info) and every placement updates together — one edit to draw
  distance re-tunes every lamppost in the section.
- **Cheap placement edits.** Moving, duplicating, or deleting a prop touches only its 64-byte instance, a
  local reversible change ([C16.6](06-editing-scenery.md)); the model and every other placement are
  untouched.

## The instance is the friendly edit

Because a placement is just a transform + index, the common scenery edits are all instance edits:

- **Move a prop** → change its instance's transform (and its AABB — [C16.3](03-instance-record.md)).
- **Duplicate a prop** → copy an instance, give the copy a new transform; both point at the same info.
- **Remove a prop** → delete its instance (and fix the cull tree — [C16.5](05-cull-tree.md)).
- **Re-model a prop** → point the instance at a different info index (a different model).

None of these touch geometry or the model table; they are the cheapest class of world edit and the reason
scenery modding is approachable.

## Adding a new prop type

If you want a prop the section doesn't already offer, you add a *model*, then place it:

1. **Add a `SceneryInfo`** pointing at the solid you want, with sensible draw distance/flags
   ([C16.4](04-info-record.md)).
2. **Add `SceneryInstance`s** that reference the new info's index.
3. **Update the cull tree** so the new instances live in a leaf ([C16.5](05-cull-tree.md)).

Order matters: the info must exist before instances point at it, or the info index is out of range and the
prop references a model that isn't there.

> ✅ *Verified:* the 72-byte info / 64-byte instance strides and the info-index join are decoded against
> retail data; 77,783 instances all carry an in-range info index.

## The one rule: keep the index valid

The join is only as good as the index. An instance whose `info_index` exceeds the info count references a
non-existent model — undefined at best, a crash at worst. So the invariant to preserve across any scenery edit
is simply **every instance's info index is in range**, alongside the cull tree's leaf partition
([C16.6](06-editing-scenery.md)). Keep those two and scenery edits are safe.

---

### Key takeaways

- Scenery splits *model* (`SceneryInfo`, 72 B) from *placement* (`SceneryInstance`, 64 B: transform + info
  index).
- Drawing is `instance → info → solid`; the instance carries no geometry, only a reference and a transform.
- The split saves memory, keeps a model's placements consistent, and makes placement edits cheap and local.
- Instance edits cover move/duplicate/remove/re-model; adding a prop type means adding an info *then*
  instances.
- The invariant to preserve: every instance's info index is in range (and the cull tree stays a partition).

**Continue:** [C16.3 — The 64-byte SceneryInstance](03-instance-record.md) · [Chapter 16 hub](C16-Scenery-Cull.md)
