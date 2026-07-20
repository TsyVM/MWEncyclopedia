# C12.2 — Field → Type → Offset

> **The one-sentence version:** a value needs three facts the schema supplies — which field (reflection
> hash), what type (one of 16 `EA::Reflection::*` primitives), and where (byte offset) — and missing any one
> makes the bytes unreadable.

[← C12.1 — The reflection hash, recovered](01-reflection-hash.md) · [Chapter 12 hub](C12-Reflection-Schema.md) ·
[Next: C12.3 — The inline value triple →](03-value-triple.md)

---

## Three facts, one value

Reading an attribute is not "read four bytes." It is resolving three questions the schema answers together:

| Question | Answered by | Example |
|---|---|---|
| **Which** attribute? | reflection hash of the field name ([C12.1](01-reflection-hash.md)) | `0xEBCEE74C` |
| **What** type? | one of 16 `EA::Reflection::*` ([C11.3](../C11-Attribute-Vaults/03-type-names.md)) | `Float` |
| **Where** in the record? | byte offset of the value | `+0x18` |

With all three you decode the value; with two, you have bytes but not meaning.

## Why the type is non-negotiable

The same 32 bits are different values under different types — this is the single most important reason a vault
reader must be schema-driven:

```
bytes 0x00 0x00 0xA0 0x40   (little-endian 0x40A00000)
  as Float     → 5.0
  as UInt32    → 1 084 227 584
  as Int32     → 1 084 227 584
  as Reference → a hash pointing at another collection
```

A tool that assumes "everything is a float" will misread every integer, flag, and reference; one that assumes
"everything is a u32" will turn `5.0` into a billion. The type is what disambiguates, and the vault records it
so the engine (and you) read correctly ([C12.3](03-value-triple.md)).

## The compiled schema

The schema is the compiled binding of `field id → (type, offset)` for each collection's layout — effectively
a description of the record struct. It is "compiled" in the sense that the build pipeline fixes each field's
type and place ahead of time, and the runtime uses that map to read records without parsing type strings at
play time. The `EA::Reflection::*` type-name strings ([C11.3](../C11-Attribute-Vaults/03-type-names.md)) are
the human-readable side of this map; the records are laid out to match it.

For a reader, the practical form is a table you build once:

```python
# schema: field_hash → type (per collection class)
def read_record(buf, rec, schema):
    out = {}
    for (field_hash, value_off) in rec.field_slots:
        ftype = schema[field_hash]                 # what type is this field?
        out[field_hash] = decode_by_type(buf, value_off, ftype)
    return out
```

## The classes and their registration

Which *classes* does the schema cover? `speed.exe` holds the answer as a **28-name reflection class table** at file
offset `0x4ADD1C` — inline NUL-padded strings naming the simulation classes whose fields the vault serialises:

```
pvehicle  chopperspecs  damagespecs  rigidbodyspecs  RBVehicle  RBTrailer  RBCop
EffectsVehicle  EffectsCar  EffectsFragment  DamageRacer  DamageHeli  DamageCopCar
SuspensionTraffic  SuspensionTrailer  EngineRacer  SimpleChopper  EngineTraffic
DrawHeli  DrawTraffic  DrawNISCar  DrawCopCar  DrawRaceCar  SoundTraffic  SoundCop
SoundRacer  SoundHeli  SpikeStrip
```

These are the car, cop, and trailer and their engine/suspension/damage/effects/sound/draw sub-objects
([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) — the classes with a reflection field map. But
the map itself is **code-driven, not a stored table**: each class has a static-initialiser thunk that, at startup,
**registers the class onto a global linked list** (interning its name through a shared hash function) — the classic
"static object self-registers, list walked once at startup" idiom ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).
So the field→(type, offset) map is *built in memory by the registration code*, not laid out as a static structure you
can parse. This is precisely why recovering the *full* schema statically is XL, and why the vault was cracked instead
through the record-key hash ([C12.1](01-reflection-hash.md)) — the advanced-RE story is [C76.2](../C76-Advanced-RE/02-recovering-schema.md).

Disassembling one such thunk pins the exact shape — `EngineRacer`'s, at VA `0x8886F0`:

```
push  0x8ADDF0            ; the "EngineRacer" class-name string
call  0x5CC240            ; intern / hash the name  -> eax
mov   [node+0], eax       ; node +0x00 = interned name-hash
mov   [node+4], 0x6B6D30  ; node +0x04 = the class factory (constructor)
mov   eax, [0x92C660]     ; current list head
mov   [node+8], eax       ; node +0x08 = next (previous head)
mov   [0x92C660], node    ; list head = this node
```

So each registration node is a **3-dword `{name-hash, factory-ptr, next}`** threaded onto the global list head
**`0x92C660`**, walked once at startup; the per-field table is built in memory by the class factory (`0x6B6D30` for
`EngineRacer`), not stored — the reason the full static field map stays open ([C76.2](../C76-Advanced-RE/02-recovering-schema.md)).

> ✅ *Verified (by disassembly):* the 28-name class table is at `speed.exe` file offset `0x4ADD1C` (inline
> NUL-padded strings); each class self-registers via a static-init thunk (e.g. `EngineRacer` at VA `0x8886F0`) that
> interns its name (`call 0x5CC240`) and links a **`{name-hash @+0, factory @+4, next @+8}`** node onto the list head
> **`0x92C660`** — read directly from the instruction bytes. The per-field table is built by the class factory
> (code-driven), so the full static field-offset map remains open ([C76.2](../C76-Advanced-RE/02-recovering-schema.md)).

## Fixed and variable width

The type also fixes the value's **width**, which is how you advance to the next field
([C11.3](../C11-Attribute-Vaults/03-type-names.md)):

- **Fixed-width** (`Bool`=1, `UInt16`=2, `Float`/`UInt32`/`Reference`/`Type`=4, `Double`/`Int64`=8): read
  inline, advance by the size.
- **Variable-width** (`Text`, `KeyValueAttrib`): length-prefixed or table-referenced; advance past the whole
  blob.

A wrong type misjudges the width and desynchronises the walk from that field on — the same failure mode as a
wrong vertex stride ([C9.1](../C9-Meshes-FVF/01-vertex-buffer.md)) or a wrong TPK format
([C6.1](../C6-Texture-Codecs/01-format-field.md)). The three formats share a lesson: **a typed stream is only
readable if you honour the type at every step.**

## Building the schema in practice

You obtain a field's type three ways, in decreasing convenience:

1. **From the value's plausibility.** A field that reads as `5.0` under `Float` and a billion under `UInt32`
   is almost certainly a `Float` — human-scale numbers betray the type.
2. **From sibling collections.** The same field across many collections must share a type; cross-referencing
   pins it down.
3. **From the `EA::Reflection::*` strings.** The type-name table names the primitives the schema uses; mapping
   fields to them completes the picture.

In tuning data ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) most fields are `Float`, which is
why float-first decoding usually reads car data correctly — but confirm, because flags and references hide
among them.

---

### Key takeaways

- A value requires **field (hash) + type + offset**; the schema binds all three.
- The **type is mandatory**: identical bits are `5.0` (Float), a billion (UInt32), or a reference — you must
  know which.
- The schema is a compiled `field id → (type, offset)` map; readers are schema-driven, not fixed structs.
- Type also fixes width, which is how you advance; a wrong type desynchronises the field walk.
- Infer types from value plausibility, sibling collections, and the `EA::Reflection::*` names; tuning is
  mostly `Float` but not entirely.

**Continue:** [C12.3 — The inline value triple](03-value-triple.md) · [Chapter 12 hub](C12-Reflection-Schema.md)
