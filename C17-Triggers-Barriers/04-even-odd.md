# C17.4 — The Even–Odd Containment Test

> **The one-sentence version:** the engine decides "is the car inside this trigger" by casting a ray and
> counting polygon-edge crossings — odd = inside, even = outside — which is verified exact (4,230/4,230) and
> means triggers can be *any simple polygon*, not just convex, after a cheap center/radius pre-reject.

[← C17.3 — The 15 trigger types](03-trigger-types.md) · [Chapter 17 hub](C17-Triggers-Barriers.md) ·
[Next: C17.5 — Barriers →](05-barriers.md)

---

## The predicate

Containment uses the classic **even–odd (ray-crossing) rule**: from the query point, cast a ray in any fixed
direction and count how many polygon edges it crosses. An **odd** count means the point is **inside**; an
**even** count means **outside**. This is the engine's exact predicate, verified 4,230/4,230 against a
reference implementation — so a faithful tool computes containment the same way the game does.

```python
def point_in_polygon(px, py, poly):        # even–odd ray crossing
    inside = False
    n = len(poly)
    j = n - 1
    for i in range(n):
        xi, yi = poly[i]; xj, yj = poly[j]
        if (yi > py) != (yj > py):
            x_cross = xi + (py - yi) * (xj - xi) / (yj - yi)
            if px < x_cross:
                inside = not inside        # toggle on each crossing
        j = i
    return inside
```

## The two-stage test

The engine doesn't run the polygon test on every trigger for every car — that would be wasteful. It runs a
**coarse-to-fine** pair:

1. **Coarse gate (fast reject).** Compare the car's distance to the trigger's center against its radius
   (`+0x04`/`+0x14`, [C17.2](02-trigger-record.md)). If the car is beyond the radius, skip — no polygon test.
2. **Fine test (exact).** For triggers that pass the gate, run even–odd parity against the polygon vertices.

This is the same coarse-to-fine philosophy as streaming's PVS-then-frustum ([C15.5](../C15-Track-Streaming/05-visibility.md))
and the cull tree's descend-then-test ([C16.5](../C16-Scenery-Cull/05-cull-tree.md)): a cheap filter discards
most candidates, and the expensive exact test runs only on the few that survive.

## Why even–odd is the right choice

The even–odd rule buys a property that matters enormously for authoring: **any simple polygon works, convex or
not.** A convex-only test (like separating axes) would force designers to decompose an L-shaped plaza or a
curved boundary into convex pieces; even–odd handles the concave shape directly.

- **L-shaped and U-shaped regions** — a single trigger, one polygon.
- **Curved boundaries** — approximated by a many-vertex simple polygon, still one trigger.
- **The only rule is "simple."** The polygon must not self-intersect (edges crossing themselves), because
  even–odd parity is defined for simple polygons. A figure-eight is undefined; an L is fine.

So a trigger is as flexible as the road it guards — designers draw the exact footprint, concavities and all.

> ✅ *Verified:* even–odd ray parity is the engine's exact containment predicate (4,230/4,230 against a
> reference); triggers are therefore arbitrary *simple* polygons.

## Numerical care

Two edge cases deserve attention when you reimplement the test:

- **Points exactly on an edge or vertex** — even–odd needs a consistent tie-breaking convention (the
  `(yi > py) != (yj > py)` half-open comparison above handles vertices consistently). Match the engine's
  convention if you need bit-identical results at boundaries.
- **Horizontal edges** — the half-open Y comparison skips edges parallel to the ray, avoiding double-counting.

For gameplay these boundary cases rarely bite (a car is a moving area, not a mathematical point), but a tool
that classifies static points must handle them to agree with the engine.

## Editing implications

- **Keep polygons simple.** When you add or move vertices ([C17.6](06-events-editing.md)), don't create
  self-intersections — the containment test is undefined for non-simple polygons.
- **Concave is fine.** Draw the exact region; you don't need to split concavities.
- **Keep the coarse gate covering the polygon.** The center/radius must loosely contain the whole polygon, or
  the fast reject skips a trigger the car is actually inside.
- **Winding doesn't matter** for even–odd (unlike the nonzero rule), so vertex order can be either way as long
  as the polygon is simple.

---

### Key takeaways

- Containment is **even–odd ray parity** (odd crossings = inside), verified as the engine's exact predicate.
- It runs coarse-to-fine: a center/radius fast reject, then the polygon parity test on survivors.
- Even–odd allows **any simple polygon** — concave L/U shapes and curves are single triggers.
- The polygon must be *simple* (non-self-intersecting); winding order is irrelevant to the test.
- Edit polygons to stay simple, keep concavities freely, and keep the coarse gate covering the polygon.

**Continue:** [C17.5 — Barriers](05-barriers.md) · [Chapter 17 hub](C17-Triggers-Barriers.md)
