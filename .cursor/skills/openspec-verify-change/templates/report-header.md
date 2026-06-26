# YAML frontmatter отчёта verify

Копируется в начало `reports/verification-YYYY-MM-DD.md` (плейсхолдеры заменить):

```yaml
---
verify_mode: <pre-apply | post-apply>
change: <имя-change>
date: YYYY-MM-DD
verdict: <GO | NO-GO>
layer_status:
  layer_1_hygiene: <PASS | AUTOFIXED | FAIL>
  layer_2_internal_coherence: <PASS | WARNING | FAIL>
  layer_3_problem_solution: <PASS | WARNING | FAIL>
  layer_4_independent_challenge: <APPROVE | CHALLENGE | REJECT | SKIPPED-novelty | SKIPPED-override>
  layer_5_implementation_readiness: <PASS | WARNING | FAIL>
snapshot:
  accepted_tasks:
    - S1.1
    - S1.accept
  closed_decisions: []
  # agent-only: id (snake_case), summary (prose), closed_at (ISO date), source (verify-user-answer | repair-from-verify)
  open_decision_id: null
  decision_round: 0
  decision_round_max: 2
  verify_depth: full
  # full | incremental | lite — см. SKILL.md § verify depth
  assumptions_accepted: []
  open_known_questions: []
  artifacts_mtime:
    proposal.md: "YYYY-MM-DDTHH:mm:ss"
    design.md: "YYYY-MM-DDTHH:mm:ss"
    tasks.md: "YYYY-MM-DDTHH:mm:ss"
    specs/<capability-folder>/spec.md: "YYYY-MM-DDTHH:mm:ss"
  last_challenge_at: "YYYY-MM-DDTHH:mm:ss"  # mtime design.md на момент последнего успешного Layer 4
---
```

## Правила полей

### `verify_mode`

Только два значения:

- **`pre-apply`** — есть хотя бы одна `[ ]` задача (включая `S<N>.accept`); реализация не завершена.
- **`post-apply`** — все задачи `[x]`, все `S<N>.accept` приняты.

Узкий охват одного среза, переход между срезами и смешанный режим — **частные случаи `pre-apply`**, выводятся из текста запроса пользователя и состояния `tasks.md`. Отдельных значений `slice-pre`, `slice-post`, `slice-scoped`, `slice-transition`, `legacy-pre`, `legacy-mixed` **нет** — они удалены.

### `verdict`

Только **GO** или **NO-GO**. Бинарный.

- **GO** — безопасно запускать `/opsx:apply`. Допускает Layer 1 автоправки (статус `AUTOFIXED`) и Layer 2/5 WARNING без блокеров.
- **NO-GO** — есть содержательное сомнение в Layer 3 (Problem-Solution Trace), Layer 4 (Independent Challenge с вердиктом CHALLENGE/REJECT) или критический FAIL в любом другом слое. Требуется обсуждение и/или `/opsx:explore` / `/opsx:extend` до повторного verify.

### `layer_status`

Пять полей, по одному на слой. Возможные значения:

- **Layer 1 (Hygiene):** `PASS` (нечего править), `AUTOFIXED` (автоправки применены), `FAIL` (немеханические проблемы формата).
- **Layer 2 (Internal Coherence):** `PASS`, `WARNING` (несущественные несостыковки артефактов), `FAIL` (циклы зависимостей, несовпадение spec ↔ tasks).
- **Layer 3 (Problem-Solution Trace):** `PASS`, `WARNING` (орфаны без блокера), `FAIL` (Requirement без задач или задача без Requirement).
- **Layer 4 (Independent Challenge):** `APPROVE`, `CHALLENGE`, `REJECT`, `CHALLENGE-saturated`, `SKIPPED-novelty`, `SKIPPED-override`, `SKIPPED-lite`.
- **Layer 5 (Implementation Readiness):** `PASS`, `WARNING` (мелкие GAP реализуемости), `FAIL` (задача не реализуема as-is).

### `snapshot.last_challenge_at`

ISO-метка времени модификации `design.md` на момент последнего **успешного** прогона Layer 4 (вердикт APPROVE или CHALLENGE с принятым к работе через `--from-verify`). Используется на следующем verify для решения, нужен ли challenge заново:

- Если `mtime(design.md) > last_challenge_at` → Layer 4 запускается.
- Если `mtime(design.md) <= last_challenge_at` → Layer 4 пропускается (`layer_status.layer_4_independent_challenge: SKIPPED-novelty`).

При первом verify по ЗНИ `last_challenge_at` отсутствует → Layer 4 обязателен.

При вердикте REJECT в Layer 4 — `last_challenge_at` **не обновляется**, чтобы повторный verify снова запустил challenge.

### `snapshot.accepted_tasks`

Полный упорядоченный список идентификаторов строк `tasks.md` с `- [x]`. Форматы:

- `S<N>.<M>` — рабочая задача среза N.
- `S<N>.accept` — приёмочная задача среза N (один на срез).
- `F<k>` — Follow-up.

Старый формат `S<N>.T<M>` (несколько приёмочных задач на срез) поддерживается в legacy-режиме — verify читает оба формата, но новые ЗНИ через `/opsx:new` генерируют только `S<N>.accept`.

### `snapshot.artifacts_mtime`

ISO-8601 строка до секунды для `proposal.md`, `design.md`, `tasks.md` и каждого `openspec/changes/<name>/specs/**/*.md`. Используется фильтром новизны (`silent_ok` / `progress_only` / `full_run`) на следующем прогоне.

## Что хранится только в YAML и НЕ дублируется в чат

- `verify_mode`, `verdict`, `layer_status` — внутреннее техническое состояние.
- `tier` (если используется во внутренних метриках) — только в YAML.
- В первой строке секции `## Резюме для разработчика` (см. `templates/executive-summary.md`) — формулировка `<change-name> — можно запускать apply.` либо `<change-name> — до apply нужно решение: <одна техническая фраза>` (см. `templates/verdict-card.md`), а не `verdict: GO` / `verdict: NO-GO`.

## Порядок блоков файла (после YAML)

YAML — **первый** блок файла (нужен фильтру новизны и аудиту). Сразу за ним — **читаемый человеком** контент в следующем порядке:

1. `## Резюме для разработчика` — на языке кода 1С, зеркало чата.
2. `## Решения до apply` — только при `verdict: NO-GO`.
3. `## Что меняется в постановке` — карточка изменений для разработчика.
4. `### Подправил в постановке` — авто-правки гигиены простыми словами (если были).
5. `### К сведению` — мелочи, не блокирующие apply (если есть).
6. `## Технический аудит (для движка OpenSpec)` — статусы слоёв, технические алерты. **Только здесь** допустимы имена `Layer N`, `PASS/FAIL`, `CHALLENGE`, `design-challenge`, `task-readiness`.
7. `## Источники` — пути к дочерним отчётам (`quality-control-*.md`, `design-challenge-*.md`, `architecture-task-readiness-*.md`), технические коды алертов.

Полный шаблон каждой секции — `templates/executive-summary.md`.
