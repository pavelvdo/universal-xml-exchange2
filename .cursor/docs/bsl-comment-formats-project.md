# Соглашения по комментариям BSL — навигация

Таблица **Whitelist** в [openspec/project.md](../../openspec/project.md) (секция «Форматы и соглашения по комментариям BSL») — **project-level override** для rule 17 в `.cursor/docs/1c-coding-standards.md` и release-hygiene в `.cursor/agents/onec-code-reviewer.md`: комментарии в whitelist **exempt от удаления** (AP-040); **содержимое** проверяет AP-053.

**Источник истины:** [openspec/project.md](../../openspec/project.md) — `defaultDeveloper`, `cfMarkerPrefix`, whitelist, канон `domain_label`, MARKER-MERGE-001.

**Обзор четырёх слоёв (metadata → SSOT → transport → BSL):** [marker-layers-guide.md](marker-layers-guide.md). Снимок по change: `/opsx:status <name>`.

Фреймворк (`.cursor/skills`, `.cursor/agents`) использует плейсхолдеры `{developer}`, `{cfMarkerPrefix}`, `<ФИО>` — значения читаются из project.md и proposal.md.

## Whitelist (колонки)

| Колонка | Смысл |
|--------|--------|
| **Префикс после `//`** | После `//` и пробелов комментарий начинается с подстроки → строка в whitelist. |
| **Regex на всю строку `//…`** | Вся строка комментария удовлетворяет шаблону → whitelist. |

## Обязательный контроль (колонки)

| Колонка | Смысл |
|--------|--------|
| **Где проверять** | Scope: перехваты, первый комментарий после `#Вставка`, весь модуль и т.д. |
| **Regex допустимой строки `//…`** | Для mandatory control в `/review` и `/release-review`: первая значимая строка `//` после `#Вставка` должна совпадать; иначе — замечание. |
| **Уровень / kind** | Попадает в отчёт и в tasks при создании change. |
