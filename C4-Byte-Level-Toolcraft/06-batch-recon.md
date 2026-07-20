# C4.6 — Batch Reconnaissance

> **The one-sentence version:** run your tools across the *whole* data set at once — it turns a single
> decoded structure into a survey, surfaces the outliers that teach you the variants, and builds the
> file-by-file catalogue the glossary needs.

[← C4.5 — Validation harnesses](05-validation.md) · [Chapter 4 hub](C4-Byte-Level-Toolcraft.md) ·
[Next chapter: C5 — Textures →](../C5-Textures-TPK/C5-Textures-TPK.md)

---

## Why batch, not one-off

Working one file at a time answers "what is *this*?" Batch reconnaissance answers "what is *everything*,
and what's unusual?" — which is a different and often more valuable question. The retail data set is on
the order of a few thousand files; a script that visits all of them in a pass will find the three that
break your assumptions far faster than you'll stumble on them by hand. Nearly every structural fact in
this book was first *noticed* in a batch run and then *confirmed* on a single file.

## The identification sweep

The foundational batch job is: walk the tree, decompress where needed, and classify every file
([C1.9](../C1-EAGL-Container-Model/09-universal-opener.md)). The output is a census — how many chunk
trees, vaults, banks, DDS, and unknowns exist, and where.

```python
import os, collections
from mw import open_eagl

def census(root):
    counts = collections.Counter(); unknown = []
    for dp, _, files in os.walk(root):
        for fn in files:
            p = os.path.join(dp, fn)
            try:
                kind, _ = open_eagl(p)
            except Exception as e:
                kind = f'error:{type(e).__name__}'
            counts[kind] += 1
            if kind in ('unknown', 'chunks') and fn.lower().endswith('.bin'):
                unknown.append(p)   # .bin is overloaded — worth a closer look
    return counts, unknown
```

Run it once and you know the shape of the whole data set. Run it again after you teach `mw` a new format
and watch the `unknown` bucket shrink — a satisfying, measurable proxy for how much of the game you've
decoded.

## Chunk-ID frequency: a map of the unknown

A second high-value sweep tallies every chunk ID across every chunk-tree file. IDs you already understand
confirm your coverage; IDs you *don't* — especially high-frequency ones — are the next things worth
decoding, ranked by how much of the data they account for.

```python
def id_histogram(paths):
    hist = collections.Counter()
    for p in paths:
        kind, buf = open_eagl(p)
        if kind != 'chunks': continue
        from mw import walk_tree
        for _, _, cid, size, _ in walk_tree(buf):
            hist[cid] += 1
    return hist          # sort by count: the biggest unknowns are the best targets
```

This is prioritisation by evidence: rather than decoding whatever you stumble on, you attack the chunk
types that appear most, getting the most explanatory power per hour. It is how a coverage plan writes
itself.

## Outliers are the payload

The real prize of a batch run is the *exceptions*. When you run a hypothesised parser across every
instance of a structure ([C4.2](02-decoding-unknowns.md)), most files confirm it and a few don't. Those
few are never noise:

- A file whose record count is off by one reveals an optional header field.
- A file whose stride doesn't divide reveals a *second variant* of the format (the standard-vs-compressed
  TPK split of [C5](../C5-Textures-TPK/C5-Textures-TPK.md) is exactly this kind of discovery).
- A file that errors reveals an edge case your reader must handle — or a genuinely different format wearing
  the same extension.

Chase every outlier to ground. Excluding it to make your parser "pass" is how a wrong layout survives; the
outlier is the data trying to correct you.

## Building the file catalogue

Batch recon is also how the glossary's file-by-file catalogue gets built ([Glossary](../Glossary/README.md)).
A single pass can emit, per file: its identified kind, its size, its top-level chunk IDs (or vault
collections, or bank entries), and any names it spells out as text. Aggregate that and you have a complete,
evidence-backed inventory of the game — not a hand-written list that rots, but a generated one you can
re-run when a fact changes.

## Why this scales your knowledge

One decoded structure, applied by hand, explains one file. The *same* structure, applied in batch,
explains every file that contains it and flags every place it varies — turning a local decode into global
knowledge, and turning "I think this is the layout" into "this layout holds across all 340 instances, with
these 3 documented exceptions." That last sentence is a ✅ fact with its uncertainty fully accounted for,
which is exactly the standard the book holds itself to ([C4.5](05-validation.md)).

## Bending it — run wide, then run again

- **Sweep the whole corpus, early and often.** The census and the ID histogram are cheap and orient
  everything else.
- **Rank decoding targets by frequency.** Attack the chunk types that account for the most data first.
- **Treat the outlier list as your to-do list.** Each exception is a variant to document or a bug to fix.
- **Regenerate the catalogue, don't maintain it by hand.** A generated inventory stays true; a hand-written
  one drifts the moment the data or your understanding changes.

---

### Chapter 4 — where you've arrived

You have a tested core library, a repeatable method for decoding the undocumented, the two highest-yield
techniques (hex-diffing and static analysis) and the judgement of when to use each, validation harnesses
that make correctness measurable, and the batch discipline that scales one decode into a survey. This is
the workshop. From here the book takes apart real formats — and every one of them was decoded with exactly
these tools.

**Back to:** [Chapter 4 hub](C4-Byte-Level-Toolcraft.md) ·
**Next chapter:** [C5 — Textures: the TPK Container Model](../C5-Textures-TPK/C5-Textures-TPK.md)
