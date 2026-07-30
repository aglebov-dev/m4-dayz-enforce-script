# Enforce Tools (DayZ)

[English version / Английская версия](README.md)

VS Code, который понимает **DayZ Enforce Script**: ваш мод, его зависимости и скрипты движка
вместе со слоями `modded`. Автокомплит по настоящему типу, навигация, панель ссылок, ошибки
до сборки и проверка настоящим движком по `F5`.

> ⚠️ **Бета.** Возможны ошибки и изменения между релизами — баг-репорты очень приветствуются.

---

> ## ⚠️ Прочтите, прежде чем начать
>
> **Распаковка `.pbo`, разбор бинаризованного конфига и восстановление обфусцированного кода
> могут нарушать ограничения, наложенные автором мода на свою работу.**
>
> Плагин даёт такую техническую возможность — **ответственность за её применение целиком на
> вас**. Добавляя чужой мод в `enforce.project.json`, вы подтверждаете, что вправе так
> поступать. Авторы плагина не выдают на это прав и лицензий и не участвуют в вашем решении.
>
> **Зачем это нужно:** чтобы разрабатывать **своё**, когда чужой мод подключён как
> зависимость — нужен его публичный API, и ничего сверх того. Поэтому обфусцированные моды
> открываются **только как сигнатуры**: заголовки классов, сигнатуры методов, типы полей.
> Тела методов не извлекаются.

---

![Панель references и попап класса](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/references-panel-class.png)

## Проблема

Из коробки VS Code видит EnScript текстом: автокомплит предлагает всех однофамильцев корпуса,
«перейти к определению» уводит в чужой мод, а про опечатку узнаёшь минут через пять — когда
сервер не стартовал, с ошибкой в файле, который вы не трогали.

Плагин строит индекс всей картины: ваш код, зависимости, скрипты движка и каждый слой
`modded` в порядке загрузки. Всё остальное — следствие.

## Что это даёт

### Автокомплит по настоящему типу

Члены того типа, который реально стоит слева от точки, — с цепочками вызовов, дженериками,
`auto` и переменными `foreach`. У каждого пункта виден класс-владелец и сколько модов его уже
трогали. После имени enum'а предлагаются его значения, с числами и в порядке объявления.

![Автокомплит с владельцами](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/completion-owners.png)

### Навигация, которая учитывает слои

Hover и `F12` резолвят по типу, а не по имени. В попапе — кликабельная сигнатура,
класс-владелец, мод и модуль, все слои `modded` и переопределения в наследниках: сразу видно,
кто ещё менял этот метод. `super` ведёт ровно туда, куда пошёл бы движок.

![Hover с дефинициями и override'ами](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/hover-definitions.png)

### Эксплорер типов

Иконка **M4** слева открывает дерево всего, что плагин знает о проекте: сверху ваш мод, в нём
**Properties** с манифестом, **Dependencies** (движок и каждая запись `load[]`), дальше ваши
исходники — папка в папку, как на диске. Дерево ленивое, поэтому корпус в 8000 классов
открывается мгновенно.

Папка с `config.cpp` рисуется модом, сам `config.cpp` раскрывается классами `CfgPatches` /
`CfgMods`, а `#define` — отдельными узлами со значениями (иначе `*_Preload`-аддоны выглядят
пустыми). Read-only помечено замком сразу. **Only Scripts** в тулбаре срезает дерево до кода.

Зависимости подключаются прямо из дерева: **+** на `Dependencies` добавляет папку мода или
`.pbo` в манифест, **−** убирает запись (на диске ничего не трогаем, комментарии в манифесте
переживают правку). Рядом плагин ведёт `enforce.deps.json` — хеш каждого pbo и папку
распаковки, чтобы «тот ли это мод» был вопросом с ответом.

Эксплорер — просмотрщик: в контекстном меню копирование путей и «показать в проводнике».
Создание, переименование и удаление появляются, только если включить `enforce.explorerEdit`.

![Эксплорер рядом с попапом класса](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/explorer-symbols-popup.png)

![Эксплорер с раскрытым модом](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/explorer-tree.png)

Нет `enforce.project.json` — панель предложит его создать. Не нашлись скрипты движка —
появится строка **engine — not set** с кнопкой выбора папки.

### Поиск ссылок в нормальной панели

`Shift+F12` открывает нижнюю панель в стиле Visual Studio: сверху декларации и слои, ниже —
использования, разложенные на вызовы, чтения, записи, конструкторы и колбэки. Фильтры,
сортировка, закрепление вкладки. Десять тысяч строк не тормозят. Текстовые совпадения скрыты
по умолчанию — кнопка в тулбаре возвращает их.

![Панель references для члена класса](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/references-panel-member.png)

![References членов enum](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/enum-members.png)

### Дефайны: видно, что включено и что не соберётся

Половина DayZ-проекта живёт под `#ifdef`. Плагин собирает дефайны отовсюду, откуда они
берутся — `#define` в скриптах, `defines[]` в `config.cpp`, имена модов — и показывает
результат: мёртвые блоки затемнены, hover говорит, включён гард и откуда значение, `F12` ведёт
к объявлению. Гард, который нигде не объявлен, — ошибка: такой код не соберётся никогда.

То же и с типами: если все объявления класса лежат под выключенной ручкой, использования
затемняются и помечаются. Это движковый `Unknown type`, показанный в редакторе, а не в конце
сборки, и с указанием виноватой ручки.

![Карточка необъявленного дефайна](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/define-unknown.png)

![Карточка дефайна, объявление которого закомментировано](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/define-off.png)

### Ошибки ловятся до сборки

На сохранении файл парсится и проверяется линтером и семантикой: необъявленные методы и
переменные (включая out-параметры), неизвестные типы, неверная арность, конструирование
классов с приватным конструктором и ещё полсотни правил. Каждое опирается на факт о движке,
проверенный реальной сборкой, поэтому серьёзность честная: что движок прощает — предупреждение,
что отвергает — ошибка. Любое правило перекрывается настройкой `enforce.rules` по его коду из
Problems.

Имена, которые движок объявляет со стороны C++, живут в секции `environments` манифеста —
поиск, hover и переход к объявлению по ним работают:

```jsonc
"environments": { "DBT_OK": 0, "DBB_NONE": 0, "DMT_INFO": 1 }
```

![Диагностика](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/diagnostics.png)

С `build.preflight` та же проверка прогоняется по всему проекту перед сборкой — чтобы не
тратить круг через сервер на опечатку. Останавливает только на том, что движок не примет, и
в диалоге всегда есть **Build anyway**. Находки помечены `[pre-flight]` и живут в Problems до
следующего прогона: правка файла снимает их по этому файлу, а правка манифеста — нет
(изменение `defines` переворачивает вердикт целиком, нужен новый прогон).

![Предполёт остановил сборку](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/preflight-dialog.png)

### Инлайн-подсказки

Четыре вида подсказок прямо в строке — каждый включается отдельно и молчит там, где вывести
нечего:

```c
auto: eAIBase ai = eAIBase.Cast(from: g_Game.CreateObject(type: "eAI_SurvivorM_Mirek", pos: "1 1 1"));
if (!Class.CastTo(out to: data, from: params))
if (is null: !ai)
```

| Настройка | Что показывает |
| --- | --- |
| `enforce.inlayHints.autoTypes` | выведенный тип для `auto` |
| `enforce.inlayHints.parameterNames` | имена параметров в вызовах |
| `enforce.inlayHints.parameterModifiers` | `out` / `inout` у аргументов |
| `enforce.inlayHints.truthiness` | что на самом деле проверяет не-bool условие |

Подсказки убираются в VS Code как обычно: `editor.inlayHints.enabled` (например, `offUnlessPressed`
— показывать по `Ctrl+Alt`).

### Одна клавиша — собрать и спросить движок

`F5` пакует мод в PBO (без AddonBuilder и прочих тулов) и гоняет компиляцию headless DayZ
Server — вердикт настоящего компилятора секунд за десять. Первопричины отделены от каскада,
каждая ошибка замаплена обратно на ваш файл в Problems, пути в логе кликабельны — включая
ошибки внутри зависимостей. `Ctrl+Shift+B` — только собрать PBO.

Зависимости движку берутся из того же `load[]`, что индексирует анализатор: готовый мод
подставляется как есть, исходники с `config.cpp` пакуются в `<outDir>/deps/@Имя`. Один
источник правды — «в редакторе зелено, а сборка падает» здесь просто неоткуда взяться.

Если движок покажет модальное окно («Addon 'X' requires addon 'Y'» и подобные), плагин
закроет его сам и напишет текст в **Output**: сборка не ждёт, пока кто-то нажмёт OK, и
результат не зависит от того, сидит ли человек у экрана.

![Проверка движком](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/build-check.png)

![Ошибки предполёта в панели Problems](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/preflight-problems.png)

### Мелочи, которые складываются

- **Signature help** с подсветкой активного параметра.
- **Форматтер** файла или выделения (или свой AStyle через `enforce.formatting.astylePath`).
- **CodeLens и семантические цвета** — «N references» / «N layers» над декларациями.
- **Исходники движка и зависимостей — read-only**, случайно не испортите.

![Signature help](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/signature-help.png)

![CodeLens и подсветка](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/codelens-highlighting.png)

## Как начать

### 1. Распакованные скрипты игры

Плагин читает ванильные скрипты как обычные исходники. Обычный путь — **DayZ Tools** →
*Extract Game Data*: он монтирует Work Drive как `P:` и кладёт скрипты в `P:\scripts`
(папки `1_Core` … `5_Mission`). Подойдёт и любая другая локальная копия — укажите путь в
`engine`.

Зависимости в `load[]` — это исходники, распакованный аддон или воркшоп-папка `@Mod` с
`.pbo`: запакованное распаковывается в кэш плагина само (только скрипты и конфиги,
бинаризованные конфиги разворачиваются в текст) и обновляется, когда мод обновился на диске.
Кэш — зеркало pbo, а не рабочая копия: правки в нём стираются. Нужно менять код зависимости —
распакуйте pbo сами и подключите папку как исходники.

⚠️ Про чужие моды — см. предупреждение в начале страницы.

**Обфусцированные моды** открываются в режиме *API-only*: объявления есть, тел нет.
Результат не гарантируется — API может извлечься частично или не извлечься вовсе.

### 2. `enforce.project.json` в корне проекта

Кладётся рядом с `config.cpp` вашего мода. Минимальный рабочий — `{}`, но лучше сразу так:

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

Ключи подсказываются при наборе, опечатка подчёркивается сразу.

![Автокомплит ключей в enforce.project.json](https://raw.githubusercontent.com/aglebov-dev/m4-dayz-enforce-script/main/images/manifest-completion.png)

### 3. Запуск по `F5`

`.vscode/launch.json` в корне воркспейса:

```jsonc
{
  "version": "0.2.0",
  "configurations": [
    { "type": "enforce", "request": "launch", "name": "Enforce: Build & Check" }
  ]
}
```

VS Code предложит её и сам: *Run and Debug* → *create a launch.json file* → **Enforce Build &
Check**.

**Платформа.** Анализатор (индекс, автокомплит, навигация, диагностика, предполёт) работает
везде, где работает VS Code: парсер — wasm. Сборка и проверка движком — только Windows, гейт
запускает `DayZServer_x64.exe`. На macOS/Linux эта часть честно скажет, что сервер не найден,
остальное продолжит работать.

### Нет DayZ Server под рукой?

Включите предполёт — по `F5` плагин прогонит проверку по всему проекту, покажет ошибки и
честно упрётся в отсутствующий сервер:

```jsonc
{ "engine": "P:/scripts", "build": { "preflight": true } }
```

Проверка не такая полная, как вердикт настоящего компилятора, — часть ошибок вскроется позже.
Зато начать можно без игры.

## Команды

Все в палитре под префиксом `Enforce:`.

| Команда | Описание |
| --- | --- |
| `Find All References (Panel)` | Панель ссылок для символа под курсором (`Shift+F12`) |
| `Build Check (Engine Compile)` | PBO + компиляция движком (`F5`) |
| `Rebuild Index` | Полная переиндексация |
| `Show All Modifications (discovery)` | Все `modded`-слои класса |
| `Declare Unknown Defines in enforce.project.json` | Дописать неизвестные дефайны в манифест как `false` |
| `Create Project File` | Создать `enforce.project.json` |
| `Set Engine Scripts Folder…` | Указать папку скриптов движка |
| `Add Dependency…` / `Remove Dependency` | Правка `load[]` из дерева |
| `Search Types` / `Filter Symbol Kinds` / `Only Scripts` | Инструменты эксплорера (они же кнопки в тулбаре) |
| `Show Current File in Type Explorer` / `Show in Type Explorer` | Показать файл или определение в дереве |

Файловые операции (`New Mod…`, `New Script…`, `Rename…`, `Delete` и т. п.) появляются в
эксплорере, только когда включён `enforce.explorerEdit`.

## Настройки

| Настройка | Дефолт | Описание |
| --- | --- | --- |
| `enforce.workDrive` | `P:` | Work Drive с распакованными скриптами движка |
| `enforce.parseDiagnostics` | `warning` | Диагностика парсинга: `warning` / `error` / `off` |
| `enforce.semanticDiagnostics` | `warning` | Семантика (члены, типы, арность): `warning` / `error` / `off` |
| `enforce.unknownDefines` | `warning` | Гарды, дефайн которых нигде не объявлен |
| `enforce.deadTypeDiagnostics` | `warning` | Типы, объявленные только под выключенной ручкой |
| `enforce.dimDeadTypes` | `true` | Затемнять такие типы в редакторе |
| `enforce.rules` | `{}` | Severity по коду правила: `{"ternary-unsupported": "off"}` |
| `enforce.conditionalMembers` | `hide` | Члены за ложным гардом: `hide` / `dim` |
| `enforce.codeLensReferences` | `true` | «N references» над декларациями |
| `enforce.inlayHints.autoTypes` | `true` | Выведенный тип для `auto` |
| `enforce.inlayHints.parameterNames` | `true` | Имена параметров в вызовах |
| `enforce.inlayHints.parameterModifiers` | `true` | `out` / `inout` у аргументов |
| `enforce.inlayHints.truthiness` | `true` | Что проверяет не-bool условие |
| `enforce.references.includeTextMatches` | `false` | Текстовые совпадения в панели ссылок |
| `enforce.preflight` | `false` | Проверять проект перед запуском движка |
| `enforce.build` | см. ниже | Дефолты сборки, перекрываются `build{}` манифеста |
| `enforce.readonlyExternal` | `true` | Движок и зависимости — read-only |
| `enforce.autoRefreshSeconds` | `300` | Ревалидация индекса, сек (0 = выкл) |
| `enforce.formatting` | `{}` | `useSpaces`, `tabSize`, `allmanBraces`, `spaceAfterKeyword`, `astylePath` |
| `enforce.explorerEdit` | `false` | Разрешить файловые операции из эксплорера |

## Справочник `enforce.project.json`

Все ключи необязательные.

```jsonc
{
  // ===== ЧТО ИНДЕКСИРУЕТСЯ =================================================
  // Распакованные скрипты игры (папка с 1_Core … 5_Mission).
  // Нет ключа — берётся <workDrive>/scripts. Работает ${WORKDRIVE}.
  "engine": "P:/scripts",

  // Зависимости в порядке загрузки: исходники, распакованный аддон или
  // запакованный мод (@Mod с addons/*.pbo, либо одиночный .pbo).
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
  // Объявленное в коде и зависимостях подхватывается само.
  // Проверка движком печатает его реальный список и предлагает импорт.
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

    // Где стоит сервер. Убрать оба — найдётся сам по путям Steam.
    // Нужен DayZServer_x64.exe; DayZDiag требует запущенный Steam.
    "serverExe": "D:/Steam/steamapps/common/DayZServer/DayZServer_x64.exe",
    "serverCwd": "D:/Steam/steamapps/common/DayZServer",

    "mission": "mpmissions/dayzOffline.chernarusplus",
    "timeoutSec": 240
  }
}
```

В путях работают и `/`, и `\\`. Префикс PBO не настраивается — он берётся из `config.cpp`
каждого модуля; проект из нескольких модулей пакуется в отдельный PBO на модуль. В PBO идут
только скрипты: движку этого достаточно для вердикта, упаковка ассетов появится позже.

## Спасибо

В примерах выше упомянуты два великолепных проекта:

- [DayZ-Expansion-Scripts](https://github.com/salutesh/DayZ-Expansion-Scripts)
- [DayZ-CommunityFramework](https://github.com/Arkensor/DayZ-CommunityFramework)

Они здесь не случайно: именно на их коде мы разбирали синтаксис и лексику Enforce Script —
большой живой проект показывает, как языком пользуются на самом деле, куда лучше любой
документации. Спасибо их авторам и сообществу.

## Лицензия

MIT
