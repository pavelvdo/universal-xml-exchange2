# AGENTS.md — навигационный индекс

**Главный диспетчер:** `.cursor/rules/1c-agent-delegation.mdc` — HALT-условия, делегирование агентам, BSL/XML write guard.
**Диспетчер on-demand гейтов:** `.cursor/rules/gate-dispatcher.mdc` — подгружает архитектурные и верификационные гейты по триггерам.

## OpenSpec Workflow

`.cursor/rules/sdd-workflow.mdc` — explore → new → verify → apply → verify → archive.

Команды: `/opsx:intake`, `/opsx:explore`, `/opsx:debug`, `/opsx:new`, `/opsx:verify`, `/opsx:apply`, `/opsx:archive`, `/opsx:extend`, `/opsx:status`, `/opsx:knowledge-add`, `/opsx:knowledge-init`, `/opsx:knowledge-audit`, `/opsx:sync`, `/opsx:bulk-archive`, `/review`, `/release-review`, `/init-project`.

Устаревшие алиасы: `/opsx:ff` и `/opsx:continue` → `/opsx:new <name>` (stub-redirect, новых сценариев не несут).

### Decision tree команд

Термины workflow — в таблице ниже и в [`openspec/project.md`](openspec/project.md).

| Задача пользователя | Команда | Чем отличается |
|---------------------|---------|----------------|
| Сырой текст/скрины заказчика, неясная постановка | `/opsx:intake` | Нормализация в Intake Brief; маршрут в explore/debug/new без чтения кода и трасс |
| Любой вопрос, дефект, идея, постановка (в т.ч. свободный текст) | `/opsx:explore` | Единая точка входа; бриф-чекпойнт. Итог: дефект — блок `## Постановка ЗНИ` в чате; вопрос — свод «Итог / Вердикт / Дальше»; фича без исследования — redirect на `/opsx:new` |
| Дефект в контексте ЗНИ (трасса, RCA, debug.md) | `/opsx:debug` | RCA в артефактах change; без правки BSL оркестратором |
| Создать или дозавершить change | `/opsx:new <name>` | Все артефакты сразу; повторный вызов — resume |
| Где я в этом change | `/opsx:status <name>` | Read-only снимок |
| Можно ли запускать apply | `/opsx:verify <name>` | Pre-flight + self-repair; один вердикт в первой строке. Опционально `/opsx:verify <name> --lite` — только исполнимость без повторного независимого аудита (guardrails в SKILL verify). |
| Добавить новое требование | `/opsx:extend <name>` | Бриф → правки → подсказка verify |
| Учесть отчёт ревью/архитектора | `/opsx:extend <name> --from-review` / `--from-architecture` | Классификация findings → правки артефактов |
| Код упростили вручную, артефакты отстали | `/opsx:extend <name> --code-sync` | explorer читает факт → артефакты догоняют |
| Реализовать задачи | `/opsx:apply <name>` | Делегирует writer/reviewer; пауза на приёмке среза |
| Архивировать завершённый change | `/opsx:archive <name>` | Финализация + извлечение ADR/KB |
| Ревью кода (точечное) | `/review` | Git diff, файл, расширение, ЗНI; light-review на тривиальном diff. Памятка: [`.cursor/docs/review-guide.md`](.cursor/docs/review-guide.md) |
| Предрелизное ревью | `/release-review …` | Все `.bsl` расширения или change-scoped; Category 12; Tier 2 explorer; без light-review. Примеры — там же |
| Зафиксировать факты вне ЗНИ | `/opsx:knowledge-add <path>` | Без ЗНИ; source + KB-карточка |
| Создать/обновить таксономию KB | `/opsx:knowledge-init` | Bootstrap или идемпотентный re-sync `_taxonomy.yaml` |
| Ревизия базы знаний | `/opsx:knowledge-audit` | TTL, якоря, индекс, метрики, taxonomy-sync |
| Синхронизировать delta specs в main | `/opsx:sync` | Перенос specs из change без archive |
| Архивировать несколько change | `/opsx:bulk-archive` | Пакетная архивация завершённых ЗНИ |
| Первичная настройка проекта | `/init-project` | Phase 0–4 inline-протокол; без отдельного SKILL |

## Карта SSOT (один якорь на тему)

**Чат и стиль:**
- Лимиты, non-events, HALT-жаргон, 5 принципов диалога, роль навигатора → `.cursor/rules/chat-output-budget.mdc`.
- Бриф = Sync Card (карточка слотов B0–B3) → `.cursor/docs/templates/brief-card.md`; классификатор → `.cursor/docs/opsx-output-style.md` §5.1.
- Язык, тон, шаблоны вывода, Chat Surface Contract (§2.6) → `.cursor/docs/opsx-output-style.md`.
- Словарь запретов (SSOT) → `.cursor/docs/chat-lexicon.md`; каталог AI-tells → `.cursor/skills/stop-slop/SKILL.md`.

**Делегирование и код 1С:**
- HALT (always-apply stub), делегирование, APPLY/XML write guard → `.cursor/rules/1c-agent-delegation.mdc`.
- HALT-таблица, Light/Mechanical, исключения → `.cursor/rules/1c-halt-triggers.mdc` (on-demand, globs `**/*.bsl`).
- LINT GATE (полный текст), writer pipeline → `.cursor/rules/1c-writer-pipeline.mdc` (globs `**/*.bsl`).
- Запрет создания метаданных → `.cursor/rules/1c-no-metadata-creation.mdc`; валидация имён метаданных по выгрузке `src/` до использования → `.cursor/rules/1c-metadata-validation.mdc` (globs `**/*.bsl`).
- Инструмент субагентов = Task → `.cursor/rules/tool-name-guard.mdc`; модели → `.cursor/rules/model-selection.mdc`.
- Анализ ошибок (трасса → trace-analyst) → `.cursor/rules/1c-error-analysis.mdc`.
- Утилитарные агенты, формы → `.cursor/rules/1c-utility-agents.mdc`, `.cursor/skills/1c-forms/SKILL.md`.
- Паттерны промптов агентов → `.cursor/skills/1c-agent-patterns/SKILL.md`; промпты → `.cursor/agents/*.md` (changelog `.cursor/agents/CHANGELOG.md`).

**Сессии и стратегия:**
- Skill-first, persistence, context-strategy → `.cursor/rules/session-discipline.mdc`.
- Прямое чтение vs субагенты → `.cursor/skills/context-strategy/SKILL.md`.
- Сохранение отчётов субагентов → `.cursor/rules/preserve-subagent-reports.mdc`.

**Гейты качества:**
- Архитектурное ревью (триггеры, Simplicity Check) → `.cursor/rules/architect-gate.mdc`.
- Root cause + impact перед фиксом → `.cursor/rules/verified-cause-gate.mdc`.
- Приоритет существующих механизмов → `.cursor/rules/existing-mechanism-priority.mdc`.
- Code-Truth (phantom-symbol) → `.cursor/rules/code-truth-gate.mdc`; precedent/regression → `.cursor/rules/precedent-regression-gate.mdc`.
- Delta specs → `.cursor/rules/openspec-specs-gate.mdc`; срезы → `.cursor/rules/vertical-slices.mdc`.
- Антипаттерны BSL (reviewer-only) → `.cursor/rules/bsl-antipatterns.mdc`; стандарты → `.cursor/docs/1c-coding-standards.md`, `.cursor/skills/1c-vendor-standards/SKILL.md`.

**Знания и решения:**
- ADR → `openspec/adrs/` + `.cursor/rules/adr-format.mdc`.
- Knowledge Base → `openspec/knowledge/` + `.cursor/rules/knowledge-format.mdc`.
- Маркеры разработчика → `openspec/project.md`; фиксация договорённостей → `.cursor/rules/capture-to-project.mdc`.
- Пути к выгрузке (cf/cfe) → `openspec/project.md` + `.cursor/rules/project-paths.mdc`.
- Запрет ROI/оценок → `.cursor/rules/no-roi-estimates.mdc`.
- Инфраструктура 1С → `.cursor/docs/onec-infrastructure.md`.

**Ревью кода:** памятка заказчика → [`.cursor/docs/review-guide.md`](.cursor/docs/review-guide.md); команды → `/review`, `/release-review`; протокол → `.cursor/skills/review/SKILL.md`.

**Доменные навыки 1С:** `1c-bsp`, `1c-extensions`, `1c-forms`, `1c-mxl`, `1c-roles`, `1c-query-optimization` — через `available_skills` и `.cursor/skills/*/SKILL.md`. Справочники: `.cursor/docs/platform/`, `.cursor/docs/standard/`.
