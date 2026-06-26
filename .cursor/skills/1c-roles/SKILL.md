---
name: 1c-roles
description: Toolkit for 1C roles - create from JSON DSL, analyze rights. Use when working with access rights and security in 1C configurations.
---

# 1C Roles - Access Rights Toolkit

Инструменты для работы с ролями и правами доступа в 1С. Поддержка JSON DSL для быстрого создания ролей.

## Quick Start

### Создать роль из JSON
```bash
# 1. Создать JSON описание прав
# 2. Скомпилировать в Rights.xml
1c-roles compile role-definition.json Roles/МояРоль/Rights.xml
```

### Проанализировать существующую роль
```bash
# Показать все права роли
1c-roles info Roles/Администратор/Rights.xml
```

## Включенные Skills

### 1. compile - Создание роли из JSON DSL
**Путь**: `1c-roles/compile/SKILL.md`
- Создает Rights.xml из компактного JSON
- Поддержка всех типов прав (Read, Insert, Update, Delete, View, etc)
- Настройка RLS (Row Level Security)
- Шаблоны ограничений
- *Примечание: Валидация структуры встроена в процесс компиляции.*

### 2. info - Анализ прав роли
**Путь**: `1c-roles/info/SKILL.md`
- Показывает права по объектам
- Выводит RLS ограничения
- Анализирует шаблоны ограничений

## Workflow

```yaml
Создание роли:
  1. Определить набор прав (JSON DSL)
  2. Скомпилировать: 1c-roles compile
  3. Интегрировать в конфигурацию

Анализ прав:
  1. Проанализировать: 1c-roles info
  2. Сравнить с требованиями
  3. Скорректировать при необходимости
```

