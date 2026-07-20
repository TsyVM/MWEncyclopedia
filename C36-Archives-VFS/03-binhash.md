# C36.3 — The BinHash

> **The one-sentence version:** the VFS key is the BinHash of a resource's path — the engine's path-hashing
> function that turns a path string into the 32-bit key the loader indexes by, so resources are hash-identified
> internally and path-authored externally.

[← C36.2 — The virtual file system](02-vfs.md) · [Chapter 36 hub](C36-Archives-VFS.md) ·
[Next: C36.4 — MemoryFile intercept →](04-memoryfile.md)

---

## The path hash

The VFS ([C36.2](02-vfs.md)) keys resources by the **BinHash** of their path — the game's function for turning a
path string into a 32-bit key:

```
BinHash("GLOBAL\CARS\COBALTSS\GEOMETRY.BIN") → 0x……   (the VFS lookup key)
```

Every resource's VFS identity is this hash. So the loader doesn't compare path strings — it hashes the requested
path once and looks up the resulting key ([C36.2](02-vfs.md)). This is the file-loading member of the engine's
hash family ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)): names hashed to keys for
compact, fast reference.

## Which hash

The engine uses a small set of string hashes ([Chapter 2](../C2-Identifiers-And-Hashing/C2-Identifiers-And-Hashing.md)):
the **reflection hash** (`lookup2`/`0xABCDEF00`) for vault fields and class names
([C32.4](../C32-Runtime-Class-System/04-registration.md)), and the **Bin hash** (an FNV-style multiply-xor) used
for asset/path keys. The **BinHash** for VFS paths is the latter family — a path-string hash distinct from the
reflection hash. So the engine's *two hash worlds* ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)) are
joined here: reflection (fields/classes) and asset/path (textures, geometry names, and VFS paths).

- **Reflection hash** — vault fields, class names (recoverable, [C12.1](../C12-Reflection-Schema/01-reflection-hash.md)).
- **Asset/BinHash** — texture keys ([C5.6](../C5-Textures-TPK/06-the-texture-key.md)), geometry names
  ([C8.3](../C8-Geometry-Solids/03-object-hash.md)), and VFS paths.

> 🟡 *Reasoned:* the BinHash is the asset/path-hash family (distinct from the reflection hash), consistent with
> the two-hash-world model ([C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)); the VFS resolving path →
> BinHash key is documented. The exact BinHash algorithm for paths shares the asset-hash family's properties
> (deterministic, path-derived).

## Case and separator normalisation

Path hashing must be **stable** across the ways a path can be written, so the BinHash typically normalises before
hashing:

- **Case** — Windows paths are case-insensitive, so `GEOMETRY.BIN` and `geometry.bin` should hash the same
  (lowercase or uppercase before hashing).
- **Separators** — `\` vs `/` normalised to one form.
- **Prefixes** — a consistent root so `GLOBAL\...` and a fully-qualified path resolve alike.

Without normalisation, the same resource written two ways would hash to two keys and one wouldn't resolve. So the
BinHash canonicalises the path first — the same care the texture/geometry asset hashes take with their name forms
([C5.6](../C5-Textures-TPK/06-the-texture-key.md)).

## Compact, fast, name-authored

The BinHash gives the VFS the standard hash-key benefits ([C36.2](02-vfs.md)):

- **Compact references** — a 32-bit key per resource, not a path string, throughout the runtime.
- **Fast lookup** — a hash-table probe ([C8.6](../C8-Geometry-Solids/06-lookup.md)), not a directory walk.
- **Name-authored** — content is authored with readable paths; the BinHash converts them at load.

So authoring uses paths, the runtime uses hashes, and the BinHash is the bridge — the same author-by-name,
resolve-by-hash pattern as the vault ([C11.2](../C11-Attribute-Vaults/02-erts-strings.md)) and class registry
([C32.4](../C32-Runtime-Class-System/04-registration.md)).

## RE implications

- **VFS keys are path BinHashes** — a resource's runtime identity is its path-hash.
- **BinHash is the asset/path-hash family** (distinct from the reflection hash,
  [C7.5](../C7-Materials-TexAnim/05-two-hash-worlds.md)).
- **Paths are normalised** (case/separator) before hashing — account for it when reproducing keys.
- **Author by path, resolve by hash** — the BinHash bridges readable paths and runtime keys.

---

### Key takeaways

- The VFS key is the **BinHash** of a resource's path — a path string → 32-bit key.
- It's the **asset/path-hash family** (distinct from the reflection hash) — joining the engine's two hash worlds
  at the VFS.
- Paths are **normalised** (case, separators) before hashing so a resource hashes stably however it's written.
- BinHash gives compact, fast, name-authored references — the author-by-name, resolve-by-hash pattern for files.
- A resource's runtime identity is its path-hash; author by path, the VFS resolves by BinHash.

**Continue:** [C36.4 — MemoryFile intercept](04-memoryfile.md) · [Chapter 36 hub](C36-Archives-VFS.md)
