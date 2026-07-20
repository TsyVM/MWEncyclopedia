# C22.6 — Editing Engine Sound

> **The one-sentence version:** re-voice an engine by swapping its grains while keeping the header's RPM range,
> the grain table, and the accel/`_DCL` pairing intact — re-encode grains to the engine codec at the fixed
> rate, fit their lengths, and test across the whole rev band on- and off-throttle.

[← C22.5 — Accel, decel & load](05-accel-decel.md) · [Chapter 22 hub](C22-Engine-Sound-GIN.md) ·
[Next: Chapter 23 — Video (EA Multimedia / VP6) →](../C23-Video-VP6/C23-Video-VP6.md)

---

## Swap the audio, keep the map

The reliable engine-sound edit is: **change the grains (the sound), preserve the structure (the RPM map).** The
GIN is a synth ([C22.1](01-granular-synthesis.md)); its "programming" is the RPM range
([C22.2](02-gnsu-header.md)) and grain table ([C22.3](03-grains.md)), and its "sound" is the grain waveforms.
Replace the waveforms and the synth plays your engine through the same RPM mapping — the crossfade behaviour
stays intact, only the timbre changes.

```python
def revoice_gin(gin, new_engine_wav):
    grains = slice_into_grains(new_engine_wav, gin.grain_count)   # match the grain count
    grains = [resample(g, gin.rate) for g in grains]              # fixed rate (C20.2)
    grains = [encode_eaxas(g) for g in grains]                    # engine codec (C20.4)
    fit_grain_lengths(grains, gin.grain_lengths)                  # keep offsets stable if you can
    write_grains(gin, grains)                                     # re-stamp the offset table if lengths changed
    # header rpmMin/rpmMax and grain count unchanged
```

## The constraints

Four rules keep an engine edit sound:

- **Fixed rate.** GIN doesn't honour a patched rate ([C20.2](../C20-Audio-Codecs/02-replacement-rules.md)) —
  resample your grains to the file's rate.
- **Engine codec.** Encode grains as EA-XAS (the engine-audio codec, [C20.4](../C20-Audio-Codecs/04-xas-ima.md)),
  mono ([C20.2](../C20-Audio-Codecs/02-replacement-rules.md)).
- **Grain count and order.** Preserve them — the RPM→grain mapping assumes both
  ([C22.3](03-grains.md)); reordering re-maps the rev band.
- **Fit lengths, or re-stamp the table.** Same-length grains keep every offset valid (no table edit); different
  lengths shift later offsets, so re-stamp the grain-offset table (`+0x20`) and keep the data offset
  consistent ([C22.2](02-gnsu-header.md)).

## Edit the pair

Because a car's engine is two files ([C22.5](05-accel-decel.md)), re-voicing means editing **both** the main
and `_DCL` GINs:

- **Keep accel and decel consistent** — they should sound like the same engine under load and on overrun; if you
  replace one, replace the other to match.
- **Match RPM ranges** across the pair so the load crossfade stays aligned across revs.
- **Preserve the character distinction** — main = loaded/aggressive, `_DCL` = overrun/lighter.

Edit only the main GIN and the engine sounds inconsistent the moment you lift off the throttle.

## Re-mapping the rev band (advanced)

To change *where* the timbre sits — e.g. make an engine rev higher — you edit the **RPM range**
([C22.2](02-gnsu-header.md)):

- **Raise `rpmMax`** to extend the top of the band (and provide grains that sound right up there).
- **Match the car's tuning.** The GIN's range should match the car's actual redline
  ([Chapter 13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)), or the synth runs off the grains or never
  reaches the top.

This is advanced because it couples to the simulation's RPM — change the range without matching the car and the
sound and the tach disagree ([C22.4](04-rpm-bridge.md)).

## Verify across the rev band

An engine edit can sound fine at one RPM and wrong at another, so verification must sweep:

1. **Header reads back** — `Gnsu`, RPM range, grain count intact ([C22.2](02-gnsu-header.md)).
2. **Grain offsets valid** — each points at a grain start; the table is consistent with the data
   ([C22.3](03-grains.md)).
3. **Decode-and-compare grains** — duration/level/character vs intent ([C20.6](../C20-Audio-Codecs/06-portable-decoder.md)).
4. **Rev it through the whole band** — idle to redline, on-throttle and coasting, and the accel↔decel
   transition. This is the decisive test: the note must track the tach smoothly, without clicks between grains
   or a jarring lift-off.

The full-sweep listen is non-negotiable — granular synthesis hides its bugs at the RPMs and loads you don't
test.

---

### Key takeaways

- Re-voice by swapping **grains** while preserving the **RPM range** and **grain table** — the synth plays your
  timbre through the same map.
- Constraints: fixed rate (resample), engine codec (EA-XAS, mono), preserve grain count/order, fit lengths or
  re-stamp the offset table.
- Edit the **pair** (main + `_DCL`) together, matching character and RPM ranges, or lift-off sounds wrong.
- Re-mapping the rev band means editing `rpmMin`/`rpmMax` to match the car's tuning — advanced, coupled to the
  simulation.
- Verify by header/offset checks, decode-compare, and a full idle-to-redline, on/off-throttle sweep.

**Continue:** [Chapter 23 — Video (EA Multimedia / VP6)](../C23-Video-VP6/C23-Video-VP6.md) ·
[Chapter 22 hub](C22-Engine-Sound-GIN.md)
