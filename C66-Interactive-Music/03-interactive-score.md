# C66.3 — The Interactive Score

> **The one-sentence version:** the interactive score is the dynamic gameplay music — it assembles MUS/MPF segments
> live to match the moment, adapting its intensity rather than playing a fixed song, the responsive counterpart to
> the Trax jukebox.

[← C66.2 — EA Trax](02-ea-trax.md) · [Chapter 66 hub](C66-Interactive-Music.md) ·
[Next: C66.4 — Intensity & Heat →](04-intensity-heat.md)

---

## Dynamic, not fixed

The **interactive score** (`Interactive`, `InteractiveMusicVol`) is the *dynamic* half of the music
([C66.1](01-music-runtime.md)) — music that *responds* to gameplay rather than playing a fixed track like Trax
([C66.2](02-ea-trax.md)):

- **It adapts** — the score changes with the game state (calm ↔ tense), unlike a song that plays start-to-finish
  regardless of what's happening.
- **It's assembled live** — from *segments* and *transitions* (the MUS/MPF routing graph,
  [Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) — the runtime picks and sequences segments to match the
  moment.
- **It scores the drama** — pursuits ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) and tense moments,
  where fixed music couldn't respond.

So the interactive score is *adaptive music* — the MUS/MPF format ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md))
in action, assembled at runtime to fit the gameplay. Where Trax ([C66.2](02-ea-trax.md)) is a *player* (play the
song), the interactive score is a *composer* (assemble the music to the moment). `InteractiveDone` marks a score
segment/sequence completing.

> ✅ *Verified:* `Interactive`, `InteractiveDone`, `InteractiveMusicVol`, and `Intensity` are present in
> `speed.exe` — the interactive score system. The MUS/MPF routing graph is
> [Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md).

## Assembling from segments

The interactive score works by *assembling segments* — the MUS/MPF routing graph
([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) is a set of music *segments* with *transition* rules, and
the score picks a path through them to match the gameplay:

```
music = a graph of segments (calm, building, tense, climax) + transitions
   the score walks the graph:
      pursuit calm → (Heat rises) → building → (cornered) → tense → (escape) → resolve
```

So the score is a *live walk* of the music graph ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) — as the
gameplay state changes ([C66.4](04-intensity-heat.md)), the score *transitions* to the matching segment (building
tension, climaxing, resolving). The transitions are *musical* — they happen at bar boundaries so the music flows
smoothly (no jarring cuts), which the MPF routing rules ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md))
encode. This is the payoff of the MUS/MPF format ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)): it's not
just stored songs but a *composable* music system, and the interactive score is what composes it live. The runtime
assembles a *bespoke* score for each pursuit, matching its arc.

> 🟡 *Reasoned:* the segment-assembly/graph-walk model of the interactive score is the standard adaptive-music
> design, consistent with the verified `Interactive`/`Intensity` system and the MUS/MPF routing graph
> ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)); the exact segment set and transition rules are per-MPF
> data. The interactive-score system is verified.

## Why an interactive score

Having a *dynamic* score (not just licensed songs) is what makes MW's pursuits feel *cinematic*:

- **The music matches the drama.** A pursuit that *builds* — from a lone cruiser to a cornered last-stand
  ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) — needs music that *builds with it*. A fixed song
  couldn't; the interactive score does ([C66.4](04-intensity-heat.md)).
- **Every chase is scored.** Because the score adapts to *your* pursuit's arc, each chase gets a *fitted*
  soundtrack — the music climaxes when *you're* cornered, resolves when *you* escape. It's personal to the moment.
- **Emotional feedback.** The score is *feedback* ([C66.4](04-intensity-heat.md)) — the rising tension tells you
  you're in danger, the resolution tells you you're clear — an aural read of the stakes, like the busted meter
  ([C65.5](../C65-HUD-Runtime/05-gauges-meters.md)) is a visual one.

So the interactive score is MW's *cinematic* music — the adaptive soundtrack that scores each pursuit's drama in
real time. It's the responsive counterpart to Trax's ([C66.2](02-ea-trax.md)) fixed songs: Trax gives the game its
*brand*, the interactive score gives its *tension*. Together they're a soundtrack that's both recognisable and
alive. The interactive score is where the music becomes *part of the gameplay* — not a backing track, but a
dynamic response to what you do, driven by the pursuit ([C66.4](04-intensity-heat.md)).

## RE implications

- **The interactive score** is dynamic music — adapts to gameplay, unlike a fixed Trax song.
- **Assembled from MUS/MPF segments** ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) — a live walk of the
  music graph, transitioning at bar boundaries.
- **It scores the drama** — pursuits especially; every chase gets a fitted soundtrack.
- **Emotional feedback** — rising tension = danger, resolution = clear — the aural read of the stakes.

---

### Key takeaways

- The **interactive score** (`Interactive`/`InteractiveMusicVol`) is **dynamic** music — it **adapts to gameplay**,
  unlike a fixed Trax song ([C66.2](02-ea-trax.md)).
- It's **assembled live from MUS/MPF segments** ([Chapter 21](../C21-Music-MUS-MPF/C21-Music-MUS-MPF.md)) — a walk
  of the music graph (calm → building → tense → resolve), transitioning **at bar boundaries** for smooth flow.
- Where **Trax is a player** (play the song), the interactive score is a **composer** (assemble the music to the
  moment) — the payoff of the composable MUS/MPF format.
- It **scores the drama** — each pursuit gets a **fitted** soundtrack that climaxes when *you're* cornered and
  resolves when *you* escape.
- The score is **emotional feedback** — rising tension = danger, resolution = clear — the aural counterpart of the
  busted meter ([C65.5](../C65-HUD-Runtime/05-gauges-meters.md)).

**Continue:** [C66.4 — Intensity & Heat](04-intensity-heat.md) · [Chapter 66 hub](C66-Interactive-Music.md)
