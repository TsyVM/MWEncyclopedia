# C31.6 — Editing Saves

> **The one-sentence version:** to edit a save, peel any platform wrapper, parse `LOCH`→`LOCI`→payload, change
> the progress you want, keep references valid and sizes truthful, and — the non-negotiable step — recompute
> the integrity data before writing it back.

[← C31.5 — Platforms & memory cards](05-platforms.md) · [Chapter 31 hub](C31-Save-MemoryCard.md) ·
[Next: Chapter 32 — The Runtime Class System & Object Model →](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)

---

## The edit pipeline

Editing a save is a container round-trip with an integrity step that must not be skipped:

```python
def edit_save(save_bytes, changes):
    body = peel_platform_wrapper(save_bytes)          # console: strip card framing (C31.5)
    loch = read_loch(body)                            # C31.1
    items = walk_loci(body, loch)                     # C31.2
    payload = parse_payload(items)                    # C31.3
    apply(payload, changes)                           # edit progress (money, cars, Blacklist…)
    rebuild = write_items(payload)                    # re-serialize items
    fix_sizes(loch, rebuild)                          # LOCH/LOCI sizes/count (C31.1–C31.2)
    stamp_integrity(rebuild)                          # ★ recompute checksum/signature (C31.4)
    return rewrap_platform(rebuild)                   # console: re-add card framing (C31.5)
```

Every step matters, but **`stamp_integrity` is the one people forget** — skip it and the game rejects the save
at load ([C31.4](04-integrity.md)).

## What to change

Common save edits ([C31.3](03-save-payload.md)):

- **Money / bounty** — the economy values.
- **Cars owned** — add a car reference (to a real car definition,
  [Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and its tuning.
- **Blacklist position** — the career progress index.
- **Unlocks** — flags/ids for earned content.
- **Settings** — options and controls.

Each is a payload field or reference; changing it changes the resumed game state.

## Keep it consistent and valid

Save edits fail in two ways beyond integrity — invalid references and incoherent state:

- **References must resolve.** An owned-car reference must name a real car
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); an unlock id must exist; a Blacklist index must
  be in range ([C31.3](03-save-payload.md)). A dangling reference can crash or corrupt the load.
- **State must be coherent.** The pieces should agree — a car marked owned should have valid tuning; progress
  flags shouldn't contradict (e.g. Blacklist #5 done but #10 not, if ordering matters). Incoherent state can
  confuse the game even if it loads.

So a good edit is *valid* (references resolve), *coherent* (state agrees), *sized* (container sizes truthful),
and *sealed* (integrity recomputed).

## Sizes and the container

If an edit changes the payload's length (adding a car, a longer field), the container sizes must follow
([C31.1](01-loch.md)–[C31.2](02-loci.md)):

- **`LOCI` item size** — the edited item's size field.
- **`LOCH` size / count** — the container's total size and item count.
- **Platform wrapper** — on console, the card block count may change ([C31.5](05-platforms.md)).

Same-size edits (changing a value in place) avoid the size cascade — prefer them when possible, the save version
of an in-place edit.

## Integrity is the gate (again)

It bears repeating because it's the difference between a working edit and a rejected one
([C31.4](04-integrity.md)):

- **Determine the integrity algorithm** — plain checksum (recomputable) or keyed signature (may not be).
- **Recompute over the right bytes** — the checksum covers a specific region; compute it over the same one.
- **A save isn't edited until it's re-sealed** — the last step, always.

If the algorithm is a keyed signature you can't reproduce, the save is effectively read-only to edits — know
this before you start.

## Verify

After editing a save:

1. **Container parses** — `LOCH`/`LOCI` read back with truthful sizes/count ([C31.1](01-loch.md)–[C31.2](02-loci.md)).
2. **References resolve** — cars, unlocks, indices are valid ([C31.3](03-save-payload.md)).
3. **Integrity checks out** — the recomputed checksum matches what you wrote ([C31.4](04-integrity.md)).
4. **The game loads it** — the decisive test: load the save and confirm the intended progress, with no
   corruption warning.

The in-game load is the real verification — a save is only correct when the game accepts it and the progress is
what you intended.

---

### Key takeaways

- Edit pipeline: peel wrapper → parse `LOCH`/`LOCI`/payload → change progress → fix sizes → **recompute
  integrity** → rewrap.
- Change money, cars, Blacklist, unlocks, settings — payload fields and references.
- Keep edits **valid** (references resolve) and **coherent** (state agrees), or the load crashes or misbehaves.
- Same-size edits avoid the container size cascade; otherwise fix `LOCI`/`LOCH` sizes and any card blocks.
- **Recompute integrity** (determine the algorithm first) — a save isn't edited until re-sealed; verify by
  loading it in game.

**Continue:** [Chapter 32 — The Runtime Class System & Object Model](../C32-Runtime-Class-System/C32-Runtime-Class-System.md) ·
[Chapter 31 hub](C31-Save-MemoryCard.md)
