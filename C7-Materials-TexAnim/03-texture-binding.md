# C7.3 — Binding a Texture by Asset Key

> **The one-sentence version:** a shading group does not embed a texture — it holds a reference that resolves,
> through the solid's texture table, to a 32-bit **asset key**, which the engine looks up in whatever TPK is
> bound at draw time; keep the keys and swap the pack, and the geometry is re-skinned.

[← C7.2 — The shading-group descriptor](02-shading-groups.md) · [Chapter 7 hub](C7-Materials-TexAnim.md) ·
[Next: C7.4 — Material usage-name strings →](04-usage-names.md)

---

## The indirection, end to end

Getting from a triangle to its pixels is a three-hop resolution, and understanding the hops is what lets you
re-skin, debug missing textures, and retarget materials:

```
shading group (C7.2)                 solid texture table            bound TPK (C5)
   tex_ref  ────────────────────►  entry[tex_ref] = asset key ──►  hashtable[key] → texture → pixels (C6)
```

1. The group's texture reference (`+0x38`) indexes the solid's **texture table** — a small per-solid list of
   the asset keys this solid uses.
2. That table entry is a 32-bit **asset key** — the same kind of key Chapter 5 found stored in TPK entries
   ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)).
3. At draw time a TPK is **bound** (the car's own pack, or a shared world pack); the engine scans that pack's
   hash table (`0x33310002`) for the key and gets the texture, then decodes it ([Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)).

The group therefore names *what texture it wants*, and the binding decides *which concrete pixels that is*.
The two are deliberately separate.

## Why the indirection exists: re-skinning

This is not accidental complexity — it is the mechanism that makes Most Wanted's car customisation and the
world's shared-texture economy possible:

- **Re-skinning a car.** Two liveries are two TPKs with the *same keys* mapping to *different pixels*. The
  geometry is untouched; the engine simply binds a different pack, and every group that referenced key *K*
  now draws the new art for *K*. If the keys were baked into the mesh, you could not swap skins without
  rewriting geometry.
- **Sharing world textures.** Hundreds of world solids reference a shared library of surface textures by key;
  one bound world pack serves them all. Keys are the common currency.

## The key is the asset hash, not the reflection hash

The key at the end of the chain belongs to the **asset-hash world**, and this matters for anyone trying to
compute or match it. As Chapter 5 proved across 2,012 cross-pack names, the texture key is deterministic per
name but reproduces **no** standard string hash, because it is minted by the offline packer
([C5.6](../C5-Textures-TPK/06-the-texture-key.md)). The very same is true of geometry object names: the solid
`COBALTSS_BASE_A` carries the name-hash `0x54DF8EF4`, and that value likewise matches no Joaat, Bin, CRC, or
lookup2 of the name ([C8](../C8-Geometry-Solids/C8-Geometry-Solids.md), [C7.5](05-two-hash-worlds.md)).

So both ends of the binding — the object that owns the material and the texture it points at — live in the
asset-hash world. You do not (and cannot, from shipped data) recompute these keys from names; you **read**
them and preserve them. That constraint is liberating, not limiting: it is exactly why re-skinning works by
keeping keys constant.

## Working with bindings

- **To re-skin, preserve keys.** Build your replacement TPK so each texture keeps the *original* key
  ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)); the geometry then binds your new art automatically.
- **To retarget a group to a different texture,** change the group's texture reference (or the solid's
  texture-table entry it selects), not the pixels — point it at a key that exists in the bound pack.
- **A missing texture is a binding failure, not a decode failure.** If a group draws untextured/white, the
  usual cause is that the bound TPK has no entry for the group's key — check the key exists in the pack's
  hash table before suspecting the pixels.
- **Keys are pack-relative in practice.** The same key must be present in whatever pack is bound for that
  solid; a car's keys live in the car's pack, world keys in the world pack.

> ✅ *Verified:* textures are keyed by the asset hash (C5, 2,012-name determinism); the solid's own
> name-hash is likewise asset-hash-world (`COBALTSS_BASE_A` → `0x54DF8EF4`, matching no standard hash).
> 🟡 *Reasoned:* the exact per-solid texture-table layout and the packed form of the group's `+0x38`
> reference are identified by role; the *model* (group → table → key → bound TPK) is what to rely on.

---

### Key takeaways

- A group references a texture indirectly: group `+0x38` → solid texture table → 32-bit asset key → bound
  TPK lookup → pixels.
- The indirection is the re-skinning mechanism: same keys, different pack = new art with untouched geometry.
- The key is asset-hash-world (not recomputable), and so is the solid's own name-hash — both are read and
  preserved, never regenerated.
- Re-skin by preserving keys; retarget by changing references; treat a white surface as a missing-key
  binding failure.

**Continue:** [C7.4 — Material usage-name strings](04-usage-names.md) · [Chapter 7 hub](C7-Materials-TexAnim.md)
