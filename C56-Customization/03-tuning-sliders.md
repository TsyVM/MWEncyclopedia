# C56.3 — Tuning Sliders

> **The one-sentence version:** beyond discrete parts, tuning sliders (`TuningScreen`/`TuningSlider`) let you
> fine-tune handling continuously — brake balance, steering, ride height, gearing — each slider mapping to a vault
> field, dialing in the car's exact feel within the installed parts' range.

[← C56.2 — Performance parts](02-performance-parts.md) · [Chapter 56 hub](C56-Customization.md) ·
[Next: C56.4 — Visual customization →](04-visual.md)

---

## Continuous tuning atop discrete parts

Parts ([C56.2](02-performance-parts.md)) are *discrete* upgrades (install a tier). **Tuning sliders**
(`TuningScreen`/`TuningSlider`) are the *continuous* layer on top — fine adjustments to the car's handling within
the range the installed parts allow:

- **Brake balance** — front/rear braking bias (more front = stable, more rear = rotates).
- **Steering** — sensitivity/response ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)).
- **Ride height / suspension** — stiffness, height ([C42.3](../C42-Suspension-Tyres-Drivetrain/03-suspension.md)).
- **Gearing** — final drive / ratios ([C42.2](../C42-Suspension-Tyres-Drivetrain/02-engine-drivetrain.md)) —
  trading acceleration for top speed.

So sliders are the *fine* control where parts are the *coarse*: parts set the car's *capability* (how good the
suspension is); sliders set its *balance* (how that suspension is configured). A driver dials in their preference —
a stable car or a sharp one — via the sliders, within what their parts permit.

> ✅ *Verified:* `Tuning`, `TuningScreen`, and `TuningSlider` are present in `speed.exe` — the fine-tuning UI.

## Each slider is a vault field

Like parts ([C56.2](02-performance-parts.md)), each slider **maps to a vault field**
([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) — but *continuously*
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)):

```
move the "brake balance" slider →
   set the brake-bias vault field to the slider's value (continuous, 0..1)
   → the sim ([C42.3]) reads the new bias → the car brakes differently
```

Where a part *jumps* a field to a tier's value ([C56.2](02-performance-parts.md)), a slider *sweeps* a field
continuously across its range. So sliders give *analog* control over the handling parameters — you can set the brake
balance to exactly where you like it, not just "stock" or "pro." This is the finest grain of the data-over-code
customization ([C56.1](01-two-customizations.md)): the player directly authoring a continuous vault value via a UI
slider, and the sim ([Chapter 42](../C42-Suspension-Tyres-Drivetrain/C42-Suspension-Tyres-Drivetrain.md)) reading it
live.

## Why sliders matter

Continuous tuning sliders serve the *serious* player and the *feel* of ownership:

- **Personalisation of feel.** Two players with identically-parted cars can tune them to drive *differently* — one
  stable, one aggressive — via the sliders. The car becomes *theirs* in handling, not just looks.
- **Situational tuning.** A slider setup good for tight circuits ([Chapter 55](../C55-Race-Events/C55-Race-Events.md))
  differs from one for top-speed events ([C55.4](../C55-Race-Events/04-speed-modes.md)) — tune for the challenge.
- **Depth for the engaged.** Casual players ignore the sliders (the parts are enough); engaged players dial them in
  — the system rewards investment without demanding it.

So tuning sliders are the *depth* layer of performance customization — optional, continuous refinement atop the
essential discrete parts. They express the vault's ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md))
full granularity to the player: the same handling fields the developers tuned
([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)) are exposed as sliders for the player to tune.
This closes the data-over-code loop completely — the player has (bounded) access to the very parameters that define
the car's handling.

> 🟡 *Reasoned:* the specific slider set (brake balance, steering, ride height, gearing) is the standard tuning
> surface, consistent with the verified `TuningSlider` UI and the handling vault fields
> ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)); the exact sliders and ranges are per-config data.
> The tuning UI is verified.

## RE implications

- **Tuning sliders** (`TuningScreen`/`TuningSlider`) fine-tune handling continuously — brake balance, steering,
  ride height, gearing.
- **Each slider maps to a vault field** — continuously (sweep), where a part jumps it (discrete).
- **Fine atop coarse** — parts set capability, sliders set balance.
- **Depth for the engaged** — optional continuous refinement; the player authoring vault handling values directly.

---

### Key takeaways

- **Tuning sliders** (`TuningScreen`/`TuningSlider`) are the **continuous** layer atop the **discrete** parts —
  fine-tuning brake balance, steering, ride height, and gearing.
- Each slider **maps continuously to a vault field** — sweeping a handling parameter across its range (where a part
  jumps it to a tier value).
- Sliders are the **fine control** (balance/configuration) where parts are the **coarse** (capability) — dialing in
  the car's exact feel.
- They enable **personalisation** (same parts, different feel), **situational tuning** (circuit vs. top-speed), and
  **depth** for engaged players (optional).
- Sliders **close the data-over-code loop** — the player directly authors the very handling parameters the
  developers tuned ([C42.5](../C42-Suspension-Tyres-Drivetrain/05-tuning-surface.md)).

**Continue:** [C56.4 — Visual customization](04-visual.md) · [Chapter 56 hub](C56-Customization.md)
