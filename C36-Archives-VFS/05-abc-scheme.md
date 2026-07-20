# C36.5 — The A/B/C Bundle Scheme

> **The one-sentence version:** bundles load in layered tiers — a base "A", then "B", then "C" — so later
> layers extend or override earlier ones, and a path resolves to the highest layer that provides it.

[← C36.4 — MemoryFile intercept](04-memoryfile.md) · [Chapter 36 hub](C36-Archives-VFS.md) ·
[Next: C36.6 — Loading a resource end to end →](06-loading.md)

---

## Layered loading

The game composes its data from **layered bundles** in an A/B/C scheme. Rather than one monolithic data set,
resources load in tiers, and later tiers **extend or override** earlier ones:

```
A (base)     ─ the foundational global data
B (layer)    ─ context/phase data, overriding/adding to A
C (layer)    ─ further overrides/additions
```

The naming shows up in the files — `GLOBALA.BUN`/`GlobalB.lzc` (A and B tiers of the global data),
and the streaming/loading system composes them. So the active data at any moment is the *composition* of the
loaded layers, with higher layers winning where paths overlap.

## Resolution: highest layer wins

The VFS ([C36.2](02-vfs.md)) resolves a path against the loaded layers, and the **highest layer that provides it
wins**:

```python
def resolve(path, layers):                 # layers = [A, B, C], low → high
    key = bin_hash(path)                    # C36.3
    for layer in reversed(layers):          # check C, then B, then A
        if key in layer.index:
            return layer.get(key)           # highest layer providing it
    return None
```

So a resource in the base (A) can be **overridden** by a later layer (B or C) providing the same path — the VFS
returns the override. This is transparent to the requesting code ([C36.2](02-vfs.md)): it asks for a path and
gets whatever the composed layers currently provide.

## Why layer

The A/B/C scheme buys the composition flexibility a large game needs:

- **Shared base + context specifics.** The A layer holds data common to everything; B/C layers add context-
  specific data (a phase, a mode, a track) — loaded and unloaded as context changes
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
- **Overrides without replacement.** A later layer can override a base resource without editing the base — the
  mechanism a patch or DLC would use to change content additively.
- **Memory scoping.** Layers load/unload as needed, so only the relevant tiers are resident — the streaming/
  residency system ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md))
  manages which layers are live.

So the game's data at any time is the base plus the currently-loaded context layers, composed by the VFS.

> 🟡 *Reasoned:* the A/B/C layered-loading-and-override scheme is documented and reflected in the file naming
> (`GLOBALA`/`GlobalB`); the exact layer-composition code is per-system RE. The VFS resolution (highest layer
> wins) follows from the layering and the path-hash lookup ([C36.2](02-vfs.md)).

## Layers and phases

The layers tie to the game's **phases** ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)):

- **Front-end phase** loads the front-end layers ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)).
- **In-game phase** loads the world/gameplay layers ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).
- **Global** underlies all phases (the A base).

So as the game moves between phases (front-end → race → free-roam), the loaded layers change, and the composed
data changes with them — the VFS always resolving to the current composition. This is why the same code sees
different data in different phases: the layers beneath it changed.

## Editing implications

- **Override via a higher layer** — add a resource at a later layer to override a base one, without editing the
  base ([C36.2](02-vfs.md)).
- **Know which layer wins** — a path resolves to its highest-loaded layer; a base edit is invisible if a higher
  layer overrides it.
- **Layers are phase-scoped** — edit the layer loaded in the phase you're targeting
  ([Chapter 38](../C38-Resource-Streaming-Residency/C38-Resource-Streaming-Residency.md)).
- **The composition is dynamic** — resident layers change with the phase.

---

### Key takeaways

- Bundles load in a layered **A/B/C** scheme; later layers **extend or override** earlier ones.
- The VFS resolves a path to the **highest layer providing it** — overrides are transparent.
- Layering gives a shared base + context specifics, additive overrides (patch/DLC), and memory scoping.
- Layers tie to **phases** (front-end, in-game, global); the composed data changes as phases change.
- Override via a higher layer (not by editing the base); know which layer wins; layers are phase-scoped.

**Continue:** [C36.6 — Loading a resource end to end](06-loading.md) · [Chapter 36 hub](C36-Archives-VFS.md)
