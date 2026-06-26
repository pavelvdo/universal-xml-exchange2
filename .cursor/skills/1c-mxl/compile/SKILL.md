---
name: 1c-mxl-compile
description: "Compile a 1C spreadsheet document (MXL/Template.xml) from a JSON definition. Use when generating print form layouts from a DSL specification."
---

# 1C MXL Compile — Spreadsheet Layout Compiler from DSL

Takes a compact JSON definition and generates a correct Template.xml for a 1C spreadsheet document. The agent describes *what* is needed (areas, parameters, styles), the script ensures XML *correctness* (palettes, indices, merges, namespaces).

## Usage

```
1c-mxl-compile <JsonPath> <OutputPath>
```

| Parameter | Required | Description |
|-----------|:--------:|-------------|
| JsonPath | yes | Path to JSON layout definition |
| OutputPath | yes | Path for generated Template.xml |

## Command

```powershell
powershell.exe -NoProfile -File skills/1c-mxl/compile/scripts/mxl-compile.ps1 -JsonPath "<path>.json" -OutputPath "<path>/Template.xml"
```

## Workflow

1. Write JSON definition (Write tool) → `.json` file
2. Run `1c-mxl-compile` to generate Template.xml
3. Run `1c-mxl-validate` to verify correctness
4. Run `1c-mxl-info` to verify structure

**If creating a layout from an image** (screenshot, scanned print form) — determine column boundaries and proportions, then use `"Nx"` widths + `"page"` for automatic size calculation.

## JSON DSL Schema

Full format: JSON with `version`, `page`, `areas` (name, parameters, columnSets). Use Read on existing Template.xml or MXL examples for structure.

Brief structure:

```
{ columns, page, defaultWidth, columnWidths,
  fonts: { name: { face, size, bold, italic, underline, strikeout } },
  styles: { name: { font, align, valign, border, borderWidth, wrap, format } },
  areas: [{ name, rows: [{ height, rowStyle, cells: [
    { col, span, rowspan, style, param, detail, text, template }
  ]}]}]
}
```

Key rules:
- `page` — page format (`"A4-landscape"`, `"A4-portrait"` or number). Automatically calculates `defaultWidth` from sum of `"Nx"` proportions
- `col` — 1-based column position
- `rowStyle` — auto-fills empty cells with style (borders across full width)
- Fill type is determined automatically: `param` → Parameter, `text` → Text, `template` → Template
- `rowspan` — vertical cell merging (rowStyle accounts for occupied cells)

## Поиск в репозитории

Используй поиск по репозиторию (Grep/SemanticSearch) для нахождения примеров существующих макетов. Используй Glob/Read для проверки имён объектов, используемых в параметрах.
