## Verify decision ledger

```yaml
closed_decisions:
  - id: engine-origin
    summary: >
      Движок универсального обмена — обработка расширения ргУниверсальныйОбменXML,
      созданная на базе штатной УниверсальныйОбменДаннымиXML ЗУП с доработками по ТЗ;
      используется на всех базах участников обмена (не V8Exch82.epf, не прямой вызов cf).
    closed_at: "2026-06-22"
    source: verify-user-answer
  - id: transport-tradeoff
    summary: >
      Транспорт GetData — SOAP + base64Binary ZIP (D7); HTTP-сервис с бинарным телом
      отклонён для v1.
    closed_at: "2026-06-22"
    source: verify-repair
  - id: as-built-metadata
    summary: >
      Источник истины имён метаданных — фактическая выгрузка src/ (рг_НастройкиОбменаXML,
      рг_УниверсальныйОбменXMLСервер, рг_ВыполнениеОбменаXML, рг_ИспользованиеОбменаXML).
    closed_at: "2026-06-22"
    source: verify-user-answer
  - id: scheduled-job-role-rights
    summary: >
      Права роли на регламентное задание не назначаются — ScheduledJob отсутствует в Rights.xml;
      управление через РегламентныеЗаданияСервер в привилегированном режиме (паттерн БСП).
    closed_at: "2026-06-22"
    source: verify-user-report
  - id: schedule-requisit-value-storage
    summary: >
      Реквизит справочника Расписание — только ХранилищеЗначения; тип РасписаниеРегламентногоЗадания
      в метаданных недопустим.
    closed_at: "2026-06-22"
    source: verify-user-report
  - id: form-schedule-data-path
    summary: >
      SUPERCEDED by schedule-hidden-object-field. РасписаниеПрофиля и ИзменитьРеквизиты отклонены на приёмке.
    closed_at: "2026-06-22"
    source: verify-user-report
  - id: schedule-hidden-object-field
    summary: >
      SUPERCEDED. DataPath Объект.Расписание (ХранилищеЗначения) на форме недоступен в Конфигураторе.
    closed_at: "2026-06-22"
    source: verify-user-report
  - id: schedule-form-requisit-rassylki
    summary: >
      Реквизит формы Расписание (произвольный Type, нехранимый) — паттерн РассылкиОтчетов;
      персистентность через ТекущийОбъект.Расписание (ХранилищеЗначения) при записи.
    closed_at: "2026-06-22"
    source: verify-user-report
  - id: schedule-no-type-name-check
    summary: >
      В BSL запрещено Тип("РасписаниеРегламентногоЗадания"); проверка через ЗначениеЗаполнено и Строка().
    closed_at: "2026-06-22"
    source: verify-user-report
decision_round: 1
open_decision_id: null
assumptions_accepted: []
last_verify_at: "2026-06-22"
last_challenge_at: "2026-06-22-2"
verify_mode: pre-apply
snapshot:
  design.md: "2026-06-22T13:09:32"
  tasks.md: "2026-06-22T13:09:37"
```

## Extend — 2026-06-22

- Источник: ответ пользователя на verify decision `engine-origin` + repair из verification-2026-06-22.md
- Изменения: design D6, Manual Configuration, tasks S1.11/S2 reorder, specs/proposal/project без V8Exch82
- Architect Gate: не требовался (уточнение зафиксированного решения verify, без смены Why/Non-Goals)
- Drift-check: pass

## Extend — verify repair 2026-06-22

- Источник: `/opsx:verify` + сообщение пользователя (нельзя указать регламентное задание в роли) + as-built выгрузка `src/`
- Изменения: S1.11 — только права на справочник, без ScheduledJob; D3/D4/Manual Configuration — as-built имена и паттерн `РегламентныеЗаданияСервер`; tasks.md синхронизирован с выгрузкой
- Architect Gate: не требовался (уточнение платформенного ограничения и code-sync имён)
- Drift-check: pass

## Apply — 2026-06-22 (session pause)

- Прогресс S1: S1.1–S1.6, S1.7–S1.9 [x]; S1.10 BSL написан, Form.xml не донастроен; S1.11 права только чтение
- Блокеры: Form.xml (реквизит `ПарольИзменен`; **не** добавлять `Расписание` типа РРЗ — только `Объект.Расписание` ХранилищеЗначения); Rights.xml (Edit/InteractiveInsert = false)

### Code-Truth — S1.7+S1.9 — 2026-06-22
- task: S1.7, S1.9
- symbols:
  - ПередЗаписью @ Catalogs/рг_НастройкиОбменаXML/Ext/ObjectModule.bsl:6-33, action=created
  - ПриЗаписи @ same:63-114, action=modified (privileged password + job sync)
  - РасписаниеРегламентногоОбменаЗаполнено @ same:145-150, action=modified
- verification: grep/read OK
- source: writer.created_or_modified_symbols

### Code-Truth — S1.8 — 2026-06-22
- task: S1.8
- symbols:
  - ВыполнитьОбменПоПрофилю @ CommonModules/рг_УниверсальныйОбменXMLСервер/Ext/Module.bsl:10-31, action=modified
- verification: grep/read OK
- source: writer.created_or_modified_symbols

### Code-Truth — S1.10 — 2026-06-22
- task: S1.10 (BSL only; Form.xml pending)
- symbols:
  - ПередЗаписьюНаСервере @ Forms/ФормаЭлемента/Ext/Form/Module.bsl:20-32, action=created
  - РасписаниеРегламентногоОбмена @ same:54-58, action=created
- verification: grep/read OK; Form.xml attributes missing — runtime blocker
- source: writer.created_or_modified_symbols

## Extend — verify repair 2026-06-22 (schedule type)

- Источник: `/opsx:verify` + реквизит Расписание только ХранилищеЗначения
- Изменения: S1.3/S1.10, design Manual Configuration, decision `schedule-requisit-value-storage`
- Drift-check: pass

### Code-Truth — S1.10 refactor — 2026-06-22
- task: S1.10
- symbols: ПолучитьРасписаниеНаСервере, УстановитьРасписаниеНаСервере @ Form/Module.bsl
- verification: reviewer PASS
- source: writer.created_or_modified_symbols

### Code-Truth — S1.10/S1.12/S1.13 — 2026-06-22
- task: S1.10, S1.12, S1.13
- symbols:
  - РасписаниеЗаполнено @ Form/Module.bsl:178-187, ObjectModule.bsl:153-162, action=created
  - ПриСозданииНаСервере, ПередЗаписьюНаСерверe, Получить/УстановитьРасписание @ Form/Module.bsl, action=modified
  - РасписаниеРегламентногоОбмена, РасписаниеРегламентногоОбменаЗаполнено @ ObjectModule.bsl, action=modified
  - ОбеспечитьРеквизитРасписаниеПрофиля, ПрочитатьРасписаниеПрофиля @ Form/Module.bsl, action=removed
- verification: grep OK (no РасписаниеПрофиля, no Тип()); reviewer PASS
- source: writer.created_or_modified_symbols

- task: S1.10, S1.12
- symbols:
  - ОбеспечитьРеквизитРасписаниеПрофиля @ Forms/ФормаЭлемента/Ext/Form/Module.bsl:279-303, action=created
  - ПрочитатьРасписаниеПрофиля @ same:309-319, action=created
  - ПолучитьРасписаниеИзОбъектаИлиЗадания @ same:325-395, action=created
  - ПриСозданииНаСервере, ПередЗаписьюНаСервере, ПолучитьРасписаниеНаСервере, УстановитьРасписаниеНаСервере @ same, action=modified
- verification: grep/read OK; no Объект.Расписание reads; reviewer PASS (2 iterations)
- source: writer.created_or_modified_symbols

### Code-Truth — S1.10/S1.12/S1.13 — 2026-06-22
- task: S1.10, S1.12, S1.13
- symbols:
  - ПриСозданииНаСерверe @ Forms/ФормаЭлемента/Ext/Form/Module.bsl — `Расписание = ПолучитьРасписаниеИзОбъектаИлиЗадания()`, action=modified
  - ПередЗаписьюНаСерверe @ same — безусловная запись ХранилищеЗначения + ДополнительныеСвойства, action=modified
  - ПолучитьРасписаниеНаСерверe, УстановитьРасписаниеНаСерверe @ same, action=modified
- verification: grep OK (0× Объект.Расписание; 0× Тип("…"); 0× РасписаниеПрофиля)
- source: writer.created_or_modified_symbols
- blocker: S1.3 — resolved 2026-06-22 (Form.xml `<Attribute name="Расписание"><Type/></Attribute>`)

### Slice S1 — Профили обмена и регламентные задания (2026-06-22, post S1.3 export)
Срез: S1 — Профили обмена и регламентные задания
Решение: awaiting-acceptance
Обоснование: S1.3 выгружен; все рабочие задачи S1 [x]; реквизит формы Расписание в Form.xml подтверждён.
Изменения tasks: S1.3 [x]
Связанный отчёт: —

### Slice S1 — Профили обмена и регламентные задания (2026-06-22, manual shortcut)
Срез: S1 — Профили обмена и регламентные задания
Решение: принят (manual shortcut)
Обоснование: пользователь подтвердил «принято» после Primary + расписание.
Изменения tasks: S1.accept [x]
Связанный отчёт: —

### Slice S2 — apply start (2026-06-22)
Срез: S2 — Сквозной обмен данными
Решение: paused (metadata Configurator)
Обоснование: в src/ нет DataProcessors и WebServices; S2.1–S2.5 — ручная настройка до BSL.
Изменения tasks: нет
Связанный отчёт: —

## Slice Gate Decisions

### Slice S1 — apply fix rejected (2026-06-22, verify)
Срез: S1
Решение: не принят (inside-slice rework)
Обоснование: «Переменная не определена (РасписаниеПрофиля)»; `Тип("РасписаниеРегламентногоЗадания")` недопустим; S1.3/S1.10/S1.12 переоткрыты, добавлены S1.13.
Изменения tasks: S1.3 [ ], S1.10–S1.13 [ ]
Связанный отчёт: reports/verification-2026-06-22-6.md

### Slice S1 — Профили обмена и регламентные задания (2026-06-22, apply fix)
Срез: S1 — Профили обмена и регламентные задания
Решение: awaiting-acceptance
Обоснование: S1.12 fix реализован — РасписаниеПрофиля + ЗначениеРеквизитаОбъекта; повторная приёмка Primary + сценарий расписания.
Изменения tasks: S1.10, S1.12, S1.3 [x]
Связанный отчёт: —

### Slice S1 — Профили обмена и регламентные задания (2026-06-22)
Срез: S1 — Профили обмена и регламентные задания
Решение: не принят (inside-slice rework)
Обоснование: при приёмке ошибка «Поле объекта не обнаружено (Расписание)» на команде расписания; добавлены S1.12, S1.10 переоткрыта.
Изменения tasks: S1.10 [ ], S1.12 добавлена
Связанный отчёт: temp/reports/bug-2026-06-22-form-raspisanie-field.md
