# Debugging

[← Documentation index](../../README.md) · [Русская версия](../ru/05-debugging.md)

Breakpoints, call stack, variables, watches and a debug console for Enforce Script, driven from
VS Code.

Debugging works **only on `DayZDiag_x64.exe`** — release builds have no script debugger at all.
Windows only.

## How it is wired

```
VS Code ──DAP── extension adapter ──TCP 1000/1001── DayZDiag_x64.exe
```

The game connects to the debugger **by itself**; the port decides the role and no command line
switch is involved:

| Port | Side |
| --- | --- |
| 1000 | client |
| 1001 | server |
| 1002 | second client |

Only one session at a time — the engine hangs on two. If the port is busy, the Workbench Script
Editor is usually holding it.

## Running it

Open the type explorer → **Run and debug** (under `Properties`):

| Row | What it does |
| --- | --- |
| `Listen for DayZDiag` | Holds the port of the chosen side (`client` / `server` / `off`). When the game connects, the debug session starts on its own — nothing to press. |
| `Run server` | Starts the diag as a server with a debug copy of the config |
| `Run client` | Starts the diag as a client and connects it to that server |

A running process is stopped by the same button. The same actions live in the palette:
**Enforce: Listen for DayZDiag**, **Enforce: Run Server**, **Enforce: Run Client**,
**Enforce: Stop Server and Client**.

Order that works: turn the listener on for the side you want, then start that side, then attach
the other one if you need it. A diag client will not connect to a release server (`0x00020017`)
— run the server with the diag too.

To attach to a game you started yourself, use the attach configuration in `launch.json` — see
[Manifest and launch.json](06-manifest.md#launchjson).

## Configuration

Everything is optional: the diag is looked up in the standard Steam folders, `serverDZ.cfg` is
taken from next to the server executable, the mission from `build.mission`.

```jsonc
"debug": {
  "diagExe":      "D:/Steam/steamapps/common/DayZ/DayZDiag_x64.exe",
  "serverConfig": "D:/Steam/steamapps/common/DayZServer/serverDZ.cfg",
  "mission":      "mpmissions/dayzOffline.chernarusplus",
  "port":         2302,
  "playerName":   "dev"
}
```

| Key | Notes |
| --- | --- |
| `diagExe` | Take it from the **client** folder. The copy shipped with a server install is usually older than the game data and fails with `Multiple declaration of function 'GetGame'`. |
| `serverConfig` | A **live** `serverDZ.cfg`. The extension copies it and forces `BattlEye=0`, `verifySignatures=0`, `forceSameBuild=0`, `allowFilePatching=1`; the original is never touched. A hand-made minimal config is not enough — the engine shuts the server down after 10 seconds. |
| `mission` | A relative path is resolved against the **server** folder, not the diag folder. A wrong path starts the server without the mission `init.c`: `player connect will stay disabled`, no economy, no clients. |
| `port` | Game port of the debug server; the client connects to the same one. Not the debugger port — that one is fixed by the engine. |
| `playerName` | Name the debug client joins with (`-name=`). |

## What you get on a break

- Breakpoints in your own code, in dependencies, in the engine scripts and in the mission's
  `init.c`.
- Call stack with the real files — paths are translated from the engine's PBO form back to disk,
  including letter case, so a frame never opens a second copy of the file.
- **Variables**: `Locals` and `This`, nested objects expanded. Values are shown as content
  rather than an address: `Param1<string> {param1='SCRIPT : …'}`. Where the engine sends no
  fields for `this`, the group says so explicitly instead of looking empty.
- **Watch** expressions, including member access (`params.param1`).
- **Debug Console** executes a **single statement** in the `Mission` context, so `GetGame()` is
  available:

  ```c
  GetGame().GetMission().OnInit();
  ```

  A compound statement (several operators in one line) is not executed at all, and classes from
  the `5_Mission` module are not visible by name — call through a base type.

## Limits

**Mission start-up code cannot be caught on the first pass.** The game connects to the debugger
strictly after the mission is initialized — `Module: Mission` lands at about +17 s, the debugger
connection at +18…19 s. Breakpoints in `OnInit` and in module constructors are verified, but
their execution has already happened.

Two ways around it:

1. Re-run it from the Debug Console — `GetGame().GetMission().OnInit();`. Costs a second pass
   and repeats the side effects.
2. Defer the start-up work, which catches the **first and only** execution:

   ```c
   override void OnInit()
   {
       super.OnInit();
       GetGame().GetCallQueue(CALL_CATEGORY_SYSTEM).CallLater(MyDeferredInit, 15000, false);
   }

   void MyDeferredInit()   // breakpoints here fire on the first pass
   {
       GetMyModule().Init();
   }
   ```

Everything that runs after the connection is caught normally: `OnUpdate`, events, RPCs, and
player join (`CreateCharacter`, `StartingEquipSetup` in the mission `init.c`).

Do not spend time on: `-waitForDebugger` (it waits for a **native** debugger and hangs the
server), `-donothing`, Workbench (same engine, connects no earlier), `PlayMission()` (a no-op on
the server), `AbortMission()` (drops the mission into `MissionDummy`).

**Obfuscated dependencies are excluded** from the path map — only their API is known, so line
numbers on disk do not match the real file. Breakpoints in them stay unverified instead of
stopping in the wrong place, and the console lists them by name when the session starts.

## Commands

| Command | Description |
| --- | --- |
| `Enforce: Listen for DayZDiag (debug side)` | Hold the port of a side; `off` releases it |
| `Enforce: Run Server` | Start the diag as a server |
| `Enforce: Run Client` | Start the diag as a client |
| `Enforce: Stop Server and Client` | Stop both |
