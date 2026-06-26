---
name: openspec-apply-change
description: Implement tasks from an OpenSpec change. Use when the user wants to start implementing, continue implementation, or work through tasks.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

Implement tasks from an OpenSpec change.

**Input**: Optionally specify a change name. If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **Select the change**

   If a name is provided, use it. Otherwise:
   - Infer from conversation context if the user mentioned a change
   - Auto-select if only one active change exists
   - If ambiguous, run `openspec list --json` to get available changes and use the **AskUserQuestion tool** to let the user select

   Always announce: "Using change: <name>" and how to override (e.g., `/opsx:apply <other>`).

1b. **Parse флаги команды** (см. `.cursor/commands/opsx-apply.md`):

   | Флаг | Эффект в скилле |
   |------|-----------------|
   | *(нет флагов)* | default = **step-by-slice** от первого непринятого среза до конца |
   | `--slice S<N>` | выполнить **только** срез S<N>, без перехода к следующему (целевой фикс / повтор после rejection) |
   | `--since-slice S<N>` | начать со среза S<N>, пропуская предыдущие (в т.ч. не помеченные `[x]` — legacy / частично принятые) |
   | `--step-by-step` | пауза после **каждой задачи** вместо приёмки целого среза (отладка) |
   | `--batch` | все оставшиеся срезы без остановок на slice-gate (только явный флаг или legacy-ЗНИ без срезов) |

   Конфликт флагов (`--slice` + `--batch` и т.п.) — одно уточнение пользователю. Выбранный режим зафиксировать в TodoWrite-плане сессии.

2. **Check status to understand the schema and verify Metadata**
   ```bash
   openspec status --change "<name>" --json
   ```
   Parse the JSON to understand:
   - `schemaName`: The workflow being used (e.g., "spec-driven")
   - Which artifact contains the tasks (typically "tasks" for spec-driven, check status for others)

   **Metadata Prep (MANDATORY):** Before writing any code, check `proposal.md` for comment marker placeholders:
   - Read or Grep `openspec/changes/<name>/proposal.md` for `<developer>`, `<ФИО>`, «Уточнить до `/opsx:apply`» or «Уточнить».
   - If `developer` placeholder and `defaultDeveloper` empty — STOP, один вопрос в чат: только ФИО (Фамилия И.О.).
   - If `comment_suffix` placeholder or empty without `marker_style: minimal` — STOP, один вопрос: только **описание** для маркера (`comment_suffix`; можно пусто → `marker_style: minimal`).
   - Не монолит «ФИО + domain_label» в одном сообщении.
   - Replace placeholders in `proposal.md`; mark F1 follow-up `[x]` if present.

3. **Get apply instructions**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   This returns:
   - Context file paths (varies by schema - could be proposal/specs/design/tasks or spec/tests/implementation/docs)
   - Progress (total, complete, remaining)
   - Task list with status
   - Dynamic instruction based on current state

   **Handle states:**
   - If `state: "blocked"` (missing artifacts): show message, suggest `/opsx:new <name>` (resume)
   - If `state: "all_done"`: congratulate, suggest archive
   - Otherwise: proceed to implementation

4. **Read tasks + minimal context (lazy loading) + verify pre-flight**

   **Mandatory read:** tasks.md (navigation, progress, dependencies) + `openspec/project.md` (project-level constraints: allowed directories, editing rules).
   
   **Lazy reads (по необходимости):**
   - proposal.md — only on first run (for overview), skip on resume
   - design.md — only if current task references an architectural decision
   - specs/ — only when verifying acceptance criteria
   
   Writer subagents receive **paths** to design.md and specs/ in their prompt and read needed sections independently. This keeps orchestrator context lean.

   **Pre-flight: verify check & Metadata**

   Glob в change dir: `reports/verification-*.md`. Взять последний по дате в имени, прочитать YAML `verify_mode` (`pre-apply` / `post-apply`) — фаза в снапшоте, не в имени файла.

   - **If found** → show summary line from report (first CRITICAL/WARNING counts). Continue.
   - **If NOT found** → soft warning:
     ```
     "Pre-apply verify не проводился. Рекомендуется `/opsx:verify <name>`
     для проверки качества артефактов (формат tasks, gates, конкретность задач).
     [Запустить verify / Продолжить без]"
     ```
     - Option 1 → STOP apply, suggest running `/opsx:verify <name>` first
     - Option 2 → continue implementation

   Verify check is advisory — does not block apply.

   **Metadata (comment markers) check:**
   Прочитать `proposal.md` и `openspec/project.md` (секция «Разработчик по умолчанию»: `defaultDeveloper`, `cfMarkerPrefix`; § «Канон маркеров (domain_label)» — запреты).

   **Извлечение Metadata (dual-parser):** из секции `## Metadata (comment markers)`:
   - yaml-like: строки `developer:`, `comment_suffix:`, `marker_style:`
   - list: `- **developer:**`, `- **comment_suffix:**`, `- **marker_style:**`
   - `marker_style` default = `canonical` если не указан

   **domain_label validation** (на `comment_suffix` после trim):
   - Если match запретам из project.md § Канон domain_label → **STOP**, один вопрос: «перепишите domain_label для маркера» (как при плейсхолдере developer).
   - Пустой `comment_suffix` без `marker_style: minimal` → STOP с предложением заполнить или пометить mechanical.

   - Если `developer` пустой/плейсхолдер — взять `defaultDeveloper` из project.md; если и там пусто — один вопрос в чат: **только ФИО** (не смешивать с описанием).

   **marker_scope** (Grep `tasks.md` и при необходимости `design.md` на пути `src/.../*.bsl`):
   - только `src/ЭДО и ЭА/cf/` → **cf-ea**
   - только cfe (`src/ЭДО ПАО/cfe/`, `src/ДО3 Демо/cfe/`) → **cfe**
   - оба → **mixed** (writer получает оба шаблона + таблицу «путь → шаблон»)

   Сформировать (date = текущая `dd.MM.yyyy`):
   - **cfe:** `open_marker` = `// +++ {developer} {date}` или `// +++ {developer} {date} {comment_suffix}`; `close_marker` = `// --- {developer}`
   - **cf-ea:** `open_marker` = `// {cfMarkerPrefix} {comment_suffix} +++` (или однострочный `// {cfMarkerPrefix} {comment_suffix}` для целой процедуры); `close_marker` = `// {cfMarkerPrefix без двоеточия} ---`
   - **mixed:** передать оба набора + явная инструкция scope по пути файла из задачи

   Если секции Metadata нет — сначала ФИО (если нет `defaultDeveloper`), затем отдельный вопрос про **описание** (`comment_suffix`; можно пусто → `marker_style: minimal`); дописать Metadata в proposal.md. Не монолит «ФИО + domain_label».
   Передать маркеры в промпт `onec-code-writer` (см. §3a COMMENT MARKERS).

5. **Resume with pending verdict & Show current progress**

   **Resume with pending verdict (FIRST ACTION, before any work):**

   1. Grep последнюю запись в `debug.md` `## Slice Gate Decisions` для каждого среза.
   2. Найти самую раннюю по порядку запись с `Решение: awaiting-acceptance`, где:
      - все non-acceptance задачи среза S<N> = [x],
      - `S<N>.accept` (или legacy `S<N>.T<M>`) = `[ ]`.
   3. Если найдено — НИЧЕГО не реализовывать, сразу AskQuestion:

      Slice Gate S<N> — <имя> — ожидает вердикта

      Ты вернулся с проверки. Что по результату чеклиста среза S<N>?

      [1] Принят — отметить `S<N>.accept` (или legacy `S<N>.T<M>`) [x], перейти к S<N+1>
      [2] Не принят — опишу что не работает, rework внутри S<N>
      [3] Дефект в предыдущем срезе S<K> — укажу S<K> и суть дефекта

   4. По ответу:
      - [1] → определить целевой чекбокс приёмки: предпочтительно `S<N>.accept`; если в срезе только legacy `S<N>.T<M>` — отметить их все (срез принят целиком); mark [x], append debug.md "решение: принят", сгенерировать reports/slice-acceptance-S<N>-YYYY-MM-DD.md, перейти к задачам S<N+1>.
      - [2] → запросить описание проблемы; append debug.md "решение: не принят" + секция ## Debug — S<N>; создать fix-задачи перед `S<N>.accept` (или legacy `S<N>.T<M>`); начать их выполнение.
      - [3] → запросить **S<K>** и описание дефекта; **Grep** в `tasks.md` приёмочную строку среза (`S<K>.accept` или legacy `S<K>.T<M>`):
        - Если приёмочная задача среза `S<K>` = **`[ ]`** (срез S<K> **не** принят) → **inside-slice rework** по `.cursor/rules/vertical-slices.mdc` (**ИНВАРИАНТ: Defect placement**): добавить fix-задачи **внутрь** `S<K>` **перед** приёмочной задачей; **не** создавать `# Срез S<N+1>` без cross-slice; append в `debug.md` `Решение: inside-slice rework` + RCA-кратко; начать выполнение fix-задач.
        - Если приёмочная задача среза `S<K>` = **`[x]`** (срез S<K> принят, frozen) → создать **fix-срез** `# Срез S<N+1>: …` с отдельным `S<N+1>.accept` по `vertical-slices.mdc`; **снять** `[x]` с приёмки `S<K>` только если инвариант/постановка явно требует повторной приёмки S<K> (зафиксировать в `debug.md`); иначе — только приёмка нового среза.

   5. Если awaiting-acceptance не обнаружено — перейти к обычному "Show current progress" ниже.

   **Show current progress:**

   Display:
   - Schema being used
   - Progress: "N/M tasks complete"
   - **Slice progress (if slices detected):** For each slice, show task count and acceptance status:
     ```
     Срез S1: Флажок в шаблоне — 3/3 done, S1.accept [x] ПРИНЯТ
     Срез S2: Копирование флага в БП — 2/4 done, S2.accept [ ]  ← текущий
     Срез S3: Индикатор на форме параметров — 0/3 pending
     ```
   - Remaining tasks overview
   - Dynamic instruction from CLI
   - **Slice Gate Decisions (if debug.md contains them):** Show previous slice gate decisions from `debug.md` section `## Slice Gate Decisions` — helps orient after session breaks

5.5 **Analyze parallelization (slices + file dependencies)**

   Before starting implementation:

   **Slice-aware ordering:**
   1. Grep `tasks.md` for `^# Срез S\d+` — if found, ЗНИ is in **slice mode** (default).
   2. Read slice metadata blocks (Сценарий, Приёмка, Связь со spec, Зависимости) for each slice.
   3. Glob `reports/quality-control-*.md` in change dir — if found, use slice dependency graph and coverage matrix from it.
   4. Order slices by dependency graph: S<N> can start only when all slices in `**Зависимости:**` have `S<K>.accept` (or legacy `S<K>.T<M>`) = `[x]`.
   5. Within a slice, group tasks by layer/file:
      - Tasks touching different files = independent = can run in parallel (up to 3 concurrent via Task tool).
      - Tasks touching the same file = sequential.
      - Layer ordering inside slice is guidance only (метаданные → UI: Конфигуратор или BSL модуля формы → код → приёмка); enforce via task-level dependency graph from tasks.md.
   6. Display plan:
     ```
     Срезы для выполнения:
     - S1: Флажок в шаблоне (3 задачи + S1.T1)
     - --- Slice Gate S1 ---
     - S2: Копирование флага в БП (4 задачи + S2.T1), зависит от S1
     - --- Slice Gate S2 ---
     - S3: ...
     Реализация будет останавливаться на каждом slice gate для приёмки.
     ```
   Ref: `.cursor/rules/vertical-slices.mdc`.

   **Legacy mode (no slices in tasks.md):**
   - Warn: "ЗНИ без срезов. Перестройка: `/opsx:extend <name>` (architect slice decomposition) или ручная правка по vertical-slices.mdc."
   - AskQuestion: `[Продолжить без срезов] / [Мигрировать на срезы (verify)] / [Стоп]`.
   - `Продолжить` → fall back to file-only parallelization (tasks touching different files = independent; same file = sequential). Нет slice-gate пауз.
   - `Мигрировать` → STOP apply, предложить `/opsx:extend <name>` или ручную правку tasks.md.
   - `Стоп` → завершить apply.

   **5.6 Determine execution mode**

   **Step-by-slice mode** (DEFAULT for slice mode):
   - Выполнять задачи одного среза подряд (параллелизация по файлам внутри среза допустима).
   - После последней non-acceptance задачи среза — ОБЯЗАТЕЛЬНАЯ ПАУЗА на slice-gate (карточка приёмки, см. шаг 6).
   - Не начинать следующий срез до принятия (или явного пропуска) текущего.

   **Пошаговый режим** (внутренний код `step-by-step`, в чат не цитируется) — пауза после **каждой** задачи только при срабатывании триггеров ниже.

   **Приоритет правил** (сверху вниз):

   1. **Slice-gate** — пауза всегда (шаблон B приёмки среза).
   2. **Explicit-request** — пользователь попросил «пошагово» / «по одной».
   3. **Pure mechanical slice** — **нет** auto `slice-size-threshold`; между задачами **тихий успех** (одна строка).
   4. **Mixed slice** (код + UX в одном срезе) — auto step-by-step **не** включать; пауза только на slice-gate.
   5. **`slice-size-threshold`** (≥5 задач в срезе) — только если есть **хотя бы одна** задача с явным ручным критерием (`убедиться`, `проверить`, `**Приёмка:** ручной тест`) **и** срез **не** mechanical/mixed.

   **Триггеры step-by-step:**
   - `explicit-request`
   - `debug-session` — `debug.md` изменён сегодня
   - `fix-slice` — срез Fix/Фикс или fix-задачи в debug
   - `slice-size-threshold` — см. приоритет 5 (не для mechanical/mixed)

   **Детекция mechanical slice** (не grep как единственный SSOT):
   - **Primary:** metadata `**Режим apply:** mechanical`
   - **Fallback:** все рабочие задачи без `**Приёмка:** ручной тест` и без Primary, требующего ИБ; эвристика «миграц», «замен», «rename» только если нет UX-accept
   - **Mixed побеждает mechanical** — одна задача «верифицировать по коду» в UX-срезе не делает срез mechanical

   **Три режима вывода между задачами:**

   | Режим | Когда | Чат |
   |-------|-------|-----|
   | **Тихий успех** (default) | Mechanical; mixed; обычный slice без step-by-step | Одна строка: «Задача `S<N>.<M>` («…») реализована.» |
   | **Explicit step-by-step** | step-by-step + явный критерий в tasks | Шаблон A |
   | **Slice-gate** | Все рабочие `[x]`, accept `[ ]` | Шаблон B |

   **Запрет meta-статуса pipeline в чате:** не сообщать «автопроверки пройдены», «линтер чист», «reviewer PASS», «diff не обязателен» — non-events (§3a `chat-output-budget.mdc`).

   **Batch mode** — только legacy без срезов + явный запрос пользователя.

   **Announce mode** (один раз в начале сессии, не в каждой карточке):
   - «Пауза на приёмку каждого среза» — default slice mode
   - «Пошагово — по вашей просьбе / отладка / фиксы» — при step-by-step
   - «Между задачами — без пауз» — mechanical / тихий успех

   **5.5b Слой чата при apply (анти-шум)** — относится ко **всем** промежуточным сообщениям в чате (старт сессии, между задачами, перед делегированием writer). Финальный **T-HANDOFF** (шаг 7) не отменяется.

   | В чат пользователю | В артефактах / контексте модели |
   |---|---|
   | Блокеры, предупреждения, вопросы с выбором | Полные пути, grep, вывод CLI |
   | Короткий UX-итог: что сделано / что будет дальше (1–3 предложения) | Повторный дословный pre-apply, дубли одной строки PASS |
   | Прогресс: «задача M/N», срез в формате §10 `opsx-output-style` (**Срез S\<N\>: «название»**) | Перечисление сырого `S1.1–S1.T5` в одной строке без названия среза |
   | Режим **человеческим языком**: «пауза после каждой задачи — в срезе много шагов» / «вы просили пошагово» / «активная отладка по debug» | Обязательное цитирование внутренних id триггеров (`slice-size-threshold`, `step-by-step`, `awaiting-acceptance`); при необходимости id один раз в `debug.md` или в конце сообщения мелким блоком «Техническое» |
   | «Маркеры разработчика в proposal заполнены» | Полный текст строк `// +++` / `// ---` в чате |
   | Одна строка со ссылкой на отчёт verify при старте (если нужна): `reports/verification-*.md` | Дважды одно и то же PASS + длинный путь |
   | Пошаговая пауза (шаблон A) — только явный критерий приёмки в tasks | Списки lint/reviewer/spot-check в чат |
   | Pre-apply verify NO-GO — одна строка **только** при первом сообщении apply, если блокирует | NO-GO / OQ в каждой карточке паузы |
   | Роли исполнителей по-русски: «агент», «агент + ревью», «пользователь», «оркестратор» | Имена агентов (`onec-code-writer`, `onec-code-reviewer`, `onec-code-explorer`, `onec-code-architect`, `openspec-quality-controller`) — только во внутреннем контексте делегирования; в чат не цитируются |
   | Обозначение проверки на человеческом языке: «архитектурное ревью закрыто», «проверка прошла резервным агентом» | Имена гейтов (`Architect Gate`, `Slice Gate`, `Implementation Impact Gate`) и техкоды режимов (`Step-by-step`, `checkpoint`, `Tier`, `Standard/Lite/Full`) — только в `debug.md`, отчётах `reports/` и в скрытом контексте модели |

   **Не считать обязательным содержимым ответа пользователю:** нарратив «читаю скилл / по протоколу команды»; строки вида «Explored N files» / «Ran OpenSpec…» как **текст итогового сообщения** (это следы инструментов — остаются в UI, но не обязаны дублироваться в сводке для человека).

   Launch up to 3 independent tasks in parallel via **Task** tool when applicable внутри одного среза. Делегирование субагентам — через инструмент **Task** (см. `.cursor/rules/tool-name-guard.mdc`).

6. **Implement tasks (loop until done or blocked)**

   **Conditional Task Detection (перед началом loop):**
   Просканировать tasks.md на паттерны условного ветвления (ссылки между задачами вида `Если в п.`, `При отрицательн`, `Альтернатив`, `workaround`, `Иначе →`, `Иначе —`).

   Если обнаружены:
   - Идентифицировать **задачу-верификацию** (определяет ветку) и **зависимые задачи-ветки**.
   - После выполнения задачи-верификации — **ОБЯЗАТЕЛЬНАЯ ПАУЗА:**
     ```
     "Задача N (верификация) выполнена. Результат: [краткий итог].
     Следующие задачи зависят от этого результата:
     - Задача X: [при положительном результате]
     - Задача Y: [при отрицательном результате]
     Какую ветку выполнять?"
     ```
   - НЕ выбирать ветку автоматически. Решает пользователь.

   **Task Dispatch (для каждой задачи перед реализацией):**

   Классифицировать задачу по типу и назначить исполнителя:

   | Тип задачи | Маркеры в тексте задачи | Исполнитель |
   |---|---|---|
   | BSL-код (новая логика, правка процедур) | «реализовать», «добавить», «доработать» + путь к .bsl | **onec-code-writer** + **onec-code-reviewer** после |
   | Модуль формы (BSL) | путь `Forms/.../Ext/Form/Module.bsl`, «программно создать элемент», «Элементы.Добавить», «видимость элементов» без правки `Form.xml` | **onec-code-writer** + **onec-code-reviewer** после |
   | Форма / Form.xml / Конфигуратор | «Form.xml», «реквизиты формы», «добавить форму», «колонки в Конфигураторе», создание/изменение XML формы | **СТОП.** Инструкция ручного конфигурирования (`1c-agent-delegation.mdc` § XML WRITE GUARD). WAIT — не продолжать до выгрузки |
   | Верификация метаданных | «проверить соответствие», «проверить наличие» | Оркестратор (Glob/Grep/Read — только проверка, не реализация) |
   | Приёмочная задача (`S<N>.accept` или legacy `S<N>.T<M>`) | «ручной тест», «убедиться», `S<N>.accept`, `S<N>.T<M>` | **Режимы по срезам / пошаговый** (внутренние коды `step-by-slice` / `step-by-step`): trigger Slice Gate (см. шаг 6). **Legacy batch:** пропустить с предупреждением; включить в Session Summary секцию «Отложенные ручные тесты» |
   | Создание метаданных | «создать регистр», «создать справочник», «создать форму» | **СТОП** — блокер пользователю (`1c-no-metadata-creation.mdc`) |

   **HALT:** Оркестратор НЕ реализует задачи типов BSL-код и модули формы самостоятельно. Оркестратор готовит промпт и делегирует. Прямое использование Write/StrReplace для .bsl и **любая правка Form.xml** (включая скрипты/JSON-конвейеры) — запрещены. Для `Form.xml`/Конфигуратора: СТОП — инструкция ручного конфигурирования и WAIT до выгрузки/приёмки пользователя. Для программного создания элементов в `Form/Module.bsl`: обычный BSL pipeline — полный эталонный порядок в `1c-agent-delegation.mdc` § WRITER PIPELINE (writer → ReadLints → NAMING PROVENANCE CHECK → API/METADATA CHECK → EXTENSION VERIFICATION → reviewer). Ref: `1c-agent-delegation.mdc` (§ XML WRITE GUARD), `1c-utility-agents.mdc`.

   **Code-Truth Journal (mandatory after writer/reviewer success):**
   - После каждой BSL/form-module задачи извлечь из ответа `onec-code-writer` блок `created_or_modified_symbols`.
   - Spot-check: Grep/Read каждый `evidence` или `name` в указанном файле. Если символ не найден — НЕ отмечать задачу `[x]`; pause с рекомендацией `/opsx:extend <change-name> --code-sync`.
   - В `debug.md` записывать только факты из кода, не план:
     ```markdown
     ### Code-Truth — <task-id> — YYYY-MM-DD
     - task: <S<N>.<M> / legacy id>
     - symbols:
       - <name> @ <file>:<lines>, annotation=<annotation>, action=<created|modified|removed>
     - verification: grep/read OK | mismatch <details>
     - source: writer.created_or_modified_symbols
     ```
   - Запрещено писать в `debug.md` имена процедур/хелперов, которых нет в `created_or_modified_symbols` или spot-check.

   **Карта правок (`reports/code-map.md`, mandatory at Slice Gate):**
   - **Автор:** оркестратор (механическое сведение journal + `tasks.md`; без нового агента).
   - **Источник:** записи `### Code-Truth — <task-id>` текущего среза S<N> в `debug.md` + формулировка задачи из `tasks.md` (поле «суть» — одно предложение на русском; **не** из ответа writer).
   - **Файл:** `openspec/changes/<name>/reports/code-map.md` — **append** секция `# Срез S<N> — <имя среза> (YYYY-MM-DD)`; накопительный артефакт на всё ЗНИ.
   - **Формат строки в файле** (эталон):
     ```markdown
     - **S<N>.<M>** · <модуль/объект> · <процедура/функция> (<created|modified>) — <суть из tasks.md>. [`src/.../Module.bsl`](src/.../Module.bsl):<start>-<end>
     ```
   - **Чат (acceptance-handoff):** блок `### Карта правок (перед тестом)` — до **5** нумерованных пунктов: **одно предложение прозой**, затем один code-citation [`start:end:path`](...). Строка «Полная карта: `openspec/changes/<name>/reports/code-map.md`». **Не** дублировать §1 «Что реализовано» телеграммой путей.
   - **HALT в блоке карты (чат):** подстроки из `chat-output-budget.mdc` §7; `created_or_modified_symbols`, `spot-check`, имена агентов, «источник: writer», YAML, internal Review Boundaries.
   - **Self-check apply (карта):** каждый пункт карты в чате понятен без открытия `debug.md`; в блоке карты нет HALT-подстрок.

   **Investigation Loop при apply.** Если reviewer в отчёте включил секцию `## Investigation Request`:
   1. Вызвать explorer (contract resolution deep) по таблице из Investigation Request. Шаблон: «Explorer — contract resolution (deep)» из `1c-agent-patterns/explorer.md`.
   2. Сохранить вывод в `reports/resolved-contract-<slug>-YYYY-MM-DD.md` (артифакт ЗНИ).
   3. Повторно вызвать reviewer с Resolved Contracts в промпте.
   4. При последующем устранении замечаний — передать Resolved Contracts в промпт writer.
   Протокол: см. `1c-agent-delegation.mdc`, секция CONTRACT RESOLUTION; шаблоны: `1c-agent-patterns/explorer.md` (Explorer — contract resolution), `1c-agent-patterns/writer.md` (Writer — review fix), `1c-agent-patterns/reviewer.md` (Reviewer — ревью кода).

   **Task loop:**

   For each pending task:
   - **Slice Gate (после последней non-acceptance задачи среза):**
     Детектор: все рабочие задачи `S<N>.<M>` = `[x]`, приёмочная задача среза (`S<N>.accept` или legacy `S<N>.T<M>`) = `[ ]`.

     **Внутренний verify (обязательно до приёмочного handoff):**
     1. Оркестратор выполняет проверки по скиллу `openspec-verify-change/SKILL.md` для **текущего среза** в `pre-apply` контексте (узкий охват выводится из текста запроса и факта свежей приёмки/`debug.md`), **без** длинного вывода в чат.
     2. Сохранить полный отчёт в `openspec/changes/<name>/reports/verification-YYYY-MM-DD.md` (если день уже использовался — суффикс `-2`, `-3`).
     3. **Чат:** если вердикт внутреннего прогона **NO-GO** — вывести **кратко** по `.cursor/rules/chat-output-budget.mdc` + ссылка на файл отчёта; **не** переходить к шагу приёмочного handoff, пока пользователь не обработает блокеры (как при обычном `/opsx:verify`).
     4. Если вердикт **GO** — **не упоминать** verify в сообщении пользователю.

     Действие — сгенерировать T-HANDOFF вариант `acceptance` (формат — `.cursor/docs/opsx-output-style.md` §5.2; НЕ вызывать AskQuestion), **только после** успешного внутреннего verify по п.1–4:

     ```
     ## Срез S<N> — передача на приёмку: <change-name>

     **Прогресс:** N/N рабочих задач [x]; приёмочная задача среза (`S<N>.accept` или legacy `S<N>.T<M>`) — `[ ]` (ручная приёмка).

     ### 1. Что реализовано
     <2–3 предложения прозой: что именно сделано в коде, какие файлы затронуты. Для нетривиальных срезов — краткое описание логики. Без однострочных списков-телеграмм.>
     - [только при проблеме] Замечание: <spot-check / линтер / ревью — одна строка>

     ### Карта правок (перед тестом)
     <до 5 пунктов: проза + [`start:end:path`](...); полная карта — в `reports/code-map.md`>

     ### 2. Что проверить СЕЙЧАС
     **Primary acceptance** из metadata среза (императив, 1 нумерованный список). Optional-сценарии — одной строкой «остальные — см. tasks.md». Legacy `S<N>.T<M>` — Primary или первый UX-сценарий.

     ### 3. Как вернуться
     `/opsx:apply <change-name>` — новая сессия начнётся с запроса вердикта (принят / не принят / дефект в предыдущем срезе). Если нужно изменить scope/design/tasks — `/opsx:extend <change-name>`; затем снова `/opsx:apply <change-name>`. Пока вы проверяете в 1С — оркестратор ничего не делает.

     ### 4. Short-cut
     Если уже проверено и принято — напишите обычной фразой («принято», «срез S<N> принят»), отмечу без полного handoff.
     ```

     Параллельно:
     1. **Карта правок:** собрать символы среза из journal + `tasks.md`; **Write/append** `reports/code-map.md` (секция `# Срез S<N> — …`).
     2. Append в `debug.md` `## Slice Gate Decisions`:
        ```markdown
        ### Slice S<N> — <имя> (YYYY-MM-DD)
        Срез: S<N> — <имя>
        Решение: awaiting-acceptance
        Обоснование: все рабочие задачи реализованы; приёмочная задача передана на ручной прогон Primary.
        Изменения tasks: нет (S<N>.accept остаётся [ ])
        Связанный отчёт: reports/handoff-acceptance-S<N>-YYYY-MM-DD.md
        ```
     3. **Write** полный handoff в `reports/handoff-acceptance-S<N>-YYYY-MM-DD.md` (T-HANDOFF §5.2; все слоты варианта `acceptance`, включая «Карта правок» между «Что реализовано» и «Что проверить СЕЙЧАС»). **Обязательно** — не только тонкий чат.
     4. T-HANDOFF вариант `acceptance` в **чат** по шаблону шага 7 (заголовок — `## Срез S<N> — передача на приёмку: <change-name>`). Секция «Что проверить СЕЙЧАС» — чеклист из `S<N>.accept` (или legacy `S<N>.T<M>`). Блок «Карта правок» — компактно (до 5 пунктов).
     5. Завершить сессию.
   - **Manual acceptance shortcut:**
     Если пользователь в любой момент сессии обычной фразой подтверждает приёмку («принято», «срез S<N> принят», «проверил, работает») — для среза, у которого все рабочие задачи [x] и приёмка [ ]; при неоднозначности (несколько срезов в ожидании) — одно уточнение, какой срез:
     1. Не генерировать Acceptance Handoff Card.
     2. Отметить **родительский** чекбокс приёмки среза: `S<N>.accept` = `[x]`. Для legacy-формата (`S<N>.T<M>` без `S<N>.accept`) — отметить все `S<N>.T<M>` среза = `[x]`.
     3. Append в debug.md "решение: принят (manual shortcut)".
     4. Определить, **последний ли** срез S<N> в `tasks.md`: нет следующего заголовка вида `# Срез S<K>:` с K > N.
     5. **Если срез не последний** — продолжить к следующему срезу / задачам (как раньше).
     6. **Если срез последний** — в **чат** (тонко, `chat-output-budget.mdc`): подтвердить приёмку среза с названием (§10 стайл-гайда); спросить: «Архивировать ЗНИ? Напишите `архив` для подтверждения или `стоп` чтобы остановиться.» Завершить ход **без** автозапуска archive до ответа.
     7. **Авто-archive:** если на шаге 6 пользователь ответил **`архив`** (или явная эквивалентная фраза) — в **этой же сессии** выполнить шаги скилла `openspec-archive-change/SKILL.md` для `<change-name>` целиком (как при `/opsx:archive <name>`), с **тонким** итогом в чат (T-CONFIRM §5.5); при блокере архива — карточка по правилам archive, не молчать. Ответ **`стоп`** — завершить сессию apply без archive.

   - **Ранний выход ("стоп" в любой момент):**
     Пользователь явно: "стоп" / "stop" / "прекрати" / "прерви" / "пока хватит".
     1. Завершить текущий task/subagent.
     2. Append debug.md "решение: стоп" (если прерван на Slice Gate) или просто запись в debug.md секции "Interrupted sessions" (если в середине).
     3. T-HANDOFF вариант `pause` (по шаблону шага 7).
     4. Завершить apply.

   - **Slice Gate log (mandatory for all options):** After the user's decision at a slice gate, append a record to `debug.md` (create section `## Slice Gate Decisions` if absent):
     ```
     ### Slice S<N> — <имя> (YYYY-MM-DD)
     Срез: S<N> — <slice name>
     Решение: awaiting-acceptance / принят / принят (manual shortcut) / не принят / fix предыдущего S<K> / стоп
     Обоснование: <user's rationale or "без замечаний">
     Изменения tasks: <"нет" / "N задач добавлено" / "создан S<K>.fix">
     Связанный отчёт: reports/slice-acceptance-S<N>-YYYY-MM-DD.md (если принят)
     ```
   - Show which task is being worked on
   - **Classify** (Task Dispatch table above) — announce type and executor
   - **Delegate** to the designated executor (agent or skill)
   - Orchestrator role: prepare prompt with context, delegate, spot-check result
   - **Spot-check (post-verification):** After the agent reports completion, verify the change: Grep for a pattern that confirms the fix (e.g. after "replace ТекущаяДата with ТекущаяДатаСеанса" → Grep for `ТекущаяДата()` in that file must return 0 matches). For batch tasks (5+ files), spot-check at least 3 files (first, middle, last in the list). If the result does not match expectations → STOP, report to user, do NOT mark task complete. **LINT GATE** — см. [`.cursor/rules/1c-agent-delegation.mdc`](../../rules/1c-agent-delegation.mdc) § LINT GATE + [`.cursor/rules/1c-writer-pipeline.mdc`](../../rules/1c-writer-pipeline.mdc). **Чат (non-events, §3a `chat-output-budget.mdc`):** при успешном writer + spot-check + reviewer PASS без MUST_FIX — **не** выводить строки «Авто-проверка: OK», «Линтер чист», «reviewer PASS», «Spot-check: OK»; между задачами (без пошагового режима) достаточно одной строки «Задача `S<N>.<M>` («…») реализована.».
   - Mark task complete in the tasks file: `- [ ]` → `- [x]`
   - **Вывод после задачи** — по режиму из §5.6 (тихий успех / шаблон A / slice-gate B):

     **Тихий успех (default):** одна строка «Задача `S<N>.<M>` («…») реализована.» — без паузы, без кнопок.

     **Шаблон A — explicit step-by-step** (только при step-by-step **и** явном критерии в tasks: `убедиться`, `проверить`, `**Приёмка:** ручной тест`):

     ```
     Задача `S<N>.<M>` («<описание>») выполнена — <одно предложение эффекта, без путей>.
     1. <шаги из критерия приёмки tasks>
     Ответ: Проблема | Пропустить
     Дальше: `S<N>.<M+1>` («…»). `/opsx:apply <name>`
     ```

     Без «Подтвердить», если нет шагов для ИБ. Diff — только по запросу пользователя.

     **Шаблон B — приёмка среза** (`S<N>.accept` / slice-gate):

     ```
     ## Срез S<N>: «<название>» — проверка на ИБ

     **Primary acceptance:** <из metadata>
     1. <шаги только primary>
     Опционально: …
     Ответ: Принято | Не принято | Отложить
     ```

     **Правило «первое упоминание ID — с описанием».** §10 / §10.1 `opsx-output-style.md`.
   - **If this was a verification/decision task** (identified in Conditional Task Detection) → trigger ОБЯЗАТЕЛЬНАЯ ПАУЗА above before proceeding
   - Continue to next task

   **Pause if:**
   - **Slice Gate reached** — шаблон B «приёмка среза» (выше в этом шаге)
   - **Explicit step-by-step** — шаблон A только при явном критерии в tasks
   - Task is unclear → ask for clarification
   - Implementation reveals a design/scope issue → stop implementation and suggest `/opsx:extend <change-name>` (or `/opsx:extend <change-name> --from-review <report-path>` if the issue came from review), затем снова `/opsx:apply <change-name>` после обновления постановки
   - Error or blocker encountered → report and wait for guidance
   - User interrupts
   - **Verification/decision task completed** → пауза условной задачи (см. выше «Conditional Task Detection»)

7. **On completion or pause — T-HANDOFF (единый шаблон)**

   Формат — **T-HANDOFF** из `.cursor/docs/opsx-output-style.md` §5.2. Заголовок выбирается по варианту:
   - `acceptance` → `## Срез S<N> — передача на приёмку: <change-name>` (handoff в конце среза, default)
   - `pause` → `## Сессия приостановлена: <change-name>` (issue / пользовательский стоп)
   - `final` → `## Реализация завершена: <change-name>` (все срезы приняты)

   **Имена секций одинаковы во всех трёх вариантах** (см. раздел «Output — T-HANDOFF» ниже):

   - `### 1. Что реализовано` — за каждую задачу этого сеанса: `[x] N.M — описание`, файл `path` со строками, «что изменено» одним предложением; строка про автопроверку — **только** при расхождении или замечании ревью (без «OK» в успешном пути — §3a `chat-output-budget.mdc`). Для нетривиальных срезов в чате допускается **2–3 предложения прозой** (что именно сделано в коде, какие файлы затронуты) вместо списка-телеграммы.
   - `### Карта правок (перед тестом)` — **только вариант `acceptance`**: до 5 пунктов (проза + citation); в файле handoff — полный список секции среза из `code-map.md`. Язык — §5.2 «Язык карты».
   - `### 2. Что проверить СЕЙЧАС` — **три режима** (согласовано с §5.6):
     - **Slice-gate / acceptance handoff:** только **Primary acceptance** из metadata + optional одной строкой.
     - **Тихий успех между задачами:** «Ручная проверка не требуется» или пропустить секцию.
     - **Explicit step-by-step:** только шаги из явного критерия задачи в tasks — не diff, не pipeline.
     **Запрещено:** перечислять критерии **каждой** закрытой задачи в handoff.
   - `### 3. Следующие задачи` — таблица `Задача | Действие | Тип | Исполнитель | Зависит от | Статус`; 3–5 следующих. Для каждой:
     - **Задача** — ID `S<N>.<M>` / `S<N>.accept` (или legacy `S<N>.T<M>`) в backticks.
     - **Действие** — короткое описание задачи из `tasks.md` (одна фраза в «ёлочках» или без — глагол + что делается). Без имён процедур и кодов категорий.
     - **Тип** — русское обозначение из набора: `Код модуля` (BSL общий модуль / объектный модуль), `Код формы` (BSL `Forms/.../Ext/Form/Module.bsl`, программное создание элементов), `Ручной тест`, `Ручная регрессия`, `Конфигуратор` (создание/правка метаданных через UI 1С), `Проверка` (Grep/Read оркестратора).
     - **Исполнитель** — русские роли: `агент`, `агент + ревью`, `пользователь`, `оркестратор`. Имена `onec-code-writer` / `onec-code-reviewer` / `onec-code-explorer` / `onec-code-architect` в чат не выводятся (см. §3.1 `opsx-output-style.md`).
     - **Зависит от** — ID задач в backticks; если зависимость `[ ]` — пометить «невыполнима до `<id>`».
     - **Статус** — `[x]` / `[ ]`.
   - `### 4. Как вернуться` — `/opsx:apply <change-name>`, одна строка. Если выявлен scope/design mismatch, добавить вторую строку: `Обновить scope: /opsx:extend <change-name>` (затем снова `/opsx:apply <change-name>`).
   - `### 5. Blockers` — нумерованный список задач, которые не могут продолжаться, и почему.
   - `### 6. Issue` — **только в варианте `pause`**: описание проблемы 1 абзац + нумерованные **Options** из 2–3 вариантов решения. **Тонкий чат:** при сообщении пользователю дублировать развилку блоком прозой по стилю Варианта 3 verify (полное Issue и таблицы — в файле `reports/handoff-*.md`, см. §5.2 выше).
   - `### 7. Short-cut` — **только в варианте `acceptance`**: строка про подтверждение обычной фразой («принято», «срез принят»).

   Если все срезы приняты (`final`) — добавить строку «All tasks complete. Ready to archive: `/opsx:archive <change-name>`». Если `pause` из-за design/scope mismatch — предложить `Follow-up: /opsx:extend <change-name>` рядом с вариантами решения. Если `pause` — ждать ответа пользователя. Если `acceptance` — end turn.

   **Self-check** (см. §7 стайл-гайда) перед выводом: слои разделены, нумерованные списки, одинаковые имена секций, длина в пределах лимитов.

**Output During Implementation**

Между задачами в **чат** — минимум строк (§3a `chat-output-budget.mdc`): без заголовка `## Implementing`, без `Schema:`, без «✓ Task complete» при успехе. Достаточно: «Задача `S<N>.<M>` («…») реализована.» При ошибке/blocker — кратко что не так.

```
Задача `S2.3` («…») реализована.
```

**Output — T-HANDOFF (единый шаблон, см. `.cursor/docs/opsx-output-style.md` §5.2)**

Три варианта заголовка при одинаковых именах секций — выбирается по состоянию:

| Вариант | Заголовок | Когда |
|---------|-----------|-------|
| `acceptance` | `## Срез S<N> — передача на приёмку: <change-name>` | Slice Gate: все рабочие задачи `[x]`, `S<N>.accept` (или legacy `S<N>.T<M>`) ждёт ручной приёмки |
| `pause` | `## Сессия приостановлена: <change-name>` | Issue / неясное требование / пользователь «стоп» |
| `final` | `## Реализация завершена: <change-name>` | Все задачи `[x]`, включая `S<N>.accept` (или все legacy `S<N>.T<M>`) |

**Общая структура вывода (секции 1–5; секции 6–7 — только в указанных вариантах):**

```
## <Заголовок варианта>

**Change:** <change-name>
**Schema:** <schema-name>
**Прогресс:** N/M задач [x] (приёмка среза S<N>.accept: <статус>)

### 1. Что реализовано
1. [x] <id задачи> — <одно предложение>
   - Файл: `<path>`, строки X-Y
   - Что изменено: <одно предложение>
   - [только при проблеме] Замечание: <spot-check / линтер / ревью — кратко>

### Карта правок (перед тестом)
1. <проза> — [`start:end:path`](...)
...
Полная карта: `openspec/changes/<name>/reports/code-map.md`

### 2. Что проверить СЕЙЧАС
1. <шаг 1 в императиве> → <ожидаемый результат> (регрессия R<N>, если применима)
2. <шаг 2> → <ожидаемый результат>
(или: «Ручная проверка не требуется для задач этого сеанса».)

### 3. Следующие задачи
| Задача | Действие | Тип | Исполнитель | Зависит от | Статус |
|--------|----------|-----|-------------|------------|--------|
| `S1.3` | заполнить значения по умолчанию на новом цикле | Код модуля | агент + ревью | `S1.2` | [ ] |
| `S1.4` | проверить пересчёт состава при смене настройки | Код формы | агент + ревью | `S1.3` | [ ] |
| `S1.5` | прогнать сценарий повторного согласования вручную | Ручная регрессия | пользователь | `S1.1`–`S1.4` | [ ] |
| `S1.accept` | принять срез S1 — пройти чеклист сценариев | Ручной тест | пользователь | см. выше | [ ] |

### 4. Как вернуться
`/opsx:apply <change-name>` — продолжит с первого непринятого среза.

### 5. Blockers (если есть)
1. <блокер> — <чем закрывается>

<!-- только для варианта `pause` -->
### 6. Issue
<1 абзац: описание проблемы>

**Options:**
<Развилка прозой по стилю Варианта 3 verify: контекст → варианты с но/зато>

<!-- только для варианта `acceptance` -->
### 7. Short-cut
Если уже проверено и принято — напишите обычной фразой («принято», «срез S<N> принят»), отмечу без полного handoff.

<!-- только для варианта `final` -->
> All tasks complete. Ready to archive: `/opsx:archive <change-name>`.
```

**Имена секций** (`### 1. Что реализовано`, `### 2. Что проверить СЕЙЧАС`, `### 3. Следующие задачи`, `### 4. Как вернуться`, `### 5. Blockers`) — **одинаковы во всех трёх вариантах**; это же именование используется в `debug.md` `## Slice Gate Decisions` → «Связанный отчёт».

**Self-check перед выводом** (§7 стайл-гайда):
1. В «Что проверить СЕЙЧАС» нет внутренних ID (`R<N>/SC<N>/D<N>`) в тексте пункта — только в скобках в конце.
2. Имена команд / модулей / UI — по типографическим правилам §2.
3. Перечисления — нумерованный список.
4. «Что проверить» — императивы, без формулировок гипотез.
5. Каждая секция ≤7 пунктов; длиннее — выносить в отчёт `reports/`.
6. **Первое упоминание `S<N>.<M>` / `S<N>.accept` (или legacy `S<N>.T<M>`)** в заголовке секции / карточки / handoff-сообщения — с коротким описанием задачи в «ёлочках» (например «Что проверить у себя — задача `S1.2` («перенос флага из шаблона в БП»)»). Голый ID без описания в заголовке (например «к `S1.2`», «`S1.2` закрыта с моей стороны») — провал self-check. Срез — по §10 `opsx-output-style.md`.
7. **Колонка «Действие»** в таблице «Следующие задачи» — обязательна; «Тип» и «Исполнитель» — только из русских наборов значений (см. описание шага 7); имена `onec-code-*` в чат не выводятся.
8. **Англицизмы** (`Step-by-step`, `checkpoint`, `Tier`, `Standard/Lite/Full` как метки) и имена движка (`Architect Gate`, `Slice Gate`, `Implementation Impact Gate`) — в пользовательский вывод не попадают; внутренние ID триггеров (`slice-size-threshold`, `awaiting-acceptance`) — только в `debug.md` / в скрытом контексте модели.

**Guardrails**
- **Output style:** T-HANDOFF §5.2 + **Chat Surface Contract** §2.6 `opsx-output-style.md`: handoff среза на языке эффекта; next step — только user-action команды (`/opsx:verify`, `/opsx:apply`, `/opsx:archive`); thin handoff `acceptance` **включает** строку «после проверки: обычная фраза „принято“ или `/opsx:apply <name>`»; без internal-команд и перечней файлов. Self-check §2.6 перед отправкой.
- Keep going through tasks until done or blocked
- Always read context files before starting (from the apply instructions output)
- **Spot-check after each task:** Grep (or Read) to confirm the change; for 5+ files, check at least 3. Do not mark task complete if verification fails.
- If task is ambiguous, pause and ask before implementing
- If implementation reveals issues, pause and suggest artifact updates
- Keep code changes minimal and scoped to each task
- Update task checkbox immediately after completing each task
- **Code-Truth Journal:** for every writer-completed BSL/form task, append factual symbols from `created_or_modified_symbols` to `debug.md`; if code and artifacts diverge, pause and route to `/opsx:extend <change-name> --code-sync`.
- **Code Map:** at Slice Gate (acceptance), append `reports/code-map.md` and mandatory `reports/handoff-acceptance-S<N>-YYYY-MM-DD.md`; chat block «Карта правок» — human prose first, anchors second (≤5 items).
- Pause on errors, blockers, or unclear requirements - don't guess
- Use contextFiles from CLI output, don't assume specific file names
- **Task Dispatch:** classify each task before implementation; delegate to the correct executor per dispatch table. Orchestrator MUST NOT implement BSL or form-module tasks directly — only prepare context and delegate.
- **Form XML / Configurator tasks:** STOP — produce manual configuration instructions (`1c-agent-delegation.mdc` § XML WRITE GUARD); do not edit `Form.xml` or continue until the user has performed configuration and re-exported.
- **Form module BSL tasks:** if the task changes `Forms/.../Ext/Form/Module.bsl` only (for example `Элементы.Добавить`, visibility, event handlers), run the standard BSL writer/reviewer pipeline; this is not a `Form.xml` task.
- Reference: `1c-agent-delegation.mdc` (BSL gate), `1c-utility-agents.mdc` (forms, queries, tests).
- **vertical-slices.mdc:** вердикт Slice Gate **[3]** и любое добавление `# Срез` — только по **ИНВАРИАНТ: Defect placement** (не создавать fix-срез при приёмочной задаче среза `S<K>.accept` (или legacy `S<K>.T<M>`) = `[ ]` без cross-slice).

**Fluid Workflow Integration**

This skill supports the "actions on a change" model:

- **Can be invoked anytime**: Before all artifacts are done (if tasks exist), after partial implementation, interleaved with other actions
- **Allows artifact updates**: If implementation reveals design issues, suggest updating artifacts - not phase-locked, work fluidly
