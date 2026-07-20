# C50.3 — Hash Verification

> **The one-sentence version:** the reflection hash (Jenkins lookup2, seed `0xABCDEF00`) enables a decisive test —
> a name is a real engine key if and only if its hash appears in `attributes.bin`, so `rh("EngineRacer")=0xB2809518`
> appearing ×4 *proves* `EngineRacer` is a real spec, and a made-up name would hash to nothing.

[← C50.2 — Byte verification](02-byte-verification.md) · [Chapter 50 hub](C50-Verification-Methodology.md) ·
[Next: C50.4 — VTable verification →](04-vtable-verification.md)

---

## The reflection hash

Most Wanted keys its vault data ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) and its runtime
class names ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)) with a **reflection hash** — Jenkins
`lookup2`, seeded `0xABCDEF00`. The algorithm processes the name in 12-byte blocks with a mix function, then a tail
step, then a final mix ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)). Given a name, it
produces a 32-bit key:

```
rh("EngineRacer")     = 0xB2809518
rh("CollisionReactionRecord") = 0x63E3B021
rh("BustSpeed")       = 0x769E8D9E
```

Because the algorithm and seed are known and verified, the hash is *computable* for any candidate name — which is
the basis of the decisive test below.

> ✅ *Verified:* the reflection hash is Jenkins lookup2, seed `0xABCDEF00`, confirmed by dozens of names hashing to
> values present as keys in `attributes.bin` (`EngineRacer` ×4, `CopCountRecord` ×22, surface tags, collision
> tags, …) — a match rate that a wrong algorithm/seed could not produce.

## Hash-in-vault: the decisive test

The key technique is **hash-in-vault**: a name is a real engine key *if and only if* its reflection hash appears as
a little-endian dword in `attributes.bin`.

```python
def is_real_key(name):
    return struct.pack('<I', rh(name)) in attributes_bin
```

This works because the hash is **effectively collision-free** for the short identifier names the engine uses. The
32-bit space (4 billion values) is vast relative to the number of keys (thousands), so a *random* string has ~1-in-a-million
chance of colliding with any present key. Therefore:

- **If `rh(name)` is in the vault → `name` is real** (a random name wouldn't hit a present value by chance).
- **If `rh(name)` is absent → `name` is not a vault key** (the engine doesn't use it, at least not in this file).

So "is `EngineRacer` a real spec?" becomes a one-line computation: hash it (`0xB2809518`), search the vault (found
×4) → **yes, verified**. This is how the book confirmed the physics specs, the surface taxonomy, the collision
tags, the cop dispatch records, and the pursuit fields — all by hash-in-vault.

## Recovering hashed-away enums

The most powerful use of hash-in-vault is **recovering enumerations the engine hashed away**
([C44.6](../C44-Surfaces-Grip/06-reading-surfaces.md)). The surface tags, collision types, and mechanic behaviours
are stored in the vault as *hashes*, not strings — you can't grep for them. But you can *guess candidate names and
test them*:

```python
for tag in ["concrete","grass","asphalt","sand","ice","lava","cheese"]:
    n = attributes_bin.count(struct.pack('<I', rh(tag)))
    print(tag, n)   # concrete 23, grass 7, asphalt 4, sand 4, ice 2, lava 0, cheese 0
```

The names that hit (`concrete`, `grass`, …) are the real surfaces; the ones that miss (`lava`, `cheese`) are not.
So the vault *itself* tells you its vocabulary — you propose plausible names and the hashes confirm or deny each.
This is how the book recovered the surface set ([Chapter 44](../C44-Surfaces-Grip/C44-Surfaces-Grip.md)), the
`carhit*` collision tags ([C43.3](../C43-Collision-Contacts/03-classification.md)), and the AI goal/action tuning
keys — enumerations that are invisible to a string search but legible to a hash test.

## The counts are data too

Hash-in-vault returns not just presence but a **count** — how many times a key appears — and the count is
informative ([C44.6](../C44-Surfaces-Grip/06-reading-surfaces.md)):

- **High counts mark central records** — `CollisionReactionRecord` ×35, `TireEffectRecord` ×50, `CopCountRecord`
  ×22 are heavily-referenced, i.e. important, systems.
- **The distribution maps the design** — `concrete` ×23 vs. rare surfaces ×1 shows where the world's material
  effort went ([C44.1](../C44-Surfaces-Grip/01-surface-taxonomy.md)).
- **Absence is a finding too** — a plausible name that hashes to *nothing* (e.g. base `*Params` classes not in
  GLOBAL) tells you it's a code-only class, not a vault record ([C42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)).

So the hash test yields a *quantitative* picture — not just "is X real?" but "how central is X?" — turning the
vault into a measurable map of the game's data. This is verification that also *informs*: the same check that
proves a key exists reveals how much the engine leans on it.

## RE implications

- **The reflection hash** (Jenkins lookup2, seed `0xABCDEF00`) is computable for any candidate name.
- **Hash-in-vault** is decisive — a name is real iff its hash is in `attributes.bin` (the hash is collision-free
  for these names).
- **It recovers hashed-away enums** — propose candidate names, keep the ones that hit (surfaces, collision tags).
- **Counts inform** — high counts mark central records; the distribution maps the design; absence is a finding.

---

### Key takeaways

- The **reflection hash** (Jenkins lookup2, seed `0xABCDEF00`) is verified and **computable** for any name.
- **Hash-in-vault** is a **decisive test** — a name is a real engine key *iff* its hash appears in `attributes.bin`
  (collision-free for short identifiers).
- It **recovers enumerations the engine hashed away** — propose candidate names, keep the ones whose hashes hit
  (how the surface taxonomy and collision tags were found).
- The **counts** are data — high counts (`TireEffectRecord` ×50, `CopCountRecord` ×22) mark central systems; the
  distribution maps the design; **absence** is itself a finding.
- The same one-line check both **proves** a key exists and **reveals** how central it is.

**Continue:** [C50.4 — VTable verification](04-vtable-verification.md) · [Chapter 50 hub](C50-Verification-Methodology.md)
