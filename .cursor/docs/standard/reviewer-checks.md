### 1. Standards Compliance
```yaml
Check against:
  - БСП (Библиотека стандартных подсистем)
  - 1C coding conventions
  - Naming conventions (Russian/English)
  - Code structure and organization
  - Documentation requirements
```

For deep vendor-standards compliance (transactions, event handlers, queries, locking): Read `.cursor/skills/1c-vendor-standards/SKILL.md`, check all domains touched by reviewed code. For details: Read the relevant domain file from `.cursor/docs/standard/` (see `1c-standards-navigator.md`). Do NOT read platform documentation routinely — only if the reviewed code uses an unfamiliar platform mechanism.

Integration exchange trigger: if reviewed code prepares JSON/HTTP/RMQ messages or data exchange (`ЗаписатьJSON`, `ЗаписьJSON`, `ПрочитатьJSON`, `СтруктураСообщения`, `Вставить("GUID"`, `HTTP`, `RMQ`, `Обмен`, `Выгруз`, `Загруз`), read `.cursor/docs/standard/std-12-integration-exchange.md` and apply AP-048..AP-050 via the anti-pattern registry. Do not treat AP-050 as a mass lint for all user strings: only flag blocking messages that stop a scenario and do not explain actual/expected state.

### 2. Performance Analysis
```yaml
Identify:
  - N+1 query problems
  - Missing indexes
  - Inefficient loops
  - Unnecessary database calls
  - Memory leaks
  - Slow algorithms
```

### 3. Security Review
```yaml
Detect:
  - SQL injection vulnerabilities
  - XSS in forms
  - Insufficient access control
  - Hardcoded credentials
  - Unsafe data handling
  - RLS bypass attempts
```

### 4. Code Quality
```yaml
Evaluate:
  - Cyclomatic complexity
  - Code duplication
  - Function length
  - Parameter count
  - Error handling
  - Testability
  - Stub/placeholder code returning empty or dummy values (empty Thumbprint, hardcoded "TODO", always-false conditions) — HIGH — AP-024. See anti-pattern registry
  - AP-007: Parameter overwrite / collection mutation — HIGH/MEDIUM. See anti-pattern registry
  - Duplicated magic constant: same numeric literal (not 0/1/-1) or same string literal appears 2+ times in module — MEDIUM. Exception: query text, structure keys, metadata names, 0/1/-1/Истина/Ложь
  - Mixed responsibilities: procedure >40 lines combining 3+ distinct concerns (rights check, transaction management, business logic, persistence/write, logging, UI feedback) — MEDIUM. Sign: procedure could be split into independent functions without passing internal state
```

### 5. Extension Annotations (1C Extensions)
```yaml
Check:
  - &Перед/&После applied to a function (not procedure) — CRITICAL — AP-022. See anti-pattern registry
  - &Вместо used where &Перед/&После would suffice — HIGH
  - &ИзменениеИКонтроль used by default without justification — HIGH. This annotation is brittle and should be avoided if the task can be solved via &После, &Перед, or &Вместо. Reviewer MUST demand justification for its use.
  - &ИзменениеИКонтроль: code outside #Вставка/#Удаление blocks differs from base — HIGH (prerelease: CRITICAL). Signs: variable renaming, formatting/indent changes, refactoring outside blocks, adding/removing #Область/#КонецОбласти in base (typed) code, NEW CODE added outside directive blocks. Any of these breaks extension applicability.
  - Business logic placed directly inside #Вставка block instead of delegating to a separate procedure — MEDIUM
  - AP-046: Hook-scope early return — feature-flag or business guard suppresses entire body of intercepted procedure (&Перед/&После/&Вместо/&ИзменениеИКонтроль) — HIGH; for &Вместо without ПродолжитьВызов — CRITICAL. See anti-pattern registry
  - &ИзменениеИКонтроль VERIFICATION PROCEDURE (mandatory when such methods exist):
    1. Identify all methods with &ИзменениеИКонтроль in the reviewed file.
    2. For each: load the base method from cf/ (path: replace cfe/<ExtName>/ with cf/ in file path).
    3. Extract code OUTSIDE #Вставка/#КонецВставки and #Удаление/#КонецУдаления blocks from the extension method.
    4. Diff against base: any divergence (added lines, deleted lines, modified lines) — CRITICAL (prerelease) / HIGH (normal).
    5. If base file not found — mark NEEDS_MANUAL_REVIEW.
```

### 6. Module Structure
```yaml
Check:
  - Missing #Область markup entirely — MEDIUM (only if module > 100 lines; for modules ≤ 100 lines — LOW)
  - Wrong order: ПрограммныйИнтерфейс → СлужебныйПрограммныйИнтерфейс → СлужебныеПроцедурыИФункции — MEDIUM
  - Duplicate #Область or #КонецОбласти directives — HIGH
  - Export method placed in #Область СлужебныеПроцедурыИФункции (private region) — MEDIUM
  - Export procedure/function in form module (Forms/*/Module.bsl) — HIGH — AP-033. Exception: Подключаемый_* (BSP attachable commands), callbacks via ОписаниеОповещения(..., ЭтотОбъект). See anti-pattern registry
  - Module header comment name does not match actual module name — LOW
  - No module header comment (module purpose) — LOW
```

### 7. Method Documentation
```yaml
Check:
  - Export method without header comment (purpose, parameters, return value) — MEDIUM
  - Event handler without description — LOW
  - Header format does not match BSP template (Параметры/Возвращаемое значение/Пример) — LOW
```

### 8. Extension Naming
```yaml
Check:
  - Intercepted method (&Вместо/&Перед/&После) without extension prefix — HIGH
  - Own new method (not intercept) with extension prefix — MEDIUM
  - Export own method without unique readable name — MEDIUM
  - Inconsistent prefix usage: export method without extension prefix in module that contains other exports WITH prefix — MEDIUM. Exception: intercept methods (&Вместо/&Перед/&После of base method) naturally lack prefix
```

### 9. Code Cleanliness
```yaml
Check:
  Principle: comments describe code intent, not change history.
  Do NOT treat as release-hygiene or remove: directives #Вставка, #КонецВставки, #Удаление, #КонецУдаления — they are 1C extension override syntax, required for correct merge.

  Project-level whitelist: if openspec/project.md contains section «Форматы и соглашения по комментариям BSL»
  with subsection «Whitelist предрелиза» (table: prefix after //, optional regex on full // line, scope glob per row),
  comments in files matching that scope that match the whitelist are exempt from AP-040 **removal** (not from AP-053 content check).
  Read project.md before flagging changelog-style markers (e.g. +++/---) in whitelisted lines.

  Whitelisted marker normalization (AP-051):
    - When changelog markers fall under project whitelist (openspec/project.md → Whitelist предрелиза),
      they are NOT removed (AP-040 does not apply), but they MUST be compact:
      adjacent +++/--- blocks for the same domain_label / [ID#NNN] and the same semantic change
      are merged into one outer block.
    - When merged opening markers carry different dates, keep the latest.
    - Do NOT propose marker merging where AP-040 applies (markers not in whitelist) —
      correct finding there is removal, not merging.

  Whitelisted marker content (AP-053):
    - Open and single-line cf whitelist markers MUST contain meaningful domain_label (Russian).
    - Process-only suffix (release-review, findings, kebab change name, etc.) — MEDIUM; remediation: rewrite text, NOT delete pair.
    - See openspec/project.md § Канон domain_label for forbidden list.

  Release hygiene (process metadata in comments only):
    - Whitelisted ЗНИ-пары // +++ / // --- (openspec/project.md) — не AP-040 delete (см. AP-040 whitelist); содержимое — AP-053.
    - Устаревшие / вне-whitelist маркеры: // НАЧАЛО/КОНЕЦ Изменения, // РГИТС ..., Заявка №, Подрядчик:, date-author без шаблона whitelist — MEDIUM
    - JSDoc над Процедура/Функция: сноски (см. <kebab-change> ...), пути reports/openspec/temp/reports, упоминание *.md как доказательства решения — MEDIUM (AP-040)
    - Commented-out old code with replacement markers
      ("Оригинальный код:", "Новый код:", "Старый вариант:") — MEDIUM
    - Work instructions in code
      ("Добавить в xml...", "Перенести в...", "TODO перенести") — MEDIUM
    - Design/process artifact references in comments — MEDIUM:
        Short-form refs: // Design В§3, // D11, // F5
        Natural-language refs: // По design Decision 6 (fix-signing-result): ...
        Change names (kebab-case in comments): fix-signing-result, add-feature-X
        Process terms in comments: design, decision, proposal, architecture,
          exploration, root cause, trace-analysis, ЗНИ, ADR
        Task number refs: п. 3.1, задача 2.2, Decision N
      Detection: look for English process nouns in Russian comment lines,
        kebab-case identifiers, and numbered decision/task references.

  Code waste:
    - Dead code — see category 15 (Obsolete and Unused Code)
    - Logic duplication between modules — MEDIUM
    - Commented-out code without explanation — MEDIUM
```

### 10. Specific 1C Patterns
```yaml
Check via Anti-pattern Registry (see category 16):
  AP-001: Server scope by default in form modules (Перем/Тип without &НаКлиенте) — HIGH
  AP-002: Client-only methods in server context — HIGH
  AP-003: ЭтаФорма instead of ЭтотОбъект in callbacks — HIGH
  AP-004: Defensive check on fixed-contract source — HIGH
  AP-005: Свойство() on non-Structure type — HIGH
  AP-006: Defensive cake (stacked redundant checks) — HIGH
  AP-008: Попытка wrapping deterministic operation — CRITICAL
  AP-009: Silent degradation in Исключение — HIGH
  AP-010: Traceless exception suppression — HIGH
  AP-011: ТекущаяДата() instead of ТекущаяДатаСеанса() — CRITICAL
  AP-012: Сообщить() instead of BSP methods — HIGH
  AP-013: Query in loop (N+1) — HIGH
  AP-014: Attribute via reference dot notation — HIGH
  AP-017: Inverted early exit (condition tied to exit instead of action) — MEDIUM
  AP-018: ТекущаяСтрока() on form TableValue attribute (use Элементы.<Name>.ТекущиеДанные) — HIGH
  AP-019: Новый Тип(...) instead of Тип(...) for type descriptor — HIGH
  AP-020: Missing explicit directive on procedure/function in form module — HIGH
  AP-021: Fail-fast violation (silent skip on structural precondition) — HIGH
  AP-025: User-facing string without НСтр() — MEDIUM
  AP-027: Guard-then-catch (Попытка after guard validating same value) — HIGH
  AP-028: Check-after-establish (property/attribute check after type established) — HIGH
  AP-029: Defense stack (Попытка + Свойство as contract uncertainty compensation) — HIGH/CRITICAL
  AP-030: Hidden partial result (Попытка+Продолжить/Возврат without user feedback) — HIGH
  AP-032: Inconsistent persistent state (Попытка + persistent write + no re-raise + downstream dependency) — CRITICAL
  AP-047: Substituted Authority — local implementation replaces delegation to the owner of behavior (base/BSP/platform/common module) — HIGH
  AP-048: Manual reference/GUID serialization in exchange code — HIGH
  AP-049: Numeric string via Строка() + whitespace cleanup — MEDIUM
  AP-050: Uninformative blocking user message — MEDIUM (REFACTOR by default; MUST_FIX when user cannot recover)

Remain inline (not in registry):
  - Ternary operator ?() — MEDIUM
  - Excessive info logging inside loop or 3+ info-level calls — LOW
```

### 11. Band-Aid Detection
```yaml
Check via Anti-pattern Registry:
  AP-016: Band-aid fix — HIGH (see .cursor/rules/verified-cause-gate.mdc)

Remain inline:
  - Design-prescribed anti-pattern: guard in code matches design.md recommendation,
    but violates rule 14 — HIGH (tag: design-prescribed)
```

### 12. Release Readiness (checked only in mode=prerelease)
```yaml
Check:
  - Typos and encoding errors in user-facing strings: mixed Cyrillic/Latin chars (С vs C, а vs a, о vs o, е vs e), spelling errors in НСтр/ПоказатьПредупреждение arguments — HIGH
  - Stub/placeholder code — see category 4 Code Quality (always checked, not prerelease-only)
  - Попытка/Исключение without logging — moved to category 10 (always-checked, HIGH). See category 10 for detection; do NOT duplicate finding here
```

### 13. Transactions and Locking
```yaml
Check:
  AP-015: Transaction without safety pattern (НачатьТранзакцию without Попытка+Зафиксировать+Отменить) — CRITICAL. See anti-pattern registry
  AP-023: User interaction (ПоказатьВопрос, Предупреждение, Сообщить) inside transaction — HIGH. See anti-pattern registry
  Remain inline:
  - Read-then-write without БлокировкаДанных in concurrent scenario — HIGH
  - Nested НачатьТранзакцию() without justification — MEDIUM
```

### 14. Resource Leaks
```yaml
Check:
  - COMОбъект (Новый COMОбъект()) created without Попытка/Исключение ensuring release — HIGH
  - HTTPСоединение/FTPСоединение created but not wrapped in Попытка for timeout/error handling — MEDIUM
  - File reader/writer (ЧтениеXML, НачатьЗаписьXML, ЧтениеJSON, ЗаписьJSON, ТекстовыйДокумент.Открыть) opened without close in error path — MEDIUM
  - Temporary file created (ПолучитьИмяВременногоФайла) without cleanup in error path — LOW
```

### 15. Obsolete and Unused Code
```yaml
Check:
  Unused procedures/functions (std-06 Рї.2, std-01 Рї.2.2):
    - For each Процедура/Функция in reviewed files, verify at least one call exists
      in the extension scope (Grep by name across all .bsl in extension directory).
      Exceptions: event handlers (ОбработчикСобытия), BSP-registered commands
      (ExternalDataProcessorInfo / ВнешняяОбработкаСведения), callback procedures
      passed as string to ОписаниеОповещения or ОбработкаВнешнегоСобытия.
    - Unused non-export procedure/function — MEDIUM
    - Unused export procedure/function (no callers found in extension scope) — HIGH
      (broken public contract or dead API surface)

  Obsolete code markers (std-10 Рї.3.1):
    - Procedure/function with comment "Устарела:" or "Deprecated" — MEDIUM
      Reason: document replacement or plan removal per std-10 Рї.3.1
    - #Область УстаревшиеПроцедурыИФункции present — LOW
      (informational: track obsolete API surface; confirm replacement is documented)
    - Obsolete procedure (marked Устарела:/Deprecated) still called from
      non-obsolete code — HIGH (caller must migrate to replacement per std-10)

  Unused parameters (std-06):
    - Procedure/function parameter never referenced in body — LOW
```

## AVAILABLE TOOLS

### Primary validation (BSL LSP NOT_CONNECTED)

Use MCP as primary quality gate:

```yaml
1. Syntax: user-1c-syntax-checker-syntaxcheck(code) — parse errors
2. Logic: user-1c-code-checker-check_1c_code(code, "logic") — recommendations
3. Continue with reduced diagnostics; do NOT block review
4. If BSL LSP later available — switch to bsl_lsp_* for full diagnostics
```

### BSL LSP Bridge (when available)
```yaml
bsl_lsp_diagnostics(file_path):
  - Get all diagnostics: errors, warnings, hints
  - Categorize by severity
  - Prioritize fixes

bsl_lsp_symbols(file_path):
  - Get function list
  - Analyze complexity
  - Check naming

bsl_lsp_format(file_path):
  - Validate formatting
  - Suggest improvements
```

### Skills
```yaml
1c-bsp:
  - Check БСП patterns and registration
  - Validate command structure
  - Verify ExternalDataProcessorInfo

1c-agent-patterns:
  - Agent delegation patterns, review workflow
  - Spec-driven validation
```

### MCP Servers
```yaml
user-1c-syntax-checker-syntaxcheck(code):
  - Validate BSL syntax
  - Check for parse errors

user-1c-code-checker-check_1c_code(code, check_type):
  - Logic analysis via 1С:Напарник
  - Performance recommendations
  - Best practices

user-PROJECT-codemetadata (project-specific MCP)-codesearch(query):
  - Find similar code
  - Check for duplicates
  - Learn from existing solutions

user-PROJECT-graph (project-specific MCP)-search_metadata(query):
  - Analyze metadata dependencies
  - Check for circular references
  - Validate object relationships
```

### RLM Integration (когда подключен)
```yaml
status: NOT_CONNECTED
Когда доступен:
  user-rlm-toolkit-rlm_route_context(query) — context from past reviews
  user-rlm-toolkit-rlm_add_hierarchical_fact(...) — record findings
  user-rlm-toolkit-rlm_record_causal_decision(...) — document choices
```

## REVIEW WORKFLOW

### Phase 0: Intent & Reasoning Analysis (выполнять ПЕРВЫМ, если не сработал Skip Gate)

**Skip Gate:** Пропустить Phase 0, если выполняются ВСЕ условия: scope ревью ≤ 10 строк; нет внешних источников данных (API, результаты функций со сложными структурами); максимальная вложенность ≤ 2; только mechanical changes (rename, formatting, regions).

**Обязательные артефакты** — строить до генерации замечаний; включать в отчёт (секция Reasoning Artifacts).

#### 0.1 Артефакт: Intent Map

Для каждой процедуры/функции и каждого значимого блока (цикл, ветвление, Попытка):
- Намерение блока — одно предложение (что делает).
- Ожидаемая сложность — из намерения (сколько строк/уровней вложенности нужно для этой задачи).
- Фактическая сложность — из кода (строки, уровни вложенности).

Формат (пример): procedure → intent; blocks → location, intent, expected_complexity, actual_complexity; sub_blocks при необходимости.

#### 0.2 Артефакт: Contract Map

Для каждого источника данных (параметр, результат вызова API/функции) — таблица обращений к полям в рамках процедуры/блока:
- source, origin (откуда данные).
- field_accesses: field, access, line (или диапазон).

**Типы доступа (access):**
- **DIRECT** — Объект.Поле, без проверок.
- **DEFENSIVE** — Объект.Свойство("Поле", ...), одна проверка.
- **EXPLORATORY** — несколько альтернативных путей к одному семантическому значению.
- **GUARDED** — доступ после guard clause (любая конструкция: Если ТипЗнч, Колонки.Найти, булев флаг, ЗначениеЗаполнено-as-guard, ЕстьРеквизит и др.).

#### 0.3 Артефакт: Knowledge Assessment

Для каждого источника из Contract Map:
- evidence_of_knowledge / evidence_of_ignorance (СЃРїРёСЃРєРё).
- verdict: FULL / PARTIAL / ABSENT.
- explanation (одно предложение).

**Антикруговое правило:** guard (Свойство, ТипЗнч, Колонки.Найти, ЕстьРеквизит, булев флаг и т.п.) НЕ является evidence_of_knowledge для того же источника. Guard — объект проверки, а не доказательство. Evidence = Form.xml, метаданные объекта, текст запроса, документация функции, Resolved Contracts, код вызываемой функции.

#### 0.4 Evaluation Questions (генерация замечаний Phase 0)

После построения артефактов — заполнить таблицу Evaluation Checklist.
ОБЯЗАТЕЛЬНО: для КАЖДОГО вопроса зафиксировать ответ (yes/no + источник/обоснование).
Пропуск вопроса = Phase 0 не завершена; к Phase 1 не переходить.

Evaluation Checklist (включить в отчёт в секции Reasoning Artifacts):

| # | Вопрос | Ответ | Источник/обоснование | Finding (если yes) |
|---|--------|-------|----------------------|--------------------|
| 1 | Proportionality: actual > 3x expected по строкам или > 2x по вложенности? | yes/no | | DISPROPORTIONATE_COMPLEXITY |
| 2 | Consistency: один источник — часть полей DIRECT, часть DEFENSIVE/EXPLORATORY? | yes/no | | CONTRACT_INCONSISTENCY |
| 3 | Knowledge: источники с verdict PARTIAL или ABSENT? | yes/no | | KNOWLEDGE_DEFICIT |
| 4 | Exploratory: поля с access=EXPLORATORY? | yes/no | | CONTRACT_INFERENCE |
| 5 | Попытка as contract compensation: источник PARTIAL/ABSENT + Попытка на его полях? | yes/no | | KNOWLEDGE_DEFICIT + contract-compensating-try |
| 6 | Naming: есть идентификатор, чьё имя (a) отражает постановку/оркестрацию, (b) содержит номер ЗНИ/задачи или kebab-токен change-name, (c) содержит jargon blocklist (`Fallback`, `PostWrite`, `PreWrite`, `Guard`, `Mechanics`/`Механика`, `Gate`/`Гейт`, `Temp`/`Tmp`, `Wrapper`, `Orchestrat`), (d) описывает роль в коде (контейнер, промежуточное, накопитель) без доменного квалификатора, или (e) при наличии в том же scope доменно названных данных — контейнер для них не отсылает к домену? Тест: мысленно убрать реализационное слово (Массив, Новый, Добавляемые) — остаётся ли доменный смысл? (см. AP-031; при наличии `## Naming Signals (evidence)` — Phase 1c приоритетнее) | yes/no | | CLARITY_DEFICIT + Supporting AP-031 (или отдельное finding AP-031 без дублирования с CLARITY в том же месте) |
| 7 | Authority: код локально реализует поведение, у которого есть явный владелец (база, БСП, платформа, общий модуль, внешний контракт), и доступен механизм делегирования владельцу? | yes/no | | AUTHORITY_MISPLACEMENT + Supporting AP-047 |

Completeness gate: 7 строк в таблице. Меньше — Phase 0 не завершена.

Каждый yes → замечание с counterfactual. Supporting: при совпадении с AP указать AP-NNN, не дублировать.

### Phase 1: Initial Analysis
```yaml
1. Check syntax (primary): user-1c-syntax-checker-syntaxcheck(code)
2. Analyze logic: user-1c-code-checker-check_1c_code(code, "logic")
3. If BSL LSP available: bsl_lsp_diagnostics(file_path), bsl_lsp_symbols(file_path)
   Else: proceed with MCP-only; manual structure analysis from Read(file)
```

### Phase 1b: BSL Linter Signals Gate (ОБЯЗАТЕЛЬНО при наличии блока `## Linter Signals (evidence)`)

**Политика:** предупреждения bsl-language-server / `ReadLints` на **in-scope** строках **не откладываются** на предрелизное ревью. Погашение документационного и структурного техдолга позже повышает риск регресса (правки шапок/JSDoc/аннотаций на уже принятом коде).

**In-scope** (любое из):
- строка попадает в `## Review Boundaries` (diff-focused);
- строка в теле/шапке процедуры или функции, созданной или изменённой в текущей задаче writer;
- `[module-level]` в границах ревью.

**Алгоритм (для каждой строки таблицы Linter Signals с severity `warning` или `error`):**

| Шаг | Действие |
|-----|----------|
| 1 | **confirm → MUST_FIX** (default для in-scope warning/error). Severity finding: **MEDIUM** для документации (`MissingReturnedValueDescription`, `MissingParameterDescription`, `MissingDescription`, …); **HIGH** если warning указывает на риск поведения/контракта. |
| 2 | **dismiss** — только с явной причиной в отчёте: `out-of-scope` (вне Review Boundaries), `false-positive` (обоснование + строка кода), `pre-existing-unchanged` (full-ревью: процедура не менялась в текущем diff). |
| 3 | **reclassify** — если warning дублирует Phase 0 / AP finding, объединить; не создавать второй MUST_FIX на то же место. |

**Запрещённые основания для dismiss:** «не блокирует apply», «отложим на prerelease», «стиль», «линтер слишком строгий» без `false-positive`.

**Документация (JSDoc / шапка метода):**
- Экспортные и перехватывающие (`&Вместо`, `&После`, `&Перед`) процедуры/функции in-scope **без** блоков «Параметры:» / «Возвращаемое значение:» (если применимо) → **MUST_FIX**.
- JSDoc размещать **выше** аннотаций расширения (`&Вместо`, …), не между аннотацией и объявлением (std-06 / vendor doc comments).
- Тип коллекции: `Массив из Тип1, Тип2 — …`, не «Массив ссылок» без уточнения элементов.

**Completeness gate Phase 1b:** если в промпте есть `## Linter Signals (evidence)` с ≥1 in-scope warning и ни одного finding с Action MUST_FIX или явного dismiss в отчёте — Phase 1b **не завершена**; Status ≠ PASS.

**Если блок отсутствует или `Linter unavailable`:** зафиксировать в Summary `Linter Signals: unavailable`; для **новых** in-scope `Функция`/`Процедура` без шапки — вручную проверить документацию (Phase 2 п.3) и выдать MUST_FIX при отсутствии.

**Формат finding (Linter):**
```yaml
[MEDIUM] Line N: MissingReturnedValueDescription (bsl-language-server)
  Procedure: <РёРјСЏ>
  Anchor: <строка объявления>
  Action: MUST_FIX
  Issue: Linter warning on in-scope line; not deferred to prerelease
  Fix: <конкретная шапка / перенос JSDoc>
  kind: linter-signal
  LinterRule: MissingReturnedValueDescription
  LinterVerdict: confirm
```

**Status PASS:** только если нет unresolved MUST_FIX из Phase 0, Phase 1b, Phase 1c, Phase 2–2.5 и Standards.

### Phase 1c: Naming Provenance Gate (ОБЯЗАТЕЛЬНО при наличии блока `## Naming Signals (evidence)` с ≥1 match)

**Политика:** артефакты ЗНИ/оркестрации в **идентификаторах** (номер задачи, kebab-токены change-name, jargon blocklist) — AP-031 MUST_FIX. Оркестратор передаёт механический grep-evidence; reviewer обязан confirm/dismiss каждую строку (fail-closed).

**In-scope:** каждая строка таблицы Naming Signals, кроме `Naming Signals: clean` и `Naming scan skipped: no diff`.

**Алгоритм (для каждой строки evidence):**

| Шаг | Действие |
|-----|----------|
| 1 | **confirm → AP-031 MUST_FIX** (default). Severity: **HIGH** если export / `&После` / `&Перед`; иначе **MEDIUM**. |
| 2 | **Remediation:** конкретный доменный синоним (не «переименовать»). |
| 3 | **dismiss** — только с явной причиной: `metadata-name` (likely-metadata + цитата метаданных), `false-positive` (обоснование + строка кода), `pre-existing-unchanged` (full-review: строка не в diff). |

**Completeness gate Phase 1c:** если в промпте есть `## Naming Signals (evidence)` с ≥1 match и ни одного finding AP-031 или явного dismiss в отчёте — Phase 1c **не завершена**; Status ≠ PASS.

**Если блок отсутствует или `Naming scan skipped`:** зафиксировать в Summary `Naming Signals: skipped`; для full-review — дополнительно Phase 0 Q6 и AP-031 pass в Phase 2.

**Формат finding (Naming):**
```yaml
[MEDIUM] Line N: AP-031 meta-name from change artifact
  Procedure: <имя>
  Anchor: <строка объявления или ключ ДопСвойств>
  Action: MUST_FIX
  Issue: Identifier embeds ticket/slug/orchestration jargon (Naming Signals evidence)
  Fix: <конкретный доменный синоним>
  kind: naming-signal
  AP: AP-031
  NamingVerdict: confirm | dismiss (metadata-name | false-positive | pre-existing-unchanged)
```

### Phase 2: Deep Analysis
```yaml
1. Performance check:
   - Scan for queries in loops
   - Check index usage
   - Analyze algorithm complexity
   - Measure database calls

2. Security audit:
   - Check for SQL injection
   - Validate input sanitization
   - Review access control
   - Check for hardcoded secrets

3. БСП compliance:
   - Verify naming conventions
   - Check module structure
   - Validate error handling
   - Review documentation

4. Code quality:
   - Calculate cyclomatic complexity
   - Detect code duplication
   - Check function length
   - Analyze parameter count
   - Fail-fast: scan for silent skips on structural checks (Продолжить, silent Возврат, empty branch when precondition fails on type/property/size/format) — AP-021. See anti-pattern registry
   - Data contract: в†' see AP-004, AP-005, AP-006 in anti-pattern registry (category 16)
   - Design authority: design.md decisions do NOT exempt code from anti-pattern checks. Tag: "design-prescribed anti-pattern".
   - Detect stub/placeholder code: empty Thumbprint, hardcoded "TODO" return values, always-false conditions — HIGH — AP-024 (always, not prerelease-only). See anti-pattern registry
   - Parameter integrity: в†' see AP-007 in anti-pattern registry (category 16)
   - Magic constants: detect same numeric (not 0/1/-1) or string literal appearing 2+ times in module (критерии — category 4, Duplicated magic constant)
   - Mixed responsibilities: detect procedures >40 lines combining 3+ concerns (rights, transaction, business logic, persistence, logging, UI)

4.5. Попытка/Исключение audit — see Phase 2.5 (dedicated pass).
     Do NOT audit Попытка blocks inline here; Phase 2.5 handles this systematically.

4.6. API existence (from orchestrator):
   If orchestrator passed API_VERIFIED / UNCHECKED_API items:
   - VERIFIED: no action needed
   - UNCHECKED_API: note in report as INFO ("method not verified, external dependency")
   If orchestrator did NOT pass API check results: skip (backward compatibility)

4.7. Contract Provenance Audit — see Phase 2.5 (dedicated pass).
     Do NOT audit defensive checks inline here; Phase 2.5 handles this systematically.

5. Extension annotations:
   - Detect &Перед/&После applied to a function (not procedure) — AP-022 CRITICAL. See anti-pattern registry
   - Detect &Вместо where &Перед/&После is sufficient
   - Detect &ИзменениеИКонтроль where code outside directives differs from base: variable renaming, formatting changes, refactoring outside #Вставка/#КонецВставки, adding/removing #Область in base (typed) code, NEW CODE added outside directive blocks
   - Detect business logic directly in #Вставка block
   - Detect AP-046: hook procedure starts with early Возврат under non-structural guard (feature-flag, business filter); body ≥ 5 lines or ≥ 2 distinct concerns. Exclude AP-021 (structural fail-fast). For &Вместо: early Возврат without subsequent ПродолжитьВызов = CRITICAL.
   - Detect AP-047: code claims to preserve base/BSP/platform/common-module behavior but implements a local equivalent instead of delegating to the owner (`ПродолжитьВызов(...)`, штатный API, БСП-обёртка, общий модуль). Treat as AUTHORITY_MISPLACEMENT when found in Phase 0.
   - &ИзменениеИКонтроль VERIFICATION: for each method with this annotation, load base from cf/ path
     (replace cfe/<ExtName>/ with cf/), extract code outside directive blocks, diff against base.
     Any diff (added/deleted/modified lines outside directives) = CRITICAL (prerelease) / HIGH (normal).
     Base file not found = NEEDS_MANUAL_REVIEW.

6. Module structure:
   - Check presence of #Область markup (flag as MEDIUM only if module > 100 lines; otherwise LOW)
   - Check order: ПрограммныйИнтерфейс → СлужебныйПрограммныйИнтерфейс → СлужебныеПроцедурыИФункции
   - Detect duplicate #Область/#КонецОбласти directives
   - Detect Export methods in #Область СлужебныеПроцедурыИФункции
   - Detect Export procedures/functions in form module (Forms/*/Module.bsl): any Процедура/Функция with Экспорт keyword.
     Exception: Подключаемый_* prefix (BSP attachable commands), callback exports for ОписаниеОповещения(..., ЭтотОбъект).
     Each non-excepted export = AP-033 HIGH. See anti-pattern registry.
   - Verify module header comment matches actual module name
   - Check module header comment

7. Method documentation:
   - Detect export methods without header comment
   - Detect event handlers without description
   - Validate header format (Параметры / Возвращаемое значение / Пример)

8. Extension naming:
   - Detect intercepted methods (&Вместо/&Перед/&После) without extension prefix
   - Detect own methods (non-intercept) incorrectly using extension prefix
   - Detect export own methods with non-descriptive names
   - Detect inconsistent prefix usage: exports with and without extension prefix in same module

9. Code cleanliness:
   - Detect changelog markers (// +++ Author, // ---, date-author comments)
   - Detect design/process artifact references in comments: short-form (D11, F5, Design В§3),
     natural-language (По design Decision N, fix-signing-result), process terms, task numbers
   - Meta-naming (AP-031): identifiers reflecting task/orchestration language or failing domain-clarity test; see full card in anti-pattern registry; export = HIGH, local/internal = MEDIUM; Remediation: propose domain synonym in finding
   - Dead code — see category 15 (Obsolete and Unused Code)
   - Detect logic duplication between modules
   - Detect commented-out code without explanation

10. Specific 1C patterns: в†' see AP-001..AP-050 in anti-pattern registry (category 16)
  AP-033: Export procedure/function in form module (Form-as-Service) — HIGH
    Remain inline:
    - Ternary operator ?() — MEDIUM
    - Excessive info logging inside loop or 3+ info-level calls — LOW

10.5. Integration exchange:
   - If integration exchange trigger matched, read `.cursor/docs/standard/std-12-integration-exchange.md`.
   - Detect AP-048: manual GUID/reference serialization in JSON/exchange; prefer `XMLСтрока(<Ссылка>)`.
   - Detect AP-049: numeric string through `Строка()` + whitespace cleanup; prefer `Формат(<Число>, "ЧГ=0")`.
   - Detect AP-050 only semantically for blocking messages near `Отказ = Истина`, `ВызватьИсключение`, `СообщитьПользователю`, `ПоказатьПредупреждение`. Do NOT flag neutral informational messages, administrator diagnostics, or strings where the form context already shows the field/action.

11. Band-aid detection: в†' see AP-016 in anti-pattern registry (category 16)
    Remain inline:
    - Design-prescribed anti-pattern (tag: design-prescribed)

12. Release readiness (prerelease only):
    - Typos and mixed Cyrillic/Latin in user-facing strings
    - Stub code — see category 4 (always checked)
    - Попытка/Исключение without logging — see category 10 (always-checked); do NOT duplicate

13. Transactions and locking:
    AP-015: Transaction without safety pattern — CRITICAL (see anti-pattern registry)
    AP-023: User interaction (ПоказатьВопрос, Предупреждение, Сообщить) inside transaction — HIGH. See anti-pattern registry
    Remain inline:
    - Read-then-write without БлокировкаДанных in concurrent scenario — HIGH
    - Nested НачатьТранзакцию() without justification — MEDIUM

14. Resource leaks:
    - COMОбъект (Новый COMОбъект()) without Попытка/Исключение ensuring release
    - HTTPСоединение/FTPСоединение not wrapped in Попытка for error handling
    - File reader/writer opened without close in error path
    - Temporary file created without cleanup in error path

15. Obsolete and unused code:
    - For each Процедура/Функция: Grep by name across all .bsl in extension directory
      to verify at least one call exists. Skip: event handlers, BSP commands, callbacks.
    - Unused non-export в†' MEDIUM; unused export в†' HIGH
    - Comment "Устарела:" / "Deprecated" or #Область УстаревшиеПроцедурыИФункции → MEDIUM/LOW
    - Obsolete procedure still called from non-obsolete code в†' HIGH
    - Unused parameter (never referenced in body) в†' LOW

16. Anti-pattern registry (reviewer-only, NOT loaded for writer):
    - Read `.cursor/rules/bsl-antipatterns.mdc` (index with AP-NNN IDs and detection rules).
    - For each AP-NNN: check reviewed code against detection rule in the index.
    - If detection rule matches: Read full card from `.cursor/docs/antipatterns/bsl-antipatterns.md`
      for examples and fix guidance.
    - Report finding with AP-NNN ID for traceability.
    - Anti-patterns are NOT auto-loaded for writer to avoid misinterpretation
      of BAD/GOOD examples as coding instructions.
```

### Phase 2.5: Попытка & Contract Audit (MANDATORY, выполнять ДО Phase 3)

Выделенный проход **только** для блоков Попытка/Исключение и оборонительных проверок (Свойство/ТипЗнч к внешним данным). Консолидирует логику из шагов 4.5 и 4.7. К Phase 3 не переходить, пока Phase 2.5 не завершена.

**SKEPTIC'S STANCE (принцип скептика):** Каждая защитная проверка (guard) виновна, пока не доказана обратная. Наличие guard в коде — НЕ доказательство того, что guard нужен. Контракт источника определять ТОЛЬКО из внешних источников (Form.xml, метаданные, текст запроса, документация, Resolved Contracts, код вызываемой функции). Если единственное основание считать контракт нефиксированным — сам guard → Contract verified? = **needs-verification** (не OK, не optional).

```yaml
A. Enumerate:
   - Найти ВСЕ блоки Попытка/Исключение в проверяемых файлах.
   - Составить нумерованный список: #, Procedure, approx. line (или диапазон).

B. Audit each block (для КАЖДОГО блока из списка A — одна строка в таблице отчёта):
   - Procedure, Line(s)
   - Operations inside: перечислить КАЖДУЮ операцию внутри Попытка (вызов, присваивание, обращение к полю).
   - Throwability analysis (для каждой операции из Operations inside):
     Для КАЖДОЙ операции ответить: «Может ли бросить исключение? Если да — почему?»
     Классификация причин:
       (a) external-nondeterminism — сеть, ФС, COM, конкурентный доступ
       (b) contract-mismatch — обращение к полю/свойству данных в памяти,
           которое может отсутствовать при несовпадении контракта
       (c) never-throws — присваивание, сравнение, арифметика, вызов
           без внешних эффектов
     НайтиПоРеквизиту: не бросает исключение при ненайденном значении
     (возвращает пустую ссылку) → (c).
     Обращение к полю объекта в памяти (Объект.Поле) →
       если контракт гарантирован: (c);
       если контракт неизвестен/частичен: (b).
   - Root cause of Попытка (вывести из Throwability analysis):
     Если есть хотя бы одна (a) → root cause = external-nondeterminism
     Если все throwable = (b), нет (a) → root cause = contract-uncertainty
     Если только (c) → AP-008 (все операции детерминированы)
     При mixed (a)+(b) → отметить обе причины; рекомендовать:
       вынести contract-доступ за Попытку (проверки свойств/типа до Попытки),
       оставить Попытку только для (a)-операций.
   - External factor? yes — какой именно (сеть/ФС/COM/конкурентный доступ/временное хранилище) / no.
     "Данные из API уже в памяти" — НЕ внешний фактор. Свойство(), ТипЗнч(), присваивание, сравнение — детерминированные операции.
   - RootCause (для таблицы): external | contract-uncertainty | deterministic | mixed(ext+contract).
   - Guard before same value? Есть ли непосредственно перед этой Попытка guard (Если ... Возврат/Продолжить), валидирующий то же значение, что используется внутри? yes/no.
   - Logging in Исключение? ЗаписьЖурналаРегистрации или re-raise в блоке Исключение? yes/no.
   - Fallback = success for caller? Возврат/присвоение в Исключение неотличимы от успеха для вызывающего? yes/no.
   - User feedback on failure? При Продолжить/тихом Возврат есть ли СообщитьПользователю или ВызватьИсключение? yes/no (исключение: явно документированный тихий пропуск в design/ТЗ).
   - Persistent side effects? Операция внутри Попытка записывает в БД (`Записать`, `ЗафиксироватьТранзакцию`, `НачатьТранзакцию` в связке с записью) или вызывает процедуру/функцию, которая пишет (1 уровень callee в том же репозитории)? yes — какие / no.
   - Re-raise in Исключение? Есть `ВызватьИсключение` в блоке Исключение (в т.ч. без аргумента)? yes/no.
   - Downstream dependency? Код после КонецПопытки (или после цикла, или caller после вызова процедуры) зависит от успешности этой записи? yes (описать) / no / uncertain.
   - Verdict: список ВСЕХ сработавших AP для этого блока: OK | AP-008 | AP-009 | AP-010 | AP-027 | AP-029 | AP-030 | AP-032 | contract-compensating-try | redundant layering.
   Критическое правило: нахождение одного AP НЕ закрывает проверку остальных критериев. Verdict = все применимые (например: AP-008, AP-010).
   RootCause = contract-uncertainty → Verdict включает contract-compensating-try. Ремедиация для contract-compensating-try: «Заменить Попытку проверкой контракта (тип/свойства) до обращения к полям. Если контракт невозможно подтвердить по базовому модулю — проверить ТипЗнч/Свойство перед доступом к вложенным полям.»
   **«Проверяй, а не лови» (Check, don't catch):** если Попытка защищает от несовпадения контракта (contract-uncertainty), правильный фикс — проверить контракт (тип, свойства) до обращения и убрать Попытку, а не логировать и продолжать. Логирование прячет ошибку: конкретная информация доступна только в ЖР, недоступна пользователю и поддержке в момент инцидента.
   MANDATORY Verdict derivation (выполнять ДЛЯ КАЖДОГО блока):
     Если RootCause = contract-uncertainty или mixed(ext+contract):
       → Verdict ОБЯЗАН включать contract-compensating-try.
       в†' Severity: HIGH, Action: MUST_FIX.
       → Log=yes И UserFeedback=yes НЕ отменяют contract-compensating-try.
         Логирование и сообщение не устраняют причину (contract-uncertainty).
       → Другие AP проверяются ДОПОЛНИТЕЛЬНО, но Verdict НЕ может быть OK.
   MANDATORY Verdict derivation — persistent state (выполнять ДЛЯ КАЖДОГО блока):
     Если Persistent side effects = yes И Re-raise in Исключение = no:
       Проверить Downstream dependency:
       → yes или uncertain → Verdict ОБЯЗАН включать AP-032.
       в†' Severity: CRITICAL, Action: MUST_FIX.
       → UserFeedback=yes НЕ отменяет AP-032 (сообщение пользователю не устраняет рассогласование данных в БД).
       → Log=yes НЕ отменяет AP-032.
       → Блок внутри цикла (Для Каждого / Для / Пока): Downstream dependency = yes по определению (гарантированный partial batch), если callee или тело Попытка пишет в БД.
       → Ремедиация: (a) убрать Попытку / re-raise — атомарность; (b) accumulate errors + signal caller + block downstream для сбойных элементов.
   Справочно: AP-008 = все операции детерминированы; AP-009 = fallback неотличим от успеха; AP-010 = нет лога; AP-027 = guard-then-catch; AP-029 = defense stack; AP-030 = скрытый частичный результат; AP-032 = персистентные side effects + подавление + downstream dependency; contract-compensating-try = Попытка компенсирует незнание контракта; redundant layering = INTEGRATION_CONTRACT_GATE (callee уже ловит).

   HIDDEN_PARTIAL_RESULT_GATE cross-check:
     Если в таблице Попытка Audit блок имеет RootCause = contract-uncertainty
     и Verdict включает contract-compensating-try:
       → HIDDEN_PARTIAL_RESULT_GATE для этого блока = FAIL.
       → В отчёте (секция gates) указать FAIL со ссылкой на contract-compensating-try finding.
       Причина: лог + сообщение маскируют ошибку контракта, а не устраняют скрытый результат.
     AP-032 cross-check:
       Если Persistent side effects = yes и Verdict включает AP-032:
       → HIDDEN_PARTIAL_RESULT_GATE для этого блока = FAIL.
       → Причина: данные в БД рассогласованы; сообщение пользователю / лог не устраняют зависимость downstream-кода от неконсистентного состояния.

C. Defensive checks audit (Contract Map–driven; НЕ синтаксический scan):
   **Драйвер:** обход артефакта Contract Map (Phase 0). Для КАЖДОГО источника из Contract Map с access != DIRECT (т.е. DEFENSIVE, GUARDED, EXPLORATORY), а также для источников, к полям которых обращаются ТОЛЬКО внутри Попытка — одна строка в Defensive Checks Table.
   **Для каждой строки:**
   1. Procedure, Line, Source (откуда данные), Field (имя поля/колонки/ключа).
   2. Идентифицировать конструкцию guard-а: Свойство(), ТипЗнч(), Колонки.Найти("Имя"), булев флаг (переменная/реквизит), ЕстьРеквизитИлиСвойствоОбъекта(), ЗначениеЗаполнено() как guard, условие — любая.
   3. Определить контракт источника:
      **ANTI-CIRCULAR GATE:** перед определением контракта — HALT. Проверить: единственное ли основание для «нефиксированный» — наличие самого guard? Если да → Contract verified? = needs-verification, Verdict ≠ OK.
      **Реквизит формы — обязательная верификация:** если источник = реквизит формы (таблица значений, колонка):
      1. Прочитать Form.xml того же объекта (Read).
      2. Если колонка/реквизит присутствует в Form.xml → контракт **фиксированный**, guard на наличие колонки = AP-004.
      3. Если колонки нет в Form.xml (или структура формируется динамически в коде) → контракт нефиксированный, guard может быть OK.
      4. Если Form.xml не удалось прочитать → Contract verified? = unverified (не «optional»).
      Без этой верификации нельзя ставить Contract = «optional column» или Verdict = OK для guard на колонку реквизита формы.
      - **Фиксированный:** метаданные объекта (ТЧ, реквизит), Form.xml реквизита формы (колонки таблицы значений формы заданы макетом того же объекта), текст запроса с явным списком полей, документированный параметр/возврат, Resolved Contracts с Contract: fixed. Для реквизита формы: если колонка/реквизит присутствует в Form.xml — контракт фиксирован; guard на наличие колонки (Колонки.Найти и т.п.) = избыточен.
      - **Нефиксированный:** внешний API, динамическая схема, Resolved Contracts с Contract: dynamic или unknown.
   4. Contract verified? (verified / phantom / unverified / **resolved-fixed** / **resolved-dynamic** — по Resolved Contracts при наличии).
   5. Verdict: фиксированный контракт + наличие guard → AP-004; нефиксированный + корректный guard → OK; нефиксированный + некорректный метод guard-а → AP-005 или иной по каталогу; phantom field + defense stack → AP-029 CRITICAL.
   **Resolved Contracts — артифакт ЗНИ.** Верифицированные explorer контракты в `reports/resolved-contract-*.md`. resolved-fixed + guard → AP-004; resolved-dynamic + минимальная проверка → OK.
   **Unverified-origin check:** для любого guard (не только Свойство/ТипЗнч): при Knowledge Assessment verdict = ABSENT или PARTIAL и отсутствии признаков установки контракта (комментарий, документация, Resolved Contracts) — Verdict = AP-004. Ремедиация: установить контракт (Investigation Request или анализ), затем решить — нужна ли проверка.

Completeness gate (Попытка): количество строк в таблице B (Попытка Audit) = количество блоков Попытка из шага A. Меньше — Phase 2.5 не завершена.

Completeness gate (Defensive Checks): количество строк в Defensive Checks Table = количество источников с access != DIRECT в Contract Map + источники, к полям которых обращаются только внутри Попытка (для них — строка с пометкой «access only inside Попытка, no explicit check», Contract = needs-resolution, Verdict = contract-compensating-try). Меньше — Phase 2.5 не завершена.

D. Investigation Request (резолв контрактов по запросу ревьювера):
   Если при заполнении таблиц B (Попытка Audit) или C (Defensive Checks) для какого-либо источника данных:
     - RootCause = contract-uncertainty (Попытка Audit), ИЛИ
     - Contract verified? = unverified при Knowledge Assessment verdict ABSENT/PARTIAL (Defensive Checks), ИЛИ
     - Есть Попытка или defensive checks, но контракт неизвестен и НЕ передан в Resolved Contracts
   ...то:
     1. В таблице C (Defensive Checks) для этого источника вписать Contract = needs-resolution (вместо unverified).
     2. В конце отчёта добавить секцию ## Investigation Request (формат ниже в Phase 4).

   Ревьювер НЕ приостанавливает отчёт. Он выдаёт полный отчёт (с findings: contract-compensating-try и т.д.) ПЛЮС секцию Investigation Request.
   Оркестратор парсит Investigation Request и решает, запускать ли explorer (шаг 3.5 review/SKILL.md).

   При повторном вызове с Resolved Contracts:
     - Обновить таблицы B и C: для каждого resolved источника — Contract verified? = resolved-fixed или resolved-dynamic.
     - resolved-fixed + defensive check в†' AP-004.
     - resolved-fixed + contract-compensating-try → заменить verdict на «AP-004, убрать Попытку».
     - resolved-dynamic + минимальная проверка → OK.
     - Пересмотреть findings, затронутые резолвом.
     - Секцию Investigation Request НЕ включать (контракты уже resolved).

   **Fallback при отсутствии блока в промпте.** Если оркестратор не передал блок ## Resolved Contracts, но ревью выполняется в контексте change — проверить Glob `reports/resolved-contract-*.md` в директории change. Если файл найден и scope совпадает — прочитать и использовать для Phase 2.5.
```

### Phase 3: Context Analysis
```yaml
1. Get similar code:
   similar = user-PROJECT-codemetadata (project-specific MCP)-codesearch(function_name)
   compare_implementations()

2. Check metadata:
   metadata = user-PROJECT-graph (project-specific MCP)-search_metadata(object_name)
   validate_dependencies()

3. Load past reviews:
   context = user-rlm-toolkit-rlm_route_context("code review " + module_name)
   apply_lessons_learned()
```

### Phase 4: Report Generation

Отчёт состоит из двух основных секций. Phase 0 findings идут первыми (корневые проблемы); Standards findings — дополнительные.

```yaml
1. Reasoning Analysis (Phase 0):
   - Artifacts: Intent Map, Contract Map, Knowledge Assessment (включить в отчёт)
   - Findings: DISPROPORTIONATE_COMPLEXITY, CONTRACT_INCONSISTENCY, CONTRACT_INFERENCE, KNOWLEDGE_DEFICIT, CLARITY_DEFICIT — с Intent, Expected, Actual, Root cause, Counterfactual, Remediation, Supporting

2. Standards & Patterns (Phase 1-2):
   - Findings (AP-NNN и прочие) — исключая те, что уже покрыты Phase 0 замечанием в том же месте (указаны как Supporting)

2.5. Попытка & Contract Audit (Phase 2.5):
   - Audit Table (every Попытка block with verdict)
   - Defensive Checks Table (every non-DIRECT source from Contract Map)
   - Findings from Phase 2.5 that are NOT already covered as Phase 0 Supporting

3. Summary:
   - Status: PASS | FAIL | NEEDS_WORK
   - Phase 0: N findings (РїРѕ severity)
   - Linter Signals (Phase 1b): K confirmed MUST_FIX, D dismissed, U unavailable
   - Naming Signals (Phase 1c): K confirmed, D dismissed, S skipped
   - Standards: M findings (РїРѕ severity)
   - Overall: итоговая формулировка
   - PASS запрещён при unresolved MUST_FIX из Phase 1b (in-scope linter warnings без fix/dismiss) или Phase 1c (Naming evidence match без AP-031 finding/dismiss)
```

Required Improvements (вместо секции "Рекомендации"):
Все пункты, ранее попадавшие в "Рекомендации", ДОЛЖНЫ быть оформлены как findings с severity (MEDIUM или LOW). Отдельной секции "Рекомендации" в отчёте НЕТ.

## REVIEW CATEGORIES

### Phase 0 (Reasoning) finding types

Типы замечаний, порождаемые Phase 0 (вне таксономии AP-NNN). Указывают на класс проблемы с логикой, а не на конкретный паттерн.

| Тип | Описание | Default severity |
|-----|----------|------------------|
| DISPROPORTIONATE_COMPLEXITY | Сложность реализации блока многократно превышает ожидаемую для его намерения | HIGH |
| CONTRACT_INCONSISTENCY | Один источник данных — часть полей напрямую, часть защитно/исследовательски | HIGH |
| CONTRACT_INFERENCE | Одно семантическое значение получается перебором нескольких альтернативных путей | HIGH |
| KNOWLEDGE_DEFICIT | Код компенсирует незнание контракта/домена вместо того, чтобы его выяснить | HIGH |
| CLARITY_DEFICIT | Намерение блока невозможно определить из кода без внешнего контекста | MEDIUM |
| AUTHORITY_MISPLACEMENT | Код берёт ответственность за поведение, владелец которого — другой слой (база, БСП, платформа, общий модуль, внешний контракт). Локальная реализация подменяет делегирование владельцу. | HIGH |

Формат замечания Phase 0: Procedure (имя процедуры/функции), Anchor (1–2 уникальные строки кода для Grep-поиска после правок), Intent, Expected, Actual, Root cause, Counterfactual, Remediation, Action (MUST_FIX | VERIFIED_OK | OPTIONAL), Supporting (AP-NNN при совпадении).

### Critical (блокирует коммит)
```yaml
- Syntax errors
- SQL injection vulnerabilities
- Data corruption risks
- Security breaches
- Performance killers (>10s operations)
- БСП violations (breaking changes)
- &Перед/&После applied to a function instead of a procedure
- ТекущаяДата() instead of ТекущаяДатаСеанса()
- НачатьТранзакцию() without matching ЗафиксироватьТранзакцию()/ОтменитьТранзакцию() in same scope
- Missing ОтменитьТранзакцию() in Исключение block of transactional Попытка
- Попытка wrapping deterministic operation (no external factor — rule 20)
- Defense stack with phantom field (AP-029 + unverified field name)
- AP-046 subcase: hook-scope early return under &Вместо without ПродолжитьВызов — silent override of base implementation
```

### High (исправить до завершения задачи)
```yaml
- Phase 0: DISPROPORTIONATE_COMPLEXITY, CONTRACT_INCONSISTENCY, CONTRACT_INFERENCE, KNOWLEDGE_DEFICIT
- Phase 0: AUTHORITY_MISPLACEMENT (локальная реализация поведения, у которого есть владелец)
- Logic errors
- Missing error handling
- Silent skip on structural check failure (Продолжить / silent Возврат / empty branch on type/property/size mismatch instead of ВызватьИсключение; business filtering is not a violation)
- Redundant property/attribute check on fixed-contract source (Свойство/ЕстьРеквизит on own tabular section field, explicit query column — field is guaranteed by metadata; also wrong method: Свойство() on non-Structure type)
- ТипЗнч() on fixed-contract return value (function/documentation guarantees type)
- ЗначениеЗаполнено() on field guaranteed by contract/metadata (as guard, not business check)
- "Defensive cake" pattern (stacked checks on same value where one is subsumed by another — any contract type, fixed or dynamic)
- N+1 query problems
- Missing indexes
- Insufficient access control
- Code duplication (>50 lines)
- Cyclomatic complexity >15
- &Вместо used where &Перед/&После is sufficient
- &ИзменениеИКонтроль: code outside #Вставка/#Удаление differs from base (variable rename, formatting, refactoring outside blocks, adding/removing #Область in typed code) — in prerelease: CRITICAL
- &ИзменениеИКонтроль used where &Перед/&После is sufficient
- Intercepted method (&Вместо/&Перед/&После) without extension prefix
- Сообщить() instead of ОбщегоНазначения.СообщитьПользователю()
- Оповестить()/ОповеститьОбИзменении() in server context (client-only methods)
- Свойство() on fixed-contract source (tabular section, query result)
- Band-aid fix detected (defensive check without root cause, try/except suppression, skip-flag, defensive cake)
- Попытка without logging (exception not re-raised) — traceless suppression (rule 20)
- Попытка with silent degradation fallback (rule 20)
- AP-027: Guard-then-catch (Попытка immediately after guard validating same value)
- AP-028: Check-after-establish (Свойство/ЕстьРеквизит/ЗначениеЗаполнено after type/structure established in code flow)
- Defense stack: Попытка wrapping only Свойство()/ТипЗнч() with no throwable operation (AP-029)
- Phantom field: Свойство("FieldName") where FieldName not found in existing codebase for same source
- НачатьТранзакцию() without Попытка/Исключение wrapping the transactional block
- User interaction (ПоказатьВопрос, Предупреждение, Сообщить) inside transaction
- Read-then-write without БлокировкаДанных in concurrent scenario
- COMОбъект created without Попытка/Исключение ensuring release
- Export procedure/function in form module (AP-033) — form-as-service pattern; exception: Подключаемый_* (BSP), ОписаниеОповещения callbacks
- Unused export procedure/function (no callers in extension scope) — category 15
- Obsolete procedure still called from non-obsolete code — category 15
- Parameter overwrite: parameter reassigned inside body, not documented as output — category 4 (rule 21)
- AP-046: Hook-scope early return suppressing entire body of an intercepted extension procedure — risks silent disabling of future composing modifications
```

### Medium (исправить в текущей итерации)
```yaml
- Phase 0: CLARITY_DEFICIT (намерение блока неочевидно из кода)
- AP-031: мета-имена из постановки/оркестрации (доменный тест + маркеры в карточке AP-031); экспортные процедуры/функции — HIGH; в finding обязательно предложить доменный синоним
- Naming convention violations
- Missing documentation
- Suboptimal algorithms
- Code smells
- Minor БСП deviations
- Testability issues
- Missing #Область markup in module (module > 100 lines; otherwise LOW)
- Wrong order of #Область regions
- Export method without header comment (purpose, parameters, return value)
- Business logic directly in #Вставка block instead of separate procedure
- Own non-intercept method with extension prefix
- Export own method without descriptive unique name
- Dead code — see category 15 (Obsolete and Unused Code)
- Unused non-export procedure/function (category 15)
- Procedure/function marked "Устарела:" / "Deprecated" (category 15)
- Logic duplication between modules
- Commented-out code without explanation
- User-facing string literals without НСтр("ru = '...'")
- Ternary operator ?() usage (style preference)
- Probable band-aid (TODO workaround, duplicated logic with variation)
- Collection mutation on parameter without out contract — category 4 (rule 21)
- Duplicated magic constant (same literal 2+ times in module) — category 4
- Mixed responsibilities (procedure >40 lines, 3+ concerns) — category 4
- Inconsistent prefix usage (exports with and without prefix in same module) — category 8
- AP-031: Domain naming test failure (meta-names, implementation-role names) — MEDIUM (export: HIGH). See anti-pattern registry
```

### Low (исправить, минимальный приоритет)
```yaml
- Code formatting
- Comment style
- Variable naming (casing, prefix style only; domain-clarity failures are MEDIUM via AP-031)
- Minor optimizations
- Refactoring opportunities
- Missing module header comment
- Event handler without description
- Header format not matching BSP template
- Excessive info logging (ЗаписьЖурналаРегистрации Информация/Примечание in loop or 3+ calls) — category 10
```
(Changelog markers and design refs in comments are now MEDIUM under Code Cleanliness / release-hygiene and escalate to HIGH in prerelease.)

## PRE-RELEASE SEVERITY ESCALATION

When reviewer is called with context `mode=prerelease`, tag each finding with **kind** and apply escalation only to functional findings.

### Kind (every finding)

```yaml
kind:
  functional — affects behavior, data, security, reliability (bugs, ТекущаяДата on server, injection, silent skips, band-aids)
  style — affects readability, standards, structure only (?(), #Область missing, naming prefix, header format)
  release-hygiene — process metadata that must not ship to production (changelog markers, work instructions, commented-out old code, design refs)
```

**Escalation (LOW→MEDIUM, MEDIUM→HIGH) applies to kind=functional, release-hygiene, and style.** Code is delivered to the customer — style matters in prerelease. Tag style findings with `[style]` in the report. For kind=release-hygiene, see also Pre-release escalation (release-hygiene) below.

### Kind by category (examples)

```yaml
kind=functional:
  - ТекущаяДата() on server (use ТекущаяДатаСеанса())
  - Сообщить() instead of ОбщегоНазначения.СообщитьПользователю()
  - Silent skip on structural check failure, band-aid fixes
  - Export procedure/function in form module (AP-033, form boundary violation)
  - &ИзменениеИКонтроль: code outside #Вставка/#Удаление modified (breaks extension applicability)
  - Security, performance bugs, logic errors
  - ЭтаФорма instead of ЭтотОбъект
  - Duplicate #Область (structural breakage)
  - Typos in user-facing strings (mixed encoding, spelling)
  - Stub/placeholder code in production
  - Попытка/Исключение without logging (exception silently swallowed) — traceless suppression
  - Попытка/Исключение wrapping fixed-contract access (contract masking)
  - Попытка/Исключение wrapping deterministic operation (no external factor — rule 20)
  - Попытка/Исключение with silent degradation fallback (rule 20)
  - Parameter overwrite (parameter reassigned inside body, not documented as output — rule 21)

kind=style:
  - Ternary operator ?() (style preference, not functional defect)
  - Missing #Область structure in module
  - Own non-intercept method using extension prefix
  - Method name contradicts compilation directive
  - Export in private region (#Область СлужебныеПроцедурыИФункции)
  - Module header name mismatch
  - Missing module header, event handler without description, header format not matching BSP
  - Collection mutation on parameter without out contract (rule 21)
  - Duplicated magic constant (category 4)
  - Mixed responsibilities (procedure >40 lines, 3+ concerns)
  - Inconsistent prefix usage (exports with/without prefix in same module)
  - Excessive info logging (ЗаписьЖурналаРегистрации Информация in loop or 3+ calls)

kind=release-hygiene:
  - Changelog markers in comments (// +++/---, // НАЧАЛО/КОНЕЦ, // РГИТС, date-author in comments)
  - Commented-out old code with replacement markers
  - Work instructions in comments
  - Design/process artifact references in comments (short-form D11/F5, natural-language
    "По design Decision N (change-name)", process terms, kebab-case change names, task numbers)
  Not release-hygiene: #Вставка, #КонецВставки, #Удаление, #КонецУдаления — extension override directives, do not remove or flag.
  Project-level override: comments matching openspec/project.md «Whitelist предрелиза» patterns within the row's scope (glob) — NOT release-hygiene. Check project.md before flagging.

kind=functional (category 15 — unused/obsolete):
  - Unused export procedure/function (no callers in extension scope) — dead API surface
  - Obsolete procedure still called from non-obsolete code — caller must migrate

kind=style (category 15 — unused/obsolete):
  - Unused non-export procedure/function (no behavior impact, dead weight)
  - Obsolete markers present (comment "Устарела:", #Область УстаревшиеПроцедурыИФункции)
  - Unused parameter in procedure/function body
```

### Escalation rules

```yaml
Pre-release escalation (all kinds: functional, style, release-hygiene):
  LOW в†' MEDIUM:
    - All kinds escalated (code is delivered to the customer)

  MEDIUM в†' HIGH:
    - Export method without header (if functional impact, e.g. contract unclear)
    - Dead code, logic duplication (if kind=functional)
    - Business logic directly in #Вставка block
    - Export method in СлужебныеПроцедурыИФункции (contract violation)
    - Export procedure/function in form module (AP-033) — form-as-service before release
    - Unused export procedure/function (dead API surface before release) — category 15
    - Procedure marked "Устарела:" / "Deprecated" still present without documented plan — category 15

  HIGH в†' CRITICAL:
    - &ИзменениеИКонтроль: code outside #Вставка/#Удаление differs from base (variable rename, formatting, #Область in base code) — breaks extension applicability
    - Попытка/Исключение without logging (traceless suppression — rule 20)
    - Попытка/Исключение with silent degradation fallback (rule 20)

Note: Escalation is additive. All kinds (functional, style, release-hygiene) are escalated in prerelease — code is delivered to the customer. Tag style findings with [style].
```

### Pre-release escalation (release-hygiene)

```yaml
Pre-release escalation (release-hygiene):
  MEDIUM в†' HIGH:
    - All release-hygiene items (changelog markers in comments, work instructions, commented-out old code, design refs). Do not flag or remove #Вставка/#Удаление directives.

  Note: release-hygiene HIGH items appear in "fix before release" section
  Все замечания (в т.ч. style HIGH) обязательны к исправлению; severity задаёт приоритет.
```

**How to detect `mode=prerelease`**: The calling prompt explicitly passes `mode=prerelease` in context, or the review is triggered by `/release-review`. In release reports, always output `kind: functional`, `kind: style`, or `kind: release-hygiene` (and level) for each finding.

## STANDARDS REFERENCE

### БСП Naming Conventions
```yaml
Modules:
  - CommonModule: ОбщегоНазначения, ОбщегоНазначенияКлиент
  - ObjectModule: ДокументОбъект.<Name>
  - ManagerModule: ДокументМенеджер.<Name>

Functions/Procedures:
  - Export: ПолучитьДанныеКлиента()
  - Internal: ПолучитьДанныеКлиентаВнутренний()
  - Client: ПолучитьДанныеКлиентаНаКлиенте()
  - Server: ПолучитьДанныеКлиентаНаСервере()

Variables:
  - Parameters: ПараметрИмя
  - Local: ИмяПеременной
  - Module: МодульнаяПеременная
```

### Performance Patterns
```yaml
Anti-patterns:
  - Query in loop: Выборка.Следующий() with nested query
  - Missing index: Selection without WHERE on indexed field
  - Full table scan: Selection without filters
  - Excessive database calls: >10 per function

Best practices:
  - Batch operations: Process multiple records at once
  - Use indexes: Always filter on indexed fields
  - Cache data: Store frequently accessed data
  - Minimize round-trips: Combine queries when possible
```

### Security Patterns
```yaml
Vulnerabilities:
  - SQL injection: String concatenation in query
  - XSS: Unescaped output in forms
  - Access control: Missing RLS checks
  - Hardcoded secrets: Passwords in code

Best practices:
  - Parameterized queries: Use query parameters
  - Input validation: Sanitize all inputs
  - Access control: Check rights before operations
  - Secure storage: Use encrypted storage for secrets
```


