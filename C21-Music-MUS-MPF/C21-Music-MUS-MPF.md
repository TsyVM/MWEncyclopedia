# Chapter 21 — Music (MUS/MPF) & the Routing Graph

> **Goal of this chapter:** decode the interactive soundtrack — the MUS file of EA-stream sections with
> big-endian `SCHl` headers and loop points, and its paired MPF "PathFinder" director that maps gameplay
> events to music sections — so you understand how the music reacts to the game.

Most Wanted's soundtrack is not a set of songs that play start to finish — it is **interactive music** that
tightens in a pursuit, relaxes when you escape, and stings on events. That behaviour is two files working as a
unit: **MUS**, a sequence of loopable audio sections, and **MPF**, a director graph that decides which section
plays when. This chapter decodes both and the join between them.

> **Verified against retail data.** The music pair is confirmed in `SOUND/PFDATA/`: `MW_Music.mus`
> (533,877,888 bytes) opens with the MUS magic `0x8CA5CEFA` and contains `SCHl`/`SCCl`/`SCDl`/`GSTR` stream
> blocks; `MW_Music.mpf` (136,148 bytes) is the paired director with magic `PFDx` (PathFinder Data), version
> 5.1. The two are a matched set — the MPF routes into the MUS's sections.

---

## Deep-dive pages

- [C21.1 — MUS: EA-stream sections](01-mus-sections.md): the file magic and the section model.
- [C21.2 — The SCHl/SCCl/SCDl/SCEl blocks](02-section-blocks.md): the four block tags, the `GSTR` format, and
  the big-endian trap.
- [C21.3 — Loop points & interactive sections](03-loops-sections.md): what makes a section loopable and the
  soundtrack reactive.
- [C21.4 — MPF: the PathFinder director](04-mpf-director.md): the paired graph that drives the music.
- [C21.5 — EventID → NodeID → section](05-routing.md): how a gameplay event selects a music section.
- [C21.6 — Editing music safely](06-editing-music.md): replacing sections and re-routing within loops and
  alignment.

---

## 21.1 MUS is a sequence of sections

A MUS file (`0x8CA5CEFA` magic) is a sequence of **DWORD-aligned EA-stream sections**, each a self-contained
chunk of audio with its own format and loop points. Rather than one long stream, the soundtrack is cut into
sections that can be **entered, held, and exited on cue** — the structural basis of interactive music
([C21.1](01-mus-sections.md)). Each section's audio is one of the shared codecs
([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)).

## 21.2 Four block tags per section

Each section is built from EA-stream blocks:

| Tag | Role |
|---|---|
| `SCHl` | section **header** — codec + format, in a `GSTR` sub-block (**big-endian** fields) |
| `SCCl` | **continuation** (optional) |
| `SCDl` | **data** — the compressed audio |
| `SCEl` | **end** — finalize the section, advance to the next DWORD boundary |

Codec tags in the header: `0x00` PCM, `0x02` EA-ADPCM, `0x0A` EA-XA, `0xEA` EA-MP3. The header's format fields
are **big-endian** — the same byte-order trap as ABK `PT` records ([C19.4](../C19-Audio-Banks/04-pt-records.md));
read them little-endian and you get nonsense rates ([C21.2](02-section-blocks.md)).

## 21.3 Loop points make it interactive

Sections carry **loop points** — the sample range that repeats — so a section can be *held* indefinitely (a
pursuit tension loop) and *exited* on cue (you escape). This is what makes the soundtrack reactive: discrete
loopable sections that the director enters and leaves as gameplay demands, versus a monolithic track that can
only play through ([C21.3](03-loops-sections.md)).

## 21.4 MPF directs the music

The **MPF** file (`PFDx`, "PathFinder Data") is the **director**: a graph that maps in-game situations to music
sections. It's the "PathFinder" that finds a *path* through the music as gameplay unfolds — which section to
play, when to move, where to loop. MPF and its `.MUS` are a **unit**; neither is complete without the other
([C21.4](04-mpf-director.md)).

## 21.5 From event to section

The routing is a two-hop lookup: a gameplay **EventID** (pursuit started, heat rose, race won) maps to a
**NodeID** in the MPF graph, and the node maps to a **MUS section index**. So an event chooses a node, and the
node chooses the music ([C21.5](05-routing.md)). The `sectionIndex` is the join between the director (MPF) and
the audio (MUS) — the reason the two files are inseparable.

---

### Key takeaways

- Music is two paired files: **MUS** (loopable EA-stream sections) and **MPF** (the director graph) — verified
  in `PFDATA/`.
- A MUS section is `SCHl` (header, big-endian format in `GSTR`) + `SCDl` (data) + `SCEl` (end/align), optionally
  `SCCl`.
- Sections carry **loop points**, making the soundtrack interactive — held and exited on cue.
- **MPF** (`PFDx`, PathFinder) maps gameplay to sections; MPF and `.MUS` are one unit.
- Routing is EventID → NodeID → MUS `sectionIndex` — the join between director and audio.

**Next:** [Chapter 22 — Engine Sound (GIN) & the RPM→Synth Bridge](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md):
the car's own reactive audio.
