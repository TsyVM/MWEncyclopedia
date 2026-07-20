# C11.3 — The Reflection Type-Name Table

> **The one-sentence version:** after the `ErtS` names comes a table of the 16 `EA::Reflection::*` primitive
> type names — `Bool`, the signed/unsigned integers `Int8`…`UInt64`, `Float`, `Double`, `Text`, `Type`,
> `Reference`, `KeyValueAttrib` — and every field in the vault has one of these as its data type.

[← C11.2 — The ErtS string table](02-erts-strings.md) · [Chapter 11 hub](C11-Attribute-Vaults.md) ·
[Next: C11.4 — The data records →](04-data-records.md)

---

## The 16 types

Immediately after the `ErtS` string block, the file carries fully-qualified C++ type names from EA's
reflection system. Extracted from the real file, there are exactly **16**:

| Type name | Size (bytes) | Meaning |
|---|---|---|
| `EA::Reflection::Bool` | 1 | boolean |
| `EA::Reflection::Char` | 1 | character |
| `EA::Reflection::Int8` / `UInt8` | 1 | 8-bit integer |
| `EA::Reflection::Int16` / `UInt16` | 2 | 16-bit integer |
| `EA::Reflection::Int32` / `UInt32` | 4 | 32-bit integer |
| `EA::Reflection::Int64` / `UInt64` | 8 | 64-bit integer |
| `EA::Reflection::Float` | 4 | 32-bit float |
| `EA::Reflection::Double` | 8 | 64-bit float |
| `EA::Reflection::Text` | var | string |
| `EA::Reflection::Type` | 4 | a type/enum reference |
| `EA::Reflection::Reference` | 4 | reference to another collection |
| `EA::Reflection::KeyValueAttrib` | var | key-value attribute (nested) |

This is a small, closed, standard set — the primitive vocabulary in which every attribute value is expressed.
A car's top speed is a `Float`; a boolean flag is a `Bool`; a texture pointer is a `Reference`/`TextureRef`; a
name is `Text`.

## Why the type table matters

The type is what turns four raw bytes into a *value*. The same 32-bit pattern is a very different thing as a
`Float`, a `UInt32`, or a `Reference`:

- As `Float`, `0x40A00000` is `5.0`.
- As `UInt32`, it is `1 084 227 584`.
- As `Reference`, it is a hash pointing at another collection.

So you cannot interpret a field's bytes without knowing its type, and the type table is where the vault
records which is which. This is the crux of the resolved-value model
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)): **field-hash selects *which* attribute,
type selects *how to read its bytes*.**

## Fixed vs variable width

The types split into two groups that a reader must handle differently:

- **Fixed-width** (`Bool`…`Double`, `Type`, `Reference`) — the value is exactly the type's byte size, read
  inline. Most numeric tuning values are here.
- **Variable-width** (`Text`, `KeyValueAttrib`) — the value is a length-prefixed or table-referenced blob,
  not a fixed number of inline bytes. Strings and nested attribute sets are here.

When walking a record's fields ([C11.4](04-data-records.md)), the type tells you how many bytes the value
occupies and therefore where the next field begins — so a wrong type reading desynchronises the walk, exactly
as a wrong stride desynchronises a vertex buffer ([C9.1](../C9-Meshes-FVF/01-vertex-buffer.md)).

## Types you'll meet most

In practice the tuning and gameplay vaults are dominated by a handful:

- **`Float`** — the workhorse: speeds, forces, rates, multipliers, durations.
- **`UInt32` / `Int32`** — counts, flags, enum-like selectors.
- **`Reference`** — pointers between collections, including the parent reference (`default`) that drives
  inheritance ([C11.4](04-data-records.md)).
- **`Bool`** — on/off switches.

`Text`, `Double`, and the 64-bit integers appear but are comparatively rare; `KeyValueAttrib` handles the
nested cases.

## Editing implications

- **Never change a field's type to fit a value.** Write the value *in the field's declared type*: a `Float`
  field takes float bytes, a `UInt32` field integer bytes. Writing float bytes into an integer field yields
  nonsense.
- **Respect width when overwriting.** A fixed-width value must keep its width for an in-place edit; changing
  width shifts every following field.
- **Handle variable-width carefully.** Editing a `Text` value changes its length and therefore the record
  size — a repack, not an in-place edit.

---

### Key takeaways

- The vault uses a closed set of **16** `EA::Reflection::*` primitive types (Bool, Int/UInt 8–64, Float,
  Double, Text, Type, Reference, KeyValueAttrib).
- The type decides how a field's bytes become a value — the same bits differ as Float vs UInt32 vs Reference.
- Types are fixed-width (read inline) or variable-width (`Text`, `KeyValueAttrib`); width sets where the next
  field starts.
- `Float`, `UInt32/Int32`, `Reference`, and `Bool` dominate tuning/gameplay data.
- Edit values in their declared type and width; changing type or width is a repack, not an in-place edit.

**Continue:** [C11.4 — The data records](04-data-records.md) · [Chapter 11 hub](C11-Attribute-Vaults.md)
