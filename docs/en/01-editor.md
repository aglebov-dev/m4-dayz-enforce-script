# Code and editor

[← Documentation index](../../README.md) · [Русская версия](../ru/01-editor.md)

Everything here works off one index: your code, the dependencies from `load[]`, the engine
scripts and every `modded` layer in load order.

## Completion

- Members of the type that actually stands to the left of the dot — call chains, generics,
  `auto`, `foreach` variables.
- Each item shows its owner class and how many mods already override it.
- After an enum name — its values, with numbers, in declaration order.
- Members behind a guard that is off are hidden (`enforce.conditionalMembers: dim` shows them
  struck through instead).

## Navigation and hover

- `F12` and hover resolve by type, not by name.
- The popup shows the signature, owner class, mod and module, every `modded` layer and the
  overrides in descendants.
- `super` resolves the way the engine does: in a `modded` layer it goes to the previous layer
  in load order, in a base class to the parent.
- `this` and `super` get the same popup as a type name.
- **Signature help** while typing a call: the full signature with the active parameter
  highlighted, narrowed to a single overload when the arguments allow it.
- Engine and dependency files open read-only (`enforce.readonlyExternal`).

## Inlay hints

| Setting | What it shows |
| --- | --- |
| `enforce.inlayHints.autoTypes` | inferred type of an `auto` declaration |
| `enforce.inlayHints.parameterNames` | parameter names at call sites |
| `enforce.inlayHints.parameterModifiers` | `out` / `inout` on arguments |
| `enforce.inlayHints.truthiness` | what a non-bool condition actually tests |

```c
auto: eAIBase ai = eAIBase.Cast(from: g_Game.CreateObject(type: "eAI_SurvivorM_Mirek", pos: "1 1 1"));
if (!Class.CastTo(out to: data, from: params))
if (is null: !ai)
```

Nothing is shown where the type or the overload cannot be inferred. Hints are hidden the usual
VS Code way — `editor.inlayHints.enabled` (e.g. `offUnlessPressed` to show them on `Ctrl+Alt`).

## Diagnostics

Files are parsed and checked on save: parse errors, the linter and semantics — undeclared
methods and variables (including out-parameters), unknown types, wrong call arity, constructing
a class with a private constructor, and about fifty more rules.

- Severity follows the engine: what the engine forgives is a warning, what it rejects is an
  error.
- Every rule has a code, shown in Problems. Override it in `enforce.rules`:

  ```jsonc
  "enforce.rules": { "ternary-unsupported": "off", "undeclared-variable": "error" }
  ```

- Whole categories are switched with `enforce.parseDiagnostics` and
  `enforce.semanticDiagnostics` (`warning` / `error` / `off`).
- Names the engine declares on the C++ side are not in the scripts — list them in the
  manifest's `environments` so they stop being reported and gain search, hover and
  go-to-definition:

  ```jsonc
  "environments": { "DBT_OK": 0, "DBB_NONE": 0, "DMT_INFO": 1 }
  ```

To check the whole project before a build instead of file by file, see
[pre-flight](04-build.md#pre-flight).

## Preprocessor defines

Defines are collected from every source the engine has:

| Source | Example |
| --- | --- |
| `#define` in scripts | `#define EXPANSION_MODSTORAGE` |
| `defines[]` in `config.cpp` (`CfgMods`) | `defines[] = {"EXPANSIONMOD"};` |
| `CfgMods` class name | the engine declares a define per loaded mod |
| `defines` in the manifest | what no source declares (engine C++ side, closed PBOs) |
| `//#define` | declared and switched off by the author |

Result in the editor:

- blocks under a guard that is off are dimmed;
- hover on a guard says whether it is on and where the value came from;
- `F12` on a define goes to its declaration, `Shift+F12` lists every `#ifdef`/`#ifndef`;
- a guard declared nowhere is reported (`enforce.unknownDefines`) — that code never compiles.
  Command **Enforce: Declare Unknown Defines in enforce.project.json** writes them into the
  manifest as `false`.

Priority: manifest > declaration in a live context > off.

### Types that will never compile

If every declaration of a class sits under a guard that is off, its usages are dimmed
(`enforce.dimDeadTypes`) and reported (`enforce.deadTypeDiagnostics`) with the guilty guard
named — this is the engine's `Unknown type`, shown before the build instead of after it.

Only decidable guards are reported: explicitly `false` in the manifest, or declared nowhere.
Guards the engine declares on the C++ side are left alone.

## Formatter

Format a document or a selection with the standard VS Code commands. The built-in formatter is
used by default; its defaults were measured on the real corpus — tabs, Allman braces, space
after keywords.

```jsonc
"enforce.formatting": {
  "useSpaces": false,
  "tabSize": 4,
  "allmanBraces": true,
  "spaceAfterKeyword": true,
  "astylePath": "",        // set it to use AStyle instead (falls back to built-in on error)
  "astyleRcPath": "",
  "astyleArgs": []
}
```

## Syntax and semantic colors

The extension ships a TextMate grammar plus semantic tokens: methods, classes, properties,
parameters, variables, enum members and type parameters are colored by what the index knows
about them, not by a regex guess.

## Index

- Built on activation, kept warm in `%LOCALAPPDATA%/EnforceTools/cache`.
- Workspace changes are picked up instantly by the file watcher and on window focus.
- Changes outside the workspace (engine, dependencies) are revalidated every
  `enforce.autoRefreshSeconds` (300 by default, `0` = off).
- **Enforce: Rebuild Index** forces a full re-index.

## Commands

| Command | Description |
| --- | --- |
| `Enforce: Rebuild Index` | Full re-index |
| `Enforce: Show All Modifications (discovery)` | Every `modded` layer of a class |
| `Enforce: Declare Unknown Defines in enforce.project.json` | Add unknown defines to the manifest as `false` |
| `Enforce: Create Project File` | Create `enforce.project.json` |

All settings are listed in [Manifest and launch.json](06-manifest.md#vs-code-settings).
