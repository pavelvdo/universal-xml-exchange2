---
name: /opsx:sync
id: opsx-sync
category: Workflow
description: Синхронизация delta specs change в основные specs
---

**Первое действие:** прочитать `.cursor/skills/openspec-sync-specs/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких других чтений.

**Output style:** T-CONFIRM (`.cursor/docs/opsx-output-style.md` §5.5).

**Input**:
- `/opsx:sync` — без аргументов: синхронизирует спеки активного (единственного) ЗНИ.
- `/opsx:sync <name>` — синхронизирует спеки указанного ЗНИ.
