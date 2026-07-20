# C33.6 — Using the Registry in RE

> **The one-sentence version:** navigate the runtime via the registry — scan the ~124 registrations from the
> intern function and list-heads, name classes by computing/matching their key hashes, and identify live
> objects by vtable pointer + size — turning `speed.exe` into a browsable class system.

[← C33.5 — Sizes & vtables as fingerprints](05-fingerprints.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md) ·
[Next: Chapter 34 — VTable Anatomy & Method Roles →](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)

---

## The registry-driven RE workflow

The registry is the runtime's index, so RE flows through it:

1. **Find the registrations.** Scan `.text` for calls to the intern function `0x5CC240`
   ([C33.1](01-intern.md)); filter to the ~124 that link onto a family list-head
   ([C33.2](02-factory-registration.md)).
2. **Read each registration** — name (or key), family (role), constructor, vtable
   ([C33.2](02-factory-registration.md)).
3. **Measure** — object size and vtable method count ([C33.5](05-fingerprints.md)).
4. **Catalogue** — build the class reference ([C33.4](04-class-reference.md)).
5. **Cross-reference data** — match class names to vault collections
   ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)).

The output is the class system as a browsable list, from which you can dive into any class's behaviour
([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).

## Naming classes by hash

Because the intern hash is the reflection hash ([C33.1](01-intern.md)), names and keys convert both ways
([C12.1](../C12-Reflection-Schema/01-reflection-hash.md)):

```python
# name → key: find where a known class is registered/used
key = reflection_hash("EngineRacer")          # 0xB2809518

# key → name: recover an unknown class's name
def recover_name(key, dictionary):            # dictionary = candidate names (ErtS strings, C11.2)
    for name in dictionary:
        if reflection_hash(name) == key:
            return name
    return None
```

So a class whose name isn't directly in the binary can often be recovered by hashing candidate names (the
vault's `ErtS` strings, [C11.2](../C11-Attribute-Vaults/02-erts-strings.md), are a ready dictionary) and matching
the key. This computability ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)) is what makes the class
system *legible* — mystery keys resolve to readable names.

## Identifying a live object

To name an object you find in memory (e.g. while tracing the frame loop,
[Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)):

1. **Read its vtable pointer** at object `+0x00` ([C32.5](../C32-Runtime-Class-System/05-object-model.md)).
2. **Match the vtable** against the catalogue ([C33.4](04-class-reference.md)) — the vtable address identifies
   the class.
3. **Confirm with size** — the object's byte size should match the class's size
   ([C33.5](05-fingerprints.md)).

So an anonymous object becomes a named class, and you know its role, size, and behaviour. This is the core loop
of runtime RE: from a pointer in memory to a class in the catalogue.

## Anchors and pitfalls

The registry-driven approach has a few reliable anchors and one classic pitfall:

- **Anchors** — the intern function `0x5CC240` ([C33.1](01-intern.md)) and the eleven list-heads
  ([C32.3](../C32-Runtime-Class-System/03-eleven-families.md)) are fixed entry points.
- **Pitfall — intern ≠ register.** Most `0x5CC240` calls are plain hashes, not registrations
  ([C33.1](01-intern.md)); filter by the list-head link ([C33.2](02-factory-registration.md)) or you'll catalogue
  non-classes.
- **Pitfall — lookalike functions.** As with the allocator ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)),
  a naive scan can mistake a getter stub for a real function; confirm behaviour, don't key on call-site shape
  alone.

## The registry ties the book together

The class registry is where the encyclopedia's two halves meet:

- **The file formats** (Parts I–VI) are the *data* that names and configures classes.
- **The class system** (this part) is the *code* the data drives.
- **The shared reflection hash** ([C32.4](../C32-Runtime-Class-System/04-registration.md)) is the *link* — a
  name is a key in both worlds.

So decoding a class and decoding its vault collection are two views of one entity, joined by the hash. The
registry is the hinge: from a name in data to a constructor in code, and back.

---

### Key takeaways

- RE the runtime **through the registry**: find registrations (intern + list-heads), read/measure/catalogue,
  cross-reference the vault.
- **Name classes by hash** — compute a name's key, or recover a name by matching a key to a dictionary
  (computable reflection hash).
- **Identify live objects** by vtable pointer + size against the catalogue.
- Anchors: `0x5CC240` and the eleven list-heads; pitfalls: intern≠register, and lookalike getter stubs.
- The registry is the **hinge** between data (file formats) and code (classes), joined by the shared reflection
  hash.

**Continue:** [Chapter 34 — VTable Anatomy & Method Roles](../C34-VTable-Anatomy/C34-VTable-Anatomy.md) ·
[Chapter 33 hub](C33-Class-Registry-Factories.md)
