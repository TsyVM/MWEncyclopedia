# C32.3 — The Eleven Families

> **The one-sentence version:** every runtime class registers into one of eleven families — each a global
> list-head in `speed.exe` — and the family sizes (mechanics 51, AI actions 16, managers 14, …) show where the
> game's complexity lives.

[← C32.2 — The five class roles](02-five-roles.md) · [Chapter 32 hub](C32-Runtime-Class-System.md) ·
[Next: C32.4 — Registration: name → hash → list-head →](04-registration.md)

---

## The eleven lists

Classes register ([C32.4](04-registration.md)) onto one of **eleven global list-heads** — fixed addresses in
`speed.exe` whose references this book confirmed in the binary:

| # | Family | List-head | Classes |
|---|---|---|---:|
| 1 | Vehicle **mechanics** | `0x0092C660` | 51 |
| 2 | **AI actions** | `0x0090D8EC` | 16 |
| 3 | **Managers & activities** | `0x0092C668` | 14 |
| 4 | **AI goals** | `0x0090D8E8` | 14 |
| 5 | **Connectors** | `0x00988EC0` | 8 |
| 6 | **Director actions** | `0x009111FC` | 7 |
| 7 | **Named systems** | `0x00988DFC` | 5 |
| 8 | **World bodies** | `0x0092C66C` | 3 |
| 9 | **Players** | `0x00988EC4` | 3 |
| 10 | **World tasks** | `0x00988EBC` | 2 |
| 11 | **Devices** | `0x00920464` | 1 |

Each list-head anchors a linked list of the classes in that family; the runtime walks a family's list to
enumerate or construct its classes ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).

> ✅ *Verified:* the family list-head addresses are referenced throughout `speed.exe` (e.g. `0x0092C660`
> mechanics: 105 refs, `0x0090D8EC` AI actions: 37, `0x0090D8E8` AI goals: 33, `0x00988EC0` connectors: 21) —
> confirming they are live global registration anchors.

## The sizes tell the story

The family populations are a map of where the game's complexity concentrates:

- **Mechanics (51)** — by far the largest. The **vehicle simulation** is the game's most elaborate system: the
  eight mechanics ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)) plus their variants and
  supporting components. A racing game's heart is the car, and the class count reflects it.
- **AI (16 actions + 14 goals + 7 director = 37)** — the second concentration. The pursuit, racer, and traffic
  AI ([Chapters 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)–[49](../C49-Cops-Dispatch-Roadblocks/C49-Cops-Dispatch-Roadblocks.md))
  is a large, structured system.
- **Managers/systems (14 + 5 + 2 = 21)** — the coordinators that own populations and orchestrate gameplay.
- **The small families (connectors 8, world bodies 3, players 3, devices 1)** — the wiring and the few
  top-level entities.

So the class census says: this is a game about **cars** (mechanics) driven by **AI**, coordinated by
**managers** — exactly Most Wanted's shape.

## Why register by family

Grouping classes into family lists (rather than one big registry) serves the runtime:

- **Bulk operations by family.** The engine can walk *all mechanics* or *all AI goals* — e.g. to update every
  mechanic of a vehicle ([Chapter 40](../C40-Eight-Mechanics/C40-Eight-Mechanics.md)), or enumerate AI actions
  for a planner ([Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)).
- **Role-scoped construction.** Constructing "an AI goal named X" searches the AI-goals list, not everything
  ([Chapter 33](../C33-Class-Registry-Factories/C33-Class-Registry-Factories.md)).
- **Clear organisation.** The families mirror the subsystems, so the registry is legible.

The families are the registration structure; the roles ([C32.2](02-five-roles.md)) are the comprehension
grouping over them.

## Using the families to navigate the binary

The list-head addresses are navigation anchors in `speed.exe` ([C32.6](06-reading-binary.md)):

- **Find a family's classes** by finding registrations onto its list-head (e.g. code referencing `0x0092C660`
  registers a mechanic).
- **Identify a class's family** by which list-head its registration targets — which tells you its role.
- **Gauge a subsystem's size** by its family count — the mechanics list's 51 entries measure the vehicle sim's
  breadth.

So the eleven addresses are a table of contents for the runtime's classes.

## Editing/RE implications

- **The families are fixed** — eleven lists at fixed addresses; a class belongs to exactly one.
- **Family = role hint** — a class's family tells you what it broadly does ([C32.2](02-five-roles.md)).
- **Count = complexity** — a large family is a large subsystem; start RE there for impact (mechanics, AI).
- **List-heads are RE anchors** — reference them to find registrations ([C32.6](06-reading-binary.md)).

---

### Key takeaways

- Classes register into **eleven families**, each a global list-head in `speed.exe` (addresses verified by their
  many references).
- **Mechanics (51)** dominate — the vehicle sim is the largest system; **AI (37 across goals/actions/director)**
  is second.
- The class census maps the game: cars (mechanics), driven by AI, coordinated by managers.
- Families enable bulk operations, role-scoped construction, and legible organisation.
- The eleven list-head addresses are a **table of contents** for the runtime classes and RE anchors.

**Continue:** [C32.4 — Registration: name → hash → list-head](04-registration.md) · [Chapter 32 hub](C32-Runtime-Class-System.md)
