---
verify_mode: pre-apply
change: universal-xml-exchange2
date: 2026-06-25
verdict: GO
layer_status:
  layer_1_hygiene: PASS
  layer_2_internal_coherence: PASS
  layer_3_problem_solution: PASS
  layer_4_independent_challenge: CHALLENGE-saturated
  layer_5_implementation_readiness: WARNING
snapshot:
  accepted_tasks: []
  closed_decisions: []
  open_decision_id: null
  decision_round: 0
  decision_round_max: 2
  verify_depth: full
  assumptions_accepted: []
  open_known_questions:
    - Контракт публичного API V8Exch82.epf уточняется в задаче S2.1 до кода выгрузки/загрузки
  artifacts_mtime:
    proposal.md: "2026-06-25T12:00:00"
    design.md: "2026-06-25T14:00:00"
    tasks.md: "2026-06-25T14:00:00"
  last_challenge_at: "2026-06-25T14:00:00"
repair_attempt: 1
---

## Резюме для разработчика

universal-xml-exchange2 — можно запускать apply.

**Следующий шаг:** `/opsx:apply universal-xml-exchange2`

План вводит статусный профиль обмена в `рг_НастройкиОбменаXML`, двухфазный контракт `GetData`/`ConfirmDataReceipt` в `rgExchangeService` и движок `V8Exch82.epf` в обработке `ргУниверсальныйОбменXML`. Срез S1 — модель и форма; S2 — полный цикл на приёмнике; S3 — отказные пути и журнал.

Подправил в постановке: идемпотентность `ConfirmDataReceipt` (повтор при «Выполнено» — успех), порядок проверок в `ПередЗаписью`, уточнение уникальности префиксов, дополнена таблица ручной настройки.

К сведению: перед кодом S2 нужны файл `V8Exch82.epf` в макете и static-сверка API в S2.1; supersede ADR-0002/0003 задокументирован в Blast Radius.

## Что меняется в постановке

Расширение `src/ЗУП/cfe/рг_УниверсальныйОбменXML/`: перечисление `рг_СтатусыОбмена`, новые реквизиты справочника, веб-сервис, обработка с epf, доработки `рг_УниверсальныйОбменXMLСервер` и модуля объекта справочника. Отменяет сигнатуру GetData и движок из ADR-0002/0003 — осознанно, с Blast Radius.

## Подправил в постановке

- `design.md` / `exchange-web-service` spec / `S3.4`: идемпотентный `ConfirmDataReceipt` при статусе «Выполнено»
- `design.md` D6: порядок `ПередЗаписью` (проверки до рег. задания)
- `S1.8`: уникальность префиксов только среди профилей-источников
- `## Manual Configuration`: форма элемента, роль, форма запуска, профиль безопасности

### К сведению

- S2.1 блокирует S2.9/S2.15 до фиксации API epf — остаточный риск проверяется в S2.accept
- Архивный change `universal-xml-exchange` — другая модель (имя макета, cf-движок)

## Технический аудит (для движка OpenSpec)

| Layer | Status |
|-------|--------|
| 1 Hygiene | PASS |
| 2 Internal Coherence | PASS (QC OK, precedent-documented) |
| 3 Problem-Solution | PASS |
| 4 Independent Challenge | CHALLENGE → repair closed gaps → CHALLENGE-saturated |
| 5 Implementation Readiness | WARNING (S2.1 epf API assumption) |

## Источники

- `reports/quality-control-2026-06-25.md`
- `reports/design-challenge-2026-06-25.md`
- `reports/architecture-task-readiness-2026-06-25.md`
- `reports/architecture-new-2026-06-25.md`
