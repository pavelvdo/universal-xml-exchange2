---
name: openspec-debug
description: Investigate a bug (trace/remark/screenshots) in context of an OpenSpec change; produce RCA and capture fix tasks in change artifacts.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: project
  version: "2.2"
---

Investigate a test-found bug and produce a root-cause analysis and a fix plan that is **captured inside the same OpenSpec change**.

This skill is designed for the `/opsx:debug` command (alias: `/debug`).

## Input (free-form)

The user may provide any combination of:
- Trace file path (e.g. `C:/GitHub/PavDO/temp/Замеры/...txt`)
- Textual remark / expected vs actual behavior
- Screenshots (attached images)
- Optional OpenSpec change id (e.g. `fix-exclude-participants-v2`)
- Optional task number/id (e.g. `4.3`)

## Output (what you must produce)

1. **Scope**: which change and which task(s) this relates to
2. **Evidence summary**: what the trace/remark/screenshots show
3. **Root cause**: why it happens (with file/function references)
4. **Fix plan**: code steps + artifact steps (tasks.md, design.md)
5. **Apply (default scenario)**: capture tasks in `tasks.md` for `/opsx:apply` and update artifacts (add/refine tasks, add note in `design.md` if the bug reveals a design gap). Then hand off to `/opsx:apply <change>` for implementation.

## Guardrails

- **Always separate verified facts from hypotheses in RCA.** Never present a hypothesis as a verified fact.
- **Default scenario:** after producing the fix plan, **capture tasks** in tasks.md and update design.md. Code implementation is done via `/opsx:apply`. Only skip artifact edits if the user explicitly asks for "plan only".
- **Never invent metadata names**. Any reference to metadata or types must be validated against XML dumps in `src/` (see "Metadata validation" below).
- If information is insufficient: ask 1–2 targeted questions, then continue.

## Steps

## Entry Protocol (MANDATORY)

При входе в debug **первый шаг** — определить change, загрузить контекст, сформировать и **показать бриф**. **Исключение:** при вызове из `/opsx:intake` п.7a **А** в том же ответе выше уже есть intake-бриф; после шага **0** выводите только `### Дополнение: /opsx:debug` (continuation-from-intake). До подтверждения брифа запрещены: вызов Task (trace-analyst, explorer, architect), чтение файла трассы. Единственные допустимые действия до брифа: Read этого скилла (command-skill-gate), определение change, загрузка артефактов change, `openspec/project.md`, Knowledge Discovery для поля «KB в scope»: Read `openspec/knowledge/_index.yaml` и при необходимости `_taxonomy.yaml` и выбранных KB `.md` (алгоритм как в explore Entry Protocol шаг 1.5, бюджет Top-10). **Без** чтения BSL/XML и файла трассы. После показа брифа — **END TURN**; продолжение — только после явного подтверждения пользователя в следующем сообщении.

### 0. Определить change и загрузить контекст

- Если change id указан в запросе — использовать его.
- Иначе: выполнить `openspec list --json`, при неоднозначности — **AskQuestion** для выбора change.
- Объявить: `Using change: <name>`, способ переопределения: `/debug <other-change>`.
- Выполнить `openspec instructions apply --change "<name>" --json`.
- Прочитать существующие `contextFiles` (proposal.md, design.md, tasks.md, specs/** при наличии).
- Прочитать `openspec/project.md` — извлечь пути к cf и cfe из секции «Структура репозитория» (для передачи субагентам по правилу project-paths.mdc).
- **KB в scope (до брифа):** прочитать `openspec/knowledge/_index.yaml`; при необходимости `_taxonomy.yaml` и KB `.md` — по алгоритму Entry Protocol шаг 1.5 `.cursor/skills/openspec-explore/SKILL.md` (Tier 1 по `anchor-paths` и путям из `design.md` / `tasks.md`; при отсутствии совпадений — явная формулировка «нет совпадений по anchor-paths и домену» для брифа).
- Если пользователь указал номер задачи: найти её в tasks.md и считать основной областью; если не найдена — показать соседние и уточнить.

### 0b. Классификация входа и бриф

- **Continuation-from-intake:** если **выше в том же сообщении** уже выведен полный блок `## Бриф: /opsx:intake | …` (intake п.7a **А**, маршрут `/opsx:debug`), после выполнения шага **0** — **не** выводить второй полный заголовок `## Бриф: /opsx:debug | …`. Вывести **только** `### Дополнение: /opsx:debug` по SSOT `.cursor/docs/opsx-output-style.md` §5.1 подраздел «Составной бриф». **Обязательно во вложении:** **KB в scope** (шаг 0); **Контекст change** — одна строка: имя ЗНИ `<name>` и суть из `proposal.md` (intake этого не даёт); **План** (как в примере ниже); **Подтвердить?** (запуск шага 3 / делегирования). **Не** дублировать **Симптом**, **Артефакты**, **Контекст**, **Что я понял** и прочие секции, уже присутствующие в intake-блоке. Допускается **Технический контекст (дополнение)** из `design.md`, если критично для плана и отсутствует во intake. **END TURN.** Не вызывать Task, не читать трассу. Дождаться явного подтверждения («ОК», «Да», «Подтверждаю») в **следующем** сообщении.
- **Обычный вход** (прямой `/opsx:debug` или ответ пользователя после AskQuestion intake):
  - **Классифицировать вход:** трасса (путь к .pff / *_TRACE_*.txt) / текстовое замечание (ожидание vs реальность) / скриншот(ы).
  - **Сформировать бриф** из: контекст (proposal); сценарий (ожидаемое из tasks.md); технический контекст (модули/процедуры из design.md); симптом (только из ввода пользователя, при трассе **не** читая файл); артефакты (пути); **KB в scope** (шаг 0); план исследования: 1 → onec-trace-analyst при трассе; 2 → onec-code-explorer; 3 → onec-code-architect при необходимости; 4 → RCA → tasks.md → `/opsx:apply`
  - **Показать бриф** пользователю по шаблону ниже (полный `T-BRIEF`).
  - **END TURN.** Не вызывать Task, не читать трассу. Дождаться явного подтверждения («ОК», «Да», «Подтверждаю») в **следующем** сообщении.

**Шаблон брифа (только для обычного входа):** адаптивный `T-BRIEF` в чате по `.cursor/docs/opsx-output-style.md` §5.1. Заголовок: `## Бриф: /opsx:debug | change: <name>`.

**Обязательно:** Контекст; Что я понял (суть симптома / задачи); **Сценарий** (ожидаемое из tasks — опциональный заголовок §5.1, но для debug обычно заполнен); **Симптом** или блок **Факты** (только наблюдаемое, без «должно быть»); **Технический контекст** из design; **Артефакты** при наличии; **KB в scope**; **План** (ниже); **Подтвердить?**

Пример каркаса **План** (в **Плане** допустимы внутренние ID):

1. → `onec-trace-analyst` (если есть трасса) — путь к файлу трассы + контекст брифа
2. → `onec-code-explorer` — верификация VQ / цепочка вызовов
3. → `onec-code-architect` — при необходимости по гейтам
4. → RCA → задачи в `tasks.md` → hand off на `/opsx:apply <change>`

Закрывающая строка: «Бриф верный? Подтвердите — начну с шага 3 (загрузка входов) и делегирования.»

Файлы `temp/briefs/*.md` не создаются.

**Self-check перед выводом** (см. `.cursor/docs/opsx-output-style.md` §7):

- **Continuation-from-intake:** в блоке `### Дополнение` обязательны **KB в scope**, **Контекст change**, **План**, **Подтвердить?**; пункты (1)–(5) для полей, уже покрытых intake-блоком выше, к дублю в дополнении не применяются; **KB в scope** — только в дополнении.
- **Обычный вход:** (1) слои разделены — в «Симптом / Сценарий / Контекст / Что я понял» нет `D<N>/S<N>.T<M>/R<N>/I<N>/SC<N>` и номеров задач вида `12.9`; (2) имена объектов — UX-надписи в «ёлочках», идентификаторы кода в backticks; (3) любое перечисление ≥2 пунктов — нумерованный список; (4) «Симптом» / факты без «должно быть / ожидается»; (5) каждое поле ≤3 строк или ≤7 пунктов; (6) секция **KB в scope** в чате заполнена.

Если данных недостаточно для брифа — задать 1–2 уточняющих вопроса (AskQuestion). **После подтверждения** перейти к шагу 3 (Load debug inputs) и далее.

---

## Per-turn Delegation Gate (MANDATORY на follow-up)

На **каждом follow-up ходе** (после завершения Entry Protocol) перед выполнением действия:

1. **Классифицировать запрос:** подразумевает ли он обследование кода, трассировку вызовов, анализ модулей?
2. **Маркеры обследования:** «обследуй», «проверь в коде», «найди где», «проследи вызов», «уточни в коде», «посмотри модуль», «как вызывается», «откуда берётся», а также контекст задач из tasks.md типа «уточнить в коде базы».
3. **При срабатывании → СТОП:**
   - НЕ запускать Grep, Glob, Read по .bsl/.xml модулям для анализа логики.
   - Сформировать бриф (агент + что искать + артефакты контекста).
   - Делегировать через Task (onec-code-explorer / onec-trace-analyst / onec-code-architect).
4. **Допустимо до делегирования:** Read артефактов OpenSpec (proposal, design, tasks, specs) для обогащения брифа — до 3 файлов. Grep/Glob/Read по .bsl для обследования логики — запрещено; допустимы точечные обращения (пути к файлам, проверка наличия процедуры) в объёме до 3 обращений.

Контекст оркестратора — дорогой ресурс; обследование кода выполняют субагенты в изолированном контексте.

---

### 3) Load debug inputs

- **Если указан путь к файлу трассы:** зафиксировать путь. **Не читать** файл трассы — он будет передан onec-trace-analyst по пути. Записать только текстовое описание ошибки пользователя как доказательство (симптом). См. `.cursor/rules/1c-error-analysis.mdc` (ЗАПРЕТ подменять trace-analyst ручным чтением).
- If screenshots are attached: read image attachments and extract what the UI shows (titles, messages, states).
- If only a textual remark is provided: treat it as evidence and ask for missing reproduction details only if necessary.

### 3.5) Trace analysis (when trace or multi-line stack is available)

**3.5.0. Контекст для trace-analyst.** Бриф уже подготовлен в Entry Protocol (шаг 0b). Здесь — финализация: убедиться, что в промпт trace-analyst входят суть доработки (proposal), ожидаемое поведение затронутой задачи (tasks.md), затронутые модули (design.md), что искать в трассе (вывести из ожидаемого поведения), фактический симптом (из шага 3 — текст ошибки/замечание пользователя). Передавать trace-analyst **путь к файлу трассы** (не содержимое) и подготовленный бриф (см. `.cursor/rules/1c-error-analysis.mdc`, шаг 1 ДЕЙСТВИЕ — «Подготовить бриф»).

If step 3 loaded a trace (PFF/TRACE file) or an error stack with 3+ call lines:

> Перед вызовом Task — **Task Pre-call Checklist** из `.cursor/rules/tool-name-guard.mdc` (subagent_type из списка 1С-агентов; `model` — по `.cursor/rules/model-selection.mdc`).

1. **Run onec-trace-analyst** (Task tool, subagent_type="onec-trace-analyst"):
   - Pass the trace file path **and the enriched context brief** (from step 3.5.0).
   - Do not pass only "Parse trace" — use the structured brief so the agent can focus analysis (expected behavior, relevant modules, what to look for in the trace).
   - Obtain: structured summary, key findings relevant to the error, Verified facts / Hypotheses.

2. If the trace-analyst report contains "Insufficient data" or "Request TRACE_FULL", ask the user for the full trace and do not continue the chain until it is provided. Otherwise proceed.

3. **Verification queries** — check if the trace-analyst report contains a `## Verification queries for explorer` section with VQ-items:
   - **If VQ-items exist**: run **onec-code-explorer** with the targeted verification prompt (template «Explorer — верификация гипотез trace-analyst» from `1c-agent-patterns/explorer.md`). Pass: path to the trace-analyst report, the VQ list, and modules_hint from the report. Explorer returns a confirms/refutes/inconclusive answer for each VQ with code citations.
   - **If no VQ-items**: skip to step 4.

4. **Merge VQ answers into RCA** (only if step 3 ran explorer):
   - For each VQ where explorer answered **confirms** — move the corresponding hypothesis to Verified facts (with explorer's code citation).
   - For each VQ where explorer answered **refutes** — record as "Refuted hypothesis" (valuable context for architect).
   - For each VQ where explorer answered **inconclusive** — keep as Hypothesis with updated verification plan.
   - Result: updated Verified facts / Hypotheses / Refuted hypotheses for the final RCA.

5. **Full code exploration** (conditional) — run **onec-code-explorer** for the full call chain only if:
   - Step 3 did NOT already call explorer (no VQ-items existed), OR
   - After step 4 merge, significant unresolved hypotheses remain, OR
   - The trace involves 3+ modules and a full call chain reconstruction is needed.
   - Task: restore the full call chain in code (trace shows "what", code shows "why").
   - Pass: list of modules and line numbers from the trace-analyst summary + VQ verification results (if available), focus on extension files (e.g. `src/**/cfe/**`).
   - If step 3 already called explorer and all hypotheses are resolved — skip this step.

6. If trace-analyst or explorer identified an **architectural issue** (e.g. write conflicts, transaction problems, data flow issues):
   - **Run onec-code-architect**:
     - Task: propose fix options (transaction boundaries, write order, or skip write in handler).
     - Pass: RCA from trace-analyst/explorer, paths to relevant files.

7. Merge results into step 6 (Root cause): Verified facts and Hypotheses come from trace-analyst output, enriched by VQ verification (step 4) and supplemented by explorer (step 5).

**3.5.8. Сохранение отчётов субагентов.** После шагов 3.5.1–3.5.7 сохранить полные отчёты аналитических агентов по правилу `preserve-subagent-reports.mdc`: отчёт onec-trace-analyst — в `openspec/changes/<id>/reports/trace-analysis-YYYY-MM-DD.md` (или `temp/reports/` при отсутствии change); отчёт onec-code-explorer — в `reports/exploration-YYYY-MM-DD.md` или `reports/resolved-contract-*-YYYY-MM-DD.md` при investigation loop; отчёт onec-code-architect — в `reports/architecture-YYYY-MM-DD.md`. В design.md / debug.md ссылаться на полный отчёт (путь к файлу), не дублировать выжимку.

7.5. **Anti-pattern detection (optional).**
   After RCA is established, evaluate whether the root cause represents a **generalizable anti-pattern** that could recur in other code.
   Three criteria (ALL must be met):
   - **Повторяемость**: паттерн может встретиться в другом коде (не уникален для данного модуля/сценария).
   - **Пробел в ревью**: текущий ревьювер НЕ обнаружит этот паттерн (нет AP-NNN в `.cursor/rules/bsl-antipatterns.mdc`).
   - **Обобщаемость**: можно сформулировать абстрактный принцип (без привязки к конкретному change/модулю).

   If all three criteria are met:
   - Inform user: «Ситуация [описание] тянет на антипаттерн — зарегистрируем? [Да / Нет]»
   - User may also directly ask: «зарегистрируй антипаттерн».
   - On confirmation:
     1. Grep `.cursor/docs/antipatterns/bsl-antipatterns.md` for existing similar AP.
     2. Determine next AP-NNN ID.
     3. Formulate generalized principle (abstract, not tied to current incident).
     4. Add full card to `.cursor/docs/antipatterns/bsl-antipatterns.md`.
     5. Add index entry to `.cursor/rules/bsl-antipatterns.mdc`.
   - On rejection: note in debug.md (section «Anti-pattern considered but not registered»).

### 4) Investigate in codebase (read-only)

If step 3.5 was already run (trace analyzed by subagents), use their output as the basis; do not duplicate manual search.

**Context Strategy:** если исследование требует анализа 3+ файлов или файлов данных (XML, HTML, CSV) — применить стратегию из `.cursor/skills/context-strategy/SKILL.md` (инвентаризация → decision matrix → субагенты → синтез).

Use the trace/remark to identify likely modules and entry points:
- Search for procedure/function names from the trace
- Search for key phrases from exception messages
- Narrow down to the modules changed by this change (prefer extension `src/**/cfe/**` first)

Read the relevant files and map an execution path:
- "What calls what"
- Inputs/outputs that lead to the failure
- Conditions that differ from the expected behavior in design/tasks

### 5) Metadata validation (mandatory)

Before concluding anything that mentions metadata names or types, validate against XML in `src/`.

**Applies to any of these forms:**
- `Перечисления.X.Y`
- `РегистрСведений.X` / `РегистрНакопления.X` / `РегистрБухгалтерии.X` / …
- `Справочники.X`, `Документы.X`, `БизнесПроцессы.X`, `Задачи.X`, …
- `Тип("СправочникСсылка.X")`, `Тип("ПеречислениеСсылка.X")`, etc.
- `Метаданные.<Type>.<Name>` usage

**How to validate (unified paths):**
- Search under `src/` recursively for the relevant XML:
  - Enum: `src/**/Enums/<X>.xml` and its `EnumValue` names
  - Register: `src/**/InformationRegisters/<X>.xml` and its dimensions/resources/attributes
  - Catalog/Document/BP/Task/etc.: `src/**/<Kind>/<X>.xml`
  - If unsure of kind: look in `src/**/Configuration.xml` for `<Enum>`, `<InformationRegister>`, `<Catalog>`, etc.

**If not found:**
1. Search alternatives (similar names in the same folder and in `Configuration.xml`)
2. Show the user available candidates (a short list)
3. **STOP**: do not use an unverified name in conclusions or plan

### 6) Root cause and conclusions

Write a **structured RCA** with two mandatory sections. Never present a hypothesis as a verified fact.

**## Verified facts**

- What is **definitely established** from the evidence (trace, log, code).
- Each fact must have a **concrete reference**: trace line number, log entry, file:line in code.
- Example: "In trace line 15540, M15:47, `pavIU_ИсполнительИсключен` is called; trace shows branch 'excluded' at M15:50."

**## Hypotheses**

- What is **assumed** but not directly proven (e.g. "algorithm returns exclude because of missing executor context").
- For each hypothesis, add a **verification plan**: what to log, which scenario to run, what to check in the trace — or mark "hypothesis-based fix" with a follow-up verification task.

Keep the RCA compact; the split ensures apply/debug can later enforce the verified cause gate (see `verified-cause-gate.mdc`).

### 5.5) Architect Gate (после RCA, до фикса)

Перед переходом к плану фикса проверить триггеры из `.cursor/rules/architect-gate.mdc` (объективные маркеры, семантические, структурные). В контексте debug автоматически срабатывают: **bug fix** (объективный маркер); наличие отчётов trace-analyst или explorer в сессии (объективный маркер «вызывался trace-analyst/explorer»). Дополнительно проверить остальные триггеры (перехват базовой процедуры, новый объект в design, несколько точек реализации, фикс меняет UX-сценарий и т.д.).

- **При срабатывании любого триггера:** вызвать **onec-code-architect** с брифом (RCA, корневая причина, предложенный подход, Fix Quality чеклист). Использовать шаблон «Architect — fix quality review» из `1c-agent-patterns/architect.md`. Результат сохранить в `reports/architecture-debug-YYYY-MM-DD.md` по `preserve-subagent-reports.mdc`. Учесть рекомендации в плане фикса (шаг 7). **К шагу 7 переходить ТОЛЬКО после получения отчёта архитектора.**
- **Исключение:** пользователь явно пишет «пропустить архитектора» → в debug.md секция «Architect Gate» с записью «Пользователь отклонил. Причина: …» → затем шаг 7. Оркестратор **НЕ** принимает решение «пропустить» самостоятельно (не допускается обоснование «фикс точечный»).
- **Триггеры не сработали** — перейти к шагу 7 без вызова архитектора.

Исключения по architect-gate.mdc (Gate не срабатывает): Light Mode без семантических триггеров и без изменения UX-сценария, рефакторинг без изменения поведения, опечатки. В debug типичный сценарий — bug fix, поэтому проверка обязательна.

### 7) Plan and capture tasks (default: artifacts + hand off)

**7a. Plan** — предложить план в двух слоях:

**Code steps** (конкретные, мелкие): какие файлы менять, что менять, что логировать / граничные случаи.

**Artifact steps:**
- Если RCA показывает **постановочный дефект** (причина в `proposal.md` / `design.md` / `tasks.md` / spec, а не в реализации), не превращать его сразу в code fix. Сформировать раздел `Scope/design rework` и предложить:
  `/opsx:extend <change> --from-debug openspec/changes/<change>/debug.md`
  Затем: `/opsx:verify <change>` → `/opsx:apply <change>` при необходимости.
- Добавить или уточнить задачи в `openspec/changes/<change>/tasks.md` **с привязкой к срезу**:
  - **Если ЗНИ в slice mode** (есть `# Срез S<N>`): **перед записью в tasks.md** выполнить **mechanical placement gate** (источник истины — `.cursor/rules/vertical-slices.mdc`, раздел **«ИНВАРИАНТ: Defect placement»** и decision tree):
    1. Определить целевой срез воспроизведения `S<K>` (по сценарию / spec / явной директиве пользователя; при неоднозначности — AskQuestion).
    2. Grep `tasks.md`: строка приёмки `S<K>.T<M>` — чекбокс **`[x]`** или **`[ ]`**?
       - **`[ ]`** (срез не принят, включая `awaiting-acceptance` / «не принят») → **только** inside-slice: добавить `S<K>.<M+1>` **перед** `S<K>.T<M>`. **Запрещено** создавать `# Срез S<N+1>` для этого дефекта, если фикс **не** cross-slice (не затрагивает ≥2 среза по файлам/сценариям/spec).
       - **`[x]`** (срез принят) → **fix-срез** `S<N+1>` с отдельным `S<N+1>.T<M>` по инварианту в `vertical-slices.mdc` (frozen-slice).
    3. Cross-slice (≥2 среза) → новый срез `S<N+1>` **только** с метаданной `**Причина fix-среза:** cross-slice` и отдельной приёмкой; иначе — не оправдан.
    4. Дефект вне любого существующего среза → новый срез только после AskQuestion (имя, сценарий, приёмка) и проверки, что это **не** случай ошибочного определения `S<K>`.
  - **Запрещено:** добавлять fix-задачи «в следующий по порядку срез» или плодить `S<N+1>` без прохождения чеклиста **«Fix-срез разрешён?»** из `vertical-slices.mdc`.
  - **Если ЗНИ в legacy mode** (нет `# Срез`): вставлять в соответствующую секцию или «7. Рефакторинг и качество» как «7.x Исправить: …»; при наличии устаревших `<!-- phase-gate` маркеров — рекомендовать `/opsx:migrate-slices <name>` отдельным сообщением (но **не** автоматически).
- При выявленном пробеле в design — краткая заметка в `design.md` (для slice-mode — обязательно обновить `## Slices`, если меняется состав срезов).
- **Hypothesis gate:** если корневая причина в **## Hypotheses**: первая задача плана — верификация (логирование, воспроизведение, проверка трассы) **или** явная пометка «hypothesis-based fix» + задача follow-up верификации после фикса. Если корневая причина в **## Verified facts** — переходить к задачам фикса.
- **Развилки в чате:** если перед пользователем несколько взаимоисключающих путей (верификация гипотезы vs правка «на допущении», постановочный vs кодовый фикс и т.д.), описать каждую развилку **обычным языком**: короткий заголовок проблемы, **логичный путь** со ссылкой на артефакт, буллеты-альтернативы **одной строкой**, блок **«что зацепит»** — по образцу `openspec-verify-change/templates/card-decision.md` и тонкому чату verify (`templates/chat-summary.md`). Кодов ответа (`1a`, `<N>b`) **не использовать**; пользователь отвечает свободным текстом. Подробности RCA остаются в `debug.md` и отчётах.

**Slice Gate Decisions log:** если фикс — следствие неуспешной приёмки среза в `/opsx:apply` (пользователь выбрал `[3] Дефект в предыдущем срезе` или `[2] Не принят`), эта debug-сессия должна оставить запись в `debug.md` секции `## Slice Gate Decisions`: дата, ID среза, решение (`inside-slice rework` при вставке задач в `S<K>` до подписанной приёмки, либо иное по словарю `vertical-slices.mdc`), RCA, ссылки на отчёты trace-analyst/explorer/architect, новые задачи фикса.

**Default:** после плана выполнить 7b (артефакты) и hand off. Не спрашивать подтверждение перед правкой артефактов, если пользователь не просил «plan only». **Если пользователь явно:** "plan only" — не править артефакты.

---

**7b. Update artifacts.** Обновить `tasks.md` и при необходимости `design.md`. Формирование задач в tasks.md:
- Каждая задача — одна конкретная правка (файл, процедура, что менять, почему — ссылка на RCA).
- В тексте задачи указать корневую причину (Verified / Hypothesis), чтобы apply при вызове writer мог передать Root Cause Context.
- Slice-aware вставка: применяются правила из 7a и **инвариант** из `.cursor/rules/vertical-slices.mdc` (slice mode → inside-slice по умолчанию; fix-срез только по инварианту; legacy → плоский список). Для slice mode каждая новая задача получает ID `S<N>.<M>` **в рамках выбранного среза** (не «следующего свободного номера среза»).
- Задачи на верификацию гипотезы или follow-up — по Hypothesis gate в 7a.

### 8) Hand off

После обновления артефактов:
- Предложить: `/opsx:apply <change>` для реализации задач. При наличии hypothesis-based задач — предупредить о необходимости верификации (или задачи follow-up в tasks.md).
- Если debug зафиксировал постановочный дефект или необходимость пересмотреть scope/design, вместо прямого apply (или перед ним) предложить:
  `/opsx:extend <change> --from-debug openspec/changes/<change>/debug.md`
  Extend покажет обязательный бриф, обновит ЗНИ и вернёт в `/opsx:verify`.

---

## Интеграция

- **Output style:** `.cursor/docs/opsx-output-style.md` — бриф отладки выводится по шаблону **T-BRIEF**; перед отправкой — self-check-5 (§7).
- **command-skill-gate.mdc:** первый и единственный инструмент в первом батче при вызове команды — Read этого скилла.
- **command-session-persistence.mdc:** протокол debug действует на **каждом** ходе сессии (Entry Protocol, Per-turn Gate, шаги 3–8). Выход из протокола — только по явному завершению пользователем или смене команды.
- **1c-agent-delegation.mdc (APPLY GATE):** debug не реализует код — реализация через `/opsx:apply`. Writer и reviewer вызываются только в контексте apply.
- **preserve-subagent-reports.mdc:** полные отчёты trace-analyst, explorer, architect сохранять в `openspec/changes/<id>/reports/` (или `temp/reports/`); в design/debug ссылаться на файл отчёта.
- **architect-gate.mdc:** единый источник триггеров для шага 5.5 (Architect Gate после RCA).
- **project-paths.mdc:** пути к cf и cfe брать из `openspec/project.md`; передавать субагентам (explorer, trace-analyst, architect) в промпте.
- **context-strategy-gate.mdc:** при исследовании 3+ файлов или файлов данных — загружать `.cursor/skills/context-strategy/SKILL.md` и следовать Entry Protocol до чтения файлов.
- **1c-error-analysis.mdc:** не читать трассу вручную; делегировать onec-trace-analyst с путём и брифом. TRACE_FULL по запросу агента — запросить у пользователя.
- **verified-cause-gate.mdc:** перед фиксом — разделение Verified facts / Hypotheses, цепочка «Почему», корневая причина; hypothesis-based fix только с задачей верификации или follow-up.
- **vertical-slices.mdc:** placement fix-задач в `tasks.md` — только по **ИНВАРИАНТ: Defect placement** и mechanical gate шага 7a (не плодить `# Срез` до подписанной приёмки без cross-slice / frozen-slice).

---

