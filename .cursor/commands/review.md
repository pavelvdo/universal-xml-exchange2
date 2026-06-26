---
name: /review
id: review
category: Quality
description: Полное ревью кода по контексту запроса с опциональным исправлением через writer и повторным ревью
---

Провести полное подробное ревью кода в объёме по контексту запроса (модуль, файлы, расширение), затем по желанию — устранение замечаний через onec-code-writer с повторным ревью.

**Input**: Опционально — путь к модулю (.bsl), список файлов, имя расширения, имя ЗНИ или «текущий файл». Если не указано — git diff `.bsl`.

**Флаги:**
- `--full` — полное ревью файлов (отключить light-review triage).

**Предрелиз перед выкладкой:** [`/release-review`](release-review.md) — отдельная команда (Category 12, Tier 2 explorer, без light-review).

**Памятка заказчика:** [`.cursor/docs/review-guide.md`](../docs/review-guide.md) — когда `/review` vs `/release-review`, примеры, отличия от apply-reviewer.

**Первое действие:** прочитать `.cursor/skills/review/SKILL.md` с пометкой «вызов `/review` → `release_mode=false`» и далее идти по шагам skill. До прочтения скилла — никаких чтений артефактов, трасс, модулей.
