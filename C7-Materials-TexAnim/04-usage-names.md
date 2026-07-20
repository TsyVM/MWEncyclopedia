# C7.4 — Material Usage-Name Strings

> **The one-sentence version:** geometry carries human-readable material slot names — `DEFAULT`,
> `BODY_ALUMINUM`, `BODY_DULL_PLASTIC`, `BODY_MOLDING`, `WINDOW_FRONT`, `WINDOW_LEFT_REAR` — that tell the
> shader system what *kind* of surface each group is, and they are your fastest way to understand an
> unfamiliar solid.

[← C7.3 — Binding a texture by asset key](03-texture-binding.md) · [Chapter 7 hub](C7-Materials-TexAnim.md) ·
[Next: C7.5 — The two hash worlds →](05-two-hash-worlds.md)

---

## The names are in the file

Unlike the numeric keys, these strings are stored as plain ASCII in the geometry and fall out of any byte
scan. Parsing `CARS/COBALTSS/GEOMETRY.BIN` yields, among others:

```
DEFAULT
BODY_ALUMINUM
BODY_DULL_PLASTIC
BODY_MOLDING
WINDOW_FRONT
WINDOW_LEFT_REAR
WINDOW_RIGHT_REAR
```

They read like what they are: **semantic surface types**. `BODY_ALUMINUM` is painted metal that takes the
car's chosen color and a metallic environment reflection; `BODY_DULL_PLASTIC` is unpainted trim; `WINDOW_*`
are glass groups that take transparency and a glass reflection; `DEFAULT` is the fallback for a group with no
more specific classification.

## What they drive

The usage name is the bridge between a group and the **shader/paint system**. Most Wanted's cars are
re-colored at runtime (you pick a paint), and reflections are applied per surface-type. The engine cannot
guess that a triangle is "paintable body" versus "fixed plastic" from geometry alone — the usage name tells
it:

- **`BODY_*`** surfaces receive the player's paint color and the appropriate specular/metallic response.
  `BODY_ALUMINUM` reacts to light differently than `BODY_DULL_PLASTIC`, though both are "body."
- **`WINDOW_*`** surfaces are treated as glass: semi-transparent, environment-reflective, not paintable.
- **`DEFAULT`** takes the baseline material path.

So the *same* base texture can render as glossy paint or matte plastic depending on which usage slot its
group is in. This is why editing a car's look is often about the *material classification*, not the pixels.

## Why they are indispensable for reverse-engineering

When you open an unfamiliar solid, the numeric keys and packed references are opaque, but the usage names
are self-documenting. They let you:

- **Identify parts without rendering.** A group tagged `WINDOW_FRONT` is the windshield; `BODY_MOLDING` is
  trim. Combined with the per-group bounding box ([C7.2](02-shading-groups.md)) you can map a car's structure
  in seconds.
- **Target edits precisely.** Want to make only the windows tint darker? Find the `WINDOW_*` groups. Want to
  change body finish? The `BODY_*` groups.
- **Understand the naming convention.** The prefix is the family (`BODY`, `WINDOW`), the suffix the
  specialization (`_ALUMINUM`, `_FRONT`, `_LEFT_REAR`). This convention recurs across cars, so learning one
  car's vocabulary transfers to the rest.

## Names versus keys

Keep the two identifier systems straight. The **usage name** is a readable classification stored for the art
pipeline and shader system; the **asset key** ([C7.3](03-texture-binding.md)) is the numeric handle that
resolves to actual pixels. A group has both: a usage name saying *what kind of surface* it is, and a texture
reference saying *which image* to sample. They answer different questions and you use both — the name to
understand, the key to render.

> ✅ *Verified:* the usage strings are present as ASCII in `COBALTSS/GEOMETRY.BIN` (`DEFAULT`, `BODY_ALUMINUM`,
> `BODY_DULL_PLASTIC`, `BODY_MOLDING`, `WINDOW_FRONT`, `WINDOW_LEFT_REAR`, `WINDOW_RIGHT_REAR`).
> 🟡 *Reasoned:* the precise shader/paint behaviour each name selects is inferred from the naming and from how
> MW paints cars; the strings themselves and their role as material slot names are verified.

---

### Key takeaways

- Solids carry readable material **usage names** (`BODY_ALUMINUM`, `WINDOW_FRONT`, `DEFAULT`, …) as ASCII.
- The name classifies a group's **surface type**, driving paint application and reflections — not the pixels.
- Prefix = family (`BODY`, `WINDOW`), suffix = specialization; the convention recurs across cars.
- Use names + per-group bounding boxes to map an unfamiliar solid and to target edits precisely.
- A group has both a usage *name* (what kind of surface) and a texture *key* (which image) — use both.

**Continue:** [C7.5 — The two hash worlds](05-two-hash-worlds.md) · [Chapter 7 hub](C7-Materials-TexAnim.md)
