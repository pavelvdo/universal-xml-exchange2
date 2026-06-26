---
name: openspec-archive-change
description: Archive a completed change in the experimental workflow. Use when the user wants to finalize and archive a change after implementation is complete.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.1"
  generatedBy: "1.1.1"
---

Archive a completed change in the experimental workflow.

**Output style:** итог в чат — **одна строка эффекта** («заархивировано») + опц. actionable KB/ADR; **Chat Surface Contract** §2.6: без internal-команд, без перечня non-events. T-CONFIRM §5.5. Self-check §2.6 + §3a `chat-output-budget.mdc`.

**Auto-yes policy:** Invoking archive means the user accepts the recommended path: proceed despite incomplete artifacts/tasks, **sync delta specs to main** when a delta exists, and **extract all ADR-worthy decisions** from architecture reports. Do **not** use **AskQuestion** for ADR/sync. **Не** выводить в чат предупреждения о: незакрытых follow-up в `tasks.md`, пустом диффе маркеров `0/0`, отсутствии analytical reports / KB-кандидатов / отложенном KB (см. шаги 2–3, 5.5). **Исключения (AskQuestion разрешён):** шаг 1 — выбор change при неоднозначности; шаг 3.5 — непринятые приёмочные задачи в slice mode (`S<N>.accept` или legacy `S<N>.T<M>`); шаг 5.5 — сохранение KB-фактов (когда кандидаты есть).

**Input**: Optionally specify a change name. If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **Select the change**

   If a name is provided, use it. Otherwise:
   - Infer from conversation context if the user mentioned a change
   - Auto-select if only one active change exists
   - If multiple active changes: run `openspec list --json`, show only active changes (not archived), include schema, and use **AskQuestion** to let user select

   Имя change **не** дублировать отдельной строкой в чате до итога — оно входит в финальное T-CONFIRM (non-event). При AskQuestion на шаге 1 — только варианты выбора.

2. **Check artifact completion status**

   Run `openspec status --change "<name>" --json` to check artifact completion.

   Parse the JSON to understand:
   - `schemaName`: The workflow being used
   - `artifacts`: List of artifacts with their status (`done` or other)

   Maintain a **warnings accumulator** for step 7.

   **If any artifacts are not `done`:**
   - Зафиксировать внутренне (не выводить в чат — non-event по §3a); **do not** use AskQuestion; continue automatically

2a. **Verify freshness (внутренний прогон, без «лишнего» шага для пользователя)**

   Glob в `openspec/changes/<name>/reports/verification-*.md`. Взять последний по дате, прочитать YAML `snapshot` / `verify_mode`. Свежий финальный отчёт — это прогон с `verify_mode: post-apply` после последней правки `tasks.md`.

   **Если свежий финальный отчёт verify найден** — **silent** в чат (non-event); отчёт остаётся в `reports/` архивируемой ЗНИ.

   **Если свежий финальный отчёт не найден:**
   1. Выполнить **молча** (без рекомендации отдельной команды verify) полный прогон по скиллу `openspec-verify-change/SKILL.md` в режиме финальной проверки (`verify_mode: post-apply`).
   2. Сохранить отчёт в `reports/verification-YYYY-MM-DD.md` (имя по шагу 18 verify; суффикс `-2`/`-3` при повторе за день).
   3. **Чат:** не упоминать факт прогона, если итог чистый и архив не заблокирован.
   4. Если прогон выявил **блокеры** или обязательные решения пользователя — **СТОП** до шага 4 (не переносить change в archive): краткая карточка на языке заказчика + путь к `reports/verification-*.md` (бюджет `.cursor/rules/chat-output-budget.mdc`). Пользователь устраняет замечания и снова вызывает `/opsx:archive <name>`.

   **Не** добавлять в warnings текст вида «рекомендуется запустить verify перед архивом» — вместо предупреждения выполнен внутренний прогон или найден существующий отчёт.

2b. **Knowledge verify в scope ЗНИ (soft-gate, не блокирующий)**

   Определить изменённые файлы: `git diff --name-only <merge-base>...HEAD` (точка ответвления change-ветки от main).

   Найти KB-факты, чьи anchor-paths пересекают этот diff:
   Read `openspec/knowledge/_index.yaml`, фильтр по `anchor-paths` ∈ `diff.files`.

   Verify каждый (алгоритм §3 из knowledge-format.mdc). **Только если** drift > 0: для каждого drift добавить в warnings accumulator краткую строку: `KB-NNNN: <drift type> — <title>`. Если drift = 0 — **silent** (не выводить «verified N, drift 0»). Не блокировать archive.

3. **Check task completion status**

   Read the tasks file (typically `tasks.md`) to check for incomplete tasks.

   Count tasks marked with `- [ ]` (incomplete) vs `- [x]` (complete).

   **If incomplete tasks found:**
   - **Не** добавлять в warnings и **не** выводить в чат (follow-up остаются в архивном `tasks.md` — non-event).
   - **Do not** use AskQuestion; continue automatically

   **If no tasks file exists:** Proceed.

   3.2. **Check developer comment markers balance (HARD BLOCKER)**

   Read `proposal.md` and `openspec/project.md` (`cfMarkerPrefix`) and check for the `## Metadata (comment markers)` block.
   - If the block exists, extract `developer` (ФИО) via dual-parser (yaml or list `- **developer:**`).
   - Build a diff of all `*.bsl` files in the change relative to the baseline (merge-base with main/master).
   - In the diff, count **added** open markers:
     - **cfe canon:** `// +++ {developer}` (substring match on developer FIO)
     - **cfe legacy:** `// +++ {developer} ... [` (contains `[ID#` or `[б/н#` or `[б/н]`)
     - **cf canon:** lines matching `// {cfMarkerPrefix}.*\+\+\+` (cfMarkerPrefix from project.md)
   - Count **added** close markers:
     - **cfe:** `// --- {developer}` (optional trailing `[...]` for legacy)
     - **cf:** `// {cfMarkerPrefix without colon} ---`
   - **If the counts do not match:** hard blocker — сообщить дисбаланс, stop.
   - **If `count_open = 0` and `count_close = 0`:** silent, proceed.
   - **If counts match:** proceed.

   3.5. **Check slice acceptance status (slice mode only — HARD BLOCKER)**

   Detect slice mode: grep `tasks.md` for `^# Срез S\d+`.

   **Legacy mode (нет `# Срез`):** пропустить шаг 3.5.

   **Bypass `--force-legacy`:** Если в команде пользователя есть флаг **`--force-legacy`**, не показывать карточку и AskQuestion. Запомнить для шага 7 одну строку T-CONFIRM (не `### Warnings`): кратко «Архив с --force-legacy; непринятые приёмочные задачи (`S<N>.accept` / legacy `S<N>.T<M>`) остаются `[ ]`» (перечислить только при 1–2 задачах, иначе «см. `tasks.md`»). Продолжить со шага 4.

   **Если slice mode и нет `--force-legacy`:**

   1. Парсинг `tasks.md` по блокам между заголовками `# Срез S<N>`.
      - **Приёмочная задача среза** (новый формат) — строка чеклиста с ID **`S<N>.accept`**.
      - **Приёмочная задача среза** (legacy формат) — одна или несколько строк с ID **`S<N>.T<M>`** (после первой точки идёт буква **`T`**), при отсутствии `S<N>.accept` в этом срезе.
      - **Рабочая задача среза** — строка с **`S<N>.<число>`**, где после точки только цифры (например `S1.3`), без `T` и без `accept`.
      - Для каждого среза определить **acceptance set**: новый формат — `S<N>.accept`; legacy — все `S<N>.T<M>`. Если в acceptance set есть `[ ]` — пометить срез «есть непринятые приёмочные задачи» и проверить: **все** рабочие задачи этого среза `[x]`? Если да — срез **готов к приёмке из archive**; если нет — срез **не готов** (перечислить незакрытые рабочие ID).

   2. **Если acceptance set всех срезов целиком `[x]`** — продолжить со шага 3.6. В чат **не** выводить строку `Slices: K/K приняты` (non-event), кроме случаев п.5 `--force-legacy` / п.5 вариант **D** / явного принятия задач вариантом **A** при архивации (тогда одна строка в итоговом T-CONFIRM: «Приёмка: зафиксирована при архивации» или «Архив с --force-legacy»).

   3. **Если есть `[ ]` на любой приёмочной задаче (`S<N>.accept` или legacy `S<N>.T<M>`):** показать **краткую карточку** (не полный T-HANDOFF из `/opsx:apply`; итоговый вывод archive остаётся **T-CONFIRM**, §5.5):

      ```
      ## При архивации — непринятые срезы

      | Срез | Primary acceptance (из metadata) | Готовность |
      |------|-----------------------------------|------------|
      | S1: «…» | <текст Primary или «не задан»> | да / нет |
      ```

      Пояснение одной строкой: без **`[x]`** на accept контракт среза не закрыт. Опция **A** — подтверждение, что **Primary пройден на ИБ** (ответственность пользователя).

   4. **Условие опции A:** опция **A** в AskQuestion допустима **только если** каждый срез, в котором есть `[ ]` в acceptance set, уже **«готов»** (все рабочие задачи среза `[x]`). Если условие не выполнено — перед AskQuestion вывести строку «**Вариант A недоступен:** есть срезы с незакрытыми задачами реализации; завершите через `/opsx:apply <name>`.» и выдать **AskQuestion только с опциями B, C, D**.

   5. **AskQuestion** (исключение из auto-yes). Подписи опций должны явно содержать дисклеймер: подтверждение успешного прогона на ИБ — ответственность пользователя.

      - **A.** *(если выполнено условие п.4)* Primary пройден на ИБ — отметить accept каждого незакрытого среза `[x]`, продолжить архив. **Дисклеймер:** подтверждаете успешный прогон Primary на ИБ.
      - **B.** Тесты не пройдены / нужна доработка → **STOP**; рекомендовать `/opsx:apply <name>` или `/opsx:verify <name>`.
      - **C.** Отложить архив → **STOP**.
      - **D.** Принудительное продолжение **без** отметки в `tasks.md` (семантика **`--force-legacy`**) → шаги 4–7; в warnings: `Archived with --force-legacy: …` (перечислить непринятые приёмочные задачи).

   6. Обработка ответов: **B** или **C** → завершить archive (**return**). **D** → для шага 7 строка T-CONFIRM (не Warnings): «Принудительное продолжение без отметки приёмочных задач; см. `tasks.md`.», продолжить шаг 4.

   7. **Ответ A — обязательные артефакты (MUST):**
      - Для каждого затронутого среза заменить acceptance set с `[ ]` на `[x]`:
        - Новый формат: одна строка `- [ ] S<N>.accept …` → `- [x] …` (без изменения чеклиста и текста заголовка).
        - Legacy: каждая `- [ ] S<N>.T<M> …` → `- [x] …`.
      - **Append** в `debug.md` секцию **`## Slice Gate Decisions`** — по **каждому** затронутому срезу отдельный подблок или один сводный с перечнем приёмочных задач; формат:
        ```markdown
        ### Slice S<N> — <краткое имя из заголовка среза> (YYYY-MM-DD)
        Срез: S<N> — <имя>
        Решение: принят (archive)
        Обоснование: подтверждение пользователя при `/opsx:archive`; приёмочные задачи отмечены в tasks.md.
        Изменения tasks: отмечены [x]: <S<N>.accept | S<N>.T<M>, …>
        Связанный отчёт: reports/slice-acceptance-S<N>-YYYY-MM-DD.md
        ```
      - Для **каждого** затронутого среза создать **`reports/slice-acceptance-S<N>-YYYY-MM-DD.md`**: дата, `Primary acceptance: pass | fail | skipped (причина)`, optional-сценарии, перечень accept-задач, напоминание что прогон подтверждён пользователем при archive.

   8. После выполнения п.7 **повторить** парсинг п.1–2: каждый acceptance set должен быть полностью `[x]`. Если после правок остался `[ ]` — **STOP**, сообщить о несоответствии `tasks.md` (ошибка парсера/формата). Иначе — перейти к шагу **3.6**.

   Итог в чат после успешного прохода без force-legacy и без варианта A: **без** строки про срезы. Если был вариант **A** — одна короткая строка в T-CONFIRM (см. шаг 7).

   3.6. **Code-Truth Gate (HARD BLOCKER for completed scope)**

   Выполнить `.cursor/rules/code-truth-gate.mdc` для `design.md`, `tasks.md`, `debug.md`, `specs/**`:
   - извлечь технические символы (`pav_*`, имена процедур/функций в backticks, аннотации расширения, стабильные имена элементов формы);
   - проверить их `Grep`/`rg` по путям из `openspec/project.md` и явно затронутым файлам;
   - игнорировать строки-флаги, ADR/KB/Scenario/Requirement, команды `/opsx:*`.

   Если найден `phantom-symbol`, относящийся к `[x]` задаче, принятому срезу или завершённому legacy-scope:
   - это **hard blocker** (override auto-yes);
   - не выполнять шаги 4–7;
   - показать пользователю:
     ```markdown
     ## Архив заблокирован: артефакты ссылаются на несуществующий код

     Найдены технические имена в `design.md` / `tasks.md` / `debug.md` / `specs/**`, которых нет в выгрузке.

     | Symbol | Artifact | Scope |
     |--------|----------|-------|
     | <symbol> | <path> | <task/slice> |

     Варианты:
     1. `/opsx:extend <name> --code-sync` — если код верен, а артефакты устарели.
     2. `/opsx:apply <name>` — если артефакты верны, а код не реализован.
     ```
   - завершить archive (return).

   Если `phantom-symbol` относится только к незавершённым follow-up задачам — **silent** (не warning в чат); продолжить.

4. **Assess delta spec sync state**

   Check for delta specs at `openspec/changes/<name>/specs/`. If none exist, proceed without sync.

   **If delta specs exist:**
   - Compare each delta spec with its corresponding main spec at `openspec/specs/<capability>/spec.md`
   - Determine what changes would be applied (adds, modifications, removals, renames)
   - **Не** выводить в чат сводку «что было бы смержено» (non-event); при необходимости аудита — опционально `reports/archive-delta-assessment-<name>-YYYY-MM-DD.md` в каталоге change **до** шага 6.

   **Default action (automatic):**
   - If the delta is **already fully reflected** in main specs: **silent** в чат (не писать `Specs: Already up to date`); **do not** re-run sync unless you detect drift
   - If **changes are needed**: **always** execute sync using `/opsx:sync` logic (read and follow `.cursor/skills/openspec-sync-specs/SKILL.md` for the active change)

   Proceed to the next step after sync completes or after confirming no merge was needed.

5. **ADR extraction (architecture decision records)**

   Before reading reports, resolve the **planned archive path** for Source fields: `openspec/changes/archive/YYYY-MM-DD-<change-name>/` (same date and name as in step 6). Use `.../reports/<architecture-file>.md` in each ADR **Source** even though the move happens in step 6.

   Check if `reports/architecture-*.md` exists in the change directory.

   **If architecture reports found:**
   - For each `architecture-*.md`, read the report and identify key **decisions** (not analysis/validation).
   - A decision is ADR-worthy if: it affects future changes, involves trade-offs between alternatives, or establishes a contract/pattern/principle. See criteria in `.cursor/rules/adr-format.mdc`.
   - **Не** выводить в чат список кандидатов ADR до извлечения (non-event).
   - **Automatically extract all** ADR-worthy decisions (equivalent to former option «Да, извлечь все»):
     1. Determine next ADR number: Glob `openspec/adrs/ADR-*.md`, take max NNNN + 1 (or 0001 if empty)
     2. For each selected decision, create ADR file using format from `.cursor/rules/adr-format.mdc`:
        - Status: **Accepted** (decision was implemented in the change being archived)
        - Source: planned archive path to the report file (see above)
        - Area: derive from change context (proposal.md topic)
     3. Update `openspec/adrs/README.md` — add row to the index table
   - If no ADR-worthy candidates exist after review: **silent** в чат (no files created).

   **If no architecture reports found:** Skip this step; **silent** в чат.

5.5. **Facts extraction (single-decision)**

   Использовать единый extraction-протокол из `.cursor/skills/openspec-knowledge-add/SKILL.md` (разделы `Extraction Contract`, `Candidate Validation`, `Preview / AskQuestion`, `Save Protocol`) с archive-specific overrides ниже. Это единственный источник истины для отбора KB-кандидатов, проверки anchors, dedup, TTL и отказных состояний.

   **Inputs:** `reports/exploration-*.md`, `reports/trace-analysis-*.md`, `reports/resolved-contract-*.md` в каталоге архивируемой ЗНИ. Качество фактов — ответственность самих reports: archive агрегирует verified-факты и применяет фильтры `knowledge-worthy` + `Reuse Value Test`, но не проводит новое обследование кода.

   **Archive-specific overrides:**
   - Planned stable source для `source.report`: `openspec/changes/archive/YYYY-MM-DD-<change-name>/reports/<report>.md` (тот же путь, куда report попадёт после шага 6).
   - Source bundle **не создаётся**: архивный report сам является стабильным source.
   - Если reports не найдены → state = `Skipped — no analytical reports`.
   - Если taxonomy отсутствует → state = `Blocked — taxonomy missing`; в **чат** в итоге T-CONFIRM одна actionable строка: «База знаний: заблокирована — нет таксономии. Дальше: `/opsx:knowledge-init` или `/init-project`.» (см. шаг 7).
   - Если кандидатов нет после фильтров → state = `No candidates after filters`; archive продолжается; **silent** в чат.
   - Если кандидаты отброшены Reuse Value Test → state = `Deferred N — reuse value not justified`; **не** добавлять в warnings и **не** выводить в чат (non-event).
   - Если кандидаты есть → показать per-candidate карточки из `openspec-knowledge-add` и один AskQuestion: «Сохранить N извлечённых KB-фактов? [yes | no]».
   - `yes` → сгенерировать `KB-NNNN-slug.md` + атомарно обновить `_index.yaml` → state = `Saved N (KB-NNNN, ...)`.
   - `no` → ничего не писать → state = `Declined by user`.

   **Mapping состояний (резюме):**

   | Условие | Knowledge state |
   |---|---|
   | Saved (yes на AskQuestion) | `Saved N (KB-NNNN, ...)` |
   | Declined (no на AskQuestion) | `Declined by user` |
   | Reports есть, кандидатов нет после фильтров | `No candidates after filters` |
   | Кандидаты есть, но все/часть отложены Reuse Value Test | `Deferred N — reuse value not justified` (silent в чат) |
   | Reports не найдены вовсе | `Skipped — no analytical reports` |
   | Reports есть, taxonomy отсутствует | `Blocked — taxonomy missing` |

5.5.b **Invariant extraction (optional, поведенческие ЗНИ)**

   Выполнять, если change содержит принятые срезы и меняет **поведение пользователя** (есть `## Behavior Contract` в design или UX-сценарии в spec). Цель — зафиксировать устойчивые контракты для будущих verify Layer 2.4 / precedent-regression-gate.

   **Эвристика кандидатов (до 5 на change):** прочитать `design.md` и архивный `proposal.md` до переноса; выделить строки в `## Goals`, `## Decisions`, `## Behavior Contract`, содержащие маркеры вроде «сохраняется», «не очищается», «инвариант», «обязательно», «не снимать». Исключить чисто технические ограничения без UX-эффекта.

   **Делегирование:** вызвать `onec-code-architect` с `mode=invariant-extraction`. Архитектор классифицирует каждый кандидат: **Load-bearing ADR** (создать/обновить `openspec/adrs/ADR-NNNN.md` по `.cursor/rules/adr-format.mdc` с `Load-bearing: yes` и `Protects-invariants:`), **invariant KB** (факт в `openspec/knowledge/` с `invariant: true`, якорями и `protected-by-changes:`), или **отклонить** (не несущий контракт). Применить **Reuse Value Test** из `openspec-knowledge-add` перед записью KB.

   **Запись:** новые ADR — по формату adr-format; KB — атомарное обновление `_index.yaml` + файл факта. Если пользователь отклонил сохранение — **silent** в чат (non-event); archive не блокировать.

   **Порядок:** шаг 5.5.b выполнять **после** 5.5 (facts extraction), **до** шага 6 (perform the archive), чтобы пути Source указывали на планируемый архивный каталог.

6. **Perform the archive**

   Create the archive directory if it doesn't exist:
   ```bash
   mkdir -p openspec/changes/archive
   ```

   Generate target name using current date: `YYYY-MM-DD-<change-name>`

   **Check if target already exists:**
   - If yes: Fail with error, suggest renaming existing archive or using different date
   - If no: Move the change directory to archive

   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

   **`reports/code-map.md`:** при архивации переезжает вместе с каталогом change в `openspec/changes/archive/YYYY-MM-DD-<name>/reports/` как **история правок для человека** (non-event в чат — не дублировать путь отдельной строкой, если нет других материальных результатов).

7. **Display summary**

   Собрать **T-CONFIRM** в чат по правилам **skip-on-empty** и **§3a** `chat-output-budget.mdc`.

   **Обязательная строка (всегда):**
   - `ЗНИ в архиве: openspec/changes/archive/YYYY-MM-DD-<name>/`

   **Опциональные строки (только если есть материальный результат):**
   - **Спека:** если на шаге 4 **фактически изменены** файлы `openspec/specs/**/spec.md` — `Спека обновлена: <path>[, <path>…]`. Если delta уже была в main / синхронизация не меняла файлов — **не** выводить.
   - **ADR:** если создан **новый** файл `openspec/adrs/ADR-NNNN*.md` — `ADR: ADR-NNNN[, ADR-MMMM]`. Если только **обновлён** существующий ADR — `ADR: ADR-NNNN (обновлён)`. Если ADR не создавались и не обновлялись — **не** выводить строку ADR.
   - **База знаний:** только если шаг 5.5 дал `Saved N (KB-…)` — `База знаний: KB-NNNN[, …]`. Если state = `Blocked — taxonomy missing` — actionable строка с `/opsx:knowledge-init` (см. шаг 5.5). Иные состояния Knowledge — **silent**.
   - **Приёмка / force-legacy:** только если был вариант **A** при архивации — коротко: «Приёмочные тесты отмечены при архивации.»; если **`--force-legacy`** или **D** — одна строка: «Архив с принудительным продолжением: непринятые тесты остались в `tasks.md`.»

   **`### Warnings`** — выводить **только** если после фильтра §3a в accumulator остались пункты (например **только** KB-drift из шага 2b). Если пусто — **не** выводить секцию Warnings. Не включать: незакрытые задачи, пустой дифф маркеров, отсутствие KB-отчётов, Deferred/Declined Knowledge, незапуск invariant-extraction.

**Output On Success**

```
## Архивация выполнена

ЗНИ в архиве: openspec/changes/archive/YYYY-MM-DD-<name>/
[Спека обновлена: <paths>]  ← только если были правки main spec
[ADR: ADR-NNNN …]           ← только если созданы/обновлены
[База знаний: KB-NNNN …]     ← только Saved; или строка про taxonomy blocked
[Приёмка / force-legacy — одна строка при необходимости]

### Warnings
- … ← только KB-drift или иные остаточные actionable пункты; иначе секцию опустить
```

**Guardrails**
- Always prompt for change selection if not provided (step 1 only)
- Use artifact graph (`openspec status --json`) for completion checking
- **Don't block archive** на soft-gates; hard blockers — шаги 3.2 (дисбаланс маркеров), 3.5 (карточка), 3.6 (phantom по принятому scope), внутренний verify 2a при блокерах.
- **EXCEPTION (slice mode, step 3.5):** при любой приёмочной задаче среза (`S<N>.accept` или legacy `S<N>.T<M>`) = `[ ]` — **pause** до **AskQuestion**: варианты **A** (отметить `[x]` и продолжить при готовых срезах), **B/C** (стоп), **D** / **`--force-legacy`**. Флаг **`--force-legacy`** в команде обходит карточку.
- **Recommended actions are automatic:** delta spec sync when merge is needed; ADR extraction for all ADR-worthy decisions from `reports/architecture-*.md`
- Preserve `.openspec.yaml` when moving to archive (it moves with the directory)
- Use `openspec-sync-specs` approach (agent-driven) whenever sync runs
- If delta specs exist, always run the sync assessment **без вывода сводки в чат** (шаг 4); при необходимости — файл в `reports/`.
