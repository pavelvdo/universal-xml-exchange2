---
name: /opsx:archive
id: opsx-archive
category: Workflow
description: Архивация завершённого change
---

Archive a completed change in the experimental workflow.

**Первое действие:** прочитать `.cursor/skills/openspec-archive-change/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких чтений артефактов, трасс, модулей.

**Output style:** T-CONFIRM (`.cursor/docs/opsx-output-style.md` §5.5).

Input:
- Optionally specify a change name (e.g., `/opsx:archive add-auth`). If omitted, the skill will prompt for selection (step 1).
- **`--force-legacy`:** пропуск slice-gate на шаге 3.5 при непринятых **`S<N>.accept`** (legacy `S<N>.T<M>`) — архив без отметки в `tasks.md`, warnings. Иначе при `[ ]` на приёмочных задачах — карточка и **AskQuestion** (подтверждение как на slice gate в apply: отметка и продолжение / стоп / force-legacy).
