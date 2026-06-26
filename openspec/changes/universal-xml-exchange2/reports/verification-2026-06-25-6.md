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
  closed_decisions:
    - id: extend-engine-2026-06-25
      summary: Встроенная рг_УниверсальныйОбменДаннымиXML в cfe
  last_challenge_at: "2026-06-25T16:30:00"
scope_gate_decision: user-clarification-manual-config
---

## Резюме для разработчика

universal-xml-exchange2 — можно продолжать apply и приёмку S2.

**S2.6 — закрыт:** в `Rights.xml` стоит `setForNewObjects=true` — роль автоматически получает права на все объекты расширения (обработка, WS). Отдельные строки `WebService.*` / `DataProcessor.*` в файле не обязательны.

**Форма — OK:** в выгрузке есть `InputField` с `DataPath` `Объект.ПрофильПриемника`.

**S2.3 — уточнение:** параметры в **модуле** WS и параметры **метаданных операции** — разные уровни. Модуль BSL достаточен для платформы; WSDL для внешнего клиента (приёмник) строится из вкладки **Параметры** операции в Конфигураторе. В `rgExchangeService.xml` у `GetData` и `ConfirmDataReceipt` по-прежнему пустые `<ChildObjects/>` — если в Конфигураторе параметры заданы, перевыгрузите WS; если нет — добавить на вкладке «Параметры» операции (не в тексте модуля).

**Самопроверка S2.3:** открыть опубликованный WSDL → операции `GetData`/`ConfirmDataReceipt` должны содержать аргументы `Префикс`, `ПрефиксБазыПриемника`. Если есть — S2.3 закрыт независимо от выгрузки в git.

**Следующий шаг:** end-to-end тест Primary `S2.accept`.

## Технический аудит

| Проверка | Результат |
|----------|-----------|
| Form.xml `ПрофильПриемника` | OK |
| Rights `setForNewObjects` | OK для runtime |
| rgExchangeService.xml параметры | Пустые ChildObjects в repo — проверить WSDL/Конфигуратор |

## Источники

- `WebServices/rgExchangeService/Ext/Module.bsl` — сигнатуры OK
- `WebServices/rgExchangeService.xml` — ChildObjects пустые
- `Roles/.../Rights.xml` — setForNewObjects=true
- Эталон параметров в метаданных: cf `WebServices/RemoteAdministrationOfExchange_2_4_5_1.xml`
