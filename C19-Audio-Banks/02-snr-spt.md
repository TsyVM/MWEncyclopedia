# C19.2 — SNR Routing + SPT Payload

> **The one-sentence version:** `.SNR` is a routing table of 32-byte entries — id, name offset, the sound's
> offset/size in the `.SPT`, codec, sample rate, channels, duration — and `.SPT` is the payload blob; SNR is
> the *one* bank whose sample-rate field you may patch on replace.

[← C19.1 — Two bank shapes](01-two-shapes.md) · [Chapter 19 hub](C19-Audio-Banks.md) ·
[Next: C19.3 — The ABKC/BNKl container →](03-abk-bnkl.md)

---

## The split

The SNR/SPT pair separates routing from payload:

- **`.SNR`** — the routing table: a header plus one entry per sound, each entry describing the sound and
  pointing into the SPT.
- **`.SPT`** — the payload: the concatenated encoded audio blobs the SNR entries point at.

```
SNR header (LE):   magic, version, entryCount, name-pool offset/size
SNREntry (32 B):   +0x00 id
                   +0x04 nameOffset   (into the name pool)
                   +0x08 sptOffset    (into the .SPT payload)
                   +0x0C sptSize      (bytes of encoded audio)
                   +0x18 codecTag     (which codec — Chapter 20)
                   +0x1C durationMs
                   + sampleRate, channels
```

To play sound *k*: read `SNREntry[k]`, seek to `sptOffset` in the SPT, read `sptSize` bytes, decode with
`codecTag` at `sampleRate`/`channels` ([Chapter 20](../C20-Audio-Codecs/C20-Audio-Codecs.md)). The name pool
gives each sound a readable name for tools and routing.

## The entry, field by field

Each 32-byte entry is a complete play descriptor:

- **`id`** — the sound's identifier, how other systems reference it.
- **`nameOffset`** — into the name pool, resolving to a readable name.
- **`sptOffset` / `sptSize`** — where the encoded audio is and how big — the join to the SPT payload.
- **`codecTag`** — which codec decodes the bytes ([C20.1](../C20-Audio-Codecs/01-codec-set.md)).
- **`sampleRate` / `channels`** — playback format.
- **`durationMs`** — length, for scheduling and UI.

The entry is self-sufficient: given it and the SPT, you can extract and decode the sound with no other data.

## SNR's unique privilege: the editable rate

Across all the game's audio containers, **SNR is the one whose sample-rate field the engine actually reads and
respects on replacement.** ABK, GIN, and MUS ignore a patched rate ([C19.6](06-editing-banks.md),
[C20.2](../C20-Audio-Codecs/02-replacement-rules.md)) — they play the replacement at the *original* rate — so
for those you must resample your audio to match. SNR lets you instead **patch the rate to your audio**, which
makes pitch-correct replacement far easier.

So the SNR replacement recipe is the friendly one:

1. Encode your replacement with the entry's codec.
2. Write it into the SPT (and patch `sptOffset`/`sptSize`).
3. **Patch `sampleRate`** (and `channels`, `durationMs`) to your audio — the privilege SNR uniquely grants.

> ✅ *Verified (archive):* the SNR/SPT routing/payload split, the 32-byte entry layout, and SNR's editable
> rate field are confirmed; the codec tags are the shared set ([C20.1](../C20-Audio-Codecs/01-codec-set.md)).

## Editing the SPT

Because the SPT is a concatenation of blobs indexed by offset/size, editing it follows the same discipline as
every offset-indexed payload in the book (the TPK blob, the stream file):

- **Same-size replacement** — overwrite the sound's `sptSize` bytes in place; no offsets move.
- **Different-size replacement** — rewrite the SPT and re-stamp `sptOffset` for the edited sound *and every
  later one*, then fix `sptSize`. The offset cascade is the SNR/SPT version of the size-tree fixup
  ([C5.5](../C5-Textures-TPK/05-extract-replace.md)).

Combined with the patchable rate, same-size (or carefully-fit) replacement in SNR is the most controllable
audio edit in the game.

---

### Key takeaways

- `.SNR` is a routing table of 32-byte entries (id, name, `sptOffset`/`sptSize`, codec, rate, channels,
  duration); `.SPT` is the payload.
- Play a sound by reading its entry, seeking into the SPT, and decoding with the entry's codec.
- SNR is the **only** bank whose sample-rate field is respected on replace — patch the rate to your audio.
- SNR replacement: encode → write to SPT (fix offset/size) → patch rate/channels/duration.
- SPT edits follow offset-cascade discipline: same-size in place, different-size re-stamps later offsets.

**Continue:** [C19.3 — The ABKC/BNKl container](03-abk-bnkl.md) · [Chapter 19 hub](C19-Audio-Banks.md)
