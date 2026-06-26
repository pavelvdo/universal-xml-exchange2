---
name: 1c-mxl
description: Complete toolkit for 1C spreadsheet templates (MXL) - compile from JSON DSL, decompile to JSON, analyze structure, validate. Use when working with print forms and tabular documents.
---

# 1C MXL - Spreadsheet Templates Toolkit

Полный набор инструментов для работы с макетами табличных документов 1С (Template.xml). Поддержка JSON DSL для быстрого создания печатных форм.

## Quick Start

### Создать макет из JSON
```bash
# 1. Создать JSON описание макета
# 2. Скомпилировать в Template.xml
1c-mxl compile template-design.json Documents/ЗаказКлиента/Templates/ПечатнаяФорма/Ext/Template.xml
# 3. Валидировать
1c-mxl validate Documents/ЗаказКлиента/Templates/ПечатнаяФорма/Ext/Template.xml
```

### Декомпилировать существующий макет
```bash
# Преобразовать Template.xml в компактный JSON
1c-mxl decompile Documents/ЗаказКлиента/Templates/ПечатнаяФорма/Ext/Template.xml template.json
```

## Включенные Skills

### 1. compile - Компиляция из JSON DSL
**Путь**: `1c-mxl/compile/SKILL.md`
- Компилирует JSON (20-30 строк) в Template.xml (200-500+ строк)
- Автоматически создает области, параметры, колонки
- Поддержка всех типов ячеек (Text, Number, Date, Picture)

### 2. decompile - Декомпиляция в JSON DSL
**Путь**: `1c-mxl/decompile/SKILL.md`
- Обратная операция к compile
- Извлекает структуру в компактный JSON
- Сохраняет форматирование и параметры

### 3. info - Анализ структуры
**Путь**: `1c-mxl/info/SKILL.md`
- Выводит именованные области
- Показывает параметры
- Перечисляет наборы колонок

### 4. validate - Валидация
**Путь**: `1c-mxl/validate/SKILL.md`
- Проверяет структурные ошибки
- Валидирует ссылки на области
- Проверяет параметры

## Workflow

```yaml
Создание печатной формы:
  1. Спроектировать структуру (JSON DSL)
  2. Скомпилировать: 1c-mxl compile
  3. Валидировать: 1c-mxl validate
  4. Интегрировать в конфигурацию

Редактирование существующей:
  1. Декомпилировать: 1c-mxl decompile
  2. Отредактировать JSON
  3. Скомпилировать: 1c-mxl compile
  4. Валидировать: 1c-mxl validate
```

