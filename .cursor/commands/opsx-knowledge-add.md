---
name: /opsx:knowledge-add
id: opsx-knowledge-add
category: Workflow
description: Добавить verified KB-факты из отчётов или файлов вне ЗНИ
---

Добавить verified-факты в `openspec/knowledge/` из исследовательских отчётов или ручных markdown-заметок вне ЗНИ.

**Первое действие:** прочитать `.cursor/skills/openspec-knowledge-add/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких чтений артефактов, отчётов, модулей или KB.

**Output style:** T-CONFIRM (`.cursor/docs/opsx-output-style.md` §5.5).

**Input:**
- Один или несколько путей к источникам: `/opsx:knowledge-add <path1> [path2 ...]`.
- Без аргументов выполняется Session Context Fallback (см. `SKILL.md`): команда подбирает кандидата из recently viewed / упомянутых в текущей сессии отчётов и спрашивает подтверждение через AskQuestion. Если кандидатов нет — показывает короткую ошибку с примерами.

**Флаги:**
- `--no-bundle` — не копировать source в `openspec/knowledge/_sources/knowledge-add/...`; допустимо только для стабильных источников из `openspec/changes/archive/.../reports/` или `openspec/knowledge/_sources/knowledge-add/.../sources/`.
- `--ttl <days>` — явный override TTL для всех сохраняемых кандидатов; по умолчанию TTL определяется по типу факта из `knowledge-format.mdc`.

**Примеры:**

```text
/opsx:knowledge-add temp/reports/exploration-pavДополнительныеФункции-2026-04-28.md
/opsx:knowledge-add temp/reports/report1.md temp/reports/report2.md
/opsx:knowledge-add openspec/changes/archive/2026-04-25-do2-pavlik-merge-roli-avtopodstanovka/reports/trace-analysis-2026-04-25.md --no-bundle
```
