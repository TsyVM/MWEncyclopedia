# C44.1 — The Surface Taxonomy

> **The one-sentence version:** each wheel is always on a surface identified by a verified tag — `concrete`
> (×23), `grass` (×7), `asphalt`, `sand`, `gravel`, `dirt`, `cobble`, `water`, `snow`, `mud`, `ice` — and the
> frequencies reveal Rockport as a paved city with grassy verges.

[← Chapter 44 hub](C44-Surfaces-Grip.md) · [Next: C44.2 — Grip →](02-grip.md)

---

## The verified tags

Every point of the drivable world has a **surface tag** — a classification of what that ground *is*, physically.
The tags are verified vault keys ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)): the reflection hash
of each appears in `attributes.bin`:

| Surface | `rh(tag)` | Occurrences |
|---|---|---|
| `concrete` | `0x6CA26F9B` | **23** |
| `grass` | `0x772FB736` | 7 |
| `asphalt` | `0x19DB2F1E` | 4 |
| `sand` | `0x999ACD78` | 4 |
| `water` | `0x5A2E0437` | 4 |
| `dirt` | `0xD929E923` | 2 |
| `ice` | `0x444BA7CB` | 2 |
| `gravel` | `0xC1C577D6` | 1 |
| `cobble` | `0x3830BABB` | 1 |
| `snow` | `0xEF405BC5` | 1 |
| `mud` | `0x3010962F` | 1 |

That these exact words hash to values present in `attributes.bin` confirms they're the real surface keys the
engine uses ([C43.6](../C43-Collision-Contacts/06-reading-collision.md)) — a decisive test.

> ✅ *Verified:* the surface tags are reflection-hash keys in `GLOBAL/attributes.bin` at the counts above —
> `concrete` ×23, `grass` ×7, `asphalt`/`sand`/`water` ×4, `dirt`/`ice` ×2, `gravel`/`cobble`/`snow`/`mud` ×1.

## The frequencies tell a story

The reference counts are a quiet portrait of Rockport ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)):

- **`concrete` (×23) and `grass` (×7) dominate.** Rockport is a *city* — concrete roads, sidewalks, and lots;
  grass verges, parks, and medians. These are the two surfaces you're on most, so they have the most tuning
  entries (per-context grip/sound/effect records, [C44.5](05-three-reads.md)).
- **`asphalt`, `sand`, `water` (×4).** Asphalt for the highways and better roads; sand and water for the
  waterfront and outskirts ([Chapter 15](../C15-Track-Streaming/C15-Track-Streaming.md)).
- **`dirt`, `gravel`, `cobble`, `mud` (×1–2).** The occasional back road, alley, or construction site — present
  but rare.
- **`ice`, `snow` (×1–2).** Marginal in Rockport's climate — a handful of entries, perhaps for specific spots or
  legacy from the shared engine ([C41.6](../C41-Physics-RigidBody/06-vehicle-types.md)).

So the taxonomy isn't uniform — it's weighted toward what the city actually is. The surface set is comprehensive
(the engine can represent snow and ice), but Rockport *uses* mostly concrete and grass. Reading the frequencies is
reading the world's material composition.

> 🟡 *Reasoned:* the interpretation of the frequencies (concrete/grass dominating because Rockport is a paved
> city) is inferred from the verified counts and the known setting; the exact per-surface record breakdown is
> further RE. The tags and their counts are verified.

## How a wheel gets its tag

A wheel's surface tag comes from its **ground contact** ([C43.1](../C43-Collision-Contacts/01-detection.md)): the
collision geometry of the world ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)) carries a
surface classification per region, and when a wheel contacts the ground, the contact reports which surface it's on.
So the world is *painted* with surfaces — each piece of drivable geometry tagged with its material — and the wheel
reads that tag at the contact point each frame.

This means the surface can change *under* the car continuously: as you drive off asphalt onto grass, each wheel's
tag flips as it crosses the boundary — and because the reads are per-wheel ([C43.1](../C43-Collision-Contacts/01-detection.md)),
you can have two wheels on asphalt and two on grass (half on the road, half on the verge), each contributing its
own grip and effect. The taxonomy is thus a *spatial* property of the world, sampled per wheel per frame.

## Why a tag, not per-material physics

Classifying surfaces into a small tag set (rather than giving every piece of ground unique physics) is the same
data-driven economy as the rest of the engine ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)):

- **A finite, shared vocabulary.** ~11 surfaces cover the whole world; each is tuned once (grip, sound, effect)
  and reused everywhere that surface appears. Paint a new road `concrete` and it behaves like all concrete.
- **Designer-authorable.** Artists paint surface tags onto the world geometry
  ([Chapter 16](../C16-Scenery-Cull/C16-Scenery-Cull.md)); the physics/audio/effects follow from the
  tag — no per-region tuning.
- **Consistent behaviour.** All `grass` grips, sounds, and smokes the same, so the world is coherent — you learn
  that grass is slippery and it always is.

So the surface tag is the unit of the whole "driving on the world" system: a small vocabulary, painted onto the
world, that the three reads ([C44.5](05-three-reads.md)) turn into feel, sound, and look.

## RE implications

- **The surface tags are verified vault keys** — `concrete`/`grass`/`asphalt`/`sand`/… — confirmed by hash-in-vault.
- **The frequencies reflect Rockport** — concrete (×23) and grass (×7) dominate a paved city with verges.
- **A wheel's tag comes from its ground contact** — the world is painted with surfaces, sampled per wheel per
  frame.
- **A small shared vocabulary** — ~11 surfaces, tuned once each, painted onto the world; designer-authorable and
  consistent.

---

### Key takeaways

- Each wheel is always on a **surface tag** — verified vault keys: `concrete` (×23), `grass` (×7), `asphalt`,
  `sand`, `water` (×4), `dirt`/`ice` (×2), `gravel`/`cobble`/`snow`/`mud` (×1).
- The **frequencies portray Rockport** — a paved city (concrete) with grassy verges (grass), waterfront (sand,
  water), and rare back-roads (dirt, gravel, cobble).
- A wheel gets its tag from its **ground contact** — the world geometry is painted with surfaces, sampled **per
  wheel, per frame** (you can straddle a boundary).
- The tag is a **small shared vocabulary** (~11 surfaces), tuned once each and reused everywhere — designer-painted,
  consistent.
- The tag is the **unit** of the driving-on-world system — the three reads ([C44.5](05-three-reads.md)) turn it
  into feel, sound, and look.

**Continue:** [C44.2 — Grip: the functional read](02-grip.md) · [Chapter 44 hub](C44-Surfaces-Grip.md)
