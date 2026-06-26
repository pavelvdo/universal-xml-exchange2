---
name: review
description: Full code review by request context (module, files, extension) with optional fix via writer and reviewer. APPLY GATE exception — см. .cursor/rules/1c-agent-delegation.mdc (секция APPLY GATE).
license: MIT
compatibility: Delegates to onec-code-reviewer (prompt_contract_version=3), onec-code-writer, onec-code-simplifier, onec-code-explorer, onec-code-architect. Requires Task tool.
metadata:
  author: project
  version: "2.2"
  expected_reviewer_prompt_contract_version: 3
---

Провести полное подробное ревью кода в объёме по контексту запроса пользователя, делегировать ревью **onec-code-reviewer**, сохранить отчёт, затем по подтверждению пользователя передать замечания **onec-code-writer** для устранения с обязательным повторным ревью (макс. 2 итерации).

**Input**: путь к модулю (.bsl), каталог, список файлов, имя расширения, имя ЗНИ (каталог `openspec/changes/<name>`), «текущий файл», либо **без аргументов** (тогда scope — изменения `.bsl` в рабочем дереве git).

**Памятка заказчика (вызов, примеры, отличия `/review` vs `/release-review`):** [`.cursor/docs/review-guide.md`](../../docs/review-guide.md).

**Входы:** `/review` — операционное ревью (этот skill, `release_mode=false`). `/release-review` — предрелиз (`release_mode=true`; команда [`.cursor/commands/release-review.md`](../../commands/release-review.md)).

**Флаги (`/review` only):** `--full` (полное ревью, отключить light-review triage).

**Review Focus Level:** `full` — ревью всего указанного кода в файлах; `diff-focused` — замечания только в изменённых процедурах/функциях и `[module-level]` участках (см. шаг 1.5 и `onec-code-reviewer.md`, Review Boundaries).

**Ключевая архитектурная идея v2.0:** оркестратор собирает **evidence** (scope, границы, линтер, история, whitelist проекта, architectural context). Ревьювер — **единственный владелец вердиктов** (severity/kind/action/risk). AP-каталог — единственный источник истины для AP-правил (см. `.cursor/rules/bsl-antipatterns.mdc`).

---

## Шаг 0. Флаги и `release_mode`

Зафиксировать до резолва scope:

| Переменная | Условие |
|---|---|
| `release_mode` | `true` — команда `/release-review`; `false` — `/review` |
| `full_review` | `true`, если `--full` **или** `release_mode` |
| `review_mode` | `full-extension` — `/release-review` без change; `change-scoped` — `/release-review` + change; иначе — по таблице 1.2 |
| `extension_name` | имя папки cfe (из аргументов или контекста) |
| `change_name` | имя каталога change при `change-scoped` |

При `release_mode`: acknowledgement в шаге 1.1 — «Понял: запускаю **предрелизное** ревью…».

---

## Шаг 1. Resolve scope (со smart defaults)

Определить scope по **приоритету** (первое совпадение из контекста запроса пользователя). Зафиксировать **`review_focus`** глобально и/или **пофайлово** (см. шаг 1.5).

### 1.0 Приоритет `/release-review` (`release_mode`, до таблицы 1.2)

Если `release_mode = true`:

| Ввод | Действие | `review_focus` |
|---|---|---|
| Расширение без change (`/release-review <ext>`) | Glob все `*.bsl` в `src/*/cfe/<ext>/`; `extension_all_bsl` = полный список | `full` |
| Расширение + change (`/release-review <ext> <change>`) | Шаг **1.3a** (change-scoped resolver); Tier 1 = `target_files` | `diff-focused` |
| Только change без расширения | Валидировать change; resolver 1.3a; cfe из путей в `target_files` | `diff-focused` |

После резолва — шаг **1.3c** (`extension_all_bsl` для Tier 2). Light-review (1.4) **не применять**.

### 1.1 Smart defaults (S12)

**Acknowledgement first:** При получении запроса на ревью (любой вариант из таблицы 1.2) **первой строкой** ответа, до вызова любых инструментов (Glob, Shell, Read), вывести:
> «Понял: запускаю ревью по <scope: диффу .bsl / файлу / расширению> в <change / ветке>, файлов: N. Если scope другой — скажите.» (подставить актуальный scope).

Вместо AskQuestion при неоднозначности — применить следующие правила:

1. **Активный change + непустой git diff `.bsl`:**
   - Если в контексте сессии виден активный change (`openspec/changes/<name>/` существует, пользователь ведёт ЗНИ) И `git diff --name-only HEAD` ∪ `--cached` содержит `.bsl`-файлы — **default scope = change-scoped diff-focused**.
   - НЕ задавать AskQuestion; вывести явное уведомление пользователю:
     > `Detected: change "<name>", diff-focused review over <N> .bsl file(s). Continue (Enter) / specify other scope (file/ext/full).`
   - Если пользователь следующим сообщением не переопределил scope — продолжать с defaults.
2. **Неоднозначность без change и без diff:** AskQuestion (fallback, как в таблице ниже).

### 1.2 Таблица резолва

| Вход пользователя | Действие | `review_focus` |
|-------------------|----------|----------------|
| **Явные пути** к `.bsl` или каталогу | Нормализовать относительно корня репо; для каталога — все `*.bsl` под ним (Glob). | `full` |
| **Текущий/открытый файл** («ревью этого модуля», `review current file`) | Путь из IDE; файл должен быть `.bsl`. | `full` |
| **Расширение** (имя папки cfe или «ревью расширения X») | `openspec/project.md` → пути cfe; Glob `*.bsl` в каталоге расширения. | `full` |
| **Имя ЗНИ / каталог change** (`openspec/changes/<name>`, `.../archive/<name>`, или явное имя в тексте) | Валидировать `tasks.md`. Список файлов: задачи `[x]`, пути из `` `src/...bsl` `` + наследование «тот же файл» в секции `##`. При пустом списке — git-fallback по slug. | `diff-focused` |
| **Без аргументов, активный change и непустой git diff** | Smart default (1.1 п.1). | `diff-focused` |
| **Без аргументов, нет active change, есть git diff** | `git diff --name-only HEAD` ∪ `git diff --name-only --cached`, оставить `.bsl`. | `diff-focused` |
| **Без аргументов, scope неясен, git diff пуст** | **AskQuestion** — указать файл/каталог, расширение, имя ЗНИ или подтвердить полное ревью с явным scope. | — |

**Валидация:** пути существуют, расширение `.bsl`. Исключить удалённые-only файлы из списка ревью.

**Итог шага:** список путей к `.bsl` + для каждого файла черновик режима (`full` | `diff-focused`); окончательно уточняется в шагах 1.4–1.5.

### 1.3a Change-scoped resolver (только `release_mode` + change)

Построить списки с provenance:

- **`target_files`** — `.bsl` внутри выбранного cfe; Tier 1 и mandatory evidence.
- **`reference_files`** — `.bsl` вне cfe; read-only context, findings запрещены.
- **`doc_files`** — markdown/spec; не в BSL reviewer.
- **`ambiguous_files`** — пути без однозначного `src/`.

**Источники (приоритет):** `reports/architecture-*.md` (`scope.files`) → `tasks.md` (`[x]`, backticks) → `design.md` (backticks) → git fallback по slug change.

**Suffix matching:** для путей без `src/` — `Glob **/<suffix>`; 1 hit → resolve; >1 или 0 → `ambiguous_files`.

**Scope Preview:** если `target_files` пуст **или** `ambiguous_files` не пуст — **AskQuestion**:

> **Scope Preview:** Target: N, Reference: M, Ambiguous: K ([список]). Продолжить / переключиться в full-extension / остановиться?

Если всё однозначно — продолжить без вопроса. Затем шаг 1.5 для `target_files` (Review Boundaries).

### 1.3c `extension_all_bsl` (только `release_mode`)

Glob `src/*/cfe/<extension>/**/*.bsl`. При `full-extension` — Tier 1 и Tier 2; при `change-scoped` — Tier 2 only (Tier 1 = `target_files`).

---

## Шаг 1.4. Light-review триаж (S15)

Выполнять ПОСЛЕ шага 1 и ДО шага 1.5 для `diff-focused` сценариев. Цель — не гонять Phase 0 / Phase 2.5 / AP-пасс на тривиальных правках.

### Алгоритм

Для каждого файла в scope собрать unified diff (как в 1.5.1). Классифицировать изменения:

1. **Whitespace-only** — отличия только в пробелах/табах/пустых строках.
2. **Comment-only** — изменены только комментарии (строки `//`).
3. **Rename-only** — единый find-replace идентификатора (имя переменной/параметра) без изменения структуры/логики.

Если объединение всех файлов batсh попадает **целиком** в (1 ∪ 2 ∪ 3) — предложить light review.

### Решение

- **`release_mode = true`** — пропустить шаг 1.4 целиком (как при `--full`).
- **Активный `--full` флаг пользователя** (явное указание «полное ревью» / «не сокращай») — игнорировать триаж, идти в обычный flow.
- **Default** (без `--full`): применить light review без AskQuestion; уведомить:
  > `Trivial diff detected (<whitespace/comments/renames>). Light review: linter + whitelist check only. Use /review --full to override.`

### Light review scope

- Выполнить шаг 1.8 (Linter pass) — обязательно.
- Выполнить чтение whitelist из project.md (шаг 1.6).
- Пропустить шаги 1.5 (procedure mapping не нужен для простых diff), 2–3 (агент не вызывается).
- Отчёт (шаг 4) — короткая сводка: «Light review; линтер: N диагностик; whitelist: K нарушений».
- Если линтер или whitelist-проверка нашли проблемы — эскалировать: AskQuestion «Переключиться в полное ревью?».

Если триаж НЕ сработал — продолжать стандартный flow с шага 1.5.

---

## Шаг 1.5. Diff extraction и procedure mapping (при `diff-focused`)

Выполнять для файлов, которым назначен **`diff-focused`**. Для **`full`** — шаг пропустить.

### 1.5.1 Baseline для `git diff`

1. **Рабочее дерево (без аргументов, есть незакоммиченные `.bsl`):** объединение `git diff HEAD -- <file>` и `git diff --cached -- <file>`. Baseline: `working tree vs HEAD (+ index)`.
2. **Ветка ≠ основной:** определить основную: `main` → `master` → `origin/main`. `BASE=$(git merge-base <main> HEAD)`; `git diff $BASE..HEAD -- <files>`. Baseline: `merge-base <main>..HEAD = <hash>`.
3. **На основной ветке + scope по ЗНИ:** slug из имени change; `git log --oneline --grep="<slug>"`; диапазон `git diff <parent>^..HEAD -- <files>`; если коммиты не найдены — fallback п.4.
4. **Fallback:** `git log --oneline -10 -- <file>`; **AskQuestion** выбрать baseline-коммит или перейти на `full` с предупреждением.
5. **Пустой diff при ненулевом списке:** предложить `full` по этим файлам или уточнить baseline (п.4).

### 1.5.2 Новый файл в diff

Если для файла `new file mode` / `from /dev/null` — `per_file_focus = full`; в `## Review Boundaries`: `Focus: full`.

### 1.5.3 Извлечение номеров строк в текущей версии

Из unified diff для каждого файла собрать множество номеров строк **новой** версии (префикс `+`, не считая `+++`). Учитывать `@@ -a,b +c,d @@`.

### 1.5.4 Маппинг на процедуры BSL

Прочитать файл; найти границы методов regex: `^\s*(Процедура|Функция)\s+(\w+)` / `^\s*КонецПроцедуры|КонецФункции\b`. Каждую изменённую строку отнести к процедуре или `[module-level]`.

**Результат по файлу:** список `{kind, name, startLine, endLine}` + опционально `[module-level]` диапазоны.

### 1.5.5 Сборка секции `## Review Boundaries`

Вставить в промпт ревьювера (перед `## Reasoning focus`):

```markdown
## Review Boundaries

Focus: diff-focused
Baseline: <baseline-description>

File: <path>
Target Procedures:
- <name> (lines <start>-<end>)
- [module-level] (lines <start>-<end>)

File: <path2>
Focus: full (new file)
```

---

## Шаг 1.6. Project constraints (evidence)

**Только чтение `openspec/project.md`** для передачи ревьюверу как evidence. Механические grep-проходы (прошлые 1.6.2/1.6.3/1.6.5/1.6.6) перенесены в агент (AP-040..AP-045 каталога, Phase 2 «Release-hygiene pass»). Whitelist exempt **removal** only; AP-053 content — в Phase 2 reviewer.

### 1.6.1 Whitelist и обязательный контроль

Прочитать [openspec/project.md](../../../openspec/project.md), секция **«Форматы и соглашения по комментариям BSL»**. Извлечь таблицы **Whitelist предрелиза** и **Обязательный контроль**. Whitelist exempt AP-040 **removal**; содержимое domain_label — AP-053 в Phase 2 reviewer. Если секции нет — обе таблицы пусты.

**Памятка по колонкам и слоям:** [.cursor/docs/bsl-comment-formats-project.md](../../docs/bsl-comment-formats-project.md), [.cursor/docs/marker-layers-guide.md](../../docs/marker-layers-guide.md).

Результат передать в бриф шага 2 как `## Whitelist & Mandatory Controls (from project.md)` — две таблицы целиком + scope globs.

### 1.6.2 Обязательный контроль (regex-based rules)

Если в п.1.6.1 таблица **«Обязательный контроль»** содержит строки с непустым **Regex допустимой строки** — оркестратор может выполнить проверку как evidence (не заменять ревьювера). Алгоритм mandatory control: regex из `project.md` § «Обязательный контроль» на первой значимой `//` после `#Вставка`. Результаты — блок `## Mandatory Control Signals (evidence)`:

```markdown
## Mandatory Control Signals (evidence)

| Rule ID | File:Line | Observed | Expected (regex) |
|---------|-----------|----------|------------------|
| DIR-001 | X.bsl:12 | // стандарт... | ^//\s*Required:\s*.+$ |
```

Ревьювер обязан для каждой строки выдать confirm/dismiss/reclassify.

**Прочие grep-эвристики (kebab-case, жаргон, log-literal, empty methods) оркестратор НЕ выполняет** — кроме **Naming Provenance** (шаг 1.9). Остальное делегировано агенту (AP-040..AP-045 + AP-031 naming в Phase 0/1c) с использованием Intent Map / Contract Map.

---

## Шаг 1.9. Naming Provenance pass (evidence, не findings)

Выполнить **после** шага 1.8, **до** вызова ревьювера. Алгоритм — **SSOT:** `.cursor/rules/1c-writer-pipeline.mdc` § NAMING PROVENANCE CHECK.

### Входные параметры

- **apply/review в рамках ЗНИ:** `openspec/changes/<name>/proposal.md`, `tasks.md`, имя change.
- **Standalone `/review` без change:** только статический blocklist из pipeline; ticket/slug — пропустить (grep по blocklist).
- **Scope:** изменённые `.bsl` из writer-diff (apply) или in-scope файлы из Review Boundaries (full-file / Mechanical).

### Обработка

- Выполнить grep по алгоритму pipeline (3 класса целей, исключения whitelist-маркеров).
- Агрегировать в блок **`## Naming Signals (evidence)`** — всегда передавать reviewer (clean / matches / `Naming scan skipped: no diff`).
- Оркестратор **не** создаёт findings — только evidence (аналог шага 1.8).

```markdown
## Naming Signals (evidence)

| # | File:Line | Match | Class | Suggested |
|---|-----------|-------|-------|-----------|
| 1 | Module.bsl:287 | ВыполнитьPostWrite… | procedure | AP-031 MUST_FIX |
```

Если совпадений нет: `Naming Signals: clean (N files scanned)`.

- Блок передаётся ревьюверу. Ревьювер в **Phase 1c** (`.cursor/docs/standard/reviewer-checks.md` § Phase 1c: Naming Provenance Gate) обязан обработать каждую строку evidence: по умолчанию **confirm → AP-031 MUST_FIX**; dismiss только с явной причиной (`metadata-name`, `false-positive`, `pre-existing-unchanged`).

**Важно:** оркестратор НЕ переводит сигналы в findings — это работа ревьювера. Оркестратор **обязан** передать блок evidence, не отфильтровывая совпадения (кроме исключений pipeline).

---

## Шаг 1.8. Linter / LSP pass (evidence, не findings) — S9

Выполнить ДО вызова ревьювера. Собрать статические диагностики как **сигналы** (reviewer решает, что с ними делать).

### Инструменты (порядок предпочтения)

1. `user-1c-syntax-checker-syntaxcheck(code)` для каждого `.bsl` в scope (с учётом Review Boundaries — можно передавать усечённый фрагмент или весь файл).
2. `user-1c-code-checker-check_1c_code(code, check_type)` для logic checks.
3. `bsl_lsp_diagnostics` (если BSL LSP доступен в сессии) — платформенная диагностика.
4. Fallback: `ReadLints` по изменённым файлам (Cursor-native).

### Обработка

- Если инструменты недоступны (LSP офлайн, linter не найден) — gracefully degrade: в промпт ревьювера вставить `## Linter Signals (evidence)` с комментарием `Linter unavailable: <reason>`. В Summary отчёта — warning.
- Если инструмент выдал диагностики — агрегировать в единый блок:

```markdown
## Linter Signals (evidence)

| # | Tool | File:Line | Severity (tool) | Message | Suggested AP/Category |
|---|------|-----------|-----------------|---------|-----------------------|
| 1 | syntax-checker | X.bsl:45 | error | Unclosed НачатьТранзакцию | AP-015 |
```

- Блок передаётся ревьюверу. Ревьювер в **Phase 1b** (`.cursor/docs/standard/reviewer-checks.md` § Phase 1b: BSL Linter Signals Gate) обязан обработать каждую in-scope диагностику: по умолчанию **confirm → MUST_FIX**; dismiss только с явной причиной (`out-of-scope`, `false-positive`, `pre-existing-unchanged`). **Запрещено** откладывать in-scope warning на prerelease («погасим техдолг позже»).

**Важно:** оркестратор НЕ переводит сигналы в findings и не выставляет Action — это работа ревьювера (единый источник вердиктов — S3 принцип). Оркестратор **обязан** передать warnings в блоке evidence, не отфильтровывая их.

---

## Шаг 2. Контекст для ревьювера

Сформировать бриф:

- **Version contract (S13):** в начало промпта вставить `expected_reviewer_prompt_contract_version: 3` (значение из frontmatter скилла). Ревьювер сверяет с `prompt_contract_version` в своём frontmatter; несовпадение → warning в отчёт.
- Список файлов.
- Стандарты: `.cursor/docs/1c-coding-standards.md`.
- Задача:
  - **`release_mode = false`:** «Полный подробный ревью. Без `mode=prerelease`.»
  - **`release_mode = true`:** «Предрелизное ревью. **mode=prerelease**. Category 12 Release Readiness (`.cursor/docs/standard/reviewer-checks.md` §12). Эскалация severity по AP-каталогу. Release-hygiene AP-040..AP-045.»
  - При `diff-focused`: «Соблюдать `## Review Boundaries` (замечания только в границах).»
- **Whitelist & Mandatory Controls (from project.md)** — блок из шага 1.6.1.
- **Mandatory Control Signals (evidence)** — блок из шага 1.6.2 (если есть).
- **Linter Signals (evidence)** — блок из шага 1.8 (всегда, даже если пусто / unavailable).
- **Naming Signals (evidence)** — блок из шага 1.9 (всегда: clean / matches / skipped).
- **Prior Findings History (S8)** — см. блок ниже.
- **Architectural Context** — см. блок ниже.
- **Review Boundaries** — при `diff-focused` (шаг 1.5), только для файлов батча.
- **Resolved Contracts** — при повторном прогоне после Investigation Loop (шаг 3.5).
- **Base-файл для &ИзменениеИКонтроль:** если в scope есть файлы с этой аннотацией, прочитать `openspec/project.md` (секция «Структура репозитория»), подставить cf-путь в пару к cfe-пути. Передать в промпт: «Base-файл: <путь>» для каждого такого файла (EXTENSION GATE из `1c-agent-delegation.mdc`).

### 2.1 Prior Findings History (S8)

Цель — дать ревьюверу видеть повторяющиеся замечания.

1. Определить `scope-slug` (совпадает с именем сохраняемого отчёта из шага 4).
2. Glob:
   - `openspec/changes/*/reports/review-<scope-slug>-*.md`
   - `openspec/changes/archive/**/reports/review-<scope-slug>-*.md`
   - `temp/reports/review-<scope-slug>-*.md` (если temp/ входит в репо)
3. Отфильтровать по дате ≤ 90 дней (mtime файла).
4. Из каждого отчёта извлечь список findings (таблица или нумерация; парсить секцию `## Findings`).
5. Передать в промпт блок:

```markdown
## Prior Findings History

Отчёты ≤ 90 дней по тому же scope. Ревьювер обязан для каждого совпадающего finding (по file:Anchor:AP-NNN) добавить tag `recurrent` и применить эвристику risk model (scope up, confidence ≥ 0.9).

### Отчёт: <путь/имя файла> (<дата>)

| # | AP | Procedure | Anchor | Action | Severity | Status |
|---|----|-----------|--------|--------|----------|--------|
| 1 | AP-015 | ЗаписатьДокументы | НачатьТранзакцию(); | MUST_FIX | CRITICAL | unknown |
```

Если отчётов нет — блок не вставлять (или вставить `## Prior Findings History — none`).

### 2.2 Architectural Context

- Если ревью в рамках ЗНИ (`openspec/changes/<name>`): прочитать `design.md` и `reports/architecture-*.md`. Передать в промпт как `## Architectural Context` (сокращённо или целевой раздел).
- Если ЗНИ нет — блок пропустить.

---

## Шаг 3. Делегирование ревью

> Task Pre-call Checklist — `.cursor/rules/tool-name-guard.mdc` (subagent_type из списка; `model` — по `.cursor/rules/model-selection.mdc`).

### Шаблон промпта

| `release_mode` | Шаблон |
|---|---|
| `false` | «Reviewer (ревью кода)» — `.cursor/skills/1c-agent-patterns/reviewer.md` |
| `true` | «Reviewer (предрелиз)» — тот же файл |

Вызвать **Task**(`subagent_type="onec-code-reviewer"`) с промптом по выбранному шаблону:

- Файлы: список из шага 1 (для батча — подмножество).
- `expected_reviewer_prompt_contract_version`: 3.
- **`release_mode = true`:** явно **`mode=prerelease`** в промпте; Category 12; release-hygiene focus.
- **`release_mode = false`:** **не** передавать `mode=prerelease`.
- Диагностики линтера: блок **`## Linter Signals (evidence)`** из шага 1.8 (все severity, включая warning; не сокращать до «линтер чист»).
- Naming Provenance: блок **`## Naming Signals (evidence)`** из шага 1.9 (clean / matches / skipped).
- Whitelist / Mandatory: блоки из шага 1.6.
- Prior Findings History: блок из шага 2.1 (если есть).
- Architectural Context: из шага 2.2 (если есть).
- Review Boundaries: из шага 1.5 (при `diff-focused`).
- Reference Files (только `change-scoped`): read-only context, findings запрещены.
- Base-файл(ы): из шага 2 для файлов с &ИзменениеИКонтроль.
- Resolved Contracts: при повторном прогоне.

**Батчи:** >5–8 файлов — разбить. При **`release_mode = true`** — группы A (object/manager/common), B (`ВызовСервера`/`Глобальный`), C (`Forms/*/Module.bsl`); A >5 → A1/A2 по 3–5 файлов. Для каждого батча — отдельный `## Review Boundaries`. Итоговый список findings — объединение с дедупликацией (шаг 4 S14).

### 3.2 Tier 2 — архитектурный explorer (только `release_mode`)

Параллельно с шагом 3.1 (или после первого батча): **Task**(`subagent_type="onec-code-explorer"`) по шаблону «Explorer — deep-analysis» из `.cursor/skills/1c-agent-patterns/explorer.md`:

- Scope: **`extension_all_bsl`** (все `.bsl` расширения).
- Промпт: архитектурный анализ расширения; `mode=prerelease`; findings `kind=architecture`.
- Результат объединить с Tier 1 в шаге 4 (дедупликация S14).

**Risk Surfacing (release):** в слоте «Что важно» — при `change-scoped`: «Tier 2 покрыл всё расширение; изменения вне `target_files` не блокеры релиза для Tier 1».

### 3.1 Version contract check

После ответа агента: если в самом ответе ревьювер явно сообщил `prompt_contract_version mismatch` или ревьювер пометил, что не поддерживает переданные блоки — записать warning в Summary отчёта, продолжить работу (не падать).
**Honest Subagent Handling:** Если Task завершился ошибкой (`failed`) или был прерван пользователем (`interrupted-by-user`) — следовать протоколу `chat-output-budget.mdc` §5: явно назвать состояние, не придумывать диагноз, предложить retry/fallback.

---

## Шаг 3.5. Investigation Loop (multi-iteration, S11)

1. **Парсинг:** искать `## Investigation Request` в выводе ревьювера. Нет секции → пропустить шаг.
2. **Если есть:**
   - Извлечь таблицу (Метод, Контекст, Что нужно определить).
   - Вызвать **Task**(`subagent_type="onec-code-explorer"`) по шаблону «Explorer — contract resolution (deep)». Передать всю таблицу.
   - Explorer возвращает `## Resolved Contracts`.
3. **Сохранение артефакта:**
   - Активный change: `openspec/changes/<id>/reports/resolved-contract-<scope-slug>-YYYY-MM-DD[-iter<N>].md`.
   - Без change: `temp/reports/resolved-contract-<scope-slug>-YYYY-MM-DD[-iter<N>].md`.
4. **Повторное ревью:** `onec-code-reviewer` — тот же scope + `## Resolved Contracts` + `## Review Boundaries`. Промпт: «Повторное ревью с Resolved Contracts. Секцию Investigation Request НЕ включать **или** включить только для ненайденных контрактов.»
5. **Лимит итераций:** `max_iterations = 3` (reviewer → explorer → reviewer считается 1 итерацией). Если на итерации `k` ревьювер снова выдал непустой `## Investigation Request` — повторить цикл до `max_iterations`.
6. **Termination:** если после `max_iterations` остались нерезолвленные контракты:
   - В итоговом отчёте — секция `## Remaining Unresolved Contracts` (таблица из последнего Investigation Request).
   - Writer получает эти точки как риски с понижением `confidence` (эвристика: confidence ≤ 0.5, blast_radius проекция зависит от операций).
**Honest Subagent Handling:** Любые сбои explorer в цикле обрабатываются по §5 (failed/interrupted-by-user).

### Сохранение промежуточных отчётов

Каждая итерация ревью — отдельный файл: `review-<scope-slug>-YYYY-MM-DD-iter<N>-initial.md` (до investigation) и без `-initial` (после). Итоговый = последний.

---

## Шаг 4. Отчёт (split: main + reasoning appendix, с дедупликацией)

### 4.1 Парсинг ответа ревьювера

Ответ агента состоит из двух секций:

- `## === MAIN REPORT ===` ... (Summary + Findings + опционально Investigation Request / Unverified API / Remaining Unresolved Contracts / Elegance Score).
- `## === REASONING APPENDIX ===` ... (Intent Map, Contract Map, Knowledge Assessment, Evaluation Checklist, Попытка Audit Table, Defensive Checks Table).

Разделить на два артефакта.

### 4.2 Дедупликация findings (S14)

Ключ: `(file, anchor_hash, ap_id)`, где:
- `file` — нормализованный относительный путь от корня репо.
- `anchor_hash` — SHA-1 от нормализованного Anchor (trim, lowercase пробелов, убрать лишние пробелы).
- `ap_id` — `AP-NNN` из finding; если finding не AP-based (Phase 0 TYPE или release-hygiene TAG) — использовать сам тег как ключ (например `Phase0-KNOWLEDGE_DEFICIT`, `release-hygiene-AP-040`).

При дубликатах — оставить finding с большим `risk_score`; в сохраняемый отчёт добавить помёту `merged from batch <K>`. Для findings без `risk_score` (теоретически невозможно после S2) — по severity desc, далее — alphabetical.

### 4.3 Сохранение

- **Main report:** 
  - Активный change: `openspec/changes/<id>/reports/review-<scope-slug>-YYYY-MM-DD.md`.
  - Иначе: `temp/reports/review-<scope-slug>-YYYY-MM-DD.md`.
- **Reasoning appendix:**
  - `<тот же каталог>/review-<scope-slug>-YYYY-MM-DD-reasoning.md`.
- В Summary main report — ссылка на appendix.
- Если выполнялся Investigation Loop — путь к `resolved-contract-*.md` в Summary.

### 4.4 Сводка пользователю (4-слотная карточка)

Вместо техничной сводки вывести в чат компактную карточку (см. `chat-output-budget.mdc` §5):

1. **Что отрецензировано** — UX-формулировка scope (модуль / файлы / расширение / ЗНИ).
2. **Итог** — топ-3 находки в пользовательском языке + общее число (без severity-меток в заголовках).
3. **Что важно** (Risk Surfacing) — «не покрыто этим ревью»: при `diff-focused` — неизменённый код, при `light review` — Phase 0 / AP-пасс, при mismatch контракта ревьювера — затронутые findings.
4. **Куда дальше** — одна команда (устранение / extend / archive). При **`release_mode`:** если есть ARCH-findings или MUST_FIX scope/design — `/opsx:extend <change> --from-review <main-report-path>`; иначе устранение через writer или «только отчёт».

Полный отчёт (счётчики CRITICAL/HIGH/MEDIUM/LOW, разделение CODE/ARCHITECTURE) остаётся в `reports/review-<scope-slug>-YYYY-MM-DD.md` и appendix.

---

## Шаг 5. Предложение устранить

Если есть findings с `Action = MUST_FIX | REFACTOR`:

**AskQuestion:**
1. Есть **MUST_FIX CODE** → «Устранить дефекты через onec-code-writer? (Да / Нет, только отчёт)».
2. Есть **REFACTOR** (и нет MUST_FIX, или после их исправления) → «Код работает, но ревьювер нашёл N запахов кода. Запустить onec-code-simplifier? (Да / Нет)».

Действия:
- **Нет** — к шагу 7.
- **Да** — к шагу 6.

Если только `VERIFIED_OK` / `OPTIONAL` / `ARCHITECTURE` — к шагу 7.

---

## Шаг 6. Устранение (при «Да»)

### 6.1 Фильтрация findings

- **Architecture (Type: ARCHITECTURE)** — списком пользователю (шаг 7, architect trigger S10).
- **MUST_FIX CODE** — передать writer.
- **REFACTOR** — передать simplifier.
- Сортировка внутри каждой группы — **по `risk_score` desc** (S2), не по severity.

### 6.2 Порядок

1. Writer (MUST_FIX CODE).
2. Simplifier (REFACTOR, если пользователь согласился).

### 6.3 Чеклист перед Task

- Если был шаг 3.5: файл `resolved-contract-*.md` существует; `## Resolved Contracts` включён в промпт writer/simplifier.
- Без блока — НЕ вызывать агента для findings, затрагивающих контракты из Investigation Request.

### 6.4 Для каждого затронутого файла

1. **MUST_FIX:** Task `onec-code-writer` по шаблону «Writer — review fix». В промпте findings отсортированы по `risk_score` desc.
2. **REFACTOR:** Task `onec-code-simplifier` (шаблон: `.cursor/skills/1c-agent-patterns/simplifier.md`).
3. **LINT GATE** — см. [`.cursor/rules/1c-agent-delegation.mdc`](../../rules/1c-agent-delegation.mdc) § LINT GATE + [`.cursor/rules/1c-writer-pipeline.mdc`](../../rules/1c-writer-pipeline.mdc).
4. **API EXISTENCE CHECK (smart default S12):**
   - По правилам из `1c-writer-pipeline.mdc` § API EXISTENCE CHECK проверить новые вызовы `МодульИмя.МетодИмя(` в diff (порядок шага — `1c-agent-delegation.mdc` § WRITER PIPELINE).
   - **Auto-pass правило:** если каждый WARN-вызов имеет **≥ 3 hits в extension scope** (Grep по `МодульИмя\.МетодИмя\(`) — auto-pass с записью INFO в отчёт, без AskQuestion.
   - Если < 3 hits — AskQuestion как обычно.
5. При `&ИзменениеИКонтроль` — EXTENSION VERIFICATION (base vs код вне #Вставка/#Удаление).
6. Task `onec-code-reviewer` по изменённым файлам. Пересобрать `## Review Boundaries` (новый diff).

**Максимум 2 итерации writer → review.** После — к шагу 7 даже при оставшихся findings.

---

## Шаг 7. Итог и Architect trigger

### 7.1 Резюме

- Что отрецензировано (scope).
- Сколько findings устранено / осталось.
- Пути: main report, reasoning appendix, resolved-contract (если есть).

### 7.2 Architect trigger (S10) — при Architecture findings > 0

Если в финальном отчёте есть findings с `Type: ARCHITECTURE` (после всех итераций):

**AskQuestion:**
> Выявлено N архитектурных замечаний, которые не могут быть исправлены writer/simplifier (требуют изменения метаданных/контрактов/прав/точки расширения). Запустить `onec-code-architect` для пересмотра `design.md` / RCA / обновления openspec? (Да / Нет — оставить в отчёте)

**При «Да»:**
- Вызвать **Task**(`subagent_type="onec-code-architect"`) по шаблону «Architect — глубокий анализ» или «Architect (ревью плана)» (в зависимости от контекста: есть `design.md` или нет).
- Передать:
  - Список ARCH-findings из отчёта (полный блок с Evidence, Counterfactual, Remediation).
  - Текущий `design.md` (если есть).
  - `reports/architecture-*.md` (если есть).
  - Главный вопрос: «Требуется ли обновление design.md / spec / создание ADR для устранения этих findings? Если да — что конкретно?»
- Результат архитектора сохранить:
  - Активный change: `openspec/changes/<id>/reports/architecture-review-<scope-slug>-YYYY-MM-DD.md`.
  - Иначе: `temp/reports/architecture-review-<scope-slug>-YYYY-MM-DD.md`.
- Уведомить пользователя о пути к отчёту архитектора; если архитектор рекомендует обновить `design.md` / `specs/` / `tasks.md` или finding оказался дефектом постановки, вывести handoff:
  `Follow-up: /opsx:extend <change-name> --from-review <main-report-path>` (и при наличии архитектора: `--from-architecture <architecture-review-path>`). После extend — `/opsx:verify <change-name>`.

**При «Нет»:** оставить ARCH-findings в отчёте без действия; сообщить: «ARCH-findings требуют ручного решения; рекомендуется `/opsx:extend <change-name> --from-review <main-report-path>` для обновления ЗНИ, либо `/opsx:explore` для обсуждения подхода».

### 7.2b Scope/design rework trigger

Если в финальном отчёте нет `Type: ARCHITECTURE`, но есть одно из:
- finding `MUST_FIX`, исправление которого противоречит `design.md`, ADR, `tasks.md` или проверенным метаданным;
- finding указывает, что постановка стимулирует плохой код (например, локальная копия типового поведения, альтернативная точка расширения, несогласованность `proposal`/`design`/`tasks`);
- reviewer просит изменить scope, контракт или подход, а не только код;

не передавать это writer как обычный CODE fix. В Summary добавить **Review disposition required** и предложить:

`/opsx:extend <change-name> --from-review <main-report-path>`

Цель extend: показать обязательный бриф, классифицировать findings (accepted/rejected/deferred), при необходимости вызвать architect, обновить `proposal.md` / `design.md` / `specs/` / `tasks.md`, затем вернуть в `/opsx:verify`.

### 7.3 Remaining Unresolved Contracts (S11)

Если есть секция `## Remaining Unresolved Contracts` — вывести её в сводку пользователю с пометкой: «Требуют ручной верификации контракта; writer получил их как точки с пониженным confidence».

---

## Интеграция

- **session-discipline.mdc:** первый инструмент в первом батче — Read этого скилла; протокол действует весь `/review`.
- **1c-agent-delegation:** правки `.bsl` только через writer/simplifier; после правок — обязательный reviewer; LINT GATE (SSOT: `1c-agent-delegation.mdc` + `1c-writer-pipeline.mdc`) и API EXISTENCE CHECK; EXTENSION GATE и EXTENSION VERIFICATION при `&ИзменениеИКонтроль`. **Investigation Loop** (шаг 3.5) — multi-iteration (max 3). Формат — `1c-writer-pipeline.mdc` § CONTRACT RESOLUTION.
- **Review Focus Boundaries:** при `diff-focused` оркестратор формирует `## Review Boundaries`; reviewer следует Review Boundaries Protocol.
- **Evidence separation:** все механические проверки (linter, whitelist, mandatory controls, prior history) — **evidence**; вердикты (severity/kind/action/risk) — **reviewer**.
- **Release-hygiene:** AP-040..AP-045 каталога (не grep оркестратора); reviewer использует Intent Map / Contract Map.
- **APPLY GATE:** writer и reviewer в `/review` разрешён без `/opsx:apply` — исключение в `.cursor/rules/1c-agent-delegation.mdc` (секция APPLY GATE).

---

## Граничные случаи

- **Замечаний нет:** сообщить, указать путь к отчёту.
- **Только архитектурные:** шаг 5 пропустить; шаг 7.2 предложить architect.
- **Нет открытого файла при «текущий файл»:** AskQuestion.
- **Нет изменений `.bsl` в git и scope не задан:** smart default не срабатывает → AskQuestion.
- **Не удалось определить baseline:** fallback на `full` или выбор пользователя.
- **Linter недоступен (S9):** `## Linter Signals (evidence)` = `Linter unavailable: <reason>`; warning в Summary.
- **Light review triggered (S15):** линтер + whitelist only; если найдены проблемы — эскалировать.
- **Prompt contract version mismatch:** warning в Summary; продолжить (fail-open).
- **Investigation loop превышен (S11):** `## Remaining Unresolved Contracts` в отчёт; writer работает с пониженным confidence.

---

## Self-check (Conversational Discipline)

Перед выводом финального сообщения пользователю проверить:
1. **Acknowledgement:** первая строка ответа была «Понял: запускаю ревью...» (шаг 1.1).
2. **Карточка 4 слота:** сводка выведена строго в 4 слота (Что отрецензировано / Итог / Что важно / Куда дальше).
3. **Risk Surfacing:** слот «Что важно» заполнен (неизменённый код, AP-пасс, unresolved contracts).
4. **Honest Subagent:** сбои агентов названы честно (`failed` / `interrupted-by-user`), без выдуманных причин.
5. **HALT-жаргон:** в чате нет CRITICAL/HIGH/MEDIUM/LOW, нет внутренних ID.
6. **Полный отчёт:** счётчики и технические детали сохранены в файл `reports/review-*.md`.
