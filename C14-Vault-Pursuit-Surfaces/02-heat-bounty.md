# C14.2 — The Heat & Bounty System

> **The one-sentence version:** heat is a tuned curve mapping your notoriety to the police response —
> `heat_meter`, `cops_in_pursuit`, `bounty_in_pursuit` and their kin parameterise how many and which cops
> appear at each level and how bounty accrues — so escalation is data you can retune.

[← C14.1 — The pursuit & AI vault](01-pursuit-ai.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md) ·
[Next: C14.3 — Surface records →](03-surfaces.md)

---

## Heat is a level → response mapping

The rising intensity of Most Wanted's pursuits is not scripted — it is a curve. As your **heat** increases,
the game consults vault data to decide the police response: more units, faster cars, helicopters, roadblocks,
and higher bounty. The relevant collections are verified present: `heat_meter` (`0xE9A4423C`), plus pursuit
counters and accumulators like `cops_in_pursuit`, `bounty_in_pursuit`, `cost_to_state_in_pursuit`, and
`cops_destroyed_in_pursuit` (all in the string table, [C11.2](../C11-Attribute-Vaults/02-erts-strings.md)).

Conceptually, heat indexes a table: at heat level *N*, use these `AICopManager` settings
([C14.1](01-pursuit-ai.md)), spawn these `COP*` vehicles, apply this bounty rate. Raising the ceiling or
steepening the curve makes the game escalate harder and sooner.

## The pieces

- **`heat_meter`** — the heat scale itself and how it fills/decays; the parameterisation of notoriety.
- **`cops_in_pursuit`** — how many police are active, a key escalation lever tied to heat and `AICopManager`.
- **`bounty_in_pursuit`** — how bounty accrues during a chase (the currency of the Blacklist progression).
- **`cost_to_state_in_pursuit`** — the damage/cost you inflict, feeding the pursuit's economy.

These are attribute fields resolved and edited like any others ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md));
their values shape the pursuit's difficulty and rewards.

> ✅ *Verified:* `heat_meter` (0xE9A4423C) and the `pursuit`-family names (`cops_in_pursuit`,
> `bounty_in_pursuit`, `cost_to_state_in_pursuit`, `cops_destroyed_in_pursuit`) are present in the vault.
> 🟡 *Reasoned:* the exact per-level table structure (which fields key on heat level) is inferred from the
> names and MW's known heat system; the collections and their presence are verified.

## Heat couples to the pursuit layers

Heat is the *selector*; the pursuit layers ([C14.1](01-pursuit-ai.md)) are the *content*. Heat decides *which*
`AICopManager` scale and *which* `COP*` vehicles apply; those collections decide *how* they behave. So a heat
change and a pursuit change compound:

- Raise the heat curve → the game reaches the harsh settings sooner.
- Raise the harsh settings (`AICopManager`, roster) → those levels hit harder when reached.

A convincing "brutal cops" mod adjusts both: an escalation curve that ramps fast, feeding pursuit layers that
are themselves aggressive.

## Bounty and progression

Bounty is the Blacklist's scoring: `bounty_in_pursuit` and related fields govern how quickly you earn the
notoriety that advances the career. Tuning them changes pacing — higher bounty rates make the Blacklist climb
faster; lower rates stretch it out. Because it is vault data, this is a value edit, not a code change
([C14.6](06-editing-gameplay.md)).

## Editing implications

- **Curve vs content.** Decide whether you want escalation to arrive *sooner* (edit `heat_meter`/level
  mapping) or *harder* (edit `AICopManager`/roster) — they are different levers.
- **Keep it winnable.** Escalation that outruns the player's cars makes pursuits unlosable *and* unwinnable;
  test against real driving.
- **Bounty tunes pacing.** Adjust `bounty_in_pursuit` to speed or slow Blacklist progression.
- **Same discipline.** Resolve, edit in the declared type, verify by re-resolving
  ([C12.6](../C12-Reflection-Schema/06-writing-values.md)).

---

### Key takeaways

- Heat is a tuned **level → response** curve, not a script; it selects pursuit settings by notoriety.
- Key collections: `heat_meter` (the scale), `cops_in_pursuit` (scale lever), `bounty_in_pursuit` (rewards),
  `cost_to_state_in_pursuit` (economy) — all verified.
- Heat *selects* which `AICopManager`/`COP*` content applies; heat and pursuit edits compound.
- Bounty fields tune Blacklist pacing.
- Choose the curve lever vs the content lever deliberately, keep pursuits winnable, and edit with the standard
  resolve-then-write discipline.

**Continue:** [C14.3 — Surface records](03-surfaces.md) · [Chapter 14 hub](C14-Vault-Pursuit-Surfaces.md)
