# Enforce Tools (DayZ)

[Русская версия / Russian version](README.ru.md)

VS Code that actually understands **DayZ Enforce Script** — your mod, its dependencies and
the engine scripts, `modded` layers included. Completion, navigation, references, errors
before the build, and a real engine compile check on `F5`.

> ⚠️ **Beta.** Things may break or change between releases — bug reports are very welcome.

![References panel and class popup](images/references-panel-class.png)

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

![Completion with owners](images/completion-owners.png)

Enum values are offered after the enum name, with their numbers, in declaration order.

### Navigation that follows the layers

Hover and `F12` resolve by type, not by name. The popup shows a clickable signature, the
owning class, the mod and module it lives in, all `modded` layers and the overrides in
derived classes — so you can see at a glance who else has changed this method.
`super` leads exactly where the engine would go.

![Hover with definitions and overrides](images/hover-definitions.png)

### Find all references, in a proper panel

`Shift+F12` opens a bottom panel in Visual Studio style: declarations and layers on top,
usages below, split into calls, reads, writes, constructors and callbacks. Filter by column,
sort, collapse groups, pin a result to its own tab. Even ten thousand rows stay responsive.

![References panel for a member](images/references-panel-member.png)

Plain text matches are hidden by default — one button in the toolbar brings them back when
you actually want them. Enum values get their own reference counts, string literals from
MVC bindings included:

![Enum member references](images/enum-members.png)

### Defines: see what is on, off, and what will never compile

Half of a DayZ project lives under `#ifdef`. The plugin collects defines from everywhere
they can come from — `#define` in scripts, `defines[]` in `config.cpp`, mod names, and
switches the author left commented out — and shows you the result:

- dead `#ifdef` blocks are dimmed, so you never read code that is not there;
- hover on a guard tells you whether it is on or off, and where that came from;
- `F12` jumps to the declaration, `Shift+F12` lists every place the guard is used;
- a guard nobody declares anywhere is reported — such code can never compile.

<!-- TODO screenshot: define hover card; references panel for a define -->

The same applies to types: if every declaration of a class sits under a switch that is off,
its usages are dimmed and marked. This is the engine's `Unknown type` — shown in the editor
instead of at the end of a build, with the guilty switch named.

<!-- TODO screenshot: dead type dimmed in live code + hover -->

### Mistakes caught before the build

On save the file is parsed, linted and checked semantically: undeclared methods and
functions, unknown types, wrong argument counts. Severity is tuned to the engine — what it
forgives is a warning, what it rejects is an error.

![Diagnostics](images/diagnostics.png)

Optionally the same check runs over the whole project right before a build, so a five-minute
round trip through the server is not spent on a typo. It only ever stops on errors the
engine would reject, and the dialog always offers **Build anyway**.

<!-- TODO screenshot: pre-flight dialog -->

### One key to build and ask the engine

`F5` packs your mod into a PBO (no AddonBuilder, no extra tools) and runs a headless
DayZ Server compile — the real compiler's verdict in about ten seconds. Root causes are
separated from the cascade that follows them, and every error is mapped back to your source
file in the Problems panel. `Ctrl+Shift+B` packs the PBO only.

The packer puts **scripts only** into the PBO — enough for the engine to compile your code
and give its verdict. Full mod packing, assets included, will come later.

![Engine build check](images/build-check.png)

The build log is colorized and every path in it is clickable — including the `config.cpp`
of a module that was skipped, so you can see why. Pressing `F5` again after a failure just
works: the plugin waits for the previous server to shut down.

<!-- TODO screenshot: build output after a failed build -->

### Small things that add up

- **Inlay hints for `auto`** — the inferred type right in the line, and nothing at all if it
  cannot be inferred.
- **Signature help** with the active parameter highlighted as you type the arguments.
- **Formatter** for the whole file or a selection (or point it at your AStyle binary).
- **CodeLens and semantic colors** — "N references" / "N layers" above declarations.
- **Engine and dependency sources open read-only**, so you cannot break them by accident.

![Signature help](images/signature-help.png)

![CodeLens and highlighting](images/codelens-highlighting.png)

## Getting started

### 1. Unpacked game scripts

The plugin reads the vanilla scripts as ordinary sources, so you need them unpacked on disk.
The usual way is **DayZ Tools** → *Extract Game Data*: it mounts the Work Drive as `P:` and
puts the scripts into `P:\scripts` (the `1_Core` … `5_Mission` folders). Any other local copy
works too — just tell the plugin in the config where it is.

Same for the mods you depend on: their **sources** or unpacked addons. Hooking up a `.pbo`
directly is not possible yet — it is planned. Without unpacked dependencies the plugin still
works, but it will not see their classes: no completion, no navigation into them.

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

<!-- TODO screenshot: key completion inside enforce.project.json -->

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

  // Dependency mods — SOURCES or unpacked addons, in load order (not .pbo yet).
  // A folder holding several mods (like DayZ-Expansion-Scripts) is split into
  // layers automatically.
  "load": [
    // github.com/Arkensor/DayZ-CommunityFramework
    { "name": "CF",        "path": "D:/mods/DayZ-CommunityFramework/JM/CF", "role": "dependency" },
    // github.com/salutesh/DayZ-Expansion-Scripts
    { "name": "Expansion", "path": "D:/mods/DayZ-Expansion-Scripts", "role": "dependency" }
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
