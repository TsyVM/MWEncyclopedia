# C21.3 — Loop Points & Interactive Sections

> **The one-sentence version:** each section carries a loop range so it can be *held* indefinitely and *exited*
> on cue — which is what turns a library of fragments into an adaptive score that tightens, sustains, and
> releases with gameplay.

[← C21.2 — The SCHl/SCCl/SCDl/SCEl blocks](02-section-blocks.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md) ·
[Next: C21.4 — MPF: the PathFinder director →](04-mpf-director.md)

---

## The loop range

A section's header ([C21.2](02-section-blocks.md)) carries **loop points** — a start and end sample defining
the range that repeats. When the director ([C21.4](04-mpf-director.md)) wants to *hold* a musical state, it
plays the section into its loop and then repeats the loop range until told to move on:

```
[ intro ][ ═══ loop range ═══ ][ outro ]
          ↑ repeat here until cued to exit
```

So a section has three musical zones: a lead-in, a **loopable core** (the sustained state), and an exit. The
loop points mark the core.

## Why loops make music interactive

Loops are the mechanism behind adaptive scoring. A gameplay state has no fixed duration — a pursuit lasts as
long as it lasts — so the music for it must be **stretchable**, and a loop is exactly that: a musically-coherent
phrase that repeats seamlessly for however long the state persists.

- **Tension holds.** A pursuit-tension section loops while you're chased — one bar of music covering ten
  seconds or ten minutes.
- **States transition on cue.** When you escape, the director exits the loop (at a musical point) and moves to a
  release section ([C21.5](05-routing.md)).
- **Stingers punctuate.** Short non-looping sections fire on events (bust, win) over or between loops.

Without loops, music could only play fixed-length clips — it couldn't *sustain* a state, which is the essence
of interactive scoring.

## Seamless looping

For a loop to sound like continuous music rather than a repeating sample, the loop boundary must be **seamless**
— the audio at the loop end must flow into the audio at the loop start without a click or a beat glitch. Two
things make this work:

- **Loop points on musical boundaries.** The loop start/end land on bar or beat boundaries so the repeat is
  musically in time.
- **Sample-accurate continuity.** The waveform at the end connects to the start; with predictive codecs
  ([C20.3](../C20-Audio-Codecs/03-eaxa-decode.md)) the decoder's history must be consistent at the loop point,
  or the first samples after the loop click.

This is why editing music loops is delicate ([C21.6](06-editing-music.md)): a loop that ends a few samples off,
or on a non-musical boundary, produces an audible click or a stumble every repeat.

> ✅ *Verified (archive):* MUS sections carry loop points, and the section+loop model is what makes the
> soundtrack interactive under the director.

## Sections as musical states

Taken together, the section+loop design maps musical *states* to gameplay *states*:

| Gameplay state | Music section behaviour |
|---|---|
| calm / cruising | ambient section, gently looping |
| pursuit rising | tension section, tighter loop |
| pursuit peak | high-intensity loop |
| escape / win | release/resolve section (may not loop) |
| event (bust, milestone) | short stinger, non-looping |

The director ([C21.4](04-mpf-director.md)) walks between these as the game's state changes, and the loops let
each state hold for as long as it lasts. The music is, in effect, a **state machine of loops**.

## Editing implications

- **Keep loops on musical boundaries.** Move a loop point to a bar/beat, or the repeat stumbles
  ([C21.6](06-editing-music.md)).
- **Preserve seamlessness.** The loop end must connect to the loop start; watch codec history at the boundary
  ([C20.3](../C20-Audio-Codecs/03-eaxa-decode.md)).
- **Don't break non-looping stingers.** Some sections are meant to play once; don't add loops that turn a
  one-shot into a repeat.
- **Match the state.** A replacement section should suit the gameplay state its node represents
  ([C21.5](05-routing.md)) — a calm loop where tension is expected feels wrong.

---

### Key takeaways

- Sections carry **loop points** marking a loopable core between a lead-in and an exit.
- Loops make music **stretchable**, so a section can sustain a gameplay state of any duration.
- The soundtrack is effectively a **state machine of loops**: gameplay states map to looping (or one-shot)
  sections.
- Loops must be seamless — on musical boundaries and sample-continuous (mind codec history).
- Edit loops onto musical boundaries, preserve seamlessness, respect one-shot stingers, and match the state.

**Continue:** [C21.4 — MPF: the PathFinder director](04-mpf-director.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md)
