# C25.1 — Event Sequences Are CARP Scripts

> **The one-sentence version:** a cutscene's timeline is an `EventSequenceChunk` (`0x0003B811`, wrapped by
> `EventSequencePack` `0x8003B810`) that — after an 8-byte `0x11` sentinel — is a CARP attribute blob, so you
> parse it as a CARP directory, not with the universal chunk walker.

[← Chapter 25 hub](C25-NIS-Events.md) · [Next: C25.2 — The EventSequenceChunk format →](02-sequence-format.md)

---

## The same CARP again

The event script uses the **CARP** attribute-blob format you already met as the road network
([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)). Verified: the `EventSequenceChunk`
(`0x0003B811`) in `L2RA.BUN`, after its 8-byte `0x11` sentinel, carries the **`CARP`** magic (stored reversed
as `PRAC`) — and the NIS scene bundle carries CARP event data too. So the cutscene timeline is *attribute
data*, and the critical rule from Chapter 18 applies verbatim:

> **CARP is not a chunk tree.** Do not walk `EventSequenceChunk` with the universal EAGL reader
> ([Chapter 1](../C1-EAGL-Container-Model/C1-EAGL-Container-Model.md)); parse it as a CARP tag directory
> ([C18.1](../C18-Road-Network-CARP/01-carp-format.md)), or you'll misread its tags as chunk headers and,
> writing back, corrupt the script.

## Two chunks: pack and chunk

The event data comes as a pair:

- **`0x8003B810 EventSequencePack`** — the container/wrapper.
- **`0x0003B811 EventSequenceChunk`** — the CARP blob itself (one or more per pack).

A world or scene can hold several event chunks (the retail track has many `0x0003B811`s), each a CARP script
for some sequence — a cutscene, a scripted world event, a triggered moment. They share the format; they differ
in content.

## Why CARP for scripts too

CARP is EA's general attribute-blob format, and it suits event scripts for the same reasons it suits the road
graph:

- **Tabular, indexed data.** A script is a registry of entities plus a schedule of timed events
  ([C25.2](02-sequence-format.md)) — arrays of typed records indexed by a directory, which CARP's tag directory
  serves cleanly.
- **Self-describing.** The CARP tags name each table (registry, schedule, parameters), so a reader finds them
  without hard-coded offsets.
- **Reused format.** EA already had CARP (for road networks, car attributes); using it for event scripts avoided
  a new format — the same economy as the shared codec layer ([C20.1](../C20-Audio-Codecs/01-codec-set.md)).

So "the cutscene script is CARP" is the same design instinct as "the road network is CARP": tabular attribute
data in a self-describing directory.

> ✅ *Verified:* `EventSequenceChunk` (`0x0003B811`) is a CARP blob (`PRAC` magic after the 8-byte sentinel),
> wrapped by `EventSequencePack` (`0x8003B810`); NIS bundles contain CARP event data.

## Parsing it

```python
def read_event_sequence(chunk_0003B811):
    body = chunk_0003B811.payload[8:]         # skip the 8-byte 0x11 sentinel
    assert body[:4] == b"CARP" or body[:4] == b"PRAC"[::-1] or has_carp_magic(body)
    return parse_carp(body)                    # tag directory → registry + schedules (C25.2)
```

Skip the sentinel, confirm CARP, and parse the tag directory — then the registry and schedules
([C25.2](02-sequence-format.md)) fall out of the tables.

## Editing implications

- **Branch CARP out of the chunk walker** — the same rule as the road network
  ([C18.1](../C18-Road-Network-CARP/01-carp-format.md)); walking it as chunks corrupts it on write.
- **Read tags reversed.** CARP tags are stored reversed (`PRAC` = `CARP`).
- **Treat the pack and chunk together.** Edits to a chunk may need the `EventSequencePack` wrapper's size fixed
  ([C25.6](06-editing-scripts.md)).
- **Preserve the sentinel.** The 8-byte `0x11` prefix is part of the chunk.

---

### Key takeaways

- A cutscene timeline is an **`EventSequenceChunk`** (`0x0003B811`) — a **CARP** attribute blob (verified),
  wrapped by `EventSequencePack` (`0x8003B810`).
- CARP is **not a chunk tree** — parse it as a tag directory, never with the universal walker.
- CARP suits scripts (tabular registry + schedule), is self-describing, and reuses EA's existing format.
- Skip the 8-byte sentinel, confirm CARP, and parse the directory into registry + schedules.
- Branch CARP out of the walker, read tags reversed, keep pack+chunk consistent, and preserve the sentinel.

**Continue:** [C25.2 — The EventSequenceChunk format](02-sequence-format.md) · [Chapter 25 hub](C25-NIS-Events.md)
