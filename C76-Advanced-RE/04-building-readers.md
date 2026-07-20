# C76.4 — Building & Validating Readers

> **The one-sentence version:** a decoding is only as good as its validation — a reader earns trust by *round-tripping*
> (rebuild the bytes it read) and by holding *statistically* (the vault keys resolving 66.8% under `lookup2` vs. <0.2%
> noise), and recording the dead-ends and false negatives is part of the method, not a footnote.

[← C76.3 — Static vs. dynamic recovery](03-static-vs-dynamic.md) · [Chapter 76 hub](C76-Advanced-RE.md) ·
[Next: C76.5 — The advanced-RE method →](05-advanced-method.md)

---

## From hypothesis to reader

A schema hypothesis ([C76.2](02-recovering-schema.md)) becomes knowledge only when it's *built into a reader and
tested*. The loop is incremental:

1. **Hypothesise** a structure — "records are 12–392 B, keyed by a 4-byte hash at offset 0, then `{field, value,
   type}` triples."
2. **Build a reader** that parses it — turning the hypothesis into code that *consumes every byte*.
3. **Run it on the real data** — does it parse *all* records without running off the end or hitting garbage?
4. **Validate** — round-trip and statistics ([below](#the-two-validations)).
5. **Refine** — where it fails, the hypothesis is wrong; fix and repeat.

The key discipline is **consume every byte**: a reader that parses *most* of a file but skips "the parts it doesn't
understand" is hiding its own gaps. A reader that accounts for *every* byte — even as "preserved, undecoded"
([C63.8](../C63-Collision-World/08-wcollisionpack.md)) — is one whose understanding is *complete enough to rebuild*,
which is the bar ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).

## The two validations

A reader is trustworthy when it passes *two independent* checks:

- **Round-trip** ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)) — parse the data and rebuild it;
  the output must be **byte-for-byte identical** to the input. This proves the reader's model is *complete* — it
  captured everything, because it can reproduce everything ([C75.4](../C75-Modding-Workflow/04-verify-test.md)). A
  reader that round-trips *understands the format's structure*, whatever the fields mean.
- **Statistical** — where a hypothesis predicts a *pattern*, measure how often it *holds* against how often it would
  hold by chance. The vault keys are the model: under the *right* hash (`lookup2`/`0xABCDEF00`,
  [C76.2](02-recovering-schema.md)) **66.8%** of keys resolve to names; under the *wrong* hash (Joaat/Bin) only
  **<0.2%**. A 66.8%-vs-0.2% gap is *decisive* — 66.8% is far above noise, so the hypothesis is *right*; 0.2% is
  noise, so it's *wrong*.

The two catch different errors: round-trip catches *structural* mistakes (a wrong size, a missed field), statistics
catch *interpretive* mistakes (the wrong hash, the wrong meaning). A reader that round-trips *and* whose
interpretations hold statistically is *verified* in both senses — structure and meaning.

> ✅ *Verified:* the vault keys resolve at ~66.8% under `lookup2`/`0xABCDEF00` vs. <0.2% under Joaat/Bin — the
> statistical gap that confirmed the key hash ([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)).

## Statistics beat intuition

The vault's key hash is the chapter's sharpest lesson: **let the numbers judge the hypothesis, not your intuition**.
An early, careful analysis tried the two *expected* hashes (Joaat, Bin — the ones the engine uses elsewhere,
[Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)), got <0.2%, and concluded *"the keys aren't
inline name hashes"* — a **false negative**. The keys *were* inline name hashes; the analysis had just tried the
*wrong two hashes*. Only when a *third* hash (`lookup2`) was tried did 66.8% resolve.

The lesson isn't "that analysis was sloppy" — it was careful. It's that **a negative result is only as broad as the
hypotheses you tested**: "not Joaat, not Bin" is *not* "not a hash." The fix is to *state negatives narrowly* ("keys
don't match Joaat or Bin") rather than broadly ("keys aren't hashed"), so the next person knows *exactly what's left
to try* ([C77](../C77-Hash-Recovery/C77-Hash-Recovery.md)). Statistics are ruthless and honest — 66.8% vs. 0.2%
settles it — but only over the hypotheses you actually test.

## Record the dead-ends

The final discipline: **write down what *didn't* work**. A dead-end recorded is a dead-end the next person doesn't
re-walk:

- **The false negative** — "keys don't match Joaat/Bin" ([above](#statistics-beat-intuition)) saved re-testing those,
  and *pointed at* trying other hashes ([C77](../C77-Hash-Recovery/C77-Hash-Recovery.md)).
- **The static dead-end** — the `TOPSPEED`/`ACCEL`/`HANDLING` "registrar" that was a UI widget
  ([C76.3](03-static-vs-dynamic.md), [Chapter 69](../C69-Performance-Upgrades-Tuning/C69-Performance-Upgrades-Tuning.md)),
  not the vault schema — saved re-investigating a plausible-but-wrong lead.

Recording dead-ends is *positive* work: it narrows the search space for everyone after. This is why the book tiers
honestly ([C50.1](../C50-Verification-Methodology/01-confidence-tiers.md)) and states open problems plainly — a
documented "this doesn't work, here's what's left" is as valuable as a documented "this works," because both *move
the frontier*. An RE effort that only records successes hides half of what it learned.

## RE implications

- **Hypothesis → reader → test** — build a reader that *consumes every byte*; incompleteness hides gaps.
- **Two validations** — round-trip (structure: rebuilds identically) + statistical (meaning: resolves far above
  noise).
- **Statistics beat intuition** — 66.8% vs. 0.2% settles the key hash; a negative is only as broad as the hypotheses
  tested.
- **Record dead-ends** — false negatives and wrong leads, stated narrowly, narrow the search for everyone after.

---

### Key takeaways

- Turn a schema hypothesis into a **reader that consumes every byte** (even as "preserved, undecoded") — a reader that
  skips what it doesn't understand **hides its gaps**; the bar is *rebuild-complete*
  ([C50.2](../C50-Verification-Methodology/02-byte-verification.md)).
- Validate **two ways**: **round-trip** (proves the *structure* — rebuilds byte-for-byte) and **statistical** (proves
  the *meaning* — e.g. **66.8%** of vault keys resolve under `lookup2` vs. **<0.2%** noise); they catch different
  errors.
- **Let statistics judge, not intuition** — the vault's key hash was missed as a **false negative** (tried only
  Joaat/Bin → <0.2%) until `lookup2` gave 66.8%; **state negatives narrowly** ("not Joaat/Bin", not "not a hash").
- **Record the dead-ends** — the false negative and the `TOPSPEED`/`HANDLING` UI-widget lead
  ([C76.3](03-static-vs-dynamic.md)) — a documented dead-end **narrows the search** for everyone after; recording
  failures **moves the frontier** as much as recording successes.

**Continue:** [C76.5 — The advanced-RE method](05-advanced-method.md) · [Chapter 76 hub](C76-Advanced-RE.md)
