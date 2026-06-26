---
name: /release-review
id: release-review
category: Quality
description: Предрелизное ревью расширения или change — Category 12, эскалация severity, Tier 2 explorer
---

Предрелизный контроль перед выкладкой: все `.bsl` расширения или change-scoped Tier 1 + архитектурный обзор всего расширения (Tier 2). Делегирование **onec-code-reviewer** с `mode=prerelease`.

**Режим (фиксированно):** `release_mode = true` — см. шаг 0 в [`.cursor/skills/review/SKILL.md`](../skills/review/SKILL.md).

**Input** (обязателен хотя бы один аргумент — расширение и/или change):

| Вызов | Scope |
|-------|-------|
| `/release-review <расширение>` | Все `.bsl` в cfe расширения + Tier 2 explorer |
| `/release-review <расширение> <change>` | Change-scoped Tier 1 + Tier 2 по всему расширению |
| `/release-review <change>` | Change-scoped; cfe из путей в артефактах change |

Имя расширения — папка в `src/*/cfe/<имя>/` (см. [`openspec/project.md`](../../openspec/project.md)).

**Памятка заказчика:** [`.cursor/docs/review-guide.md`](../docs/review-guide.md) — когда `/review` vs `/release-review`, отличия от apply-reviewer.

**Первое действие:** прочитать `.cursor/skills/review/SKILL.md` с пометкой «вызов `/release-review` → `release_mode=true`» и далее идти по шагам skill. До прочтения скилла — никаких чтений артефактов, трасс, модулей.
