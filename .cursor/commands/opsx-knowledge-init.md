---
name: /opsx:knowledge-init
id: opsx-knowledge-init
category: Workflow
description: Bootstrap или re-sync таксономии Knowledge Base (_taxonomy.yaml)
---

Первичный bootstrap таксономии базы знаний (`openspec/knowledge/_taxonomy.yaml`) для существующего проекта без полного `/init-project`, либо идемпотентный re-sync после добавления/удаления базовых конфигураций `src/*/cf` и расширений `src/*/cfe/*`.

**Первое действие:** прочитать `.cursor/skills/openspec-knowledge-init/SKILL.md` и далее идти по его шагам. До прочтения скилла — никаких чтений артефактов или таксономии.

**Output style:** T-CONFIRM (`.cursor/docs/opsx-output-style.md` §5.5) — diff/summary таксономии и подтверждение перед записью.

**Input:** без аргументов и флагов.

**Поведение:**
- Файла `_taxonomy.yaml` нет → построить draft (по домену на каждую конфигурацию и расширение, блок `cross` из `openspec/knowledge/_taxonomy.template.yaml`), показать summary, записать после подтверждения.
- Файл есть → diff текущей таксономии с draft; существующие subdomains не удаляются, только добавляются отсутствующие.
- `_index.yaml` и файлы фактов (KB-*.md) команда не трогает.
