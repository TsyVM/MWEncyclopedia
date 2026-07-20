# C38.2 — Sections & Residency

> **The one-sentence version:** the unit of residency is a section — a loaded block of resources — and each
> section entry in the manager's list carries a name/key pointer at `+0x08`, so "is X loaded?" is a walk of the
> list comparing keys.

[← C38.1 — The StreamMgr singleton](01-streammgr.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md) ·
[Next: C38.3 — Refcounted acquire/release →](03-refcounting.md)

---

## The section: a unit of residency

Streaming operates on **sections** — loaded blocks of resources that are resident (in memory) or not. A section
might be:

- A **world section** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — a chunk of the streamed
  city.
- A **preload set** ([C38.5](05-manifests.md)) — the resources a phase needs.
- A **car's assets** — a vehicle's geometry and textures.

Residency is per-section: a section is loaded as a unit, kept resident while needed
([C38.3](03-refcounting.md)), and freed as a unit. So the manager's list ([C38.1](01-streammgr.md)) is a list of
*sections*, and residency is tracked at section granularity.

## The section entry

Each section in the manager's list ([C38.1](01-streammgr.md)) is a record with, verified, a **name/key pointer
at `+0x08`**:

```
section entry:
  +0x00  list link (next section)
  +0x08  name / key pointer   ← identifies the section (matched by 0x460D20)
  …      residency state, refcount (C38.3), data pointers
```

The `+0x08` key is how the manager **identifies** a section — the comparison function `0x460D20` matches a
requested key against a section's `+0x08`. So finding a section is a list walk comparing `+0x08` keys, and the
key is the section's identity (a path-hash / BinHash, [C36.3](../C36-Archives-VFS/03-binhash.md), or a section
id, [C15.2](../C15-Track-Streaming/02-section-table.md)).

> ✅ *Verified:* section entries carry a name/key pointer at `+0x08`, compared by `0x460D20`; the manager's
> section list is at `[0x91A098]+0x18` ([C38.1](01-streammgr.md)).
> 🟡 *Reasoned:* the full section-entry field layout (refcount, data pointers) beyond `+0x08` is per-struct RE;
> the key at `+0x08` and its comparison are verified.

## FindResidentSection: is it loaded?

The core query is **"is section X resident?"** — implemented as `FindResidentSection`, a walk of the manager's
list comparing each entry's `+0x08` key against X's key:

```python
def find_resident_section(mgr, key):          # StreamMgr @ [0x91A098]
    node = mgr.section_list_head              # [this+0x18]
    while node:
        if key_equals(node.key_ptr, key):     # +0x08, compared by 0x460D20
            return node                        # resident
        node = node.next
    return None                                # not resident
```

This is the fundamental streaming operation: a system asks "do I have X?", the manager walks its list, and
answers. `Stream_FindSection` (`0x507E40`, [C38.1](01-streammgr.md)) is the public form, and
`Stream_BlockUntilLoaded` ([C38.6](06-blocking-budgets.md)) loops it while waiting for a load. So residency
lookups are list walks keyed on `+0x08`.

## Why sections, not individual files

Tracking residency by **section** rather than by individual resource is a granularity choice:

- **Bulk load/free.** A section loads and frees as a unit — one operation covers many resources, cheaper than
  per-resource tracking.
- **Coherent residency.** A world section's geometry, textures, and data are resident together — you never have
  half a section loaded.
- **Matches the data layout.** The world *is* sectioned on disk ([C15.1](../C15-Track-Streaming/01-master-layout.md)),
  so streaming sections mirror the file structure — load a stream-file section
  ([C15.3](../C15-Track-Streaming/03-residency.md)) as one resident section.

So the section is the natural unit: it matches the disk layout, loads coherently, and tracks residency cheaply.
The manager's list is a list of these units.

## RE implications

- **Residency is per-section** — loaded/freed as units; the manager lists sections
  ([C38.1](01-streammgr.md)).
- **A section's key is at entry `+0x08`** (compared by `0x460D20`) — its identity for lookup.
- **`FindResidentSection` walks the list** comparing `+0x08` keys — the core "is it loaded?" query.
- **Sections match the disk layout** ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)) — coherent,
  bulk-managed units.

---

### Key takeaways

- The unit of residency is a **section** — a loaded block of resources (world section, preload set, car assets).
- Each section entry carries a **name/key pointer at `+0x08`** (compared by `0x460D20`) — its identity.
- **`FindResidentSection`** walks the manager's list comparing `+0x08` keys — the fundamental "is it loaded?"
  query.
- Section granularity gives bulk load/free, coherent residency, and a match to the disk layout (Chapter 15).
- Residency lookups are list walks keyed on `+0x08`; `Stream_FindSection`/`BlockUntilLoaded` build on this.

**Continue:** [C38.3 — Refcounted acquire/release](03-refcounting.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md)
