# C14.6 — Editing Gameplay Safely

> **The one-sentence version:** retuning pursuit, surfaces, or gameplay uses the exact resolve-then-write
> discipline of the reflection chapters — find by name, resolve to see override vs inherited, overwrite the
> typed value in place, choose scope through inheritance, and verify by re-resolving and by playing.

[← C14.5 — The gameplay & FE_ATTRIB vaults](05-other-vaults.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md) ·
[Next: Chapter 15 — The Master Track File →](../C15-Track-Streaming/C15-Track-Streaming.md)

---

## The same discipline, new content

Nothing about editing pursuit or surfaces is special — it is the vault-editing workflow of
[C12.6](../C12-Reflection-Schema/06-writing-values.md) and [C13.6](../C13-Vault-CarTuning/06-retuning.md)
applied to different collections:

1. **Find** the collection by reflection hash (`AIPursuit`, `carsurface`, `heat_meter`, …).
2. **Resolve** the field to see its value and whether it's an **override** or **inherited**
   ([C12.5](../C12-Reflection-Schema/05-resolving-values.md)).
3. **Write** — overwrite an existing override in place (safe, no size change) or add one (repack).
4. **Scope** — edit one collection, its behavior family, or `default`
   ([C12.4](../C12-Reflection-Schema/04-default-inheritance.md)).
5. **Verify** — re-resolve to confirm the value, then playtest.

```python
def edit_gameplay(vault, collection, field, new_value, schema):
    ch, fh = reflection_hash(collection), reflection_hash(field)
    value, ftype, src = resolve(vault, ch, fh, schema)
    if src == ch:                                   # existing override → safe in-place write
        write_value_in_place(vault.buf, offset_of(ch, fh), ftype, new_value)
    else:                                           # inherited → add override (repack)
        add_override(vault, ch, fh, encode_by_type(new_value, ftype))
    assert resolve(vault, ch, fh, schema)[0] == new_value    # verify
```

## Pick the right vault and collection

Gameplay data spans three vaults ([C14.5](05-other-vaults.md)), so step zero is finding the right file:

- **Pursuit / surfaces / effects / cars** → `attributes.bin`.
- **Gameplay rules / events** → `gameplay.bin`.
- **Menu / HUD / UI** → `FE_ATTRIB.bin`.

Then the right collection: pursuit aggression in `AIPursuit`, spawn scale in `AICopManager`, a police car's
stats in its `COP*` collection, a ground type in `carsurface`/`terraindriving`, escalation in
`heat_meter`. Editing the wrong collection is the most common failure — resolve first to confirm you're
looking at the value that actually drives the behaviour.

## Scope is a design decision

Inheritance makes scope explicit, and gameplay edits especially benefit from choosing it well:

- **One actor** — override in that collection (make *this* cop car faster).
- **A whole class** — edit the behavior/family (make *all* sport interceptors faster).
- **A global rule** — edit `default` or the governing collection (change escalation for the whole game).

A "harder game" mod is usually a *combination* across scopes and vaults — aggressive `AIPursuit`, more units in
`AICopManager`, faster `COP*` cars, a steeper `heat_meter` — each a small, verified edit
([C14.1](01-pursuit-ai.md)–[C14.2](02-heat-bounty.md)).

## Keep the world coherent and winnable

Two cautions specific to gameplay:

- **Coherence.** Surfaces bundle grip, sound, and effects ([C14.3](03-surfaces.md)); events bundle physics and
  `fx*` ([C14.4](04-effects-destructibles.md)). Change one facet and the others may no longer match; coherent
  mods adjust the set.
- **Winnability.** Pursuit and heat changes can make the game unfun in both directions — cops that instantly
  catch any car, or that never threaten. The file cannot tell you this; only playtesting can, so the final
  verification is always at the wheel.

## Verify twice

- **In the file** — re-resolve the edited field and confirm your value from the intended collection; for a
  repack, re-check the `VPAK` header and `NtaD` counts ([C11.6](../C11-Attribute-Vaults/06-navigating-editing.md)).
- **In play** — drive the change: a harder pursuit should feel harder, a grippier surface should hold. Numbers
  that verify in the file but feel wrong usually mean the wrong collection, scope, or a value another system
  overrides.

---

### Key takeaways

- Editing gameplay is the standard resolve-then-write vault discipline aimed at new collections.
- Find the right **vault** (`attributes`/`gameplay`/`FE_ATTRIB`) and the right **collection** before editing.
- Overwrite existing overrides in place; add overrides as a repack; choose scope via inheritance.
- Big mods combine edits across scopes and vaults (pursuit + heat + roster).
- Keep the world coherent (grip/sound/fx together) and winnable; verify in the file *and* by playing.

**Continue:** [Chapter 15 — The Master Track File & Streaming Sections](../C15-Track-Streaming/C15-Track-Streaming.md) ·
[Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md)
