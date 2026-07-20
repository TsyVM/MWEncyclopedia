# C65.6 — Scoreboards

> **The one-sentence version:** the scoreboards — leaderboard, pursuit board, milestone board, race information —
> are FEng *list widgets* populated through a `COLUMN%d_DATA`/`LINE%d_GROUP` binding, showing tabular game state
> (positions, bounty, milestones), and the post-event result screens are their full-screen form.

[← C65.5 — Gauges & meters](05-gauges-meters.md) · [Chapter 65 hub](C65-HUD-Runtime.md) ·
[Next: C65.7 — Rendering & update →](07-rendering-update.md)

---

## The board widgets

Beyond the continuous gauges ([C65.5](05-gauges-meters.md)), the HUD shows *tabular* state — the **scoreboards**,
verified widgets:

| Board | Shows |
|---|---|
| `Hud_LeaderBoard` / `LeaderBoard` | race position — you vs. the AI racers, ranked |
| `PursuitBoard` | the pursuit tally — cops engaged, pursuit stats ([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)) |
| `Hud_MilestoneBoard` | milestone progress ([C54.4](../C54-GameFlow-Blacklist/04-bounty-milestones.md)) |
| `RaceInformation` | position / lap / checkpoint ([C55.1](../C55-Race-Events/01-race-flow.md)) |
| `Hud_Infractions` / `Infractions` | pursuit infractions (what you're wanted for) |
| `Hud_CostToState` / `InGameBounty` | bounty / cost-to-state accumulating |

So the scoreboards are the HUD's *ranked/listed* information — where do I place, how much bounty, which milestones,
what infractions. Unlike the gauges (single values), these are *lists* of entries (racers, cops, milestones),
which need a different widget kind ([below](#the-list-widget-binding)).

> ✅ *Verified:* `Hud_LeaderBoard`, `LeaderBoard` (×3), `PursuitBoard`, `Hud_MilestoneBoard`, `RaceInformation`
> (×2), `Hud_Infractions`/`Infractions`, `Hud_CostToState`, `InGameBounty`, `InGameRaceSheet` are present in
> `speed.exe`.

## The list-widget binding

Scoreboards are **list widgets** — and FEng binds them through a *column/row* idiom, verified in the exe:

- **`COLUMN1_DATA`, `COLUMN2_DATA`, `COLUMN3_DATA`** — the *columns* of a list widget (e.g. rank / name / time).
- **`LINE%d_GROUP`** — the *rows* — a repeated FEObject group per line, indexed `LINE0_GROUP`, `LINE1_GROUP`, … (a
  `%d`-templated group name).

So a leaderboard is a *template*: a set of `LINE%d_GROUP` row groups, each with `COLUMN%d_DATA` cells, populated
from the ranked data (racer positions). The runtime fills row *N* with entry *N*'s columns (rank, name, time) —
the classic data-bound table. This `COLUMN`/`LINE%d` binding is the FEng list mechanism
([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)), verified present — it's how *every* MW05 list
(leaderboard, the car-select list, the Blacklist screen) is built: a templated row group, filled per entry.

> ✅ *Verified:* `COLUMN1_DATA`/`COLUMN2_DATA`/`COLUMN3_DATA` and `LINE%d_GROUP` are present in `speed.exe` — the
> list-widget column/row binding.

## Why a templated list

Building scoreboards as *templated lists* (`LINE%d_GROUP` × `COLUMN%d_DATA`) rather than hand-placed entries is
clean and flexible:

- **Variable length.** A race has 4 or 8 racers; a leaderboard just instantiates that many `LINE%d_GROUP` rows.
  The template adapts to the entry count.
- **Data-bound.** The rows are *filled* from the game data (positions, [Chapter 55](../C55-Race-Events/C55-Race-Events.md))
  — the widget doesn't hard-code entries; it binds to the current ranking. As you pass a rival, the leaderboard
  re-sorts and re-fills.
- **Reusable idiom.** The *same* `COLUMN`/`LINE%d` mechanism serves the leaderboard, the pursuit board, the
  milestone board, and the menu lists ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)) — one list
  widget, many uses.

So the scoreboards are *data-bound tables* — a row template filled from live game state
([Chapter 54](../C54-GameFlow-Blacklist/C54-GameFlow-Blacklist.md)–[55](../C55-Race-Events/C55-Race-Events.md)). This
is why the leaderboard updates live (re-fill on rank change) and handles any field size (instantiate N rows). It's
the tabular counterpart to the gauge cluster's ([C65.5](05-gauges-meters.md)) single-value binding — both bind to
game state, one as a number (speed), one as a table (rankings).

## The post-event scoreboards

The scoreboards have a *full-screen* form — the **post-event result screens** ([C55.1](../C55-Race-Events/01-race-flow.md)):

- **`PostRace_Results` / `PostRaceResultsScreen`** — the race result (won/lost, position, time,
  [C55.1](../C55-Race-Events/01-race-flow.md)).
- **`PostRace_Pursuit` / `PostRacePursuitScreen`** — the pursuit rap-sheet (bounty, infractions,
  [Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)); `PostPursuitInfractionsScreen` details the charges.
- **`PostRace_MilestoneRewards` / `PostRaceMilestonesScreen`** — milestone rewards
  ([C54.4](../C54-GameFlow-Blacklist/04-bounty-milestones.md)).
- **`PostBusted`** — the busted screen ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)).

So when an event ends ([C55.1](../C55-Race-Events/01-race-flow.md)), the flow ([C65.7](07-rendering-update.md))
selects the right result screen — a full-screen scoreboard summarising the event (your placement, your bounty, your
milestones). These are the *same* list/data-bound widgets ([above](#the-list-widget-binding)) as the in-race
boards, presented full-screen at the event's end — the in-race leaderboard becomes the post-race results table. The
result screen is the scoreboard's *conclusion*: the in-race board tracks the *live* standings, the post-event screen
shows the *final* ones. Career glue writes the data for these (`SetRaceCompleteForFE`, `StorePursuitRepForFE`) so
the FE can display the outcome.

## RE implications

- **Scoreboards** — leaderboard, pursuit board, milestone board, race information — are the HUD's *tabular* widgets.
- **List binding** — `COLUMN%d_DATA` (columns) × `LINE%d_GROUP` (templated rows) — data-bound tables filled from
  game state.
- **Templated lists** — variable length, data-bound, reusable (the same idiom as menu lists).
- **Post-event screens** — `PostRace_Results`/`Pursuit`/`MilestoneRewards`, `PostBusted` — the full-screen,
  final-standings form.

---

### Key takeaways

- The **scoreboards** — `Hud_LeaderBoard`, `PursuitBoard`, `Hud_MilestoneBoard`, `RaceInformation`, `Infractions`,
  `CostToState`/`InGameBounty` — are the HUD's **tabular** widgets (ranked/listed state).
- They're **list widgets** bound through **`COLUMN%d_DATA`** (columns) × **`LINE%d_GROUP`** (templated rows) — a
  data-bound table filled from live game state (positions, bounty, milestones).
- The **templated list** adapts to entry count (4 or 8 racers), re-fills on change (live re-sort), and is the
  **same idiom** as the menu lists ([Chapter 27](../C27-FrontEnd-Shell-UI/C27-FrontEnd-Shell-UI.md)).
- The scoreboards have a **full-screen post-event form** — `PostRace_Results`/`Pursuit`/`MilestoneRewards`,
  `PostBusted` — showing the **final** standings (career glue: `SetRaceCompleteForFE`/`StorePursuitRepForFE`).
- Scoreboards are the **tabular counterpart** to the gauges' single-value binding ([C65.5](05-gauges-meters.md)) —
  both bind to game state, one as a number, one as a table.

**Continue:** [C65.7 — Rendering & update](07-rendering-update.md) · [Chapter 65 hub](C65-HUD-Runtime.md)
