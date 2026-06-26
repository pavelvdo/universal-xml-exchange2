---
report_type: architecture
generated_at: 2026-06-22
agent: onec-code-architect
mode: plan-review
review_mode: peer
scope:
  change: universal-xml-exchange
  files:
    - openspec/changes/universal-xml-exchange/proposal.md
    - openspec/changes/universal-xml-exchange/design.md
    - openspec/changes/universal-xml-exchange/specs/exchange-settings/spec.md
    - openspec/changes/universal-xml-exchange/specs/exchange-web-service/spec.md
    - openspec/changes/universal-xml-exchange/specs/exchange-export/spec.md
    - openspec/changes/universal-xml-exchange/specs/exchange-import/spec.md
  ext: src/ЗУП/cfe/рг_УниверсальныйОбменXML/
verdict: approve-with-changes
confidence: medium
open_questions_count: 2
---

# Architecture Review: Универсальный обмен XML между ИБ 1С

## Verdict

**approve-with-changes.** Архитектура целостна, переиспользует проверенные механизмы (БСП безопасное хранилище и регламентные задания — подтверждены в коде `cf`), Implementation Options сравнены, выбран минимальный монолитный модуль. Перед apply необходимо закрыть документационные пробелы: контракт публичного API движка V8Exch82, SOAP-сериализация XDTO `Structure`, транспорт аутентификации и жизненный цикл временных файлов.

## Simplicity Check

- **Selected simplest viable design:** Option A — монолитный `ргУниверсальныйОбменXMLСервер`.
- **Why not simpler:** регламентное задание и веб-сервис требуют серверной точки входа без формы.

## Found Patterns (verified)

### Pattern 1: БСП безопасное хранилище паролей
- **Where:** `src/ЗУП/cf/CommonModules/ОбщегоНазначения/Ext/Module.bsl` — `ЗаписатьДанныеВБезопасноеХранилище`, `ПрочитатьДанныеИзБезопасногоХранилища`.

### Pattern 2: БСП регламентные задания
- **Where:** `src/ЗУП/cf/CommonModules/РегламентныеЗаданияСервер/Ext/Module.bsl` — `ДобавитьЗадание`, `ИзменитьЗадание`, `УдалитьЗадание`.

## Recommended design.md edits

- D5: XDTO data/core, Structure, base64Binary, HTTP-basic на WS-прокси
- Behavior Contract: временные файлы, `ВнешниеОбработки.Создать` из макета
- Форма обработки для ручного запуска
