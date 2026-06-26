---
name: 1c-form-validate
description: "Validate structural correctness of an exported 1C managed form (Form.xml). Read-only: use after Configurator export to check structural errors."
---

# 1C Form Validate — Form Structure Validator

Checks exported `Form.xml` of a managed form for structural errors: ID uniqueness, companion element presence, DataPath and command reference correctness.

**Read-only.** This skill validates the dump and does not create, edit, or generate `Form.xml`. If errors are found, fix the form in Configurator/metadata and re-export, or adjust BSL runtime element creation when the UI is built from code.

## Usage

```
1c-form-validate <FormPath>
```

| Parameter | Required | Default | Description |
|-----------|:--------:|---------|-------------|
| FormPath | yes | — | Path to Form.xml file |
| MaxErrors | no | 30 | Stop after N errors |

## Command

```powershell
powershell.exe -NoProfile -File .cursor/skills/1c-forms/validate/scripts/form-validate.ps1 -FormPath "<path>"
```

## Checks Performed

| # | Check | Severity |
|---|-------|----------|
| 1 | Root element `<Form>`, version in known set (2.17, 2.20) | ERROR / WARN |
| 2 | `<AutoCommandBar>` present, id="-1" | ERROR |
| 3 | Element ID uniqueness (separate pool) | ERROR |
| 4 | Attribute ID uniqueness (separate pool) | ERROR |
| 5 | Command ID uniqueness (separate pool) | ERROR |
| 6 | Companion elements (ContextMenu, ExtendedTooltip, etc.) | ERROR |
| 7 | DataPath → references existing attribute | ERROR |
| 8 | Button CommandName → references existing command | ERROR |
| 9 | Events have non-empty handler names | ERROR |
| 10 | Commands have Action (handler) | ERROR |
| 11 | No more than one MainAttribute | ERROR |

## Output

```
=== Validation: DocumentForm ===

[OK]    Root element: Form version=2.20
[OK]    AutoCommandBar: name='FormCommandBar', id=-1
[OK]    Unique element IDs: 96 elements
[OK]    Unique attribute IDs: 38 entries
[OK]    Unique command IDs: 5 entries
[OK]    Companion elements: 86 elements checked
[OK]    DataPath references: 53 paths checked
[OK]    Command references: 2 buttons checked
[OK]    Event handlers: 41 events checked
[OK]    Command actions: 5 commands checked
[OK]    MainAttribute: 1 main attribute

---
Total: 96 elements, 38 attributes, 5 commands
All checks passed.
```

Return code: 0 = all checks passed, 1 = errors found.

## When to Use

- **After Configurator export** to `src/`: verify `Form.xml` from the dump
- **When debugging** structural issues: IDs, companions, DataPath

## Workflow

1. Change form in **Configurator** (or adjust BSL that builds elements at runtime — no `Form.xml` diff from that path)
2. `1c-form-validate` — run validation on shipped `Form.xml`
3. Fix issues in Configurator / metadata, re-export
4. `1c-form-info` — verify structure visually
