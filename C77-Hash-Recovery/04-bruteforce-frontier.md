# C77.4 — Brute-Forcing & the Frontier

> **The one-sentence version:** for hashes no dictionary resolves, brute force exhaustively generates and hashes
> candidate strings — decisive for *short* names but exploding exponentially with length, so a frontier of
> unresolved hashes (like `0x0743CFB1`) always remains, and mapping it honestly is the chapter's close.

[← C77.3 — Building a name dictionary](03-name-dictionary.md) · [Chapter 77 hub](C77-Hash-Recovery.md) ·
[Next: C77.5 — The name dictionary unlocks the book →](05-dictionary-unlocks.md)

---

## Brute force: when the dictionary fails

When a hash isn't in the dictionary ([C77.3](03-name-dictionary.md)) — the name never followed a known convention,
never appeared as a string, isn't in any wordlist — the last resort is **brute force**: generate *every* possible
string (up to some length, over some character set) and hash each, looking for the target.

```
for length in 1..N:
    for every string of that length over the charset:
        if hash(string) == target: candidate found
```

Brute force needs *no* prior knowledge — it will *eventually* find any name, because it tries them all. Its power is
completeness; its problem is *cost*, and the cost is where the frontier lives ([below](#the-combinatorial-wall)).

## The combinatorial wall

Brute force runs headlong into **exponential growth**: the number of strings of length *L* over a charset of size *C*
is *Cᴸ*, which explodes:

```
charset 26 (a–z):
  length 6:   26⁶  ≈ 300 million     → seconds
  length 7:   26⁷  ≈ 8 billion       → minutes
  length 8:   26⁸  ≈ 200 billion     → hours
  length 10:  26¹⁰ ≈ 1.4 × 10¹⁴      → months
  length 12:  26¹² ≈ 10¹⁷            → infeasible
```

Add uppercase, digits, and `_` (charset ~64) and it explodes far faster. So brute force is **decisive for short
names** (a 6–7 character identifier falls in minutes) and **hopeless for long ones** (a 12-character name like
`TOPSPEED_MAX` is out of reach by pure enumeration). This is the *combinatorial wall*: recovery by brute force is easy
below a length threshold and impossible above it, with the threshold set by your compute budget and the charset.

Because MW's *convention-following* names are often *long* (`PART_EN_COLD_AIR_INTAKE_SYSTEM` is 30 characters), pure
brute force would *never* find them — which is exactly why the **dictionary and conventions**
([C77.3](03-name-dictionary.md)) matter: they recover the long, systematic names that brute force can't, leaving brute
force for the *short, arbitrary* ones a dictionary misses. The two are complementary: dictionary for the long-and-
structured, brute force for the short-and-random.

## Smart brute force

Pure enumeration is rarely the right tool; **guided** brute force pushes the wall back:

- **Masks** — if you know the *shape* (e.g. `PART_??_STAGE_?`), fix the known characters and brute-force only the
  unknowns — collapsing the space by orders of magnitude.
- **Mutations** — take dictionary words ([C77.3](03-name-dictionary.md)) and apply transformations (append digits,
  swap `_`/casing, add known prefixes/suffixes) — catching *near-convention* names cheaply.
- **Domain charsets** — MW identifiers use `[A-Z0-9_]`, not arbitrary bytes, so restrict the charset accordingly;
  and try `UPPER_SNAKE_CASE` structure over random strings.

Guided brute force is really *dictionary thinking extended* ([C77.3](03-name-dictionary.md)): instead of "all strings,"
it's "all strings *that look like MW names*," which is a vastly smaller, higher-yield space. The unguided
all-strings-of-length-*L* brute force is the fallback of last resort; in practice, masks and mutations off the
dictionary recover far more per CPU-second.

## Collisions demand plausibility

A special hazard of brute force ([C77.1](01-recovery-problem.md)): a match is *especially* likely to be a **collision**
([C2.5](../C2-Identifiers-And-Hashing/05-collisions-and-renaming.md)), not the real name. When you enumerate billions
of random strings, *some* will hash to your target *by chance* — a meaningless `xqzptfk` that happens to collide. So a
brute-force match is only credible if it's **plausible**: a real word, a convention-following identifier, something a
developer would actually name a thing. An implausible match is almost certainly a collision, not a recovery. This is
why *guided* brute force ([above](#smart-brute-force)) is not just faster but *more trustworthy* — it only generates
plausible candidates, so its matches are credible, whereas unguided enumeration's matches must be filtered hard for
plausibility ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)).

## The honest frontier

Putting it together, the recovery frontier is *precisely* mappable — the chapter's closing discipline
([Chapter 76](../C76-Advanced-RE/C76-Advanced-RE.md)):

- **Recoverable** — names that are in the exe's strings, follow a known convention, are in a wordlist, or are short
  enough to brute-force. The large majority of MW's identifiers.
- **The frontier (⚪)** — hashes like `0x0743CFB1` whose names are *long*, *arbitrary*, and *absent* from every source:
  not in the strings, not convention-following, not short. These resist both dictionary and brute force.

The frontier isn't a failure — it's a *precisely-bounded unknown*: "this hash's name is not in any current source and
is too long to brute-force; it will fall only to a larger dictionary (a new string source, a discovered convention) or
a lucky wordlist hit." Stating *that*, per unresolved hash, is the honest output ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md))
— and it's *actionable*, because it tells the next investigator exactly what would close each gap: a name they might
find, a convention they might spot, a source not yet harvested.

## RE implications

- **Brute force** — exhaustively hash all strings up to a length; needs no knowledge but costs *Cᴸ*.
- **The combinatorial wall** — decisive for short names (≤7–8 chars), infeasible for long ones; MW's long
  convention-names need the dictionary instead.
- **Guided brute force** — masks, mutations, domain charsets — "strings that look like MW names," far higher yield.
- **Plausibility** — a brute-force match is credible only if plausible; implausible matches are collisions
  ([C2.5](../C2-Identifiers-And-Hashing/05-collisions-and-renaming.md)).

---

### Key takeaways

- **Brute force** exhaustively generates and hashes strings — needing **no prior knowledge** — but hits a
  **combinatorial wall**: *Cᴸ* strings, so it's **decisive for short names** (≤7–8 chars, minutes) and **infeasible
  for long ones** (12+ chars).
- MW's **convention names are long**, so pure brute force can't find them — which is why the **dictionary/conventions**
  ([C77.3](03-name-dictionary.md)) carry the long-and-structured names, leaving brute force for the **short-and-
  arbitrary** ones.
- **Guided** brute force — **masks** (fix known chars), **mutations** (off dictionary words), **domain charsets**
  (`[A-Z0-9_]`) — is "strings that look like MW names," vastly higher-yield than unguided enumeration.
- Brute-force matches **demand plausibility** — random enumeration produces **collisions**
  ([C2.5](../C2-Identifiers-And-Hashing/05-collisions-and-renaming.md)); an implausible match is almost certainly *not*
  the name.
- The **frontier** — hashes like `0x0743CFB1` that are long, arbitrary, and absent from every source — is a
  **precisely-bounded unknown** (⚪): stated per-hash, it tells the next investigator **exactly what would close each
  gap**.

**Continue:** [C77.5 — The name dictionary unlocks the book](05-dictionary-unlocks.md) · [Chapter 77 hub](C77-Hash-Recovery.md)
