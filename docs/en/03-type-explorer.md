# Type explorer

[← Documentation index](../../README.md) · [Русская версия](../ru/03-type-explorer.md)

The **M4** icon in the activity bar opens a tree of everything the extension knows about the
project. The tree mirrors the disk and is lazy — a corpus of 8000 classes opens instantly.

```
@MyMod                       ← solution root, name from build.modName
  Properties                 ← the manifest
  Run and debug              ← collapsed by default, see 05-debugging.md
  Dependencies  5 entries    ← the engine and every load[] entry
  MyMod  project             ← your sources, folder for folder
```

- A folder with a `config.cpp` is drawn as a mod; the `config.cpp` expands into `CfgPatches` /
  `CfgMods`, and its `#define`s become nodes with their values (otherwise `*_Preload` addons
  look empty).
- Read-only entries are marked with a lock.
- No `enforce.project.json` yet — the panel offers to create it. Engine scripts not found — an
  **engine — not set** row appears with a button to pick the folder.

## Toolbar

| Button | What it does |
| --- | --- |
| Search Types | Filter the tree by name; the button turns into Clear Search |
| Filter Symbol Kinds | Limit the tree to classes, methods, fields, enums… |
| Refresh Types | Re-read the tree |
| Show Current File in Type Explorer | Reveal the active editor's file |
| Collapse All / Expand Project | Fold everything / unfold your own mods |
| Only Scripts | Trim the tree down to code, hiding non-script files |

**Show in Type Explorer** in the editor context menu reveals the definition under the cursor.

## Dependencies

Managed from the tree, not by hand. The `Dependencies` context menu has two entries — they open
different dialogs because the Windows file picker cannot offer files and folders at once:

- **Add Mod Folder…** — a folder with `config.cpp` inside (`@Mod`, or its `addons` folder);
- **Add Single PBO…** — one packed addon, unpacked into the cache;
- **−** on an entry removes it from the manifest — nothing on disk is touched, and comments in
  the manifest survive the edit.

A config-only dependency (no `.c` files at all) is added the same way and is indexed in full.

Next to the manifest the extension keeps `enforce.deps.json`: the hash of every pbo and its
unpack folder, so "is this the same mod?" has an answer. Packed mods are unpacked into the
extension cache (scripts and configs only; binarized configs are decoded back to text) and
refreshed when the mod changes on disk.

The cache is a mirror of the PBO, not a working copy — edits inside it are wiped. To change a
dependency's code, unpack the PBO yourself and add that folder as sources.

Obfuscated mods open in *API-only* mode: declarations without bodies. The API may come out
partial, or not at all — and such mods are excluded from the debugger's path map
(see [Debugging](05-debugging.md#limits)).

Mods packed as a token `#ifdef` soup are skipped: neither API nor bodies come out of them.

## File operations

The explorer is a viewer by default: the context menu copies paths and reveals files in the OS
explorer. Set `enforce.explorerEdit` to `true` to add file operations:

`New Mod…`, `Add Existing Mod…`, `New Folder…`, `New Script…`, `New config.cpp`, `Rename…`,
`Delete`.

They appear only on your own sources — engine and dependency entries stay read-only.

## Commands

| Command | Description |
| --- | --- |
| `Enforce: Search Types` / `Clear Search` | Filter the tree by name |
| `Enforce: Filter Symbol Kinds` | Limit the tree to chosen symbol kinds |
| `Enforce: Only Scripts` | Code-only view |
| `Enforce: Refresh Types` / `Collapse All` / `Expand Project` | Tree control |
| `Enforce: Show Current File in Type Explorer` / `Show in Type Explorer` | Reveal a file or a definition |
| `Enforce: Add Dependency…` / `Remove Dependency` | Edit `load[]` from the tree |
| `Enforce: Set Engine Scripts Folder…` | Point the extension at the engine scripts |
| `Enforce: Create Project File` | Create `enforce.project.json` |
