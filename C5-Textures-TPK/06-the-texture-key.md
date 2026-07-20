# C5.6 — The Texture Key, Honestly

> **The one-sentence version:** every texture carries a 32-bit key at entry `+0x24` that is a **stable,
> deterministic hash of its name** — proven by 2,012 names that recur across packs with byte-identical keys
> — yet it matches **no** standard runtime string hash, which tells us it is minted by the offline asset
> packer and only ever *read* by the game; so treat it as an opaque-but-reproducible identifier and resolve
> textures by stored key.

[← C5.5 — Extracting & replacing](05-extract-replace.md) · [Chapter 5 hub](C5-Textures-TPK.md) ·
[Next: Chapter 6 — Texture Codecs →](../C6-Texture-Codecs/C6-Texture-Codecs.md)

---

## What the key is

Each TPK texture is identified inside the pack by a 32-bit **key**, stored in the entry at `+0x24` and
mirrored in the hash table (`0x33310002`), a flat list of `{key, pad}` pairs — one per texture, in entry
order. To find a texture by key you scan the hash table for a match and take that index into the entry
table. Verified against `GLOBALMESSAGE`:

| # | Name | Key (`+0x24`) |
|---|------|--------------:|
| 0 | `MW_LOGO`         | `0x0CD55E13` |
| 1 | `COP_LIGHT`       | `0x522D4938` |
| 2 | `FONT_MW_BODY`    | `0x545570C6` |
| 3 | `COP_LIGHT_FLASH` | `0x94B4E685` |
| 4 | `BASEPOLY`        | `0xC6AFDD7E` |

The key is what other systems reference when they want *this* texture — a material's texture reference
([Chapter 7](../C7-Materials-TexAnim/C7-Materials-TexAnim.md)) carries a key like this, and resolving it
means "find the texture in the bound pack whose stored key equals this value."

## Why the key — not the name — is the real identity

The entry's name field is only **24 bytes** (`+0x0C` through `+0x23`, immediately before the key at
`+0x24`). Names that fit are stored null-terminated; names longer than 24 characters are **truncated on
disk**. That truncation is decisive: two distinct textures whose full names share a 24-character prefix
would carry the *same* stored name, so the engine cannot use the visible name as an identity. The 32-bit
key can — and does. This is why every cross-reference in the engine keys on the hash, never the string.

## Proof that the key is a deterministic name hash

If the key were an arbitrary handle assigned at build time, the same texture name could carry different
keys in different packs. It does not. Scanning every uncompressed pack in the game and collecting **short,
untruncated** names (so the full name is unambiguous), this book found:

> **2,012 distinct names that appear in two or more packs — and in 100 % of cases the key is byte-identical
> across every pack. Zero conflicts.**

A few examples, each seen in multiple packs with one stable key:

| Name | Key | Packs |
|------|----:|:-----:|
| `MILESTONE_RACING`   | `0x054F9862` | 2 |
| `BUTTON_RING`        | `0xC4F4EBEA` | 2 |
| `ARC_BROWNBRICKWALL` | `0x3C1B1EB7` | 2 |
| `SGN_FWY_BORDER_EA`  | `0x3DF7F09E` | 2 |

Same name ⇒ same key, everywhere. The key is therefore a **pure, deterministic function of the name** — a
hash, not a counter or a slot index.

## Proof that it is *not* a standard hash

The obvious next assumption — and the claim in the older record — is that the key is `Joaat(name)`, the
engine's common string hash ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)). It
is not. This book tested an exhaustive battery of hash functions against thousands of real name→key pairs
(and against a hand-checked set of eight), across multiple input encodings, and **nothing matched**:

| Hash family tried | Input variants tried | Matches |
|---|---|:--:|
| Joaat (seed 0 and `0xFFFFFFFF`) | id / lower / upper / null-terminated / padded to 8·16·20·24·32 | 0 |
| Bin / FNV-1 / FNV-1a | id / lower / upper | 0 |
| CRC-32 (8 polynomials × init × reflect-in/out × xor-out) | id / lower / upper / null-terminated | 0 |
| djb2, djb2-xor, sdbm, ELF | id / lower / upper | 0 |
| lookup2 (seed 0 and `0xABCDEF00`) | id / lower / upper / UTF-16LE | 0 |
| lookup3 / jhash (several seeds) | id / lower | 0 |
| single-multiplier `h*C±c` sweep (C = 1…1023, four forms) | id | 0 |

For the worked pack, the concrete near-misses make the point:

| Hash of `"MW_LOGO"` | Result | = `0x0CD55E13`? |
|---|---:|:--:|
| Joaat        | `0x5F0C0917` | no |
| Bin / FNV    | `0x3556093B` | no |
| lookup2/0    | `0xFDD99D68` | no |
| lookup2/seed | `0xCA425130` | no |

> ✅ *Verified:* the key exists at `+0x24`, indexes the hash table, and is a deterministic function of the
> name (2,012 cross-pack names, 100 % consistent).
> ✅ *Verified negative:* it is none of the standard engine/runtime string hashes on any tested input form.
> The old "TPK nameHash = Joaat(name)" claim is **false** and must not be relied on.

## What this tells us about where the key comes from

Put the two proven facts together — *deterministic from the name*, yet *not any hash the runtime uses* — and
the explanation follows: the key is minted by the **offline asset packer** that builds the `.BUN` files, not
by the shipped game. The runtime only ever **reads** the stored key and **compares** it; it never needs to
re-derive a key from a name at play time, because every reference on disk is already keyed. That is exactly
why probing the runtime's hash functions never reproduces the value — the generating routine lives in EA's
build tools, which did not ship in the executable.

> 🟡 *Reasoned (from two ✅ facts):* the key is a build-tool hash, external to the runtime. This is the
> simplest hypothesis consistent with "deterministic per name" + "unmatched by every runtime hash," but the
> exact packer algorithm is not recoverable from shipped data alone.

## Why none of this blocks editing

For essentially everything you do with a TPK, the preimage is irrelevant, because the key is stored — you
read it rather than compute it:

- **Enumerate** — names and keys are already in the entry table.
- **Look up** — scan the hash table for the stored key; no re-hashing.
- **Extract / replace pixels** ([C5.5](05-extract-replace.md)) — keyed lookup finds the entry; offsets do
  the rest. You never regenerate the key.
- **Preserve references** — because the key is stable and stored, keeping it byte-identical across an edit
  keeps every material reference valid automatically.

The one operation that would want the preimage — inventing a brand-new texture from a chosen name and
computing its "correct" key — has a robust workaround: clone an existing entry, keep (or deliberately
choose) a key, and make every reference agree. References resolve by *stored* key, so an internally
consistent pack loads regardless of whether the key matches any formula.

## Working rules for the key

- **Read the key; never assume it.** Take the 32-bit value from entry `+0x24` (or the hash table) as ground
  truth. Do not compute it from the name.
- **Treat it as opaque and stable.** It is a stable identifier for cross-references, not something to reverse
  into a string.
- **Preserve it on edits.** Changing pixels must not change the key, or you silently break every material
  that references the texture.
- **For new textures, choose consistency over correctness-of-formula.** Pick a key, use it in the entry and
  the hash table, and make every reference agree.
- **Don't propagate the old "Joaat(name)" claim.** It fails against retail data; tooling that assumes it
  will mis-key real packs.

---

### Key takeaways

- The texture key lives at entry `+0x24` and is the value the hash table (`0x33310002`) indexes by.
- Entry names are capped at 24 bytes and truncate on disk, so the key — not the visible name — is the real
  identity.
- The key is a **deterministic name hash**: 2,012 names shared across packs all carry identical keys, zero
  conflicts.
- It matches **no** standard hash (Joaat, Bin/FNV, CRC-32 across 8 polynomials, djb2/sdbm/ELF, lookup2/3) on
  any tested input form — the old "Joaat(name)" claim is false.
- Best explanation: the key is minted by the offline packer and only read/compared at runtime — so for
  reading, extracting, and replacing you simply use the stored value.

**Continue:** [Chapter 6 — Texture Codecs](../C6-Texture-Codecs/C6-Texture-Codecs.md) ·
[Chapter 5 hub](C5-Textures-TPK.md)
