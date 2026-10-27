# Authoring a Parametric Block Definition

A Parametric Block Definition is a Grasshopper 1 definition (.gh) with three named groups. The plugin embeds it in the .3dm and solves it to build each parametric block. It reads only what is inside the groups, as described here; everything else on the canvas is your logic, grouped or not.

| Group | Holds | Required |
|---|---|---|
| `PB:META` | One Panel: the definition's identity | Yes (import can create it) |
| `PB:IN` | The parameters users set per block | No: without it the block is fixed |
| `PB:OUT` | The block's geometry, and optionally messages | Yes |

Each group must appear only once. Group names are matched ignoring case and surrounding spaces.

A complete definition at a glance (the bookshelf used in the examples below, [`samples/Bookshelf.gh`](../samples/Bookshelf.gh)):

```
PB:META   Panel   name: Bookshelf
                  id: 2f8d0c1a-7b3e-4e5f-9a1b-6c4d2e8f0a11
                  version: 1
                  units: cm

PB:IN                                               PB:OUT
  Width     Number Slider ─┐                     ┌─  Body           Brep param
  Shelves   Number Slider ─┼──  your logic   ────┼─  Plan outline   Curve param
  Finish    Value List    ─┘                     └─  Messages       Text param
```

## Identity: `PB:META`

One Panel in the `PB:META` group, with one `key: value` per line. Keys are case-insensitive, lines starting with `#` are comments, and unknown keys are ignored.

```
name: Bookshelf
id: 2f8d0c1a-7b3e-4e5f-9a1b-6c4d2e8f0a11
version: 1
units: cm
description: Wall bookshelf sized by width and number of shelves
origin: back-left corner in plan, at floor level; the front faces +Y
```

| Key | | Rule |
|---|---|---|
| `name` | Required | Shown everywhere and used in block and file names, so it can't contain `\ / : * ? " < > \|`. |
| `id` | Required | A GUID made once (`[guid]::NewGuid()` in PowerShell) and never changed. It's how a later file is recognised as a new version of this definition rather than a different one. |
| `version` | Required | An integer, starting at 1. Raise it with every change you publish (see Versions). |
| `units` | Required | `mm`, `cm`, `m`, `in` or `ft`, in lower case: the units you model in. |
| `description` | Optional | One line, shown in the Parametric Blocks panel (`PBManager`). |
| `layers` | Optional | `as named` puts each output on the layer its NickName names (see Layers). |
| `origin`, `author`, etc | Optional | Notes for whoever edits the .gh. Not shown to users. |

Import can write the required keys for you. When any of them is missing (or the whole group), an interactive `PBImport` offers to add them to the .gh itself: the file name as `name`, a new `id`, `version: 1` and the document's units. Check the units it chose.

## Parameters: `PB:IN`

Every object in `PB:IN` is one parameter. Its current value (or internalised data) is the default. Currently accepted parameter types (anything else is rejected):

| Grasshopper object | Parameter | Users get |
|---|---|---|
| Number Slider | Number (float slider) or Integer (integer, even or odd slider), with the slider's range and decimals | A stepper; values outside the range are clamped. Numbers show the definition's `units` beside them; integers show none, as they are usually counts |
| Value List | Choice | A dropdown of the item labels |
| Boolean Toggle | Boolean | A checkbox |
| Panel | Text | A text box |
| Number, Integer, Boolean or Text param | Number, Integer, Boolean or Text, without a range | A stepper, a checkbox or a text box |
| Geometry param: Point, Curve, Surface, Brep, Mesh, SubD or Geometry | Geometry | **Pick…** when inserting; edit mode (double-click) afterwards |

Every parameter has:
- **A NickName** (required), unique among the parameters. It is the parameter's label in Rhino and the key its value is stored under on every block, so don't rename it once blocks exist. New sliders have none, and the defaults of other objects (`Toggle`, `List`, `Crv`) pass but read badly. **Name everything.** Rhino shows no unit beside a value, so put one in the NickName where it helps: `Width (cm)`, `Angle (°)`. `PBSet` also accepts the name without the bracket, `Width`.
- **A Description** (recommended): the parameter's tooltip in Insert, Properties and edit mode. Grasshopper's interface can't set it natively: see Descriptions.
- **A value.** Internalise one in Number, Integer, Boolean and Text params; empty, they default to 0, false or empty text.
- **Nothing wired into it.** Parameters are sources.

### Value Lists

Each item has a label (what users see) and an expression (what blocks store). Therefore expressions must be kept permanent when versioning; labels can change at any time.

### Geometry parameters

- **Input only.** The block shows only what comes out of `PB:OUT`. To show the geometry itself (a wall's line), wire it to an output.
- **Encapsulated, not referenced.** The geometry is copied into the block, in block coordinates and definition units. It moves, copies and mirrors with the block, and nothing breaks when the objects it came from change or are deleted. Geometry picked during Insert stays where it is in the model, stored relative to the insertion point clicked next.
- **Edit mode.** Double-clicking a block shows its geometry parameters as ordinary Rhino objects. Users edit them with any Rhino tool, the block previews live, and **Done** applies the change.
- **The number of defaults decides one or many.** One internalised object (**Set one Curve**): users edit exactly one. Two or more (**Set Multiple Curves**): they can draw any number. A param whose Access is list or tree, set by code, also takes any number.
- **Always internalise a default.** Insert uses it until the user picks something, and import test-solves with it.
- **Prefer Curve** to Line, Arc, Circle, Rectangle or Polyline: users can draw any curve, and the narrower params fail on the wrong shape.
- **Name them for the user.** A definition can have several (a `Wall` curve and a `Floor` outline). Users edit one at a time, choosing it by NickName in edit mode.

### Descriptions

Grasshopper's interface can't set an object's Description. Set it with Python 3 in Rhino's Script Editor, with the definition open on the canvas, then save the .gh (this sets the first object with that NickName):

```python
import Grasshopper
doc = Grasshopper.Instances.ActiveCanvas.Document
next(o for o in doc.Objects if o.NickName == "Width").Description = "Overall width, in cm"
```

The MetaHopper plug-in (Food4Rhino) does the same from the canvas.

## Outputs: `PB:OUT`

`PB:OUT` holds params only, wired from your logic.

- **Geometry.** Each geometry param is one output, with a NickName unique among the outputs. Everything it carries, in every branch, becomes block geometry. An output may be empty for some values, but with the default values the outputs together must produce at least one object. Annotations (text entities, dimensions, leaders) and hatches can't be block geometry yet: they're skipped with a warning.
- **Messages.** Any other param (a Text param, a Panel) carries messages. Each non-empty item is shown under Information in the Properties panel after a solve. Start it with `WARN:` to show it as a warning, for example a failed code check.

### Layers

- By default every output goes on the user's home layer (Options › Parametric Blocks; Rhino's Default layer unless changed), and output NickNames are plain names (`Body`, `Plan outline`).
- With `layers: as named` in `PB:META`, each output's NickName is a full layer path (`Furniture::Shelving`, `Plan::Furniture outline`), created if missing.
- Either way the geometry is By Parent: the block's own layer sets colour, linetype and print settings, like an ordinary block. Named layers add visibility control: turning `Plan::Furniture outline` off hides that part of every block.

## The logic in between

Build it however you like (components, C# or Python scripts) within four rules:

1. **Everything comes in through `PB:IN`.** Don't read the Rhino document or files (Geometry Pipeline, Model Object, Read File…), bake, or write files.
2. **Same inputs, same outputs.** Fix random seeds: identical blocks share one cached geometry.
3. **Nothing local.** Scripts are embedded with the definition, but files they reference on your disk (`#r` paths, local packages) are not. Other Grasshopper plug-ins are allowed; every computer that changes those blocks needs them, and without them the blocks keep their last geometry.
4. **Scratch is harmless.** Only errors on objects that feed `PB:OUT` fail a solve. Errors elsewhere are reported as warnings.

## Checking a definition

In this repository (it needs the .NET SDK and Rhino 8, and builds the test project first):

```
./tools/validate-definition.ps1 "path\to\Bookshelf.gh"
```

It checks the definition headlessly, lists the parameters and outputs it found, and test-solves with the defaults. Fix every error; warnings are advice. `PBImport` in Rhino reports the same errors but not the warnings. Then insert a block with values other than the defaults.

| Message | Fix |
|---|---|
| `missing group PB:META`, `META: missing 'id'` (or `name`, `version`, `units`) | Let an interactive `PBImport` add them, or write them in the panel. |
| `PB:IN: a Number Slider has no NickName` | Name it. |
| `PB:IN: input 'X' has wires into it` | Parameters are primary sources: move the logic after them. |
| `PB:IN: 'X' is a Plane parameter…` (or Vector, Box) | Use a Point or Curve param and compute the plane, direction or box inside. |
| `PB:IN: unsupported object 'X'` | Only the objects in the Parameters table can be parameters. |
| `PB:OUT: 'X' is not a parameter` | Wire the component into a param and put the param in the group. |
| `component 'X' (…) is not installed here` | Install the Grasshopper plug-in it comes from. |
| `test solve: X: …` | X feeds `PB:OUT` and fails with the default values: fix it in Grasshopper. |
| `test solve with default values produced no geometry` | Make the defaults produce something (internalise default geometry). |
| Warning: `X: no description` | See Descriptions. |
| Warning: `item 'L' has an unquoted text expression` | Quote it: `"OAK"`. |
| Warning: `'…' may read the Rhino document or files` | Bring that data in through `PB:IN` instead. |

## Versions

A new version is the same .gh, changed: keep the `id`, raise the `version`. To bring it into a file, use **Edit in Grasshopper** in the Parametric Blocks panel, save, then **Update blocks**; or run `PBUpdate` with the new .gh. Every block moves to the new version, and a block whose new solve fails keeps the previous one.

Blocks carry their values over by NickName:

| Change | Existing blocks |
|---|---|
| Add a parameter | Get its default |
| Remove a parameter | Lose its value (warning) |
| Rename a parameter | Lose its value: the new name is a new parameter, with its default |
| Change a parameter's kind (a slider becomes a Value List) | Get the new default (warning) |
| Relabel a Value List item | Keep their choice |
| Change or remove a Value List expression | Get the default (warning) |
| Change a slider's range or decimals | Have their values clamped or rounded to fit (warning) |
| Change `units` | Have their stored numbers and geometry read in the new units, unconverted: wrong sizes. Don't. |
| Change `id` | Stay on the old definition: this is a different one |
