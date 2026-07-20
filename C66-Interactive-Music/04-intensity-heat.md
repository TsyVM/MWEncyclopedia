# C66.4 — Intensity & Heat

> **The one-sentence version:** `Intensity` is the scalar that drives the interactive score's tension, mapped from
> the game state — Heat and the bust envelope raise it in a pursuit (urgent when cornered, resolving on escape) —
> driven by `SoundAI`, the fleet→audio bridge.

[← C66.3 — The interactive score](03-interactive-score.md) · [Chapter 66 hub](C66-Interactive-Music.md) ·
[Next: C66.5 — Reading music in RE →](05-reading-music.md)

---

## Intensity: the tension scalar

The interactive score ([C66.3](03-interactive-score.md)) adapts through one master control — **`Intensity`**, a
scalar that drives *how tense the music is*:

- **Low intensity** — calm/ambient segments ([C66.3](03-interactive-score.md)) — the quiet of free-roam or a
  cooling pursuit.
- **High intensity** — building/climax segments — the urgency of a hot pursuit
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)).
- **Intensity selects the segment** — the score walks the music graph ([C66.3](03-interactive-score.md)) toward
  segments matching the current intensity, so the music's tension *tracks* the scalar.

So `Intensity` is the *one dial* that shapes the interactive score — raise it and the music tenses, lower it and it
calms. This is the adaptive-music equivalent of Heat ([C48.2](../C48-Pursuit-Heat/02-heat.md)): one scalar
controlling the whole system's character. The question is *what sets Intensity* — and the answer is the game state,
chiefly the pursuit ([below](#mapped-from-the-pursuit)).

> ✅ *Verified:* `Intensity` is present in `speed.exe` — the interactive score's tension scalar; `SoundAI`
> ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)) is the AI→audio bridge that drives it.

## Mapped from the pursuit

`Intensity` is *mapped from the game state* — most dramatically the **pursuit**
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)):

- **Heat raises intensity** — as Heat climbs ([C48.2](../C48-Pursuit-Heat/02-heat.md)), the score intensifies —
  more cops, more danger, more urgent music.
- **The bust envelope spikes it** — when you're *cornered* (the busted meter filling,
  [C48.4](../C48-Pursuit-Heat/04-bust-evade.md)), intensity peaks — the music climaxes at the moment of maximum
  danger.
- **Escape resolves it** — breaking away ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)) drops intensity — the
  music resolves, releasing the tension (the triumphant "you made it" cue).

So the interactive score *tracks the pursuit's arc*: it builds as Heat rises, climaxes as you're cornered, and
resolves as you escape. The music *is* the pursuit's emotional shape, mapped through `Intensity`. This is why MW's
pursuits feel so *scored* — the music isn't a fixed backing track but a live response to *your* chase's tension
([C66.3](03-interactive-score.md)). The mapping (pursuit state → intensity) is what makes the score *interactive*:
it responds to what's happening to *you*.

> 🟡 *Reasoned:* the Heat/bust-envelope → intensity mapping is the natural reading of the verified `Intensity`
> system and the pursuit's stakes ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)); the exact mapping curve is
> per-config. The `Intensity` scalar and its role are verified.

## SoundAI: the fleet→music bridge

The mapping from pursuit to intensity is driven by **`SoundAI`** ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md))
— the AI→audio bridge that translates *fleet/chase state* into the soundscape:

- **It reads the pursuit** — `SoundAI` ([C47.3](../C47-AI-Driver-Vehicle/03-managers.md)) sees the whole pursuit
  state (Heat, cops engaged, the bust envelope, [C48.4](../C48-Pursuit-Heat/04-bust-evade.md)).
- **It drives the intensity** — translating that state into the score's `Intensity` ([above](#intensity-the-tension-scalar))
  — and the siren chorus, and the chatter ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)).
- **It's the fleet-level audio** — `SoundAI` is *the* bridge from the pursuit system
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) to the whole pursuit soundscape (music + sirens +
  callouts).

So `SoundAI` ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)) is what *connects* the pursuit to the music —
it's the piece that reads "the pursuit is escalating" and *raises the intensity* accordingly. This ties three
systems together: the pursuit ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) produces the state,
`SoundAI` ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)) reads it, and the interactive score
([C66.3](03-interactive-score.md)) responds via `Intensity`. The music's tension is thus a *direct function* of the
chase, bridged by `SoundAI`. Reading the interactive score means reading this chain: pursuit → `SoundAI` →
`Intensity` → the music.

## The score completes the pursuit's presentation

The interactive score completes the *multi-channel presentation* of the pursuit
([C65.5](../C65-HUD-Runtime/05-gauges-meters.md)) — the chase is conveyed through *every* channel at once:

- **Visual** — the busted meter ([C65.5](../C65-HUD-Runtime/05-gauges-meters.md)), the flashers, the visual
  treatment ([C51.4](../C51-Render-Pipeline/04-visual-treatment.md)) shifting with Heat.
- **Sound** — the siren chorus ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)), the chatter.
- **Music** — the interactive score's intensity (this chapter).
- **Haptic** — the wheel's force feedback ([C60.3](../C60-Input-Devices/03-devices.md)) on impacts.

So a pursuit's *stakes* are read through vision (meters), sound (sirens), music (the score), and touch (the wheel)
— *all* driven by the same pursuit state ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), each a channel of
the one drama. The interactive score is the *musical* channel — arguably the most *emotional*, because music
directly shapes feeling. When you're cornered, the busted meter *shows* the danger, the sirens *surround* you, and
the score *makes you feel* it — a total sensory conveyance of the moment. This multi-channel presentation, all from
one state, is why MW's pursuits are so *immersive*: every sense is telling you the same story, and the interactive
score is the one that scores it.

## RE implications

- **`Intensity`** is the interactive score's tension scalar — selects calm ↔ climax segments.
- **Mapped from the pursuit** — Heat raises it, the bust envelope spikes it, escape resolves it — the score tracks
  the chase's arc.
- **Driven by `SoundAI`** — the fleet→audio bridge reads the pursuit state and drives the intensity.
- **The music completes the multi-channel presentation** — visual + sound + music + haptic, all from the pursuit
  state.

---

### Key takeaways

- **`Intensity`** is the interactive score's **tension scalar** — low = calm segments, high = climax segments; it
  **selects the music segment** ([C66.3](03-interactive-score.md)), so the music's tension tracks the scalar.
- Intensity is **mapped from the pursuit** — **Heat raises** it, the **bust envelope spikes** it (climax when
  cornered), **escape resolves** it — the score **tracks the chase's arc**.
- The mapping is driven by **`SoundAI`** ([C59.4](../C59-Audio-Runtime/04-sound-connectors.md)) — the fleet→audio
  bridge that reads the pursuit state and drives the intensity (and sirens, and chatter).
- The chain is **pursuit → `SoundAI` → `Intensity` → the music** — the score's tension is a **direct function of
  the chase**.
- The interactive score completes the pursuit's **multi-channel presentation** — visual (meters) + sound (sirens) +
  **music** (the score) + haptic (the wheel) — all from one state; the score is the **emotional** channel.

**Continue:** [C66.5 — Reading music in RE](05-reading-music.md) · [Chapter 66 hub](C66-Interactive-Music.md)
