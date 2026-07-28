# Enforce Tools (DayZ)

[Русская версия / Russian version](README.ru.md)

VS Code that actually understands **DayZ Enforce Script** — your mod, its dependencies and
the engine scripts, `modded` layers included. Completion, navigation, references, errors
before the build, and a real engine compile check on `F5`.

> ⚠️ **Beta.** Things may break or change between releases — bug reports are very welcome.

---

> ## ⚠️ Read this before you start using the plugin
>
> **Unpacking a `.pbo`, decoding a binarized config or recovering obfuscated code — even
> partially — may violate the terms the mod author set for their work.**
>
> **The plugin provides the technical means; the responsibility for unpacking is entirely
> yours.** By adding a mod to `enforce.project.json` you
> confirm that you are entitled to use it this way, and you accept full liability for any
> infringement of the rights holder. The plugin's authors cannot grant you rights,
> permissions, licences or any other authority to work with someone else's mod, and take no
> part in your decision to use it in your project.
>
> **What this feature is for.** It exists for one purpose: developing **your own** software
> against someone else's mod as a dependency — you need its public API to compile and
> navigate, nothing more.
>
> **How the author's rights are protected.** Obfuscated mods are exposed **as signatures
> (the public API) only, with no implementation**: class headers, method signatures, field
> types. Method bodies are not extracted.

---

![References panel and class popup](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/references-panel-class.png)

## The problem

Out of the box VS Code sees EnScript as plain text. Completion offers every same-named
symbol in the corpus, "go to definition" lands in the wrong mod, and a typo shows up only
when the server refuses to start — minutes later, pointing at a file you never touched.

The plugin indexes the whole picture instead: your code, dependencies, engine scripts and
every `modded` layer in load order. Everything below follows from that.

## What you get

### Completion that knows the real type

Not "every method with this name" — the members of the type actually standing to the left of
the dot. Call chains, generics, `auto`, `foreach` variables. Each item shows the class it
comes from and how many mods have already touched it.

![Completion with owners](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/completion-owners.png)

Enum values are offered after the enum name, with their numbers, in declaration order.

### Navigation that follows the layers

Hover and `F12` resolve by type, not by name. The popup shows a clickable signature, the
owning class, the mod and module it lives in, all `modded` layers and the overrides in
derived classes — so you can see at a glance who else has changed this method.
`super` leads exactly where the engine would go.

![Hover with definitions and overrides](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/hover-definitions.png)

### Type explorer

The **M4** icon in the activity bar opens a tree of everything the plugin knows about, laid
out like a project browser: your mod on top (named after `build.modName`, or the folder you
opened), then **Properties** with the manifest, **Dependencies** with everything that gets
loaded — the engine and every `load[]` entry — and then your own sources. Everything starts
collapsed and expands lazily, so an 8000-class corpus opens instantly. Click a file to open
it, click a symbol to land on its definition.

**Below an entry the tree is your disk, folder for folder.** Your project is already
organised — sections, mods, script modules — and that is what you see. A folder holding a
`config.cpp` is drawn as a mod (your own in blue, a dependency in its own color), everything
else is a plain folder. Folders come first, then mods, then files, so a mod never gets lost
between sections.

`config.cpp` files are in the tree too, with their `CfgPatches` / `CfgMods` classes nested as
deep as they go — the config is part of the mod, not a separate world. A mod that ships only
assets and a config is simply a mod folder without scripts; nothing is hidden away.

Preprocessor defines are nodes too. A file that declares nothing but `#define`s — the
`*_Preload` addons of a big mod are exactly that — used to look empty, and the addon looked
like a lone `config.cpp`. Now the defines are right there, with their values.

**Only Scripts** in the toolbar cuts the tree down to code: folders with no `.c`/`.cpp`
anywhere inside disappear, and the single wrapper folder that Workbench conventions put
inside a script module (`5_Mission → DayZExpansion_Core → …`) is skipped — you get the files
right away. The five script modules (`1_Core`…`5_Mission`) stay visible even when empty:
that is where code goes.

Icons are colored the way a project browser colors them: amber folders, one color for your
own code, another for dependencies, and every symbol kind — class, method, field, constant,
enum, define — in the color your theme already uses for it in Outline and breadcrumbs.

![The explorer next to a class popup](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/explorer-symbols-popup.png)

No project file yet? The panel says so and offers to create one, the way VS Code offers a
`launch.json`. The generated `enforce.project.json` already knows your mod name (from the
folder) and where builds go; if the engine scripts can be found, they are filled in too.

Engine scripts are usually still packed inside the game (`dta/scripts.pbo`), so when they
cannot be detected the tree shows an **engine — not set** row with a button: pick any folder
that has the standard `1_Core / 2_GameLib / 3_Game / 4_World / 5_Mission` layout. What's inside
is your call — the structure is all that is checked.

Next to your manifest the plugin keeps `enforce.deps.json` — what exactly is connected. A mod
does not have to come from the Workshop, and a second copy of the same mod easily sits in a
folder next to it, newer or older with nothing to tell them apart. A path cannot answer "is
this the file I picked?", so the lock records the **hash of every PBO** plus the unpack folder
the index actually reads (and the author's `CfgMods` version, when there is one). It updates
itself whenever the project is reindexed — after you edit the config by hand or use the
buttons below — and you can see it under **Properties**.

**The explorer is a browser, not a file manager.** Right-click gives you what a browser
needs: copy path, copy relative path, reveal in the file explorer. Creating, renaming and
deleting files is what the built-in Explorer is for, and it does it better. While the index
is being built the panel is disabled: there is nothing to click yet.

Dependencies are managed from the tree: **+** on the `Dependencies` branch picks a mod folder
or a `.pbo` and writes it into your manifest (the name comes from the folder or file), **−** on
an entry removes it after asking. Only the manifest entry goes — nothing on disk is touched,
and the comments you keep in `enforce.project.json` survive the edit.

Script files carry their own green **C** icon, and everything read-only — the engine and every
dependency — is marked with a lock right away, not only once you open the file. The panel
opens with your own mod already expanded; dependencies stay collapsed until you ask.

The tree never jumps on its own: it changes only when you click it, or when you use the **☰**
link in a hover popup.

The toolbar, left to right: substring search across types and members, filters for what to
show (methods / properties / constants / globals / defines), a refresh that revalidates the
index first, a jump to the file you are editing (any source connected to the project, yours
or a dependency), collapse-all, expand-project and **Only Scripts**. Expand covers your own
mod down to its files — not into them, and never into dependencies: that would mean walking
thousands of classes for one click. Collapse leaves the project root open, so the panel never
shrinks to a single line.
Every definition in a hover popup carries a **☰** link that opens the explorer with that exact
definition selected — handy when a type has several `modded` layers and you want to see where
each one lives.

![Type explorer with a mod expanded](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/explorer-tree.png)

### Find all references, in a proper panel

`Shift+F12` opens a bottom panel in Visual Studio style: declarations and layers on top,
usages below, split into calls, reads, writes, constructors and callbacks. Filter by column,
sort, collapse groups, pin a result to its own tab. Even ten thousand rows stay responsive.

![References panel for a member](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/references-panel-member.png)

Plain text matches are hidden by default — one button in the toolbar brings them back when
you actually want them. Enum values get their own reference counts, string literals from
MVC bindings included:

![Enum member references](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/enum-members.png)

### Defines: see what is on, off, and what will never compile

Half of a DayZ project lives under `#ifdef`. The plugin collects defines from everywhere
they can come from — `#define` in scripts, `defines[]` in `config.cpp`, mod names, and
switches the author left commented out — and shows you the result:

- dead `#ifdef` blocks are dimmed, so you never read code that is not there;
- hover on a guard tells you whether it is on or off, and where that came from;
- `F12` jumps to the declaration, `Shift+F12` lists every place the guard is used;
- a guard nobody declares anywhere is reported — such code can never compile.

![Hover card for an undeclared define, with the references panel below](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/define-unknown.png)

![Hover card for a define whose declaration is commented out](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/define-off.png)

The same applies to types: if every declaration of a class sits under a switch that is off,
its usages are dimmed and marked. This is the engine's `Unknown type` — shown in the editor
instead of at the end of a build, with the guilty switch named.

<!-- TODO screenshot: dead type dimmed in live code + hover -->

### Mistakes caught before the build

On save the file is parsed, linted and checked semantically: undeclared methods and
functions, unknown types, wrong argument counts, and variables that were never declared —
including the ones you pass as out-parameters, like `Class.CastTo(ai, …)` where `ai` is a
typo. Severity is tuned to the engine — what it forgives is a warning, what it rejects is an
error. A line break that splits an expression is an error too — the engine compiles line by
line and only lets a line end with `(`, `,` or `=`.

Undeclared names are reported in your own code only: the engine declares part of its constants
on the C++ side, so judging read-only sources by our data would just cry wolf. For those names
there is `environments` in the manifest — write them with their values, and they join the
index like anything else:

```jsonc
"environments": { "DBT_OK": 0, "DBB_NONE": 0, "DMT_INFO": 1 }
```

Search, hover and go-to-definition work on them; the definition lands on that very line of
your manifest.

![Diagnostics](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/diagnostics.png)

Optionally the same check runs over the whole project right before a build, so a five-minute
round trip through the server is not spent on a typo. It only ever stops on errors the
engine would reject, and the dialog always offers **Build anyway**.

Pre-flight is a **separate pass over the whole project**, not the live analysis: its findings
are marked `[pre-flight]` and stay in Problems until you run it again. Editing a file clears
that file's pre-flight marks; editing `enforce.project.json` does not — a change to `defines`
or `environments` can flip the verdict for every file, so the list is only trustworthy right
after a run. If you fixed something via the manifest, re-run the build (F5) to refresh it.
The live analysis in the editor updates on save, independently.

![Pre-flight stopped the build and offered to jump to the first error](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/preflight-dialog.png)

### One key to build and ask the engine

`F5` packs your mod into a PBO (no AddonBuilder, no extra tools) and runs a headless
DayZ Server compile — the real compiler's verdict in about ten seconds. Root causes are
separated from the cascade that follows them, and every error is mapped back to your source
file in the Problems panel. `Ctrl+Shift+B` packs the PBO only.

The packer puts **scripts only** into the PBO — enough for the engine to compile your code
and give its verdict. Full mod packing, assets included, will come later.

![Engine build check](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/build-check.png)

The build log is colorized and every path in it is clickable — including the `config.cpp`
of a module that was skipped, so you can see why. Pressing `F5` again after a failure just
works: the plugin waits for the previous server to shut down.

![Pre-flight errors in the Problems view](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/preflight-problems.png)

### Small things that add up

- **Inlay hints for `auto`** — the inferred type right in the line, and nothing at all if it
  cannot be inferred.
- **Signature help** with the active parameter highlighted as you type the arguments.
- **Formatter** for the whole file or a selection (or point it at your AStyle binary).
- **CodeLens and semantic colors** — "N references" / "N layers" above declarations.
- **Engine and dependency sources open read-only**, so you cannot break them by accident.

![Signature help](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/signature-help.png)

![CodeLens and highlighting](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/codelens-highlighting.png)

## Getting started

### 1. Unpacked game scripts

The plugin reads the vanilla scripts as ordinary sources, so you need them unpacked on disk.
The usual way is **DayZ Tools** → *Extract Game Data*: it mounts the Work Drive as `P:` and
puts the scripts into `P:\scripts` (the `1_Core` … `5_Mission` folders). Any other local copy
works too — just tell the plugin in the config where it is.

Dependency mods are easier: point `load[]` at the mod's **sources**, an unpacked addon —
or just the workshop **`@Mod` folder with packed `.pbo` files**. Packed mods are unpacked
automatically (scripts and configs only, binarized configs are decoded back to text) into
the plugin cache and re-unpacked when the mod updates.

⚠️ **Before you point `load[]` at a mod that is not yours, read the warning at the top of
this page — the responsibility for unpacking someone else's work is entirely yours.**

**Obfuscated mods** get an *API-only* view: the plugin extracts declarations —
class headers, method signatures, field types, enums — and **not the bodies**. You get
completion, navigation and type checking against such a mod, while its implementation stays
where the author put it. **This is not guaranteed to work:** for one reason or another the
API may come out partial — or not at all.

### 2. `enforce.project.json` in the project root

The file goes next to your mod's `config.cpp`. The minimal one literally works:

```jsonc
{}
```

But it is better to start from this — vanilla scripts, mod name, where to build, and where
DayZ Server lives for debugging:

```jsonc
{
  "engine": "P:/scripts",
  "build": {
    "modName": "@MyMod",
    "outDir": "./builds",
    "serverExe": "D:/Steam/steamapps/common/DayZServer/DayZServer_x64.exe",
    "serverCwd": "D:/Steam/steamapps/common/DayZServer"
  }
}
```

Add dependencies when you need them — the `load[]` key, see the reference below. Manifest
keys are suggested as you type, and a typo is underlined right away instead of silently
doing nothing.

![Key completion inside enforce.project.json](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/manifest-completion.png)

### 3. Running it with `F5`

Create `.vscode/launch.json` in the workspace root:

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    { "type": "enforce", "request": "launch", "name": "Enforce: Build & Check" }
  ]
}
```

VS Code offers it on its own too: *Run and Debug* → *create a launch.json file* →
**Enforce Build & Check**. Nothing else is required — `F5` packs the PBO and runs the engine
check.

### No DayZ Server at hand?

Then check your code with our analyzer — turn `preflight` on:

```jsonc
{
  "engine": "P:/scripts",
  "build": { "preflight": true }
}
```

`F5` will first run the check over the whole project and show the errors, and only then run
into the missing server and say so honestly. Neither packed dependencies (`build.deps`) nor
the server path are needed for this — but dependency **sources** in `load[]` are still worth
listing, otherwise resolution stays incomplete. It is the slower road: the check is not as
complete as the real compiler's verdict, and some errors will surface later on the server.
But you can start without the game.

## Commands

| Command | Description |
| --- | --- |
| `Enforce: Find All References (Panel)` | References panel for the symbol under the cursor (`Shift+F12`) |
| `Enforce: Build Check (Engine Compile)` | PBO + headless engine compile (`F5`) |
| `Enforce: Rebuild Index` | Force a full reindex |
| `Enforce: Show All Modifications (discovery)` | All modded layers of a class |
| `Enforce: Declare Unknown Defines in enforce.project.json` | Write every unknown define into the manifest as `false` |

## Settings

| Setting | Default | Description |
| --- | --- | --- |
| `enforce.workDrive` | `P:` | Work Drive with the unpacked engine scripts |
| `enforce.parseDiagnostics` | `warning` | Parse diagnostics: warning / error / off |
| `enforce.semanticDiagnostics` | `warning` | Semantic diagnostics: warning / error / off |
| `enforce.unknownDefines` | `warning` | Guards whose define is declared nowhere |
| `enforce.deadTypeDiagnostics` | `warning` | Types declared only under a switch that is off |
| `enforce.dimDeadTypes` | `true` | Dim such types in the editor |
| `enforce.conditionalMembers` | `hide` | Members behind a false guard: hide / dim |
| `enforce.codeLensReferences` | `true` | "N references" above declarations |
| `enforce.inlayHints.autoTypes` | `true` | Inferred type shown for `auto` |
| `enforce.references.includeTextMatches` | `false` | Show text matches in the panel by default |
| `enforce.preflight` | `false` | Check the project before starting a build |
| `enforce.readonlyExternal` | `true` | Open engine and dependency sources read-only |
| `enforce.autoRefreshSeconds` | `300` | Pick up changes outside the workspace (0 = off) |
| `enforce.formatting` | `{}` | Formatter options (indent, braces, AStyle path) |
| `enforce.build` | — | Build defaults, overridden by `build{}` in the manifest |

## `enforce.project.json` reference

Everything is optional. A fully filled example:

```jsonc
{
  // ===== WHAT GETS INDEXED ==================================================
  // This is how the plugin learns which code it reads: completion, navigation,
  // references and diagnostics all follow from it.

  // Unpacked base game scripts (the folder with 1_Core … 5_Mission).
  // Omit to use <workDrive>/scripts.
  "engine": "E:/DayZ/WorkDrive/scripts",

  // Dependency mods, in load order: sources, an unpacked addon, or a PACKED mod —
  // an @Mod folder with addons/*.pbo (or a single .pbo). Packed mods are unpacked
  // into the plugin cache automatically; obfuscated ones give an API-only view
  // (declarations without bodies).
  // A folder holding several mods (like DayZ-Expansion-Scripts) is split into
  // layers automatically.
  "load": [
    // github.com/Arkensor/DayZ-CommunityFramework
    { "name": "CF",        "path": "D:/mods/DayZ-CommunityFramework/JM/CF" },
    // github.com/salutesh/DayZ-Expansion-Scripts
    { "name": "Expansion", "path": "D:/mods/DayZ-Expansion-Scripts" },
    // packed workshop mod — unpacked automatically
    { "name": "COT",       "path": "D:/Steam/steamapps/common/DayZ/!Workshop/@Community-Online-Tools" }
  ],

  // Your own code. Omit it if the sources sit next to the manifest.
  "project": [
    { "name": "MyMod", "path": "./src" }
  ],

  // ===== WHAT COUNTS AS ENABLED =============================================
  // Drives #ifdef: what is dimmed, which members show up in completion, which
  // types are marked as never-compiling.

  // Defines you cannot see from sources: the ones the engine adds itself, or those
  // coming from closed PBOs. Everything declared in your code or dependencies is
  // picked up automatically — a value here simply wins over it.
  // The build check prints the engine's real list and offers to import it.
  "defines": {
    "PLATFORM_WINDOWS": true,
    "SERVER": true,
    "DZ_Expansion_Core": true,
    "DIAG": false
  },

  // ===== FINDING REFERENCES =================================================
  // How wide the usage search goes and what the panel shows.

  // Let references, CodeLens and Shift+F12 work on engine and dependency files too.
  // Popular engine types can have thousands of usages.
  "externalReferences": true,

  "references": {
    // Show plain text matches in the panel by default (the panel has a toggle anyway).
    "includeTextMatches": false
  },

  // ===== BUILDING AND ASKING THE ENGINE (F5) ================================
  // Everything needed to pack the mod into a PBO and feed it to a headless
  // server. If you do not need the engine check, preflight alone is enough.

  "build": {
    // Check the project with our analyzer before launching the engine.
    "preflight": true,

    "modName": "@MyMod",            // output folder name
    "outDir": "./builds",           // artifacts: <outDir>/<modName>/addons/*.pbo

    // Packed dependency mods for -mod=, in load order.
    "deps": [
      "D:/Steam/steamapps/common/DayZ/!Workshop/@CF",
      "D:/Steam/steamapps/common/DayZ/!Workshop/@DayZ-Expansion-Core"
    ],

    // Where DayZ Server lives. Omit both to auto-detect via Steam.
    // DayZServer_x64.exe is required; DayZDiag needs a running Steam client.
    "serverExe": "E:/Games/SteamLibrary/steamapps/common/DayZServer/DayZServer_x64.exe",
    "serverCwd": "E:/Games/SteamLibrary/steamapps/common/DayZServer",

    "mission": "mpmissions/dayzOffline.chernarusplus",
    "timeoutSec": 240
  }
}
```

Both `/` and `\\` work in paths. The PBO prefix is not configured — it is taken from each
module's `config.cpp`, and a project with several modules is packed into one PBO per module.

## Thanks

Two magnificent projects appear in the examples above:

- [DayZ-Expansion-Scripts](https://github.com/salutesh/DayZ-Expansion-Scripts)
- [DayZ-CommunityFramework](https://github.com/Arkensor/DayZ-CommunityFramework)

They are not there by accident: their code is what we studied while working out the syntax
and lexis of Enforce Script — a large living project shows how the language is really used,
far better than any documentation could. Huge thanks to their authors and communities.

## License

MIT
