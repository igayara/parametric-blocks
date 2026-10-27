# Release notes

## 0.1.1

- **Undo works for changes made in the Properties panel.** Each change is one Undo step. Before, Undo skipped them, and undoing earlier steps (a copy, a delete) could leave blocks in the wrong state.
- The Package Manager now finds the plug-in when you search for "parametric blocks".

## 0.1.0

Initial release:

- **Parametric Block Definitions.** Any Grasshopper definition with its parameters in a `PB:IN` group and its geometry in `PB:OUT` can be embedded in the Rhino file ([authoring guide](docs/definition-authoring.md)). `PBImport` checks it and offers to fill in the metadata it needs.
- **Blocks that behave like blocks.** `PBInsert` places them, the Properties panel changes their parameters (with multi-selection support), and `PBManager` lists the definitions like Block Manager. Move, copy, array, mirror and export work as they do for any block.
- **Geometry parameters** (a curve, points) are edited in place: double-click the block, use any Rhino tool, then **Done**. `PBExtract` copies them back into the model.
- **Files stay safe without the plug-in.** Rhino without Parametric Blocks shows ordinary blocks with the right geometry, and nothing is lost when it saves the file.
- `PBUpdate` brings a new version of a definition into a file and migrates its blocks. Also `PBSelectSimilar`, `PBDetach`, `PBRegenerate`, `PBPurge`, `PBInfo` and `PBOptions`.

Known limitations:
- Windows and Rhino 8 (8.35 or later) only, with Grasshopper 1 definitions.
- Text, dimensions, leaders and hatches can't be outputs yet. They are skipped with a warning.
- Every part of a block takes its attributes from the block (By Parent). Per-part colours and materials will come later.
- A parametric block nested inside an ordinary block isn't updated by `PBUpdate` or `PBRegenerate`.
- Plane, Vector and Box parameters aren't supported: import rejects them.
- A definition that uses other Grasshopper plug-ins needs them on every computer that changes its blocks. Without them the blocks keep their last geometry.
