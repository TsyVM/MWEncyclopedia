# C44.6 — Reading Surfaces in RE

> **The one-sentence version:** navigate surfaces by the tag set (hash-verified in `attributes.bin`) and the three
> records (grip tunables, `RoadNoiseRecord` ×48, `TireEffectRecord` ×50) — reading the surface system as one tag
> fanning out to feel, sound, and look.

[← C44.5 — The three synchronized reads](05-three-reads.md) · [Chapter 44 hub](C44-Surfaces-Grip.md) ·
[Next: Chapter 45 — Damage & Deformation →](../C45-Damage-Deformation/C45-Damage-Deformation.md)

---

## Anchors for surface RE

The surface system is anchored on verified vault data:

- **The surface tags** — `concrete`/`asphalt`/`grass`/`sand`/… ([C44.1](01-surface-taxonomy.md)), hash-verified
  keys.
- **`RoadNoiseRecord`** (`0xFFDB013B`, ×48) — the audio read ([C44.3](03-road-noise.md)).
- **`TireEffectRecord`** (`0x681D219C`, ×50) — the visual read ([C44.4](04-tire-effects.md)).
- **The grip tunables** — per-surface grip coefficients in the tyre/suspension data
  ([C44.2](02-grip.md)).
- **The tyre modes** — driving/skid/slide/fly/hit ([C42.4](../C42-Suspension-Tyres-Drivetrain/04-tyres-grip.md)),
  the second key dimension.

From these, the surface system is navigable: the tags, the three reads, and the mode dimension.

## The RE workflow

Reading surfaces:

1. **Enumerate the tags** — hash the candidate surface names and confirm in `attributes.bin`
   ([C44.1](01-surface-taxonomy.md)); the ones that appear are the real surfaces.
2. **Find the three records** — `RoadNoiseRecord` and `TireEffectRecord` (by hash), and the grip tunables
   ([C44.2](02-grip.md)); these are the three reads.
3. **Map the grid** — for each surface × mode, the `TireEffectRecord` entry (`fxtd_<mode>_<surface>`) and the
   `RoadNoiseRecord` entry ([C44.4](04-tire-effects.md)).
4. **Confirm coherence** — the same tag should have all three reads filled ([C44.5](05-three-reads.md)).

The output is the full surface picture: the material vocabulary and how each material feels, sounds, and looks.

## Verifying by hash-in-vault

As with collision ([C43.6](../C43-Collision-Contacts/06-reading-collision.md)), the decisive test is **hash-in-vault**:
a surface name is real iff its reflection hash appears in `attributes.bin`. This is how the whole taxonomy was
confirmed ([C44.1](01-surface-taxonomy.md)) — `concrete` (×23), `grass` (×7), `asphalt` (×4), etc. all hash to
values present in the vault, while a made-up surface wouldn't:

```python
for tag in ["concrete","grass","asphalt","sand","ice","lava"]:
    n = attributes_bin.count(struct.pack('<I', rh(tag)))
    print(tag, n)      # concrete 23, grass 7, ..., lava 0  → lava is not a surface
```

So the vault itself tells you the surface vocabulary: enumerate candidates, keep the ones whose hashes appear. This
is the verification-first way to recover an enum you can't see directly — the names are hashed away, but the hashes
betray which names are real ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)).

## The record counts are a design readout

The reference counts ([C44.1](01-surface-taxonomy.md), [C44.3](03-road-noise.md), [C44.4](04-tire-effects.md)) are
themselves informative — they quantify where the design's effort went:

- **`TireEffectRecord` ×50 > `RoadNoiseRecord` ×48 > tags.** The visual record is the most-referenced because it's
  keyed on two dimensions (surface × mode, [C44.4](04-tire-effects.md)) — the biggest grid.
- **`concrete` ×23 dominates the tags.** The city's main surface has the most tuning entries
  ([C44.1](01-surface-taxonomy.md)) — effort tracks usage.
- **Rare surfaces (×1) are present but lightly tuned** — `gravel`, `cobble`, `snow`, `mud` exist but appear once,
  matching their rare use.

So the counts are a map of the design's attention: heavily-used surfaces and the two-dimensional visual record get
the most entries. Reading the frequencies ([C44.1](01-surface-taxonomy.md)) is reading the priorities of the people
who built the world — a quantitative window that only the verified vault data provides.

## Surface closes the "touching the world" pair

With surfaces decoded, the pair of "touching the world" chapters is complete: **collision**
([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md)) is the discrete side (hitting things), and
**surfaces** (this chapter) is the continuous side (driving on things). Both fan one classification out to physics,
sound, and visuals ([C44.5](05-three-reads.md)). Together they're how the car's isolated sim
([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) meets the shared world — the membrane, discrete and
continuous. The next chapter ([45](../C45-Damage-Deformation/C45-Damage-Deformation.md)) follows the *consequence*
of the discrete side: damage.

## RE implications

- **Anchor on** the surface tags, the three records (`RoadNoiseRecord` ×48, `TireEffectRecord` ×50, grip
  tunables), and the tyre modes.
- **The RE workflow** — enumerate tags → find the three reads → map the surface × mode grid → confirm coherence.
- **Verify by hash-in-vault** — the vault's hashes betray the real surface vocabulary.
- **The counts are a design readout** — `TireEffectRecord` ×50 (biggest grid), `concrete` ×23 (main surface);
  effort tracks usage.

---

### Key takeaways

- The surface system is anchored on the **tag set**, **`RoadNoiseRecord`** (×48), **`TireEffectRecord`** (×50), the
  **grip tunables**, and the **tyre modes**.
- The RE workflow: **enumerate tags → find the three reads → map the surface × mode grid → confirm coherence**.
- The decisive verification is **hash-in-vault** — the vault's hashes reveal the real surface vocabulary (a
  made-up surface hashes to nothing).
- The **reference counts are a design readout** — `TireEffectRecord` ×50 (two-dimensional grid), `concrete` ×23
  (the city's main surface); effort tracks usage.
- Surfaces **close the "touching the world" pair** — discrete collision + continuous surfaces, both fanning one
  tag to physics + sound + visuals.

**Next:** [Chapter 45 — Damage & Deformation](../C45-Damage-Deformation/C45-Damage-Deformation.md): the consequence
of the discrete side.

**Sources:** `GLOBAL/attributes.bin` (verified: surface tags as reflection-hash keys — `concrete` ×23, `grass` ×7,
`asphalt`/`sand`/`water` ×4, `dirt`/`ice` ×2, `gravel`/`cobble`/`snow`/`mud` ×1; `RoadNoiseRecord` `0xFFDB013B`
×48; `TireEffectRecord` `0x681D219C` ×50).
