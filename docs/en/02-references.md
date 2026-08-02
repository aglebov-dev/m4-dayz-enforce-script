# References and search

[← Documentation index](../../README.md) · [Русская версия](../ru/02-references.md)

## References panel

`Shift+F12` (or **Enforce: Find All References (Panel)**, or the editor context menu) opens the
**Enforce References** panel at the bottom. Declarations and `modded` layers are grouped on
top, usages below — split into calls, reads, writes, constructors and callbacks.

Columns: `Code`, `Kind`, `File`, `Line`, `Mod`, `Module`, `Containing member`,
`Containing type`. Columns are resizable, sortable and each has a substring filter.

| Toolbar | What it does |
| --- | --- |
| ⊟ collapse all / ⊞ expand all | Fold or unfold the groups |
| ⟲ reset filters | Drop all column filters |
| ⊕ include text matches | Show plain substring hits (string literals, config keys) |
| ↻ refresh | Re-run the query against the current index |
| 📌 pin | Pin the result set to an editor tab |
| highlight results | Highlight every hit in the visible editors |

Text matches are hidden by default — they have no semantics behind them and flood the list on a
popular name. The default is `enforce.references.includeTextMatches` (or
`references.includeTextMatches` in the manifest); the toolbar toggle always works regardless.

Ten thousand rows do not lag.

## What is searched

By default only project sources. Engine and dependency files are included when
`externalReferences` is on in the manifest:

```jsonc
"externalReferences": true
```

It is off by default because popular engine types have thousands of usages.

Resolution is type-aware: a name is matched to the symbol it actually refers to, not to every
namesake in the corpus. Overloads are narrowed by argument count and argument types.

## CodeLens

`N references` / `N layers` above declarations in project files, in the Visual Studio style.
Click to open the panel. Switched off with `enforce.codeLensReferences`.

## Defines

`Shift+F12` on a define lists every `#ifdef` / `#ifndef` that tests it, `F12` goes to the
declaration. See [Code and editor](01-editor.md#preprocessor-defines).

## Layer discovery

**Enforce: Show All Modifications (discovery)** lists every `modded` layer of a class across the
project, the dependencies and the engine, in load order — the same data the hover popup shows,
in list form.

## Type search

Type search lives in the type explorer: the **Search Types** button filters the tree by name,
**Filter Symbol Kinds** limits it to classes, methods, fields and so on. See
[Type explorer](03-type-explorer.md#toolbar).

## Commands

| Command | Description |
| --- | --- |
| `Enforce: Find All References (Panel)` | References panel for the symbol under the cursor (`Shift+F12`) |
| `Enforce: Show All Modifications (discovery)` | Every `modded` layer of a class |
