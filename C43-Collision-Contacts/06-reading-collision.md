# C43.6 — Reading Collision in RE

> **The one-sentence version:** navigate collision by the contact update (`0x6A7110`, reading `[this+0xEC]`), the
> collision code classes (`CollisionInstanceList`/`CollisionObjectList`/`WCollisionPack`), the classification tags
> (`carhit*`), and the reaction records (`CollisionReactionRecord` ×35) — reading the contact layer from code and
> vault together.

[← C43.5 — Smackables](05-smackables.md) · [Chapter 43 hub](C43-Collision-Contacts.md) ·
[Next: Chapter 44 — Surfaces: Grip, Sound & Effects →](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)

---

## Anchors for collision RE

The contact layer is anchored on verified structures:

- **The contact update `0x6A7110`** ([C43.1](01-detection.md)) — `sub esp,0x128; mov esi,ecx; mov eax,[esi+0xEC]`
  — the detection site, reading the part array.
- **The collision classes** — `CollisionInstanceList`, `CollisionObjectList`, `WCollisionPack`
  ([C43.1](01-detection.md)) — the collision world.
- **The classification tags** — `carhitwall`/`carhitcar`/`carhitsmackable`/`carscrapewall`
  ([C43.3](03-classification.md)) — verified vault keys.
- **The reaction records** — `CollisionReactionRecord` (`0x63E3B021`, ×35), `AICollisionReactionRecord`
  (`0xAA229CD7`, ×14) ([C43.4](04-reactions.md)).
- **The smackables** — `RBSmackable`, `SmackableParams` ([C43.5](05-smackables.md)).

From these, the whole contact layer is navigable: detection (code), classification (tags), and response (records).

## The RE workflow

Reading collision:

1. **Find the detection** — the contact update `0x6A7110` within `Physics::Simulate`
   ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)); note its `[this+0xEC]` read ties in the
   wheels ([C43.1](01-detection.md)).
2. **Find the collision world** — the `Collision*List`/`WCollisionPack` classes; the baked geometry + dynamic
   bodies the query runs against ([C43.1](01-detection.md)).
3. **Enumerate the tags** — the `carhit*`/`carscrape*` classifications ([C43.3](03-classification.md)); hash each
   and confirm in `attributes.bin`.
4. **Read the responses** — the reaction records ([C43.4](04-reactions.md)) and the damage/presentation reads
   ([Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)) per tag.

The output is the full contact picture: how contacts are found, classified, and responded to.

## Verifying by hash-in-vault

The strongest verification technique in this chapter is **confirming a tag/record name by finding its hash in the
vault**:

```python
# a name is a real vault key iff its reflection hash appears in attributes.bin
h = rh("carhitwall")            # 0x96BA88FF
assert struct.pack('<I', h) in attributes_bin   # ×4 → verified real key
```

Because the reflection hash is effectively collision-free for these short names, a name whose hash appears in
`attributes.bin` is *certainly* a key the engine uses — a random string wouldn't hash to a present value. This is
how `carhitwall` (×4), `carhitcar` (×6), `CollisionReactionRecord` (×35), and `AICollisionReactionRecord` (×14)
were all confirmed ([C43.3](03-classification.md), [C43.4](04-reactions.md)): compute the hash, find it in the
vault. It's a clean, decisive test — the verification-first discipline
([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)) applied to names.

## The one-contact-three-reads shape

The organising idea to carry from this chapter is the **fan-out** ([C43.3](03-classification.md)): one contact,
classified by one tag, read three independent ways:

```
contact ─classify→ tag ─┬→ reaction     (CollisionReactionRecord / AI)   physics
                        ├→ damage        (Damage* mechanic, Ch 45)        consequence
                        └→ presentation  (effects + carhit* sound)        spectacle
```

This shape recurs everywhere in Most Wanted's design ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md) does
the *continuous* version for surfaces): a single classification fans out to physics, consequence, and presentation
through independent, tunable records. Recognising it is the key to the whole "touching the world" system — find
the classification, and the three reads follow. It's also the reason the system is so moddable: each read is a
separate record, tunable without touching the others.

## Collision connects the sim to the world

With the contact layer decoded, the vehicle sim ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md))
is fully connected to the world: the wheels contact the road (→ surface, [Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)),
the body hits walls/cars/props (→ reaction here, damage [Chapter 45](../C45-Damage-Deformation/C45-Damage-Deformation.md)),
and the AI's cars collide by their own tuning (→ pursuit tactics,
[Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)). Collision is the membrane between the isolated,
deterministic sim ([C39.5](../C39-Vehicle-Simulation/05-connectors.md)) and the shared world — the place the car's
physics meets everything else. The next chapter ([44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)) takes the
*continuous* side of that membrane: driving *on* surfaces.

## RE implications

- **Anchor on** the contact update `0x6A7110`, the `Collision*` classes, the `carhit*` tags, and the reaction
  records.
- **The RE workflow** — detection → collision world → tags → responses.
- **Verify names by hash-in-vault** — a name whose hash is in `attributes.bin` is certainly a real key (decisive
  test).
- **The one-contact-three-reads fan-out** is the organising idea — classify once, respond three independent ways.

---

### Key takeaways

- The contact layer is anchored on the **contact update `0x6A7110`**, the **collision classes**
  (`CollisionInstanceList`/`CollisionObjectList`/`WCollisionPack`), the **`carhit*` tags**, and the **reaction
  records**.
- The RE workflow: **detection → collision world → classification tags → responses** (reaction, damage,
  presentation).
- The decisive verification is **hash-in-vault** — a name whose reflection hash appears in `attributes.bin` is
  certainly a real key (how `carhitwall`, `CollisionReactionRecord` ×35, etc. were confirmed).
- The organising idea is the **one-contact-three-reads fan-out** — classify once, respond three independent,
  tunable ways.
- Collision is the **membrane** between the isolated sim and the shared world — the next chapter takes the
  continuous side (surfaces).

**Next:** [Chapter 44 — Surfaces: Grip, Sound & Effects](../C44-Surfaces-Grip/C44-Surfaces-Grip.md): driving on the
world.

**Sources:** `speed.exe` (verified: contact update `0x6A7110` = `sub esp,0x128; mov esi,ecx; mov eax,[esi+0xEC]`;
collision classes `CollisionInstanceList`/`CollisionObjectList`/`WCollisionPack`; smackable classes `RBSmackable`/
`SmackableParams`/`EffectsSmackable`/`SmackableRenderConn`); `GLOBAL/attributes.bin` (verified: reaction records
`CollisionReactionRecord` ×35 / `AICollisionReactionRecord` ×14, classification tags `carhitcar` ×6 / `carhitwall`
×4 / `carscrapewall` ×4 / `carhitsmackable` ×1, all as reflection-hash keys).
