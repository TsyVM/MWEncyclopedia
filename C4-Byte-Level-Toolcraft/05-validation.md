# C4.5 — Validation Harnesses

> **The one-sentence version:** a decode isn't done when it looks right — it's done when a harness proves
> it, by round-tripping every file, reproducing a known value, or agreeing with the code, and it earns a
> confidence marker that says exactly how well you know it.

[← C4.4 — Static analysis](04-static-analysis.md) · [Chapter 4 hub](C4-Byte-Level-Toolcraft.md) ·
[Next: C4.6 — Batch reconnaissance →](06-batch-recon.md)

---

## Why harnesses, not eyeballs

Human pattern-matching is good at *forming* hypotheses and terrible at *rejecting* them — we see the
records that fit and skip the one that doesn't. A validation harness is the antidote: it applies your
hypothesis to *all* the data mechanically and fails loudly on the first counterexample. Every ✅ in this
book stands on a harness, not on "it looked right."

There are three harness archetypes, in rough order of strength.

## 1 — The round trip

Parse a file into a structured model, serialise the model back to bytes, and assert the result is
identical to the input. If a *no-op* round trip isn't byte-perfect, your parser or serialiser
misunderstands the format, and the first differing offset points at the misunderstanding
([C1.11](../C1-EAGL-Container-Model/11-failure-modes.md)).

```python
def roundtrip_ok(path, parse, serialise):
    orig = open(path, 'rb').read()
    rebuilt = serialise(parse(orig))
    if orig == rebuilt: return True, None
    i = next(k for k in range(min(len(orig), len(rebuilt))) if orig[k] != rebuilt[k])
    return False, i        # first divergence localises the bug
```

Run it across an entire directory, not one file. A parser that round-trips every retail TPK is a parser
you can trust to *edit* one — because "reproduces the original exactly" means "understands every byte,
including the ones you didn't think mattered."

## 2 — The known-value reproduction

When a format embeds a value you can compute independently, reproduce it. The archetype is
`lookup2("default") == 0xEEC2271A` ([C2.2](../C2-Identifiers-And-Hashing/02-reflection-hash.md)): a value
you computed, a value the compiler emitted, and a value the data ships with, all agreeing. Similarly, JDLZ
is validated by `len(decompress(x)) == decompSize(x)` across all four bundles
([C3.4](../C3-Compression-JDLZ/04-decompressor.md)) — the format hands you the expected size, and you
match it. A known-value test is powerful because the "answer" comes from a *different source* than your
code, so agreement is real corroboration, not self-consistency.

## 3 — The cross-source agreement

The strongest facts are where two *independent* methods converge: a data-derived stride
([C4.2](02-decoding-unknowns.md)) equals a code-derived loop increment
([C4.4](04-static-analysis.md)); a hex-diff'd field offset ([C4.3](03-hex-diffing.md)) matches a struct
access in the reader routine. When black-box and white-box agree, the fact is as solid as it gets — mark
it ✅ without reservation.

## Confidence discipline

A harness tells you not just *whether* you're right but *how confidently* to state it. Map the harness
result to the marker:

- ✅ **Verified** — a round trip passes across the data set, a known value reproduces, or code and data
  agree. State it as fact and give the evidence (offset, address, or measurement).
- 🟡 **Reasoned** — the *layout* checks out but a claim about *intent or meaning* rests on inference ("this
  padding is for GPU alignment"). Say so; the mechanism is proven, the motive is argued.
- ⏳ **Open** — you know a structure exists but a field's meaning or a sub-layout isn't decoded. Mark the
  edge honestly rather than papering it with a guess. An explicit ⏳ is more useful than a confident wrong
  answer, because it tells the next person exactly where to dig.

This discipline is what makes the encyclopedia *trustworthy*: a reader can act on a ✅ and knows to
double-check a 🟡. Dropping the markers to sound authoritative would be the worst thing you could do —
it converts calibrated knowledge into indistinguishable assertion.

## Regression: harnesses are permanent

A harness is not a one-time gate; it is a regression test. Keep them and re-run them whenever you change a
parser. When you promote a parser into `mw` ([C4.1](01-core-library.md)), its harness comes with it. The
result is a workshop where "the whole data set still round-trips" is a green light you can re-check in
seconds — so a refactor that quietly breaks the geometry reader is caught by the same run that once proved
it correct.

## Bending it — validate like you mean it

- **Run across the corpus, never one file.** The counterexample is the point; a single passing file proves
  almost nothing.
- **Prefer independent-source checks.** A round trip proves self-consistency; a known-value or
  code-agreement check proves correctness. Reach for the latter when you can.
- **Localise failures.** A harness that says "different" is half a harness; one that says "first diff at
  0x4A20" points you at the fix.
- **Never launder a 🟡 into a ✅.** The markers are the product's integrity. Keep meaning-claims and
  layout-claims distinct.

---

**Continue:** [C4.6 — Batch reconnaissance](06-batch-recon.md) · [Chapter 4 hub](C4-Byte-Level-Toolcraft.md)
