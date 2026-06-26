---
name: openspec-extend-change
description: Controlled scope extension for an existing OpenSpec change. user-extend (brief + confirm) or repair-from-verify (internal, silent).
license: MIT
compatibility: Does not call writer/reviewer and never edits BSL/XML metadata. May delegate read-only analysis to onec-code-architect / onec-code-explorer when gates require it.
metadata:
  author: project
  version: "1.1"
---

Контролируемо расширить или пересмотреть scope существующего OpenSpec change по новому требованию, отчёту ревью, debug/RCA, verify-отчёту, архитектурному отчёту или итогу `/opsx:explore`.

**Extend правит только артефакты change**: `proposal.md`, `design.md`, `specs/**`, `tasks.md`, `debug.md`, `reports/**`. Код BSL, XML метаданных и реализация остаются за `/opsx:apply`.

## Режимы (user-extend vs repair-from-verify)

| Режим | Триггер | Бриф | Сообщение в чат |
|-------|---------|------|-----------------|
| **user-extend** | `/opsx:extend` от пользователя; `--from-verify` **после** decision 3a | T-BRIEF + «Подтвердить?» | Handoff на языке эффекта + hint `/opsx:verify <name>` |
| **repair-from-verify** | internal из verify Repair Loop (`apply_repairs_from_report`) | **SKIP** | **нет** — вызывающий verify продолжает loop |

**repair-from-verify:** без Entry Protocol, без lite-брифа, без «Подтвердить?»; сразу §6 Artifact update rules по remediation из отчёта verify; запись в `debug.md`; **0** строк в чат.

**user-extend с `--from-verify` после decision:** бриф **B2** (delta extend) + «Подтвердить?» → после правок handoff на языке эффекта.

Chat Surface Contract — §2.6 `opsx-output-style.md`.

---

## Input (free-form)

Пользователь может передать:

- `<change-name>` — имя change. Если не указано, определить по `openspec list --json`; при неоднозначности — `AskQuestion`.
- Текст нового требования / пересмотра / замечания.
- Ссылки на файлы в любом виде:
  - `@path/to/file.md`
  - относительный или абсолютный путь
  - `--from-review <path>` — отчёт `/review`
  - `--from-debug <path>` — `debug.md` или RCA-отчёт
  - `--from-verify <path>` — отчёт `/opsx:verify`
   - `--from-architecture <path>` — отчёт архитектора
  - `--from-report <path>` — итог `/opsx:explore`: `temp/reports/<тип>-YYYY-MM-DD-<slug>.md` (полный отчёт trace-analyst / explorer / architect) или `temp/explore-handoff-*.md` (опциональный handoff с блоком `## Постановка ЗНИ`)
  - `--from-explore <path>` — **legacy-источники, только по явной ссылке пользователя**: `temp/explore-summary-*.md`, `openspec/sessions/<slug>/analysis.md` (RCA, рецепт, fix-задачи). Новые файлы этих форматов не создаются
  - `--code-sync` — штатная синхронизация OpenSpec-артефактов с фактическим кодом после ручного упрощения, writer/apply или Code-Truth Gate (`phantom-symbol`, устаревшие имена процедур, drift design/tasks/debug).

Если текст требования отсутствует, но указан файл — извлечь намерение из файла и показать в брифе. Если намерение неоднозначно — спросить пользователя после брифа, до правок.

**Output style:**
Стиль вывода в чат (брифы, сводки, развилки) — см. `.cursor/docs/opsx-output-style.md` §2.5 «Человеческий слой».

---

## Entry Protocol (MANDATORY для user-extend)

**repair-from-verify:** Entry Protocol **пропускается** — сразу §6 Artifact update rules.

Первый шаг команды **user-extend**:

1. Прочитать этот `SKILL.md` (Command → Skill, `session-discipline.mdc`).
2. Определить change:
   - если `<change-name>` указан — использовать его;
   - иначе выполнить `openspec list --json` и выбрать активный / спросить пользователя.
3. Выполнить `openspec instructions apply --change "<name>" --json`.
4. Прочитать `openspec/project.md` и артефакты change из `contextFiles`: `proposal.md`, `design.md`, `tasks.md`, `specs/**` при наличии; для расширения scope также прочитать `debug.md` (если есть) — нужен для счётчика Scope Coherence Audit и записей `## Extend`.
5. Прочитать только явно переданные source-файлы (`--from-*`, `@path`, пути в запросе). Трассы — через `/opsx:explore` (профиль bug). `--from-report` принимает (приоритет): `temp/reports/<тип>-YYYY-MM-DD-<slug>.md` (отчёт `Task` из Ultra-Lite explore), `temp/explore-handoff-*.md` (handoff с блоком `## Постановка ЗНИ`); legacy-файлы — только через `--from-explore` по явной ссылке пользователя. Если указан `--code-sync`, source = артефакты change + `debug.md` + отчёты `reports/**` + результаты Code-Truth Gate; чтение BSL/XML до брифа всё равно запрещено.
5a. **KB Discovery (internal):** прочитать `openspec/knowledge/_index.yaml` и при необходимости `_taxonomy.yaml` и выбранные KB `.md` — по алгоритму Entry Protocol шаг 1.5 `.cursor/skills/openspec-explore/SKILL.md` (anchor-paths из путей в уже прочитанных артефактах и source-файлах; бюджет Top-10). Результат — **не в чат**, только для промптов architect/explorer после «да».
5b. **Brief Depth Classifier** — `.cursor/docs/opsx-output-style.md` §5.1 (B1 по умолчанию; B2 при drift/decision/`--code-sync`/`--from-review` >3 findings/`--from-verify` после decision; **никогда B3**).
6. Сформировать и показать **бриф extend** (B1 или B2) по шаблону ниже.
7. **END TURN.** До подтверждения пользователя запрещены: запись артефактов, вызовы writer/reviewer, вызовы architect/explorer, чтение BSL/XML для анализа логики.

Разрешённые действия до брифа: чтение этого скилла, `openspec list --json`, `openspec instructions apply --json`, чтение `project.md`, чтение артефактов change, чтение явно переданных markdown/text/source-report файлов, чтение `openspec/knowledge/_index.yaml` / `_taxonomy.yaml` и выбранных KB `.md` (шаг 5a, internal).

### Шаблон брифа (T-BRIEF)

SSOT: `.cursor/docs/templates/brief-card.md` (классификатор — `opsx-output-style.md` §5.1). Заголовок: `Бриф: <change-name>`. Файлы `temp/briefs/*.md` не создаются.

**Чат-бриф extend = B1 или B2** (классификатор шаг 5b). Слоты:

| Уровень | Слоты в чате |
|---------|--------------|
| **B1** | **Изменение** (+ опц. **Затронутое**) + **Подтвердить?** |
| **B2** | **Изменение** + **Почему выбор** + **Варианты** (нумерованный список) + **Подтвердить?** (ответ по номеру; голый «да» не принимается) |

Бюджет содержания — `brief-card.md` (≤6 слотов, ~30 сек).

**Запрещено в чат-брифе extend:** `Что понял`; слот **План** с перечислением артефактов; `Как буду искать`; `Бриф для исследования`; `Drift-check:`; `proposal.md / design.md` как план шагов; варианты **A/B** буквами.

**Внутренние блоки (до брифа, в чат НЕ попадают — в `debug.md` § Extend после подтверждения):**

```markdown
# (internal, не в чат)
Предлагаемое изменение артефактов: proposal.md / design.md / specs/** / tasks.md / debug.md — что и где.
Соответствие исходному scope:
- Усиливает пункт Why: …
- Затрагивает Non-Goals: yes / no
- Меняет Behavior Contract: yes / no
- Отменяет архивный инвариант: yes / no
- Drift-check: pass / drift-warning / scope-violation

План после подтверждения (internal):
1. Уточнить неоднозначности при необходимости
2. Проверить гейты архитектуры и факты в коде
3. При --code-sync — делегировать исследователя кода → reports/exploration-code-sync-YYYY-MM-DD.md
4. Обновить артефакты change
5. Hint /opsx:verify <name>
```

Если extend вызван **internal repair-from-verify** — **B0**, бриф не показывается, правки сразу по §6.

Если extend вызван **user-extend с `--from-verify` после decision** — **B2**: только delta extend («что допишем в постановку по вашему ответу»), **не** повтор verify-карточки.

**Матрица входов → уровень брифа:**

| Флаг / вход | Уровень |
|-------------|---------|
| `--code-sync` | B2 (всегда) |
| `--from-review` с >3 findings | B2 (сводка + приоритет в развилке A/B) |
| `--from-verify` после decision | B2 |
| Конкретное уточнение, drift pass | B1 |
| `drift-warning` / `scope-violation` | B2 |

Self-check перед выводом: уровень B1/B2 по классификатору; слоты по `brief-card.md`; UX-поля без `S<N>.T<M>` / `D<N>`; блок **«Соответствие исходному scope»** и **План после подтверждения** заполнены **внутренне**, в чат не выводятся; `rg`-HALT: нет `Как буду искать`, `Что понял`, `Бриф для исследования`.

**Развилки в чате после брифа:** любые варианты выбора до или после подтверждения (AskQuestion, неоднозначный `Drift-check` и т.п.) — слот **Варианты** нумерованным списком `1.` `2.` `3.` (как в entry-брифе B2), приглашение ответить **номером варианта**. Коды `<N>a` / `<N>b` / `A`/`B` в чате запрещены. Варианты **в чате**, не только в длинном теле брифа.

### Соответствие исходному scope — критерии заполнения (оркестратор)

Источник правды: текущие `proposal.md` (`## Why`, `## What Changes`) и `design.md` (`## Behavior Contract`, `## Goals / Non-Goals`).

**Затрагивает Non-Goals: yes** — предлагаемое изменение явно описывает или подразумевает действие, перечисленное или запрещённое в `## Non-Goals` design.md.

**Меняет Behavior Contract: yes** — предлагается добавить новый пункт в `## Behavior Contract`, удалить существующий пункт или изменить формулировку пункта так, что меняется наблюдаемое поведение (не просто уточнение терминологии).

**Drift-check:**
- `pass` — ответ **no** по Non-Goals и Behavior Contract **и** «Отменяет архивный инвариант: **no**» **и** «Усиливает пункт Why» указывает конкретный пункт `## Why` (не «не относится напрямую»).
- `drift-warning` — `Затрагивает Non-Goals: yes` **или** `Меняет Behavior Contract: yes` **или** «Усиливает пункт Why: не относится напрямую» **или** осознанное расширение при сохранении архивных контрактов (Behavior Contract меняется, но инвариант не отменяется — пояснить в брифе).
- `scope-violation` — любое из: оркестратор не находит ни одного пункта Why, который усиливается предлагаемым изменением, **и** при этом хотя бы одно из полей Non-Goals / Behavior Contract = **yes**; **или** `Отменяет архивный инвариант: **yes**` при отсутствии заполненной секции `## Blast Radius` в `design.md` (нет таблицы контракт / источник / бизнес-эффект / альтернативы / обоснование). В этом случае **не** использовать `drift-warning` — только `scope-violation`.

Если классификация неоднозначна до подтверждения брифа — `AskQuestion` с тремя вариантами итогового `Drift-check`: `pass` / `drift-warning` / `scope-violation`; результат отразить в брифе.

---

## Per-turn Delegation Gate

На каждом follow-up ходе:

1. Если запрос требует обследования кода, трассировки вызовов, проверки метаданных или анализа 3+ модулей — не читать `.bsl`/XML самостоятельно.
2. Сформировать бриф и делегировать:
   - `onec-code-explorer` — для обследования кода;
   - `onec-code-architect` — для выбора/пересмотра подхода;
   - `onec-trace-analyst` не используется в extend; трассы перенаправлять в `/opsx:explore` (профиль bug).
3. До делегирования допустимо читать только OpenSpec-артефакты и явно переданные отчёты.

---

## Source File Classification

После чтения явно переданных файлов классифицировать источник:

| Source | Признаки | Что извлекать |
|--------|----------|---------------|
| `review` | `review-*.md`, `code-review-*.md`, секции `Findings`, `Action`, `Type`, `Severity` | findings, disposition, `ARCHITECTURE`, `MUST_FIX`, противоречия design, пути/anchors |
| `debug` | `debug.md`, `trace-analysis-*`, RCA, `Verified facts`, `Hypotheses`, `Root cause` | verified facts, hypotheses, root cause, target slice |
| `verify` | `verification-*.md`, классы Repair Loop (`decision` / `artifact-hygiene`), `CRITICAL/WARNING/SUGGESTION` (legacy-маркер `Phase B`) | decision cards, scope/design/task issues, recommended remediation |
| `architecture` | `architecture-*.md`, `architecture-review-*` | рекомендуемые изменения design/spec/tasks/ADR, alternatives |
| `explore` | legacy-источники только по явной ссылке пользователя (`explore-summary-*`, `Explore Summary`, `Decisions`, `Open questions`) | сформулированные требования, slice hints, unresolved questions |
| `report` | `temp/reports/<тип>-*.md` (trace-analyst / explorer / architect), `temp/explore-handoff-*.md` (блок `## Постановка ЗНИ`) | root cause, verified facts, рецепт, задачи для `debug.md` и `tasks.md`, slice placement |
| `code-sync` | флаг `--code-sync`, `phantom-symbol`, расхождения `design/tasks/debug` с фактическим кодом | фактические символы, устаревшие рецепты, какие артефакты догнать до кода |
| `other` | markdown/text без явных маркеров | факты и open questions; если непонятно — AskQuestion |

Если файл содержит несколько типов (например raw review + reasoning appendix) — извлечь все, но в брифе отделить факты от рекомендаций.

---

## Workflow Steps

### 1. Resolve change

- Проверить, что `openspec/changes/<name>/` существует и не находится в `archive/`.
- Если change не найден — спросить пользователя или предложить `/opsx:new <name>`.
- Вывести `Using change: <name>` и способ переопределения.

### 2. Load context

- Прочитать `proposal.md`, `design.md`, `tasks.md`, `specs/**`, `debug.md` (если существует).
- Прочитать `openspec/project.md` и извлечь пути cf/cfe (для возможного architect/explorer).
- Прочитать явно переданные source-файлы.
- Не читать BSL/XML для анализа логики на этом шаге.

### 3. Prepare and show brief

- Сопоставить вход с текущим scope change.
- Определить тип изменения:
  - новое требование;
  - уточнение существующего требования;
  - пересмотр архитектурного решения;
  - постановочный дефект по результатам review/debug/verify;
  - перенос open question в решение.
- Для `--code-sync`: перечислить потенциальные drift-точки без чтения BSL/XML: имена процедур из `tasks.md`/`debug.md`, отчёты writer/reviewer/explorer, файлы `src/**`, которые потребуется прочитать через `onec-code-explorer` после подтверждения.
- Заполнить блок **«Соответствие исходному scope»** по критериям из секции под шаблоном T-BRIEF (сопоставить предлагаемые изменения с `## Why`, `## Non-Goals`, `## Behavior Contract`).
- Показать бриф и остановиться до подтверждения.

### 4. Ambiguity Gate

После подтверждения, но до правок, задать `AskQuestion`, если неясно:

- менять существующий `Requirement` или добавить новый;
- inside-slice, fix-срез или новый срез;
- принимать рекомендацию review/architecture или оставить как rejected/deferred;
- требуются ли изменения specs;
- считать source finding дефектом кода, постановки или ложным срабатыванием.

### 5. Architect Gate

Проверить `.cursor/rules/architect-gate.mdc`.

#### 5a. Scope Coherence Audit (режим `scope-coherence-audit`)

Цель — обнаружить расползание ЗНИ (scope drift) до дорогого `/opsx:verify`. Отчёт: `reports/architecture-extend-coherence-YYYY-MM-DD.md`. Шаблон промпта: `.cursor/skills/1c-agent-patterns/architect.md`, секция **Architect — scope coherence audit (extend)**.

**Триггер 1 (семантический, из брифа):** в блоке «Соответствие исходному scope» зафиксирован `Drift-check: drift-warning` или `Drift-check: scope-violation`.

**Триггер 2 (объективный, счётчик Extend без архитектора):**

1. Выполнить по `openspec/changes/<name>/debug.md`:
   ```bash
   rg -c "Architect Gate:\\s*(не вызывался|не требовался|declined|—)" debug.md
   ```
   Обозначить результат как **M**.

2. Выполнить `Glob` по `openspec/changes/<name>/reports/architecture-extend-coherence-*.md`.

3. Триггер 2 срабатывает, если **M ≥ 3** и выполняется одно из условий:
   - файлов `architecture-extend-coherence-*.md` **нет**;
   - **или** дата из заголовка последней секции `## Extend — YYYY-MM-DD` в `debug.md` (парсинг ГГГГ-ММ-ДД из строки заголовка) **строго новее** даты из имени новейшего файла `architecture-extend-coherence-*.md` (по дате в суффиксе имени).

Примечание: **M** — число строк в `debug.md`, где поле `Architect Gate:` совпадает с шаблоном «Extend без архитектора» (маркеры выше); это приближение к числу секций `## Extend —`, после которых архитектор не вызывался.

**Если сработал Триггер 1 или Триггер 2:**

**Грейс-исключение (антидубль):** если в ответ на **этот же** подтверждённый бриф уже записан файл `reports/architecture-extend-coherence-YYYY-MM-DD.md`, повторный Scope Coherence Audit до завершения handoff не вызывать. Следующий прогон `/opsx:extend` — новый бриф, грейс не действует.

**Если грейс не применим:**

1. Выполнить ADR Discovery и KB Discovery (как ниже для обычного architect).
2. Вызвать `Task(subagent_type="onec-code-architect")` с `mode=scope-coherence-audit` и шаблоном из `1c-agent-patterns/architect.md`. Оркестратор передаёт блок «Соответствие исходному scope» из брифа, полные тексты артефактов, при наличии — фрагмент исходного `proposal.md` из git (`git show <hash>:openspec/changes/<name>/proposal.md`, где `<hash>` — коммит создания change или первый коммит с каталогом change; если недоступно — явно указать в промпте «исторический proposal недоступен»).
3. Сохранить полный отчёт в `openspec/changes/<name>/reports/architecture-extend-coherence-YYYY-MM-DD.md`.
4. Добавить в `debug.md` запись:
   ```markdown
   ## Extend Coherence Audit — YYYY-MM-DD

   - Триггер: semantic | counter | both
   - Drift-check из брифа: pass | drift-warning | scope-violation
   - Вердикт архитектора: coherent | drift-warning | scope-violation
   - Отчёт: `reports/architecture-extend-coherence-YYYY-MM-DD.md`
   - Решение пользователя: accepted recommendations | partial | rejected — <кратко>
   ```
   Поле «Решение пользователя» заполняется после `AskQuestion`, если вердикт архитектора требует решения.

**Если вердикт архитектора в отчёте — `scope-violation`:** до обновления остальных артефактов остановиться и через `AskQuestion` предложить: (a) принять рекомендации архитектора (например отделить часть в новую ЗНИ через `/opsx:new`), (b) отклонить и зафиксировать риск в `debug.md`, (c) свернуть extend без правок.

Simplicity Check для режима `scope-coherence-audit` **не требуется** (см. `.cursor/rules/architect-gate.mdc`).

---

Architect обязателен также по общим правилам Gate (ниже). Обычный отчёт расширения без coherence сохранять в `reports/architecture-extend-YYYY-MM-DD.md`, если вызывается архитектор по классическим триггерам **после** или **вместо** coherence — не дублировать два полных прохода без необходимости; при одновременном срабатывании 5a и классических триггеров достаточно **одного** вызова с приоритетом шаблона **scope coherence audit**, если пользователь не запросил явно отдельное архитектурное решение по API/перехватам.

Architect обязателен, если:

- source содержит `ARCHITECTURE` finding;
- предлагается изменить точку расширения (`&Перед`, `&После`, `&Вместо`, `&ИзменениеИКонтроль`);
- меняется контракт/export/API;
- требуется новый объект метаданных;
- есть несколько альтернатив реализации без выбранного решения;
- пересматривается уже зафиксированный `design.md` approach;
- решение затрагивает 2+ файла или UX-сценарий.

Для `--code-sync` architect **не обязателен**, если не меняется Behavior Contract / ADR / точка расширения, а артефакты только догоняют фактический код. Если фактический код вводит новый подход (новая точка перехвата, новый контракт, сосуществование расширений), проверить обычные триггеры выше.

Перед вызовом architect:

- выполнить ADR Discovery по `openspec/adrs/`;
- выполнить KB Discovery по `openspec/knowledge/_index.yaml` / `_taxonomy.yaml`, если файлы есть;
- передать `## Existing Knowledge` по правилам проекта.

Если архитектор вызывался **только** по п. 5a (Scope Coherence Audit), полный отчёт уже сохранён в `reports/architecture-extend-coherence-YYYY-MM-DD.md`. Иначе сохранить полный отчёт расширения в `openspec/changes/<name>/reports/architecture-extend-YYYY-MM-DD.md`.

Если пользователь явно отказался от architect — записать отказ в `debug.md` / `reports/extend-decision-*.md` и продолжать только если правила допускают отказ.

### 6. Artifact update rules

Порядок правок:

1. `proposal.md` — если меняется scope / Why / What Changes / Impact. При существенной смене доменной темы — **опционально** один вопрос: обновить `comment_suffix` (domain_label) в `## Metadata (comment markers)`? Не блокер extend.
2. `specs/**/spec.md` — delta spec (`ADDED`, `MODIFIED`, `REMOVED`), минимум один Scenario на Requirement.
3. `design.md` — `Existing Mechanisms`, `Design Rationale`, `Decisions`, `Slices`, `Risks`, `Open Questions`.
4. `tasks.md` — slice-aware вставка:
   - непринятый срез → inside-slice перед `S<N>.accept` (или legacy `S<N>.T<M>`);
   - принятый срез → fix-срез;
   - новая функциональность → новый срез или расширение непринятого, только после Ambiguity Gate;
   - legacy → подходящая секция или «Рефакторинг и качество».
5. `debug.md` — секция `## Extend — YYYY-MM-DD`:
   - источник (`--from-review`, `--from-debug`, ...);
   - что добавлено/изменено;
   - disposition по findings: `accepted`, `rejected`, `deferred`;
   - ссылки на отчёты architect/explorer;
   - следующий шаг.

### 6a. Verify decision ledger (после user-extend `--from-verify` по decision)

**Триггер:** user-extend с `--from-verify` после decision 3a (ответ пользователя на развилку verify).

**Действия (до hint `/opsx:verify`, без ожидания следующего verification YAML):**

1. Добавить/обновить `debug.md` § **`## Verify decision ledger`** (YAML-блок, agent-only):
   ```yaml
   closed_decisions:
     - id: <snake_case>
       summary: "<проза>"
       closed_at: "YYYY-MM-DD"
       source: verify-user-answer
   open_decision_id: null
   decision_round: <N+1>
   verify_depth: incremental
   assumptions_accepted: []
   ```
2. Добавить зеркало в `design.md` § **`## Решения verify (зафиксировано)`** — 1–2 строки прозой **без** `id:` (не дублировать параграфы из `## Decisions`).
3. Удалить из `open_known_questions` (если велся в debug или последнем verification snapshot) темы, закрытые этим decision.
4. Internal mapping: парсинг ответа пользователя → `decision_id` + вариант; при неоднозначности — один уточняющий вопрос **прозой**.

**repair-from-verify:** ledger **не** меняет `decision_round`; только технические правки design/tasks.

### 6c. Repair-from-verify: slice acceptance remediation

**Триггер:** internal repair-from-verify; в `quality-control-*.md` или verification report есть блоки `### Remediation (auto-repair)` для slice acceptance alerts.

**Ограничения:** repair **не** ставит `[x]` на accept; **не** архивирует; **не** меняет scope/spec Requirement без decision (merge cross-slice → decision, не auto).

**Алгоритм:**

1. Parse все `Remediation (auto-repair)` из последнего `reports/quality-control-*.md`.
2. Порядок правок: `design.md` (`## Slices`, матрица, Primary column) → `tasks.md` (metadata, accept, merge) → sync `**Связь со spec:**`.
3. По alert-id:
   - `primary-acceptance-missing` — добавить `**Primary acceptance:**` в metadata; первый sub-bullet `**Primary (обязательно):**` в `S<N>.accept` из Behavior Contract / design.
   - `accept-bullets-missing-scenario` — optional sub-bullet в `S<N>.accept` **или** agent `S<N>.<M>` «верифицировать по коду» **или** explorer в apply; user IB spike **запрещён** (User Task Contract).
   - `acceptance-simplicity-overload` — оставить один mandatory Primary; остальные → «(опционально)» или `S<N>.<M>`.
   - `slice-not-vertical` / `slice-foundation-with-gate` — делегировать **onec-code-architect** `mode=slice-restructuring`; merge foundation → consumer; затем post-merge checklist ниже.
4. Append `debug.md` § `## Verify repair — slice acceptance` (дата, alerts, files touched).

**Post-merge checklist** (после merge foundation → consumer):

1. Перенумерация `# Срез S<N>` и `S<N>.<M>` без пропусков.
2. `**Зависимости:**` пересчитаны; нет ссылок на слитый срез.
3. Ровно один `S<N>.accept` + `<!-- slice-gate -->` на оставшийся срез.
4. `**Primary acceptance:**` синхронны в design и tasks.
5. Append `debug.md` § `## Verify repair — slice merge` (было S1+S2 → стало S1).
6. Grep: нет orphan `S<K>.` в задачах других срезов.

### 6d. Repair-from-verify: user task contract

**Триггер:** internal repair-from-verify; в `quality-control-*.md` или verification report есть `user-task-contract-violation` (CRITICAL).

**Ограничения:** repair **не** ставит `[x]`; **не** меняет scope/spec Requirement без decision; `[x]` задачи **не** перенумеровывать.

**Алгоритм:**

1. Удалить нарушающие `S<N>.<M>` (user runtime-spike).
2. Grep `tasks.md` и снять условные зависимости: «При успешном verify S*.2», «после verify S*», «после стенда».
3. Слить blocked кодовые задачи в одну без условия «после стенда».
4. `design.md`: open question из spike → § Assumptions; § Risks — «runtime-подтверждение в Primary accept среза».
5. Перенумеровать только `[ ]` задачи; `[x]` не трогать (номера выполненных могут остаться с gap).
6. Append `debug.md` § `## Verify repair — user task contract` (дата, удалённые задачи, files touched).

### 6b. Code-sync update rules (`--code-sync`)

После подтверждения брифа:

1. Делегировать `onec-code-explorer` с точным списком файлов из `tasks.md`/`debug.md`/reports; запросить:
   - реальные процедуры/функции/аннотации;
   - порядок вызовов;
   - расхождения `artifact says` vs `code says`;
   - цитаты `file:line` для каждого вывода.
2. Сохранить полный отчёт в `reports/exploration-code-sync-YYYY-MM-DD.md`.
3. Оркестратор перепроверяет ключевые цитаты точечным `Read` перед правкой артефактов.
4. Обновлять `proposal.md`, `design.md`, `specs/**`, `tasks.md`, `debug.md` так, чтобы они отражали код как source of truth.
5. Если код противоречит Behavior Contract, не «догонять» артефакты молча: остановиться, сформировать decision card (код править через `/opsx:apply` или менять scope через extend).

### 7. Verification Gate

После обновления артефактов:

- проверить, что tasks/spec/design согласованы по названиям срезов и сценариев;
- если specs менялись — пройти Delta Specs Gate;
- если добавлены задачи фикса — проверить defect placement invariant из `vertical-slices.mdc`;
- если запуск был `--code-sync` — выполнить Code-Truth Gate из `.cursor/rules/code-truth-gate.mdc`; `phantom-symbol` после синхронизации = BLOCKER до повторной правки артефактов;
- **User Task Contract (user-extend):** mechanical grep DENY/ALLOW (`vertical-slices.mdc` § User Task Contract) на добавленные/изменённые `S<N>.<M>`; CRITICAL `user-task-contract-violation` → не завершать extend без правки;
- следующий шаг для пользователя: `/opsx:verify <name>` (не дублировать в чат длинный handoff — см. п. 8).

### 8. Handoff

Финальный вывод в **чат** — Chat Surface Contract §2.6 + §3a `chat-output-budget.mdc`:

- **repair-from-verify (internal):** **нет** сообщения в чат.
- **user-extend, правок не было:** одна строка на языке эффекта: «Артефакты соответствуют запросу, правок не потребовалось.»
- **user-extend, артефакты изменены:** «Постановка дополнена, можно проверять. Следующий шаг: `/opsx:verify <name>`.» — **без** перечня файлов, **без** internal-команд с флагами.

Полный перечень правок — в `debug.md` (`## Extend — YYYY-MM-DD`) и при необходимости `reports/extend-summary-*.md`, **не** в чат.

**Запрещено:** «Обновлено: `tasks.md`, `design.md`, … Дальше: `/opsx:extend --from-verify …`».

Если изменения не внесены из-за неоднозначности — компактная развилка (исключение из однострочного режима).

---

## Integration

- `session-discipline.mdc`: первым tool call при `/opsx:extend` — Read этого скилла; протокол extend действует на каждом follow-up ходе.
- `1c-agent-delegation.mdc`: extend не реализует BSL/XML и не вызывает writer/reviewer (BSL и XML write guard — там).
- `openspec-specs-gate.mdc`: при изменении specs соблюдать delta-формат.
- `vertical-slices.mdc`: все изменения `tasks.md` slice-aware.
- `code-truth-gate.mdc`: `--code-sync` — штатный remediation path для `phantom-symbol` и drift code↔artifacts.
- `verified-cause-gate.mdc`: если вход — defect/RCA, разделять Verified facts и Hypotheses.
- `preserve-subagent-reports.mdc`: сохранять полные отчёты architect/explorer.
- `architect-gate.mdc`: scope-drift триггеры и закрытие Gate через `architecture-extend-coherence-*.md` (см. п. 5a).

---

## Common Follow-up Recommendations

Команды семейства `/opsx:*` должны ссылаться на extend, когда вывод показывает необходимость изменить scope:

- `/review`: `Architecture findings` или findings, противоречащие `design.md` → `/opsx:extend <name> --from-review <report-path>`.
- `/opsx:explore` (Ultra-Lite): RCA и рецепт → `/opsx:extend <name> --from-report temp/reports/<тип>-YYYY-MM-DD-<slug>.md` (отчёт `Task`) или `temp/explore-handoff-*.md` (если был handoff).
- `/opsx:verify`: Repair Loop даёт класс `decision` по scope/design/tasks → `/opsx:extend <name> --from-verify <report-path>`.
- `/opsx:apply`: реализация выявила scope mismatch → `/opsx:extend <name>`.
- `/opsx:explore`: есть активный change и обсуждение даёт новое требование → `/opsx:extend <name>`.
