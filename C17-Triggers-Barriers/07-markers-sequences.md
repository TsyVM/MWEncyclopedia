# C17.7 — Position Markers & Event Sequences

> **The one-sentence version:** two more gameplay-data families ride alongside triggers — **position markers**
> (`0x00034146`, 48-byte records naming 3-D anchor points) and **event sequences** (`0x0003B811`, a `CARP` directory
> of scripted event steps) — the *anchors* and the *scripts* that triggers fire.

[← C17.6 — Events, messages & editing](06-events-editing.md) · [Chapter 17 hub](C17-Triggers-Barriers.md) ·
[Next: Chapter 18 — The Road Network (CARP) →](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)

---

## The gameplay-volume family

Triggers ([C17.2](02-trigger-record.md)) are one of a *family* of gameplay-data chunks in the world sections
([C15.7](../C15-Track-Streaming/07-section-contents.md)), and two more complete the picture of *where things happen*
and *what happens*:

- **Position markers** (`0x00034146`) — *named 3-D points*: the anchors gameplay refers to (spawn points, camera
  nodes, race positions).
- **Event sequences** (`0x0003B811`) — *scripted steps*: the ordered actions an event runs (the responses a trigger
  fires, [C17.6](06-events-editing.md)).

Where a trigger is a *where* (a 2-D region, [C17.1](01-footprints.md)) and its message a *what* ([C17.6](06-events-editing.md)),
markers are *point* wheres and sequences are *composite* whats — the anchors and the scripts of the gameplay layer.

> ✅ *Verified:* `0x00034146` (position markers) and `0x0003B811` (event sequence) are chunk IDs tested by their
> loader dispatch in `speed.exe` (`cmp dword [edx], <id>`); `0x0003B811` is wrapped by the parent `0x8003B810`.

## Position markers (`0x00034146`)

A position marker is a **named 3-D point** — a 48-byte record. After an optional leading `0x11111111` pad dword, the
records run:

```
optional pad: 0x11111111 dwords
then N x 48-byte records:
  +0x00  u32  0x0B          version pair (11, 11)
  +0x04  u32  0x0B
  +0x08  u32  nameHash      the marker's name (hashed, Ch.77)
  +0x0C  u32  index
  +0x10  f32[3]  position   (x, y, z) — a 3-D point (world, Z-up)
  +0x20  u32  paramA        +0x24  u32  paramB
  … to 48 bytes
```

The key fields are the **`nameHash`** (`+0x08`) — the marker's identity, recoverable to a name
([Chapter 77](../C77-Hash-Recovery/C77-Hash-Recovery.md)) — and the **3-D position** (`+0x10`). Unlike triggers
(2-D footprints, [C17.1](01-footprints.md)), markers are *full 3-D points*, because an anchor needs a height: a spawn
point places a car *at* a spot, a camera node sits *above* the road. So markers are the world's *named coordinates* —
the labelled points gameplay and the director ([Chapter 53](../C53-Cameras-Director/C53-Cameras-Director.md)) refer to
by name. The `0x0B` version pair opening each record is the same `{11, 11}` versioning the scenery and smackable
records use ([C63.9](../C63-Collision-World/09-smackables-emitters.md)) — a shared world-data record convention.

> ✅ *Verified:* `0x00034146` records are **48 bytes** (optional `0x11111111` pad prefix), each opening with the
> `{0x0B, 0x0B}` version pair, then `nameHash` (`+0x08`), a 3-D `position` (`+0x10`), and two params — decoded
> against retail track data.

## Event sequences (`0x0003B811`)

An event sequence is a **`CARP` directory** ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) of
scripted steps — the *same* reusable container the road network and collision packs use
([C18.1](../C18-Road-Network-CARP/01-carp-format.md), [C63.8](../C63-Collision-World/08-wcollisionpack.md)). Its
layout:

```
8 pad bytes, a control dword, zeros, then a 'CARP' directory at +0x18:
  16-byte entries: { u16 index, char tag[2], u8 typeCode, u24 size, u32 value, u32 offset }
  the 'Ni' entry's value is the sequence's NAME hash
then the script body — embedded ASCII event / sfx / particle names
```

So a sequence is a *named script*: the `Ni` directory entry gives its name hash
([Chapter 77](../C77-Hash-Recovery/C77-Hash-Recovery.md)), and the body is a list of steps referencing events, sound
cues, and particle effects *by name* (embedded ASCII strings). This is the *composite* side of the event system: a
trigger posts a message ([C17.6](06-events-editing.md)), and an event sequence is the *ordered set of actions* that
message can run — play this NIS ([Chapters 24–25](../C24-NIS-Animation/C24-NIS-Animation.md)), fire that effect
([Chapter 52](../C52-Effects-Particles/C52-Effects-Particles.md)), post the next message
([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)). That it's a `CARP`
directory is the recurring lesson ([C18.1](../C18-Road-Network-CARP/01-carp-format.md)): recognise the container, and
the road graph, the collision pack, and the event script all parse the same way.

> ✅ *Verified:* `0x0003B811` carries a `CARP` directory at `+0x18` (16-byte entries; the `Ni` entry's value is the
> sequence name hash), followed by embedded ASCII event/sfx/particle names — decoded against retail data; the
> container is the same `CARP` format as [Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md).
> 🟡 *Reasoned:* the per-entry `typeCode`/`value` semantics (which step does what) are partly undecoded; the
> directory structure, the `Ni` name entry, and the embedded string names are verified.

## The gameplay layer, complete

With markers and sequences, the world's gameplay data is complete across its chunk families:

- **Triggers** (`0x3414A`, [C17.2](02-trigger-record.md)) — *where* something is sensed (2-D regions).
- **Barriers** (`0x34190`, [C17.5](05-barriers.md)) — *where* the car is confined (2-D walls).
- **Markers** (`0x34146`, this page) — *named* 3-D anchor points.
- **Event sequences** (`0x3B811`, this page) — *scripted* composite actions.

Together they are the *gameplay geometry and scripts* laid over the world's *visual and physical* data
([C15.7](../C15-Track-Streaming/07-section-contents.md)): triggers and barriers shape *where* gameplay happens,
markers anchor its *points*, and sequences drive *what* it does — all fired through the message system
([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)) that ties the world to the
scripted flow ([Chapter 72](../C72-Lua-Scripting/C72-Lua-Scripting.md)).

## RE implications

- **Position markers** (`0x34146`) — 48-byte records; `nameHash` (`+0x08`) + 3-D `position` (`+0x10`) — named
  anchor points.
- **Event sequences** (`0x3B811`) — a `CARP` directory ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md));
  `Ni` = name hash; body references events/sfx/particles by name.
- **The gameplay family** — triggers (region) + barriers (wall) + markers (point) + sequences (script), fired via
  messages ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)).
- **`CARP` again** — the event sequence reuses the road/collision container; recognise it once, parse it everywhere.

---

### Key takeaways

- Two more gameplay-data families ride with triggers: **position markers** (`0x00034146`) and **event sequences**
  (`0x0003B811`) — the **anchors** and the **scripts** that triggers fire.
- **Position markers** are **48-byte records** — a `{0x0B, 0x0B}` version pair, a **`nameHash`** (`+0x08`), a **3-D
  position** (`+0x10`), and two params — the world's **named coordinates** (spawn/camera/race points), *3-D* because
  an anchor needs height (unlike a 2-D trigger footprint).
- **Event sequences** are a **`CARP` directory** ([Chapter 18](../C18-Road-Network-CARP/C18-Road-Network-CARP.md)) —
  the `Ni` entry names the sequence, the body references **events/sfx/particles by name** — the *composite what* a
  trigger's message runs ([C17.6](06-events-editing.md)).
- That an event sequence is **`CARP`** — the same container as the road network and collision pack — is the recurring
  lesson: **recognise the container, parse them all**
  ([C18.1](../C18-Road-Network-CARP/01-carp-format.md)).
- The four families — **triggers** (region), **barriers** (wall), **markers** (point), **sequences** (script) —
  complete the world's **gameplay layer**, fired through the message system
  ([Chapter 73](../C73-Message-Vocabulary-Stategraph/C73-Message-Vocabulary-Stategraph.md)).

**Continue:** [Chapter 18 — The Road Network (CARP)](../C18-Road-Network-CARP/C18-Road-Network-CARP.md) ·
[Chapter 17 hub](C17-Triggers-Barriers.md)
