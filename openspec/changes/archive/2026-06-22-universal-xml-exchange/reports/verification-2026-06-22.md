---
verify_mode: pre-apply
change: universal-xml-exchange
date: 2026-06-22
verdict: NO-GO
layer_status:
  layer_1_hygiene: PASS
  layer_2_internal_coherence: PASS
  layer_3_problem_solution: PASS
  layer_4_independent_challenge: CHALLENGE
  layer_5_implementation_readiness: FAIL
snapshot:
  accepted_tasks: []
  closed_decisions: []
  open_decision_id: engine-origin
  decision_round: 0
  decision_round_max: 2
  verify_depth: full
  assumptions_accepted: []
  open_known_questions:
    - OQ1 — публичный контракт V8Exch82.epf
    - engine-origin — выбор движка выгрузки/загрузки
    - transport-tradeoff — обоснование SOAP+base64 vs HTTP-сервис
  artifacts_mtime:
    proposal.md: "2026-06-22T11:22:04"
    design.md: "2026-06-22T11:29:19"
    tasks.md: "2026-06-22T11:35:17"
    specs/exchange-settings/spec.md: "2026-06-22T11:23:36"
    specs/exchange-export/spec.md: "2026-06-22T11:23:37"
    specs/exchange-import/spec.md: "2026-06-22T11:23:40"
    specs/exchange-web-service/spec.md: "2026-06-22T11:23:44"
  last_challenge_at: "2026-06-22T11:29:19"
---

## Резюме для разработчика

universal-xml-exchange — до старта нужен ваш выбор по логике движка универсального обмена: внешний `V8Exch82.epf` в макете или штатная обработка `УниверсальныйОбменДаннымиXML` из типовой ЗУП.

**Следующий шаг:** ответьте в чате (A или B). После фиксации в постановке — снова `/opsx:verify universal-xml-exchange`.

План в целом согласован: два среза (профили S1 → сквозной обмен S2), 20 сценариев из спеки покрыты задачами, User Task Contract чист. Блокеры: не зафиксирован выбор движка (в cf уже есть штатная обработка КД 2.1), нет задачи на права роли `обменXML_ОсновнаяРоль`, в `design.md` нет сводной таблицы ручной конфигурации, задача S2.9 (сверка API движка) стоит после S2.6, которая этот API использует.

## Решения до apply

### Развилка 1. Происхождение движка выгрузки/загрузки

**Цель ЗНИ:** универсальный pull-обмен между ИБ по правилам в макетах с загрузкой на приёмнике штатными механизмами универсального обмена. Обе ветки закрывают цель; различается способ вызова движка и профиль рисков совместимости.

**Что в коде сейчас.** В выгрузке типовой ЗУП (`src/ЗУП/cf/DataProcessors/УниверсальныйОбменДаннымиXML`) уже есть штатная обработка универсального обмена КД 2.1. Расширение `рг_УниверсальныйОбменXML` — каркас без объектов подсистемы. План предполагает упаковку внешнего `V8Exch82.epf` в макет `ДвижокОбмена` обработки `ргУниверсальныйОбменXML`.

**Что предлагает план.** Извлечь движок из макета во временный файл, создать через `ВнешниеОбработки.Создать`, выгрузить/загрузить XML. Контракт API движка (OQ1) отложен на apply в S2.9, хотя S2.6/S2.12 уже опираются на него.

**Почему это развилка.** Независимый аудит нашёл в cf штатную обработку, которая снимает заявленный в design риск несовместимости `V8Exch82.epf` с платформой, но в `Implementation Options` этот путь не сравнивался с текущим выбором.

**Варианты решения.**

- **A. Оставить `V8Exch82.epf` в макете `ДвижокОбмена`** — фиксированная версия движка, одинаковый бинарник на источнике и приёмнике независимо от состава cf; **компромисс:** риск несовместимости с платформой остаётся на стороне проекта, нужна сверка API в S2.9 до кода S2.6.
- **B. Вызывать штатную `Обработки.УниверсальныйОбменДаннымиXML` из cf** — без файла и бинарника в макете, совместимость с текущей ЗУП гарантирована составом конфигурации; **компромисс:** зависимость от API типовой обработки при обновлениях ЗУП, на приёмнике без этой обработки в составе cf потребуется иной контур (переносимость на «любую БСП» — отдельное решение).

**Влияет на:** задачи S2.1–S2.3 (макеты), S2.6–S2.12 (BSL выгрузки/загрузки), формулировки spec exchange-export/import, риск №1 в design § Risks.

**Что изменится после выбора.** Выбор зафиксируют в `design.md` § Decisions и скорректируют tasks/spec при необходимости; затем repair допишет права роли, таблицу ручной конфигурации и порядок S2.9.

**Источники (техническое):** design-challenge-2026-06-22.md; architecture-task-readiness-2026-06-22.md (G2); `src/ЗУП/cf/DataProcessors/УниверсальныйОбменДаннымиXML.xml`.

## Что доработать в постановке

### Рекомендации (repair-класс, после решения по движку)

- **Права роли:** добавить задачу ручной конфигурации — выдать права `обменXML_ОсновнаяРоль` на справочник, обработку, веб-сервис, регламентное задание (сейчас роль пустая, DefaultRoles расширения).
- **Manual Configuration:** в `design.md` — сводная таблица реквизитов/ТЧ/элементов формы/параметров веб-сервиса (сейчас детали только в tasks).
- **Порядок S2.9:** перенести верификацию контракта V8Exch82 (или штатной обработки) **перед** S2.6–S2.8.
- **Транспорт:** дополнить `Implementation Options` сравнением SOAP+base64 vs HTTP-сервис относительно риска таймаута на крупных выгрузках (можно repair без смены axis, если SOAP подтверждён).

## Что меняется в постановке

**Расширение:** `src/ЗУП/cfe/рг_УниверсальныйОбменXML/`.

**Точки изменения:**

- `ргНастройкиОбменаXML` — справочник профилей (URL, логин, макет правил, ТЧ параметров, регламент).
- `ргУниверсальныйОбменXMLСервер` — пароль через БСП, регламент, сеанс обмена, выгрузка/загрузка.
- `rgExchangeService.GetData` — SOAP, XDTO Structure → Base64 ZIP.
- `ргУниверсальныйОбменXML` — обработка с макетами движка и правил, форма ручного запуска.

**Что НЕ меняется:** типовой обмен данными ЗУП / EnterpriseData; автоматический поиск ссылок по примитивам (ответственность правил КД 2.1).

## Технический аудит (для движка OpenSpec)

### Слои проверки

- **Layer 1 (Гигиена артефактов):** PASS — чекбоксы и slice-gate на месте.
- **Layer 2 (Internal Coherence):** PASS; QC: `quality-control-2026-06-22.md` (20/20 scenarios, OK).
- **Layer 3 (Problem-Solution Trace):** PASS — Why покрыт requirements; все requirements имеют scenarios; срезы S1/S2 с accept.
- **Layer 4 (Independent Challenge):** CHALLENGE; отчёт: `design-challenge-2026-06-22.md` — engine-origin + transport-tradeoff.
- **Layer 5 (Implementation Readiness):** FAIL; отчёт: `architecture-task-readiness-2026-06-22.md` — G1 (role rights), G3 (S2.9 order), G4 (manual-config-incomplete).

### Авто-исправлено (Layer 1)

Не применялось.

### Code-Truth (pre-apply)

status: WARNING — phantom symbols для новых объектов расширения ожидаемы до apply. БСП API (`ОбщегоНазначения`, `РегламентныеЗаданияСервер`) подтверждены в `src/ЗУП/cf/`.

### Precedent regression

Skipped — только ADDED specs, архивов exchange-* нет.

## Источники

- `openspec/changes/universal-xml-exchange/reports/quality-control-2026-06-22.md`
- `openspec/changes/universal-xml-exchange/reports/design-challenge-2026-06-22.md`
- `openspec/changes/universal-xml-exchange/reports/architecture-task-readiness-2026-06-22.md`
- Алерты: `manual-config-incomplete`, `engine-origin`, `task-ordering-forward-ref`, `phantom-symbol` (pre-apply WARNING)
