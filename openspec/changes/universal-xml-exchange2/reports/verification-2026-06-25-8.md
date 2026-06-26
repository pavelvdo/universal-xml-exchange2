---
verify_mode: pre-apply
verify_depth: incremental
change: universal-xml-exchange2
date: 2026-06-25
verdict: GO
layer_status:
  layer_1_hygiene: PASS
  layer_2_internal_coherence: WARNING
  layer_3_problem_solution: PASS
  layer_4_independent_challenge: SKIPPED-novelty
  layer_5_implementation_readiness: PASS
snapshot:
  accepted_tasks:
    - S1.1-S1.12
    - S1.accept
    - S2.1-S2.6
    - S2.7-S2.17
  open_tasks:
    - S2.3
    - S2.accept
    - S3.*
  last_challenge_at: "2026-06-25T16:30:00"
---

## Резюме для разработчика

universal-xml-exchange2 — можно продолжать apply и приёмку S2.

**S2.3 (выгрузка):** параметры WS добавлены — у `GetData` и `ConfirmDataReceipt` по два входных `string` в `rgExchangeService.xml`. Модуль WS синхронизирован: `GetData(pref, prefReceive)`, `ConfirmDataReceipt(pref, prefReceive)`.

**Имена параметров:** в метаданных `pref` / `prefReceive`, в design/spec — `Префикс` / `ПрефиксБазыПриемника`. Для 1С-прокси это не блокер: вызов `Прокси.GetData(ПараметрыСеанса.Префикс, …)` позиционный, значения передаются корректно. По v8std имена WS-параметров часто на английском — текущие имена допустимы. Переименование — опционально для совпадения с текстом spec.

**S2.6, форма:** без замечаний (как в прошлом verify).

**Открыто:** `S2.accept` (end-to-end тест), срез S3.

**Следующий шаг:** `/opsx:apply universal-xml-exchange2` → Primary `S2.accept`.

## Технический аудит

| Проверка | Результат |
|----------|-----------|
| rgExchangeService.xml `<Parameter>` | OK (4 параметра, 2×2 операции) |
| WS Module ↔ metadata names | OK (pref, prefReceive) |
| Server `ВызватьGetDataНаИсточнике` | OK (позиционный вызов) |
| design имена Префикс/ПрефиксБазыПриемника | WARNING — расхождение с WSDL, не блокирует 1С↔1С |
| HTTP Basic публикация | вне выгрузки — проверить на стенде |

## Источники

- `src/ЗУП/cfe/рг_УниверсальныйОбменXML/WebServices/rgExchangeService.xml`
- `WebServices/rgExchangeService/Ext/Module.bsl`
- `CommonModules/рг_УниверсальныйОбменXMLСервер/Ext/Module.bsl` — `ВызватьGetDataНаИсточнике`
