# C41.3 — The Hash Unification

> **The one-sentence version:** the physics class names double as **vault keys** — the same reflection hash that
> keys vault fields hashes the class names, and those hashes appear in `attributes.bin` (e.g.
> `rh("EngineRacer")=0xB2809518` ×4), so one hash namespace spans runtime class names and vault data.

[← C41.2 — Physics_Base](02-physics-base.md) · [Chapter 41 hub](C41-Physics-RigidBody.md) ·
[Next: C41.4 — Physics::Simulate byte by byte →](04-simulate-thiscall.md)

---

## Two worlds, one hash

Most Wanted has two "hash worlds" ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)): the **asset
hash** (packer-minted, keys texture/geometry names, [Chapter 6](../C6-Texture-Codecs/C6-Texture-Codecs.md)) and
the **reflection hash** (Jenkins-derived, seed `0xABCDEF00`, keys vault fields). The remarkable thing the physics
classes reveal is that the reflection hash is **unified across code and data**: it keys not only vault field
names but the **runtime class names** too.

The evidence is direct. Compute the reflection hash of a physics class name, and its value appears as a **key in
`attributes.bin`**:

| Class name | `rh(name)` | Occurrences in `attributes.bin` |
|---|---|---|
| `RigidBody` | `0xB7235989` | 1 |
| `RBVehicle` | `0x3109B00F` | 1 |
| `RBCop` | `0xBE725CD4` | 1 |
| `RBTrailer` | `0x87FF41E0` | 1 |
| `EngineRacer` | `0xB2809518` | **4** |
| `SuspensionTraffic` | `0x12D5313C` | 2 |
| `SuspensionTrailer` | `0xD44C9372` | 1 |
| `DamageRacer` | `0x6AE5E09C` | **3** |
| `DamageCopCar` | `0x1DF44901` | 1 |
| `EffectsVehicle` | `0xCED20BA4` | 1 |

The class names are in the **executable** ([C41.1](01-rigidbody-tree.md)); their hashes are keys in the **vault**.
Same hash function, same namespace, spanning both.

> ✅ *Verified:* the reflection hashes of the physics class/spec names appear as keys in `GLOBAL/attributes.bin`
> at the counts above (`rh` = Jenkins lookup2, seed `0xABCDEF00`). `EngineRacer` ×4 and `DamageRacer` ×3 are the
> most-referenced. The same hashes do **not** appear as literal dwords in `speed.exe` — they're computed at
> runtime from the names.

## Computed, not stored

A crucial subtlety: the class-name hashes are **not stored as constants in the executable** — searching
`speed.exe` for the little-endian dword `0xB7235989` (RigidBody) finds nothing. The runtime **computes** the hash
from the class-name string when it registers or looks up a class
([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)). So the flow is:

```
class name string (in .rdata)  ──rh()──►  hash  ──►  vault lookup key (in attributes.bin)
```

The name lives in the code; the hash is minted on demand; the vault is keyed by that hash. This is why the vault
can hold per-class data without the executable ever storing the hash: both sides derive the key from the name,
independently, via the shared hash function.

## Class name = spec key

What this buys is that a car's **per-class specs are looked up by hashing the spec class name**. The mechanics
([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) have named spec classes — `EngineRacer` (an engine
spec), `SuspensionTraffic`/`SuspensionTrailer` (suspension specs), `DamageRacer`/`DamageCopCar` (damage specs),
`EffectsVehicle` (effects spec) — and each car references the specs it uses. To load the `EngineRacer` engine
parameters, the runtime hashes `"EngineRacer"` → `0xB2809518` and looks that key up in the vault. That
`EngineRacer` appears **4 times** and `DamageRacer` **3 times** reflects multiple references — several cars or
sub-configs pointing at the same spec.

So the unification is not a curiosity — it's the *mechanism* by which the class system
([Chapter 32](../C32-Runtime-Class-System/C32-Runtime-Class-System.md)) and the data vault
([Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) connect: a class's data is found by hashing the
class's name. Code and data meet at the hash.

## Why unify code and data at the hash

Using one hash namespace for class names and vault keys is a strong design choice:

- **No separate ID tables.** There's no need for a hand-maintained map from class to data ID — the class *name*
  is the key. Add a class, name a spec after it, and the lookup works.
- **Human-readable authoring.** Designers author vault specs under readable names (`EngineRacer`,
  `SuspensionTrailer`), and the engine hashes them at load — the authoring format
  ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) uses names, the runtime uses hashes.
- **Uniform reflection.** The same reflection machinery ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md))
  that resolves vault fields resolves class specs — one system, not two.

This is the reflective heart of the engine: names hashed into a single namespace, spanning the class system and
the data vault, so that "what are this class's parameters?" is answered by hashing its name and reading the vault.
It's an elegant unification, and it's *verified* — the hashes are right there as keys in `attributes.bin`.

## RE implications

- **The reflection hash spans class names and vault keys** — one namespace, verified by the class-name hashes
  appearing as keys in `attributes.bin`.
- **The hashes are computed, not stored** — the exe holds the *names*; the runtime mints the hash; the vault holds
  it as a *key*.
- **Class name = spec key** — a car's engine/suspension/damage specs are looked up by hashing the spec class name
  (`EngineRacer` → `0xB2809518`, ×4).
- **Code and data meet at the hash** — the mechanism connecting the class system and the vault.

---

### Key takeaways

- The physics class names double as **vault keys** via the reflection hash — `rh("EngineRacer")=0xB2809518`
  appears ×4 in `attributes.bin`, `rh("RigidBody")=0xB7235989` ×1, etc. (**verified**).
- The hashes are **computed at runtime from the names**, not stored as constants in `speed.exe`.
- A class's **per-class specs are looked up by hashing its name** — the spec classes (`EngineRacer`,
  `SuspensionTraffic`, `DamageRacer`…) are the mechanics' data.
- This **one-hash unification** of class names and vault data is the reflective heart of the engine — code and
  data meet at the hash.
- It needs **no separate ID tables** and keeps authoring **human-readable** — name a spec after its class, and
  the lookup works.

**Continue:** [C41.4 — Physics::Simulate, byte by byte](04-simulate-thiscall.md) · [Chapter 41 hub](C41-Physics-RigidBody.md)
