# C19.5 — SFX Vocabulary & Routing

> **The one-sentence version:** a game event names a sound, that name (or id) routes to a bank entry, and the
> entry's codec + payload play it — so the sound system is the same "reference resolves to data" indirection as
> textures and vault fields, applied to audio.

[← C19.4 — The big-endian PT records](04-pt-records.md) · [Chapter 19 hub](C19-Audio-Banks.md) ·
[Next: C19.6 — Editing banks safely →](06-editing-banks.md)

---

## From event to sound

Playing a sound is a routing chain, and it looks familiar:

```
game event (crash, UI click, cop siren)
   → sound name / id
   → bank lookup (SNR entry or BNKl PT by id)
   → codec + payload → decoded samples → mixer
```

The event doesn't carry audio — it names a sound; the bank resolves the name/id to an entry; the entry names a
codec and points at the payload; the codec ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)) produces
samples. This is the audio version of the indirection that runs the whole engine: a **reference resolves to
data** — the same shape as a material's texture key ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)) or a
vault field's value ([C12.5](../C12-Reflection-Schema/05-resolving-values.md)).

## The vocabulary

Sounds are named, and the names are a vocabulary you can read from the banks (the SNR name pool
[C19.2](02-snr-spt.md), or bank/file names). The naming is domain-organised, much like the vault's `fx*`
instances ([C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md)) and the sound directory
structure:

- **`ENGINE/`** — per-car engine banks (`CAR_00_ENG_*`), paired with GIN
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).
- **`FE/` and front-end banks** — menu/UI sounds.
- **`SKIDS/`, `NOS/`, `TURBO/`, `SHIFTING/`** — the car's dynamic effects (tyre skids, nitrous, turbo, gear
  shifts), each a domain of sounds.
- **`SPEECH/`** — cop and character voice.
- **`STREAMS/`** — streamed audio (NIS cutscene audio, `NISAudio.*`).

The directory *is* the vocabulary's top level: to find the skid sounds, look in `SKIDS/`; to find engine sound,
`ENGINE/`. Learning the layout is learning where each kind of sound lives.

## Routing binds events to sounds

The binding from a game event to a specific sound is data-driven — the vault
([Chapter 14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)) and event systems reference sounds
by name/id, and the audio system resolves them. So, as with everything else in the engine, *what plays when* is
editable data:

- A collision event ([C14.4](../C14-Vault-Pursuit-Surfaces/04-effects-destructibles.md)) references a crash
  sound by id; changing the reference changes what you hear on impact.
- An engine's sound is bound to its car via the engine bank + GIN pairing
  ([Chapter 22](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)).

This means re-sounding the game is often about **re-routing** (which sound an event names) as much as
**replacing** (the sound's bytes) — the same two levers as re-skinning (retarget vs replace pixels).

## Using the vocabulary to navigate

When reverse-engineering or modding audio, the vocabulary is your map:

- **Find the right bank by domain** — the directory tells you where a kind of sound lives.
- **Enumerate a bank's sounds by name** — the SNR name pool or `PT`/file names label each one.
- **Follow references** — a missing or wrong sound is usually a routing issue (an event naming the wrong id),
  not a decode failure — check the reference resolves before suspecting the audio.

## Editing implications

- **Re-route to change what plays when** — point an event at a different sound id.
- **Replace to change how a sound sounds** — edit the bank entry's payload ([C19.6](06-editing-banks.md)).
- **Preserve ids/names** so existing references keep resolving — the audio analogue of preserving texture keys
  ([C7.3](../C7-Materials-TexAnim/03-texture-binding.md)).
- **Use the directory as the index** — it's the fastest way to locate a domain of sounds.

---

### Key takeaways

- Playing a sound is a routing chain: event → name/id → bank entry → codec + payload → samples — the engine's
  "reference resolves to data" again.
- Sounds are named; the `SOUND/` directory (`ENGINE`, `SKIDS`, `NOS`, `SPEECH`, `STREAMS`, …) is the
  vocabulary's top level.
- Event-to-sound bindings are data-driven — re-sounding is re-routing (which sound) plus replacing (its bytes).
- Navigate by domain (directory), enumerate by name (SNR pool / `PT`), and follow references to diagnose.
- Re-route to change what plays; replace to change how it sounds; preserve ids so references resolve.

**Continue:** [C19.6 — Editing banks safely](06-editing-banks.md) · [Chapter 19 hub](C19-Audio-Banks.md)
