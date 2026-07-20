# Chapter 14 — Vault Categories: Pursuit, Surfaces & Gameplay

> **Goal of this chapter:** tour the rest of the vault — the police pursuit and AI data, the surface and
> effect/destructible records, and the separate `gameplay` and `FE_ATTRIB` vaults — so you can find and retune
> the systems that make Most Wanted's world react.

Chapter 13 covered cars; this chapter covers everything else the vault drives: how aggressively the police
chase, how the heat system escalates, how road surfaces change grip and sound, how props shatter, and where
the front-end and gameplay rules live. It is the same reflection model
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)) applied to the game's *behaviour* rather
than its cars.

> **Verified against retail data.** Every collection named here is located by reflection hash in the live
> `GLOBAL/attributes.bin`: `AIPursuit = 0x1F319B62`, `AICopManager = 0x5DB210B6`, `COPMIDSIZE = 0x7D29EBD1`,
> `COPSPORT = 0x5339838D`, `COPHELI = 0x47C914A3`, `carsurface = 0xFDA45513`, `terraindriving = 0x0AEE9EE6`,
> `pursuit = 0xDAA252C2`, `heat_meter = 0xE9A4423C` — all present. And two further vaults are confirmed as
> VPAK files: `GLOBAL/gameplay.bin` and `GLOBAL/FE_ATTRIB.bin` (9 blocks).

---

## Deep-dive pages

- [C14.1 — The pursuit & AI vault](01-pursuit-ai.md): `AIPursuit`, `AICopManager`, and the `COP*` vehicle
  roster that defines the police.
- [C14.2 — The heat & bounty system](02-heat-bounty.md): `heat_meter`, `cops_in_pursuit`, `bounty_in_pursuit`
  — how escalation is tuned.
- [C14.3 — Surface records](03-surfaces.md): `carsurface` and `terraindriving` — how the ground changes grip,
  sound, and effects.
- [C14.4 — Effects & destructibles](04-effects-destructibles.md): collision-world events, smackable props, and
  the `fx*` effect instances.
- [C14.5 — The gameplay & FE_ATTRIB vaults](05-other-vaults.md): the separate VPAK files and how they relate to
  the master vault.
- [C14.6 — Editing gameplay safely](06-editing-gameplay.md): retuning pursuit and surfaces with the same
  resolve-then-write discipline.

---

## 14.1 The police are data

Most Wanted's signature pursuit is entirely attribute-driven. `AIPursuit` (`0x1F319B62`) tunes chase
behaviour — how the police pursue, ram, and coordinate; `AICopManager` (`0x5DB210B6`) governs how many cops
spawn and when reinforcements arrive; and the **`COP*` roster** (71 collections — `COPMIDSIZE`, `COPSPORT`,
`COPSUV`, `COPHELI`, `COPGTO`, …) defines each police vehicle, using the same behavior-collection model as
player cars ([C13.2](../C13-Vault-CarTuning/02-behavior-classes.md)). Tuning the pursuit is editing these
collections ([C14.1](01-pursuit-ai.md)).

## 14.2 Heat is a curve

The escalation you feel as heat rises is tuned data: `heat_meter` (`0xE9A4423C`) and companions like
`cops_in_pursuit` and `bounty_in_pursuit` parameterise how many and which police appear at each heat level and
how bounty accrues. Heat is effectively a curve mapping notoriety to police response, and that curve lives in
the vault ([C14.2](02-heat-bounty.md)).

## 14.3 The ground talks back

Surfaces are collections too. `carsurface` (`0xFDA45513`) and `terraindriving` (`0x0AEE9EE6`) define how
different ground types affect the car — grip multipliers that make dirt slippery and asphalt planted, plus the
sound and particle effects a surface produces ([C14.3](03-surfaces.md)). This is why leaving the road changes
both how the car handles and what you hear and see.

## 14.4 Things that break

The world's reactive props and collision events are vault-driven: collision-world event collections
(`collisionworld.interrupts`, `collworld_*`) and smackable/destructible records govern what happens when you
hit a sign, a fence, or traffic, and the many `fx*` instances ([C11.2](../C11-Attribute-Vaults/02-erts-strings.md))
name the visual effects those events trigger ([C14.4](04-effects-destructibles.md)).

## 14.5 More than one vault

The master `attributes.bin` is not alone. `GLOBAL/gameplay.bin` and `GLOBAL/FE_ATTRIB.bin` are **also VPAK
files** ([C11.1](../C11-Attribute-Vaults/01-vpak-header.md)) — the gameplay-rules vault and the front-end
(menu/UI) attribute vault respectively — read with the exact same tools as the main vault
([C14.5](05-other-vaults.md)). Knowing which vault owns which data is half of finding it.

---

### Key takeaways

- Pursuit is data: `AIPursuit` (chase behaviour), `AICopManager` (spawning), and the 71-strong `COP*` roster —
  all verified collections.
- Heat/bounty escalation is a tuned curve (`heat_meter`, `cops_in_pursuit`, `bounty_in_pursuit`).
- Surfaces (`carsurface`, `terraindriving`) change grip, sound, and effects by ground type.
- Collision events, smackable props, and `fx*` effect instances make the world reactive.
- There are multiple VPAK vaults — `attributes.bin`, `gameplay.bin`, `FE_ATTRIB.bin` — read with identical
  tools.

**Next:** [Chapter 15 — The Master Track File & Streaming Sections](../C15-Track-Streaming/C15-Track-Streaming.md):
leaving the vault for the world itself.
