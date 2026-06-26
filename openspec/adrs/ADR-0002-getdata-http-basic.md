# ADR-0002: Контракт GetData и аутентификация HTTP Basic

**Статус:** Accepted
**Дата:** 2026-06-22
**Область:** exchange-web-service
**Источник:** openspec/changes/archive/2026-06-22-universal-xml-exchange/reports/architecture-new-2026-06-22.md
**Load-bearing:** yes

## Контекст

База-приёмник инициирует pull-выгрузку у источника через SOAP. Нужен стабильный контракт операции и способ передачи учётных данных.

## Решение

- Операция `GetData(ИмяМакетаПравил: String, Параметры: Structure) → base64Binary` (ZIP с XML)
- XDTO-пакет `http://v8.1c.ru/8.1/data/core`, тип `Structure`; значения — примитивы
- Учётные данные — HTTP Basic на WS-прокси, не параметры GetData

## Альтернативы

| Вариант | Плюсы | Минусы | Почему отклонён |
|---------|-------|--------|-----------------|
| Логин/пароль в теле GetData | Явно в контракте | Риск утечки в логах SOAP | Безопасность |
| JSON-строка параметров | Простота | Не нативный XDTO | Платформенный стандарт |

## Последствия

- Положительные: сжатие ZIP, типовой XDTO
- Отрицательные: требуется публикация с Basic Auth

## Связи

- **Specs:** openspec/specs/exchange-web-service/spec.md
- **Changes:** archive/2026-06-22-universal-xml-exchange/
