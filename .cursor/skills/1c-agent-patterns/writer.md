# Writer — шаблоны промптов

Шаблоны для делегирования `Task(subagent_type="onec-code-writer")`. Общие правила, выбор модели, обработка ошибок, Shared instruction blocks (DATA_CONTRACT_GATE, INTEGRATION_CONTRACT_GATE, EXISTING_MECHANISM_GATE, EXTENSION_GUARD, CONTEXT_SAFETY, CONDITIONAL_ACTION_GATE, ORCHESTRATOR_IMPLEMENTATION_GATE) — `SKILL.md` (навигатор), секция «Shared instruction blocks».

**Модель writer:** по таблице `.cursor/rules/model-selection.mdc` writer вызывается **без** параметра `model=` (inherit — модель родительского чата); во frontmatter агента — `model: inherit`. Явный `model=` — только по запросу пользователя. Детали — `tool-name-guard.mdc`.

---

## Writer (реализация задачи)

```
Task(
  description="Реализовать задачу N [feature]",
  prompt="## 1. ЗАДАЧА
         Реализуй задачу N для [feature].
         Задача: [описание и критерии из tasks.md]

         ## 2. КОНТЕКСТ
         Design: [путь к design.md]
         Файлы: [пути к .bsl файлам для правки]
         ## 3. ОГРАНИЧЕНИЯ
         Только правка существующих .bsl, без новых файлов и метаданных.
         При отсутствии нужного модуля/объекта — СТОП с указанием что добавить.

         ## 3a. COMMENT MARKERS
         Все новые вставки кода должны быть обёрнуты в маркеры:
         Open: [вставить open_marker]
         Close: [вставить close_marker]
         domain_label на русском, без process-меток (openspec/project.md § Канон domain_label). При mixed scope — шаблон по пути файла.
         (Правила размещения — см. твой системный промпт §3a).

         ## 4. INPUT CONTRACT
         [Вставить блок ## Project paths из project-paths.mdc]
         [Вставить блок ## BSL_LSP Status: connected|not_connected]

         ## 5. GATES
         [Вставить блок DATA_CONTRACT_GATE]
         [Вставить блок INTEGRATION_CONTRACT_GATE]
         [Вставить блок EXISTING_MECHANISM_GATE]
         [Вставить блок CONDITIONAL_ACTION_GATE]
         [Вставить блок ORCHESTRATOR_IMPLEMENTATION_GATE]
         [Вставить блок EXTENSION_GUARD — если файл в cfe/]
         [Вставить блок CONTEXT_SAFETY — если файл в cfe/]

         ## 6. СТАНДАРТЫ
         Следуй .cursor/docs/1c-coding-standards.md.

         ## 7. САМОКОНТРОЛЬ
         Перед завершением проверь:
         - Каждая защитная проверка обоснована контрактом (не компенсация незнания)
         - Каждая Попытка обоснована внешним фактором (rule 20); нет silent degradation
         - Код вне #Вставка/#Удаление не изменён (для &ИзменениеИКонтроль); используй rollback protocol из onec-code-writer.md → Phase 5 п.6 при обнаружении нарушения
         - Нет дублирования логики вызываемых функций
         - Контекст выполнения корректен (клиент/сервер)
         - Нет инвертированных ранних выходов перед единственным действием (rule 23)
         - Для новых функций/процедур: параметры не перезаписываются (AP-007); при нормализации ввести локальную переменную

         ## 8. OUTPUT REQUIREMENTS
         В финальном ответе обязательны блоки (полный формат — onec-code-writer.md § OUTPUT FORMAT):
         - ## Changes (с полными code fences или diff)
         - ## Code-Truth Symbols (mandatory, YAML по формату агента)
         - ## Gate Results (G14, G16, G18, G19, G20)
         - ## Key Decisions
         - ## Acceptance Criteria
         - ## Next Steps",
  subagent_type="onec-code-writer"
)
```

---

## Writer — bug fix (исправление ошибки)

```
Task(
  description="Исправить [описание бага]",
  prompt="## 1. ЗАДАЧА
         Исправь ошибку для [feature/модуль].
         Задача: [описание и критерии из tasks.md или промпта пользователя]

         ## 2. ROOT CAUSE (ОБЯЗАТЕЛЬНО для bug fix)
         Симптом: [что наблюдается — текст ошибки, неверное поведение]
         Корневая причина (verified): [конкретная точка в коде — файл:строка, что именно не так]
         Доказательство: [трасса строка N / лог / код файл:строка / воспроизведённый сценарий]
         Подход к фиксу: [что менять и ПОЧЕМУ это устраняет причину, а не маскирует симптом]

         КРИТИЧНО: Фикс должен устранять корневую причину. Если задача выглядит
         как заплатка (добавить проверку на Неопределено без объяснения ПОЧЕМУ
         значение Неопределено) — STOP, запросить уточнение.

         ## 3. КОНТЕКСТ
         Design: [путь к design.md, если есть]
         Файлы: [пути к .bsl файлам для правки]

         ## 4. ОГРАНИЧЕНИЯ
         Только правка существующих .bsl, без новых файлов и метаданных.

         ## 4a. COMMENT MARKERS
         Все новые вставки кода должны быть обёрнуты в маркеры:
         Open: [вставить open_marker]
         Close: [вставить close_marker]
         domain_label на русском, без process-меток (openspec/project.md § Канон domain_label).
         (Правила размещения — см. твой системный промпт §3a).

         ## 5. INPUT CONTRACT
         [Вставить блок ## Project paths из project-paths.mdc]
         [Вставить блок ## BSL_LSP Status: connected|not_connected]

         ## 6. GATES
         [Вставить блок DATA_CONTRACT_GATE]
         [Вставить блок INTEGRATION_CONTRACT_GATE]
         [Вставить блок EXISTING_MECHANISM_GATE]
         [Вставить блок CONDITIONAL_ACTION_GATE]
         [Вставить блок ORCHESTRATOR_IMPLEMENTATION_GATE]
         [Вставить блок EXTENSION_GUARD — если файл в cfe/]
         [Вставить блок CONTEXT_SAFETY — если файл в cfe/]

         ## 7. СТАНДАРТЫ
         Следуй .cursor/docs/1c-coding-standards.md.

         ## 8. САМОКОНТРОЛЬ
         Перед завершением проверь:
         - Фикс устраняет корневую причину, а не маскирует симптом
         - Каждая защитная проверка обоснована контрактом (не компенсация незнания)
         - Каждая Попытка обоснована внешним фактором (rule 20); нет silent degradation
         - Код вне #Вставка/#Удаление не изменён (для &ИзменениеИКонтроль); используй rollback protocol из onec-code-writer.md → Phase 5 п.6 при обнаружении нарушения
         - Нет дублирования логики вызываемых функций
         - Нет инвертированных ранних выходов перед единственным действием (rule 23)
         - Для новых функций/процедур: параметры не перезаписываются (AP-007); при нормализации ввести локальную переменную

         ## 9. OUTPUT REQUIREMENTS
         В финальном ответе обязательны блоки (полный формат — onec-code-writer.md § OUTPUT FORMAT):
         - ## Changes (с полными code fences или diff)
         - ## Code-Truth Symbols (mandatory, YAML по формату агента)
         - ## Gate Results (G14, G16, G18, G19, G20)
         - ## Key Decisions
         - ## Acceptance Criteria
         - ## Next Steps",
  subagent_type="onec-code-writer"
)
```

---

## Writer — review fix (устранение замечаний ревью)

```
Task(
  description="Устранить замечания ревью [scope]",
  prompt="## 1. ЗАДАЧА
         Устрани кодовые замечания ревью в файле [путь].

         ## 2. ЗАМЕЧАНИЯ (MUST_FIX)
         Для каждого замечания (отсортированы по risk_score desc — начинай с верха списка):
         - ID: [порядковый номер]
         - AP: [AP-NNN или Phase0-TYPE / release-hygiene-TAG]
         - Type: CODE
         - Severity: [CRITICAL/HIGH/MEDIUM/LOW]
         - Risk: [risk_score + оси: scope/blast_radius/frequency/confidence]
         - Procedure: [имя процедуры]
         - Anchor: [фрагмент кода для поиска]
         - Issue: [описание]
         - Fix: [рекомендация ревьювера]

         Правила приоритизации (новая модель риска из onec-code-reviewer v3.0):
         - Начинай с верхних findings (high risk_score) — они несут больший совокупный риск
           с учётом scope (blast-radius вовне), blast_radius (от cosmetic до security),
           frequency (hot-path > rare) и confidence.
         - Severity (CRITICAL/HIGH/MEDIUM/LOW) — база, но не единственный критерий порядка.
         - Recurrent tag (если есть) — finding встречался в предыдущих ревью того же scope;
           уделить особое внимание root cause, а не симптомам.

         [Список замечаний с Action=MUST_FIX и Type=CODE]

         ## 3. INPUT CONTRACT
         [Вставить блок ## Project paths из project-paths.mdc]
         [Вставить блок ## BSL_LSP Status: connected|not_connected]

         ## 3a. RESOLVED CONTRACTS (если переданы оркестратором)
         [Вставить блок ## Resolved Contracts из reports/resolved-contract-*.md]

         Как использовать:
         - Contract: fixed — поля контракта гарантированы кодом.
           НЕ добавлять ТипЗнч/Свойство/Попытка для полей этого контракта.
           Обращаться к полям напрямую.
         - Contract: dynamic — поля могут отсутствовать.
           Следовать ремедиации ревьювера (минимальная проверка
           или обоснованная Попытка).
         - Если замечание ревьювера касается контракта из блока —
           использовать Evidence и Keys для корректной правки.

         ## 4. ОГРАНИЧЕНИЯ
         - Правь только по перечисленным замечаниям.
         - Не менять логику за пределами замечаний.
         - Если замечание требует вынос в новую функцию — проверь
           AP-007 (не перезаписывать параметры: ввести локальную
           переменную).

         ## 4a. COMMENT MARKERS
         Если исправление требует добавления новых строк — оберни их в маркеры:
         Open: [вставить open_marker]
         Close: [вставить close_marker]
         domain_label на русском, без process-меток (openspec/project.md § Канон domain_label).
         (Правила размещения — см. твой системный промпт §3a).

         ## 5. GATES
         [Вставить блок DATA_CONTRACT_GATE]
         [Вставить блок INTEGRATION_CONTRACT_GATE]
         [Вставить блок CONDITIONAL_ACTION_GATE]
         [Вставить блок ORCHESTRATOR_IMPLEMENTATION_GATE]
         [Вставить блок EXTENSION_GUARD — если файл в cfe/]

         ## 6. СТАНДАРТЫ
         Следуй .cursor/docs/1c-coding-standards.md.

         ## 7. САМОКОНТРОЛЬ
         - Каждое замечание из списка адресовано (или обосновано, почему нет)
         - Для новых функций: параметры не перезаписываются (AP-007)
         - Для НСтр: кавычки внутри строки экранированы двойными
         - Нет побочных изменений за пределами замечаний
         - Код вне #Вставка/#Удаление не изменён (для &ИзменениеИКонтроль); используй rollback protocol из onec-code-writer.md → Phase 5 п.6 при обнаружении нарушения

         ## 8. OUTPUT REQUIREMENTS
         В финальном ответе обязательны блоки (полный формат — onec-code-writer.md § OUTPUT FORMAT):
         - ## Changes (с полными code fences или diff)
         - ## Code-Truth Symbols (mandatory, YAML по формату агента)
         - ## Gate Results (G14, G16, G18, G19, G20)
         - ## Key Decisions
         - ## Acceptance Criteria
         - ## Next Steps",
  subagent_type="onec-code-writer"
)
```
