# C50.5 — Cross-Checking & Correcting Received Wisdom

> **The one-sentence version:** the most reliable facts are confirmed by multiple independent anchors at once
> (strings + hashes + bytes), and the same discipline that confirms also *corrects* — testing community claims
> against the bytes and replacing the wrong ones (the GIN rpm offsets, the TPK "Joaat" hash).

[← C50.4 — VTable verification](04-vtable-verification.md) · [Chapter 50 hub](C50-Verification-Methodology.md) ·
[Next: C50.6 — The discipline & why it matters →](06-the-discipline.md)

---

## Convergence: multiple anchors

The strongest verification is **convergence** — when several *independent* techniques point to the same conclusion.
The physics classes ([C41.7](../C41-Physics-RigidBody/07-reading-physics.md)) are verified three ways at once:

- **Strings** — `RigidBody`, `RBVehicle`, `RBCop` are present in `.rdata` ([C41.1](../C41-Physics-RigidBody/01-rigidbody-tree.md)).
- **Hashes** — their reflection hashes are keys in `attributes.bin` ([C41.3](../C41-Physics-RigidBody/03-hash-unification.md)).
- **Bytes** — the physics functions decode as claimed ([C41.4](../C41-Physics-RigidBody/04-simulate-thiscall.md)).

When strings, hashes, and opcodes *all* agree, the conclusion is certain — three independent measurements can't all
coincidentally support a false claim. So the book's most confident facts are the *cross-checked* ones: a class
named in the code, keyed in the vault, and exercised by verified functions. Convergence turns three separate ✅
checks into one near-unshakeable conclusion.

## Correcting received wisdom

Verification isn't only confirmation — it's **correction**. Reverse-engineering communities accumulate claims that
get repeated until they're taken as fact, and some are *wrong*. The same techniques that confirm can *test and
refute* them. The book corrected two notable examples:

**The GIN rpm offsets.** The community-sourced claim placed the engine-sound `Gnsu` header's `rpmMin`/`rpmMax` at
offsets `+0x20`/`+0x24`. Reading the actual header bytes ([C22.2](../C22-Engine-Sound-GIN/02-gnsu-header.md)) showed
`rpmMin` at `+0x08` (2267.0) and `rpmMax` at `+0x0C` (8638.1); the `+0x20` region is the grain-offset table, not
the rpm range. The received offsets were wrong; the bytes gave the right ones.

**The TPK "Joaat" hash.** The received claim was that texture keys are the standard Jenkins one-at-a-time (Joaat)
hash of the texture name. Testing this ([Chapter 5](../C5-Textures-TPK/C5-Textures-TPK.md)) against 2012 real
texture names showed the keys match *no* standard hash — they're deterministic but tail-additive, a packer-minted
scheme, not Joaat. The received algorithm was wrong; the data disproved it.

> ✅ *Verified corrections:* the `Gnsu` rpm offsets (`+0x08`/`+0x0C`, values 2267.0/8638.1) and the TPK
> non-standard-hash finding were established by reading the actual bytes/data, refuting the community claims.

## Why received wisdom drifts

Understanding *why* community claims go wrong sharpens the verification instinct:

- **Repetition launders guesses into facts.** A plausible early guess, repeated across forum posts and tools,
  loses its "probably" and becomes stated fact — with no one re-checking the original evidence.
- **Partial correctness misleads.** A claim right about *structure* but wrong about *offsets* (like the GIN header)
  is convincing because most of it checks out — the error hides in the details.
- **Standard-algorithm assumptions.** "It's probably Joaat" is a reasonable prior, but a *prior is not a
  measurement* — assuming a standard hash without testing it against the data is how the TPK claim went wrong.

So the book treats *every* received claim as ⚪ Unverified until checked ([C50.1](01-confidence-tiers.md)) — not
from distrust of the community, but because a claim's *pedigree* (who said it, how often) is not evidence. Only the
bytes are evidence. This is the humbling, essential discipline: even confident, widely-repeated claims must face
the check, and some fail it.

## The archive as a source to verify, not copy

This book was built by *redoing* an earlier reference, not copying it — and cross-checking is why. The prior
archive contained many correct facts *and* some errors and unverified assertions. The method was: take each claim,
find its check ([C50.2](02-byte-verification.md)–[C50.4](04-vtable-verification.md)), and either confirm it (✅,
often with a sharper detail than the original) or correct it. This is how, for example, the archive's detailed
vtable method counts ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md),
[Chapter 46](../C46-AI-Goals-Actions/C46-AI-Goals-Actions.md)) were *all re-verified* against the executable (and
found correct), while the C15 "visibility data" mislabel and the GIN offsets were *corrected*. The archive was a
*source of hypotheses*, each earning its ✅ only by passing the check. A reference that copies its predecessor
inherits its errors; one that re-verifies every claim *improves* on it.

## RE implications

- **Convergence** — strings + hashes + bytes agreeing — gives near-certain facts (the physics classes).
- **Verification corrects** — the GIN rpm offsets and TPK hash claims were refuted by the bytes/data.
- **Received wisdom drifts** — repetition launders guesses, partial-correctness misleads, standard-algorithm
  assumptions fail.
- **The archive was a source to verify** — every claim re-checked, confirmed (sharper) or corrected — improving on
  it.

---

### Key takeaways

- The most reliable facts are **cross-checked** — confirmed by **multiple independent anchors** (strings + hashes +
  bytes) that can't all coincidentally support a falsehood.
- Verification **corrects, not just confirms** — the book refuted the community's **GIN rpm offsets** (really
  `+0x08`/`+0x0C`) and **TPK "Joaat"** claim (really a packer-minted scheme) by reading the bytes/data.
- Received wisdom **drifts** because repetition launders guesses into facts, partial correctness misleads, and
  "it's probably the standard algorithm" is a prior, not a measurement.
- Every community/archive claim is **⚪ Unverified until checked** — a claim's pedigree is not evidence; only the
  bytes are.
- The book **redid** its predecessor rather than copying it — re-verifying every claim, which is how it **improves**
  on the archive instead of inheriting its errors.

**Continue:** [C50.6 — The discipline & why it matters](06-the-discipline.md) · [Chapter 50 hub](C50-Verification-Methodology.md)
