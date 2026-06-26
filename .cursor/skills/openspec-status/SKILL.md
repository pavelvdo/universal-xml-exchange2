---
name: openspec-status
description: Read-only снимок состояния OpenSpec change (срезы, отчёты, awaiting-acceptance, рекомендация следующей команды). Использовать при запросе /opsx:status или когда нужно быстро понять где change сейчас без тяжёлого verify.
license: MIT
metadata:
  author: openspec
  version: "1.0"
---

Read-only skill. Не вызывает субагентов, не правит артефакты. Задача — собрать фактические данные из `openspec/changes/<name>/` и показать компактный снимок.

**Output style:** снимок выводится по шаблону **T-STATUS** из `.cursor/docs/opsx-output-style.md` §5.4 (Шапка → **Маркеры** → Прогресс срезов → …). Для `--short` — слоты 1, 3 (одной строкой), 6.

## Входные данные

- `<change-name>` (обязательно; если не передано — AskUserQuestion по активным changes).
- Флаги: `--short` (урезанный вывод), `--reports` (только таблица отчётов).

## Шаги

1. **Validate change.**
   - Существует ли `openspec/changes/<name>/` и не в архиве (не в `openspec/changes/archive/`)?
   - Если нет — сообщение: «Change `<name>` не найден среди активных. Архивные: `openspec list --archived`.» и END.

2. **Schema & artifacts.**
   - `openspec status --change "<name>" --json` → `schemaName`, перечень артефактов (`proposal`, `design`, `specs`, `tasks`, `debug`).
   - Отметить: какие `done`, какие отсутствуют.

2b. **Маркеры (comment markers snapshot).**

   Read `proposal.md` § `## Metadata (comment markers)` + `openspec/project.md` (`defaultDeveloper`, `cfMarkerPrefix`, запреты domain_label).

   **Dual-parser:** yaml (`developer:`, `comment_suffix:`, `marker_style:`) и list (`- **developer:**`, …).

   Вывести в слот «Маркеры»:
   - `developer`, `comment_suffix` (domain_label), `marker_style` (default `canonical`)
   - **process-only:** `да` если suffix match запретам project.md § Канон domain_label; иначе `нет`
   - **marker_scope:** Grep tasks/design на `src/.../*.bsl` → `cfe` | `cf-ea` | `mixed` | `не определён`
   - **Preview transport** (date = сегодня `dd.MM.yyyy`):
     - cfe: `// +++ {developer} {date} {suffix}` / `// --- {developer}`
     - cf: `// {cfMarkerPrefix} {suffix} +++` / close по project.md
   - Ссылка: `.cursor/docs/marker-layers-guide.md`

   Если Metadata нет — «Metadata: не заполнена → `/opsx:new` или дописать proposal».

3. **Mode detection.**
   - Read `tasks.md`: Grep `^# Срез S\d+`. Если есть → **slice mode**, иначе **legacy**.
   - Для slice mode — распарсить срезы:
     - `S<N>` номер, имя из заголовка.
     - Приёмка `S<N>.accept` (или legacy `S<N>.T<M>`) — `[x]` (принят) / `[ ]` (ожидает).
     - Количество задач в срезе, сколько `[x]` / `[ ]`.
   - Для legacy — счётчики `[x]` / `[ ]` по всему tasks.md.

4. **Slice Gate Decisions.**
   - Read `debug.md` (если есть), секция `## Slice Gate Decisions`.
   - Найти последнее решение по каждому срезу; отдельно выделить `awaiting-acceptance` (требуют действия пользователя).

5. **Reports inventory.**
   - `Get-ChildItem openspec/changes/<name>/reports/*.md` — сортировать по дате имени (`YYYY-MM-DD`), группировать по типу (`verification-*`, `architecture-*`, `exploration-*`, `trace-analysis-*`, `resolved-contract-*`, `design-review-*`, `migrate-to-slices-*`, `slice-acceptance-*`, `review-*`, `prerelease-review-*`).
   - Для каждого типа — последний файл с датой.

6. **Recommend next command.**
   - Если `awaiting-acceptance` в debug.md → рекомендация: `/opsx:apply <name>` (войдёт в resume-with-pending-verdict).
   - Если slice mode и все `S<N>.accept` = `[x]` → `/opsx:archive <name>` (+ возможно `/opsx:verify <name>` если нет final verify).
   - Если есть `[ ]` в tasks и нет pre-apply verify отчёта → `/opsx:verify <name>`.
   - Если есть `[ ]` и есть pre-apply verify → `/opsx:apply <name>`.
   - Если artifacts не `done` (нет tasks.md) → `/opsx:new <name>` (resume).
   - Иначе — общая рекомендация с перечнем вариантов.

7. **Output (полный формат, без `--short`).**

   ```
   ## OpenSpec Status — <change-name>
   
   **Schema:** <schemaName>
   **Режим verify:** pre-apply | post-apply (есть ли незакрытые `[ ]` в tasks.md)
   **Структура:** срезы | legacy
   **Прогресс:** K/M задач; принятых срезов: X/Y (slice mode)
   
   ### Срезы (slice mode)
   | Срез | Название | Задачи | Приёмка | Статус |
   |------|----------|--------|---------|--------|
   | S1   | <имя>    | 3/3    | S1.accept [x] | принят |
   | S2   | <имя>    | 2/4    | S2.accept [ ] | в работе |
   | S3   | <имя>    | 0/3    | S3.accept [ ] | ожидает |
   
   ### Slice Gate Decisions (последние)
   - S1: принят (2026-04-17)
   - S2: awaiting-acceptance (2026-04-18) ← требует действия
   
   ### Последние отчёты
   | Тип | Дата | Файл |
   |-----|------|------|
   | verification (pre-apply) | 2026-04-17 | reports/verification-2026-04-17.md |
   | architecture | 2026-04-15 | reports/architecture-2026-04-15.md |
   | ... |
   
   ### Рекомендация
   Следующая команда: `/opsx:apply <change-name>` — есть awaiting-acceptance в debug.md, войдёт в resume-with-pending-verdict.
   
   Альтернативы: `/opsx:verify <change-name>` (в запросе указать проблемный срез S2 или причину приёмки), `/opsx:explore` → `/opsx:extend <change-name> --from-report` (если возник дефект при приёмке S2).
   ```

   **Формат `--short`:** одна строка
   ```
   <name> | post-apply, срезы (S1 принят, S2 awaiting, S3 ожидает) | 5/10 задач | next: /opsx:apply
   ```

   **Формат `--reports`:** только таблица последних отчётов.

## Ограничения

- Не читать сами `.bsl` файлы — только артефакты change и метаданные.
- Не запускать subagents.
- Если `debug.md` / `reports/` отсутствуют — показать секции с пометкой `—`.
