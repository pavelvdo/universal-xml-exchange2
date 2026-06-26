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
  accepted_tasks: [S1.1, S1.2, S1.3, S1.4, S1.5, S1.6, S1.7, S1.8, S1.9]
  closed_decisions:
    - engine-origin
    - transport-tradeoff
    - as-built-metadata
    - scheduled-job-role-rights
    - schedule-requisit-value-storage
  open_decision_id: null
  decision_round: 1
  verify_depth: full
  scope_gate_decision: user-issue-schedule-requisit-type
  prior_report: verification-2026-06-22-3
---

## Резюме для разработчика

universal-xml-exchange — можно запускать apply.

**Следующий шаг:** `/opsx:apply universal-xml-exchange`

Вы правы: реквизит справочника `Расписание` — только **ХранилищеЗначения**. В as-built выгрузке это уже так (`v8:ValueStorage` в `рг_НастройкиОбменаXML.xml`). Тип `РасписаниеРегламентногоЗадания` в метаданных справочника или формы создавать нельзя.

Паттерн (как в object module S1.9 и типовой БСП):
- хранение: `Объект.Расписание = Новый ХранилищеЗначения(РасписаниеРегламентногоЗадания)`;
- чтение: `Объект.Расписание.Получить()` → локальная переменная для диалога `ДиалогРасписанияРегламентногоЗадания`.

На форме добавить только **`ПарольИзменен` (Булево)** и событие `ПарольПриИзменении`. Отдельный реквизит формы `Расписание` типа РРЗ — **не нужен**.

**Apply:** модуль формы (черновик S1.10) ссылается на реквизит формы `Расписание` — при apply агент переведёт на работу через `Объект.Расписание` (ХранилищеЗначения).

**Остаётся вручную:** S1.11 — права записи на справочник в `рг_ИспользованиеОбменаXML`.

## Подправил в постановке

- S1.3, S1.10, design § Manual Configuration, decision ledger: `schedule-requisit-value-storage`.
- Убрана ошибочная инструкция «добавить на форму Расписание типа РРЗ».

## Технический аудит

| Слой | Статус |
|------|--------|
| Layer 1 | PASS |
| Layer 2 | PASS — контракт D3 согласован с as-built |
| Layer 3 | PASS |
| Layer 4 | SKIPPED-novelty (repair по user-issue) |
| Layer 5 | PASS — S1.10 уточнён, apply исправит черновик формы |

## Источники

- `src/ЗУП/cfe/рг_УниверсальныйОбменXML/Catalogs/рг_НастройкиОбменаXML.xml` — `Расписание` → `v8:ValueStorage`
- `src/ЗУП/cf/Catalogs/РассылкиОтчетов/Ext/ObjectModule.bsl` — синхронизация РЗ + `ДополнительныеСвойства.Расписание`
