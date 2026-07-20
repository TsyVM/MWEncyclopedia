# C38.6 — Blocking Loads & Budgets

> **The one-sentence version:** most loading is asynchronous, but when a resource is needed *now*,
> `Stream_BlockUntilLoaded` waits — looping `FindResidentSection` and pumping deferred callbacks until the
> section arrives — while per-frame load budgets keep streaming from stalling the frame.

[← C38.5 — The preload manifests](05-manifests.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md) ·
[Next: C38.7 — Reading streaming in RE →](07-reading-streaming.md)

---

## Async by default

Streaming is **asynchronous** by default: a system requests a resource, the manager begins loading it over
subsequent frames, and the game continues. This is what keeps the open world seamless
([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — sections load ahead of the player without
stalling the frame. The load happens in the background (across frames, within a budget), and the resource
becomes resident when done.

So a request doesn't block; it schedules. The system checks later (or is notified) when the resource is ready.
This is essential for a game that must maintain frame rate while loading.

## When you must wait: BlockUntilLoaded

Sometimes a resource is needed **immediately** — a phase transition ([C38.4](04-gameflow.md)) can't proceed
without its essentials, a system needs data this frame. **`Stream_BlockUntilLoaded` (`0x503380`)** provides the
synchronous wait:

```python
def stream_block_until_loaded(key):            # 0x503380
    while find_resident_section(key) is None:   # loop FindResidentSection (C38.2)
        run_deferred_callbacks()                # PUMP: advance the async loads
    return find_resident_section(key)           # now resident
```

Verified, it **loops `FindResidentSection`** ([C38.2](02-sections-residency.md)) and, while the section is
absent, **pumps `RunDeferredCallbacks`** — driving the async load machinery forward so the load actually
completes. Without pumping, blocking would deadlock (the load never advances); by pumping, the block *drives*
the load to completion, then returns. So `BlockUntilLoaded` is a synchronous wait that keeps the async system
running underneath.

> ✅ *Verified:* `Stream_BlockUntilLoaded (0x503380)` loops `FindResidentSection` and pumps `RunDeferredCallbacks`
> while the section is absent, then returns it.

## The loading screen

`BlockUntilLoaded` is what backs the **loading screen** at a phase transition
([C38.4](04-gameflow.md)):

```
transition to phase P:
   acquire P's manifest (C38.5)                // start the loads
   Stream_BlockUntilLoaded(P's essentials)     // wait, pumping, showing the loading screen
   // essentials ready → run phase P
```

While `BlockUntilLoaded` pumps the loads, the game shows a loading screen (drawing the loading UI each pump
iteration). When the essentials are resident, the block returns and the phase runs. So the loading screen is the
*visible face* of `BlockUntilLoaded` — the game waiting (and pumping) for a phase's data. This is why loading
screens appear at transitions and their length tracks how much must load.

## Load budgets

To keep async streaming from stalling the frame ([C37.4](../C37-Frame-Spine-Modules/04-frametick.md)), the
manager works within **per-frame load budgets** — a cap on how much streaming work runs each frame:

- **Bounded per-frame work.** Only so many bytes/sections are loaded per frame, so streaming never consumes the
  whole frame and drops the rate.
- **Spread over frames.** A large load is spread across several frames, each doing a budget's worth — the world
  streams in smoothly rather than in one hitch.
- **Prioritised.** Nearer/needed-sooner sections ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md))
  are budgeted first, so the most-needed data arrives first.

So the budget is what makes async streaming *invisible*: it does a little each frame, staying within a time
allowance, so the frame rate holds while the world loads around the player. `BlockUntilLoaded`
([C38.6](06-blocking-budgets.md)) is the exception — it *does* stall (behind a loading screen) when data is
needed now.

> 🟡 *Reasoned:* per-frame load budgets bounding streaming work are the standard async-streaming design,
> consistent with the verified async manager and the frame loop ([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md));
> the exact budget values are per-system RE. `BlockUntilLoaded`'s pump-loop is verified.

## RE implications

- **Streaming is async by default** — requests schedule, don't block; the world streams within a budget.
- **`Stream_BlockUntilLoaded (0x503380)`** is the synchronous wait — loops `FindResidentSection`, pumps
  `RunDeferredCallbacks`.
- **The loading screen is `BlockUntilLoaded`** at a phase transition ([C38.4](04-gameflow.md)) — waiting +
  pumping.
- **Load budgets** bound per-frame streaming work — keeping the frame rate stable while loading.

---

### Key takeaways

- Streaming is **async by default** — requests schedule over frames within a budget, keeping the frame seamless.
- **`Stream_BlockUntilLoaded (0x503380)`** synchronously waits — looping `FindResidentSection` and **pumping**
  `RunDeferredCallbacks` so the load completes.
- The **loading screen** is `BlockUntilLoaded` at a phase transition — the visible face of waiting for a phase's
  data.
- **Per-frame load budgets** bound streaming work so it never stalls the frame — async loads spread over frames,
  prioritised.
- Async (invisible, budgeted) is the norm; blocking (loading screen) is the exception for needed-now data.

**Continue:** [C38.7 — Reading streaming in RE](07-reading-streaming.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md)
