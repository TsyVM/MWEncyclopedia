# C68.4 — What "Buying" Does

> **The one-sentence version:** buying runs the `CustomizeShoppingCart` flow — selected parts become
> `FEShoppingCartItem`s, and confirming debits the player's cash, marks the parts **owned**, and installs them into
> the car's slots so the vault-driven sim picks them up; it's a state transition on the `PlayerCar`, not a file
> operation.

[← C68.3 — Parts as catalog entries](03-part-catalog.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md) ·
[Next: C68.5 — Reading vehicles in RE →](05-reading-vehicles.md)

---

## The shopping cart

Customization uses a **cart** model rather than buy-on-click. As you browse the categories
([C68.2](02-shop-categories.md)) and pick parts, each choice becomes a **`FEShoppingCartItem`** in the
**`CustomizeShoppingCart`**; you confirm the whole cart at once. This lets you *preview* a build — try a spoiler, a
paint, a turbo — and see the total cost before committing, then buy it all in one transaction. The
`ShoppingCart_BACKROOM` is the confirmation/checkout surface.

> ✅ *Verified:* `CustomizeShoppingCart`, `FEShoppingCartItem`, `ShoppingCart`, `ShoppingCart_BACKROOM`, and
> `ShoppingCart_QR` are strings in `speed.exe` — the cart, its items, and the checkout.

## What "confirm" does

Confirming the cart is a **three-part state transition** on the `PlayerCar` ([C68.1](01-car-object.md)):

1. **Debit cash.** The total price is subtracted from the player's wallet — the career economy
   ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)). If you can't afford the cart, the purchase
   is refused.
2. **Mark owned.** Each purchased part is recorded as **owned** in the player's save
   ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)) — you bought it, so it's yours to fit and re-fit
   for free thereafter.
3. **Install into slots.** Each part is assigned to its slot ([C68.1](01-car-object.md)) on the `PlayerCar`, so the
   vault lookup ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — and thus the sim
   ([Chapter 39](../C39-Vehicle-Simulation/C39-Vehicle-Simulation.md)) or renderer
   ([Chapter 70](../C70-Visual-Customisation/C70-Visual-Customisation.md)) — now sees the new configuration.

So "buying" is *not a file write* — it mutates runtime and save state: the wallet, the owned-parts set, and the
car's slot map. The on-disk car data ([Chapter 8](../C8-Geometry-Solids/C8-Geometry-Solids.md) geometry,
[Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md) tuning) is *read-only reference*; what changes is *which
of it your car points at*.

> 🟡 *Reasoned:* the debit → own → install sequence is the natural reading of the cart flow, the career economy, and
> the slot model ([C68.1](01-car-object.md)); the exact save fields are [Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)
> territory. The cart strings and the read-only-reference model are verified.

## Owned vs installed

The transaction separates two states that the player experiences as one:

- **Owned** — you've *paid for* the part; it's in your save. Buying sets this, once, permanently.
- **Installed** — the part is *currently in its slot* on this car. Fitting/removing changes this freely.

Because owning is permanent and installing is free, you pay for a part *once* and can then swap it in and out at no
cost — the shop only charges for parts you don't yet own. This is why re-tuning a car
([C13.6](../C13-Vault-CarTuning/06-retuning.md)) is cheap once you've bought the parts: you're changing *installed*,
not *owned*. The distinction is the economic spine of the customization loop — spend cash to *acquire*, then
reconfigure for free.

## Buying as the bridge to the sim

Zooming out, the buy flow is the **last link** in the chain from shop to driving:

```
category (C68.2) -> part (C68.3) -> cart -> confirm -> owned + installed -> vault -> sim/renderer
```

Everything upstream (browsing, choosing) is *pending*; the confirm is what makes it *real* — committing the cash and
writing the slot so the car actually changes. This is the moment the *player's intent* ("I want this turbo") becomes
the *car's state* ("this turbo is installed"), which the sim then reads as behaviour
([C68.1](01-car-object.md)). Reading the buy flow completes the customization picture: the categories organise, the
catalog identifies, and the cart *commits* — turning a selection into an installed, simulated part.

## RE implications

- **The cart model** — selections become `FEShoppingCartItem`s in `CustomizeShoppingCart`; confirm buys the whole
  cart.
- **Confirm = three-part transition** — debit cash ([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)),
  mark owned ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)), install into slots.
- **Not a file write** — the on-disk car data is read-only reference; buying mutates wallet/owned/slot state.
- **Owned vs installed** — pay once (owned, permanent), reconfigure free (installed) — the economic spine.

---

### Key takeaways

- Customization uses a **cart** — selected parts become **`FEShoppingCartItem`s** in the **`CustomizeShoppingCart`**;
  you preview a build and **confirm the whole cart** at once.
- Confirming is a **three-part state transition** on the `PlayerCar`: **debit cash** (career economy,
  [Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)), **mark owned**
  ([Chapter 31](../C31-Save-MemoryCard/C31-Save-MemoryCard.md)), **install into slots** so the vault/sim see it.
- Buying is **not a file write** — the on-disk geometry/tuning is read-only reference; what changes is **which of it
  your car points at**.
- **Owned** (paid, permanent) vs **installed** (currently fitted, free to change) — pay once, reconfigure free — the
  economic spine of the loop, and why re-tuning ([C13.6](../C13-Vault-CarTuning/06-retuning.md)) is cheap.
- The confirm is where **player intent becomes car state** — the last link from shop to simulated behaviour.

**Continue:** [C68.5 — Reading vehicles in RE](05-reading-vehicles.md) · [Chapter 68 hub](C68-Vehicles-Customisable-Object.md)
