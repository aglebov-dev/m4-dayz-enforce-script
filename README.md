# Enforce Tools (DayZ)

[Русская версия / Russian version](README.ru.md)

VS Code extension for **DayZ Enforce Script**. It indexes your mod, its dependencies and the
engine scripts with every `modded` layer, and adds type-aware editing, a references panel, a
type explorer, an engine build gate and a script debugger.

Marketplace: **`m4rf.enforce-script-tools`**.

> **Beta.** Expect bugs and changes between releases. Bug reports are welcome:
> [issues](https://github.com/aglebov-dev/m4-dayz-enforce-script/issues).

## Documentation

| Section | Contents |
| --- | --- |
| [01 · Code and editor](docs/en/01-editor.md) | Completion, navigation, hover, inlay hints, diagnostics, defines, formatter |
| [02 · References and search](docs/en/02-references.md) | References panel, CodeLens, `modded` layer discovery, type search |
| [03 · Type explorer](docs/en/03-type-explorer.md) | Project tree, dependency management, file operations |
| [04 · Build](docs/en/04-build.md) | PBO packing, engine compile on `F5`, pre-flight |
| [05 · Debugging](docs/en/05-debugging.md) | Breakpoints in DayZDiag, run server/client, limits |
| [06 · Manifest and launch.json](docs/en/06-manifest.md) | Full `enforce.project.json` reference, launch configurations, VS Code settings |
| [07 · Working with config.cpp](docs/en/07-config.md) | Config index, hover, references, completion, foreign configs |

## Requirements

- VS Code 1.85 or newer.
- Unpacked engine scripts (**DayZ Tools** → *Extract Game Data*, or any local copy).
- Windows for the build gate and the debugger — they run `DayZServer_x64.exe` and
  `DayZDiag_x64.exe`. The analyzer itself (index, completion, navigation, diagnostics,
  pre-flight) works anywhere VS Code does: the parser is wasm.

## Quick start

1. Unpack the engine scripts (`1_Core` … `5_Mission`).
2. Create `enforce.project.json` next to your mod's `config.cpp` — command
   **Enforce: Create Project File**, or the button in the empty explorer:

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

3. Add `.vscode/launch.json` and press `F5` to build and ask the engine:

   ```jsonc
   {
     "version": "0.2.0",
     "configurations": [
       { "type": "enforce", "request": "launch", "name": "Enforce: Build & Check" }
     ]
   }
   ```

Details: [Manifest and launch.json](docs/en/06-manifest.md).

## Third-party mods

**Unpacking `.pbo` files, decoding binarized configs and recovering obfuscated code may violate
the restrictions a mod author placed on their work.** The extension provides the technical
capability — using it is entirely your responsibility. By adding someone else's mod to
`enforce.project.json` you confirm that you are entitled to do so.

The purpose is developing **your own** code against a dependency: you need its public API and
nothing beyond that. Obfuscated mods are therefore opened as **signatures only** — class
headers, method signatures, field types. Method bodies are never extracted.

## License

MIT — see [LICENSE](LICENSE). The license covers what is distributed: the extension package
published on the Marketplace and the documentation in this repository.

**The source code is not published.** The `.vsix` ships a single bundled and minified file, not
readable source, and this repository holds documentation only. MIT is a permissive license and
does not require publishing sources — this note is here so the scope is explicit rather than
inferred.

Third-party components inside the package — the tree-sitter runtime, the Enforce Script grammar
and the VS Code codicons — keep their own licenses; their notices ship with the extension in
`THIRD-PARTY-NOTICES.txt`.
