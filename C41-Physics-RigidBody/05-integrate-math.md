# C41.5 — IntegrateMotion & the Math

> **The one-sentence version:** `IntegrateMotion` (`0x6BA510`, `sub esp,0x530`) is the rigid-body integrator, and
> its speed computation calls `Math::Sqrt` (`0x5C5E80`) — verified to be *literally* `fld dword [esp+4]; fsqrt;
> ret`, an FPU square root beyond any doubt.

[← C41.4 — Physics::Simulate byte by byte](04-simulate-thiscall.md) · [Chapter 41 hub](C41-Physics-RigidBody.md) ·
[Next: C41.6 — Vehicle types →](06-vehicle-types.md)

---

## The integrator's frame

`IntegrateMotion` at `0x6BA510` is the function that advances a body's motion, and its prologue advertises how
much math it does:

```asm
0x6BA510  IntegrateMotion:
    81 EC 30 05 00 00    sub  esp, 0x530     ; 1328-byte stack frame
    53                   push ebx
    8B D9                mov  ebx, ecx        ; ebx = this (__thiscall)
    8B ...
```

The **`sub esp, 0x530`** reserves **1328 bytes** of local stack — a very large frame. That's the signature of a
heavy floating-point routine: it needs room for many temporary vectors (3 floats each) and matrices (a 3×3 is 9
floats, a transform is more) as it computes the velocity update, the position update, and the orientation update
of a rigid body. A trivial function has a small frame; a `0x530` frame means serious math.

The `mov ebx, ecx` confirms it's a `__thiscall` too — `IntegrateMotion` is a member of the body, operating on the
`this` in `ECX` (saved to `EBX`).

> ✅ *Verified:* `IntegrateMotion` at `0x6BA510` = `81 EC 30 05 00 00 53 8B D9` = `sub esp,0x530; push ebx;
> mov ebx,ecx` — a `__thiscall` with a 1328-byte math frame.

## `Math::Sqrt` — verified beyond doubt

The single most cleanly-verified function in the physics system is **`Math::Sqrt` at `0x5C5E80`**, which
`IntegrateMotion` calls to compute speed from velocity ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)). Its
entire body is seven bytes:

```asm
0x5C5E80  Math::Sqrt:
    D9 44 24 04     fld   dword ptr [esp+4]   ; load the float argument
    D9 FA           fsqrt                      ; square root it
    C3              ret                        ; return (result in ST0)
```

Byte string: `D9 44 24 04 D9 FA C3`. This is *unambiguously* a square-root function: it loads a single-precision
float from the stack (`[esp+4]` — the first argument in `__cdecl`), executes the x87 **`fsqrt`** instruction, and
returns with the result in `ST0`. There is no interpretation needed — the `fsqrt` opcode (`D9 FA`) is the FPU
square root, and the function does nothing else. This is the gold standard of RE verification: a function whose
identity is proven by its opcodes.

> ✅ *Verified:* `Math::Sqrt` at `0x5C5E80` = `D9 44 24 04 D9 FA C3` = `fld dword [esp+4]; fsqrt; ret` — an FPU
> square root, identity proven by the `fsqrt` (`D9 FA`) opcode.

## Speed = |velocity|

Why does the integrator need a square root? Because **speed is the magnitude of the velocity vector**:

```
speed = |velocity| = sqrt(vx² + vy² + vz²)
```

`IntegrateMotion` computes the new velocity (from the accumulated forces,
[C39.4](../C39-Vehicle-Simulation/04-integrate.md)), then takes its magnitude via `Math::Sqrt` to get the scalar
speed. That speed is the number the whole game reads ([C39.4](../C39-Vehicle-Simulation/04-integrate.md)) — the
HUD speedometer ([Chapter 25](../C27-FrontEnd-Shell-UI/04-hud.md)), the AI's speed judgements
([Chapter 47](../C47-AI-Driver-Vehicle/C47-AI-Driver-Vehicle.md)), the pursuit escape logic
([Chapter 48](../C48-Pursuit-Heat/C48-Pursuit-Heat.md)), the engine sound's RPM coupling
([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)). So the humble `fsqrt` at `0x5C5E80` is on the
path from "the car is moving" to "the speedometer reads 180."

## The integration scheme

The rigid-body integration `IntegrateMotion` performs is the standard timestep update
([C39.4](../C39-Vehicle-Simulation/04-integrate.md)):

```
velocity     += (force / mass) · dt          // linear
angular_vel  += (torque / inertia) · dt      // rotational
position     += velocity · dt                 // move
orientation  += angular_vel ⊗ dt             // rotate (quaternion/matrix)
speed         = |velocity|                    // Math::Sqrt (0x5C5E80)
transform     = compose(position, orientation) // write [this+0xF0]
```

over the frame's timestep `dt` (the global `[0x9259BC]`, [C37.4](../C37-Frame-Spine-Modules/04-frametick.md)). The
1328-byte frame ([above](#the-integrators-frame)) holds the intermediate vectors and the orientation math (the
most expensive part — normalising a quaternion or orthonormalising a matrix each step). The result is the body's
new transform, written at `[this+0xF0]` ([C39.3](../C39-Vehicle-Simulation/03-part-array.md)).

> 🟡 *Reasoned:* the specific integration scheme (semi-implicit Euler, the orientation representation) is the
> standard rigid-body integration, consistent with the verified large math frame, the `dt` global, and the
> `Math::Sqrt` speed computation; the exact per-instruction math is deeper RE. The `sub esp,0x530` frame,
> the `__thiscall`, and `Math::Sqrt`'s `fld;fsqrt;ret` are verified.

## RE implications

- **`IntegrateMotion (0x6BA510)`** is the integrator — `sub esp,0x530` (1328-byte math frame), `__thiscall`
  (`mov ebx,ecx`).
- **`Math::Sqrt (0x5C5E80)` is `fld [esp+4]; fsqrt; ret`** — a square root proven by the `fsqrt` opcode, the
  book's cleanest verification.
- **Speed = |velocity|** via `Math::Sqrt` — the scalar the HUD/AI/pursuit/audio read.
- **The integration** is the standard force→velocity→position update over `dt`, writing `[this+0xF0]`.

---

### Key takeaways

- `IntegrateMotion (0x6BA510)` is the **rigid-body integrator** — a `__thiscall` with a **1328-byte** (`sub
  esp,0x530`) floating-point frame.
- **`Math::Sqrt (0x5C5E80)` = `fld dword [esp+4]; fsqrt; ret`** — an FPU square root, identity **proven by the
  `fsqrt` opcode** (the gold standard of verification).
- The integrator uses `Math::Sqrt` to compute **speed = |velocity|** — the scalar the whole game reads.
- The integration is the standard **force → velocity → position** update over the frame's `dt` (`[0x9259BC]`),
  writing the transform at **`[this+0xF0]`**.
- These byte-level verifications make the physics core one of the **most firmly grounded** parts of the book.

**Continue:** [C41.6 — Vehicle types: the engine's breadth](06-vehicle-types.md) · [Chapter 41 hub](C41-Physics-RigidBody.md)
