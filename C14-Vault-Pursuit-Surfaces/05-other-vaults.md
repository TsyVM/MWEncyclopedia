# C14.5 — The gameplay & FE_ATTRIB Vaults

> **The one-sentence version:** `attributes.bin` isn't the only vault — `GLOBAL/gameplay.bin` and
> `GLOBAL/FE_ATTRIB.bin` are also VPAK files (gameplay rules and front-end/menu attributes), read with the
> exact same tools, so knowing which vault owns which data is half of finding it.

[← C14.4 — Effects & destructibles](04-effects-destructibles.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md) ·
[Next: C14.6 — Editing gameplay safely →](06-editing-gameplay.md)

---

## Three vaults, one format

The vault system is split across files, each a **VPAK** ([C11.1](../C11-Attribute-Vaults/01-vpak-header.md))
read with identical tooling:

| File | Magic | Contents |
|---|---|---|
| `GLOBAL/attributes.bin` | `VPAK` v1, 3 blocks | the master vault: cars, cops, pursuit, surfaces, effects |
| `GLOBAL/gameplay.bin` | `VPAK` | gameplay rules and event data |
| `GLOBAL/FE_ATTRIB.bin` | `VPAK` v1, **9 blocks** | front-end (menu/UI) attributes |

All three begin with the `VPAK` magic and follow the header→string-table→records→trailer structure of
Chapter 11, so the same reader, the same reflection hash ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)),
and the same resolve-then-decode ([C12.5](../C12-Reflection-Schema/05-resolving-values.md)) work on every one.
Verified: `gameplay.bin` (2 105 216 bytes) and `FE_ATTRIB.bin` (113 088 bytes, 9 blocks) both carry the
`VPAK` magic.

## Why split the vault

Splitting attributes across files serves loading and organisation:

- **Load scope.** The front-end vault (`FE_ATTRIB`) is needed in menus; the gameplay and master vaults in the
  world. Separate files let the engine load what a context needs.
- **Ownership clarity.** Menu/UI tuning lives in `FE_ATTRIB`; moment-to-moment gameplay rules in
  `gameplay.bin`; the simulation content (cars, cops, surfaces) in `attributes.bin`.
- **Modularity.** A change to menu layout doesn't touch the physics vault and vice versa.

The practical upshot for a modder: **look in the right vault.** A car stat is in `attributes.bin`; a menu
value is in `FE_ATTRIB`; a gameplay rule may be in `gameplay.bin`. Searching the wrong file wastes time.

## FE_ATTRIB — the front end

`FE_ATTRIB.bin` (9 blocks) holds the **front-end** attributes — the menus, HUD, and UI-facing values (the
`HUD*` and `HUDS_Custom_*` files alongside it are the HUD's textures/layouts). Its 9-block header is a richer
VPAK than the master vault's 3 blocks, reflecting the front end's many small attribute groups. It is read
identically; only the content domain differs.

## gameplay.bin — the rules

`gameplay.bin` carries **gameplay rules and event data** — the larger of the auxiliary vaults at ~2 MB. Where
`attributes.bin` defines the *actors* (cars, cops, surfaces), `gameplay.bin` leans toward the *rules and
events* that orchestrate them. Read it with the same VPAK tooling and resolve its collections by name.

> ✅ *Verified:* `gameplay.bin` and `FE_ATTRIB.bin` are VPAK files (magic confirmed), `FE_ATTRIB` with 9
> blocks; sizes as stated.
> 🟡 *Reasoned:* the precise division of responsibility between the three vaults is described from their names,
> sizes, and the collections observed; the VPAK format identity is verified.

## One toolchain, many vaults

Because all three are VPAK, a single set of tools serves them:

```python
def open_vault(path):
    buf = open(path, "rb").read()
    assert buf[:4] == b"VPAK"
    return parse_vpak(buf)          # header, ErtS, records, trailer — C11

for p in ["GLOBAL/attributes.bin", "GLOBAL/gameplay.bin", "GLOBAL/FE_ATTRIB.bin"]:
    vault = open_vault(p)           # same reader, same reflection hash, same resolution
```

Build the reader once ([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) and it opens every
vault. This is the reward of the reflection system's generality: one format, one toolchain, all the game's
data-driven behaviour.

---

### Key takeaways

- The vault is split across three VPAK files: `attributes.bin` (master), `gameplay.bin` (rules),
  `FE_ATTRIB.bin` (front end, 9 blocks).
- All share the VPAK format, reflection hash, and resolution — one toolchain reads them all.
- Split serves load scope, ownership clarity, and modularity; find data in the vault that owns its domain.
- `FE_ATTRIB` = menus/HUD/UI; `gameplay.bin` = rules/events; `attributes.bin` = simulation actors.
- Verify a file is VPAK by its magic, then reuse the Chapter 11 reader.

**Continue:** [C14.6 — Editing gameplay safely](06-editing-gameplay.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md)
