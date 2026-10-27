# Parametric Blocks for Rhino

Parametric Blocks extend Rhino's Blocks with parameters. Their geometry is computed by a Grasshopper definition embedded in the Rhino file. 

- **They are real Rhino blocks.** Move, copy, array, mirror, Make2D, render and export them like any block. Users opening the file without the plug-in installed will see ordinary blocks with the right geometry, and nothing is lost when they save.
- **The definition travels with the file.** No linked .gh files to manage (and lose).
- **Feels like Rhino.** The plugin aims to feel intuitive and familiar. Insert, a manager like Block Manager, parameters in the Properties panel, double-click to edit.

Free to use ([licence](LICENSE.md)). Rhino 8 for Windows, service release 8.35 or later. What changed in each version: [release notes](CHANGELOG.md).

## Install

1. In Rhino, run `PackageManager` and search for `parametric-blocks` (with the hyphen).
2. Select it and click **Install**.
3. Restart Rhino and run `PBManager`.

Updates appear in the Package Manager too.

## Getting started

1. `PBManager` opens the Parametric Blocks panel. Import a definition: try the one in [`samples`](samples).
2. `PBInsert` places a block: choose the definition, set its parameters, pick a point.
3. Select blocks, then use the **Parametric Block** page of the Properties panel to change their parameters.
4. Double-click a block with geometry parameters to edit that geometry with any Rhino tool. **Done** applies it as one Undo step; `PBExtract` copies that geometry back into the model.

## Make your own

Any Grasshopper definition can become a Parametric Block Definition by grouping its inputs and outputs: see the [authoring guide](docs/definition-authoring.md). The plugin also needs a metadata panel to properly process the definition, but that can be configured on import.

## Bug reports & feature requests

[Open an issue](../../issues/new/choose). Include the diagnostics (the `PBInfo` command copies them to the clipboard; so does "Copy diagnostics" in Options › Parametric Blocks) and, if you can, a small file that shows the problem.

## Support

This plug-in is free. If it has been useful to you, please consider [making a donation on Ko-fi](https://ko-fi.com/rafaigayara).
