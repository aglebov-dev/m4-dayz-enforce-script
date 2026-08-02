# Манифест и launch.json

[← К оглавлению](../../README.ru.md) · [English version](../en/06-manifest.md)

## `enforce.project.json`

Лежит в корне проекта, рядом с `config.cpp` вашего мода. Все ключи необязательные, минимальный
рабочий файл — `{}`. Создать: **Enforce: Create Project File** или кнопка в пустом дереве типов;
путь к движку и имя мода подставляются, если их удалось определить.

Ключи подсказываются при наборе, неизвестный ключ и неверный тип значения подчёркиваются сразу
(JSON-схема + валидатор). В путях работают и `/`, и `\\`; раскрываются `${WORKDRIVE}` и
`${env:NAME}`.

```jsonc
{
  // ===== ЧТО ИНДЕКСИРУЕТСЯ =================================================
  // Распакованные скрипты игры (папка с 1_Core … 5_Mission).
  // Нет ключа — берётся <workDrive>/scripts.
  "engine": "${WORKDRIVE}/scripts",

  // Зависимости в порядке загрузки: исходники, распакованный аддон или
  // запакованный мод (@Mod с addons/*.pbo, папка с .pbo, либо одиночный .pbo).
  // ЭТОТ ЖЕ СПИСОК идёт движку в -mod= при сборке.
  "load": [
    { "name": "CF",        "path": "D:/mods/DayZ-CommunityFramework/JM/CF" },
    { "name": "Expansion", "path": "D:/mods/DayZ-Expansion-Scripts" },
    { "name": "COT",       "path": "D:/Steam/steamapps/common/DayZ/!Workshop/@Community-Online-Tools" }
  ],

  // Ваш код: линтуется и пакуется в PBO. Нет ключа — берётся папка манифеста.
  "project": [
    { "name": "MyMod", "path": "./src" }
  ],

  // ===== ЧТО СЧИТАТЬ ВКЛЮЧЁННЫМ ============================================
  // Только то, чего не видно из исходников: дефайны движка и закрытых PBO.
  // Объявленное в коде и зависимостях подхватывается само. Значения — true/false.
  "defines": { "PLATFORM_WINDOWS": true, "SERVER": true, "DIAG": false },

  // Имена, которые движок объявляет со стороны C++, — со значениями.
  "environments": { "DBT_OK": 0, "DMT_INFO": 1 },

  // ===== ПОИСК ССЫЛОК ======================================================
  // Разрешить ссылки и CodeLens на файлах движка и зависимостей
  // (у популярных типов бывают тысячи использований).
  "externalReferences": true,
  "references": { "includeTextMatches": false },

  // ===== СБОРКА И ПРОВЕРКА ДВИЖКОМ (F5) ====================================
  "build": {
    "preflight": true,              // своя проверка ДО запуска движка
    "modName": "@MyMod",            // имя выходной папки
    "outDir": "./builds",           // <outDir>/<modName>/addons/*.pbo

    // Отдельного списка зависимостей нет — движку идёт load[].

    // Где стоит сервер. Убрать оба ключа — найдётся сам по путям Steam.
    "serverExe": "D:/Steam/steamapps/common/DayZServer/DayZServer_x64.exe",
    "serverCwd": "D:/Steam/steamapps/common/DayZServer",

    "serverCfg": "serverDZ.cfg",
    "mission": "mpmissions/dayzOffline.chernarusplus",
    "port": 2402,                   // нет ключа — случайный из 2416..2465
    "profilesDir": "./builds/profiles",
    "timeoutSec": 240
  },

  // ===== ОТЛАДКА СКРИПТОВ ==================================================
  "debug": {
    "diagExe": "D:/Steam/steamapps/common/DayZ/DayZDiag_x64.exe",
    "serverConfig": "D:/Steam/steamapps/common/DayZServer/serverDZ.cfg",
    "mission": "mpmissions/dayzOffline.chernarusplus",
    "port": 2302,
    "playerName": "dev"
  }
}
```

### Справочник ключей

| Ключ | Умолчание | Описание |
| --- | --- | --- |
| `engine` | `<workDrive>/scripts` | Распакованные скрипты движка |
| `load[]` | `[]` | Зависимости в порядке загрузки: `{ name, path }`. Индексируются, не линтуются, не пакуются; идут движку в `-mod=` |
| `project[]` | папка манифеста | Ваши исходники: `{ name, path }`. Линтуются и пакуются |
| `defines` | `{}` | Дефайны препроцессора, только `true`/`false` |
| `environments` | `{}` | Имена, объявленные движком со стороны C++, со значениями |
| `externalReferences` | `false` | Ссылки и CodeLens на файлах движка и зависимостей |
| `references.includeTextMatches` | `false` | Текстовые совпадения в панели ссылок |
| `build.modName` | `@Mod` | Имя папки мода-обёртки |
| `build.outDir` | `./builds` | Куда кладётся папка мода |
| `build.preflight` | `false` | Проверить проект до запуска движка |
| `build.serverExe` | авто | `DayZServer_x64.exe`; нет ключа — автопоиск по Steam |
| `build.serverCwd` | папка `serverExe` | Рабочая папка |
| `build.serverCfg` | `serverDZ.cfg` | Конфиг сервера для прогона сборки |
| `build.mission` | `mpmissions/dayzOffline.chernarusplus` | Миссия для прогона сборки |
| `build.port` | случайный 2416..2465 | Порт сервера |
| `build.profilesDir` | — | Папка профилей (логи) |
| `build.timeoutSec` | `240` | Таймаут сборки |
| `build.assets` | `false` | Зарезервировано: паковать ассеты |
| `build.deps` / `build.mods` | — | **Удалены**, игнорируются с предупреждением. Используйте `load[]` |
| `debug.diagExe` | авто | `DayZDiag_x64.exe`, из папки **клиента** |
| `debug.serverConfig` | рядом с `serverExe` | Боевой `serverDZ.cfg` — источник для отладочной копии |
| `debug.mission` | `build.mission` | Миссия для отладочного прогона; относительный путь — от папки сервера |
| `debug.port` | `2302` | Игровой порт отладочного сервера |
| `debug.playerName` | `dev` | Имя, под которым заходит отладочный клиент |

Приоритет параметров сборки: `build{}` манифеста > настройка `enforce.build` > автоопределение.

## `launch.json`

Две конфигурации, обе типа `enforce`. VS Code предложит их сам: *Run and Debug* → *create a
launch.json file* → **Enforce Build & Check**.

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    // F5: собрать PBO и скомпилировать headless-сервером
    { "type": "enforce", "request": "launch", "name": "Enforce: Build & Check" },

    // Подключиться к DayZDiag, запущенному руками
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

| Атрибут | Для чего | Умолчание | Описание |
| --- | --- | --- | --- |
| `port` | attach | `1000` | Порт, на который приходит игра: 1000 — клиент, 1001 — сервер, 1002 — второй клиент |
| `workDrive` | attach | `P:` | Рабочий диск с распакованными скриптами — от него разворачиваются пути движка в стеке |

У запроса `launch` атрибутов нет: всё берётся из `build{}` манифеста. Для обычной отладки
attach-конфигурация не нужна вовсе — секция **Run and debug** в дереве типов поднимает сессию
сама, см. [Отладка](05-debugging.md).

`Ctrl+Shift+B` запускает задачу `enforce: build PBO`, ей конфигурация не нужна.

## Настройки VS Code

| Настройка | Умолчание | Описание |
| --- | --- | --- |
| `enforce.workDrive` | `P:` | Work Drive с распакованными скриптами движка |
| `enforce.parseDiagnostics` | `warning` | Диагностика разбора: `warning` / `error` / `off` |
| `enforce.semanticDiagnostics` | `warning` | Семантика (члены, типы, арность): `warning` / `error` / `off` |
| `enforce.unknownDefines` | `warning` | Гарды, дефайн которых нигде не объявлен |
| `enforce.deadTypeDiagnostics` | `warning` | Типы, объявленные только под выключенным гардом |
| `enforce.dimDeadTypes` | `true` | Затемнять такие типы в редакторе |
| `enforce.rules` | `{}` | Серьёзность по коду правила: `{"ternary-unsupported": "off"}` |
| `enforce.conditionalMembers` | `hide` | Члены за ложным гардом: `hide` / `dim` |
| `enforce.codeLensReferences` | `true` | «N references» над декларациями |
| `enforce.inlayHints.autoTypes` | `true` | Выведенный тип для `auto` |
| `enforce.inlayHints.parameterNames` | `true` | Имена параметров в вызовах |
| `enforce.inlayHints.parameterModifiers` | `true` | `out` / `inout` у аргументов |
| `enforce.inlayHints.truthiness` | `true` | Что проверяет не-bool условие |
| `enforce.references.includeTextMatches` | `false` | Текстовые совпадения в панели ссылок |
| `enforce.preflight` | `false` | Проверять проект перед запуском движка |
| `enforce.build` | `{ "mission": …, "timeoutSec": 240 }` | Дефолты сборки, перекрываются `build{}` манифеста |
| `enforce.readonlyExternal` | `true` | Движок и зависимости открываются read-only |
| `enforce.autoRefreshSeconds` | `300` | Интервал ревалидации индекса, сек (0 — выкл) |
| `enforce.formatting` | `{}` | `useSpaces`, `tabSize`, `allmanBraces`, `spaceAfterKeyword`, `astylePath`, `astyleRcPath`, `astyleArgs` |
| `enforce.explorerEdit` | `false` | Разрешить файловые операции из дерева типов |

Расширение активируется, когда в воркспейсе есть `enforce.project.json` или когда открыто дерево
типов.
