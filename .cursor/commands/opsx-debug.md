---
name: /opsx:debug
id: opsx-debug
category: Workflow
description: "Расследование бага в контексте OpenSpec change: trace, RCA, задачи фикса для apply"
---

Расследование бага и RCA с захватом задач фикса в OpenSpec change.

**Что можно передать (в любой комбинации):**

- путь к трассе (`.pff`, `*_TRACE_*.txt`) — текстом;
- ожидаемое vs фактическое поведение — текстом;
- скриншоты — attachment (drag-and-drop);
- id change (опц., напр. `do2-partial-repeat-saved-executors-do21-pavlik`);
- номер задачи (опц., напр. `S5.1` или `4.3`).

**Примеры:**

- `/opsx:debug do2-partial-repeat-saved-executors-do21-pavlik task=S5.1 trace=C:\GitHub\PavDO\temp\TRACE_2026-04-19.txt "ожидалось 3 строки, видна 1"`
- `/opsx:debug "форма параметров: лишняя колонка «Результат сохранён», см. скриншот"`

**Первое действие:** прочитать `.cursor/skills/openspec-debug/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких чтений артефактов, трасс, модулей.
