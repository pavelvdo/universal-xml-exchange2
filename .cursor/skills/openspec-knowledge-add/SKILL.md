---
name: openspec-knowledge-add
description: Add verified OpenSpec Knowledge Base facts from standalone reports or markdown sources outside a change archive.
---

# Skill: openspec-knowledge-add

## Назначение

`/opsx:knowledge-add` добавляет в `openspec/knowledge/` только верифицированные KB-факты из standalone-источников: аналитических отчётов, ручных markdown-заметок или уже стабильных reports. Команда не создаёт ЗНИ и не извлекает знания напрямую из BSL/XML-кода.

**Принцип:** либо полностью валидный KB с проверяемыми anchors, либо ничего. Если источник не даёт verified knowledge-worthy фактов, команда честно завершает работу без записи файлов.

## Размежевание

- `/opsx:archive <name>` — основной путь создания KB из reports активной ЗНИ.
- `/opsx:knowledge-audit --from-archive <name>` — повторное извлечение из уже архивированной ЗНИ.
- `/opsx:knowledge-add <path1> [path2 ...]` — произвольные источники вне ЗНИ (`temp/reports/...`, локальные markdown-заметки, одиночные reports).
- `/opsx:knowledge-audit` без `--from-archive` — TTL/drift/reindex/metrics; новые KB не создаёт.

## Input

```text
/opsx:knowledge-add <path1> [path2 ...] [--no-bundle] [--ttl <days>]
```

Без аргументов:

1. Выполнить **Session Context Fallback** (см. ниже): подобрать source только из путей, уже видимых в текущей командной сессии.
2. Если fallback не нашёл кандидатов — не выполнять поиск по `temp/reports/` и не задавать AskQuestion со списком файлов.
3. Вывести краткую ошибку и примеры:

```text
/opsx:knowledge-add temp/reports/exploration-foo-2026-04-28.md
/opsx:knowledge-add temp/reports/report1.md temp/reports/report2.md
/opsx:knowledge-add openspec/changes/archive/2026-04-25-name/reports/exploration-2026-04-25.md --no-bundle
```

## Session Context Fallback

Применяется только когда команда вызвана без путей. Цель — подхватить отчёт, который уже был явно показан пользователю в текущей командной сессии, не превращая пустой вызов в поиск по файловой системе.

### Источники кандидатов

Использовать только пути, уже видимые в контексте сессии:

- пути к `.md`, упомянутые в сообщениях ассистента текущей командной сессии;
- `<open_and_recently_viewed_files>` / focused file, если они относятся к каноническим source-каталогам ниже.

Запрещено выполнять Glob/Grep/поиск по `temp/reports/`, `openspec/changes/**/reports/` или `openspec/knowledge/_sources/` для подбора кандидатов. Точечная проверка существования уже найденного пути разрешена и выполняется так же, как для явно переданного input.

### Фильтры

Каталоги:

- `temp/reports/`;
- `openspec/changes/<name>/reports/`;
- `openspec/knowledge/_sources/knowledge-add/<YYYY-MM-DD-slug>/sources/`.

Каталог `reports/` в корне репозитория **не является каноническим source-каталогом** и не участвует в fallback. Если такой путь уже виден в сессии, fallback его игнорирует без AskQuestion; пользователь может передать путь явно, тогда сработает обычная классификация с warning.

Имена файлов:

- `exploration-*.md`;
- `trace-analysis-*.md`;
- `resolved-contract-*.md`;
- `architecture-*.md`;
- `deep-analysis-*.md`;
- `design-review-*.md`.

Markdown вне этих масок при fallback отклоняется без warning: пользователь может передать такой source явно, если хочет применить обычное правило `Markdown вне канонических масок`.

### Результат fallback

- `0` кандидатов → вывести обычную ошибку и примеры из `## Input`, ничего не писать.
- `1` кандидат → задать AskQuestion: «Использовать source `<path>`?»
  - `yes` → передать этот путь в `Preflight` и дальше выполнять обычный pipeline.
  - `no` → итог `Declined by user`, ничего не писать.
  - `specify` → попросить пользователя прислать явный путь; ничего не писать.
- `2+` кандидата → задать AskQuestion со списком путей и вариантами `all`, `cancel`, `specify`.
  - `all` → передать все выбранные пути в `Preflight`.
  - `cancel` → итог `Declined by user`, ничего не писать.
  - `specify` → попросить пользователя прислать явный путь; ничего не писать.

При показе вопроса создать/обновить TodoWrite-чекпоинт:

```yaml
id: kb-confirm
content: "Ожидается выбор source из контекста сессии"
status: in_progress
```

Пока этот чекпоинт в статусе `in_progress`, следующий ход командной сессии сначала трактуется как ответ на выбор source. После выбора source чекпоинт переводится в `completed` или `cancelled`, затем запускается обычный `Preflight`.

## Флаги

- `--no-bundle` — не копировать source. Допустим только если все входные пути уже стабильны:
  - `openspec/changes/archive/<YYYY-MM-DD-name>/reports/...`
  - `openspec/knowledge/_sources/knowledge-add/<YYYY-MM-DD-slug>/sources/...`
  Иначе остановиться: `Blocked — --no-bundle requires stable sources`.
- `--ttl <days>` — явный override TTL для всех сохраняемых кандидатов. Без флага TTL выбирать по таблице `knowledge-format.mdc` (`TTL POLICY`).

## Preflight

1. Прочитать `.cursor/rules/knowledge-format.mdc`.
2. Прочитать `openspec/knowledge/_taxonomy.yaml`.
   - Если файла нет → `Blocked — taxonomy missing`, предложить `/opsx:knowledge-init`, ничего не писать.
3. Прочитать `openspec/knowledge/_index.yaml`.
   - Если файл отсутствует или corrupt → предложить `/opsx:knowledge-audit --reindex`, ничего не писать.
4. Проверить, что каждый путь из input существует и читается.
   - Отсутствующие пути перечислить в ошибке, ничего не писать.
5. Выполнить structure check корня `openspec/knowledge/` по whitelist из `knowledge-format.mdc`.
   - Нарушения добавить в Warnings, но не блокировать extraction.

## Input Classification

Классифицировать каждый входной путь:

| Тип | Действие |
|-----|----------|
| `exploration-*.md`, `trace-analysis-*.md`, `resolved-contract-*.md`, `architecture-*.md`, `deep-analysis-*.md`, `design-review-*.md` | Пригоден как аналитический report |
| Markdown вне канонических масок | Пригоден только если содержит verified facts + anchors; добавить warning «вне канонических масок reports» |
| Markdown из корневого `reports/` | Пригоден только при явном пути; добавить warning «non-canonical source location, recommend `temp/reports/`» |
| `openspec/knowledge/**/KB-*.md` | `Skipped — already a KB` |
| `.bsl`, `.xml`, `.mxl`, `.json`, `.txt`, трассы `.pff` / `*_TRACE_*.txt` | `Skipped — not a knowledge source`; подсказка: сначала `/opsx:explore`, затем `/opsx:knowledge-add <report>` |
| Каталог | `Skipped — directory input is not supported`; пользователь должен передать конкретные files |

Команда **не запускает** `onec-code-explorer` сама. Она работает с уже подготовленными источниками знаний, а не проводит обследование.

Если все входы skipped на этапе классификации → итог `Saved 0`, ничего не писать.

## Extraction Contract

Применяется к пригодным источникам.

1. Извлечь до 5 KB-кандидатов за сессию.
2. Кандидат должен удовлетворять критериям `knowledge-format.mdc`, включая обязательный `Reuse Value Test`:
   - verified факт о поведении, контракте, цепочке вызовов, метаданных или стабильном имени;
   - не ADR (нет решения с trade-off);
   - не spec (не требование к будущему поведению);
   - не debug-контекст разового инцидента;
   - не тривиальное имя объекта без полезного контракта.
3. Если кандидатов больше 5:
   - выбрать top-5 по релевантности: verified в source, наличие конкретных anchors, отсутствие dedup-конфликта, ширина будущего переиспользования;
   - остальные перечислить в Warnings с `source:lines`.
4. Для каждого кандидата подготовить:
   - `title` ≤ 80 символов;
   - `priority`: `core` или `peripheral`;
   - `domain` и `subdomain`;
   - `anchors`;
   - `source.report`, `source.lines`, `source.also-mentioned-in`;
   - `ttl-days`;
   - `reuse-scenario` — одно предложение: где будущая ЗНИ/расследование переиспользует факт;
   - `why-knowledge`;
   - `supersedes` / `supersedes-by`, если это замена существующего KB;
   - текст секций `## Факт` и `## Почему это knowledge, а не ADR/spec`.

### Main Fact Priority

Перед выбором top-5 и перед preview классифицировать кандидатов:

- `core` — центральный факт source-отчёта: совпадает с заголовком, первым разделом «Определение»/«Факт», основной причиной запуска исследования или главным вопросом пользователя.
- `peripheral` — смежный факт, который помогает контексту, но не отвечает на главный вопрос отчёта.

Если все `core` кандидаты заблокированы (`taxonomy mismatch`, `unverified content`, `signature-drift`, `behavioral-drift`, `anchor-missing`) или отложены Reuse Value Test, команда **не подменяет** их `peripheral` кандидатами. Итог: `Saved 0 — Blocked: core facts unsaveable`; в Warnings перечислить причины по каждому core-кандидату и предложить следующий шаг (`/opsx:knowledge-init` для taxonomy mismatch, `/opsx:explore` для re-verify).

`peripheral` кандидаты можно показывать в preview только если хотя бы один `core` кандидат прошёл validation и попал в saveable-набор. Это защищает KB от шума: вторичный факт не должен становиться единственным результатом команды, если главное знание сохранить нельзя.

## Candidate Validation

Для каждого кандидата:

### Domain / Subdomain

1. Подобрать `domain` по `_taxonomy.yaml`: `anchor.path` должен попадать под `domain.source`.
2. Если anchors в нескольких зонах — домен выбирать по основному anchor, смежные факты отражать в `related.kb` только если есть существующие KB.
3. Если нельзя уверенно назначить domain/subdomain → `Blocked — taxonomy mismatch` для кандидата. Не создавать новый domain и не править taxonomy.

### Anchors

Допустимые типы anchors — только из `knowledge-format.mdc`:

- `procedure`
- `event-name`
- `metadata-object`
- `query-pattern`

Для `procedure` извлечь `fingerprint.declaration` и `fingerprint.body-head` по правилам `knowledge-format.mdc`.

### Verify

Проверить anchors против текущего `src/` по алгоритму `knowledge-format.mdc`.

- `verified` → кандидат может идти в карточку.
- `signature-drift`, `behavioral-drift`, `anchor-missing`, `count-drift` → `Skipped — stale anchors at extraction time`.
- `verify-failed` → `Skipped — unverified content`.

### Dedup

Проверить `_index.yaml` и файлы KB (включая `_archive/`):

- точное совпадение `anchor.path + name/value/signature`;
- близкий `title`;
- значимое пересечение `anchor-paths`.

Результат:

- существующий KB полностью покрывает факт → `Skipped — duplicate of KB-NNNN`;
- новый факт заменяет старый → карточка `SUPERSEDES KB-NNNN`;
- факты различны → кандидат остаётся.

### TTL

Без `--ttl` выбирать по `knowledge-format.mdc`:

- внешний API → 30;
- точка расширения типовой процедуры → 60;
- цепочка вызовов / поведение внутри расширения → 90;
- имя ЖР, константа, метаданные → 180.

### Reuse Value

Применить `Reuse Value Test` из `knowledge-format.mdc`.

- Все 5 критериев = `да` → кандидат остаётся.
- Любой критерий = `нет` или `не уверен` → `Deferred — reuse value not justified`.
- Deferred-кандидат не показывается в preview, не сохраняется и добавляется в Warnings со списком проваленных критериев (`RVT-1`..`RVT-5`) и `source:lines`.
- `reuse-scenario` обязателен. Если невозможно честно назвать будущую ЗНИ/расследование, где факт переиспользуется, считать проваленным `RVT-1`.

### Why Knowledge

`why-knowledge` обязательно и должно включать `reuse-scenario` из Reuse Value Test. Если невозможно написать честное предложение «почему это KB, а не ADR/spec/debug/project», кандидат отбрасывается: `Skipped — knowledge-worthy criterion not justified`.

## Source Bundle

KB хранит короткую карточку и ссылку на source, а не копию отчёта.

Стабильные sources:

- `openspec/changes/archive/<YYYY-MM-DD-name>/reports/...`
- `openspec/knowledge/_sources/knowledge-add/<YYYY-MM-DD-slug>/sources/...`

Все остальные sources считаются нестабильными и копируются в bundle **только после подтверждения сохранения**.

### Planned Source Path

До показа AskQuestion вычислить итоговый путь source:

```text
openspec/knowledge/_sources/knowledge-add/<YYYY-MM-DD>-<slug>/sources/<original-name>
```

Именно этот planned path показывать в карточке preview и записывать в будущий `source.report`.

### Bundle-on-save

При подтверждённом сохранении (`all`/`yes` или выбранное подмножество) и хотя бы одном нестабильном source:

```text
openspec/knowledge/_sources/knowledge-add/<YYYY-MM-DD>-<slug>/
  sources/
    <original-name-1>
    <original-name-2>
  knowledge-add-report.md
```

Правила:

- bundle создаётся только при сохранении хотя бы одного KB;
- при `cancel`, `no`, `Declined by user` или `Saved 0` bundle не создаётся;
- `<slug>` брать из title первого сохраняемого KB или объединяющей темы;
- при коллизии каталога добавлять `-2`, `-3`;
- если один факт подтверждён несколькими sources, `source.report` — самый подробный source, остальные → `source.also-mentioned-in`.

## Preview / Confirmation

Если после validation нет кандидатов:

1. Не задавать AskQuestion и не переводить сессию в `awaiting confirmation`.
2. Ничего не писать.
3. Вывести `Saved 0` и список причин:
   - `No candidates after filters`
   - `Skipped — ...`
   - `Blocked — ...`
   - `Deferred — reuse value not justified`

Если кандидаты есть, в **одном пользовательском ответе** сначала показать per-candidate карточки всех кандидатов, затем сразу задать единый вопрос подтверждения. Карточки не переносить в финальный T-CONFIRM: финал сообщает только результат сохранения.

**Cards-first guardrail:** AskQuestion разрешён только после видимого вывода карточек в том же ходе. Перед вызовом AskQuestion выполнить self-check: в пользовательском ответе должны быть показаны ровно `N` блоков `### KB-<next-id-preview>: <title>`, где `N` — число кандидатов. Если карточки не выведены или их количество не совпадает с `N`, AskQuestion не вызывать, ничего не писать и завершить с итогом `Saved 0 — cards not rendered`.

```markdown
### KB-<next-id-preview>: <title>
- **Domain / subdomain:** <domain> / <subdomain>
- **TTL:** <days>
- **Anchors:** <path>:<name> (+ дополнительные при наличии)
- **Reuse scenario:** <one sentence>
- **Why knowledge:** <one sentence>
- **Source:** <planned-stable-source>:<line-range>
- **Supersedes:** KB-NNNN (если применимо)
```

Единый вопрос: «Сохранить N KB-фактов?»

Options:

- AskQuestion должен быть `allow_multiple: true`.
- `all` — сохранить все `N` кандидатов (`KB-NNNN`, `KB-NNNN`, ...).
- `KB-<next-id-preview>` — по одному option на каждого кандидата: `KB-<next-id-preview>: <title ≤60 символов>`.
- `cancel` — отменить сохранение, ничего не писать.

Preview ID не считается зарезервированным. Окончательная нумерация выполняется только при save.

### Selection resolution

Результат AskQuestion интерпретировать так:

- выбран `cancel` в любой комбинации → `Declined by user`;
- выбран `all` → Save Protocol для всех кандидатов;
- выбраны только конкретные `KB-<next-id-preview>` → Save Protocol только для указанных кандидатов;
- выбран `all` вместе с конкретными KB → Save Protocol для всех кандидатов, индивидуальный выбор игнорировать, в финальный summary добавить warning о конфликте выбора;
- пустой выбор или `Questions skipped by the user` → ничего не писать, кроме краткой строки ожидания из `Confirmation channels`, состояние = `awaiting confirmation`.

### Confirmation channels

Подтверждение принимается двумя каналами:

1. **AskQuestion** — нормальный путь. `all` запускает Save Protocol для всех кандидатов, выбор конкретных `KB-<next-id-preview>` запускает Save Protocol только для указанных кандидатов, `cancel` завершает команду как `Declined by user`.
2. **Текстовый ответ пользователя на следующем ходе** — fallback для случаев, когда форма закрыта или инструмент вернул `Questions skipped by the user`. Допустимые ответы:
   - `все`, `all`, `да`, `yes`, `сохранить`, `сохранить все` → Save Protocol для всех кандидатов;
   - `нет`, `no`, `отмена`, `cancel` → `Declined by user`;
   - `сохранить KB-0005, KB-0007`, `0010 0012`, `10, 12` или другой список номеров/preview-id → Save Protocol только для выбранных кандидатов;
   - `KB-0010..0012` или `0010-0012` → Save Protocol для кандидатов из диапазона;
   - `все кроме KB-0011` или `all except 0011` → Save Protocol для всех кандидатов, кроме перечисленных.

Номера без префикса `KB-` сопоставлять только с текущими preview-id: например, `10` соответствует `KB-0010`, если такой preview-id показан. Если текстовый ввод не содержит ни одного совпадения с текущими preview-id и не является `all`/`cancel`, ничего не сохранять и вывести: `Ожидаю выбор: "все", "отмена" или номера KB через запятую`.

`Questions skipped by the user` **не равно отказу**. В этом случае ничего не писать, кроме краткой строки: `Ожидаю выбор: "все", "отмена" или номера KB через запятую`. Состояние сессии = `awaiting confirmation`; на следующем ходе skill сначала обрабатывает подтверждение, а не начинает новую extraction-сессию.

Guardrail UX: при `Questions skipped by the user` вывести **ровно одну** строку выше, без повторной карточки кандидатов, без пересказа опций и без пояснения «это не отказ». Anti-pattern: длинное сообщение с повторным списком вариантов рядом с ещё видимой формой AskQuestion.

### Awaiting confirmation state

При показе кандидатов создать/обновить TodoWrite-чекпоинт:

```yaml
id: kb-confirm
content: "Ожидается подтверждение сохранения KB-кандидатов"
status: in_progress
```

Пока `kb-confirm` в статусе `in_progress`, каждый следующий ход этой командной сессии начинается с разбора ответа пользователя как подтверждения. После `all`/`cancel`/выбора подмножества чекпоинт переводится в `completed` или `cancelled`, затем выполняется соответствующий результат.

## Save Protocol

Только при подтверждённом сохранении: `all`/`yes` для всех кандидатов или выбранное подмножество preview-id.

1. Определить сохраняемый набор кандидатов: все кандидаты при `all`/`yes` или только выбранные preview-id при частичном выборе.
2. Определить следующие номера: Glob `openspec/knowledge/**/KB-*.md` включая `_archive/`, max + 1.
3. Перед созданием bundle проверить, что директория `openspec/knowledge/_sources/` отсутствует или соответствует whitelist из `knowledge-format.mdc`; при нарушении — rollback, warning, без записи KB.
4. Создать bundle, если нужен.
5. Создать `openspec/knowledge/<domain>/KB-NNNN-slug.md` для каждого сохраняемого кандидата, прошедшего Reuse Value Test; deferred-кандидаты не писать.
6. Пересобрать `openspec/knowledge/_index.yaml` из фактических KB-файлов на диске.
7. Создать `knowledge-add-report.md` в bundle (если bundle был создан) со списком:
   - сохранённых KB;
   - skipped/blocked/deferred candidates;
   - structure warnings;
   - sources.

### Atomicity / Rollback

Если ошибка возникла до перезаписи `_index.yaml`:

- удалить созданные в этой сессии KB-файлы;
- удалить bundle этой сессии;
- оставить старый `_index.yaml`.

Если ошибка возникла после перезаписи `_index.yaml`:

- выполнить `/opsx:knowledge-audit --reindex`-совместимую пересборку индекса из disk-state;
- вывести warning о recovery.

Номер KB не переиспользуется только после успешного создания KB-файла. Preview номера не резервируются.

## Output

Использовать T-CONFIRM (`.cursor/docs/opsx-output-style.md` §5.5).

Итоговые состояния:

- `Saved N (KB-NNNN, ...)`
- `Deferred N — reuse value not justified`
- `Declined by user`
- `Saved 0 — No candidates after filters`
- `Saved 0 — cards not rendered`
- `Saved 0 — Blocked: core facts unsaveable`
- `Blocked — taxonomy missing`
- `Blocked — --no-bundle requires stable sources`

В summary показать:

- сохранённые KB-файлы;
- bundle path, если создан;
- обновление `_index.yaml`;
- Warnings только если есть;
- следующий шаг: `/opsx:knowledge-audit --metrics` или повторный `/opsx:knowledge-add <path>`.

Перед финальным выводом выполнить self-check-5 из `.cursor/docs/opsx-output-style.md`.

## Archive Integration

`openspec-archive-change` шаг 5.5 использует тот же extraction contract:

- inputs: `reports/exploration-*.md`, `reports/trace-analysis-*.md`, `reports/resolved-contract-*.md` из архивируемой ЗНИ;
- source считается стабильным planned archive path: `openspec/changes/archive/YYYY-MM-DD-<change>/reports/<report>.md`;
- bundle не создаётся;
- Reuse Value Test применяется до preview; deferred-кандидаты попадают в Warnings archive, а не в AskQuestion;
- archive сохраняет бинарный выбор `[yes | no]`: если кандидаты есть, archive показывает карточки и единый AskQuestion только для KB-кандидатов, как было в шаге 5.5; при `no` archive продолжается со state `Declined by user`.
- cards-first guardrail обязателен и для archive: AskQuestion в archive нельзя задавать, пока per-candidate карточки не выведены в том же ходе.

## Guardrails

- Не создавать и не изменять ЗНИ.
- Не писать в `src/`.
- Не создавать новые domain/subdomain в taxonomy.
- Не извлекать KB напрямую из BSL/XML/trace; сначала нужен аналитический report.
- Session Context Fallback не запускает Glob/Grep по `temp/reports/` и не сканирует файловую систему; кандидаты — только пути, уже видимые в текущей командной сессии. Save по-прежнему требует подтверждения через AskQuestion.
- Не создавать bundle и KB при `Saved 0` или `Declined by user`.
- Не скрывать причины отказа: каждый skipped/blocked candidate должен быть отражён в summary или `knowledge-add-report.md` (если bundle создан).
