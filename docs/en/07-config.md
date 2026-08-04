# Working with config.cpp

[← Back to contents](../../README.md) · [Русская версия](../ru/07-config.md)

Configs are indexed separately from scripts: the project, the dependencies from `load[]`
and the engine data (`DZ` next to `scripts`). The index is built on the first `config.cpp`
request; after a save only the saved file is re-read.

A config is a single tree merged from every loaded addon, so classes are addressed by
FULL PATH: the same name under `CfgPatches`, `CfgMods` and `CfgVehicles` means three
different classes.

## What is supported

| Feature | Where |
| --- | --- |
| Completion | classes, addons in `requiredAddons[]`, properties along the inheritance chain |
| Hover | classes, addons, properties |
| `F12` | class base, `class X;`, class name in a string, addon, overridden property |
| `Shift+F12` | **Enforce References** panel — the same one as for scripts |
| Diagnostics | a `requiredAddons[]` entry is not loaded; a type missing from every dependency |

## Hover

**Class** — full path (every segment is clickable), origin, base chain:

```
class
● defines (1)   ● usages

CfgVehicles.TShirt_Black (engine)

inherits:
→ TShirt_ColorBase
→ Clothing
→ Clothing_Base
→ Inventory_Base
```

**Nested class** — instead of a base chain it shows LAYERS: the same-named nested classes
in the owner's ancestors. That is where the properties come from, even though there is not
a single `:` in the text:

```
CfgVehicles.Animal_BosTaurus.DamageSystem (engine)
● defines (1)   ● layers (2)   ● usages

inherited from:
→ AnimalBase.DamageSystem
→ AllVehicles.DamageSystem
```

`class GlobalHealth: GlobalHealth` in vanilla is not a cycle but "extend the inherited
one": the name on the right resolves to the ancestor's same-named nested class.

**Property** — the path down to the property itself and what it overrides, with values.
For `+=` the section is called `appends to:` — that is an append to the inherited array.
Arrays longer than 100 characters are shown as `{.....}`.

```
property
● defines (3)   ● usages

CfgVehicles.TShirt_Black.scope

overrides:
→ Clothing_Base.scope = 0
→ Inventory_Base.scope = 0
```

**Addon** — origin and `requiredAddons[]` as a list, every dependency is clickable.

## References panel

`Shift+F12` on a class, addon or property opens the same panel as for scripts.
Definitions come first as a group, usages below. The `File` column holds the path relative
to the mod, `Mod` the mod itself: every config is called `config.cpp`, otherwise the rows
are indistinguishable.

The popup links (`defines` / `layers` / `usages` / `all`) open the panel with a pre-filter.

## Completion

- inside a class body — properties of all ancestors, each row shows its source class;
  nearest ancestor first, anything already written in this body is not offered;
- after `:` — class names, inside `requiredAddons[]` — addon names;
- a match means an exact occurrence of the typed text.

## Foreign configs

Engine and dependency files open read-only (`enforce.readonlyExternal`) and are marked
with a lock in the tree. Engine data is available as the `engine data` root.

## Not there yet

- **value validation** — config field types and checking what is written in them come
  later;
- `#include` in configs is not expanded;
- the load order from `load[]` is not wired into config analysis: we show WHAT overrides
  what by inheritance, but do not claim which value wins between mods.
