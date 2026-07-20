# C33.1 — The Intern Function

> **The one-sentence version:** `0x5CC240` is the intern function — it takes a class-name string, checks it's
> non-empty, and hashes it with the reflection hash (wrapping the core `lookup2` at `0x5CC090`) — but a call to
> it means "hash a name," not always "register a class."

[← Chapter 33 hub](C33-Class-Registry-Factories.md) · [Next: C33.2 — Factory registration →](02-factory-registration.md)

---

## The function, verified

`0x5CC240` is the runtime's name-**intern** function. Its opening bytes, read straight from `speed.exe`, are:

```
8B 44 24 04    mov  eax, [esp+4]     ; the const char* name argument
85 C0          test eax, eax         ; null check
74 23          jz   …                ; bail if null
80 38 00       cmp  byte [eax], 0    ; empty-string check
74 1E          jz   …                ; bail if empty
8B C8          mov  ecx, eax         ; name → ecx
56             push esi              ; … then measure + hash
```

So it takes a `const char*`, guards against null/empty, and proceeds to measure and hash the string. The
disassembly confirms it wraps the **core hash at `0x5CC090`** — the Jenkins `lookup2` seeded `0xABCDEF00`
([C2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)) — and returns the 32-bit key. This is *the*
function that turns a name into a reflection-hash key across the whole runtime.

> ✅ *Verified:* `0x5CC240` reads a string arg, null/empty-checks it, and hashes it (reflection hash, wrapping
> `0x5CC090`); the hash constants (`0x9E3779B9`, `0xABCDEF00`) are present in `speed.exe`.

## "Hash a name" ≠ "register a class"

The most important subtlety: **a call to `0x5CC240` does not mean a class is being registered.** A linear scan
of `.text` finds **686 call sites** to `0x5CC240`, but only about **124** are class registrations. The other
~560 hash names for other purposes:

- **Vault field/collection lookups** ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) — hashing
  a field name to find its value.
- **Runtime lookups** — hashing a name to find an object, resource, or class by key.
- **General interning** — anywhere the code needs a name's key.

So to tell a **registration** from a plain hash, you read *what the caller does with the returned key*
([C33.2](02-factory-registration.md)): a registration links a constructor+vtable onto a family list-head; a
lookup searches for the key. The intern function is shared; its callers differ.

## Why one intern function

Centralising name-hashing in one function is deliberate and useful:

- **One hash, everywhere.** Every name — class, field, collection — goes through `0x5CC240`, so they all share
  the reflection hash ([C32.4](../C32-Runtime-Class-System/04-registration.md)). This is *why* a vault
  collection and a class of the same name share a key.
- **One place to guard.** The null/empty checks and the hash live in one function, not duplicated at 686 sites.
- **One RE anchor.** Finding `0x5CC240` and its callers finds every name-keyed operation in the game.

So the intern function is the hub of the reflection-hash world ([C32.4](../C32-Runtime-Class-System/04-registration.md)):
data and code names both pass through it.

## Computing what it computes

Because it's the reflection hash, you can reproduce it ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)):

```python
intern = reflection_hash          # lookup2 seeded 0xABCDEF00
assert intern("EngineRacer") == 0xB2809518     # a class / collection key
```

So you can go from a class name to the key `0x5CC240` would produce, find that key in the binary, and locate the
class's registration and uses ([C33.6](06-using-registry.md)). This computability is the lever for mapping the
registry.

## RE implications

- **`0x5CC240` is the name-hash hub** — its callers are every name-keyed operation.
- **Distinguish registration from lookup** by the caller's behaviour, not the call itself
  ([C33.2](02-factory-registration.md)).
- **Compute keys** with `reflection_hash` to find a named class/field in the binary
  ([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)).
- **~124 of 686 calls register classes** — filter to registrations to build the catalogue
  ([C33.4](04-class-reference.md)).

---

### Key takeaways

- `0x5CC240` is the **intern** function: guards a class-name string, hashes it (reflection hash, wrapping
  `0x5CC090`), returns the key — verified.
- A call to it means **"hash a name," not "register a class"** — 686 calls, only ~124 registrations.
- Distinguish registration from lookup by what the caller does with the key.
- Centralising name-hashing is why data (vault) and code (class) names share the reflection hash.
- Compute the key with `reflection_hash` to locate a named class/field in `speed.exe`.

**Continue:** [C33.2 — Factory registration](02-factory-registration.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md)
