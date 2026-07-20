# C17.6 — Events, Messages & Editing

> **The one-sentence version:** when the car's inside-state for a trigger changes, the engine fires an
> `MTriggerEnter`/`Exit`/`Inside` message that flows to scripts — so gameplay is *data-defined regions →
> parity test → message → script*, and editing a trigger means reshaping its polygon, keeping the AABB tight,
> and repacking the variable-length chunk.

[← C17.5 — Barriers](05-barriers.md) · [Chapter 17 hub](C17-Triggers-Barriers.md) ·
[Next: C17.7 — Position markers & event sequences →](07-markers-sequences.md)

---

## From region to gameplay

The trigger system's whole point is to turn *where the car is* into *gameplay*, and it does so through a
message chain:

```
car position → even–odd test (C17.4) → inside-state change
            → MTriggerEnter / MTriggerExit / MTriggerInside message
            → Trigger::FireStateMessage → Game.* script handlers
```

When the car crosses into a trigger, its inside-state flips and the engine emits **`MTriggerEnter`**; while it
stays, **`MTriggerInside`**; when it leaves, **`MTriggerExit`**. `Trigger::FireStateMessage` routes these, and
scripts receive them as the events that *are* the gameplay — a checkpoint passed, a speed trap clocked, an
event begun. The engine provides the mechanism; the data (the regions) and the scripts (the reactions) provide
the content.

## Data + script, no engine code

This architecture is why designers can place gameplay from data alone:

- **The region is data** — a typed polygon in the trigger chunk ([C17.2](02-trigger-record.md)).
- **The reaction is script** — a handler for the trigger's message.
- **The engine is fixed** — it runs the parity test and fires messages; it doesn't know what a "checkpoint"
  means.

So a new gameplay event is a new trigger (data) plus a new handler (script) — no engine changes. This is the
same data-driven philosophy as the attribute vaults ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)):
behaviour lives in editable data, not compiled code.

## Editing a trigger

Reshaping or repurposing a trigger is a handful of coordinated changes:

1. **Reshape the polygon.** Move or add X–Y vertices to change the footprint ([C17.1](01-footprints.md)); keep
   it a *simple* polygon ([C17.4](04-even-odd.md)).
2. **Recompute the AABB.** After moving vertices, rewrite `(minX, minZ, maxX, maxZ)` at `+0x20` to bound them
   ([C17.2](02-trigger-record.md)).
3. **Keep the coarse gate covering the polygon.** Update the center/radius so the fast reject doesn't skip the
   trigger.
4. **Match the type to intent.** If you change what the trigger should do, set the right type
   ([C17.3](03-trigger-types.md)) and ensure a script handles it.
5. **Repack.** Adding vertices grows the variable-length record, shifting later records — fix the `0x80034147`
   wrapper's size and the section/stream size tree ([C15.6](../C15-Track-Streaming/06-editing-track.md)).

```python
def add_vertex(trigger, x, y):
    trigger["poly"].append((x, y))
    trigger["n_verts"] += 1
    xs = [p[0] for p in trigger["poly"]]; ys = [p[1] for p in trigger["poly"]]
    trigger["aabb"] = (min(xs), min(ys), max(xs), max(ys))   # keep AABB tight
    # record grew by 8 bytes → repack the chunk and fix wrapper size
```

## Adding a new trigger

To place a brand-new gameplay volume:

1. **Author the polygon** in the world plane (a simple footprint).
2. **Choose the type** so it lands in the right per-type list and fires the right message
   ([C17.3](03-trigger-types.md)).
3. **Fill the head** — type, coarse center/radius, tight AABB, vertex count — then the vertices
   ([C17.2](02-trigger-record.md)).
4. **Add a script handler** for its message, or reuse an existing one for that type.
5. **Repack** the chunk and fix sizes.

Without the script handler, the trigger fires messages nothing listens to — the region exists but does
nothing.

## Verify

After editing triggers, confirm:

1. every record walks cleanly (each type field lands where expected — the variable-stride walk
   [C17.2](02-trigger-record.md));
2. each polygon is simple and its AABB bounds it;
3. the `0x80034147` wrapper and section/stream sizes are consistent if records changed length;
4. in game, the trigger fires when the car enters its footprint and the script reacts.

The last step is the real test: a trigger is only correct when crossing it produces the intended gameplay.

---

### Key takeaways

- Crossing a trigger fires `MTriggerEnter/Inside/Exit` → `Trigger::FireStateMessage` → script handlers.
- Gameplay = data region + script reaction + fixed engine — a new event is a new trigger plus a handler, no
  engine code.
- Edit a trigger: reshape the polygon (keep it simple), recompute the AABB, keep the coarse gate covering it,
  match the type, and repack.
- Adding a trigger needs the right type *and* a script handler, or it fires into the void.
- Verify the record walk, polygon simplicity, AABB fit, sizes, and — decisively — the in-game gameplay.

**Continue:** [Chapter 18 — The Road Network (CARP)](../C18-Road-Network-CARP/C18-Road-Network-CARP.md) ·
[Chapter 17 hub](C17-Triggers-Barriers.md)
