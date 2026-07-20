# C50.1 — The Confidence Tiers

> **The one-sentence version:** every claim in this book carries one of three marks — ✅ Verified (proven against a
> byte, hash, vtable, or constant in the shipped files), 🟡 Reasoned (a supported inference from verified anchors),
> or ⚪ Unverified (stated openly as unknown) — a contract with the reader about how much to trust it.

[← Chapter 50 hub](C50-Verification-Methodology.md) · [Next: C50.2 — Byte verification →](02-byte-verification.md)

---

## Three tiers

The book distinguishes three levels of certainty, and marks every non-trivial claim with one:

- **✅ Verified** — the claim is confirmed by a *direct check* against `speed.exe` or `attributes.bin`: a byte
  pattern ([C50.2](02-byte-verification.md)), a hash in the vault ([C50.3](03-hash-verification.md)), a vtable
  method count ([C50.4](04-vtable-verification.md)), or a constant value. This is *fact* — reproducible by anyone
  with the files.
- **🟡 Reasoned** — the claim is a *well-supported inference*: it follows from verified anchors plus standard engine
  design, and is consistent with observed behaviour, but isn't itself byte-proven. This is *likely true*, flagged
  so the reader knows it's interpretation, not measurement.
- **⚪ Unverified** — the claim is *stated as open*: a plausible reading, a community assertion not yet checked, or
  an acknowledged unknown. This is *honesty about the edge of knowledge*.

Every ✅ in this book corresponds to a check that was actually run; every 🟡 marks where verification stops and
reasoning begins.

## Why mark at all

The marks are the difference between a *textbook* and *lore*. Reverse-engineering communities accumulate a mix of
proven facts, plausible guesses, and repeated-until-believed errors — often indistinguishable in the retelling. A
reference that doesn't separate them inherits all their uncertainty invisibly. By marking every claim:

- **The reader can calibrate trust.** A ✅ can be built on; a 🟡 should be checked before relying on it; a ⚪ is a
  research lead, not a foundation.
- **The book is falsifiable.** A ✅ claim names its check, so anyone can re-run it and confirm (or refute) it. This
  is what makes the book *scientific* rather than authoritative-by-assertion.
- **Errors are contained.** A wrong 🟡 is flagged as inference, so it doesn't masquerade as fact. When the book
  corrects the community ([C50.5](05-cross-checking.md)), the correction is a ✅ replacing a ⚪.

So the tiers are not hedging — they're precision. Saying "🟡 Reasoned" is a *stronger* statement than an unmarked
assertion, because it tells you exactly how far the evidence goes.

## The discipline of the mark

Applying the tiers honestly requires discipline — the constant question "*how do I actually know this?*":

- **If the answer is "I read the bytes / computed the hash / counted the vtable"** → ✅ Verified, and the check is
  named.
- **If the answer is "it follows from X (verified) plus how engines usually work"** → 🟡 Reasoned, and the anchor X
  is cited.
- **If the answer is "the community says so" or "it seems likely"** → ⚪ Unverified, until checked.

The temptation in RE is to let a confident-sounding inference drift into stated fact. The tiers resist that: a claim
can't be ✅ without a named check. This is why the book's ✅ claims are trustworthy — they *couldn't* have been
marked so without the check existing. The mark is a promise, and the promise was kept for every one.

## Tiers in practice

Throughout the book, the pattern is consistent:

- A function's *address and prologue bytes* are ✅ ([C50.2](02-byte-verification.md)); its *detailed internal
  algorithm* is often 🟡 (the prologue is measured, the full behaviour inferred).
- A vault *record's existence* (hash-in-vault) is ✅ ([C50.3](03-hash-verification.md)); the *meaning of its
  fields* is often 🟡 (the key is measured, the semantics inferred).
- A class's *vtable and method count* are ✅ ([C50.4](04-vtable-verification.md)); the *role of each method* is 🟡.

So a typical chapter is a scaffold of ✅ anchors (addresses, hashes, counts, bytes) with 🟡 interpretation hung
between them — the verified skeleton carrying the reasoned flesh. The reader always knows which is which. That
structure — measured anchors, marked inference — is the book's method in miniature.

## RE implications

- **Three tiers** — ✅ Verified (direct check), 🟡 Reasoned (supported inference), ⚪ Unverified (open) — mark every
  claim.
- **The marks calibrate trust**, make the book falsifiable, and contain errors.
- **A claim can't be ✅ without a named check** — the mark is a kept promise.
- **Chapters are ✅ anchors with 🟡 interpretation between** — measured skeleton, reasoned flesh.

---

### Key takeaways

- Every claim carries **✅ Verified** (proven by a byte/hash/vtable/constant check), **🟡 Reasoned** (supported
  inference from verified anchors), or **⚪ Unverified** (open unknown).
- The marks are the difference between a **textbook and lore** — they let the reader calibrate trust, make claims
  **falsifiable**, and **contain** errors.
- Applying them requires the discipline of always asking **"how do I actually know this?"** — a ✅ demands a *named
  check*.
- A typical chapter is a **scaffold of ✅ anchors** (addresses, hashes, counts) with **🟡 interpretation** between —
  the reader always knows which is which.
- Marking is **precision, not hedging** — "🟡 Reasoned" states exactly how far the evidence reaches.

**Continue:** [C50.2 — Byte verification](02-byte-verification.md) · [Chapter 50 hub](C50-Verification-Methodology.md)
