# C2.2 — The Reflection Hash: lookup2 with Seed `0xABCDEF00`

> **The one-sentence version:** every key in the attribute/vault/reflection system — class names, car
> names, field names, enum values — is a Jenkins **lookup2** hash seeded with `0xABCDEF00`, and this is
> not inferred from matching outputs but read directly out of `speed.exe` and proven three ways.

[← C2.1 — Joaat](01-joaat-asset-hash.md) · [Chapter 2 hub](C2-Identifiers-And-Hashing.md) ·
[Next: C2.3 — The Bin sum hash →](03-bin-sum-hash.md)

---

## Why this page exists

For years the attribute vault's record keys resisted decoding because everyone tried the obvious
candidates — Joaat, the Bin sum, CRC32 — and none matched. They don't match because the reflection
system uses a *different* Jenkins hash (`lookup2`, not one-at-a-time) with a *non-zero seed*
(`0xABCDEF00`). Getting this one function right is what turns the vault from an opaque blob into a
readable, editable database. It deserves its own page because it is the linchpin of every attribute
chapter (C11–C14) and because the proof is worth seeing in full — this is the standard of evidence the
rest of the book aims for.

## The function, read from the executable

The public entry point at `0x005CC240` takes a `char*`, computes its length with an inline `strlen`,
and tail-calls the mixing core with the seed placed in `edx`:

```
005cc240: 8b442404       mov  eax, [esp+4]        ; eax = key pointer
005cc244: 85c0           test eax, eax
005cc246: 7423           je   0x5cc26b            ; null  -> return 0
005cc248: 803800         cmp  byte [eax], 0
005cc24b: 741e           je   0x5cc26b            ; empty -> return 0
005cc24d: 8bc8           mov  ecx, eax
005cc250: 8d7101         lea  esi, [ecx+1]        ; ---- inline strlen ----
005cc253: 8a11           mov  dl, [ecx]
005cc255: 41             inc  ecx
005cc256: 84d2           test dl, dl
005cc258: 75f9           jne  0x5cc253
005cc25a: 2bce           sub  ecx, esi
005cc25d: 894c2404       mov  [esp+4], ecx        ; store length as the 2nd arg
005cc261: ba00efcdab     mov  edx, 0xabcdef00     ; <<< THE SEED
005cc266: e925feffff     jmp  0x5cc090            ; tail-call the lookup2 core
005cc26b: 33c0           xor  eax, eax
005cc26d: c3             ret
```

The core at `0x005CC090` is textbook lookup2: it loads the golden-ratio constant `0x9E3779B9` into the
`a`/`b` accumulators, seeds `c` from `edx`, and processes the key **twelve bytes at a time** (note the
`cmp eax, 0xC` block-size test and the `0xAAAAAAAB` reciprocal-multiply used to divide the length by 12):

```
005cc090: 53             push ebx
005cc091: 55             push ebp
005cc092: 56             push esi
005cc093: 8bd8           mov  ebx, eax            ; ebx = key ptr
005cc095: 8b442410       mov  eax, [esp+0x10]     ; eax = length
005cc099: 83f80c         cmp  eax, 0xc            ; >= 12 bytes? enter the block loop
005cc09c: b9b979379e     mov  ecx, 0x9e3779b9     ; a = b = golden ratio
005cc0a6: 8bf2           mov  esi, edx            ; c = seed (0xABCDEF00)
005cc0ae: b8abaaaaaa     mov  eax, 0xaaaaaaab     ; length / 12 via reciprocal multiply
005cc0b3: f7e5           mul  ebp
005cc0b5: c1ea03         shr  edx, 3
```

Twelve-bytes-per-round processing, the `0x9E3779B9` constant, and a length-keyed tail switch are the
exact, unmistakable signature of Jenkins lookup2. The only customisation is the seed. (One-at-a-time, by
contrast, would show a per-byte `<<10 / >>6` loop and no golden-ratio constant — see
[C2.1](01-joaat-asset-hash.md).)

## The byte-faithful port

This reproduces the executable's output exactly. It is the implementation used for every reflection hash
in the book.

```python
M = 0xFFFFFFFF

def _mix(a, b, c):
    a = (a - b) & M; a = (a - c) & M; a ^= (c >> 13)
    b = (b - c) & M; b = (b - a) & M; b ^= ((a << 8)  & M)
    c = (c - a) & M; c = (c - b) & M; c ^= (b >> 13)
    a = (a - b) & M; a = (a - c) & M; a ^= (c >> 12)
    b = (b - c) & M; b = (b - a) & M; b ^= ((a << 16) & M)
    c = (c - a) & M; c = (c - b) & M; c ^= (b >> 5)
    a = (a - b) & M; a = (a - c) & M; a ^= (c >> 3)
    b = (b - c) & M; b = (b - a) & M; b ^= ((a << 10) & M)
    c = (c - a) & M; c = (c - b) & M; c ^= (b >> 15)
    return a & M, b & M, c & M

def lookup2(key, initval=0xABCDEF00):
    if isinstance(key, str): key = key.encode('latin1')
    a = b = 0x9E3779B9
    c = initval & M
    length = len(key); k = 0; ln = length
    while ln >= 12:
        a = (a + (key[k]   | key[k+1]<<8  | key[k+2]<<16  | key[k+3]<<24))  & M
        b = (b + (key[k+4] | key[k+5]<<8  | key[k+6]<<16  | key[k+7]<<24))  & M
        c = (c + (key[k+8] | key[k+9]<<8  | key[k+10]<<16 | key[k+11]<<24)) & M
        a, b, c = _mix(a, b, c); k += 12; ln -= 12
    c = (c + length) & M
    if ln >= 11: c = (c + (key[k+10] << 24)) & M
    if ln >= 10: c = (c + (key[k+9]  << 16)) & M
    if ln >= 9:  c = (c + (key[k+8]  << 8))  & M
    if ln >= 8:  b = (b + (key[k+7]  << 24)) & M
    if ln >= 7:  b = (b + (key[k+6]  << 16)) & M
    if ln >= 6:  b = (b + (key[k+5]  << 8))  & M
    if ln >= 5:  b = (b +  key[k+4])         & M
    if ln >= 4:  a = (a + (key[k+3]  << 24)) & M
    if ln >= 3:  a = (a + (key[k+2]  << 16)) & M
    if ln >= 2:  a = (a + (key[k+1]  << 8))  & M
    if ln >= 1:  a = (a +  key[k])           & M
    a, b, c = _mix(a, b, c)
    return c
```

The returned value is `c` (the third accumulator), which is why the seed lands in `c` and the length is
folded into `c` before the tail. The tail `switch` on `length % 12` — the eleven `if ln >= …` lines — is
the direct transcription of the length-keyed jump table at the end of the core routine.

## The proof — three independent sources, one number

`lookup2("default")` computes to **`0xEEC2271A`**. That same 32-bit value is:

1. **Compiled into the executable** as an immediate, the sentinel for the "default" key:
   ```
   006a5baf: b81a27c2ee     mov eax, 0xeec2271a
   ```
2. **Present throughout the shipped data** — it occurs **1071 times** in `GLOBAL/attributes.bin`, as the
   parent/`default` key of records (the value a field inherits from when it isn't overridden).
3. **Reproducible on demand** — the port above returns it for the input `"default"`.

A value we computed, a value the compiler emitted, and a value the data ships with — three sources that
could only agree if the function is exactly lookup2 with seed `0xABCDEF00`. This is ✅ verified, not
reasoned.

## What it unlocks

With the correct function, a large majority of the name-slots in `attributes.bin` resolve to readable
strings purely statically — you forward-hash a dictionary of candidate class/field/enum names and match
(the mechanics are [C2.4](04-hash-resolution.md), and the vault structure that consumes these keys is
[Chapter 11](../C11-Attribute-Vaults/C11-Attribute-Vaults.md)). Everything editable in car tuning
([C13](../C13-Vault-CarTuning/C13-Vault-CarTuning.md)) and pursuit tuning
([C14](../C14-Vault-Pursuit-Surfaces/C14-Vault-Pursuit-Surfaces.md)) is reachable because this one
function is pinned down.

## Bending it — cautions

- **Use it only for reflection keys.** Class names, collection (car) names, field names, enum values.
  For textures/solids/effects use Joaat ([C2.1](01-joaat-asset-hash.md)).
- **Lower-case the key.** The reflection system interns lower-cased strings; hash `name.lower()`.
- **Don't substitute a generic lookup2.** A stock lookup2 with the default seed `0` will *not* match —
  the seed `0xABCDEF00` is load-bearing. Verify your port with `lookup2("default") == 0xEEC2271A` before
  trusting it.

---

**Continue:** [C2.3 — The Bin sum hash](03-bin-sum-hash.md) · [Chapter 2 hub](C2-Identifiers-And-Hashing.md)
