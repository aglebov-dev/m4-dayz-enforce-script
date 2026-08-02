# Build

[← Documentation index](../../README.md) · [Русская версия](../ru/04-build.md)

Two entry points:

| Key | What happens |
| --- | --- |
| `F5` | Pack the PBOs, then compile them on a headless DayZServer and report the verdict |
| `Ctrl+Shift+B` | Pack the PBOs only (task `enforce: build PBO`) |

`F5` needs a launch configuration — see [Manifest and launch.json](06-manifest.md#launchjson).

Windows only: the gate runs `DayZServer_x64.exe`. Elsewhere it reports that the server was not
found; the analyzer keeps working.

## Packing

- Every module with a `config.cpp` is packed into its own PBO — no AddonBuilder or other tools
  involved.
- The PBO prefix is not configurable: it comes from each module's `config.cpp`, the same source
  the debugger's path map uses.
- Only scripts go in. That is all the engine needs for a verdict; asset packing comes later
  (`build.assets` is reserved).
- Output: `<outDir>/<modName>/addons/*.pbo`.

## Dependencies

The engine gets the same `load[]` the analyzer indexes — there is no second list:

- a ready mod (`@Mod` folder or `.pbo`) is passed to `-mod=` as is;
- sources with a `config.cpp` are built into `<outDir>/deps/@Name` first.

One source of truth: "green in the editor, red in the build" has nowhere to come from.
`build.deps` and `build.mods` are removed — they are ignored with a warning.

## The verdict

- Root causes are separated from the cascade behind them.
- Every error is mapped back to your file and appears in Problems.
- Paths in the Output log are clickable, including errors inside dependencies.
- If the engine pops a modal dialog (`Addon 'X' requires addon 'Y'` and friends), it is closed
  automatically and the text goes to Output — the build never waits for a human click.
- Logs of previous runs sitting in the same profile folder are ignored by timestamp.
- If the server dies on its own, the run fails immediately instead of waiting for
  `build.timeoutSec`.
- After a run the engine's real define list is offered for import into the manifest — only what
  no source declares.

A failed run leaves the server dying in the background; the next `F5` waits for it and retries
the PBO write if the file is still locked.

## Pre-flight

With pre-flight on, the built-in analyzer runs over the whole project **before** the engine
starts, so a typo does not cost a round trip through the server.

```jsonc
{ "engine": "P:/scripts", "build": { "preflight": true } }
```

- Stops only on what the engine will not accept: parse errors, call arity, undeclared symbols.
- The dialog always offers **Build anyway**.
- Findings are tagged `[pre-flight]` and stay in Problems until the next run. Editing a file
  clears its own marks; editing the manifest does not — changing `defines` flips the verdict
  everywhere, so run it again.
- `build.preflight` in the manifest overrides the `enforce.preflight` setting.

Pre-flight is not as complete as the real compiler's verdict, but it is the way to work without
a DayZ Server installed: `F5` checks the project, shows the errors and then honestly reports the
missing server.

## Where the server is

```jsonc
"build": {
  "serverExe": "D:/Steam/steamapps/common/DayZServer/DayZServer_x64.exe",
  "serverCwd": "D:/Steam/steamapps/common/DayZServer",
  "serverCfg": "serverDZ.cfg",
  "mission": "mpmissions/dayzOffline.chernarusplus",
  "port": 2402,
  "profilesDir": "./builds/profiles",
  "timeoutSec": 240
}
```

Drop `serverExe` and `serverCwd` to auto-detect through the Steam paths. Omit `port` and a
random one from 2416..2465 is used, so a stale server does not block the run. `DayZDiag_x64.exe`
also works as the gate, but it needs a running Steam client.

Full key reference: [Manifest and launch.json](06-manifest.md).

## Commands

| Command | Description |
| --- | --- |
| `Enforce: Build Check (Engine Compile)` | PBO + engine compile (`F5`) |
| Task `enforce: build PBO` | PBO only (`Ctrl+Shift+B`) |
