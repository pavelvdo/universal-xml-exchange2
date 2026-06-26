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
  accepted_tasks: []
  closed_decisions:
    - engine-origin
    - transport-tradeoff
    - as-built-metadata
    - scheduled-job-role-rights
  open_decision_id: null
  decision_round: 1
  decision_round_max: 2
  verify_depth: full
  assumptions_accepted: []
  open_known_questions: []
  artifacts_mtime:
    design.md: "2026-06-22-repair"
    tasks.md: "2026-06-22-repair"
  last_challenge_at: "2026-06-22-2"
  prior_report: verification-2026-06-22-2
  scope_gate_decision: user-issue-scheduled-job-rights
---

## Резюме для разработчика

universal-xml-exchange — можно запускать apply.

**Следующий шаг:** `/opsx:apply universal-xml-exchange`

Вы правы: в Конфигураторе **нельзя** выдать роли права на регламентное задание — тип `ScheduledJob` не участвует в правах роли (в типовой ЗУП нет ни одной записи `ScheduledJob.*` в Rights.xml). Это платформенное ограничение, не ошибка конфигурации.

Постановка исправлена: задача S1.11 теперь требует права **только** на справочник `рг_НастройкиОбменаXML`; создание и синхронизация задания `рг_ВыполнениеОбменаXML` — в коде S1.9 через `РегламентныеЗаданияСервер` в привилегированном режиме (как в `Справочники.РассылкиОтчетов`).

Дополнительно синхронизированы имена метаданных с вашей выгрузкой (`рг_НастройкиОбменаXML`, реквизиты `АдресСервиса`/`Логин`/…, роль `рг_ИспользованиеОбменаXML`).

**Перед apply:** в Конфигураторе по S1.11 выдайте роли `рг_ИспользованиеОбменаXML` права **записи** на справочник (сейчас в выгрузке только чтение) и выгрузите `Rights.xml` снова. Пункт про регламентное задание **пропустите**.

## Подправил в постановке

- S1.11: убрано невыполнимое «право на регламентное задание»; добавлен инвариант привилегированного режима.
- D3/D4, Manual Configuration, tasks.md: as-built имена из `src/ЗУП/cfe/рг_УниверсальныйОбменXML/`.
- debug.md: зафиксированы решения `as-built-metadata`, `scheduled-job-role-rights`.

## Технический аудит (для движка OpenSpec)

| Слой | Статус | Комментарий |
|------|--------|-------------|
| Layer 1 | PASS | без автоправок |
| Layer 2 | PASS | QC-3: slice coherence OK; C1/C2 закрыты repair |
| Layer 3 | PASS | Why→Requirements→Slices согласованы после sync |
| Layer 4 | SKIPPED-novelty | design delta — repair, не новый adversarial прогон |
| Layer 5 | PASS | task-readiness-3 блокеры закрыты repair |

## Источники

- `reports/quality-control-2026-06-22-3.md`
- `reports/architecture-task-readiness-2026-06-22-3.md`
- Grep `ScheduledJob\.` в `src/ЗУП/cf/Roles/**` — 0 совпадений
- `src/ЗУП/cfe/рг_УниверсальныйОбменXML/Roles/рг_ИспользованиеОбменаXML/Ext/Rights.xml`
