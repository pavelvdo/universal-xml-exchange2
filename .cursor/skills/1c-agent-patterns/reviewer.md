# Reviewer — шаблоны промптов

Шаблоны для делегирования `Task(subagent_type="onec-code-reviewer")`. Общие правила, выбор модели, обработка ошибок — `SKILL.md` (навигатор).

**Контракт v3:** reviewer агент v3.0 (`.cursor/agents/onec-code-reviewer.md`) сам читает AP-каталог `.cursor/rules/bsl-antipatterns.mdc`, применяет prerelease-эскалацию и risk-модель. Оркестратор передаёт **evidence-блоки** (linter signals, whitelist, prior history, architectural context, boundaries, resolved contracts), а не готовые findings. См. шаги 1.6–2.2 в `.cursor/skills/review/SKILL.md`.

---

## Reviewer (ревью кода)

```
Task(
  description="Ревью кода [feature]",
  prompt="Проверь качество кода для [feature].

         expected_reviewer_prompt_contract_version: 3

         Файлы: [список изменённых .bsl]
         Стандарты: .cursor/docs/1c-coding-standards.md
         Base-файл (для &ИзменениеИКонтроль): [путь к base в cf/ или 'не применимо']

         ## Linter Signals (evidence)
         [Блок из шага 1.8 review/SKILL.md — таблица ReadLints/bsl-language-server (все severity, включая warning); либо 'Linter unavailable: <reason>'. Reviewer Phase 1b (reviewer-checks.md § Phase 1b: BSL Linter Signals Gate): in-scope warning → MUST_FIX по умолчанию; отложить на prerelease запрещено.]

         ## Naming Signals (evidence)
         [Блок из шага 1.9 review/SKILL.md — таблица grep (clean / matches / skipped). Reviewer Phase 1c (reviewer-checks.md § Phase 1c: Naming Provenance Gate): match → AP-031 MUST_FIX по умолчанию; dismiss только metadata-name / false-positive / pre-existing-unchanged.]

         ## Whitelist & Mandatory Controls (from project.md)
         [Блок из шага 1.6.1 — две таблицы (Whitelist предрелиза, Обязательный контроль) для release-hygiene rules AP-040..AP-043.]

         ## Comment Markers Metadata (from proposal.md)
         [Если review_mode = change-scoped: извлечь `## Metadata (comment markers)` из proposal.md (developer, comment_suffix). BORDER-PAIR-001 по ФИО.]

         ## Mandatory Control Signals (evidence)
         [Опционально — из шага 1.6.2, если есть regex-нарушения. Таблица Rule ID / File:Line / Observed / Expected.]

         ## Prior Findings History
         [Опционально — из шага 2.1: глоб предыдущих review-<scope-slug>-*.md ≤ 90 дней. Таблица по отчётам. Reviewer для совпадающих findings ставит tag 'recurrent' и повышает риск.]

         ## Architectural Context
         [Опционально — из design.md / architecture-*.md активного change. Оценивать решения в коде на соответствие контексту.]

         [Если оркестратор передал блок ## Resolved Contracts — включить его сюда. Использовать для Phase 2.5 Defensive Checks Audit: resolved-fixed + guard → AP-004; resolved-dynamic + минимальная проверка → OK.]

         [Если оркестратор собрал diff-focused scope — вставить блок ## Review Boundaries целиком (см. `.cursor/skills/review/SKILL.md`, шаг 1.5). При полном ревью (focus=full для всех файлов батча) секцию ## Review Boundaries не вставлять. В смешанном батче — по подзаголовку ### Файл: <path> и Focus: full | diff-focused на каждый файл. Соблюдай Review Boundaries Protocol в onec-code-reviewer.md.]

         ## Reasoning focus (Phase 0 — выполнять ПЕРВЫМ)
         Перед проверкой стандартов и паттернов — понять код:
         1. Для каждого блока: что он делает? (Intent Map)
         2. Для каждого источника данных: как обращается к полям? (Contract Map)
         3. Согласованы ли обращения? Пропорциональна ли сложность?
         4. Автор знает контракт или компенсирует незнание?
         Артефакты Phase 0 включить в отчёт.
         Заполнить Evaluation Checklist (все 6 вопросов); пропуск вопроса 5 (Попытка as contract compensation) недопустим.

         ## Code Smells & Elegance (Эстетика и запахи кода)
         Оцени код на предмет 'дурного запаха' (code smells) и когнитивной перегрузки. Код должен быть не только рабочим, но и красивым, лаконичным и легко читаемым.
         Ищи следующие маркеры:
         - Arrow Code (Стрелочный код): вложенность условий Если и циклов более 3 уровней.
         - Complex Conditions: многоэтажные логические выражения Если А И (Б Или В) И НЕ Г. Их следует выносить в переменные с говорящими именами (Self-documenting code).
         - God Procedure: процедуры длиннее 50-70 строк, делающие всё подряд (нарушение Single Responsibility).
         - Inefficient Collections: поиск в цикле по Массиву вместо использования Соответствия/Структуры.
         - Magic Numbers/Strings: захардкоженные значения, смысл которых неочевиден.
         - Copy-Paste: дублирование логики с минимальными изменениями вместо выноса в общую функцию.

         ## Стандартный фокус
         Баги, БСП compliance, производительность, безопасность,
         аннотации расширений, структура модуля, документация методов.
         Если линтер выявил ошибки — включить их в категорию critical.
         Category 9 Code Cleanliness включая log-literal artefacts: имя события ЖР и текст сообщения/СтрШаблон не должны содержать маркеры постановки (S<N>, D<N>, kebab-case change, Decision); отладочный ЗаписьЖурналаРегистрации (Информация/Предупреждение/Примечание) без требования в tasks.md/design.md — удалить или оформить требование. Action: MUST_FIX.
         Пустые процедуры/функции/обработчики (empty-unused / empty-body / empty-handler) в границах ревью — Action: MUST_FIX (обработчики без привязки к элементу — инструкция правки Form.xml вручную).
         Если в промпте есть блок ## Mechanical findings (orchestrator) — не дублировать эти строки в findings; при необходимости дополнить контекстом (процедура, вызов).
         Если передан блок ## Comment Markers Metadata — BORDER-PAIR-001:
         - **cfe:** на каждый `// +++` с `developer` из metadata — парный `// --- {developer}` (legacy `[ID#…]`/`[б/н#…]` в open допустим).
         - **cf:** open `// {cfMarkerPrefix}… +++` (cfMarkerPrefix из project.md) — парный `// {cfMarkerPrefix без :} ---`.
         Нет пары — HIGH [release-hygiene]. Чужие ФИО/префиксы не проверять.
         Release-hygiene: AP-040..AP-045 включая AP-053 (whitelist exempt removal only).

         ## Проверка соблюдения gates (HALT-compliance)
         Проверить, что writer следовал gates из промпта.
         1. DATA_CONTRACT_GATE: если добавлены ТипЗнч/Свойство/ЗначениеЗаполнено —
            обоснована ли каждая проверка контрактом? Defensive cake (стек 2+ проверок
            на одном значении) — при ЛЮБОМ контракте? Свойство() вызван на
            не-Структуре? Проверка добавлена вместо выяснения типа?
            Нарушение → CRITICAL: unverified defensive check / wrong method.
         2. INTEGRATION_CONTRACT_GATE: если добавлена обёрточная логика вокруг
            вызова — проверить контракт вызываемой функции. Функция уже обрабатывает
            случай? Нарушение → HIGH: duplicated logic.
         3. EXTENSION_GUARD: если файл содержит &ИзменениеИКонтроль —
            код вне #Вставка/#Удаление побитово совпадает с base?
            Расхождение → CRITICAL (prerelease) / HIGH.
         4. CONTEXT_SAFETY: клиентские Перем базовой формы не используются
            в серверном контексте расширения (&После/&Перед на сервере)?
            Нарушение → CRITICAL.
         5. EXISTING_MECHANISM_GATE: не создаёт ли решение параллельный
            механизм при наличии штатного? Если design не содержит анализ
            Existing Mechanisms при интеграции — HIGH: missing mechanism analysis.
         6. CONDITIONAL_ACTION_GATE: если перед единственным оставшимся
            действием стоит Если <обратное условие> Тогда Возврат —
            это инвертированный ранний выход. Действие должно быть
            обёрнуто в Если <прямое условие> Тогда ... КонецЕсли.
            Guard clause в начале процедуры (перед основной логикой
            из нескольких действий) — допустим.
            Нарушение → HIGH: inverted early return instead of conditional action.
         7. ORCHESTRATOR_IMPLEMENTATION_GATE: если в коде видны паттерны,
            характерные для копирования из промпта (Попытка без внешнего
            фактора, необоснованные guard-ы, пустые Исключение) —
            проверить, не выполнил ли writer указание оркестратора
            вместо применения стандартов. Нарушение → HIGH.
         8. HIDDEN_PARTIAL_RESULT_GATE: Попытка с Продолжить/тихим fallback —
            пользователь получает обратную связь об ошибке?
            Если нет СообщитьПользователю / индикации — HIGH: hidden partial result (AP-030).
            Если операция внутри Попытка имеет персистентные побочные эффекты
            (запись в БД через callee или напрямую) и нет re-raise,
            а downstream-код зависит от успешности записи —
            CRITICAL: inconsistent persistent state (AP-032).
            AP-032 не снимается добавлением СообщитьПользователю.
            Исключение: логика приложения явно предусматривает тихий пропуск
            (задокументировано в design/ТЗ).
            Cross-check: если в Phase 2.5 блок имеет RootCause = contract-uncertainty
            и Verdict включает contract-compensating-try — gate для этого блока = FAIL
            (лог + сообщение не устраняют причину).
         Phase 2.5: при RootCause = contract-uncertainty Verdict ОБЯЗАН включать
         contract-compensating-try; Log=yes и UserFeedback=yes это не отменяют.
         9. ANNOTATION_CHOICE_GATE: если использован &ИзменениеИКонтроль, проверь, можно ли решить задачу через &После/&Перед/&Вместо. Если да -> HIGH: suboptimal extension annotation.
         10. QUERY_PATCH_GATE: если текст запроса модифицируется через СтрЗаменить или собирается через конкатенацию с Символы.ПС -> CRITICAL: brittle query patching / bad query formatting.

         ## Зрелость интеграции
         (a) Есть ли в вызывающем коде логика, которую вызываемая функция
         уже выполняет внутри?
         (b) Оправдан ли выбор точки реализации — могла ли задача быть
         решена на другом уровне проще?
         (c) Следует ли решение паттернам проекта? Если нет —
         пометить как HIGH: assumption-driven design.
         (d) Не создан ли параллельный механизм при наличии штатного
         в базе? Если design не содержит анализ Existing Mechanisms
         при интеграции — HIGH: missing mechanism analysis.

         ## Формат замечаний
         Строго по REPORT FORMAT v3 из onec-code-reviewer.md (системный промпт агента):
         обязательные поля Procedure, Anchor, Action (MUST_FIX / REFACTOR / VERIFIED_OK / OPTIONAL),
         Type (CODE / ARCHITECTURE — ARCHITECTURE при изменении метаданных, контрактов API,
         структуры хранения, прав/RLS или смене точки расширения).

         ### Оценка эстетики кода (Elegance Score)
         В конце отчета выведи блок:
         - Читаемость: [оценка 1-5] ([краткий комментарий])
         - Когнитивная нагрузка: [Низкая/Средняя/Высокая] ([краткий комментарий])
         - Вердикт: [краткий вывод о необходимости рефакторинга]",
  subagent_type="onec-code-reviewer"
)
```

---

## Reviewer (предрелиз) — `/release-review`

Использовать при `release_mode = true` (шаг 3 review/SKILL.md). Отличия от обычного ревью: **`mode=prerelease`**, Category 12, эскалация severity, release-hygiene.

```
Task(
  description="Предрелизное ревью [extension/change]",
  prompt="Проверь качество кода для предрелиза [feature/extension].

         expected_reviewer_prompt_contract_version: 3
         mode=prerelease

         Файлы: [список .bsl батча]
         Стандарты: .cursor/docs/1c-coding-standards.md
         Category 12: .cursor/docs/standard/reviewer-checks.md §12 Release Readiness
         Base-файл (для &ИзменениеИКонтроль): [путь к base в cf/ или 'не применимо']

         ## Linter Signals (evidence)
         [Блок из шага 1.8 review/SKILL.md]

         ## Naming Signals (evidence)
         [Блок из шага 1.9 review/SKILL.md — clean / matches / skipped; Phase 1c → AP-031 MUST_FIX по умолчанию]

         ## Whitelist & Mandatory Controls (from project.md)
         [Блок из шага 1.6.1]

         ## Comment Markers Metadata (from proposal.md)
         [При change-scoped: developer, comment_suffix; BORDER-PAIR-001]

         ## Mandatory Control Signals (evidence)
         [Опционально — шаг 1.6.2]

         ## Prior Findings History
         [Опционально — шаг 2.1]

         ## Architectural Context
         [Опционально — design.md / architecture-*.md]

         ## Reference Files (read-only)
         [При change-scoped: список reference_files — контекст только, findings запрещены]

         [## Review Boundaries — при diff-focused, как в шаблоне «Reviewer (ревью кода)»]

         РЕЖИМ: mode=prerelease. Применять Prerelease escalation из AP-каталога (HIGH→CRITICAL где указано).
         Release-hygiene: AP-040..AP-045 (whitelist exempt removal; AP-053 content), BORDER-PAIR-001 (cfe + cf dual-pattern), AP-042/043 по evidence-блокам.
         Category 12 Release Readiness — обязательно в prerelease.
         Язык комментариев (не JSDoc/маркеры): русский; англ. кроме TODO/FIXME — MEDIUM [style].
         AP-031 naming: доменный тест для идентификаторов; экспортные без домена — HIGH [style].

         [Далее — Reasoning focus, Code Smells, gates HALT-compliance, формат замечаний — как в «Reviewer (ревью кода)»;
          EXTENSION_GUARD: расхождение с base → CRITICAL (prerelease) / HIGH (normal) — здесь всегда prerelease → CRITICAL]

         ### Оценка эстетики кода (Elegance Score)
         [как в основном шаблоне]",
  subagent_type="onec-code-reviewer"
)
```

---

## Reviewer — bug fix (ревью исправления ошибки)

```
Task(
  description="Ревью фикса [описание бага]",
  prompt="Проверь качество исправления ошибки для [feature/модуль].

         Файлы: [список изменённых .bsl]
         Стандарты: .cursor/docs/1c-coding-standards.md
         Диагностики линтера: блок ## Linter Signals (evidence) — таблица ReadLints/bsl-language-server (включая warning); reviewer Phase 1b → in-scope MUST_FIX, без отложения на prerelease
         Naming Provenance: блок ## Naming Signals (evidence) — grep из шага 1.9; reviewer Phase 1c → AP-031 MUST_FIX по умолчанию
         Base-файл (для &ИзменениеИКонтроль): [путь к base в cf/ или 'не применимо']

         [Если оркестратор собрал diff-focused scope — вставить блок ## Review Boundaries (как в шаблоне «Reviewer (ревью кода)»). При focus=full для всех файлов — секцию не вставлять.]

         ## Architectural Context
         [Если оркестратор передал контекст из design.md и/или architecture-*.md — вставить его сюда. Оценивать решения в коде на соответствие этому контексту.]

         ## Root Cause Context
         Симптом: [что наблюдалось]
         Корневая причина: [из design.md или промпта]
         Подход к фиксу: [что должен был сделать writer]

         ## Bug fix фокус (помимо стандартного ревью)
         1. Фикс устраняет КОРНЕВУЮ ПРИЧИНУ (не симптом)?
         2. Нет ли добавленных защитных проверок без обоснования?
         3. Не появились ли анти-паттерны из verified-cause-gate.mdc?
         4. Если фикс = заплатка — пометить как HIGH: band-aid fix.
         5. Предотвратит ли фикс повторение проблемы?

         ## Code Smells & Elegance (Эстетика и запахи кода)
         Оцени код на предмет 'дурного запаха' (code smells) и когнитивной перегрузки. Код должен быть не только рабочим, но и красивым, лаконичным и легко читаемым.
         Ищи следующие маркеры:
         - Arrow Code (Стрелочный код): вложенность условий Если и циклов более 3 уровней.
         - Complex Conditions: многоэтажные логические выражения Если А И (Б Или В) И НЕ Г. Их следует выносить в переменные с говорящими именами (Self-documenting code).
         - God Procedure: процедуры длиннее 50-70 строк, делающие всё подряд (нарушение Single Responsibility).
         - Inefficient Collections: поиск в цикле по Массиву вместо использования Соответствия/Структуры.
         - Magic Numbers/Strings: захардкоженные значения, смысл которых неочевиден.
         - Copy-Paste: дублирование логики с минимальными изменениями вместо выноса в общую функцию.

         ## Стандартный фокус
         Баги, БСП compliance, производительность, безопасность,
         аннотации расширений, структура модуля, документация методов.
         Если линтер выявил ошибки — включить их в категорию critical.

         ## Проверка соблюдения gates (HALT-compliance)
         Проверить, что writer следовал gates из промпта:
         1. DATA_CONTRACT_GATE: если добавлены ТипЗнч/Свойство/ЗначениеЗаполнено —
            обоснована ли каждая проверка контрактом? Defensive cake (стек 2+ проверок
            на одном значении) — при ЛЮБОМ контракте? Свойство() вызван на
            не-Структуре? Проверка добавлена вместо выяснения типа?
            Нарушение → CRITICAL: unverified defensive check / wrong method.
         2. INTEGRATION_CONTRACT_GATE: если добавлена обёрточная логика вокруг
            вызова — проверить контракт вызываемой функции. Функция уже обрабатывает
            случай? Нарушение → HIGH: duplicated logic.
         3. EXTENSION_GUARD: если файл содержит &ИзменениеИКонтроль —
            код вне #Вставка/#Удаление побитово совпадает с base?
            Расхождение → CRITICAL (prerelease) / HIGH.
         4. CONTEXT_SAFETY: клиентские Перем базовой формы не используются
            в серверном контексте расширения (&После/&Перед на сервере)?
            Нарушение → CRITICAL.
         5. EXISTING_MECHANISM_GATE: не создаёт ли решение параллельный
            механизм при наличии штатного? Если design не содержит анализ
            Existing Mechanisms при интеграции — HIGH: missing mechanism analysis.
         6. CONDITIONAL_ACTION_GATE: если перед единственным оставшимся
            действием стоит Если <обратное условие> Тогда Возврат —
            это инвертированный ранний выход. Действие должно быть
            обёрнуто в Если <прямое условие> Тогда ... КонецЕсли.
            Guard clause в начале процедуры (перед основной логикой
            из нескольких действий) — допустим.
            Нарушение → HIGH: inverted early return instead of conditional action.
         7. ORCHESTRATOR_IMPLEMENTATION_GATE: если в коде видны паттерны,
            характерные для копирования из промпта (Попытка без внешнего
            фактора, необоснованные guard-ы, пустые Исключение) —
            проверить, не выполнил ли writer указание оркестратора
            вместо применения стандартов. Нарушение → HIGH.
         8. HIDDEN_PARTIAL_RESULT_GATE: Попытка с Продолжить/тихим fallback —
            пользователь получает обратную связь об ошибке?
            Если нет СообщитьПользователю / индикации — HIGH: hidden partial result (AP-030).
            Если операция внутри Попытка имеет персистентные побочные эффекты
            (запись в БД через callee или напрямую) и нет re-raise,
            а downstream-код зависит от успешности записи —
            CRITICAL: inconsistent persistent state (AP-032).
            AP-032 не снимается добавлением СообщитьПользователю.
            Исключение: логика приложения явно предусматривает тихий пропуск
            (задокументировано в design/ТЗ).
            Cross-check: если в Phase 2.5 блок имеет RootCause = contract-uncertainty
            и Verdict включает contract-compensating-try — gate для этого блока = FAIL
            (лог + сообщение не устраняют причину).
         Phase 2.5: при RootCause = contract-uncertainty Verdict ОБЯЗАН включать
         contract-compensating-try; Log=yes и UserFeedback=yes это не отменяют.
         9. ANNOTATION_CHOICE_GATE: если использован &ИзменениеИКонтроль, проверь, можно ли решить задачу через &После/&Перед/&Вместо. Если да -> HIGH: suboptimal extension annotation.
         10. QUERY_PATCH_GATE: если текст запроса модифицируется через СтрЗаменить или собирается через конкатенацию с Символы.ПС -> CRITICAL: brittle query patching / bad query formatting.

         ## Зрелость интеграции
         (a) Есть ли в вызывающем коде логика, которую вызываемая функция
         уже выполняет внутри?
         (b) Оправдан ли выбор точки реализации — могла ли задача быть
         решена на другом уровне проще?
         (c) Следует ли решение паттернам проекта? Если нет —
         пометить как HIGH: assumption-driven design.
         (d) Не создан ли параллельный механизм при наличии штатного
         в базе? Если design не содержит анализ Existing Mechanisms
         при интеграции — HIGH: missing mechanism analysis.

         ## Формат замечаний
         Строго по REPORT FORMAT v3 из onec-code-reviewer.md (системный промпт агента):
         обязательные поля Procedure, Anchor, Action (MUST_FIX / REFACTOR / VERIFIED_OK / OPTIONAL),
         Type (CODE / ARCHITECTURE — ARCHITECTURE при изменении метаданных, контрактов API,
         структуры хранения, прав/RLS или смене точки расширения).

         ### Оценка эстетики кода (Elegance Score)
         В конце отчета выведи блок:
         - Читаемость: [оценка 1-5] ([краткий комментарий])
         - Когнитивная нагрузка: [Низкая/Средняя/Высокая] ([краткий комментарий])
         - Вердикт: [краткий вывод о необходимости рефакторинга]",
  subagent_type="onec-code-reviewer"
)
```
