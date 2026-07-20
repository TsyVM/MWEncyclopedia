# C38.1 — The StreamMgr Singleton

> **The one-sentence version:** the streaming manager is a single global object at `[0x91A098]`, constructed in
> `GameInit`, that owns a linked list of resident sections (head at `[this+0x18]`) — the one place that knows
> what data is in memory.

[← Chapter 38 hub](C38-Resource-Streaming-Residency.md) · [Next: C38.2 — Sections & residency →](02-sections-residency.md)

---

## One manager, one list

The streaming manager is a **singleton** — a single global object at the fixed address **`[0x91A098]`**
(verified, 30 references throughout the binary). It's constructed once in `GameInit`
([C37.3](../C37-Frame-Spine-Modules/03-gameinit.md)) and lives for the whole session. Its central data is a
**linked list of resident sections**, whose head is at **`[this+0x18]`**:

```
StreamMgr @ [0x91A098]
+0x18  → section list head ──▶ section ──▶ section ──▶ …   (the resident resources)
```

So "what's in memory?" is answered by walking this one list ([C38.2](02-sections-residency.md)). Every streaming
operation — load, find, acquire, release — operates on this singleton, which is why `[0x91A098]`'s 30 references
are the map of the streaming system: each is code touching the resident set.

## The singleton pattern

Making the manager a singleton is deliberate: there is exactly **one** authority on residency, and all systems
consult it:

- **One source of truth.** Whether a section is resident is decided in one place — no two managers disagreeing.
- **Global access.** Any system, anywhere, reaches the manager via `[0x91A098]` (through the forwarders,
  [C38.6](06-blocking-budgets.md)) — the front-end, the world, a car loader.
- **One list to maintain.** Adding/removing a resident section touches one list, keeping residency coherent.

This is the same singleton shape as other engine managers (the family list-heads,
[C32.3](../C32-Runtime-Class-System/03-eleven-families.md)); streaming's is `[0x91A098]`.

> ✅ *Verified:* the streaming manager is the singleton `[0x91A098]` (30 references), with its section list head
> at `[this+0x18]`; it's the authority on resident resources.

## The public forwarders

Systems don't touch `[0x91A098]` directly — they call **public forwarder** functions that wrap the singleton
([C38.6](06-blocking-budgets.md)):

| Forwarder | Address | Role |
|---|---|---|
| `Stream_FindSection` | `0x507E40` | find a resident section by key |
| `Stream_AcquireResources` | `0x5033C0` | acquire (refcount++) a set of resources |
| `Stream_ReleaseResources` | `0x503360` | release (refcount--) — **37 callers** |
| `Stream_BlockUntilLoaded` | `0x503380` | wait for a section, pumping callbacks |

These thin wrappers over the `[0x91A098]` singleton are what call sites use — so the 30 references to the
singleton are mostly *inside* these forwarders, and the forwarders' many callers (37 for Release) are the
systems doing streaming. So the API surface is a handful of `Stream_*` functions
([C38.6](06-blocking-budgets.md)); the singleton is the implementation behind them.

## Constructed in GameInit

The manager is built in `GameInit` ([C37.3](../C37-Frame-Spine-Modules/03-gameinit.md)) — early, because most
other subsystems need to load their data ([Chapter 36](../C36-Archives-VFS/C36-Archives-VFS.md)) as they
construct. So the streaming manager's construction precedes the constructors that stream: the VFS opens
([C37.3](../C37-Frame-Spine-Modules/03-gameinit.md)), the StreamMgr is created, and then the systems that load
resources are built. This ordering ([C37.3](../C37-Frame-Spine-Modules/03-gameinit.md)) is why streaming is
available from early in startup.

## RE implications

- **The streaming manager is `[0x91A098]`** — the singleton authority on residency; its 30 refs map the system.
- **The resident-section list is at `[this+0x18]`** — walk it to see what's in memory
  ([C38.2](02-sections-residency.md)).
- **Systems use the `Stream_*` forwarders**, not the singleton directly — the API surface
  ([C38.6](06-blocking-budgets.md)).
- **Constructed early in `GameInit`** — before the subsystems that stream their data.

---

### Key takeaways

- The streaming manager is a **singleton at `[0x91A098]`** (30 refs), constructed in `GameInit`.
- Its core data is a **linked list of resident sections** (head at `[this+0x18]`) — the record of what's in
  memory.
- Being a singleton gives one source of truth, global access, and one list to maintain.
- Systems call **`Stream_*` forwarders** (`FindSection`/`Acquire`/`Release`/`BlockUntilLoaded`), not the
  singleton directly.
- Constructed early in `GameInit` so streaming is available before subsystems load their data.

**Continue:** [C38.2 — Sections & residency](02-sections-residency.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md)
