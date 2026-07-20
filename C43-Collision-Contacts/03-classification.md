# C43.3 — Classification: What Did I Hit?

> **The one-sentence version:** every contact is classified by what was hit — `carhitwall`, `carhitcar`,
> `carhitsmackable`, `carscrapewall` — verified vault keys (their reflection hashes appear in `attributes.bin`),
> and this tag is the hinge on which the reaction, damage, and presentation all branch.

[← C43.2 — The contact record](02-contact-records.md) · [Chapter 43 hub](C43-Collision-Contacts.md) ·
[Next: C43.4 — Reaction records →](04-reactions.md)

---

## The classification tag

A contact isn't just "a touch" — it's a touch *of a particular kind*, and the kind is a **classification tag**.
The contact update ([C43.1](01-detection.md)) determines what the body hit and tags the contact accordingly:

| Tag | What was hit |
|---|---|
| `carhitwall` | a wall / static barrier |
| `carhitcar` | another car |
| `carhitsmackable` | a knock-over prop ([C43.5](05-smackables.md)) |
| `carscrapewall` | a *sustained* grind along a wall |

These are verified vault keys — the reflection hash of each appears in `attributes.bin`: `carhitcar` (`0x8FE79512`)
**×6**, `carhitwall` (`0x96BA88FF`) **×4**, `carscrapewall` (`0x811DE877`) **×4**, `carhitsmackable`
(`0xA906E973`) **×1**. That these exact strings hash to values present in the vault is strong verification: a
random string wouldn't, so the tags are real keys the engine uses.

> ✅ *Verified:* the classification tags are real vault keys — `rh("carhitcar")=0x8FE79512` (×6),
> `rh("carhitwall")=0x96BA88FF` (×4), `rh("carscrapewall")=0x811DE877` (×4), `rh("carhitsmackable")=0xA906E973`
> (×1) all appear in `GLOBAL/attributes.bin`. The tag strings themselves live as hashed keys, not literal strings
> in the exe.

## The tag is the hinge

The classification is *the* pivotal fact of a collision, because **everything downstream branches on it**
([C43.4](04-reactions.md)):

```
contact classified as <tag>
   ├──▶ Reaction:  pick the reaction tuning for <tag>   (CollisionReactionRecord, C43.4)
   ├──▶ Damage:    scale zone damage for <tag>          (Ch 45)
   └──▶ Presentation:
           ├─ Effect: the impact/scrape FX for <tag>    (sparks on wall, crunch on car)
           └─ Sound:  the carhit*/carscrape* sample     (Ch 22)
```

Hitting a wall (`carhitwall`) gives a hard stop, front-zone damage, sparks, and a wall-thud; hitting a car
(`carhitcar`) gives a softer exchange of momentum, mutual damage, and a metal-crunch; brushing a cone
(`carhitsmackable`) barely reacts, does no damage, and scatters the prop. Same collision *machinery*, different
*outcome*, selected by the tag. This is why every kind of crash in Most Wanted feels distinct — the tag routes the
one contact to the right reaction, damage, and presentation.

## Hit vs. scrape

The tags distinguish **discrete hits** from **sustained scrapes**:

- **`carhit*`** — a discrete impact, a single moment: the car strikes something and the collision resolves in that
  contact.
- **`carscrape*`** — a sustained grind: the car is *continuously* in contact (dragging along a wall,
  [C43.2](02-contact-records.md)), re-detected each frame.

The distinction matters for presentation: a hit gets a one-shot spark and thud; a scrape gets a *looping* scrape
effect and sound for as long as the grind lasts ([C43.2](02-contact-records.md)). The engine knows a scrape is
continuing because the same contact is re-found each frame ([C43.2](02-contact-records.md)), and the
`carscrape*` tag tells the presentation to loop rather than fire once. So the hit/scrape split is the discrete/continuous
distinction, encoded in the tag.

## Why classify in code, respond in data

The classification is computed in **code** (the contact update, [C43.1](01-detection.md)), but the *response* to
each tag is **data** ([C43.4](04-reactions.md)):

- **Detection must be exact.** *What* you hit and *where* is physics — it must be precise and fast, so it's fixed
  in code ([C43.1](01-detection.md)).
- **Response is taste.** *How dramatic* the reaction, *how much* damage, *which* effect — these are design choices,
  so they're exposed as vault records keyed by the tag ([C43.4](04-reactions.md)).

So the tag is the boundary between the code side (find and classify the contact) and the data side (respond to the
classification). A designer tunes crash feel by editing the reaction/damage/effect records for each tag; they
can't (and shouldn't) change *what counts as* hitting a wall — that's the physics. This clean split is why the
collision system is both reliable (code detection) and tunable (data response).

## RE implications

- **Every contact is classified** — `carhitwall`/`carhitcar`/`carhitsmackable`/`carscrapewall` (verified vault
  keys).
- **The tag is the hinge** — reaction, damage, and presentation all branch on it; different tags → different crash
  outcomes.
- **Hit vs. scrape** — `carhit*` (discrete, one-shot) vs. `carscrape*` (sustained, looping), the
  discrete/continuous split.
- **Classify in code, respond in data** — detection is exact physics; the per-tag response is tunable vault
  records.

---

### Key takeaways

- Every contact carries a **classification tag** — `carhitwall`, `carhitcar`, `carhitsmackable`, `carscrapewall`
  — **verified** as real vault keys (their hashes appear in `attributes.bin`).
- The tag is the **hinge**: reaction, damage, and presentation all branch on it, so every kind of crash feels
  distinct.
- **`carhit*` vs. `carscrape*`** encodes **discrete hit vs. sustained scrape** — the scrape loops its effect/sound
  because the same contact is re-detected each frame.
- **Classification is code** (exact, fast detection); **response is data** (per-tag reaction/damage/effect
  records) — the tunable side.
- The tag is the **boundary** between the collision physics (fixed) and the crash design (tunable).

**Continue:** [C43.4 — Reaction records](04-reactions.md) · [Chapter 43 hub](C43-Collision-Contacts.md)
