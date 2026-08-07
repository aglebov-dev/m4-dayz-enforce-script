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
- After the word `override` — ready-made signatures of base methods with a body skeleton,
  taken from parents and from the previous `modded` layer.

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

One thing worth knowing about indexing: `x[k]` is not limited to `array` and `map` — on a
regular class the brackets expand into its own `Get`/`Set`. Without the required method the
engine refuses to compile the file (`Undefined function 'X.Get'`), and that is what
`index-without-getset` reports. For a write `x[k] = v` a `Get` alone is not enough: `Set`
must exist.

### Rules added in 0.6.2

| Rule | Does not compile | This is fine |
| --- | --- | --- |
| `override-no-base` | `override void EEKilled()` when the base takes `EEKilled(Class killer)` | a different signature *without* `override` — that is an overload |
| `static-local-init` | `static int k = n;` (`n` is a parameter or local) | `static int k = 5;` and `static` constants |
| `array-type-position` | `int[] a;` as a local or a field | in a parameter, a return type and a `typedef` |
| `vector-literal-malformed` | `vector v = "1 2";`, `"abc"`, `"1,2,3"` | three numbers or more, decimals, exponent |
| `enum-junk-keyword` | `enum { AA, ref BB = 1 }` | `new` in a value: `CC = new -1` |
| `else-without-if` | an `else` with no `if` of its own | — |
| `new-non-type` | `new x` where `x` is a variable | `new T` even without parentheses |
| `generic-double-ref` | `map<string, ref ref array<int>>` | `ref ref array<int> m_A;` on a field |
| `less-than-statement` | `n < 2;` as a statement | `n <= 2;`, `n > 2;`, `bool b = n < 2;` |
| `postfix-in-expression` | `a++ + 2`, `if (a++ > 0)` | `a++;`, `int b = a++;`, `(a++) + 2`, `for (…; i++)` |
| `unary-plus` | `int x = +5;` | unary minus: `int x = -5;` |
| `new-in-comparison` | `a == new array<int>()` | `T x = new T();`, `Print(new T())`, `a == null` |

The "two statements on one line" rule covers assignments, class fields, `enum` members and a
`return` before the closing brace. A line break the engine forgives, a run-on it does not:

```c
k = k | 1 k = k & 3;      // ❌ «Broken expression»
enum E { AA = 1 BB = 2 }  // ❌ on one line
enum E { AA = 1
         BB = 2 }         // ✅ with a line break it compiles
```

To check the whole project before a build instead of file by file, see
[pre-flight](04-build.md#pre-flight).

## Built-in macros

The Enforce preprocessor has exactly two built-in macros — `__LINE__` and `__FILE__` — and
both expand to a STRING: `int n = __LINE__;` does not compile, `string s = __LINE__;` does.
The analyzer knows them, completion offers them.

The Arma heritage (`__DATE__`, `__TIME__`, `__COUNTER__`, `__FUNCTION__`) does not exist
here: the compiler answers "Can't find variable" for such a name.

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
