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
    - schedule-form-requisit-rassylki
    - schedule-no-type-name-check
    - schedule-requisit-value-storage
  open_decision_id: null
  verify_depth: full
  scope_gate_decision: user-valuestorage-form-blocked
  prior_report: verification-2026-06-22-6
---

## Резюме для разработчика

universal-xml-exchange — можно запускать apply.

**Следующий шаг:** `/opsx:apply universal-xml-exchange`

**Ограничение платформы [verified]:** на управляемую форму **нельзя** вывести/привязать реквизит `Объект.Расписание` типа **ХранилищеЗначения** (ни видимый, ни скрытый DataPath).

**Решение (паттерн `РассылкиОтчетов`):**
1. **S1.3** — реквизит **формы** `Расписание` (произвольный тип, `<Type/>` в Form.xml, нехранимый, не на layout).
2. **S1.10–S1.13** — модуль формы работает с переменной `Расписание`; в `ПередЗаписьюНаСерверe`: `ТекущийОбъект.Расписание = Новый ХранилищеЗначения(Расписание)` и `ДополнительныеСвойства.Расписание`.
3. Текущий код с `Объект.Расписание` в форме — **не соответствует** (переоткрыты S1.10–S1.13).

**Конфигуратор (S1.3):** Реквизиты формы → добавить `Расписание` → тип **не фиксировать** (произвольный) → не выводить на форму → выгрузить.

## Источники

- Сообщение пользователя на verify
- `cf/Catalogs/РассылкиОтчетов/Forms/ФормаЭлемента/Ext/Form.xml` — `<Attribute name="Расписание"><Type/></Attribute>`
