# C21.5 — EventID → NodeID → Section

> **The one-sentence version:** a gameplay event maps to a node in the MPF graph, and the node maps to a MUS
> section index — so an event chooses a node and the node chooses the music, with `sectionIndex` the join that
> makes MPF and MUS inseparable.

[← C21.4 — MPF: the PathFinder director](04-mpf-director.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md) ·
[Next: C21.6 — Editing music safely →](06-editing-music.md)

---

## The two-hop routing

The soundtrack reacts to gameplay through a two-hop lookup:

```
gameplay EventID  ──▶  NodeID (in the MPF graph)  ──▶  MUS sectionIndex  ──▶  audio
   (pursuit start)        (tension node)               (section 47)          (plays)
```

1. **EventID → NodeID.** A gameplay event (pursuit started, heat rose, race won, busted) is mapped to a node in
   the MPF's graph ([C21.4](04-mpf-director.md)) — the musical state that event should move the score toward.
2. **NodeID → sectionIndex.** The node names a MUS **section index** — which of the MUS's sections
   ([C21.1](01-mus-sections.md)) to play for that state.

So events don't name music directly; they name *nodes*, and nodes name *sections*. This indirection lets the
same event produce different music depending on the current node (an event mid-pursuit resolves differently
than the same event while calm), because the transition depends on where in the graph you are.

## sectionIndex is the join

The **`sectionIndex`** is the single value that binds the director to the audio — it's how a node in
`MW_Music.mpf` points at a section in `MW_Music.mus`. This join is why the two files are inseparable
([C21.4](04-mpf-director.md)):

- Reorder or add/remove MUS sections and every MPF `sectionIndex` may now point at the wrong section.
- Change the MPF's node→section mapping and the music plays differently for the same events.

It is the exact analogue of the scenery instance's `info_index` ([C16.3](../C16-Scenery-Cull/03-instance-record.md))
or a shading group's texture index ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)): an index that joins
two tables, valid only if kept in range and in sync.

## How an event drives the score

Walking a real transition makes it concrete:

```
you're cruising          → node: CALM        → sectionIndex: ambient loop
a cop spots you (event)  → edge to TENSION    → node: TENSION → sectionIndex: tension loop
heat rises (event)       → edge to PEAK       → node: PEAK    → sectionIndex: high-intensity loop
you escape (event)       → edge to RESOLVE    → node: RESOLVE → sectionIndex: release (one-shot)
                         → back to CALM
```

Each event advances the director along a legal edge to a new node; the node's `sectionIndex` selects the
section; the section loops ([C21.3](03-loops-sections.md)) until the next event. The music *is* the path the
events trace through the graph.

> ✅ *Verified (archive):* the routing model — EventID → NodeID → MUS `sectionIndex` — is the join between MPF
> and MUS; the `sectionIndex` is what makes them a unit.
> 🟡 *Reasoned:* the exact EventID/NodeID encoding in the MPF is the format's detail; the two-hop routing and
> the section-index join are verified.

## Editing implications

- **Keep `sectionIndex` in range and in sync.** After changing MUS sections, fix every MPF node's index, or
  nodes play the wrong (or no) section.
- **Re-route by remapping nodes.** To change what music an event brings, point its node at a different
  `sectionIndex` — the music equivalent of re-routing a sound ([C19.5](../C19-Audio-Banks/05-vocabulary-routing.md)).
- **Add states as node+section+edges.** A new musical state needs a MUS section, an MPF node with its
  `sectionIndex`, and legal edges to/from it ([C21.4](04-mpf-director.md)).
- **Test with real events.** The routing is only right when the intended music plays for the intended gameplay —
  drive a pursuit and listen ([C21.6](06-editing-music.md)).

---

### Key takeaways

- Routing is two hops: **EventID → NodeID → MUS sectionIndex → audio**; events name nodes, nodes name sections.
- The indirection lets the same event produce different music depending on the current node.
- **`sectionIndex`** joins MPF to MUS — an index valid only if kept in range and in sync (like `info_index` /
  texture index).
- An event advances the director along a legal edge; the node's section loops until the next event.
- Edit by keeping indices synced, remapping nodes to re-route, adding node+section+edges for new states, and
  testing with real gameplay.

**Continue:** [C21.6 — Editing music safely](06-editing-music.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md)
