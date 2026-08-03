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

## What the extension runs on your machine

The analyzer itself starts nothing: index, completion, navigation, diagnostics and pre-flight
are pure computation inside the extension host. Processes appear only when you ask for a build
or a debug session, and there are exactly three of them:

| Process | When | Why |
| --- | --- | --- |
| `DayZServer_x64.exe` (or `DayZDiag_x64.exe`) | `F5` | Compiles the packed mod and returns the real engine verdict |
| `DayZDiag_x64.exe` | **Run server** / **Run client** | The debug session, see [05 · Debugging](05-debugging.md) |
| `powershell.exe` | alongside an engine build, Windows only | Closes the engine's modal dialogs — details below |

### Why PowerShell is involved

When the engine hits a missing addon it does not fail — it opens a modal `MessageBox` and waits
for a click. In one real run that stalled the build for 19 seconds, and with nobody at the
screen the build dies on the stall detector instead of reporting the actual problem. A verdict
that depends on whether a human is watching is not a verdict.

So during an engine build the extension starts one PowerShell helper. It uses `user32.dll`
through P/Invoke, because that is the only way to see a window without shipping a native module
or adding a dependency. What it does, and nothing else:

1. enumerates top-level windows and keeps those that belong to **the PID the extension started
   itself** — no other process is ever inspected;
2. of those, keeps only windows of class `#32770` (the standard Windows dialog) that have both
   text and a button — the engine also creates empty service windows of that class, and touching
   them kills the run;
3. copies the dialog text into the build Output, so the reason reaches you;
4. posts `BM_CLICK` to the first button, i.e. presses **OK**;
5. exits when the build ends.

The script is written to a temporary `.ps1` file and executed with `-File`, so it can be read
while it runs; it is not passed as an encoded command. No network access, no writes outside the
temporary file and your build folder, nothing persists after the build.

## Log levels

`enforce.build.logLevel` — what part of the engine log reaches the build channel.

| Value | What the channel keeps |
| --- | --- |
| `errors` | errors only |
| `important` (default) | errors, warnings and milestones — `Module: …`, mission load, shutdown countdown |
| `full` | everything, including the raw RPT tail printed when the server dies on its own |

`enforce.run.logLevel` — the same for a debug run, see [05 · Debugging](05-debugging.md#logs).

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
