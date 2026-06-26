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
    - schedule-hidden-object-field
    - schedule-no-type-name-check
  open_decision_id: null
  verify_depth: full
  scope_gate_decision: user-issue-raspisanie-profilia-type-check
  prior_report: verification-2026-06-22-5
---

## Резюме для разработчика

universal-xml-exchange — можно запускать apply (повторный фикс S1).

**Следующий шаг:** `/opsx:apply universal-xml-exchange`

**Корневая причина [verified]:** подход S1.12 с `РасписаниеПрофиля` + `ИзменитьРеквизиты` не проходит проверку платформы — «Переменная не определена (РасписаниеПрофиля)»; `Тип("РасписаниеРегламентногоЗадания")` в BSL отклонён на приёмке.

**Repair posting:**
- Отменён `РасписаниеПрофиля`; решение `schedule-hidden-object-field` — скрытое поле `Объект.Расписание` в Form.xml (S1.3).
- `schedule-no-type-name-check` — проверка через `Строка()`, без `Тип("…")`.
- Переоткрыты S1.3, S1.10, S1.12; добавлены S1.13 (форма + объект).

**Порядок apply:** S1.3 (Конфигуратор + выгрузка Form.xml) → S1.12/S1.10/S1.13 (BSL).

## Источники

- Сообщение пользователя на verify / приёмке S1
- `Form/Module.bsl` — `РасписаниеПрофиля`, `Тип("РасписаниеРегламентногоЗадания")`
