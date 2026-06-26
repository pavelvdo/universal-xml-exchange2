# Маркеры ЗНИ — четыре слоя (обзор без скриптов)

Единое поле смысла: **`domain_label`** (= `comment_suffix` в `proposal.md`). Канон синтаксиса и запреты — в [openspec/project.md](../../openspec/project.md) § «Форматы и соглашения по комментариям BSL».

## Четыре слоя

| Слой | Где | Что содержит |
|------|-----|--------------|
| **1. Metadata** | `openspec/changes/<name>/proposal.md` → `## Metadata (comment markers)` | `developer`, `comment_suffix` (domain_label), `marker_style` (`canonical` \| `minimal`) |
| **2. SSOT** | `openspec/project.md` | `defaultDeveloper`, `cfMarkerPrefix`, whitelist, канон domain_label, MARKER-MERGE-001 |
| **3. Transport** | Вычисляется в `/opsx:apply` | `open_marker` / `close_marker` (cfe или cf по scope задач) → writer §3a |
| **4. BSL + гигиена** | `src/**/*.bsl` | Фактические строки `// +++ …`, `// --- …`, `{cfMarkerPrefix} …`; AP-040 / AP-051 / AP-053 |

```text
Metadata (proposal) → SSOT (project.md) → Transport (apply) → BSL (src) → Review (AP-*)
```

## Как посмотреть по change

**Команда:** `/opsx:status <name>` — блок **«Маркеры»**: developer, comment_suffix, marker_style, флаг process-only, preview open/close.

**Вручную:** открыть `openspec/changes/<name>/proposal.md`, секция `## Metadata (comment markers)`.

**Metadata Gate (`/opsx:new`):** согласует слой 1 — только `comment_suffix` (описание); ФИО из `defaultDeveloper` в project.md или отдельный текстовый шаг; preview с датой — иллюстрация transport, SSOT metadata — proposal. Полный scope-specific preview — в status/apply после tasks.

## Как посмотреть SSOT проекта

Read [openspec/project.md](../../openspec/project.md):

- `#### Разработчик по умолчанию` — `defaultDeveloper`, `cfMarkerPrefix`
- `#### Whitelist предрелиза` — scope glob и префиксы
- `#### Канон маркеров (domain_label)` — таблица open/close по scope, GOOD/BAD, запреты

Навигация: [bsl-comment-formats-project.md](bsl-comment-formats-project.md).

## Как посмотреть BSL (слой 4)

Поиск в IDE / ripgrep по scope из таблицы канона:

| Scope | Паттерн поиска |
|-------|----------------|
| PAO cfe | `// +++`, `// ---` в `src/ЭДО ПАО/cfe/` |
| ДО3 cfe | `// +++`, `// ---` в `src/ДО3 Демо/cfe/` |
| EA cf | значение `cfMarkerPrefix` из project.md в `src/ЭДО и ЭА/cf/` |

Расширение `src/ЭДО ПАО/cfe/рг_РусГидро/` — **вне** канона ЗНИ (`+++`/`---`); не смешивать с whitelist PAO/ДО3.

## Гигиена (AP-040 / AP-051 / AP-053)

| Правило | Смысл для whitelist-маркеров |
|---------|------------------------------|
| **AP-040** | Whitelist exempt — **не удалять** пару; process-метки вне whitelist — удалить |
| **AP-051** | Сжимать смежные пары одного domain_label / developer / даты |
| **AP-053** | Содержимое domain_label осмысленное, без process-only; remediation — **переписать**, не delete |

Change-scoped: `/review` с metadata из proposal. Полный scope: `/release-review`.

Каталог: [bsl-antipatterns.md](antipatterns/bsl-antipatterns.md), индекс — `.cursor/rules/bsl-antipatterns.mdc`.

## Whitelist vs содержимое (кратко)

> Whitelist защищает **пару от удаления** (AP-040) и задаёт **формат** (AP-051).  
> **Текст** открывающих и однострочных whitelist-строк проверяет **AP-053**.
