---
name: 1c-agent-patterns
description: Reference guide for 1C agent delegation patterns - complexity assessment, role-based prompt templates (per-role files), shared instruction blocks (gates), specialized skill integration. Use when planning agent delegation for 1C development tasks.
---

# 1C Agent Patterns (навигатор)

Справочник делегирования задач 1С-агентам. Не workflow — справочник для использования в OpenSpec explore/apply.

Шаблоны промптов разнесены по ролевым файлам. Этот файл — навигатор: оценка сложности, маршрутизация по ролям, выбор модели, общие блоки инструкций (Shared instruction blocks), batch-операции, обработка ошибок и пороги делегирования.

---

## РОЛЕВЫЕ МОДУЛИ (где какой шаблон)

| Агент | Файл | Шаблоны |
|---|---|---|
| `onec-code-architect` | [architect.md](architect.md) | проектирование, ревью плана, глубокий анализ, scope-coherence audit, task readiness review (verify Layer 5), slice transition review (slice-gate, пересмотр следующего среза), slice restructuring (migrate-to-slices), slice decomposition, slice-aware task decomposition, fix quality review (debug 5.5), ADR extraction, multisampling arbiter, ТЗ quality review |
| `onec-code-writer` | [writer.md](writer.md) | реализация задачи, bug fix, review fix |
| `onec-code-reviewer` | [reviewer.md](reviewer.md) | ревью кода, ревью bug fix |
| `onec-code-explorer` | [explorer.md](explorer.md) | исследование кода, верификация гипотез trace-analyst, contract resolution (deep) |
| `openspec-quality-controller` | [quality-controller.md](quality-controller.md) | slice coherence review (verify Layer 2) |
| `onec-code-simplifier` | [simplifier.md](simplifier.md) | рефакторинг (REFACTOR-замечания) |
| `onec-trace-analyst` | (нет шаблона; контракт в `.cursor/agents/onec-trace-analyst.md`) | бриф готовится в чате (Ultra-Lite explore — см. `openspec-explore/SKILL.md`, шаг «Маршрут шагов»); полный отчёт → `temp/reports/trace-analysis-*.md` |

---

## ОЦЕНКА СЛОЖНОСТИ

| Сложность | Признаки | Агенты |
|-----------|----------|--------|
| **Простая** | 1-2 файла, очевидная реализация, нет архитектурных решений | 1 explorer, 1 writer, 1 reviewer |
| **Средняя** | 3-5 файлов, требует понимания архитектуры, несколько модулей | 1-2 explorer, 1 architect, 1 writer, 1 reviewer |
| **Сложная** | 5+ файлов, несколько подсистем, интеграции | 1-2 explorer, architect (deep analysis), architect (design), writer + reviewer |
| **Критичная** | Архитектурные изменения, высокие риски, много зависимостей | 2-4 explorer (итеративно), architect (deep analysis), 2 architect (design, мультисэмплинг), writer + reviewer |

---

## КОГДА КАКОЙ АГЕНТ

### onec-code-explorer
- Контекст из 3+ модулей
- Понять потоки вызовов и зависимости
- Найти паттерны в существующем коде
- Подготовка к архитектурному решению
- Резолв контрактов возврата (Investigation Request от reviewer)

### onec-code-architect
- Проектирование плана реализации (design.md)
- Выбор подхода (минимальные изменения / чистая архитектура / баланс)
- Ревью design.md перед реализацией
- Ревью соответствия кода дизайну (для средних/сложных)
- Мультисэмплинг: запустить 2-3 с разными подходами, выбрать лучший
- Slice decomposition / task decomposition
- Fix quality review при срабатывании Architect Gate (debug)
- ADR extraction при archive

### onec-code-writer
- Реализация задач из tasks.md
- Исправление замечаний reviewer
- Только существующие .bsl файлы
- Bug fix: обязательно получает root cause контекст (шаблон «Writer — bug fix»)

### onec-code-reviewer
- Обязательно после каждого writer
- mode=prerelease при `/release-review`
- Категории: performance, security, БСП, аннотации, структура модуля

### onec-trace-analyst
- Файл трассы (PFF/TRACE) или стек ошибки 3+ строк
- Перед вызовом: подготовить бриф.
  В extend — обогащённый бриф из артефактов change (`design.md`, `debug.md`) или из `temp/reports/<тип>-*.md` / `temp/explore-handoff-*.md`.
  В explore — бриф B3 из пользовательского ввода и слота **Вопрос** в чате (`user-goal` в промпте; `openspec-explore/SKILL.md`).

### openspec-quality-controller
- Domain-agnostic readonly Slice Coherence review
- Вызывается из `/opsx:verify` Layer 2 (MANDATORY)

### onec-code-simplifier
- REFACTOR-замечания из ревью (запахи кода, читаемость)
- Behavior Preservation — поведение не меняется
- Вызывается из `/review`

---

## ВЫЗОВ АГЕНТОВ

Вызов субагентов — через инструмент **Task** (формат, допустимые `subagent_type`, поведение при `Invalid enum value` — `.cursor/rules/tool-name-guard.mdc`). Подробностей здесь не дублируем.

**Fallback для архитектора (модель):** один агент `onec-code-architect`; цепочка `Task.model` и финальный вызов без `model=` — в `.cursor/rules/model-selection.mdc` (раздел «Цепочка для архитектора»).

---

## ВЫБОР МОДЕЛИ (все 1С-агенты)

**Единый SSOT:** [`.cursor/rules/model-selection.mdc`](../../rules/model-selection.mdc) — таблица ролей, `Task.model` для Primary и Fallback, ограничение enum, запрет `Task(model="inherit")`.

**Кратко:** во frontmatter всех кастомных агентов **`model: inherit`**. Оркестратор **передаёт** `model=<slug>` из таблицы при каждом вызове (кроме `onec-trace-analyst`, `openspec-quality-controller` и финального шага fallback — без `model=`). Переопределение вне таблицы — только по явному запросу пользователя.

**Не дублировать** полную таблицу здесь без пометки «синхронизировать с `model-selection.mdc`».

Детали вызова `Task` — `.cursor/rules/tool-name-guard.mdc`.

---

## SHARED INSTRUCTION BLOCKS

Именованные блоки инструкций для вставки в промпты 1С-агентов. Оркестратор копирует текст блока в промпт агента. Единая точка редактирования — изменение блока автоматически обновляет все шаблоны (writer, reviewer, simplifier, architect, explorer).

### DATA_CONTRACT_GATE

```
Data contract gate: перед добавлением Свойство/ТипЗнч/ЗначениеЗаполнено —
HALT. Не знаешь тип — прочитай метаданные (XML в src/) или документацию
функции. Не можешь определить сам — спроси пользователя. Не добавляй
проверку вместо выяснения типа (антипаттерн «компенсация незнания»).
Проверку добавлять только при подтверждённой опциональности поля/типа.
Свойство() — ИСКЛЮЧИТЕЛЬНО Структура / ФиксированнаяСтруктура; на любом
другом типе (ссылка, объект, строка ТЗ) — runtime-ошибка. Defensive cake
(стек 2+ проверок на одном значении, где одна поглощается другой) запрещён
при ЛЮБОМ контракте (fixed и dynamic). См. rule 14 в .cursor/docs/1c-coding-standards.md.
```

### INTEGRATION_CONTRACT_GATE

```
Integration contract gate (scope: отдельный вызов функции/процедуры):
перед добавлением логики вокруг вызова существующей функции
(конвертация, валидация, проверки типов, обработка ошибок) — HALT,
проверить контракт вызываемой функции. Если функция уже обрабатывает
эти случаи — не дублировать. Перед реализацией в новом месте —
проверить, можно ли решить расширением существующей функции.
```

### EXISTING_MECHANISM_GATE

```
Existing mechanism gate (scope: архитектура — объекты, регистры,
процессы, подсистемы): перед предложением нового объекта, модуля,
workflow, хранилища или API — HALT. Сначала проверить, как эта задача
уже решается в базе: какие штатные механизмы, точки расширения, регистры,
документы, процессы и экспортные функции уже существуют.
Если механизм найден — обосновать, почему он не подходит именно для
этой задачи (конкретные ограничения, а не «так чище»).
Preference Hierarchy:
1) использовать существующий механизм как есть,
2) расширить существующий,
3) адаптировать паттерн базы,
4) создать альтернативный механизм.
Уровни 3-4 допустимы только с документированным обоснованием.
```

### EXISTING_KNOWLEDGE

```
Existing knowledge: если во входном промпте есть секция `## Existing Knowledge`,
каждый KB-NNNN использовать как верифицированный факт (active) или как факт,
требующий переподтверждения (stale).

В отчёте обязательно сослаться на каждый KB по ID:
- used — факт использован в выводах;
- not relevant — факт оказался вне scope, указать почему;
- conflict — код противоречит active KB, добавить секцию `## Knowledge conflicts`.

Не переоткрывать факты, для которых KB уже есть, без явной причины
(например, behavioral-drift при чтении anchor-файла).
```

### EXTENSION_GUARD

```
EXTENSION GUARD (&ИзменениеИКонтроль):
HALT перед записью в метод с &ИзменениеИКонтроль.
Каждая НОВАЯ строка — ОБЯЗАТЕЛЬНО внутри #Вставка/#КонецВставки.
Каждая удаляемая типовая строка — внутри #Удаление/#КонецУдаления.
Код вне директив — ПОБИТОВО совпадает с типовым, НЕПРИКОСНОВЕНЕН.
Нарушение = поломка расширения.
```

### CONTEXT_SAFETY

```
В процедурах расширения с &После/&Перед на сервере НЕ обращаться
к клиентским переменным формы (Перем ... &НаКлиенте) — они не определены
в серверном контексте. Источник данных в ПриСозданииНаСервере — только Параметры.
```

### CONDITIONAL_ACTION_GATE

```
Conditional action gate: если вся оставшаяся логика процедуры — одно
действие или компактный блок, оборачивай в Если <условие> Тогда ...
КонецЕсли. НЕ пиши Если НЕ <условие> Тогда Возврат; КонецЕсли;
<действие> — это инвертированный ранний выход, условие привязано
к выходу, а не к действию. Ранний выход (guard clause) допустим
только как предусловие В НАЧАЛЕ процедуры, после которого идёт
основная логика из нескольких действий.
См. rule 23 в .cursor/docs/1c-coding-standards.md.
```

### ORCHESTRATOR_IMPLEMENTATION_GATE

```
Orchestrator implementation gate: если промпт описывает конкретный
паттерн реализации (Попытка, проверки, fallback, структуру кода) —
это ПОДСКАЗКА, не директива. Перед реализацией подсказанного паттерна:
1. Попытка — rule 20 (внешний фактор? fallback не silent degradation?)
2. Guard/проверка — rule 14 (контракт фиксирован? есть ли defensive cake?)
3. Fallback — не создаёт ли неотличимый от успеха результат (AP-009)?
Если подсказка нарушает стандарт — НЕ реализовывать, обосновать отказ.
```

---

## ПРИ ОШИБКЕ ВЫЗОВА АГЕНТА

### Шаг 1: Проверить правильность вызова

Ошибка `Invalid enum value` означает вызов НЕ ТОГО инструмента.
Проверь: инструмент = **Task**, subagent_type написан правильно (например `onec-code-writer`).
Если ошибка в имени инструмента или опечатка — исправить и повторить.

### Шаг 2: Retry

Если имя инструмента верное — повторить вызов Task 1 раз.

### Шаг 3: СТОП

Если retry не помог — **СТОП**. Сообщить пользователю:
- какой агент не запустился
- точную ошибку
- что уже пробовали (имя инструмента, subagent_type)

**НЕ переключаться на generalPurpose.** Специализированные агенты штатно доступны. Переключение маскирует ошибку и снижает качество результата.

---

## ИНТЕГРАЦИЯ СО СПЕЦИАЛИЗИРОВАННЫМИ СКИЛЛАМИ

При реализации задач из tasks.md определить тип и загрузить скилл:

| Тип задачи | Скилл | Что делает |
|------------|-------|------------|
| Управляемая форма | `1c-forms/SKILL.md` | info / validate / patterns; изменение формы — Конфигуратор + выгрузка или BSL модуля формы |
| Печатная форма | `1c-mxl/SKILL.md` | compile → validate |
| Настройка прав | `1c-roles/SKILL.md` | compile из JSON DSL |
| Интеграция с БСП | `1c-bsp/SKILL.md` | registration + commands + patterns |
| Расширения (аннотации) | `1c-extensions/SKILL.md` | выбор аннотации, перенос правок |
| Оптимизация запросов | `1c-query-optimization/SKILL.md` | temp tables, JOIN, индексы |
| Общая BSL-задача | onec-code-writer напрямую | |

---

## BATCH OPERATIONS (10+ файлов, одинаковая правка)

Когда задача затрагивает 10+ файлов с одинаковым паттерном (например добавление `#Область`, удаление маркеров чейнджлога):

1. **Образец:** сделать правку в первых 2–3 файлах как образец (через writer или Mechanical Mode).
2. **Одна Task на пачку:** вызвать `Task(subagent_type="onec-code-writer")` на оставшиеся файлы, передав образец (путь к одному изменённому файлу) + список файлов + чёткий критерий (что вставить/удалить).
3. **Post-verification:** Grep по всем файлам на ожидаемый паттерн (например наличие `#Область` в начале модуля); spot-check минимум 3 файла (первый, средний, последний).

Если правка чисто механическая (find-replace без вариаций) — предпочтительно Mechanical Mode (StrReplace + Grep) вместо агента на 10+ файлах.

---

## АВТО-ИСПРАВЛЕНИЕ ЗАМЕЧАНИЙ РЕВЬЮ

При получении замечаний от reviewer:

1. **Все кодовые замечания** (critical, high, medium, low) → автоматически `onec-code-writer` с замечаниями → повторный reviewer. Максимум 2 итерации.
2. **Архитектурные замечания** (новый объект, изменение API, структуры хранения, прав/RLS) → СТОП, к пользователю.

Принцип: ревьювер не выдаёт необязательных замечаний. Каждое замечание = обязательное к исправлению.

---

## DELEGATION GATE

Порог и детали — `1c-agent-delegation.mdc`, секция DELEGATION GATE.

Ключевое правило: **до вызова агента допустимо ≤3 обращения к .bsl**. Сформулировать промпт из артефактов + пути к файлам → вызвать Task tool, агент сам читает код.

---

## EXPLORE MODE

В режиме explore (opsx-explore) аналитические агенты работают как обычно:

- Трасса/стек → trace-analyst → explorer/architect
- Код 3+ модулей → explorer
- Архитектурный вопрос → architect

**Реализационные агенты (writer, reviewer) в explore НЕ запускаются.**

---

**Source**: Extracted from 1c-feature-dev-enhanced v2.2 (Phase 0-8 workflow replaced by OpenSpec). Split into role files at workflow-orphan-cleanup_d12f0402.
