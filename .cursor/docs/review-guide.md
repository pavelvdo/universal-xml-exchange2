# Памятка: ревью кода

Краткий гид для заказчика. Полный протокол агента — [`.cursor/skills/review/SKILL.md`](../skills/review/SKILL.md).

---

## Три уровня контроля

| Уровень | Когда | Команда | `mode=prerelease` |
|---------|-------|---------|-------------------|
| **1. Apply-reviewer** | Автоматически после каждой BSL-задачи в ЗНI | `/opsx:apply` | нет |
| **2. Операционное ревью** | Файл, diff, change в работе | `/review` | нет |
| **3. Предрелизный gate** | Перед выкладкой расширения | `/release-review` | да |

Apply-reviewer проверяет только файлы текущей задачи и блокирует закрытие при MUST_FIX. Это не замена предрелиза.

---

## Когда что вызывать

| Ситуация | Команда |
|----------|---------|
| «Глянь diff / этот модуль / change в работе» | `/review` |
| Тривиальный diff (комментарии, пробелы) — можно сократить до light-review | `/review` (без `--full`) |
| Нужно полное ревью файлов, не light | `/review --full` |
| «Готовим релиз расширения» | `/release-review <расширение>` |
| «Предрелиз по ЗНI, но с архобзором всего расширения» | `/release-review <расширение> <change>` |
| «Предрелиз только по change» | `/release-review <change>` |

Имя расширения — папка cfe в `src/` (см. [`openspec/project.md`](../../openspec/project.md)).

---

## Примеры `/release-review`

```
/release-review КонтурДиадок
/release-review КонтурДиадок diadoc-mchd-fio-display
/release-review diadoc-admin-edo-narrow-semantics
```

---

## Примеры `/review`

```
/review
/review src/cfe/КонтурДиадок/...
/review openspec/changes/my-change
/review --full
```

---

## Отличия `/review` и `/release-review`

| Аспект | `/review` | `/release-review` |
|--------|-----------|-------------------|
| Scope по умолчанию | diff, файл, change | все `.bsl` расширения или change-scoped + Tier 2 |
| Light-review | да (на тривиальном diff) | нет |
| `mode=prerelease` | нет | да |
| Эскалация severity (HIGH→CRITICAL) | нет | да |
| Category 12 Release Readiness | нет | да |
| Tier 2 explorer (архитектура расширения) | нет | да |
| Scope Preview (ambiguous target) | редко | при change-scoped |
| Follow-up extend из отчёта | опционально | при ARCH / MUST_FIX scope — `/opsx:extend --from-review` |

Whitelist и mandatory control из `project.md` проверяются в обоих режимах и в apply-reviewer.

---

## Что ожидать в чате

**`/review`:** «Понял: запускаю ревью…» → карточка с итогом → полный отчёт в `openspec/changes/<change>/reports/review-*.md` или `temp/reports/review-*.md`.

**`/release-review`:** «Понял: запускаю **предрелизное** ревью…» → при change-scoped возможен Scope Preview → карточка с Tier 1/Tier 2 → отчёт с Category 12 / release-hygiene.

После отчёта агент предложит устранить MUST_FIX через writer или extend.

---

## Связанные документы

- Команды: [review.md](../commands/review.md), [release-review.md](../commands/release-review.md)
- Маркеры BSL: [`openspec/project.md`](../../openspec/project.md), [bsl-comment-formats-project.md](bsl-comment-formats-project.md)
- Category 12: [reviewer-checks.md](standard/reviewer-checks.md) §12
