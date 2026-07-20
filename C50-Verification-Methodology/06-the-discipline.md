# C50.6 — The Discipline & Why It Matters

> **The one-sentence version:** verification-first RE — every claim reduced to a check, every uncertainty marked
> honestly — is what makes this book a textbook rather than lore: reproducible, falsifiable, correct where it can
> be, and openly uncertain where it can't.

[← C50.5 — Cross-checking & correcting received wisdom](05-cross-checking.md) · [Chapter 50 hub](C50-Verification-Methodology.md) ·
[Next: Chapter 51 — The Render Pipeline →](../C51-Render-Pipeline/C51-Render-Pipeline.md)

---

## Every claim reduces to a check

The unifying principle of the whole book is that **every verified claim reduces to a concrete, reproducible check**
against the shipped files:

- **"Is `0x5C5E80` a square root?"** → read the bytes: `fld; fsqrt; ret` ([C50.2](02-byte-verification.md)). ✅
- **"Is `carhitwall` a real tag?"** → hash it, search the vault: found ×4 ([C50.3](03-hash-verification.md)). ✅
- **"Is `AIVehicle` the biggest class?"** → count the vtable: 351 methods ([C50.4](04-vtable-verification.md)). ✅
- **"What's the bust radius?"** → read the float: 15.0 / 90.0 ([C48.4](../C48-Pursuit-Heat/04-bust-evade.md)). ✅

No claim rests on authority ("a well-known modder said…") or on the book's own say-so. Each ✅ names its check, and
the check can be re-run by anyone with `speed.exe` and `attributes.bin`. This is the property that makes the book
*scientific*: its factual claims are **falsifiable** — you could, in principle, run the check and prove one wrong.
That none have been is because each was run before it was marked.

## Honest uncertainty

Equally important is what the book *doesn't* claim. The 🟡 and ⚪ marks ([C50.1](01-confidence-tiers.md)) are
promises of **honest uncertainty**:

- Where the bytes stop, the book says so — a function's prologue is ✅, its detailed internal algorithm is 🟡.
- Where a claim is inference, it's flagged — "the standard X model, consistent with the verified anchors" is a 🟡,
  not a stated fact.
- Where something is genuinely unknown, it's left open — no confident-sounding filler over a gap.

This restraint is a *feature*. A reference that claims certainty it doesn't have is dangerous — you build on it and
it fails. One that marks its uncertainty lets you know exactly where the solid ground ends. The book would rather
say "🟡 Reasoned" and be trusted than assert and be wrong. Honest uncertainty is the other half of verification:
verify what you can, and *mark clearly* what you can't.

## Why this matters for a developer

The book's audience is developers learning MW05's engine, and verification-first is what makes it *useful* to them:

- **You can act on ✅ facts.** An address, a hash, an offset marked ✅ can be used in a tool, a mod, or an analysis
  with confidence — it was checked.
- **You know when to check further.** A 🟡 tells you "this is the likely picture; verify the detail before you
  depend on it" — saving you from building on an inference as if it were fact.
- **You learn the *method*, not just the facts.** By seeing *how* each fact was verified, you learn to verify your
  own findings ([C50.2](02-byte-verification.md)–[C50.4](04-vtable-verification.md)) — the book teaches the
  *practice* of RE, not just its results.

So the discipline isn't academic — it's what lets a developer *use* the book safely and *extend* it correctly. A
modder who reads that `EngineRacer` is `0xB2809518` (✅) can key the vault directly; one who reads a 🟡 about a
tyre-force formula knows to confirm it before tuning. The marks are operational guidance.

## RE as a practice

Beyond this game, the book models **reverse-engineering as a practice** — a repeatable, honest method applicable to
any binary:

1. **Anchor on the shipped artifacts** — the executable and its data are the ground truth; everything else is
   hypothesis.
2. **Reduce claims to checks** — byte, hash, vtable, constant ([C50.2](02-byte-verification.md)–[C50.4](04-vtable-verification.md))
   — cheap, decisive tests.
3. **Cross-check for certainty** — converge multiple independent anchors ([C50.5](05-cross-checking.md)).
4. **Mark confidence honestly** — ✅/🟡/⚪ ([C50.1](01-confidence-tiers.md)) — never let inference pass as fact.
5. **Treat received wisdom as hypothesis** — verify or refute; a claim's pedigree is not evidence
   ([C50.5](05-cross-checking.md)).

This is the method the whole book enacts, chapter by chapter — and it's the most transferable thing in it. The
specific facts about Most Wanted are valuable; the *discipline of establishing them* is invaluable, because it's
how you'd establish facts about *any* system whose source you don't have. That is the deepest sense in which this
is a textbook: it teaches not just what MW05 is, but how to *know* what a binary is.

## The half-way mark

Chapter 50 is the book's methodological centre — the point where the method used everywhere is made explicit. The
first half established the *formats and substrate* (containers, assets, vaults, the runtime, the vehicle, the AI,
the pursuit); the method chapter names *how we know* it all; and the chapters ahead
([51](../C51-Render-Pipeline/C51-Render-Pipeline.md)+) apply the same discipline to the rendering, effects, game
flow, and the development pipeline. Every one of them will be ✅ where the bytes allow and 🟡 where they don't —
because that is what this book *is*: a verified reference, honest about its own limits, teaching the practice of
knowing a binary.

## RE implications

- **Every ✅ claim reduces to a reproducible check** — the book is falsifiable, resting on the files, not
  authority.
- **Honest uncertainty** (🟡/⚪) marks where the bytes stop — restraint is a feature, not a weakness.
- **Verification-first makes the book usable** — act on ✅, check further on 🟡, learn the method itself.
- **RE as a practice** — anchor, reduce to checks, cross-check, mark honestly, treat received wisdom as hypothesis
  — transfers to any binary.

---

### Key takeaways

- **Every verified claim reduces to a concrete, reproducible check** — read the bytes, hash the name, count the
  vtable, read the constant — so the book is **falsifiable**, resting on the files rather than authority.
- **Honest uncertainty** (🟡 Reasoned, ⚪ Unverified) marks where the evidence stops — a **feature** that tells you
  exactly where the solid ground ends.
- Verification-first is what makes the book **usable to a developer** — act on ✅ facts, verify further on 🟡, and
  learn the *method* to extend it correctly.
- The book models **RE as a transferable practice** — anchor on the artifacts, reduce claims to checks,
  cross-check, mark confidence, treat received wisdom as hypothesis.
- This is the deepest sense in which it's a **textbook**: it teaches not just *what* Most Wanted is, but *how to
  know* what a binary is.

**Next:** [Chapter 51 — The Render Pipeline](../C51-Render-Pipeline/C51-Render-Pipeline.md): how the game turns the
world into pixels.

**Sources:** worked examples verified live in `speed.exe`/`attributes.bin` — `Math::Sqrt 0x5C5E80` =
`D9 44 24 04 D9 FA C3`; `rh("EngineRacer")=0xB2809518` (×4 in vault); `AIVehicle 0x00891998` = 351 methods; float at
`0x892FA8` = `90.0`. ImageBase `0x400000`, file-offset = VA − `0x400000`.
