# Enforce Tools (DayZ)

[Русская версия / Russian version](README.ru.md)

VS Code that understands **DayZ Enforce Script**: your mod, its dependencies and the engine
scripts together with every `modded` layer. Type-aware completion, navigation, a references
panel, errors before the build and a verdict from the real engine on `F5`.

> ⚠️ **Beta.** Expect bugs and changes between releases — bug reports are very welcome.

---

> ## ⚠️ Read this before you start
>
> **Unpacking `.pbo` files, decoding binarized configs and recovering obfuscated code may
> violate the restrictions a mod author placed on their work.**
>
> The plugin provides the technical capability — **using it is entirely your responsibility**.
> By adding someone else's mod to `enforce.project.json` you confirm that you are entitled to
> do so. The plugin authors grant no rights or licences for it and take no part in your
> decision.
>
> **Why it exists:** to develop **your own** code when a third-party mod is a dependency — you
> need its public API and nothing beyond that. That is why obfuscated mods open as
> **signatures only**: class headers, method signatures, field types. Method bodies are never
> extracted.

---

![References panel and class popup](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/references-panel-class.png)

## The problem

Out of the box VS Code sees EnScript as text: completion offers every namesake in the corpus,
"go to definition" lands in someone else's mod, and you learn about a typo five minutes later —
when the server refuses to start, with an error in a file you never touched.

The plugin indexes the whole picture instead: your code, dependencies, engine scripts and every
`modded` layer in load order. Everything else follows from that.

## What you get

### Completion by the actual type

Members of the type that really stands to the left of the dot — call chains, generics, `auto`,
`foreach` variables. Every item shows its owner class and how many mods already touched it.
After an enum name you get its values, with numbers, in declaration order.

![Completion with owners](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/completion-owners.png)

### Navigation that knows about layers

Hover and `F12` resolve by type, not by name. The popup shows a clickable signature, the owner
class, mod and module, every `modded` layer and the overrides in descendants — you see at once
who else changed this method. `super` goes exactly where the engine would go.

![Hover with definitions and overrides](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/hover-definitions.png)

### Type explorer

The **M4** icon on the left opens a tree of everything the plugin knows: your mod on top, with
**Properties** (the manifest), **Dependencies** (the engine and every `load[]` entry) and then
your sources — folder for folder, as on disk. The tree is lazy, so a corpus of 8000 classes
opens instantly.

A folder with a `config.cpp` is drawn as a mod, the `config.cpp` itself expands into
`CfgPatches` / `CfgMods`, and `#define`s become nodes with their values (otherwise
`*_Preload` addons look empty). Read-only items are marked with a lock right away.
**Only Scripts** in the toolbar trims the tree down to code.

Dependencies are managed from the tree: **+** on `Dependencies` adds a mod folder or a `.pbo`
to the manifest, **−** removes the entry (nothing on disk is touched, and your comments in the
manifest survive). Next to the manifest the plugin keeps `enforce.deps.json` — the hash of
every pbo and its unpack folder, so "is this the same mod?" is a question with an answer.

The explorer is a viewer: the context menu copies paths and reveals files. Create, rename and
delete appear only if you turn on `enforce.explorerEdit`.

![Explorer next to the class popup](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/explorer-symbols-popup.png)

![Explorer with the mod expanded](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/explorer-tree.png)

No `enforce.project.json` yet — the panel offers to create it. Engine scripts not found — an
**engine — not set** row appears with a button to pick the folder.

### References in a proper panel

`Shift+F12` opens a Visual Studio style panel: declarations and layers on top, usages below,
split into calls, reads, writes, constructors and callbacks. Filters, sorting, pinning to a
tab. Ten thousand rows do not lag. Plain text matches are hidden by default — one toolbar
button brings them back.

![References panel for a member](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/references-panel-member.png)

![References for enum members](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/enum-members.png)

### Defines: what is on, and what will never build

Half of a DayZ project lives under `#ifdef`. The plugin collects defines from everywhere they
come from — `#define` in scripts, `defines[]` in `config.cpp`, mod names — and shows the
result: dead blocks are dimmed, hover tells you whether a guard is on and where the value came
from, `F12` goes to the declaration. A guard declared nowhere is an error: that code will never
build.

Types work the same way: if every declaration of a class sits under a guard that is off, its
usages are dimmed and flagged. That is the engine's `Unknown type`, shown in the editor instead
of at the end of the build — and it names the guilty guard.

![Undeclared define card](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/define-unknown.png)

![Define whose declaration is commented out](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/define-off.png)

### Errors caught before the build

On save the file is parsed and checked by the linter and semantics: undeclared methods and
variables (including out-parameters), unknown types, wrong arity, constructing classes with a
private constructor, and fifty more rules. Each rule stands on an engine fact proven by a real
build, so severities are honest: what the engine forgives is a warning, what it rejects is an
error. Any rule can be overridden through `enforce.rules` by the code shown in Problems.

Names the engine declares on the C++ side live in the manifest's `environments` section —
search, hover and go-to-declaration work for them:

```jsonc
"environments": { "DBT_OK": 0, "DBB_NONE": 0, "DMT_INFO": 1 }
```

![Diagnostics](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/diagnostics.png)

With `build.preflight` the same check runs over the whole project before the build, so a typo
does not cost you a round trip through the server. It stops only on what the engine will not
accept, and the dialog always offers **Build anyway**. Findings are tagged `[pre-flight]` and
stay in Problems until the next run: editing a file clears its own marks, editing the manifest
does not (changing `defines` flips the verdict everywhere — run it again).

![Pre-flight stopped the build](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/preflight-dialog.png)

### Inlay hints

Four kinds of inline hints — each toggled separately, and each silent when there is nothing to
infer:

```c
auto: eAIBase ai = eAIBase.Cast(from: g_Game.CreateObject(type: "eAI_SurvivorM_Mirek", pos: "1 1 1"));
if (!Class.CastTo(out to: data, from: params))
if (is null: !ai)
```

| Setting | What it shows |
| --- | --- |
| `enforce.inlayHints.autoTypes` | the inferred type of an `auto` declaration |
| `enforce.inlayHints.parameterNames` | parameter names at call sites |
| `enforce.inlayHints.parameterModifiers` | `out` / `inout` on arguments |
| `enforce.inlayHints.truthiness` | what a non-bool condition actually tests |

They are hidden the usual VS Code way: `editor.inlayHints.enabled` (e.g. `offUnlessPressed` to
show them while `Ctrl+Alt` is held).

### One key to build and ask the engine

`F5` packs the mod into a PBO (no AddonBuilder or other tools) and runs the compile on a
headless DayZ Server — a verdict from the real compiler in about ten seconds. Root causes are
separated from the cascade behind them, every error is mapped back to your file in Problems,
and paths in the log are clickable — including errors inside dependencies. `Ctrl+Shift+B` packs
the PBO only.

The engine gets its dependencies from the same `load[]` the analyzer indexes: a ready mod is
passed as is, sources with a `config.cpp` are packed into `<outDir>/deps/@Name`. One source of
truth — "green in the editor, red in the build" has nowhere to come from.

If the engine pops a modal dialog (`Addon 'X' requires addon 'Y'` and friends), the plugin
closes it and writes the text into **Output**: the build never waits for a human click, and the
result does not depend on whether someone is watching the screen.

![Engine check](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/build-check.png)

![Pre-flight errors in Problems](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/preflight-problems.png)

### Small things that add up

- **Signature help** with the active parameter highlighted.
- **Formatter** for a file or a selection (or your own AStyle via `enforce.formatting.astylePath`).
- **CodeLens and semantic colors** — "N references" / "N layers" above declarations.
- **Engine and dependency sources open read-only**, so you cannot break them by accident.

![Signature help](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/signature-help.png)

![CodeLens and highlighting](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/codelens-highlighting.png)

## Getting started

### 1. Unpacked game scripts

The plugin reads vanilla scripts as ordinary sources. The usual way is **DayZ Tools** →
*Extract Game Data*: it mounts the Work Drive as `P:` and puts the scripts into `P:\scripts`
(folders `1_Core` … `5_Mission`). Any other local copy works too — just point `engine` at it.

Dependencies in `load[]` can be sources, an unpacked addon, or a workshop `@Mod` folder with
`.pbo` files: packed mods are unpacked into the plugin cache automatically (scripts and configs
only, binarized configs are decoded back to text) and refreshed when the mod changes on disk.
The cache is a mirror of the PBO, not a working copy — edits inside it are wiped. To change a
dependency's code, unpack the PBO yourself and add that folder as sources.

⚠️ About third-party mods — see the warning at the top of this page.

**Obfuscated mods** open in *API-only* mode: declarations without bodies. No guarantees — the
API may come out partial, or not at all.

### 2. `enforce.project.json` in the project root

Put it next to your mod's `config.cpp`. The minimal working file is `{}`, but this is a better
start:

```jsonc
{
  "engine": "P:/scripts",
  "build": {
    "modName": "@MyMod",
    "outDir": "./builds",
    "serverExe": "D:/Steam/steamapps/common/DayZServer/DayZServer_x64.exe"
  }
}
```

Keys are completed as you type and typos are underlined immediately.

![Manifest key completion](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/manifest-completion.png)

### 3. Running with `F5`

`.vscode/launch.json` in the workspace root:

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    { "type": "enforce", "request": "launch", "name": "Enforce: Build & Check" }
  ]
}
```

VS Code offers it on its own too: *Run and Debug* → *create a launch.json file* → **Enforce
Build & Check**.

**Platform.** The analyzer (index, completion, navigation, diagnostics, pre-flight) runs
anywhere VS Code does — the parser is wasm. The engine build is Windows-only: the gate launches
`DayZServer_x64.exe`. On macOS/Linux that part reports "server not found" and everything else
keeps working.

### No DayZ Server at hand?

Turn on pre-flight — `F5` will check the whole project, show the errors and then honestly run
into the missing server:

```jsonc
{ "engine": "P:/scripts", "build": { "preflight": true } }
```

The check is not as complete as the real compiler's verdict, so some errors surface later. But
you can start without the game.

## Commands

All of them live in the palette under the `Enforce:` prefix.

| Command | Description |
| --- | --- |
| `Find All References (Panel)` | References panel for the symbol under the cursor (`Shift+F12`) |
| `Build Check (Engine Compile)` | PBO + engine compile (`F5`) |
| `Rebuild Index` | Full re-index |
| `Show All Modifications (discovery)` | Every `modded` layer of a class |
| `Declare Unknown Defines in enforce.project.json` | Add unknown defines to the manifest as `false` |
| `Create Project File` | Create `enforce.project.json` |
| `Set Engine Scripts Folder…` | Point the plugin at the engine scripts |
| `Add Dependency…` / `Remove Dependency` | Edit `load[]` from the tree |
| `Search Types` / `Filter Symbol Kinds` / `Only Scripts` | Explorer tools (also toolbar buttons) |
| `Show Current File in Type Explorer` / `Show in Type Explorer` | Reveal a file or a definition in the tree |

File operations (`New Mod…`, `New Script…`, `Rename…`, `Delete` and friends) appear in the
explorer only when `enforce.explorerEdit` is on.

## Settings

| Setting | Default | Description |
| --- | --- | --- |
| `enforce.workDrive` | `P:` | Work Drive with the unpacked engine scripts |
| `enforce.parseDiagnostics` | `warning` | Parse diagnostics: `warning` / `error` / `off` |
| `enforce.semanticDiagnostics` | `warning` | Semantics (members, types, arity): `warning` / `error` / `off` |
| `enforce.unknownDefines` | `warning` | Guards whose define is declared nowhere |
| `enforce.deadTypeDiagnostics` | `warning` | Types declared only under a guard that is off |
| `enforce.dimDeadTypes` | `true` | Dim such types in the editor |
| `enforce.rules` | `{}` | Severity per rule code: `{"ternary-unsupported": "off"}` |
| `enforce.conditionalMembers` | `hide` | Members behind a false guard: `hide` / `dim` |
| `enforce.codeLensReferences` | `true` | "N references" above declarations |
| `enforce.inlayHints.autoTypes` | `true` | Inferred type for `auto` |
| `enforce.inlayHints.parameterNames` | `true` | Parameter names at call sites |
| `enforce.inlayHints.parameterModifiers` | `true` | `out` / `inout` on arguments |
| `enforce.inlayHints.truthiness` | `true` | What a non-bool condition tests |
| `enforce.references.includeTextMatches` | `false` | Text matches in the references panel |
| `enforce.preflight` | `false` | Check the project before launching the engine |
| `enforce.build` | see below | Build defaults, overridden by `build{}` in the manifest |
| `enforce.readonlyExternal` | `true` | Engine and dependencies open read-only |
| `enforce.autoRefreshSeconds` | `300` | Index revalidation interval, seconds (0 = off) |
| `enforce.formatting` | `{}` | `useSpaces`, `tabSize`, `allmanBraces`, `spaceAfterKeyword`, `astylePath` |
| `enforce.explorerEdit` | `false` | Allow file operations from the explorer |

## `enforce.project.json` reference

Every key is optional.

```jsonc
{
  // ===== WHAT GETS INDEXED =================================================
  // Unpacked game scripts (the folder with 1_Core … 5_Mission).
  // Omit it and <workDrive>/scripts is used. ${WORKDRIVE} works here.
  "engine": "P:/scripts",

  // Dependencies in load order: sources, an unpacked addon, or a packed mod
  // (@Mod with addons/*.pbo, or a single .pbo).
  // THE SAME LIST is passed to the engine as -mod= during the build.
  "load": [
    { "name": "CF",        "path": "D:/mods/DayZ-CommunityFramework/JM/CF" },
    { "name": "Expansion", "path": "D:/mods/DayZ-Expansion-Scripts" },
    { "name": "COT",       "path": "D:/Steam/steamapps/common/DayZ/!Workshop/@Community-Online-Tools" }
  ],

  // Your own code: linted and packed into PBOs. Omit it and the manifest
  // folder is used.
  "project": [
    { "name": "MyMod", "path": "./src" }
  ],

  // ===== WHAT COUNTS AS ENABLED ============================================
  // Only what sources cannot show: engine defines and those from closed PBOs.
  // Anything declared in your code or dependencies is picked up automatically.
  // The engine check prints its real list and offers to import it.
  "defines": { "PLATFORM_WINDOWS": true, "SERVER": true, "DIAG": false },

  // Names the engine declares on the C++ side, with their values.
  "environments": { "DBT_OK": 0, "DMT_INFO": 1 },

  // ===== REFERENCES ========================================================
  // Allow references and CodeLens on engine and dependency files
  // (popular engine types have thousands of usages).
  "externalReferences": true,
  "references": { "includeTextMatches": false },

  // ===== BUILD AND ENGINE CHECK (F5) =======================================
  "build": {
    "preflight": true,              // our own check BEFORE the engine starts
    "modName": "@MyMod",            // output folder name
    "outDir": "./builds",           // <outDir>/<modName>/addons/*.pbo

    // There is no separate dependency list — the engine gets load[].

    // Where the server lives. Drop both keys to auto-detect via Steam.
    // DayZServer_x64.exe is required; DayZDiag needs a running Steam client.
    "serverExe": "D:/Steam/steamapps/common/DayZServer/DayZServer_x64.exe",
    "serverCwd": "D:/Steam/steamapps/common/DayZServer",

    "mission": "mpmissions/dayzOffline.chernarusplus",
    "timeoutSec": 240
  }
}
```

Both `/` and `\\` work in paths. The PBO prefix is not configurable — it comes from each
module's `config.cpp`, and a project with several modules is packed into one PBO per module.
Only scripts go into the PBO: that is all the engine needs for a verdict, asset packing comes
later.

## Thanks

Two great projects are mentioned in the examples above:

- [DayZ-Expansion-Scripts](https://github.com/salutesh/DayZ-Expansion-Scripts)
- [DayZ-CommunityFramework](https://github.com/Arkensor/DayZ-CommunityFramework)

They are not here by accident: their code is where we studied the syntax and idioms of Enforce
Script — a large living project shows how the language is really used far better than any
documentation. Thanks to their authors and the community.

## License

MIT
