---
name: /opsx:new
id: opsx-new
category: Workflow
description: Создание или дозавершение change (ЗНИ) со всеми артефактами для реализации
---

Создание change (ЗНИ) — **новый** change или **resume** существующего (дозавершение артефактов).

**Первое действие:** прочитать `.cursor/skills/openspec-new-change/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких чтений артефактов, трасс, модулей.

**Бриф (Sync Card):** B0 при `## Постановка ЗНИ` в чате/handoff (информирующая строка, без согласования имени); B1 при свободном тексте — карточка **Изменение** + Подтвердить? — см. `.cursor/docs/templates/brief-card.md`.

**Второе действие (только новый change):** Metadata Gate. Read `openspec/project.md` (`defaultDeveloper`, канон `domain_label`). Агент собирает черновик **описания** (`comment_suffix`) из «Темы маркера» / Why; ФИО — из project.md или отдельный текстовый шаг; preview `// +++ ФИО дата описание`; `AskQuestion` — только при черновике **и** известном ФИО. Без metadata `openspec new change` не запускается. Resume — metadata из proposal.md, gate пропускается.

Input: argument after the command is the change name (kebab-case) or a description of what they want to build.
Optional flag: `--skip-architect "<причина>"` to bypass mandatory Architect Gate.
