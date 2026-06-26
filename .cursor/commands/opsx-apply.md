---
name: /opsx:apply
id: opsx-apply
category: Workflow
description: Реализация задач OpenSpec change по срезам (по умолчанию — step-by-slice)
---

Implement tasks from an OpenSpec change. По умолчанию работает в режиме **step-by-slice**: реализация одного среза → карточка приёмки → следующий срез. См. `.cursor/rules/vertical-slices.mdc`.

**Первое действие:** прочитать `.cursor/skills/openspec-apply-change/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких чтений артефактов, трасс, модулей.

**Input:**
- `<change-name>` (optional, e.g. `/opsx:apply add-auth`) — если пропущено, скилл предложит выбор.

**Флаги:**
- `--slice S<N>` — выполнить **только** срез S<N> (без перехода к следующему). Пример: `/opsx:apply add-auth --slice S2`. Полезно для целевого фикса или повторной попытки после rejection.
- `--since-slice S<N>` — начать выполнение со среза S<N>, пропуская предыдущие принятые. Пример: `/opsx:apply add-auth --since-slice S3` — продолжить с S3, даже если S1/S2 не помечены `[x]` (legacy / частично принятые срезы). По умолчанию apply сам определяет первый непринятый срез.
- `--step-by-step` — отключить step-by-slice; вместо приёмки целого среза — пауза после каждой задачи (для отладки).
- `--batch` — выполнить **все** оставшиеся срезы без остановок на slice-gate (только при явном `--batch` или для legacy-ЗНИ без срезов).

При отсутствии флагов — default = step-by-slice от первого непринятого среза до конца.
