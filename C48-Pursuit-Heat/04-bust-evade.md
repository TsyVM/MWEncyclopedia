# C48.4 — The Bust & the Evade

> **The one-sentence version:** being busted is a **distance/speed/time envelope**, not a vision model — inside
> the bust radius (15.0, or 90.0 when a cop is engaged), below `BustSpeed`, with the meter held past 5.0 for
> **3.0 seconds** → `MPerpBusted`; evading is the inverse — get fast, far, and out of the search phase →
> `MPerpEscaped`.

[← C48.3 — The escalation ladder](03-escalation-ladder.md) · [Chapter 48 hub](C48-Pursuit-Heat.md) ·
[Next: C48.5 — Reading pursuit in RE →](05-reading-pursuit.md)

---

## The bust is an envelope, not a sensor

A pursuit resolves in a **bust** (caught) or an **evade** (escaped) — and the bust is one of the most cleanly
byte-verified systems in the game. Crucially, **ground cops have no vision model**: being busted is a
**distance/speed/time envelope**, evaluated each frame on the *perp's* pursuit tick (`0x443BA0`). There's no
raycast, no line-of-sight cone — just three gates that must all hold:

1. **Distance** — you're inside a cop's **bust radius**.
2. **Speed** — you're below the **`BustSpeed`** threshold.
3. **Time** — the condition holds long enough (a meter fills, then a **3.0-second** hold).

All three, together, for long enough → **busted**. This is "cheap, robust, and tunable" — the same data-over-code
choice as the escalation ladder ([C48.3](03-escalation-ladder.md)).

> ✅ *Verified:* the bust is evaluated on the perp-side pursuit tick `0x443BA0`. `rh("BustSpeed")=0x769E8D9E` (a
> vault field). The constants are byte-verified in `.rdata`: bust radius **15.0** (`[0x890DAC]`) / **90.0**
> (`[0x892FA8]` when engaged), gauge threshold **5.0** (`[0x890DA4]`), hold timer **3.0** (`[0x8EB318]`). The
> outcomes `MPerpBusted` and `MPerpEscaped` are present as strings (racers use `AIRacerBusted`).

## The three gates, precisely

Each gate is a verified constant or field:

**Speed gate — `BustSpeed`.** You're bustable only *below* the vault field `BustSpeed` (hash `0x769E8D9E`), fetched
and converted km/h→m/s (× 0.2777…). Per-Heat values live in the `pursuit` levels ([C48.2](02-heat.md)) — so at
higher Heat, the speed below which you can be busted differs. Drive fast enough and you *can't* be busted, no matter
how close the cops.

**Distance gate — the bust radius.** The default bust radius is **15.0** (`[0x890DAC]`), but it swaps to **90.0**
(`[0x892FA8]`) when a cop's active goal is *engaged* (goal kind == 1) — an actively-pursuing cop reaches **six times
further**. So a committed cop can bust you from much greater range than a passive one — being *actively chased*
close is far more dangerous than merely near a cruiser.

**Time gate — the meter and the hold.** A **bust meter** (`[perp+0x120]`) fills while you're in the envelope:
`dt × 0.25` normally, `dt × 4.0` when a cop is *engaged* (16× faster), and *drains* at `dt × 0.5` when you're
outside. Once it crosses **5.0** (`[0x890DA4]`), you're flagged bustable and a **3.0-second** timer (`[0x8EB318]`)
runs; if the condition holds through it → `MPerpBusted`. So the bust isn't instant — you must be trapped in the
envelope long enough for the meter to fill *and* survive the 3-second hold.

> 🟡 *Reasoned:* the interpretation of the fill/drain rates and the engaged-multiplier as "why an actively-chased,
> boxed-in, slow car gets busted fast" is the natural reading; the *constants and their addresses* are all
> byte-verified above.

## The busted meter is the HUD bar

The bust meter (`[perp+0x120]`) *is* the **busted bar** you see on screen — the HUD is notified on every
0.1-quantized change of the gauge (via a vtable slot). So the bar filling up as cops box you in is a direct readout
of the meter approaching 5.0 — it's not a separate UI abstraction, it's the same value. This is why the bar fills
faster when you're surrounded and stopped (engaged cops, `dt × 4.0`) and drains when you break away (`dt × 0.5`):
the bar is the envelope's state, shown to you. Watching the busted bar, you're watching the bust envelope compute.

## Evasion: the inverse

Escaping (`MPerpEscaped`) is the inverse of the bust envelope ([above](#the-three-gates-precisely)) — defeat the
gates:

- **Get fast** — climb above `BustSpeed` so the speed gate fails; you can't be busted while quick.
- **Get far** — leave the bust radius (harder against *engaged* cops at 90.0) so the distance gate fails and the
  meter drains.
- **Break the pursuit phase** — the pursuit has a *search/cooldown* phase; once cops lose you (no engaged cop in
  range), the pursuit's cool-down timer ([C48.1](01-the-cast.md)) runs, and if you stay clear, the pursuit ends
  (`PursuitEnds`/`PursuitOver`). The `FleePursuit` brain ([C46.4](../C46-AI-Goals-Actions/04-override-goals.md)) is
  what AI evaders use to do this.

Critically, because there's **no vision model**, you *cannot* "hide in plain sight" from a ground cop by breaking
line-of-sight — you escape by being **fast, far, or out of the search phase**, not by hiding. This is the *feel* the
envelope produces: evasion is about speed and distance, not stealth. (The only "line of sight" is the helicopter's
minimap overlay — `HELICOPTER_LINE_OF_SIGHT` is consumed by the world-map screen, a *presented* detection circle,
not a raycast.)

## Why an envelope, not a sensor

Modelling the bust as a distance/speed/time envelope (rather than a perception simulation) is a deliberate,
excellent choice:

- **Cheap.** No raycasts or occlusion tests per cop per frame — just distance/speed/time comparisons. Scales to a
  swarm of cops trivially.
- **Robust.** No "cop can't see me through a thin wall" bugs — the envelope is geometric and predictable.
- **Tunable per Heat.** `BustSpeed` is a per-Heat vault field ([C48.2](02-heat.md)); the radii and rates are
  constants — the whole difficulty of being caught is dialable through data and a handful of `.rdata` numbers.
- **It shapes the feel.** The envelope *is* the reason Most Wanted's evasion feels the way it does — a frantic
  push for speed and distance, not a stealth game. You escape by *outrunning*, which is what a racing game should
  reward.

So the bust envelope is a model of pragmatic design: the simplest mechanism that produces the right feel, byte-verified
and data-tunable. It's the resolution of every pursuit, and it's just three numbers held for three seconds.

## RE implications

- **The bust is a distance/speed/time envelope**, not a vision model — evaluated on the perp tick `0x443BA0`.
- **Three gates** — inside the bust radius (15.0, or 90.0 engaged), below `BustSpeed`, meter past 5.0 held 3.0 s →
  `MPerpBusted`.
- **The busted bar is the meter** (`[perp+0x120]`) — fills `dt×0.25` (×4.0 engaged), drains `dt×0.5`.
- **Evade by fast/far/out-of-phase**, not stealth — no ground-cop line-of-sight; escape is `MPerpEscaped`.

---

### Key takeaways

- Being busted is a **distance/speed/time envelope** (no vision model): inside the **bust radius** (15.0, or
  **90.0** when a cop is engaged), below **`BustSpeed`**, with the meter held past **5.0** for **3.0 seconds** →
  `MPerpBusted` — all **byte-verified**.
- An **engaged** cop is far more dangerous — 6× the radius (90 vs. 15) and 16× the meter fill rate (`dt×4.0` vs.
  `dt×0.25`).
- The **busted bar is the meter** (`[perp+0x120]`) — you watch the envelope compute; it drains (`dt×0.5`) when you
  break away.
- **Evasion is the inverse** — get **fast** (above `BustSpeed`), **far** (out of radius), or **out of the search
  phase** → `MPerpEscaped`; you **cannot hide** in plain sight (no ground-cop line-of-sight).
- The envelope is **cheap, robust, and tunable per Heat** — the simplest mechanism that produces MW's outrun-not-hide
  evasion feel.

**Continue:** [C48.5 — Reading pursuit in RE](05-reading-pursuit.md) · [Chapter 48 hub](C48-Pursuit-Heat.md)
