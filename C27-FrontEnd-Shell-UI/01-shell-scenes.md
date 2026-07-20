# C27.1 — The Shell: Scenes & Atlases

> **The one-sentence version:** the front-end is a set of data-driven scenes — title, menu, garage, map,
> results — each a layout of UI elements that reference images in atlases and text by label, drawn by the shell
> engine.

[← Chapter 27 hub](C27-FrontEnd-Shell-UI.md) · [Next: C27.2 — UI atlases →](02-ui-atlases.md)

---

## Scenes are the screens

The "shell" is the front-end's set of **scenes** — one per screen the player navigates:

- **Title / attract** — the entry screen (with the attract movie, [Chapter 23](../C23-Video-VP6/C23-Video-VP6.md)).
- **Main menu** — the top-level navigation.
- **Garage / customization** — car selection and tuning UI.
- **Map** — the world map with the minimap data ([Chapter 29](../C29-Minimap-Map-Data/C29-Minimap-Map-Data.md)).
- **Results / progression** — race and career screens.

Each scene is *data*: a description of what elements appear, where, drawn from which atlas art, showing which
labelled text. The shell engine loads the active scene and renders it, and navigation swaps scenes.

## A scene is a layout of elements

A scene is built from UI **elements** — buttons, panels, icons, text boxes, backgrounds — each positioned and
bound to content:

```
Scene "MainMenu"
├── background   → atlas region (a full-screen image)
├── button "Start" → atlas region + label MENU_START (C27.5)
├── button "Options" → atlas region + label MENU_OPTIONS
├── car preview   → a 3D render slot
└── …
```

Each element references its **imagery** (an atlas region, [C27.3](03-layout.md)) and, where it shows text, a
**label** ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)). The scene is the arrangement;
the atlases and labels are the content it arranges.

## Data-driven UI

The front-end is **data-driven**: the screens are authored as data, not hard-coded, so the interface can be
changed by editing data rather than recompiling the game. This is the same philosophy as the rest of the engine
(the vault, [Chapter 12](../C12-Reflection-Schema/C12-Reflection-Schema.md); triggers,
[Chapter 17](../C17-Triggers-Barriers/C17-Triggers-Barriers.md); event scripts,
[Chapter 25](../C25-NIS-Events/C25-NIS-Events.md)): behaviour and content live in editable data, and a fixed
engine interprets them.

- **The scenes are data** — the layouts of elements.
- **The atlases are data** — the UI art ([C27.2](02-ui-atlases.md)).
- **The labels are data** — the text ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)).
- **The shell engine is fixed** — it draws whatever scenes reference.

## Where it ships

The front-end lives in `FRONTEND/`: `FRONTA.BUN`, the JDLZ-compressed `FrontB.lzc`
([Chapter 3](../C3-Compression-JDLZ/C3-Compression-JDLZ.md)), and `PLATFORMS/` (platform-specific UI). The
scenes and their atlases are packed here; decompressing and walking the bundle reveals the TPK atlases
([C27.2](02-ui-atlases.md)) and the layout data ([C27.3](03-layout.md)) the scenes are built from.

> 🟡 *Reasoned:* the scene/element model is the standard data-driven-UI shape and is consistent with the
> front-end's atlas+label building blocks; the atlas (TPK), compression (JDLZ), and label
> ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)) primitives it uses are verified.

## Editing implications

- **Edit scenes to rearrange the UI** — move/add/remove elements in the layout data
  ([C27.3](03-layout.md), [C27.6](06-editing-ui.md)).
- **Edit atlases to re-skin** — change the UI art without touching layouts ([C27.2](02-ui-atlases.md)).
- **Edit labels/strings to change text** — without re-authoring scenes
  ([Chapter 30](../C30-Localization-Labels/C30-Localization-Labels.md)).
- **The three are independent** — layout, art, and text can each change without the others, the payoff of the
  data-driven split.

---

### Key takeaways

- The shell is data-driven **scenes** (title, menu, garage, map, results), each a layout of UI elements.
- An element references its **imagery** (an atlas region) and, for text, a **label**.
- Scenes, atlases, and labels are all **data**; the shell engine that draws them is fixed.
- The front-end ships in `FRONTEND/` (`FRONTA.BUN`, JDLZ `FrontB.lzc`, `PLATFORMS/`).
- Layout, art, and text edit independently — rearrange scenes, re-skin atlases, or re-word labels separately.

**Continue:** [C27.2 — UI atlases](02-ui-atlases.md) · [Chapter 27 hub](C27-FrontEnd-Shell-UI.md)
