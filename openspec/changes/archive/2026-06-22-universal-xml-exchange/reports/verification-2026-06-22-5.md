---
verify_mode: pre-apply
change: universal-xml-exchange
date: 2026-06-22
verdict: GO
layer_status:
  layer_1_hygiene: PASS
  layer_2_internal_coherence: PASS
  layer_3_problem_solution: PASS
  layer_4_independent_challenge: SKIPPED-novelty
  layer_5_implementation_readiness: PASS
snapshot:
  accepted_tasks: [S1.1, S1.2, S1.4, S1.5, S1.6, S1.7, S1.8, S1.9, S1.11]
  closed_decisions:
    - engine-origin
    - transport-tradeoff
    - as-built-metadata
    - scheduled-job-role-rights
    - schedule-requisit-value-storage
    - form-schedule-data-path
  open_decision_id: null
  verify_depth: full
  scope_gate_decision: user-issue-form-raspisanie-field
  prior_report: verification-2026-06-22-4
---

## Резюме для разработчика

universal-xml-exchange — можно запускать apply (фикс внутри S1).

**Следующий шаг:** `/opsx:apply universal-xml-exchange`

**Корневая причина [verified]:** `Form/Module.bsl` обращается к `Объект.Расписание`, но в `Form.xml` нет `DataPath>Объект.Расписание` — реквизит не входит в данные управляемой формы → «Поле объекта не обнаружено (Расписание)» в `ПолучитьРасписаниеНаСервере`.

**Repair posting:**
- `S1.10` переоткрыта; добавлена `S1.12` (fix модуля формы + реквизит `РасписаниеПрофиля` на форме).
- `S1.3` переоткрыта (нужен реквизит формы `РасписаниеПрофиля` — ХранилищеЗначения).
- design + decision ledger: `form-schedule-data-path`.
- Slice S1: не принят, inside-slice rework.

**Apply:** сначала S1.3 (реквизит `РасписаниеПрофиля` в Конфигураторе + выгрузка), затем S1.12/S1.10 (BSL), повторная приёмка `S1.accept`.

## Технический аудит

| Слой | Статус |
|------|--------|
| Layer 1–3 | PASS после repair |
| Layer 4 | SKIPPED-novelty (defect fix в S1) |
| Layer 5 | PASS — задачи S1.12 исполнимы |

## Источники

- `temp/reports/bug-2026-06-22-form-raspisanie-field.md`
- `Form.xml` — нет `Объект.Расписание` в DataPath
- `Form/Module.bsl:185` — `Объект.Расписание`
