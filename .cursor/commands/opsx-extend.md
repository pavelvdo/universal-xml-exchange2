---
name: /opsx:extend
id: opsx-extend
category: Workflow
description: Контролируемое расширение scope существующего change (новые задачи/требования с обновлением артефактов и проверкой)
---

Контролируемо расширить scope активного change: пользователь описывает **новое требование**, команда обновляет `proposal.md` / `design.md` / `specs/` / `tasks.md` (при необходимости — с привязкой к существующим или новому срезу), фиксирует в `debug.md` секцию расширения и возвращает управление в `/opsx:verify` для валидации изменённого scope.

Отделена от `/opsx:verify` — verify read-only для user-path; **repair-from-verify** — internal only (verify Repair Loop, без чата).

**Режимы:**
- **user-extend** — бриф + confirm → handoff на языке эффекта → hint `/opsx:verify <name>` (не auto-chain).
- **repair-from-verify** — internal из verify; **без** сообщений в чат; вызывается только из Repair Loop verify SKILL.

**Первое действие:** прочитать `.cursor/skills/openspec-extend-change/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких других чтений артефактов, отчётов, трасс или модулей.

**Input:**
- `<change-name>` — обязательно.
- Текст расширения в сообщении пользователя (или AskQuestion при отсутствии).
- Опционально — ссылка на файл/отчёт, который нужно проанализировать как основание для правки ЗНИ:
  - `@path/to/file.md`
  - `--from-review <path>` — отчёт `/review`
  - `--from-report <path>` — итог `/opsx:explore`: `temp/reports/<тип>-YYYY-MM-DD-<slug>.md` (полный отчёт `Task`: trace-analysis, exploration, architecture), `temp/explore-handoff-*.md` (опциональный handoff с блоком `## Постановка ЗНИ`), или legacy `openspec/sessions/<slug>/analysis.md`. Основной путь capture fix.
  - `--from-debug <path>` — устаревший alias: `debug.md` или RCA в change (предпочтительно `--from-report`)
  - `--from-verify <path>` — отчёт `/opsx:verify`
  - `--from-architecture <path>` — отчёт архитектора
  - `--from-explore <path>` — legacy-источники, только по явной ссылке пользователя (`temp/explore-summary-*.md`, `openspec/sessions/<slug>/analysis.md`)
  - `--code-sync` — код упростили/поменяли вручную, артефакты отстали: explorer читает факт кода → артефакты догоняют (режим в SKILL extend)

**Примеры:**

- `/opsx:extend do2-roli-avtopodstanovka-gate --from-review openspec/changes/do2-roli-avtopodstanovka-gate/reports/review-do2-roli-avtopodstanovka-gate-2026-04-29-subagent-raw.md "Пересмотреть решение по представлению роли"`
- `/opsx:extend add-auth --from-report temp/reports/trace-analysis-2026-04-29-token-rotation.md "Добавить требование по ротации токенов"`
- `/opsx:extend add-auth @temp/explore-handoff-2026-04-29-token-rotation.md "Учесть найденный риск"`

**Поведение (кратко):**
1. Прочитать `openspec/changes/<name>/` (proposal, design, specs, tasks).
2. Прочитать явно переданные файлы/отчёты и извлечь из них факты, findings, recommendations, open questions.
3. Показать бриф **B1/B2** (карточка `brief-card.md`, §5.1 Sync Card). «Соответствие исходному scope», `Drift-check`, план правки артефактов — **internal** (`debug.md`), не в чат. B2: **Варианты** нумерованно, ответ по номеру. При drift — Scope Coherence Audit (`architecture-extend-coherence-*.md`) после подтверждения. До подтверждения — никаких правок.
4. Проанализировать новое требование: относится ли к существующему сценарию (Requirement / Scenario в spec) или требует нового.
5. Обновить артефакты:
   - proposal.md — добавить в scope (секция `## Цель` или `## Scope`).
   - specs/*/spec.md — добавить Requirement/Scenario (через openspec delta-формат, см. `openspec-specs-gate.mdc`).
   - design.md — дополнить `## Slices` (новый срез `S<N+1>`, если новое требование не укладывается в существующие).
   - tasks.md — добавить задачи по правилам vertical-slices.mdc «ПОВЕДЕНИЕ NEW (resume)» (вставка в существующий срез или новый срез).
6. Зафиксировать в `debug.md` секцию `## Extend — YYYY-MM-DD`: источник, что добавлено, disposition.
7. **user-extend:** hint `/opsx:verify <name>` одной строкой (язык эффекта, без перечня файлов). **repair-from-verify:** шаг 7 **не** выводится в чат.

**Ограничения:**
- Не вызывает writer/reviewer — реализация остаётся за `/opsx:apply`.
- Не применяется к архивным change.
- При добавлении задач в **уже принятый** срез (`S<N>.accept` = `[x]`, legacy `S<N>.T<M>`) — предупреждение: «Потребуется повторная приёмка; `S<N>.accept` будет открыт заново.»
