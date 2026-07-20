# C76.5 — The Advanced-RE Method

> **The one-sentence version:** the advanced-RE loop is classify → hypothesise → decode → validate → tier → record,
> run with honest confidence tiers and a documented frontier — the techniques that produce the evidence Chapter 50's
> method demands, shown whole on the vault, the book's hardest target.

[← C76.4 — Building & validating readers](04-building-readers.md) · [Chapter 76 hub](C76-Advanced-RE.md) ·
[Book index →](../README.md)

---

## The loop

Every hard decoding in the book runs the same loop — the synthesis of this chapter:

```
1. CLASSIFY   what kind of data is this?         (block markers, stride, strings — C76.1)
2. HYPOTHESISE a structure/schema                 (the class table, the key hash — C76.2)
3. DECODE     static and/or dynamic               (disassemble, or diff a known change — C76.3)
4. VALIDATE   round-trip + statistics             (rebuild-complete, 66.8% vs 0.2% — C76.4)
5. TIER       ✅ verified / 🟡 reasoned / ⚪ open   (honest confidence — C50.1)
6. RECORD     including the dead-ends              (false negatives, wrong leads — C76.4)
```

It's a *loop*, not a line: a failed validation (step 4) sends you back to hypothesise (step 2); a dead-end (step 6)
narrows the next hypothesis. And it's *honest at every step* — the tiering (step 5) and the dead-end record (step 6)
are not epilogue but *part of the work*. This is the method that produced the book's finished chapters; C76 just makes
it explicit.

## Companion to Chapter 50

This chapter and the verification methodology ([Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md))
are two halves of the book's *epistemology*:

- **[Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)** — the *standard*: the confidence
  tiers (✅/🟡/⚪), byte verification, hash verification. *What counts as knowing.*
- **This chapter** — the *practice*: classify, hypothesise, decode, validate. *How you come to know.*

So C50 defines the bar and C76 shows how to clear it. The round-trip that C50 makes the test of understanding
([C50.2](../C50-Verification-Methodology/02-byte-verification.md)) is the same round-trip C76 uses to validate a
reader ([C76.4](04-building-readers.md)); the tiers C50 assigns are the tiers C76's techniques *earn*. Together
they're the reason the book can claim to be *verification-first*: a standard for knowledge, and a repeatable method
for reaching it — applied even to the hardest targets, honestly.

## The vault as exemplar

The attribute vault ([Chapters 11–14](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)) is this chapter's exemplar
*because* it was XL — a target where the method is tested to its limit:

- **Classified** — string table + record array, from the block markers ([C76.1](01-identifying-data.md)).
- **Hypothesised** — code-driven registration, keys as inline name hashes ([C76.2](02-recovering-schema.md)).
- **Decoded** — statically (the class table, the registration idiom) and via the key-hash breakthrough
  (`lookup2`/`0xABCDEF00`), with dynamic diff the practical path for field offsets ([C76.3](03-static-vs-dynamic.md)).
- **Validated** — 66.8% of keys resolving vs. <0.2% noise ([C76.4](04-building-readers.md)).
- **Tiered honestly** — the block map, class table, registration mechanism, and key hash are ✅ *verified*; the full
  static field map remains *open* ([below](#the-honest-frontier)).

So the vault shows the method *working* (most of it cracked, verified) *and* its *limits* (the last mile still open) —
which is exactly why it's the right exemplar. A method is only proven by how it handles the *hard* case, not the easy
ones.

## The honest frontier

The most important output of advanced RE is often a *clearly-drawn frontier* — what's known, and what's precisely
left:

- **Known (✅)** — the vault's block map, the 28-class table, the code-driven registration, the `lookup2` key hash,
  the `{field, value, type}` value model; **66.8%** of keys resolved.
- **Open (⚪)** — the remaining ~33% of keys (names not yet in the dictionary, [C77](../C77-Hash-Recovery/C77-Hash-Recovery.md)),
  and the *full static field-offset map* (the consumer of the class-registration list, not yet traced,
  [C76.2](02-recovering-schema.md)).

Drawing this frontier is not an admission of failure — it's the *deliverable*. "66.8% resolved, here's the exact
33% left and the two concrete ways to reach it (extend the name dictionary, [C77](../C77-Hash-Recovery/C77-Hash-Recovery.md);
or trace the registration-list consumer)" is *far* more useful than a vague "mostly decoded" or a fabricated
"complete." The honest frontier tells the next investigator *exactly where to start* — which is the whole point of
recording the work ([C76.4](04-building-readers.md)). An RE effort's value is measured not just by what it closed but
by how precisely it mapped what remains.

## RE implications

- **The loop** — classify → hypothesise → decode → validate → tier → record — honest at every step, iterated.
- **Companion to C50** — C50 is the *standard* (what counts as knowing), C76 the *practice* (how you come to know).
- **The vault exemplar** — the method working (cracked, verified) and its limits (the last mile open) — proven on the
  hard case.
- **The honest frontier** — known vs. precisely-what's-left is the deliverable; it tells the next investigator where
  to start.

---

### Key takeaways

- Advanced RE is a **loop** — **classify → hypothesise → decode → validate → tier → record** — iterated, and **honest
  at every step** (the tiering and dead-end record are *part of the work*, not epilogue).
- It's the **practice** to [Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md)'s **standard**
  — C50 defines *what counts as knowing* (the tiers, the round-trip), C76 shows *how you come to know* — together, the
  book's **verification-first** epistemology.
- The **vault** is the exemplar *because* it was **XL** — classified, hypothesised, decoded (static + the `lookup2`
  key breakthrough), validated (**66.8%** vs. **0.2%**), and **tiered honestly** — showing the method's **power and
  its limits**.
- The key deliverable is often the **honest frontier** — **known (✅)** vs. **precisely what's open (⚪)** — e.g.
  "66.8% resolved; the last ~33% needs an extended name dictionary
  ([Chapter 77](../C77-Hash-Recovery/C77-Hash-Recovery.md)) or the untraced registration-list consumer" — which tells
  the next investigator **exactly where to start**.
- Advanced RE's value is measured by **both** what it closed **and** how precisely it mapped what remains.

**Next:** [Chapter 77 — Hash Recovery & Name Dictionaries](../C77-Hash-Recovery/C77-Hash-Recovery.md).

**Sources:** independently verified against the retail binaries — `GLOBAL/attributes.bin` block map (`ErtS`@`0x80`,
`NpeD`@`0x55C00`, `NrtS`@`0x55C30`, `NtaD`@`0x55C40`; 4,732 `0xEFFECADD` records); the 28-name class table @ `speed.exe`
file offset `0x4ADD1C`; the `lookup2` seed `0xABCDEF00` (×50 in the exe) under which ~66.8% of vault keys resolve
([Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md)). Method: [Chapter 50](../C50-Verification-Methodology/C50-Verification-Methodology.md).
Hash recovery of the open keys: [Chapter 77](../C77-Hash-Recovery/C77-Hash-Recovery.md).
