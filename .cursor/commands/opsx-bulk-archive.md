---
name: /opsx:bulk-archive
id: opsx-bulk-archive
category: Workflow
description: Массовая архивация нескольких завершённых changes
---

**Первое действие:** прочитать `.cursor/skills/openspec-bulk-archive-change/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких других чтений.

**Output style:** T-CONFIRM (`.cursor/docs/opsx-output-style.md` §5.5).

**Input**:
- `/opsx:bulk-archive` — без аргументов: сканирует все changes и предлагает выбор.
- `/opsx:bulk-archive <name1> <name2> ...` — архивирует указанные ЗНИ.
