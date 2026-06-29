---
change: universal-xml-exchange2
date: 2026-06-29
verify_mode: pre-apply
verdict: NO-GO
verify_depth: incremental
scope_gate_decision: repair-included
trigger: symptom-ws-error-unreadable-receiver
repair_attempt: 1
snapshot:
  accepted_tasks: S1-S3
  pending_tasks: S4.accept, S5.1-S5.2, S5.accept, FU1
  last_challenge_at: 2026-06-29
  layer_status:
    layer_1_hygiene: PASS
    layer_2_internal_coherence: PASS
    layer_3_problem_solution: PASS
    layer_4_independent_challenge: SKIPPED-novelty
    layer_5_implementation_readiness: PASS
---

# Verification — нечитаемая ошибка WS на приёмнике (симптом 2026-06-29)

## Резюме для разработчика

**universal-xml-exchange2 — приёмку S4 отложить: сначала срез S5.**

На скриншоте — полная SOAP-цепочка со стеком (`…Модуль(1935)`) и «????» после «Ошибка при загрузке правил обмена». Это не соответствует design § UX ошибок WS: пользователю нужен только бизнес-заголовок.

**Корневая причина (verified по коду):**
1. **Источник:** `ВызватьИсключениеПриОшибкеДвижкаОбмена` бросает `СформироватьТекстОшибкиДвижкаОбмена` (заголовок + фрагмент протокола с возможной неверной кодировкой) — уходит в SOAP fault.
2. **Приёмник:** `ТекстОшибкиВебСервисаДляПользователя` берёт листовое `.Описание` без обрезки обёрток «При вызове веб-сервиса» / «по причине».

## Подправил в постановке (repair verify)

- `design.md` § UX — источник отдаёт в SOAP только заголовок; приёмник отбрасывает обёртки и «?».
- `specs/exchange-import/spec.md` — THEN: читаемая кириллица, без протокола.
- `tasks.md` — срез **S5** (S5.1, S5.2, S5.accept).
- `design.md` § Slices — строка S5, матрица import → S5.

## Что решить

Приёмку S4 («принято») не фиксировать, пока не пройден Primary S5 (тот же сценарий, что на скриншоте).

## Технический аудит

| Layer | Status |
|-------|--------|
| 1 Hygiene | PASS |
| 2 Coherence | PASS (симптом ↔ gap S3.7 vs design) |
| 3 Problem-Solution | PASS |
| 4 Challenge | SKIPPED-novelty (уточнение UX, не смена оси) |
| 5 Readiness | PASS (S5.1–S5.2 READY) |

## Источники

- Симптом пользователя (скриншот 2026-06-29)
- `Module.bsl` — `ВызватьИсключениеПриОшибкеДвижкаОбмена` (~1935), `ТекстОшибкиВебСервисаДляПользователя` (~803)
- `design.md` § UX ошибок WS
