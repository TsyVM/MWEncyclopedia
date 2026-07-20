# C50.4 — VTable Verification

> **The one-sentence version:** a claimed vtable is real if it's a clean run of code pointers, and the run length
> is the class's method count — `AIVehicle` at `0x00891998` has exactly 351 consecutive `.text` pointers, verifying
> both that it's a vtable and that it's the biggest class in the game.

[← C50.3 — Hash verification](03-hash-verification.md) · [Chapter 50 hub](C50-Verification-Methodology.md) ·
[Next: C50.5 — Cross-checking →](05-cross-checking.md)

---

## The method-count check

Most Wanted's runtime is built on C++ classes with vtables
([Chapter 34](../C34-VTable-Anatomy/C34-VTable-Anatomy.md)) — arrays of function pointers. This gives a clean class
verification: **a real vtable is a run of pointers into the code section, and where the run ends is where the class
ends.** So you count consecutive code pointers:

```python
def count_vtable(va):
    fo = va - 0x400000; n = 0
    while 0x401000 <= u32(exe, fo + n*4) < 0x400000 + len(exe):  # points into code?
        n += 1
    return n
```

A valid vtable gives a clean count (every entry is a `.text` address); a non-vtable address gives a short or zero
run (the "pointers" aren't code). So the count *both* confirms the address is a vtable *and* measures the class's
method count. `count_vtable(0x00891998) == 351` verifies `AIVehicle` is a class with 351 methods.

> ✅ *Verified:* the method-count check confirmed every class-vtable claim in the book — e.g. `AIVehicle`
> `0x00891998`/351, `EngineRacer` `0x008AB6A0`/123, `AIGoalFleePursuit` `0x00892D00`/94, `DamageVehicle`
> `0x008AD288`/127 — each a clean run of exactly the stated number of `.text` pointers.

## Method count as a class fingerprint

The method count is a strong **fingerprint** — it identifies and characterises a class:

- **It confirms the class.** A claimed class with a clean N-pointer vtable at the claimed address *is* that class
  (the run is too clean to be coincidence).
- **It measures complexity.** The count is roughly how much behaviour the class has —
  `AIVehicle` (351) is the most complex; `PathFinder` (16) is a simple service
  ([C47.5](../C47-AI-Driver-Vehicle/05-reading-ai-brain.md)).
- **It ranks a family.** Within the `Engine*` or `Damage*` families
  ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)), the counts order the tiers
  (`DamageVehicle` 127 > `DamageRacer` 98 > `DamageCopCar` 36).

So the count is not just a yes/no verification — it's a *measurement* that places a class in the engine's
complexity landscape. The book uses it both to confirm classes exist and to reason about their roles (the biggest
classes are the driver brains; the smallest are services and data-only specialisations).

## The shared-vtable proof

A subtle and powerful use of vtable verification is the **shared-vtable proof** — when two differently-named things
have the *same* vtable, they are the *same class*:

- **`AIVehicleEmpty` shares `AIVehicle`'s vtable** (`0x00891998`, 351 methods) → `AIVehicleEmpty` *is* `AIVehicle`
  with the planner idle, not a new behaviour ([C47.1](../C47-AI-Driver-Vehicle/01-aivehicle-hierarchy.md)).
- **Ten AI goals share one 12-method vtable** (`0x00892B20`) → they are data-only specialisations differing only in
  their action menu, not their code ([C46.3](../C46-AI-Goals-Actions/03-data-only-goals.md)).

This is a conclusion you *cannot* reach from names or behaviour alone — only the bytes reveal that two names map to
one implementation. The shared vtable is decisive proof of code identity, and it uncovered two of the book's most
important architectural findings (the player-is-an-AI unification and the data-only-goals design). Byte-level
verification here reveals *structure the names hide*.

## Vtable offsets and dispatch

Vtable verification also validates *dispatch* claims — that a `call [reg+offset]` reaches a specific method
([C41.4](../C41-Physics-RigidBody/04-simulate-thiscall.md)):

- **`call [eax+0x4C]`** in `Physics::Simulate` dispatches vtable slot 19 (0x4C / 4) — a specific virtual on the
  body.
- **The slot index** ties the calling code to the class's vtable layout — you can follow which method a dispatch
  invokes by reading that slot's pointer in the vtable.

So the vtable connects the *call sites* (which slots they dispatch) to the *classes* (which methods those slots
hold). Verifying a dispatch means reading the vtable entry at the offset and confirming it points to a plausible
method. This closes the loop between byte verification ([C50.2](02-byte-verification.md), the `call` instruction)
and vtable verification (this page, the slot it targets) — the two techniques together trace polymorphic behaviour
through the code.

## RE implications

- **The method-count check** — a real vtable is a clean run of `.text` pointers; the run length is the method
  count.
- **Method count is a fingerprint** — confirms the class, measures complexity, ranks a family.
- **The shared-vtable proof** — same vtable = same class (uncovered player-is-AI and data-only goals).
- **Vtable offsets validate dispatch** — a `call [reg+N]` targets slot N/4; the vtable ties call sites to methods.

---

### Key takeaways

- **VTable verification** confirms a class: a real vtable is a **clean run of code pointers**, and the **run length
  is the method count** (`AIVehicle` = 351, the biggest in the game).
- The method count is a **class fingerprint** — it confirms existence, measures complexity, and ranks family tiers
  (`DamageVehicle` 127 > `DamageRacer` 98 > `DamageCopCar` 36).
- The **shared-vtable proof** — two names with the same vtable are the **same class** — uncovered the *player-is-an-AI*
  unification and the *data-only goals* design (structure the names hide).
- Vtable offsets **validate dispatch** — `call [eax+0x4C]` targets a specific slot, tying call sites to methods.
- Together with byte verification, it **traces polymorphic behaviour** through the code.

**Continue:** [C50.5 — Cross-checking & correcting received wisdom](05-cross-checking.md) · [Chapter 50 hub](C50-Verification-Methodology.md)
