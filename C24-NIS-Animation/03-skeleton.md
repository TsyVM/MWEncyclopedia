# C24.3 — The Bind-Pose Skeleton

> **The one-sentence version:** the rig falls out of the ELF symbol table — each bone and skeleton is a named
> symbol pointing at its rest transform in `.data` — so the bind-pose hierarchy is recoverable directly, even
> though the motion (keyframes) is not.

[← C24.2 — Parsing the MIPS ELF32](02-parsing-elf.md) · [Chapter 24 hub](C24-NIS-Animation.md) ·
[Next: C24.4 — Skeletons vs animation banks →](04-skeletons-banks.md)

---

## The rig is in the symbols

A **skeleton** is a hierarchy of **bones**, each with a rest (bind-pose) transform and a parent. An ELF symbol
table is the ideal container for this, because a symbol is exactly "a name pointing at some data" — so each
bone is a symbol naming it and pointing at its transform, and each skeleton is a symbol grouping its bones
([C24.2](02-parsing-elf.md)):

```python
def read_skeleton(elf):
    bones = []
    for sym in elf.symtab:
        name = elf.strtab[sym.st_name]           # bone / skeleton name
        if is_bone(name):
            data = elf.data_at(sym.st_value, sym.st_size)
            bones.append({"name": name, "bind_pose": parse_transform(data)})
    return build_hierarchy(bones)                # parent/child from names or structure
```

Walking `.symtab`, resolving names via `.strtab`, and reading each bone's transform from `.data` reconstructs
the rig — the bind pose and, from the naming/structure, the hierarchy.

## The bind pose

The **bind pose** is the skeleton's rest configuration — the pose in which the character mesh is authored,
before any animation. Each bone's bind-pose transform (its rest position and orientation relative to its
parent) is what the animation later *deviates from*: a keyframe says "this bone is rotated *this much* from its
bind pose at *this time*." So the bind pose is the reference frame for all motion, which is why recovering it
matters even without the keyframes ([C24.5](05-keyframe-problem.md)).

## What the skeleton gives you

With the bind-pose skeleton decoded, you can:

- **Reconstruct the rig** — the full bone hierarchy and rest pose of the cutscene's characters.
- **Render the static pose** — the characters standing in bind pose (which, on PC, is close to what actually
  shows — [C24.6](06-ps2-vs-pc.md)).
- **Understand the animation's target** — the keyframes ([C24.4](04-skeletons-banks.md)) animate *these* bones,
  so the skeleton is the map for interpreting motion data once it's decoded.

The skeleton is the solved half of NIS animation — the rig is fully in hand.

> ✅ *Verified (archive):* the bind-pose skeleton is recoverable from the ELF symbol table — the symbols name
> bones/skeletons and point at their data; this is the decoded half of the NIS animation format.
> 🟡 *Reasoned:* the exact per-bone transform encoding in `.data` is read per symbol; the "rig from symbols"
> structure is verified.

## Names carry the hierarchy

ELF symbol names are human-readable, so the skeleton's structure is legible: bone names typically encode their
role and often their parentage (a naming convention like `Bip01_L_UpperArm` implies a hierarchy). This is the
same gift the geometry usage names give ([C7.4](../C7-Materials-TexAnim/04-usage-names.md)) — the data
documents itself. So even before you fully model the transform data, the symbol names tell you what the rig *is*
(a humanoid, a vehicle rig, a camera) and roughly how it's structured.

## Editing implications

- **Edit the skeleton through symbols.** Bones are symbols; change a bind-pose transform by editing its
  symbol's `.data`, keeping the symbol table consistent ([C24.2](02-parsing-elf.md)).
- **The bind pose is the reference.** Change it and every keyframe (which is relative to it) shifts — so
  bind-pose edits ripple into the motion ([C24.5](05-keyframe-problem.md)).
- **Preserve the hierarchy.** Bone parentage matters for how transforms compose; don't break the naming/
  structure that encodes it.
- **Skeleton edits are the tractable ones.** Because the rig is decoded (unlike the keyframes), pose/structure
  edits are far more feasible than motion edits.

---

### Key takeaways

- The **skeleton** (bones + bind-pose transforms + hierarchy) is recoverable directly from the ELF
  symbol table.
- Each bone is a named symbol pointing at its rest transform in `.data`; skeletons group bones.
- The **bind pose** is the rest configuration that all animation deviates from — the reference frame for motion.
- Symbol **names** carry the rig's identity and structure, documenting the skeleton like usage names document
  materials.
- The skeleton is the **solved half** of NIS animation; edit it through symbols, mindful that bind-pose changes
  ripple into keyframes.

**Continue:** [C24.4 — Skeletons vs animation banks](04-skeletons-banks.md) · [Chapter 24 hub](C24-NIS-Animation.md)
