# C21.6 — Editing Music Safely

> **The one-sentence version:** replace a section by re-encoding audio to fit its byte span and codec, keep
> loops musical and DWORD alignment intact, and edit MUS and MPF together so the `sectionIndex` join stays
> valid.

[← C21.5 — EventID → NodeID → section](05-routing.md) · [Chapter 21 hub](C21-Music-MUS-MPF.md) ·
[Next: Chapter 22 — Engine Sound (GIN) →](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md)

---

## Two things move together

Music editing has a constraint the other audio chapters don't: **MUS and MPF are a unit**
([C21.4](04-mpf-director.md)). Any change that alters *which* section is at *which* index — adding, removing, or
reordering sections — must be mirrored in the MPF's node `sectionIndex` values ([C21.5](05-routing.md)), or the
director plays the wrong music. So the first rule is: **edit the pair, not one file.**

## Replacing a section's audio

The common edit — swap the audio of an existing section without changing the section layout — is the safe one,
because it leaves every `sectionIndex` valid:

```python
def replace_section_audio(section, new_wav):
    audio = load_wav(new_wav)
    audio = resample(audio, section.rate)      # MUS is fixed-rate (C20.2) — match the original
    encoded = encode(audio, section.codec)     # keep the section's codec (C21.2)
    fit_to_span(encoded, section.scdl_size)    # fit the SCDl payload span → no offset cascade
    write_scdl(section, encoded)               # keep SCEl DWORD-aligned (C21.2)
```

Three constraints ride along:

- **Fixed rate** — MUS doesn't honour a patched rate ([C20.2](../C20-Audio-Codecs/02-replacement-rules.md));
  resample to the section's original rate.
- **Same codec** — re-encode with the section's codec (or update the `SCHl` codec tag to match)
  ([C21.2](02-section-blocks.md)).
- **DWORD alignment** — after editing, `SCEl` must still land the next section on a 4-byte boundary
  ([C21.1](01-mus-sections.md)).

Fitting the re-encoded audio to the original `SCDl` span keeps the section the same size, so no later section
moves and no `sectionIndex` shifts — the music version of a same-size swap.

## Keeping loops musical

If your replacement changes the section's musical content, revisit its **loop points**
([C21.3](03-loops-sections.md)):

- **Land loops on bar/beat boundaries** so the repeat is in time.
- **Make the boundary seamless** — the loop end must flow into the loop start without a click, watching codec
  history at the boundary for predictive codecs ([C20.3](../C20-Audio-Codecs/03-eaxa-decode.md)).
- **Don't loop a one-shot** — stingers are meant to play once ([C21.3](03-loops-sections.md)).

A mis-placed loop is the most audible music-edit bug: it clicks or stumbles every repeat, which in a
long-held pursuit loop is relentless.

## Re-routing without touching audio

To change *what music an event brings* without re-encoding anything, edit the MPF: point a node at a different
`sectionIndex` ([C21.5](05-routing.md)). This re-routes the score — make pursuits use a different tension loop,
or swap which section resolves an escape — purely by remapping nodes, the cheapest music edit.

## Structural edits (add/remove sections)

Adding or removing sections is the invasive case:

1. **Edit the MUS** — insert/remove the section, keeping block order (`SCHl`→`SCDl`→`SCEl`) and DWORD alignment
   ([C21.2](02-section-blocks.md)).
2. **Re-index the MPF** — every node `sectionIndex` at or after the change shifts; fix them all
   ([C21.5](05-routing.md)).
3. **Add edges** for a new state so the director can reach and leave it ([C21.4](04-mpf-director.md)).
4. **Fix MUS offsets/sizes** — a resized section shifts later sections; re-stamp the section table/offsets.

This is the most error-prone music edit because it touches both files and the index join between them — prefer
same-size audio replacement and node re-routing when they achieve your goal.

## Verify

Music edits demand listening, because structural correctness doesn't guarantee musical correctness:

1. **Sections walk cleanly** — `SCHl`/`SCDl`/`SCEl` parse and DWORD-align ([C21.2](02-section-blocks.md)).
2. **`sectionIndex` values are all in range** after any structural change.
3. **Decode-and-compare** the replaced section's audio (duration/level/character —
   [C20.6](../C20-Audio-Codecs/06-portable-decoder.md)).
4. **Play it in context** — trigger the gameplay states and confirm the right music plays, loops seamlessly,
   and transitions on cue. This is the only test that catches a bad loop or a mis-routed node.

---

### Key takeaways

- MUS and MPF are a **unit** — edit the pair; structural changes must re-index every MPF `sectionIndex`.
- Safe edit: re-encode a section's audio to the same codec, resample to its fixed rate, fit the `SCDl` span,
  keep `SCEl` DWORD-aligned.
- Keep loops on musical boundaries and seamless; never loop a one-shot stinger.
- Re-route cheaply by remapping MPF nodes to different section indices — no re-encoding.
- Add/remove sections only with full MUS+MPF re-indexing; verify by parse, index range, decode-compare, and —
  decisively — listening in context.

**Continue:** [Chapter 22 — Engine Sound (GIN) & the RPM→Synth Bridge](../C22-Engine-Sound-GIN/C22-Engine-Sound-GIN.md) ·
[Chapter 21 hub](C21-Music-MUS-MPF.md)
