# C33.3 — Constructing an Object

> **The one-sentence version:** constructing an object is the registry's payoff — hash a name to its key, find
> the registered factory, allocate the object's memory, and run its constructor — turning a name (often from
> data) into a live instance.

[← C33.2 — Factory registration](02-factory-registration.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md) ·
[Next: C33.4 — The class reference →](04-class-reference.md)

---

## From name to instance

Once classes are registered ([C33.2](02-factory-registration.md)), constructing one by name is four steps:

```python
def construct(name):
    key = intern(name)                      # 0x5CC240 → reflection hash (C33.1)
    factory = find_registration(key)        # search the family lists for the key
    obj = allocate(factory.size)            # allocate memory (Chapter 35)
    factory.constructor(obj)                # run the constructor → live instance
    return obj                              # obj[+0] now points at factory.vtable
```

1. **Hash the name** to its key ([C33.1](01-intern.md)).
2. **Find the factory** — search the family lists for the registration with that key.
3. **Allocate** the object's memory ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) — `size`
   bytes ([C32.5](../C32-Runtime-Class-System/05-object-model.md)).
4. **Construct** — run the constructor, which sets the vtable pointer and initialises fields.

The result is a live object ([C32.1](../C32-Runtime-Class-System/01-data-to-objects.md)) — memory + vtable +
initialised state.

## Data-driven construction

The names usually come from **data**, which is what makes the system data-driven
([C32.4](../C32-Runtime-Class-System/04-registration.md)):

- The **vault** names a behavior class ([C13.2](../C13-Vault-CarTuning/02-behavior-classes.md)) —
  `EngineRacer` — and the runtime constructs it.
- A **spawn system** names an entity to create (a cop, a traffic car).
- A **scene/script** names an object to instantiate.

Because the name's key is the shared reflection hash ([C32.4](../C32-Runtime-Class-System/04-registration.md)),
data naming `EngineRacer` finds the `EngineRacer` factory and builds the `EngineRacer` class — data selects code
by name. This is the payoff of the whole class system: **content is data that names code**, and the registry
resolves it.

## Configure after construct

Construction builds the object; **data then configures it**:

```
construct(class) → a live object with default state
   → apply data (vault fields, tuning) → the object's specific configuration
```

So a fully-tuned car and a base car are the same constructed class ([C32.1](../C32-Runtime-Class-System/01-data-to-objects.md))
with different applied data — the vault's resolved values ([C12.5](../C12-Reflection-Schema/05-resolving-values.md))
set the fields. Construction is the *identity*; configuration is the *setup*. This two-step (build then
configure) is why one class serves many instances.

> ✅ *Verified:* construction uses the intern function ([C33.1](01-intern.md)) to key the factory and the real
> allocator ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md), `0x5D29D0`) to allocate; the
> object model (size + vtable + fields) is the standard layout ([C32.5](../C32-Runtime-Class-System/05-object-model.md)).
> 🟡 *Reasoned:* the exact factory-search and constructor-dispatch code is per-site; the four-step model
> (hash → find → allocate → construct) is grounded in the verified intern/allocator/object-model.

## Lifetime and destruction

Objects are also destroyed — the mirror of construction. A class's vtable includes a **destructor**
([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)), and freeing an object runs it and returns its
memory to the pool ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)). So an object's lifetime is
construct → (configure → update each frame) → destruct → free. The registry handles construction; the memory
system ([Chapter 35](../C35-Memory-Management/C35-Memory-Management.md)) and the frame loop
([Chapter 37](../C37-Frame-Spine-Modules/C37-Frame-Spine-Modules.md)) handle the rest of the lifetime.

## RE implications

- **Construction is hash → factory → allocate → construct** — trace from a `construct`/`create` call.
- **Names come from data** — the vault/spawn/script name the class ([C32.4](../C32-Runtime-Class-System/04-registration.md)).
- **Build then configure** — a constructed object is defaulted; data sets its fields
  ([C12.5](../C12-Reflection-Schema/05-resolving-values.md)).
- **Follow the constructor and destructor** in the vtable for the object's setup/teardown
  ([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)).

---

### Key takeaways

- Constructing an object: **hash the name → find the factory → allocate → run the constructor** → live instance.
- Names usually come from **data** (vault, spawn, script), so construction is **data-driven** via the shared
  hash.
- **Build then configure**: construction fixes the identity, data (vault fields) sets the specific state.
- An object's lifetime is construct → configure → update → destruct → free; the registry owns construction.
- Trace construction from `create` calls; follow constructor/destructor in the vtable for setup/teardown.

**Continue:** [C33.4 — The class reference](04-class-reference.md) · [Chapter 33 hub](C33-Class-Registry-Factories.md)
