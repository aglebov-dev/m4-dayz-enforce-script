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
| Diagnostics | syntax, values, inheritance — table below |
| Quick fixes | `Ctrl+.` on most complaints |
| Formatting | `Shift+Alt+F`, long arrays are expanded item per line |
| Highlighting | a type name is coloured only when it resolves against the corpus |

## Diagnostics

Every rule was checked against 785 configs from vanilla, Expansion and M4: no false
positives. The level follows the consequence, not the look of the entry.

| Code | Level | What it catches |
| --- | --- | --- |
| `required-addon-missing` | error | a `requiredAddons[]` entry is not loaded |
| `class-not-available` | error | the base is missing from the project, the dependencies and the engine |
| `base-not-declared` | error | a base from another addon without `class X;` in this file |
| `self-base-class` | error | `class X: X` at the top level |
| `array-without-brackets` | error | `healthLevels = {…}` without `[]` on the name |
| `value-missing-semicolon` | error | a missing `;` — the value swallows the next line |
| `string-not-closed`, `array-not-closed` | error | an unclosed quote, an unclosed `{` |
| `brace-not-closed`, `stray-brace` | error | `{}` balance across the file |
| `unquoted-expression` | error | `x = 1 + 2;` is not evaluated without quotes |
| `broken-expression` | error | `"(0.4/4"` — the evaluator silently yields 0 |
| `bool-literal` | warning | `x = true;` is stored as a string, numeric reads give 0 |
| `int-overflow` | warning | an integer outside 32 bits |
| `bad-number` | warning | `0.6.1`, `12ab` — becomes a string |
| `value-shape-mismatch` | warning | the shape differs from the corpus: "string here, number in 133 of 134 entries" |
| `unquoted-value` | info | `scope = asdasd;` — an unquoted value |
| `value-empty` | info | `maxCargo = ;` |
| `array-empty-item` | info | `{"a",,12}` — an empty element appears |
| `array-trailing-comma` | info | `{1,2,}` — the engine ignores it |
| `array-item-run-on` | warning | `{12 "path"}` — a comma is missing between values |

What matters about values (verified by probes on a live server):

- **numbers and strings are not always interchangeable.** The engine and the object getter
  (`item.ConfigGetInt("weight")`) read `"200g"` as 200 and evaluate `"0.3926/4"`; the
  global `GetGame().ConfigGetInt(path)` returns 0 for any string;
- **`int` and `float` are indistinguishable** — the same `20` is stored either way in vanilla;
- **`true`/`false` are not booleans**: write `0`/`1`;
- there is no "version" type — `0.6.1` becomes a string.

## Quick fixes

`Ctrl+.` offers a fix where it is unambiguous: replace `true` with `1`, quote the value,
add `;`, close a quote or a brace, add `[]`, declare the base (`class X;`), drop `: X`
from a self-referencing class, remove a stray comma.

Where several outcomes are possible — an `int` overflow, an unclosed block, junk such as
`"0..7"` — there is no fix: guessing is worse than leaving the decision to the author.

## Formatting

`Shift+Alt+F`: tab indentation, `{` on its own line, `};` at the end of a body, spaces
around `=`. Values are never rewritten — only indentation, spacing and line breaks change,
so the formatter is safe even on a broken file.

Long arrays are expanded item per line, nested ones recursively:

```cpp
healthLevels[] =
{
	{
		1,
		{
			"data\jacket.rvmat",
			"data\knop.rvmat"
		}
	},
	…
};
```

The line threshold is `enforce.configFormatting.maxLineLength` (120 by default).

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
  nearest ancestor first, anything already written in this body is not offered; for a
  nested class (`Protection`, `DamageSystem`) properties also come from same-named classes
  across the corpus — the owner's inheritance chain knows nothing about them;
- an array property is inserted with `[]` already attached;
- after `:` — class names, inside `requiredAddons[]` — addon names, including OUTSIDE the
  quotes: the quotes are added on insert;
- **after `=` — the values this property actually takes across the corpus**, with the count
  in the label (`scope` → `1` 9115×, `2` 3280×, `0` 361×). Only dictionary-like properties
  are offered — up to 12 distinct values with a noticeable frequency; `displayName` or
  `model` have unique values, so nothing is suggested there;
- inside a value (`{…}`, after `=` within an array) no list is shown at all;
- a match means an exact occurrence of the typed text.

## Foreign configs

Engine and dependency files open read-only (`enforce.readonlyExternal`) and are marked
with a lock in the tree. Engine data is available as the `engine data` root.

## Not there yet

- checking whether a value SUITS this particular property by meaning: we only compare the
  shape of the entry against the corpus, configs have no property contracts at all;
- `#include` in configs is not expanded;
- the load order from `load[]` is not wired into config analysis: we show WHAT overrides
  what by inheritance, but do not claim which value wins between mods.
