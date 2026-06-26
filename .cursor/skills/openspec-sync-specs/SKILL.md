---
name: openspec-sync-specs
description: Sync delta specs from a change to main specs. Use when the user wants to update main specs with changes from a delta spec, without archiving the change.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.1.1"
---

Sync delta specs from a change to main specs.

This is an **agent-driven** operation - you will read delta specs and directly edit main specs to apply the changes. This allows intelligent merging (e.g., adding a scenario without copying the entire requirement).

**Output style:** **§3a** [`.cursor/rules/chat-output-budget.mdc`](../../rules/chat-output-budget.mdc) — в **чат** только материальный итог: одна строка «спеки уже совпадают с дельтой» **или** «спеки обновлены: `<paths>`» + одна команда «Дальше». Перечень requirements/scenarios и diff — **только** в `reports/spec-sync-<name>-YYYY-MM-DD.md`, не в чат. Шаблон **T-CONFIRM** §5.5 — укороченно. Перед выводом — self-check §7 + §3a.

**Input**: Optionally specify a change name. If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **Select the change**

   If a name is provided, use it. Otherwise:
   - Infer from conversation context if the user mentioned a change
   - Auto-select if only one active change exists
   - If multiple active changes: run `openspec list --json`, show changes that have delta specs (under `specs/` directory), and use **AskUserQuestion tool** to let user select

   Always announce: "Using change: <name>" and how to override.

2. **Find delta specs**

   Look for delta spec files in `openspec/changes/<name>/specs/*/spec.md`.

   Each delta spec file contains sections like:
   - `## ADDED Requirements` - New requirements to add
   - `## MODIFIED Requirements` - Changes to existing requirements
   - `## REMOVED Requirements` - Requirements to remove
   - `## RENAMED Requirements` - Requirements to rename (FROM:/TO: format)

   If no delta specs found: в **чат** одна строка: «Дельта-spec не найдена. Дальше: `/opsx:archive <name>` или продолжите постановку.» — stop.

3. **For each delta spec, apply changes to main specs**

   For each capability with a delta spec at `openspec/changes/<name>/specs/<capability>/spec.md`:

   a. **Read the delta spec** to understand the intended changes

   b. **Read the main spec** at `openspec/specs/<capability>/spec.md` (may not exist yet)

   c. **Apply changes intelligently**:

      **ADDED Requirements:**
      - If requirement doesn't exist in main spec → add it
      - If requirement already exists → update it to match (treat as implicit MODIFIED)

      **MODIFIED Requirements:**
      - Find the requirement in main spec
      - Apply the changes - this can be:
        - Adding new scenarios (don't need to copy existing ones)
        - Modifying existing scenarios
        - Changing the requirement description
      - Preserve scenarios/content not mentioned in the delta

      **REMOVED Requirements:**
      - Remove the entire requirement block from main spec

      **RENAMED Requirements:**
      - Find the FROM requirement, rename to TO

   d. **Create new main spec** if capability doesn't exist yet:
      - Create `openspec/specs/<capability>/spec.md`
      - Add Purpose section (can be brief, mark as TBD)
      - Add Requirements section with the ADDED requirements

4. **Summary (файл + тонкий чат)**

   После применения изменений:
   - Сформировать `openspec/changes/<name>/reports/spec-sync-<name>-YYYY-MM-DD.md`: какие capability затронуты, какие требования добавлены/изменены/удалены (кратко).
   - **Чат:** если **ни один** файл `openspec/specs/**/spec.md` не изменился (дельта уже отражена в main) — одна строка: «Спеки уже синхронизированы с дельтой. Дальше: `/opsx:archive <name>`.» (или `/opsx:verify <name>` по ситуации). Если были **фактические** правки файлов — одна строка: «Спеки обновлены: `<path1>`[, `<path2>`…]. Дальше: `/opsx:verify <name>`.» Без списка requirements в чате.

5. **Post-verification (без diff в чате)**

   После записи main spec: точечно `Read`/сверка с дельтой; несоответствия — STOP с коротким сообщением. **Не** выводить diff и **не** спрашивать «Changes look correct?» в чате (non-event); полнота — в файле шага 4.

**Delta Spec Format Reference**

```markdown
## ADDED Requirements

### Requirement: New Feature
The system SHALL do something new.

#### Scenario: Basic case
- **WHEN** user does X
- **THEN** system does Y

## MODIFIED Requirements

### Requirement: Existing Feature
#### Scenario: New scenario to add
- **WHEN** user does A
- **THEN** system does B

## REMOVED Requirements

### Requirement: Deprecated Feature

## RENAMED Requirements

- FROM: `### Requirement: Old Name`
- TO: `### Requirement: New Name`
```

**Key Principle: Intelligent Merging**

Unlike programmatic merging, you can apply **partial updates**:
- To add a scenario, just include that scenario under MODIFIED - don't copy existing scenarios
- The delta represents *intent*, not a wholesale replacement
- Use your judgment to merge changes sensibly

**Output On Success (чат)**

```
Спеки обновлены: openspec/specs/<capability>/spec.md[, …]. Дальше: `/opsx:verify <change-name>`.
```
или
```
Спеки уже синхронизированы с дельтой. Дальше: `/opsx:archive <change-name>`.
```

**Guardrails**
- Read both delta and main specs before making changes
- Preserve existing content not mentioned in delta
- If something is unclear, ask for clarification
- Детали изменений — в `reports/spec-sync-*.md`, не потоком в чат
- The operation should be idempotent - running twice should give same result
