# Changelog — 1C Agent Ecosystem

## [4.8] - 2026-06-24

### Fixed (framework coherence restore)
- **Revert `747b336` («карта правок»):** снято случайное восстановление slim-down артефактов (doc-tz, estimate, migrate-slices, prerelease-review, openspec-docs/estimate/ff-change, architect-2nd stub, openspec-composer).
- **Восстановлено (cherry-pick из `747b336`/`fb7991e`):** спецификация **Карта правок** (`reports/code-map.md`) — `openspec-apply-change`, `opsx-output-style` §5.2, `chat-output-budget` §1c, `openspec-archive-change`.
- **Сохранено:** reviewer contract v3 (`fb7991e`), verify v8, explore Ultra-Lite v3.
- **Добавлено в git:** intake/debug команды и скиллы, always-apply rules (conversational-discipline, command-skill-gate, bsl-write-guard, orchestrator-as-navigator, …), секция составного брифа intake→explore/debug в `opsx-output-style.md` §5.1.
- **AGENTS.md:** команды `/opsx:intake`, `/opsx:debug` в decision tree.

## [4.7] - 2026-06-10

### Changed (framework polish: rename ff→new, pipeline sync, reviewer v3 pointers)
- **Rename:** `/opsx:ff` → `/opsx:new` (стало основным именем; `/opsx:ff` и `/opsx:continue` — deprecated-stubs); скилл `openspec-ff-change` → `openspec-new-change`; блок handoff «## Для /opsx:ff» → «## Постановка ЗНИ» (SSOT каркаса — `openspec-explore/templates/handoff-block.md`).
- **Pipeline:** единственный эталон порядка writer-пайплайна — `1c-agent-delegation.mdc` § WRITER PIPELINE (writer → ReadLints → API/METADATA CHECK → EXTENSION VERIFICATION → reviewer) + таблица «шаг → исполнитель» и сводная таблица лимитов итераций; `1c-writer-pipeline.mdc`, `review/SKILL.md`, `openspec-apply-change/SKILL.md` ссылаются, не дублируют.
- **onec-code-writer:** удалён истёкший grace period в INPUT CONTRACT; «Lint output» обязателен только для fix-итераций N≥2; OUTPUT в шаблонах `1c-agent-patterns/writer.md` выровнен с агентом (`Code-Truth Symbols`, `Acceptance Criteria` вместо `Self-Assessment`).
- **onec-code-reviewer:** добавлен INPUT CONTRACT (evidence-блоки: Linter Signals, Base-файл, Resolved Contracts, Review Boundaries); ссылки «Phase 1b» перенаправлены на реализацию `.cursor/docs/standard/reviewer-checks.md` § Phase 1b; формат findings в шаблонах — указателем на REPORT FORMAT v3; Evaluation Checklist — 6 вопросов.
- **INVOCATION всех агентов:** рудиментные «Phase 2/4/5/6 of SDD workflow» заменены на текущие команды (`/opsx:apply`, `/opsx:explore`, `/opsx:verify` Layer 4/5, `/review`); `form-generator` из промптов architect убран (ручная конфигурация или программное создание элементов формы в BSL).
- **Light/Mechanical Mode** (`1c-halt-triggers.mdc`): Light — decision tree (1 файл, 2–10 строк, нет новых экспортных символов / metadata-ссылок / межмодульных вызовов); Mechanical — чеклист (паттерн задокументирован, Grep до/после, ReadLints, diff-focused reviewer).
- **Removed:** `openspec-composer` — удалён из `.cursor/agents/` (legacy `openspec/sessions/` больше не собираются; чтение существующих legacy-сессий — только по явной ссылке пользователя).

## [4.6] - 2026-06-07

### Removed (framework slim-down)
- Команды: `/opsx:new`, `/opsx:doc-tz`, `/opsx:estimate`, `/opsx:migrate-slices`, `/prerelease-review` (stub `/opsx:continue` → `/opsx:ff` resume).
- Скиллы: `openspec-new-change`, `openspec-continue-change`, `openspec-docs`, `openspec-estimate`, `openspec-migrate-slices`, `prerelease-review`.
- Агенты/доки: `onec-code-architect-2nd`, `openspec-doc-writer`, `tz-lexicon-dictionary.md`.
- Маркеры: канон `// +++ <ФИО> <дата> [comment_suffix]` / `// --- <ФИО>`; SSOT — `openspec/project.md`; предрелиз — `/release-review`.

## [4.5] - 2026-05-12

### Changed (unified model policy: inherit in YAML + `Task.model` SSOT)
- Все кастомные агенты в `.cursor/agents/*.md`: во frontmatter **`model: inherit`**. Конкретная модель вызова — параметр **`Task(..., model=<slug>)`** по таблице в [`.cursor/rules/model-selection.mdc`](../rules/model-selection.mdc); для `onec-trace-analyst` и `openspec-quality-controller` и финальный шаг fallback — вызов **без** `model=`.
- **`onec-code-architect-2nd`** (файл-заглушка): агент удалён из активной ротации, цепочка fallback архитектора использует один `onec-code-architect`. Последовательность `Task.model` (Opus primary → Gemini → без `model=`) — в `model-selection.mdc`.
- Обновлены [`.cursor/rules/tool-name-guard.mdc`](../rules/tool-name-guard.mdc), [`.cursor/rules/architect-gate.mdc`](../rules/architect-gate.mdc), [`.cursor/rules/1c-agent-delegation.mdc`](../rules/1c-agent-delegation.mdc), [`.cursor/skills/1c-agent-patterns/SKILL.md`](../skills/1c-agent-patterns/SKILL.md), смежные скиллы/доки с упоминанием `model` / `_2nd`.

## [4.4] - 2026-05-12

### Changed (model policy: pinned first, inherit only for architect fallback)
- Во frontmatter восстановлены **закреплённые** модели: `onec-code-architect` — `claude-opus-4-8-thinking-high`; writer/explorer — `default`; reviewer/simplifier/openspec-doc-writer — `gemini-3.1-pro`; trace-analyst и openspec-quality-controller — `inherit` (без изменения).
- **`onec-code-architect-2nd`:** только **`model: inherit`** (= модель чата) и только **после двух сбоев** основного архитектора; не первый вызов. Правило: [`.cursor/rules/model-selection.mdc`](../rules/model-selection.mdc) (FALLBACK STRATEGY).
- Синхронизированы [`.cursor/rules/architect-gate.mdc`](../rules/architect-gate.mdc), [`.cursor/skills/1c-agent-patterns/SKILL.md`](../skills/1c-agent-patterns/SKILL.md), [`openspec/project.md`](../../openspec/project.md), [`AGENTS.md`](../../AGENTS.md).

## [4.3] - 2026-05-12

### Changed (model policy: chat Auto / inherit)
- Все 1С-агенты и `openspec-doc-writer`: во frontmatter **`model: inherit`** (= модель родительского чата; см. [Cursor subagents](https://cursor.com/docs/subagents.md)). Убраны пиннинги `claude-opus-*`, `gemini-*`, `default`.
- `onec-code-architect-2nd`: `model: inherit`; удалена секция с `model="default"` в промпте; деградация модели — маркер `confidence: low` в отчёте.
- Правила: [`.cursor/rules/model-selection.mdc`](../rules/model-selection.mdc), [`.cursor/rules/tool-name-guard.mdc`](../rules/tool-name-guard.mdc), [`.cursor/rules/architect-gate.mdc`](../rules/architect-gate.mdc) — в `Task(...)` **не** передаётся `model=` (кроме явного запроса пользователя); **не** использовать `Task(model="inherit")` (невалидно для параметра инструмента в ряде сборок).
- [`.cursor/skills/1c-agent-patterns/SKILL.md`](../skills/1c-agent-patterns/SKILL.md), [`AGENTS.md`](../../AGENTS.md): таблица моделей и описание fallback синхронизированы с новой политикой.

## [4.2] - 2026-04-21

### Fixed (subagent registration / Task enum)
- `onec-code-explorer`, `onec-trace-analyst`, `openspec-quality-controller`: в frontmatter `model: default` заменён на **`model: inherit`** (допустимые значения по [Cursor subagents](https://cursor.com/docs/subagents.md)). Нестандартное `default` могло приводить к тому, что кастомные субагенты не попадали в enum инструмента `Task`, и вызов `onec-code-explorer` завершался `Invalid enum value`.

### Changed (onec-code-reviewer, bsl-antipatterns, openspec/project)
- `onec-code-reviewer`: расширение release-hygiene набора — AP-040 AI-typography, AP-044 narration, AP-045 date+time.
- `bsl-antipatterns.mdc`: v4.2, три правила добавлены, полные карточки дописаны в `.cursor/docs/antipatterns/bsl-antipatterns.md`.
- `openspec/project.md`: активированы три строки обязательного контроля комментариев.

## [4.1] - 2026-04-19

### Changed (AP-001, onec-code-reviewer, bsl-antipatterns)
- AP-001: расширена карточка — двуконтекстное использование одной модульной `Перем` (граф по директивам), маркеры false-negative в комментариях, ремедиация, контр-сигналы, второй пример BAD, ссылка на [ИТС 639](https://its.1c.ru/db/v8std/content/639/hdoc), явное «не HIGH» только для чисто серверной `Перем`; кейс: change `do2-cf-partial-repeat-params-do3-ui`
- `bsl-antipatterns.mdc`: уточнена колонка Детектирование для AP-001
- `onec-code-reviewer.md`: чеклист из 3 пунктов у строки AP-001 в category 10

## [4.0] - 2026-04-17

### Removed
- onec-form-generator: конфликт с запретом правки Form.xml (1c-xml-write-guard.mdc; правило позже слито в `1c-agent-delegation.mdc` § XML WRITE GUARD)
- onec-test-generator: автотесты в проекте не используются, 0 вызовов в changes
- onec-metadata-helper: MCP user-PROJECT-graph не развёрнут, функции поглощены onec-code-explorer
- onec-query-optimizer: 0 вызовов в changes, функции поглощены onec-code-architect и onec-code-reviewer (последний с загрузкой скилла 1c-query-optimization)

## [3.1] - 2026-03-20

### Changed (onec-code-reviewer, bsl-antipatterns)
- AP-031: мета-имена из постановки/оркестрации — доменный тест + эвристики; интеграция в Phase 0 Evaluation Checklist (вопрос 6), Medium severity, Phase 2 code cleanliness; диапазон ссылок на реестр AP-001..AP-031
- 1c-coding-standards.mdc: подсекция ИМЕНОВАНИЕ — «Доменная релевантность»
- bsl-antipatterns.mdc / bsl-antipatterns.md: полная карточка AP-031

## [3.0] - 2026-03-12

### Added (openspec-quality-controller)
- New agent: domain-agnostic OpenSpec Quality Controller (model: Opus, readonly)
- Phase classification (P0-P4), dependency graph, false start detection, rework risk assessment
- Called from `/opsx:verify` step 7.6 via `Task(subagent_type="openspec-quality-controller")`
- Replaces previous `generalPurpose` call with default model via agent file

## [2.0] - 2026-03-08

### Changed (onec-code-explorer)
- Added TASK CLASSIFICATION: Focused Investigation / Hypothesis Verification / Full Feature Exploration
- Added Phase 0: Process Caller Context (use trace-analyst findings, classify task before code reading)
- Added Extension Analysis (cf/cfe): annotation types, base contract, interplay mapping, risks
- Added Verified Facts / Hypotheses format in Output Guidance with explicit markers
- Added Hypothesis Verification Template (mini-template for hypothesis reports)
- Refined anti-pattern: "no design decisions" instead of blanket "no recommendations"; exploration facts (extension points, contracts) are allowed
- Renamed Recommendations → Extension Points / Modification Notes in output template
- Linked Report Levels to Task Classification (Compact / Hypothesis Template / Full)
- Marked Architecture Analysis and Phases 3-4 as optional (Full Exploration only)
- Replaced Example 2, added Example 3 with realistic cf/cfe scenarios

## [1.1] - 2026-02-27

### Changed
- All agents: BSL LSP marked as NOT_CONNECTED with fallback to session tools
- All agents: RLM marked as NOT_CONNECTED
- sdd-workflow.mdc: reduced to navigation document
- onec-code-explorer: added depth control, report levels, anti-patterns
- onec-code-writer: added per-invocation scope limit, idempotency
- onec-code-architect: added OpenSpec integration, plan revision, realistic testing
- onec-code-reviewer: fixed SQL->1C examples, removed aspirational integrations
- onec-code-simplifier: fixed model metadata, added pipeline integration
- 1c-dispatch-gate.mdc, 1c-agent-delegation.mdc: added Light Mode (точечная правка)

## [1.0] - 2026-02-08
- Initial version of all agents
- Source: AndreevED/1c-ai-feature-dev-workflow + improvements
