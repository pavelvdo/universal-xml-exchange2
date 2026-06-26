---
change: universal-xml-exchange2
date: 2026-06-26
verify_mode: pre-apply
verdict: GO
verify_depth: full
scope_gate_decision: repair-included
trigger: S3.7-ws-error-ux
repair_attempt: 1
snapshot:
  accepted_tasks: S1-S3.6
  pending_tasks: S3.7, S3.accept, FU1
  last_challenge_at: null
  layer_status:
    layer_1_hygiene: AUTOFIXED
    layer_2_internal_coherence: PASS
    layer_3_problem_solution: PASS
    layer_4_independent_challenge: SKIPPED-novelty
    layer_5_implementation_readiness: PASS
---

# Verification — UX ошибок веб-сервиса на форме приёмника

## Резюме для разработчика

**universal-xml-exchange2 — можно запускать apply.**

По вашему симптому (полный стек вместо «Выгрузка данных недоступна…») в постановку добавлена задача **S3.7**: в `рг_УниверсальныйОбменXMLСервер` извлекать бизнес-текст из цепочки причин WS/SOAP и отдавать его в `СообщениеОбОшибке` для `ПоказатьПредупреждение` на форме обработки; полный текст — только в журнал. As-built подтверждён: `ПодробноеПредставлениеОшибки` попадает и в журнал, и пользователю.

Остаётся реализовать S3.7, затем ручная приёмка S3.accept.

## Что меняется в постanовке

| Артефакт | Изменение |
|----------|-----------|
| `tasks.md` | S3.7 `[ ]`; Primary S3 расширен UX-проверкой |
| `design.md` | § UX ошибок веб-сервиса на приёмнике |
| `specs/exchange-import/spec.md` | Scenario «Понятное сообщение об ошибке веб-сервиса» |

**Код (ожидается в apply):** `CommonModules/рг_УниверсальныйОбменXMLСервер/Ext/Module.bsl` — функция извлечения текста + правка `ВыполнитьСеансОбменаПоПрофилюПриемника` (разделить полный текст для журнала и краткий для пользователя). Форму менять не нужно.

### Подправил в постановке

- Добавлены S3.7, design § UX, spec-сценарий и шаг Primary acceptance S3 (repair verify).

### К сведению

- Для повтора обмена после ошибки выгрузки профиль-источник нужно вернуть в статус «Новое» вручную.

## Технический аудит (для движка OpenSpec)

| Layer | Status |
|-------|--------|
| 1 Hygiene | AUTOFIXED (Primary metadata S3) |
| 2 Coherence | PASS (QC OK, 25/25 scenarios) |
| 3 Problem-Solution | PASS |
| 4 Challenge | SKIPPED-novelty (delta UX, без смены D2/D3) |
| 5 Readiness | PASS (S3.7 READY) |

## Источники

- `reports/quality-control-2026-06-26.md`
- `reports/architecture-task-readiness-2026-06-26.md`
