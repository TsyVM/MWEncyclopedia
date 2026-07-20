# C21.1 — MUS: EA-Stream Sections

> **The one-sentence version:** a MUS file (magic `0x8CA5CEFA`) is a sequence of DWORD-aligned EA-stream
> sections — self-contained, loopable chunks of audio — that the director enters, holds, and exits on cue,
> which is what makes the soundtrack interactive rather than a single track.

[← Chapter 21 hub](C21-Music-MUS-MPF.md) · [Next: C21.2 — The section blocks →](02-section-blocks.md)

---

## The file

`MW_Music.mus` opens with the magic **`0x8CA5CEFA`** (bytes `FA CE A5 8C`) — verified on the 533 MB retail
file — and is a **sequence of sections**, each an EA-stream chunk of audio with its own format and loop points.
The sections are **DWORD-aligned**: each ends on a 4-byte boundary so the next section's header starts aligned
([C21.2](02-section-blocks.md)).

The size is telling: at ~533 MB, the MUS holds the *entire* interactive soundtrack — every tension level,
every stinger, every ambient bed — as sections the director selects among. It is one big library of musical
fragments, not a playlist.

## Why sections, not tracks

The section model is the whole reason the music can react to gameplay:

- **A track plays through.** A single continuous stream can only start, play, and stop — it can't respond.
- **Sections can be sequenced live.** Discrete sections with entry/exit points let the director
  ([C21.4](04-mpf-director.md)) choose the next fragment based on what's happening — tighten on a pursuit,
  release on an escape, sting on an event.

So cutting the soundtrack into sections is what turns "background music" into "adaptive score." Each section is
a musically-coherent piece (an intro, a loop, a transition, an ending) that the director stitches into a live
performance ([C21.3](03-loops-sections.md)).

## A section is self-contained

Each section carries everything needed to play it: its **format** (codec, rate, channels) in the header, its
**audio data**, its **loop points**, and its **end marker** ([C21.2](02-section-blocks.md)). This
self-containment is what lets the director jump to any section — there's no shared state a section depends on,
so any section can follow any other (subject to the director's musical logic). It's the same "self-describing
record" philosophy as the vault's triples ([C12.3](../C12-Reflection-Schema/03-value-triple.md)) applied to
audio.

## Walking the sections

```python
def walk_mus(buf):
    assert struct.unpack_from("<I", buf, 0)[0] == 0x8CA5CEFA
    sections, p = [], first_section_offset(buf)
    while p < len(buf):
        sec = parse_section(buf, p)           # SCHl…SCDl…SCEl (C21.2)
        sections.append(sec)
        p = align_dword(sec.end)              # SCEl advances to the next DWORD boundary
    return sections
```

The one structural subtlety is the **DWORD alignment**: `SCEl` finalises a section and advances to a 4-byte
boundary, so the walker must align `p` after each section or the next `SCHl` is misread — the music version of
the padding rule that governs EAGL chunks ([C1.4](../C1-EAGL-Container-Model/04-alignment-and-padding.md)).

## The codecs are shared

A MUS section's audio is one of the same codecs as everything else ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)) —
PCM, EA-ADPCM, EA-XA, or EA-MP3, named in the section header. So decoding a music section reuses the same
decoders as banks and engine sound; the MUS is a *container* ([C20.1](../C20-Audio-Codecs/01-codec-set.md)),
and the interactive machinery is layered over ordinary encoded audio.

## Editing implications

- **Respect DWORD alignment.** After editing a section, keep it ending on a 4-byte boundary via `SCEl`, or the
  next section desyncs ([C21.2](02-section-blocks.md)).
- **Edit per section.** Replace a section's audio in place; the section model means you change one fragment
  without disturbing the rest ([C21.6](06-editing-music.md)).
- **Keep sections self-contained.** Don't create cross-section dependencies; the director relies on any section
  being playable on its own.

---

### Key takeaways

- A MUS file (`0x8CA5CEFA`) is a sequence of **DWORD-aligned EA-stream sections** — the whole interactive
  soundtrack as fragments.
- Sections (not tracks) are what let the music react: the director sequences them live.
- Each section is self-contained (format, data, loop points, end) so any section can follow any other.
- Walk sections start-to-end, aligning to a DWORD after each `SCEl`.
- Section audio is the shared codec set; MUS is a container over ordinary encoded audio.

**Continue:** [C21.2 — The SCHl/SCCl/SCDl/SCEl blocks](02-section-blocks.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md)
