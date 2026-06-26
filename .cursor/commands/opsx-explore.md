---
name: /opsx:explore
id: opsx-explore
category: Workflow
description: "Единая точка входа: исследование задачи, дефекта или вопроса"
---

**Первое действие:** прочитать `.cursor/skills/openspec-explore/SKILL.md` и следовать Entry Protocol.

Любой свободный текст без другой команды — тот же протокол.

**Ввод:** текст, скриншоты, путь к трассе/PERF (`.pff`, `*_TRACE_*.txt`, `*_PERF.txt`).

**Итог исследования:** блок `## Постановка ЗНИ` **в чате** (для bug-профиля). Полные отчёты `Task` — в `temp/reports/<тип>-YYYY-MM-DD-<slug>.md`. Опциональный handoff-файл `temp/explore-handoff-*.md` — только по словесной просьбе («сохрани», «зафиксируй в файл»). Каталог `openspec/sessions/` в новых исследованиях не создаётся.

**Примеры:**

- `/opsx:explore двойное чтение двоичных данных при проверке подписи`
- `/opsx:explore @temp/trace.txt см. скрин`
