# C31.4 — Integrity: Checksums & Validation

> **The one-sentence version:** a save carries integrity data — a checksum (and possibly a signature) — so the
> game can detect a corrupted or tampered save; edit the payload and you must recompute it, or the game rejects
> the save.

[← C31.3 — The save payload](03-save-payload.md) · [Chapter 31 hub](C31-Save-MemoryCard.md) ·
[Next: C31.5 — Platforms & memory cards →](05-platforms.md)

---

## Saves are protected

A save is valuable and vulnerable: it can be corrupted (a bad write, a failing memory card) or tampered with
(edited to cheat). So the game protects it with **integrity data** — most commonly a **checksum** computed over
the payload, stored in the container, and re-verified on load. If the recomputed checksum doesn't match the
stored one, the save is treated as corrupt or invalid and rejected (or repaired from a backup).

This is the save's equivalent of the size-tree invariant ([C1.10](../C1-EAGL-Container-Model/10-editing-and-repacking.md)):
a value that must stay consistent with the data it describes, checked at load.

## The checksum

The checksum is a value (CRC, hash, or a game-specific sum) over the save's bytes:

```
checksum = compute_checksum(save_payload)      # over the payload (and maybe the header)
# stored in the container; on load, recomputed and compared
if recomputed != stored: reject_save()
```

Its purpose is **detection**, not secrecy — it detects change, whether accidental (corruption) or deliberate
(editing). For an editor, the checksum is the gate: change any protected byte and the stored checksum no longer
matches, so you must **recompute and rewrite** it for the game to accept the save.

## Possibly signed

Beyond a plain checksum, a save may carry a **signature** or a keyed hash — a checksum that depends on a secret,
so it can't be recomputed without the key. This raises the bar from "detect corruption" to "detect tampering by
someone without the key." Whether MW's save uses a plain checksum or a keyed one determines how editable it is:

- **Plain checksum** — recomputable by anyone; edited saves can be re-validated by recomputing.
- **Keyed signature** — not recomputable without the key; editing is much harder (you'd need the key or a
  signature bypass).

> 🟡 *Reasoned:* saves carry integrity data (checksum, possibly a signature) is the standard save-protection
> design; the ✅ verified facts are the `LOCH`/`LOCI` container ([C31.1](01-loch.md)–[C31.2](02-loci.md)) that
> would hold it. The exact integrity algorithm is the save format's detail — determine it before assuming an
> edit will validate.

## The editing gate

Integrity is why save editing is not just "change the bytes":

1. **Read** the save ([C31.1](01-loch.md)–[C31.3](03-save-payload.md)).
2. **Edit** the payload ([C31.6](06-editing-saves.md)).
3. **Recompute** the integrity data over the edited payload.
4. **Rewrite** it into the container.
5. **Verify** the game accepts the save.

Skip step 3 and the game rejects your edit at load — the checksum catches it. This is the single most important
save-editing rule: **an edit isn't done until the integrity data is recomputed.**

## Corruption vs tampering

The same mechanism serves two purposes, and it's worth distinguishing them:

- **Corruption detection** — a damaged save (bad sectors, interrupted write) fails the checksum, so the game
  doesn't load garbage as progress.
- **Tamper detection** — an edited save fails the checksum unless recomputed, discouraging casual cheating.

For a legitimate editor (backing up, transferring, fixing a stuck save), recomputing the checksum is the
required final step; for the game, the check protects the player's progress from silent corruption.

## Editing implications

- **Always recompute integrity** after editing ([C31.6](06-editing-saves.md)) — the non-negotiable last step.
- **Determine the algorithm first** — a plain checksum is recomputable; a keyed signature may not be.
- **Preserve the container structure** — the checksum is computed over specific bytes; keep the `LOCH`/`LOCI`
  framing so it covers the right region.
- **Test the load** — the decisive check is that the game accepts the edited save.

---

### Key takeaways

- Saves carry **integrity data** (a checksum, possibly a signature) to detect corruption and tampering, checked
  on load.
- The checksum detects change; edit any protected byte and it no longer matches — you must recompute it.
- A save may be **plain-checksummed** (recomputable) or **signed** (needs a key) — this determines editability.
- The editing gate: read → edit → **recompute integrity** → rewrite → verify the game accepts it.
- Determine the algorithm, preserve the container, always recompute, and test the load.

**Continue:** [C31.5 — Platforms & memory cards](05-platforms.md) · [Chapter 31 hub](C31-Save-MemoryCard.md)
