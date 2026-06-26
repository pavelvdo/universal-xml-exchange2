---
name: /opsx:knowledge-audit
id: opsx-knowledge-audit
category: Workflow
description: Аудит Knowledge Base — факты, TTL, якоря, индекс, метрики, синхронизация таксономии
---

Аудит базы знаний OpenSpec: актуальность фактов, сверка якорей с кодом (read-repair), пересборка индекса, метрики использования, синхронизация таксономии с кодовой базой, проверка whitelist структуры.

**Первое действие:** прочитать `.cursor/skills/openspec-knowledge-audit/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких чтений индекса, фактов или кода.

**Input:** `/opsx:knowledge-audit [флаги]`. Без флагов — full-revisit (anchors + Reuse Value Test для всех фактов в scope).

**Флаги** (полное описание — в SKILL):
- `--domain <name>` — ограничить домен/субдомен;
- `--overdue` — только факты с истёкшим TTL / `stale`;
- `--status <s>` / `--ids KB-NNNN,...` / `--action <a>` — фильтры и прямые действия;
- `--no-reuse-test` — только verify/TTL без Reuse Value Test;
- `--reindex` — пересобрать `_index.yaml` из `.md`-файлов;
- `--metrics` — отчёт по счётчикам;
- `--taxonomy-sync` — отчёт о расхождениях `_taxonomy.yaml` ↔ `src/*/cfe/*`;
- `--from-archive <name>` — повторно применить протокол archive шаг 5.5 к архивной ЗНИ;
- `--structure-check` — только проверка whitelist `openspec/knowledge/`.

**Примеры:**

```text
/opsx:knowledge-audit
/opsx:knowledge-audit --overdue
/opsx:knowledge-audit --domain KonturDiadok --metrics
/opsx:knowledge-audit --from-archive 2026-04-25-do2-pavlik-merge-roli-avtopodstanovka
```
