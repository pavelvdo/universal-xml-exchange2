---
description: Стандарты кода 1С/BSL — обязательны при любой работе с кодом 1С
globs: "**/*.bsl"
---

# 1C Coding Standards

Стандарты написания кода на платформе 1С:Предприятие (BSL).

**Приоритет**: 150 (высокий)  
**Применяется**: Всегда при работе с кодом 1С

Полный справочник стандартов (именование, запросы, обработка ошибок, структура модулей, защитные проверки и т.д.) — в вендорских доменных файлах `.cursor/docs/standard/std-NN-<domain>.md`. Индекс: `.cursor/docs/standard/1c-standards-navigator.md`.

Для architect и reviewer: чеклисты — `.cursor/skills/1c-vendor-standards/SKILL.md`; детали — Read соответствующий доменный файл из `.cursor/docs/standard/`. Платформенная документация: `.cursor/docs/platform/Оглавление-1С-документации.md`. Антипаттерны: `.cursor/docs/antipatterns/bsl-antipatterns.md`.

**FIRST ACTION**: При написании или ревью кода 1С обязательно прочитай релевантный доменный файл из `.cursor/docs/standard/` (через navigator) и `.cursor/docs/antipatterns/bsl-antipatterns.md`.

## Release-hygiene и маркеры ЗНИ (rule 17)

Комментарии, попадающие под **Whitelist предрелиза** в [openspec/project.md](../../openspec/project.md) (секция «Форматы и соглашения по комментариям BSL») в рамках scope glob, **exempt от удаления** по AP-040..AP-045 — **кроме AP-053** (содержимое `domain_label` в whitelist-строках: rewrite, не delete). Навигация: [.cursor/docs/bsl-comment-formats-project.md](bsl-comment-formats-project.md), [.cursor/docs/marker-layers-guide.md](marker-layers-guide.md).