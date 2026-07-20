# C17.3 — The 15 Trigger Types

> **The one-sentence version:** the `type` field (1–14) classifies what a trigger does — gates, engagable
> events, speed traps, checkpoints, and more — and the loader buckets records into 15 per-type lists so the
> runtime scans only the triggers relevant to each query.

[← C17.2 — The trigger record](02-trigger-record.md) · [Chapter 17 hub](C17-Triggers-Barriers.md) ·
[Next: C17.4 — The even–odd containment test →](04-even-odd.md)

---

## Type is the first thing you read

The `u32` at `+0x00` ([C17.2](02-trigger-record.md)) is the trigger's **type**, a value 1–14 that says what
kind of gameplay volume it is. Everything downstream — which list it lands in, which query finds it, what
message it fires — keys on this field. It is the trigger's identity.

## Bucketing into 15 lists

At load, `TriggerRegionChunk::Parse` sorts the flat record array into **15 per-type lists** (one per type,
plus organisation). This is a performance decision: the runtime rarely asks "is the car in *any* trigger?" —
it asks type-specific questions ("has the car hit a checkpoint?", "is it in a speed trap?"). Bucketing by type
means each query scans only the relevant handful of triggers, not all of them.

```python
def bucket_by_type(triggers):
    lists = {t: [] for t in range(1, 15)}
    for tr in triggers:
        lists[tr["type"]].append(tr)      # runtime scans lists[checkpoint], not everything
    return lists
```

## The type families

The types cover the gameplay volumes Most Wanted needs, including:

- **Gate** — a region that opens/closes a path or gates progress.
- **Engagable** — a volume that begins an interaction or event when entered (a race start, an activity).
- **Speed trap** — the photo-radar volumes that clock the car's speed as it passes.
- **Checkpoint** — race/route checkpoints that must be passed in order.
- …and further types for the game's other placed events and boundaries.

> 🟡 *Reasoned:* the exact one-to-one mapping of every numeric type 1–14 to its named role is established by
> cross-referencing the runtime's per-type handling; the ✅ verified facts are that `+0x00` is a 1–14 type
> and that the loader buckets records into 15 per-type lists. A specific record's type (`3`) is verified from
> the bytes.

## Why types matter for editing

The type isn't cosmetic — it changes behaviour, so editing it re-purposes a volume:

- **Changing a trigger's type** moves it to a different bucket and makes it fire a different kind of event —
  turning a checkpoint into a speed trap, for instance. Powerful, but only meaningful if scripts handle the new
  type ([C17.6](06-events-editing.md)).
- **Adding a trigger** requires choosing the right type so it lands in the list the runtime queries; a
  checkpoint typed as something else will never be found by the checkpoint logic.
- **The type constrains the fields.** Different types may interpret the record's flag bits or coarse gate
  differently; keep a new trigger consistent with existing ones of the same type.

## Reading the type distribution

A quick census of a track's triggers by type tells you the shape of its gameplay:

```python
from collections import Counter
dist = Counter(tr["type"] for tr in walk_triggers(chunk))
# e.g. many checkpoints on a race-heavy section, speed traps along fast roads
```

This is a useful first step when reverse-engineering an unfamiliar section: the type histogram reveals what
kind of gameplay lives there before you decode a single polygon.

---

### Key takeaways

- `+0x00` is the trigger **type** (1–14) — the volume's identity, keyed on by every downstream system.
- The loader buckets records into **15 per-type lists** so queries scan only relevant triggers.
- Types include gate, engagable, speed trap, checkpoint, and more; the precise numeric map is reasoned, the
  type-field and bucketing are verified.
- Editing the type re-purposes a volume (only useful if scripts handle the new type); adding a trigger needs
  the correct type to be found.
- A type histogram is a fast way to read what gameplay a section holds.

**Continue:** [C17.4 — The even–odd containment test](04-even-odd.md) · [Chapter 17 hub](C17-Triggers-Barriers.md)
