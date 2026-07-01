# Eco WorldGen ore visualizer

A single-page, zero-dependency tool that reads an [Eco](https://play.eco) `WorldGenerator.eco` config and charts **where each material spawns**, by biome and by world height (Y). Everything runs client-side in the browser — nothing is uploaded anywhere.

**Live:** https://nielsh.github.io/eco-biome-visualizer/

A default world (`WorldGenerator.eco.template`) is charted on load. Drop, upload, or paste your own config to replace it.

## What it shows

- **Materials:** Iron, Copper, Gold, Coal, Clay, Sulfur, Peat.
- **Two groupings** (toggle): by **ore** (compare biomes for one material — a prospecting map) or by **biome** (a vertical slice of one biome).
- **Two styles** (toggle): **violins** (density — width/shade show how rich a material is at that height) or **bars** (just the extent).
- **Hover** any point for a **global density index (0–100)** so you can compare across biomes and materials even though the violin widths are normalized per column.
- **Export** the chart as SVG or PNG.
- Biomes the loaded config doesn't generate (Voronoi weight `0`) are dimmed and marked `*`.

## How it works

1. The config JSON is parsed in the browser (resolving its `$id`/`$ref` reference graph).
2. It walks `TerrainModule.Modules` (one per biome) and their `BlockDepthRanges` → `SubModules`, pulling every occurrence of a tracked material — as scatter layers (`StandardTerrainModule`), deposits (`DepositTerrainModule`), or primary strata (mainly clay/peat).
3. Each occurrence becomes a **depth-density profile**. Deposits are spread over an estimated vertical extent derived from their `DirectionWeights` (a flat, laterally-growing deposit reads short & dense; a vertical one reads tall & thin).
4. Each profile is smeared across the biome's surface-height range (from the config's `WaterLevel`/`MaxGenerationHeight` and the engine's per-biome elevation bands) to place it on the absolute-Y axis.

## Caveats

The density value is a **relative model index**, not literal blocks-per-chunk:

- It's derived from spawn rate × average deposit size, spread over an estimated vertical extent — good for *comparing* where material concentrates, not for counting blocks.
- The X/Z spread of deposits is approximated from mean direction weights (an ellipsoid), ignoring per-deposit weight variance and edge penalties.
- Clay/peat **strata** are approximated as flat bands over their depth range.
- Violin widths are normalized **per column** (use the hover index for cross-column comparison) and drawn on a perceptual √ scale.

## Usage locally

Because the default world is loaded with `fetch`, opening `index.html` straight from disk (`file://`) will **not** auto-load it (browsers block `fetch` on `file://` origins) — the drop/paste UI still works. To preview the default world locally, serve the folder over HTTP, e.g.:

```
python -m http.server
```

then open http://localhost:8000/.

## Development

It's one file: `index.html` (HTML + CSS + vanilla JS, no build step, no dependencies). See [`.claude/CLAUDE.md`](.claude/CLAUDE.md) for a full developer guide — architecture, the data model, and exactly which tables to edit to add a material or biome.

## Data & credits

`WorldGenerator.eco.template` and the world-generation config format are from Eco by [Strange Loop Games](https://strangeloopgames.com). This tool only reads those configs; it contains no game code. Built with [Claude Code](https://claude.com/claude-code).
