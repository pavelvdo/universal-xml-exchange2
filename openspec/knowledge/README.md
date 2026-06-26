# Knowledge Base (OpenSpec)

`openspec/knowledge/` — это **база атомарных, проверяемых (verified) фактов о системе**, которые помогают быстрее принимать решения и избегать повторных расследований.

## Что считается KB-фактом

- Факт **проверяемый в коде/артефактах** (есть якоря `file:line`).
- Факт **не является** требованием (spec), решением с trade-off (ADR) или одноразовым отчётом.
- Факт **ссылается на источник извлечения** (например, `reports/exploration-*.md`, `reports/trace-analysis-*.md`, `reports/resolved-contract-*.md`).

## Кто пишет и кто читает

- Пишет:
  - **archive шаг 5.5** (извлечение кандидатов из аналитических отчётов)
  - **`/opsx:knowledge-audit`** (проверка/реиндексация/ретро-пилот через `--from-archive`)
- Читает:
  - **`/opsx:explore`** (Knowledge Discovery)
  - **Architect Gate** (Knowledge Discovery для подготовки архитектурного ревью)
  - **archive шаг 2b** (verify KB в scope git-diff)

## Что нельзя складывать в KB

- Процессные отчёты, ревью, заметки (для этого есть `temp/reports/`).
- ADR, спецификации, требования (для этого есть `openspec/adrs/`, `openspec/specs/`, `openspec/changes/**`).
- Debug-контекст и гипотезы без верификации.
- Произвольные подкаталоги и произвольные `.md` в корне `openspec/knowledge/` (whitelist — в `.cursor/rules/knowledge-format.mdc`, секция РАСПОЛОЖЕНИЕ).

## Где лежат факты

- Факты размещаются как файлы вида: `<domain>/KB-NNNN-<slug>.md` (или `<domain>/<subdomain>/KB-NNNN-<slug>.md`).
- Архив/история хранится в `_archive/`.

## Как удалить факт

- Удаление через `--force-delete` и явное подтверждение пользователя — крайняя мера.
- Обычный путь: перевод в `deprecated`/`superseded` и автоматический перенос в `_archive/` через `/opsx:knowledge-audit` (см. governance в `knowledge-format.mdc`).

## Формат

Единый формат KB-фактов и правила размещения описаны в `.cursor/rules/knowledge-format.mdc`.
