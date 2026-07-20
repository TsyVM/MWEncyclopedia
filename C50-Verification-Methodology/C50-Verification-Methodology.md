# Chapter 50 — Verification Methodology

> **Goal of this chapter:** make explicit the discipline the whole book rests on — the three confidence tiers
> (✅ Verified / 🟡 Reasoned / ⚪ Unverified), the four proof techniques (byte, hash, vtable, constant), how every
> claim reduces to a check, and why verification-first RE is more honest and more useful than the community's
> received wisdom.

Every other chapter *applies* a method; this one *names* it. Across 49 chapters, claims about Most Wanted have been
marked ✅ Verified, 🟡 Reasoned, or left explicitly uncertain — and the verified ones were each reduced to a
concrete check against `speed.exe` or `attributes.bin`. This chapter is the methodological spine: what those marks
mean, how the checks work, and why this discipline matters. It is the reason this book can claim to be a *textbook*
rather than a collection of lore.

> **Verified against the executable and vault — a worked example.** The four core techniques, demonstrated live:
> **byte** — `Math::Sqrt (0x5C5E80)` = `D9 44 24 04 D9 FA C3` = `fld [esp+4]; fsqrt; ret`; **hash** —
> `rh("EngineRacer")=0xB2809518`, present ×4 in `attributes.bin`; **vtable** — `AIVehicle (0x00891998)` = 351
> methods (the biggest class in the game); **constant** — the float at `0x892FA8` = `90.0` (the engaged bust
> radius). Each is a decisive, repeatable check. ImageBase `0x400000`; code file-offset = VA − `0x400000`.

---

## Deep-dive pages

- [C50.1 — The confidence tiers](01-confidence-tiers.md): ✅ Verified, 🟡 Reasoned, ⚪ Unverified.
- [C50.2 — Byte verification](02-byte-verification.md): reading code, prologues, the `fsqrt` gold standard.
- [C50.3 — Hash verification](03-hash-verification.md): the reflection hash and hash-in-vault as a decisive test.
- [C50.4 — VTable verification](04-vtable-verification.md): the method-count check and class identity.
- [C50.5 — Cross-checking & correcting received wisdom](05-cross-checking.md): convergence and fixing errors.
- [C50.6 — The discipline & why it matters](06-the-discipline.md): honest uncertainty, and RE as a practice.

---

## 50.1 The confidence tiers

Every claim in the book carries one of three confidence marks
([C50.1](01-confidence-tiers.md)): **✅ Verified** (checked against a byte pattern, hash, vtable, or constant in the
shipped files — a fact), **🟡 Reasoned** (a well-supported inference from verified anchors plus standard engine
design — likely, but not directly byte-proven), and **⚪ Unverified** (stated openly as unknown or uncertain). The
marks are not decoration — they are a *contract* with the reader about how much to trust each statement. A textbook
that mixes fact and guess without marking them is worse than useless; the tiers keep the two separate.

## 50.2 Byte verification

The strongest tier is **byte verification** ([C50.2](02-byte-verification.md)): reading the actual machine code at a
virtual address (file-offset = VA − `0x400000`) and decoding it. The gold standard is a function whose *opcodes*
prove its identity — `Math::Sqrt (0x5C5E80)` is `fld [esp+4]; fsqrt; ret`, and the `fsqrt` opcode (`D9 FA`) leaves
no doubt it's a square root. Prologue signatures (`push -1; push <handler>` for SEH, `mov esi,ecx` for
`__thiscall`, `sub esp,0xNNN` for a stack frame) identify and characterise functions. Bytes don't lie: a claim
backed by an opcode sequence is as solid as RE gets.

## 50.3 Hash verification

The **reflection hash** ([C50.3](03-hash-verification.md)) — Jenkins lookup2, seed `0xABCDEF00` — enables a
decisive test: **hash-in-vault**. A name is a real engine key *if and only if* its hash appears in `attributes.bin`.
Because the hash is effectively collision-free for these short names, `rh("EngineRacer")=0xB2809518` appearing ×4 in
the vault *proves* `EngineRacer` is a real spec key — a made-up name would hash to nothing. This turns "is X a real
thing?" into a one-line computation, and recovers hashed-away enums (surface tags, collision types) by testing
candidate names against the vault.

## 50.4 VTable verification

**VTable verification** ([C50.4](04-vtable-verification.md)) confirms a *class*: a claimed vtable is real if it's a
clean run of code pointers, and the run length is the class's method count. `AIVehicle` at `0x00891998` has exactly
351 consecutive `.text` pointers — verifying both that it's a vtable and that it's the biggest class in the game.
Method counts also *rank* classes by complexity ([Chapter 47](../C47-AI-Driver-Vehicle/05-reading-ai-brain.md)), and
a shared vtable proves two names are the same class idling
([C46.3](../C46-AI-Goals-Actions/03-data-only-goals.md)). The count is cheap to compute and decisive.

## 50.5 Cross-checking

The most reliable facts are **cross-checked** ([C50.5](05-cross-checking.md)) — confirmed by *multiple independent
anchors*. The physics classes are verified three ways at once (strings in `.rdata`, hashes in the vault, bytes in
the code, [C41.7](../C41-Physics-RigidBody/07-reading-physics.md)); when strings, hashes, and opcodes all point to
the same conclusion, it's certain. Cross-checking is also how the book *corrected received wisdom* — the community's
claims about the GIN rpm offsets and the TPK "Joaat" hash were tested against the bytes and found wrong
([C50.5](05-cross-checking.md)), and the real answers substituted. Verification isn't just confirmation; it's
correction.

---

### Key takeaways

- Every claim carries a **confidence tier** — **✅ Verified** (byte/hash/vtable/constant check), **🟡 Reasoned**
  (supported inference), **⚪ Unverified** (open unknown) — a contract with the reader.
- **Byte verification** reads the machine code — opcodes like `fsqrt` prove identity beyond doubt; prologues
  identify functions.
- **Hash verification** uses the reflection hash — **hash-in-vault** is a decisive test that a name is a real
  engine key.
- **VTable verification** confirms classes by their **method count** (a clean run of code pointers) — also a
  complexity ranking.
- **Cross-checking** multiple independent anchors gives certainty — and lets the book **correct** the community's
  errors, not just repeat them.

**Next:** [Chapter 51 — The Render Pipeline](../C51-Render-Pipeline/C51-Render-Pipeline.md): how the game turns the
world into pixels.
