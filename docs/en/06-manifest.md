# Manifest and launch.json

[← Documentation index](../../README.md) · [Русская версия](../ru/06-manifest.md)

## `enforce.project.json`

Lives in the project root, next to your mod's `config.cpp`. Every key is optional — the minimal
working file is `{}`. Create it with **Enforce: Create Project File** or from the empty type
explorer; the engine path and the mod name are filled in when they can be detected.

Keys are completed as you type, unknown keys and wrong value types are underlined immediately
(JSON schema + validator). Both `/` and `\\` work in paths; `${WORKDRIVE}` and `${env:NAME}` are
expanded.

```jsonc
{
  // ===== WHAT GETS INDEXED =================================================
  // Unpacked game scripts (the folder with 1_Core … 5_Mission).
  // Omitted = <workDrive>/scripts.
  "engine": "${WORKDRIVE}/scripts",

  // Dependencies in load order: sources, an unpacked addon, or a packed mod
  // (@Mod with addons/*.pbo, a folder of .pbo, or a single .pbo).
  // THE SAME LIST is passed to the engine as -mod= during the build.
  "load": [
    { "name": "CF",        "path": "D:/mods/DayZ-CommunityFramework/JM/CF" },
    { "name": "Expansion", "path": "D:/mods/DayZ-Expansion-Scripts" },
    { "name": "COT",       "path": "D:/Steam/steamapps/common/DayZ/!Workshop/@Community-Online-Tools" }
  ],

  // Your own code: linted and packed into PBOs. Omitted = the manifest folder.
  "project": [
    { "name": "MyMod", "path": "./src" }
  ],

  // ===== WHAT COUNTS AS ENABLED ============================================
  // Only what sources cannot show: engine defines and those from closed PBOs.
  // Anything declared in code or dependencies is picked up automatically.
  // Values must be true/false.
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
    "serverExe": "D:/Steam/steamapps/common/DayZServer/DayZServer_x64.exe",
    "serverCwd": "D:/Steam/steamapps/common/DayZServer",

    "serverCfg": "serverDZ.cfg",
    "mission": "mpmissions/dayzOffline.chernarusplus",
    "port": 2402,                   // omitted = random 2416..2465
    "profilesDir": "./builds/profiles",
    "timeoutSec": 240
  },

  // ===== SCRIPT DEBUGGING ==================================================
  "debug": {
    "diagExe": "D:/Steam/steamapps/common/DayZ/DayZDiag_x64.exe",
    "serverConfig": "D:/Steam/steamapps/common/DayZServer/serverDZ.cfg",
    "mission": "mpmissions/dayzOffline.chernarusplus",
    "port": 2302,
    "playerName": "dev"
  }
}
```

### Key reference

| Key | Default | Description |
| --- | --- | --- |
| `engine` | `<workDrive>/scripts` | Unpacked engine scripts |
| `load[]` | `[]` | Dependencies in load order: `{ name, path }`. Indexed, never linted, never packed; passed to the engine as `-mod=` |
| `project[]` | manifest folder | Your sources: `{ name, path }`. Linted and packed |
| `defines` | `{}` | Preprocessor defines, `true`/`false` only |
| `environments` | `{}` | Runtime names declared on the C++ side, with values |
| `externalReferences` | `false` | References and CodeLens on engine and dependency files |
| `references.includeTextMatches` | `false` | Text matches in the references panel |
| `build.modName` | `@Mod` | Wrapper mod folder name |
| `build.outDir` | `./builds` | Where the mod folder is created |
| `build.preflight` | `false` | Analyze the project before starting the engine |
| `build.serverExe` | auto | `DayZServer_x64.exe`; omitted = auto-detect via Steam |
| `build.serverCwd` | folder of `serverExe` | Working folder |
| `build.serverCfg` | `serverDZ.cfg` | Server config for the build run |
| `build.mission` | `mpmissions/dayzOffline.chernarusplus` | Mission for the build run |
| `build.port` | random 2416..2465 | Server port |
| `build.profilesDir` | — | Profiles folder (logs) |
| `build.timeoutSec` | `240` | Build timeout |
| `build.assets` | `false` | Reserved: pack assets too |
| `build.deps` / `build.mods` | — | **Removed**, ignored with a warning. Use `load[]` |
| `debug.diagExe` | auto | `DayZDiag_x64.exe`, from the **client** folder |
| `debug.serverConfig` | next to `serverExe` | Live `serverDZ.cfg` used as the source for the debug copy |
| `debug.mission` | `build.mission` | Mission for the debug run; relative resolves against the server folder |
| `debug.port` | `2302` | Game port of the debug server |
| `debug.playerName` | `dev` | Name the debug client joins with |

Precedence for build options: `build{}` in the manifest > the `enforce.build` setting > auto-detection.

## `launch.json`

Two configurations, both of type `enforce`. VS Code offers them itself: *Run and Debug* →
*create a launch.json file* → **Enforce Build & Check**.

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    // F5: pack the PBOs and compile them on a headless DayZServer
    { "type": "enforce", "request": "launch", "name": "Enforce: Build & Check" },

    // F5: listen for the server side, then build and start the server
    {
      "type": "enforce",
      "request": "launch",
      "name": "Enforce: Run Server (debug)",
      "listen": "server",
      "run": "server"
    },

    // Attach to a DayZDiag you started yourself
    {
      "type": "enforce",
      "request": "attach",
      "name": "Enforce: Attach to DayZDiag (client)",
      "port": 1000,
      "workDrive": "P:"
    }
  ]
}
```

| Attribute | Applies to | Default | Description |
| --- | --- | --- | --- |
| `listen` | launch | — | Take the debugger port before starting: `server` (1001), `client` (1000), `off` |
| `run` | launch | — | Build the PBO and start that side: `server`, `client`, `none` |
| `port` | attach | `1000` | Port the game connects to: 1000 client, 1001 server, 1002 second client |
| `workDrive` | attach | `P:` | Work drive with the unpacked scripts — engine paths in the call stack are expanded from it |

A `launch` configuration without `listen` and `run` only builds and checks; everything else
comes from `build{}` in the manifest. See [Debugging](05-debugging.md).

`Ctrl+Shift+B` runs the `enforce: build PBO` task and needs no configuration.

## VS Code settings

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
| `enforce.build` | `{ "mission": …, "timeoutSec": 240 }` | Build defaults, overridden by `build{}` in the manifest |
| `enforce.build.logLevel` | `important` | What part of the engine log reaches the build channel: `errors` / `important` / `full` |
| `enforce.run.logLevel` | `important` | The same for a debug run |
| `enforce.readonlyExternal` | `true` | Engine and dependencies open read-only |
| `enforce.autoRefreshSeconds` | `300` | Index revalidation interval, seconds (0 = off) |
| `enforce.formatting` | `{}` | `useSpaces`, `tabSize`, `allmanBraces`, `spaceAfterKeyword`, `astylePath`, `astyleRcPath`, `astyleArgs` |
| `enforce.explorerEdit` | `false` | Allow file operations from the type explorer |

The extension activates on a workspace containing `enforce.project.json`, or when the type
explorer is opened.
