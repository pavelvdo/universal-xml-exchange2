---
verify_mode: pre-apply
change: universal-xml-exchange
date: 2026-06-22
verdict: GO
layer_status:
  layer_1_hygiene: PASS
  layer_2_internal_coherence: PASS
  layer_3_problem_solution: PASS
  layer_4_independent_challenge: APPROVE
  layer_5_implementation_readiness: PASS
snapshot:
  accepted_tasks: []
  closed_decisions:
    - engine-origin
    - transport-tradeoff
  open_decision_id: null
  decision_round: 1
  decision_round_max: 2
  verify_depth: incremental
  assumptions_accepted: []
  open_known_questions: []
  artifacts_mtime:
    design.md: "2026-06-22T13:09:32"
    tasks.md: "2026-06-22T13:09:37"
  last_challenge_at: "2026-06-22-2"
  prior_report: verification-2026-06-22.md
---

## Резюме для разработчика

universal-xml-exchange — можно запускать apply.

**Следующий шаг:** `/opsx:apply universal-xml-exchange`

Постановка согласована после решения по движку (D6: adopt `ргУниверсальныйОбменXML` из штатной `УниверсальныйОбменДаннымиXML`) и внутреннего repair: права роли разделены по срезам (S1.11 / S2.5), гейт контракта движка — S2.6 перед BSL выгрузки/загрузки, зафиксированы D7 (SOAP+base64), вспомогательные макеты adopt и митигация объёма выгрузки.

## Решения до apply

Закрыты (не пересматриваются):

- **engine-origin (D6):** движок — обработка расширения `ргУниверсальныйОбменXML` на базе cf `УниверсальныйОбменДаннымиXML`; на всех базах `Обработки.ргУниверсальныйОбменXML.Создать()`.
- **transport-tradeoff (D7):** SOAP GetData + `base64Binary` ZIP; HTTP-сервис отклонён для v1.

Открытых развилок нет.

## Что доработать в постановке

Не требуется — repair-класс закрыт в этом прогоне.

## Что меняется в постановке

**Расширение:** `src/ЗУП/cfe/рг_УниверсальныйОбменXML/`.

**Срез S1:** справочник профилей, регламент, пароль БСП, права роли на объекты S1.

**Срез S2:** adopt обработки с макетами правил и вспомогательными макетами движка, веб-сервис `GetData`, BSL выгрузки/загрузки, права роли на объекты S2, форма ручного запуска.

**Остаточный риск:** публичный API штатной обработки верифицируется на apply в S2.6 (static-сверка cf) до кода S2.7–S2.13; при расхождении — остановка среза, не silent fallback.

## Технический аудит (для движка OpenSpec)

### Слои проверки

- **Layer 1 (Гигиена артефактов):** PASS — incremental diff без регрессии чекбоксов/slice-gate.
- **Layer 2 (Internal Coherence):** PASS — наследован из прогона 1; delta не затрагивает покрытие 20/20 (`quality-control-2026-06-22.md`).
- **Layer 3 (Problem-Solution Trace):** PASS — D6/D7 согласованы с proposal/specs/tasks.
- **Layer 4 (Independent Challenge):** APPROVE после repair; отчёт: `design-challenge-2026-06-22-2.md` — gaps класса `implementation_invariant` закрыты в design/tasks.
- **Layer 5 (Implementation Readiness):** PASS после repair G5; отчёт: `architecture-task-readiness-2026-06-22-2.md` — S1.11/S2.5 split, S2.6 gate.

### Repair Loop (internal, 1 attempt)

| Блокер | Действие |
|--------|----------|
| G5 — S1.11 права на объекты S2 | S1.11 → только S1-объекты; новая S2.5 → права на обработку и веб-сервис |
| transport-tradeoff | D7 + Option C в design |
| adopt auxiliary templates | Manual Configuration + S2.2 + S2.6 gate text |
| volume mitigation | design § Risks + Engine Contract |

### Code-Truth (pre-apply)

status: WARNING — phantom symbols для новых объектов расширения ожидаемы до apply. Эталон cf `УниверсальныйОбменДаннымиXML` присутствует в `src/`.

### Precedent regression

Skipped — только ADDED specs.

## Источники

- `openspec/changes/universal-xml-exchange/reports/design-challenge-2026-06-22-2.md`
- `openspec/changes/universal-xml-exchange/reports/architecture-task-readiness-2026-06-22-2.md`
- `openspec/changes/universal-xml-exchange/reports/quality-control-2026-06-22.md`
- `openspec/changes/universal-xml-exchange/reports/verification-2026-06-22.md` (prior NO-GO)
