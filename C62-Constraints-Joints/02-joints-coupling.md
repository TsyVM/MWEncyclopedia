# C62.2 — Joints & Coupling

> **The one-sentence version:** a `Joint` couples a trailer to its truck at the hitch — a point where the trailer
> pivots but can't separate — and `JointDetached` is the broken state, the hitch coming apart on a hard impact.

[← C62.1 — The constraint system](01-constraints.md) · [Chapter 62 hub](C62-Constraints-Joints.md) ·
[Next: C62.3 — Trailers →](03-trailers.md)

---

## The hitch: a pivot joint

The **`Joint`** that couples a trailer to its truck is a *hitch* — a specific kind of joint
([C62.1](01-constraints.md)) that allows *pivoting* but not *separation*:

- **Pivots** — the trailer can swing left/right (and pitch) about the hitch point, so it follows the truck through
  turns, trailing and swinging.
- **Doesn't separate** — the trailer stays *attached* at the hitch; it can't drift away (the constraint
  [C62.1](01-constraints.md) removes the separation freedom).

So the hitch is a joint that holds the trailer *at a point* while letting it *rotate* about that point — exactly a
real trailer hitch. This pivoting-but-attached behaviour is what makes a towed trailer look right: it trails behind,
swings out in corners, and can jackknife ([C62.4](04-jackknife.md)) — all the pivot allowing, the attachment
constraining. The verified `Joint` class is this coupling.

> ✅ *Verified:* `Joint` and `JointDetached` are present in `speed.exe` — the coupling and its detached state.
> `TrailerPos` and `SFXCTL_3DTrailerPos` ([C59.2](../C59-Audio-Runtime/02-3d-positional.md)) position the trailer.

## JointDetached: breaking the link

The verified **`JointDetached`** is the coupling's *broken* state — the hitch coming apart:

- **On a hard impact** — a severe collision ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md))
  can *break* the hitch, detaching the trailer from the truck.
- **The trailer becomes free** — once detached, the trailer is a *lone* rigid body
  ([Chapter 41](../C41-Physics-RigidBody/C41-Physics-RigidBody.md)) — no longer constrained, it tumbles or slides
  freely.
- **A dramatic moment** — a trailer breaking loose (in a crash, or a jackknife gone wrong,
  [C62.4](04-jackknife.md)) is a spectacular event — the linked system coming apart.

So the coupling is a joint that can *fail* — holding under normal loads, detaching under extreme ones. This is why
constraints ([C62.1](01-constraints.md)) matter for drama: a rigidly-modelled trailer (one body) couldn't break
loose, but a *jointed* trailer can (`JointDetached`), turning a hard hit into a trailer flying free. The
detach-on-impact behaviour makes truck-and-trailer collisions *consequential and cinematic* — the hitch is a point
of failure, and breaking it is a visible, physical event.

> 🟡 *Reasoned:* the break-on-hard-impact behaviour of the hitch joint is the natural reading of the verified
> `JointDetached` state and the collision system ([Chapter 43](../C43-Collision-Contacts/C43-Collision-Contacts.md));
> the exact break threshold is per-config. The joint and detached-state classes are verified.

## Why a joint, not a rigid attachment

Modelling the trailer coupling as a *joint* (two bodies) rather than a *rigid attachment* (one body) is the crucial
choice ([C62.1](01-constraints.md)):

- **Articulation** — a joint lets the trailer *swing* ([above](#the-hitch-a-pivot-joint)) — trail, corner, and
  jackknife ([C62.4](04-jackknife.md)). A rigid attachment would make the truck-and-trailer a single stiff shape
  that couldn't articulate.
- **Realistic dynamics** — the trailer has its *own* mass, inertia, and suspension
  ([C62.3](03-trailers.md)) — it responds to the road independently, connected only at the hitch. This gives the
  weight and lag of a real trailer.
- **Detachment** — a joint can break ([above](#jointdetached-breaking-the-link)); a rigid attachment can't.

So the joint is what makes a trailer a *trailer* — an articulated, independently-dynamic, detachable body coupled
to the truck, not a rigid extension of it. This is the payoff of the constraint system
([C62.1](01-constraints.md)): it turns two bodies into a linked *system* that moves believably (swinging,
jackknifing) and fails believably (detaching). The coupling is a small but characterful piece of the physics — the
reason MW's trucks-and-trailers behave like the real thing.

## RE implications

- **The hitch is a `Joint`** — a pivot coupling: the trailer swings about the hitch but can't separate.
- **`JointDetached`** is the broken state — the hitch failing on a hard impact, the trailer flying free.
- **A joint, not a rigid attachment** — enables articulation (swing/jackknife), independent dynamics, and
  detachment.
- **Characterful physics** — the coupling makes trucks-and-trailers behave (and break) like the real thing.

---

### Key takeaways

- The truck-trailer coupling is a **`Joint`** — a **hitch** that lets the trailer **pivot** (swing/corner/jackknife)
  but **can't separate** (the constraint removes the separation freedom).
- **`JointDetached`** is the coupling's **broken state** — the hitch failing on a **hard impact**, leaving the
  trailer a free rigid body that tumbles.
- Modelling it as a **joint** (two bodies) not a rigid attachment (one) enables **articulation**, **independent
  dynamics** (the trailer's own mass/suspension), and **detachment**.
- The detach-on-impact makes truck-trailer collisions **consequential and cinematic** — the hitch is a visible
  point of failure.
- The coupling is a small but **characterful** piece of physics — why MW's trucks-and-trailers behave, and break,
  like the real thing.

**Continue:** [C62.3 — Trailers](03-trailers.md) · [Chapter 62 hub](C62-Constraints-Joints.md)
