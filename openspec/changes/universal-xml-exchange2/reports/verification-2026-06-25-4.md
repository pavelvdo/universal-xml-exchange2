---
verify_mode: pre-apply
verify_depth: incremental
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
  accepted_tasks:
    - S1.1-S1.12
    - S1.accept
    - S2.1
    - S2.2
    - S2.4
  closed_decisions:
    - id: extend-engine-2026-06-25
      summary: Встроенная рг_УниверсальныйОбменДаннымиXML в cfe; без epf/макета ДвижокОбмена
      closed_at: "2026-06-25"
      source: verify-user-answer
  open_decision_id: null
  decision_round: 1
  decision_round_max: 2
  assumptions_accepted: []
  open_known_questions:
    - Ошибка на приёмнике после GetData — источник остаётся «Выполняется» до confirm (design Q1)
  artifacts_mtime:
    proposal.md: "2026-06-25T16:00:00"
    design.md: "2026-06-25T16:30:00"
    tasks.md: "2026-06-25T16:30:00"
  last_challenge_at: "2026-06-25T16:30:00"
repair_attempt: 1
scope_gate_decision: verify-only
---

## Резюме для разработчика

universal-xml-exchange2 — можно запускать apply.

**Следующий шаг:** `/opsx:apply universal-xml-exchange2`

S1 принят. Срез S2: веб-сервис `rgExchangeService` (`GetData`, `ConfirmDataReceipt`), серверный модуль `рг_УниверсальныйОбменXMLСервер` (выгрузка/загрузка через `Обработки.рг_УниверсальныйОбменДаннымиXML.Создать()`), форма `УправляемаяФорма` с `ПрофильПриемника` и командой «Выполнить обмен». Движок — as-built в cfe, без внешних epf.

Подправил в постановке: обоснование копии движка vs вызов cf; на приёмнике ZIP распаковывает серверный модуль, в движок передаются пути к XML (не `.zip`); уточнены S2.2a, S2.15, S2.17 (форма `УправляемаяФорма`).

К сведению: перед end-to-end тестом S2 — создать WS в Конфигураторе (S2.3), опубликовать на источнике, доработать форму (S2.2a).

## Что меняется в постановке

| Область | Файлы / объекты |
|---------|-----------------|
| Источник | `rgExchangeService`, `рг_УниверсальныйОбменXMLСервер` — фазы GetData, ZIP, статусы |
| Приёмник | `рг_УниверсальныйОбменXMLСервер`, форма `УправляемаяФорма` — сеанс по профилю |
| Движок | `DataProcessors/рг_УниверсальныйОбменДаннымиXML` — as-built, API verified |
| Не меняется | S1 справочник, перечисление, форма элемента профиля |

ADR-0003: согласован через встроенную копию; supersede ADR-0002 по сигнатуре GetData — в Blast Radius.

### Подправил в постановке

- `design.md` — Option C (почему не cf напрямую), drift mitigation, контракт распаковки ZIP на приёмнике.
- `tasks.md` — S2.2a без условия «если отсутствует»; S2.15/S2.17 с формой `УправляемаяФорма`.

### К сведению

- ID задачи `S2.2a` — буквенный суффикс; не блокирует apply.
- Primary S2 предполагает публикацию WS на сервере — шаг вне tasks, в design § Manual Configuration.

## Технический аудит (для движка OpenSpec)

| Layer | Status | Notes |
|-------|--------|-------|
| 1 Hygiene | PASS | |
| 2 Coherence | PASS | QC OK; phantom rgExchangeService pre-apply WARNING |
| 3 Problem-Solution | PASS | Why ↔ specs ↔ slices |
| 4 Challenge | CHALLENGE-saturated | Gaps 1–2 closed by repair; reopen cf fork filtered |
| 5 Readiness | WARNING | GAP-1/2 repaired in tasks; SUBOPTIMAL graft on типовую форму |

### Repair (attempt 1)

| Gap | Class | Action |
|-----|-------|--------|
| Copy-vs-reference | implementation_invariant | design Option C + drift mitigation |
| ZIP double-unpack | implementation_invariant | Engine Contract + S2.15 |
| S2.2a conditional | implementation_invariant | tasks de-conditioned |
| S2.17 form path | implementation_invariant | УправляемаяФорма explicit |

## Источники

- `reports/quality-control-2026-06-25-2.md`
- `reports/design-challenge-2026-06-25-2.md`
- `reports/architecture-task-readiness-2026-06-25-2.md`
