# C38.5 — The Preload Manifests

> **The one-sentence version:** each phase's resources are named by preload manifests — the Global, Permanent,
> InGame, and front-end lists (the `MemoryFile` manifests) — so entering a phase loads its manifest's contents
> to make the phase ready.

[← C38.4 — GameFlow phases](04-gameflow.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md) ·
[Next: C38.6 — Blocking loads & budgets →](06-blocking-budgets.md)

---

## Manifests name a phase's resources

A phase ([C38.4](04-gameflow.md)) becomes ready by loading a defined set of resources, and that set is named by
a **preload manifest** — a list of what to load. The manifests correspond to the **`MemoryFile` manifests**
([C36.4](../C36-Archives-VFS/04-memoryfile.md)) — the `*MemoryFile.bin` files marked by the sentinel
`0x53219999` — which name the memory-resident and preloaded files per scope:

- **`GlobalMemoryFile.bin`** — global resources, resident broadly.
- **`PermanentMemoryFile.bin`** — permanently-resident data (small, 3200 bytes — a core set).
- **`InGameMemoryFile.bin`** — the in-game phase's resources.
- **Front-end** — the front-end phase's resources.

So there are four preload scopes, one per residency tier, and each names the files a phase acquires
([C38.3](03-refcounting.md)) on entry.

## The four scopes

The four manifests map to the residency scopes ([C38.4](04-gameflow.md)):

| Manifest | Scope | Loaded |
|---|---|---|
| Permanent | never released | at boot, resident forever |
| Global | broad | early, resident across most phases |
| Front-end | menu phase | entering the front-end |
| In-game | gameplay phase | entering in-game / a race |

Verified, the `*MemoryFile.bin` manifests carry the `0x53219999` sentinel at offset 8
([C36.4](../C36-Archives-VFS/04-memoryfile.md)), and the Permanent manifest is small (3200 bytes) — a minimal
core — while Global and InGame are larger. So the manifests are a tiered residency plan: a small permanent core,
a global set, and phase-specific sets.

> ✅ *Verified:* the `MemoryFile` manifests (`Global/Permanent/InGameMemoryFile.bin`) carry the `0x53219999`
> sentinel at offset 8 ([C36.4](../C36-Archives-VFS/04-memoryfile.md)); Permanent is 3200 bytes (a small core).
> 🟡 *Reasoned:* that these are the preload manifests driving phase residency is the standard preload-manifest
> model, consistent with the verified sentinel-marked manifests and the acquire/release/phase system; the exact
> manifest record format is per-file RE.

## Manifest → acquire → ready

Making a phase ready is loading its manifest ([C38.4](04-gameflow.md)):

```
enter phase P:
   manifest = P.preload_manifest              // Global / InGame / front-end
   for resource in manifest:
       Stream_AcquireResources(resource)       // acquire (load if not resident) — C38.3
   Stream_BlockUntilLoaded(essential set)      // wait for the must-haves — C38.6
   // phase P is now ready
```

So the manifest is the phase's **shopping list**: acquire everything on it, wait for the essentials, and the
phase is ready. The `MemoryFile` intercept ([C36.4](../C36-Archives-VFS/04-memoryfile.md)) makes the
memory-resident entries RAM-served, so the hottest of the manifest's files are instantly available. The manifest
is thus both a *load list* (what to acquire) and a *residency plan* (what stays in RAM).

## Why manifests

Naming a phase's resources in a manifest (rather than discovering them at runtime) buys predictability:

- **Known load set.** The engine knows exactly what a phase needs, so it can load it in one batch at the
  transition ([C38.4](04-gameflow.md)) — no surprise mid-phase loads.
- **Preloading.** The essentials load *before* the phase runs (behind a loading screen,
  [C38.6](06-blocking-budgets.md)), so the phase starts with its data ready — no pop-in of core assets.
- **Bounded memory.** A phase's manifest bounds its resident set, so memory is predictable per phase
  ([C38.4](04-gameflow.md)).

So the manifests are the engine's declaration of "these resources make this phase ready," enabling batch
preloading and bounded, predictable residency.

## RE implications

- **Preload manifests name each phase's resources** — the Global/Permanent/InGame/front-end
  (`MemoryFile`) lists.
- **Four scopes** — permanent (core, resident forever), global (broad), front-end, in-game
  ([C38.4](04-gameflow.md)).
- **Entering a phase acquires its manifest** ([C38.3](03-refcounting.md)) and blocks on the essentials
  ([C38.6](06-blocking-budgets.md)).
- **Manifests give predictable, batch-preloaded, bounded residency** — the phase's known load set.

---

### Key takeaways

- Each phase's resources are named by **preload manifests** — the `MemoryFile` lists
  (Global/Permanent/InGame/front-end).
- Four residency **scopes**: Permanent (small core, 3200 B, forever), Global (broad), Front-end, In-game.
- Making a phase ready is **acquiring its manifest** and blocking on the essentials.
- Manifests are both a **load list** and a **residency plan** — the memory-resident entries are RAM-served
  (C36.4).
- They give **predictable, batch-preloaded, bounded** residency — the phase's known load set.

**Continue:** [C38.6 — Blocking loads & budgets](06-blocking-budgets.md) · [Chapter 38 hub](C38-Resource-Streaming-Residency.md)
