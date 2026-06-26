---
name: /opsx:verify
id: opsx-verify
category: Workflow
description: "Quality gate для ЗНИ: объём и режим по контексту; находки объясняются прозой; мелочи правятся сама развилки — обсуждение без кодов Na/Nc."
---

Проверка постановки и плана изменения перед реализацией, между срезами и перед архивацией. Режим (до/после среза, один срез, миграция в срезы, без срезов) определяется по `tasks.md`, датам артефактов, записям в `reports/`/`debug.md` и тексту сообщения пользователя; единственный ключ командной строки — `--lite` (глубина, не режим).

**Контракт для пользователя:** на вход только имя ЗНИ `<change-name>`. **Одна команда → одно финальное сообщение.** Verify **сам** чинит repair-класс (internal Repair Loop, max 2 attempts) без «Подтвердить?» и без ping-pong extend↔verify в чате. Ответ: «можно apply» + **Следующий шаг** (`/opsx:apply` \| `/opsx:archive` по `verify_mode`) \| decision + ответ в чате \| terminal fail + чат/extend \| тихий статус (1a) + next step. Chat Surface Contract — §2.6 `opsx-output-style.md`.

**Первое действие:** прочитать `.cursor/skills/openspec-verify-change/SKILL.md` и идти по шагам. До прочтения скилла — не читать артефакты ЗНИ, трассы, модули.

**Input:**
- `/opsx:verify [<change-name>]` — пример: `/opsx:verify add-auth`; опционально в том же сообщении фразы вроде «проверь срез S2», «после приёмки S3».
- `--lite` — облегчённый режим глубины: только исполнимость (без повторного независимого аудита постановки). Guardrails и условия допуска — в SKILL verify; **не** связан с «Lite tier ЗНИ» (≤5 задач) из `sdd-workflow.mdc`.

**Подробности:** шаги, фильтр новизны со снимком `snapshot` в YAML отчёта, условный вызов агентов — в скилле; шаблон чата — `.cursor/skills/openspec-verify-change/templates/chat-summary.md`; коммуникация — `.cursor/rules/verify-user-communication.mdc`.
