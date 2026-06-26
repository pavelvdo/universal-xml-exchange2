## Code-Truth — S1.6–S1.9 — 2026-06-25
- task: S1.6–S1.9
- symbols:
  - ПередЗаписью @ src/ЗУП/cfe/рг_УниверсальныйОбменXML/Catalogs/рг_НастройкиОбменаXML/Ext/ObjectModule.bsl:6-186, annotation=none, action=modified
  - ПередУдалением @ ObjectModule.bsl:103-119, annotation=none, action=modified
  - ПриЗаписи @ ObjectModule.bsl:131-186, annotation=none, action=modified
  - ПравилаОбменаДаннымиЗаполнены @ ObjectModule.bsl:243-262, annotation=none, action=created
  - ЕстьДубльПрефиксовПрофиляИсточника @ ObjectModule.bsl:264-284, annotation=none, action=modified
- verification: grep/read OK
- source: writer.created_or_modified_symbols

### Code-Truth — S1.10 — 2026-06-25
- task: S1.10
- symbols:
  - ИсточникПриИзменении @ Form/Module.bsl, annotation=none, action=created
  - УстановитьВидимостьГруппПоРоли @ Form/Module.bsl, annotation=none, action=created
  - ЗагрузитьПравилаИзФайла @ Form/Module.bsl, annotation=none, action=modified
  - ЗагрузитьПравилаИзФайлаНаСервере @ Form/Module.bsl, annotation=none, action=created
- verification: grep/read OK
- source: writer.created_or_modified_symbols

### Code-Truth — S1.11 — 2026-06-25
- task: S1.11
- symbols:
  - ПриСозданииНаСервере @ Form/Module.bsl:5-21, annotation=none, action=modified
  - ПередЗаписьюНаСервере @ Form/Module.bsl:23-37, annotation=none, action=modified
  - ЗагрузитьПравилаИзФайлаНаСервере @ Form/Module.bsl:261-270, annotation=none, action=modified
  - ПолучитьПравилаОбменаИзОбъекта @ Form/Module.bsl:309-326, annotation=none, action=created
  - ПравилаОбменаЗагружены @ Form/Module.bsl:328-347, annotation=none, action=modified
  - ЕстьРеквизитПравилаОбменаДанными @ Form/Module.bsl, annotation=none, action=removed
- verification: grep/read OK (no Объект.ПравилаОбмена)
- source: writer.created_or_modified_symbols

### Code-Truth — S1.12 — 2026-06-25
- task: S1.12
- symbols:
  - РасписаниеЗаполнено @ Form/Module.bsl:185-194, annotation=none, action=modified
  - РасписаниеЗаполнено @ ObjectModule.bsl:225-234, annotation=none, action=modified
- verification: grep OK (no ЗначениеЗаполнено(РасписаниеЗначение))
- source: writer.created_or_modified_symbols

## Slice Gate Decisions

### Slice S1 — Модель профиля и статусы (2026-06-25)
Срез: S1 — Модель профиля и статусы
Решение: awaiting-acceptance
Обоснование: все рабочие задачи реализованы; приёмочная задача передана на ручной прогон Primary.
Изменения tasks: нет (S1.accept остаётся [ ])
Связанный отчёт: reports/handoff-acceptance-S1-2026-06-25.md

### Slice S1 — дефект приёмки (2026-06-25)
Срез: S1
Решение: не принят (inside-slice rework)
Обоснование: **verified** — `ПравилаОбменаДанными` (ХранилищеЗначения) недоступен через `Объект` на управляемой форме; S1.10 ошибочно использует `Объект.ПравилаОбменаДанными`. Эталон — реквизит формы `Расписание` + `ПередЗаписьюНаСервере` → `ТекущийОбъект`.
Изменения tasks: rework S1.10 + реквизит формы в Configurator (как S1.3)
Связанный отчёт: reports/verification-2026-06-25-2.md

### Slice S1 — повторная передача на приёмку (2026-06-25)
Срез: S1 — Модель профиля и статусы
Решение: awaiting-acceptance
Обоснование: S1.11 fix — правила через реквизит формы; все рабочие задачи S1 [x].
Изменения tasks: добавлена и закрыта S1.11
Связанный отчёт: reports/handoff-acceptance-S1-2026-06-25-2.md

### Slice S1 — дефект приёмки #2 (2026-06-25)
Срез: S1
Решение: не принят (inside-slice rework)
Обоснование: **verified** — `РасписаниеЗаполнено` вызывает `ЗначениеЗаполнено` для `РасписаниеРегламентногоЗадания` (мутабельный тип); ошибка при открытии формы в `ПриСозданииНаСервере`.
Изменения tasks: fix S1.12 — `РасписаниеЗаполнено`
Связанный отчёт: reports/verification-2026-06-25-3.md

### Slice S1 — передача на приёмку #3 (2026-06-25)
Срез: S1 — Модель профиля и статусы
Решение: awaiting-acceptance
Обоснование: S1.12 fix — РасписаниеЗаполнено в форме и объекте; все рабочие задачи S1 [x].
Изменения tasks: S1.12 закрыта (+ ObjectModule)
Связанный отчёт: reports/handoff-acceptance-S1-2026-06-25-3.md

### Slice S1 — принят (2026-06-25)
Срез: S1 — Модель профиля и статусы
Решение: принят (manual shortcut)
Обоснование: без замечаний (пользователь)
Изменения tasks: S1.accept [x]
Связанный отчёт: reports/slice-acceptance-S1-2026-06-25.md

## S2 apply — блокер S2.1 (2026-06-25)
- task: S2.1
- blocker: V8Exch82.epf отсутствует в src/; обработка ргУниверсальныйОбменXML не создана
- отчёт: reports/exploration-S2.1-V8Exch82-2026-06-25.md
- design.md Engine Contract обновлён (BLOCKER + TBD)

### Code-Truth — S2.1 — 2026-06-25
- task: S2.1
- artifact: tools/УниверсальныйОбменДаннымиXML.epf (162594 bytes)
- api_source: src/ЗУП/cf/DataProcessors/УниверсальныйОбменДаннымиXML/Ext/ObjectModule.bsl
- methods: ЗагрузитьПравилаОбмена, ИнициализироватьПервоначальныеЗначенияПараметров, УстановитьЗначениеПараметраВТаблице, ВыполнитьВыгрузку, ОткрытьФайлЗагрузки, ВыполнитьЗагрузку
- verification: design.md Engine Contract updated; tasks S2.1 [x]
- source: static cross-ref cf + user artifact

## Extend — 2026-06-25

**Источник:** verify S2 — расхождение as-built export с черновиком design (epf/макет `ДвижокОбменa` vs встроенная `рг_УниверсальныйОбменДаннымиXML`).

**Решение пользователя:** вариант 1 — оставить as-built встроенную обработку; сервер через `Обработки.рг_УниверсальныйОбменДаннымиXML.Создать()`; без макета epf и без `ргУниверсальныйОбменXML`-контейнера.

**Disposition:** accepted — пересмотр архитектуры движка в рамках v2.

**Изменено:**
- `proposal.md` — п.3 обработки
- `design.md` — D4, Engine Contract, Blast Radius, Implementation Options, Manual Configuration, Risks
- `specs/exchange-export`, `specs/exchange-import` — MODIFIED requirements по движку
- `tasks.md` — S2.2 [x], S2.2a (форма), S2.4 [x] cancelled, S2.9/S2.15/S2.17/S2.accept

**Drift-check:** drift-warning (Behavior Contract: способ создания движка; Non-Goals не затронуты; Why усилен — надёжность без внешних epf).

### Verify repair — 2026-06-25

**Источник:** `reports/design-challenge-2026-06-25-2.md`, `reports/architecture-task-readiness-2026-06-25-2.md` (implementation_invariant).

**Disposition:** repaired — copy-vs-reference (Option C), последовательность распаковки ZIP на приёмнике, де-условление S2.2a, уточнение S2.15/S2.17.

**Изменено:** `design.md` (Option C, Manual Configuration, Engine Contract import), `tasks.md` (S2.2a, S2.15, S2.17).

### Slice S2 — передача на приёмку (2026-06-25)
Срез: S2 — Полный цикл обмена
Решение: awaiting-acceptance
Обоснование: все рабочие задачи S2 [x] (включая S2.3 по выгрузке); приёмочная задача передана на ручной Primary.
Изменения tasks: S2.3 [x]
Связанный отчёт: reports/handoff-acceptance-S2-2026-06-25.md

### S2.accept — дефект «ПараметрыСеанса» (2026-06-25)
Симптом: «Выполнить обмен» → `Поле объекта недоступно для записи (ПараметрыСеанса)` (стр. 993).
Корневая причина (verified): локальное имя `ПараметрыСеанса` затеняет платформенный объект.
Исправление: `ДанныеСеансаОбмена`, `ПодготовитьДанныеСеансаПриемника` в `рг_УниверсальныйОбменXMLСервер`.
Связанный отчёт: reports/verification-2026-06-25-9.md

### S2.accept — префиксы «ПРИЕМ1» / "" (2026-06-25)
Симптом: GetData на источнике — профиль не найден по префиксам "ПРИЕМ1" / "".
Корневая причина (verified): на приёмнике пустой `ПрефиксБазыПриемника`; на форме `ГруппаПриемник` нет поля для второго префикса; валидация только для источника.
Связанный отчёт: reports/verification-2026-06-25-10.md

### UX префикса приёмника (2026-06-25, решение заказчика)
На форме приёмника поле «Префикс» = семантика `ПрефиксБазыПриемника` на источнике; as-built `DataPath` ошибочно `Объект.Префикс`.
Связанный отчёт: reports/verification-2026-06-25-11.md

### Fix: привилегированная запись статуса WS (2026-06-25)
Симптом: «Нарушение прав доступа» при `Записать` профиля в `НачатьВыгрузкуGetData` под WS-пользователем.
Исправление: `ЗаписатьСтатусПрофиляИсточника` — privileged + `ОбменДанными.Загрузка`.
Связанный отчёт: reports/verification-2026-06-25-14.md
