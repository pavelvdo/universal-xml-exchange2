# Architect Report Schema

Single source of truth для YAML front-matter отчётов архитектора и `_manifest.yaml`.

## YAML Front-matter (в каждом отчёте)

Каждый отчёт, генерируемый `onec-code-architect`, обязан начинаться с YAML-блока:

```yaml
---
report_type: architecture | deep-analysis | task-readiness | fix-quality | slice-transition | tz-review | adr-extraction | plan-review
generated_at: YYYY-MM-DD
agent: onec-code-architect
mode: <из MODE списка>
scope:
  change: <change-name или null>
  slices: [S1, S2, ...]
  files: [src/...]
  modules: [ОбщийМодуль.Имя, ...]
  capabilities: [...]
related_reports:
  - reports/exploration-YYYY-MM-DD.md
  - reports/trace-analysis-YYYY-MM-DD.md
confidence: high | medium | low
open_questions_count: N
superseded_by: null | reports/architecture-YYYY-MM-DD-v2.md
---
```

## `reports/_manifest.yaml` (в каждом change)

Индекс всех отчётов архитектора в рамках ЗНИ. Оркестратор обновляет его при добавлении новых отчётов.

```yaml
reports:
  - file: architecture-2026-04-20.md
    type: architecture
    scope:
      files: [src/ДО2/cf/BusinessProcesses/Согласование/Forms/ФормаИзменениеПараметров/Ext/Form/Module.bsl]
    status: active | superseded
    legacy: false
```

Поле `legacy: true` используется для старых отчётов, сгенерированных до введения YAML front-matter.
