---
change: universal-xml-exchange2
date: 2026-06-29
verify_mode: pre-apply
verdict: NO-GO
verify_depth: incremental
scope_gate_decision: repair-included
trigger: symptom-source-journal-encoding
repair_attempt: 1
snapshot:
  accepted_tasks: S1-S4-work
  pending_tasks: S4.accept, S5.3, S5.accept, FU1
  last_challenge_at: 2026-06-29
  layer_status:
    layer_1_hygiene: PASS
    layer_2_internal_coherence: PASS
    layer_3_problem_solution: PASS
    layer_4_independent_challenge: SKIPPED-novelty
    layer_5_implementation_readiness: PASS
---

# Verification — «????» в журнале источника (симптом 2026-06-29)

## Резюме для разработчика

**universal-xml-exchange2 — приёмку S5 отложить: нужен S5.3.**

Симптом: в журнале источника после `[ИСТ1/ПРИЕМ1] Ошибка при загрузке правил обмена.` идут строки `??????` — фрагмент протокола движка прочитан в неверной кодировке.

**Корневая причина (verified):**
- Движок `рг_УниверсальныйОбменДаннымиXML`: при пустом `КодировкаФайлаПротоколаОбмена` запись протокола — **ANSI** (`КодировкаФайлаПротоколаОбмена()`, ObjectModule ~3100).
- Сервер: `НастроитьПротоколДвижкаОбмена` не задаёт кодировку; `КодировкаПротоколаДвижкаОбмена` при пустом свойстве читает как **UTF-8** (~2585).
- S5.1 пишет в журнал `СформироватьТекстОшибкиДвижкаОбмена` — mismatch даёт «????».

## Подправил в постановке

- `design.md` § UX — явное требование совпадения кодировки записи/чтения протокола.
- `tasks.md` — **S5.3**, расширен Primary `S5.accept` (журнал источника).

## Технический аудит

| Layer | Status |
|-------|--------|
| 1–3, 5 | PASS |
| 4 | SKIPPED-novelty (уточнение S5, не новая ось) |

## Источники

- Симптом пользователя
- `ObjectModule.bsl` движка — `КодировкаФайлаПротоколаОбмена()` default ANSI
- `рг_УниверсальныйОбменXMLСервер/Module.bsl` — `НастроитьПротоколДвижкаОбмена`, `КодировкаПротоколаДвижкаОбмена`
