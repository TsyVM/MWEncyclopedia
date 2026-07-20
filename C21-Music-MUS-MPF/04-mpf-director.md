# C21.4 — MPF: the PathFinder Director

> **The one-sentence version:** the MPF file (`PFDx`, "PathFinder Data") is the music director — a graph that
> finds a *path* through the MUS's sections as gameplay unfolds — and it is inseparable from its paired `.MUS`.

[← C21.3 — Loop points & interactive sections](03-loops-sections.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md) ·
[Next: C21.5 — EventID → NodeID → section →](05-routing.md)

---

## The director file

`MW_Music.mpf` (136 KB, magic **`PFDx`** = "PathFinder Data", version 5.1 — verified) is the **director** for
the soundtrack. Where the MUS ([C21.1](01-mus-sections.md)) is the *library* of musical sections, the MPF is
the *logic* that decides which section plays, when to move, and where to loop. The name is apt: it finds a
**path** through the music as the game's state changes.

## MUS and MPF are one unit

The two files are a matched pair and neither works alone:

- **MUS without MPF** is a pile of sections with no logic to sequence them.
- **MPF without MUS** is a graph of decisions pointing at audio that isn't there.

The MPF references MUS sections by index ([C21.5](05-routing.md)), so they must be authored and shipped
together — `MW_Music.mpf` + `MW_Music.mus`. Edit one's structure (add/remove/reorder sections) and the other
must follow, or the indices break ([C21.6](06-editing-music.md)).

## A graph of nodes

The MPF is structured as a **graph**: nodes represent musical states (calm, tension, peak, resolve), and edges
represent the allowed transitions between them. The director sits on a node, plays its section (looping —
[C21.3](03-loops-sections.md)), and when a gameplay event arrives, follows an edge to another node and its
section. This is the "PathFinder": it walks the graph of musical states, choosing a path that matches the
unfolding game.

```
        ┌─────────┐   pursuit    ┌──────────┐   escape   ┌──────────┐
        │  calm   │─────────────▶│ tension  │───────────▶│ resolve  │
        │ (loop)  │◀─────────────│  (loop)  │            │ (once)   │
        └─────────┘   cool down  └──────────┘            └──────────┘
```

The nodes hold (via loops) until an event triggers a transition; the edges define which moves are musically
allowed, so the score never jumps jarringly between unrelated states.

## Why a graph, not a script

Modelling the music as a graph rather than a linear script buys the same things state machines always buy:

- **Any duration.** A node loops for as long as its state lasts — the graph doesn't assume timing.
- **Legal transitions only.** Edges constrain moves to musical ones, so tension resolves gracefully rather than
  cutting to calm mid-phrase.
- **Reactive, not scripted.** The path emerges from gameplay events, so no two pursuits sound identically
  sequenced.

This is the same data-driven philosophy as triggers ([C17.6](../C17-Triggers-Barriers/06-events-editing.md))
and the vault ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)): behaviour lives in an editable
graph, and the engine just walks it.

> ✅ *Verified:* `MW_Music.mpf` is the `PFDx` PathFinder director paired with `MW_Music.mus`; it is the routing
> graph that drives interactive music.
> 🟡 *Reasoned:* the exact node/edge byte layout within the MPF is the format's detail; the director role,
> pairing, and graph model are verified, and the routing join (`sectionIndex`) is confirmed
> ([C21.5](05-routing.md)).

## Editing implications

- **Treat MUS+MPF as one deliverable.** Change sections and the MPF's indices must follow, and vice versa.
- **Respect the transition graph.** Adding a state means adding a node *and* legal edges, or the director can't
  reach it (or leaves it stuck).
- **Keep nodes pointing at valid sections.** A node whose `sectionIndex` is out of range plays nothing
  ([C21.5](05-routing.md)).
- **Version them together.** Ship the paired files as a set; a mismatched MPF/MUS desyncs the whole soundtrack.

---

### Key takeaways

- The MPF (`PFDx`, PathFinder) is the music **director** — a graph that paths through the MUS's sections.
- MUS (library) and MPF (logic) are **one unit**; the MPF references MUS sections by index.
- The graph's nodes are musical states (looping) and edges are legal transitions, walked by gameplay events.
- A graph (not a script) gives any-duration states, only-legal transitions, and reactive sequencing.
- Edit MUS and MPF together, respect the transition graph, and keep node section-indices valid.

**Continue:** [C21.5 — EventID → NodeID → section](05-routing.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md)
