# Architect — шаблоны промптов

Шаблоны для делегирования `Task(subagent_type="onec-code-architect", model=...)` (модель — по `.cursor/rules/model-selection.mdc`). Общие правила, оценка сложности, выбор модели, обработка ошибок — `SKILL.md` (навигатор).

---

## Architect (проектирование)

```
Task(
  description="Спроектировать [feature]",
  prompt="mode=design tier=[simple|medium|complex|critical]

         Спроектируй архитектуру для [feature].

         Артефакты: [пути к proposal.md, design.md если есть]
         Результат исследования: [резюме от explorer]
         Подход: [минимальные изменения / чистая архитектура / баланс]

         ## Existing Knowledge
         [Вставить KB из Discovery по формату KB CONTEXT или: Discovery выполнен, совпадений нет]
         Инструкция: применить блок EXISTING_KNOWLEDGE.

         Обязательная секция в результате: Existing Mechanisms
         (что найдено в базе, какой контракт/API у каждого механизма,
         какой уровень Preference Hierarchy выбран, почему уровни выше
         не подошли при выборе уровня 3 или 4).

         Создай план с этапами реализации (атомарные, с критериями приемки).
         Используй Mermaid диаграммы.
         Обязательно добавь YAML front-matter, Evidence для каждого паттерна и Test Scenarios.",
  subagent_type="onec-code-architect",
  model="<slug из .cursor/rules/model-selection.mdc>"
)
```

---

## Architect (ревью плана)

```
Task(
  description="Ревью плана [feature]",
  prompt="mode=plan-review review_mode=peer

         Проверь design.md для [feature].

         Артефакты: [пути к proposal.md, design.md, tasks.md]

         ## Existing Knowledge
         [Вставить KB из Discovery по формату KB CONTEXT или: Discovery выполнен, совпадений нет]
         Инструкция: применить блок EXISTING_KNOWLEDGE.

         Фокус: полнота, корректность, реалистичность, соответствие требованиям,
         чёткость критериев приемки, правильность зависимостей.
         Все задачи на verified facts? Если на hypotheses — есть ли задача верификации?

         Если проблемы — опиши и предложи исправления.
         Обязательно добавь YAML front-matter.",
  subagent_type="onec-code-architect"
)
```

---

## Architect (глубокий анализ)

```
Task(
  description="Глубокий анализ [область]",
  prompt="mode=deep-analysis

         Проанализируй результаты исследования кода.

         Отчёт explorer: [путь к exploration-*.md]
         Контекст задачи: [из брифа / proposal]

         ## Existing Knowledge
         [Вставить KB из Discovery по формату KB CONTEXT или: Discovery выполнен, совпадений нет]
         Инструкция: применить блок EXISTING_KNOWLEDGE.

         Фокус (АНАЛИЗ, не проектирование):
         1. Gap analysis: что explorer мог пропустить?
            Какие модули/вызовы не охвачены?
         2. Контракты: задокументированы ли входные контракты
            ключевых функций? Допустимые типы, внутренняя
            нормализация, гарантии.
         3. Противоречия: есть ли расхождения между
            паттернами в коде и требованиями задачи?
         4. Альтернативные точки реализации — все ли
            рассмотрены? Почему одна лучше другой?
         5. Root cause верификация (для bug fix): факты
            из explorer подтверждены кодом?
         
         Результат: дополненный анализ (gap analysis,
         верифицированные контракты, рекомендации).
         Обязательно добавь YAML front-matter.
         Сохранить по правилу preserve-subagent-reports:
         openspec/changes/<name>/reports/deep-analysis-YYYY-MM-DD.md
         при активном change или
         temp/reports/deep-analysis-YYYY-MM-DD.md вне change.",
  subagent_type="onec-code-architect"
)
```

---

## Architect — scope coherence audit (extend)

Используется `/opsx:extend`, шаг **5a**, когда сработал семантический триггер (`Drift-check` ≠ pass) или объективный счётчик Extend без архитектора (см. `.cursor/skills/openspec-extend-change/SKILL.md`). Режим **`scope-coherence-audit`**: Simplicity Check не требуется.

```
Task(
  description="Scope coherence audit [change-name]",
  prompt="mode=scope-coherence-audit

         Оцени, осталась ли ЗНИ `<name>` единой согласованной задачей или
         расползлась в несколько слабосвязанных. Сравни текущие
         proposal.md / design.md / tasks.md / debug.md с исходным замыслом
         (Why, Non-Goals в proposal/design).

         ## Артефакты и входы

         - Текущие proposal, design, tasks, debug (полные тексты или пути)
         - Исторический proposal (git show <hash>:... или явно «недоступен»)
         - Блок «Соответствие исходному scope» из брифа extend
         - Список reports/*.md с датами
         - debug.md: все секции «## Extend —» после последнего
           architecture-extend-coherence-*.md или с начала change

         ## Existing Knowledge
         [KB Discovery по правилам architect-gate.mdc]

         ## Existing ADR
         [ADR Discovery по правилам architect-gate.mdc]

         ## Вопросы (дай ответ по каждому)

         1. Каждая значимая задача в tasks.md соответствует пункту Why?
            Перечисли отклонения.
         2. Нарушены ли Non-Goals? Перечисли.
         3. Decisions в design.md согласованы между собой или есть «тихие»
            противоречия?
         4. Behavior Contract шире, чем покрывают чеклисты приёмочных задач S<N>.accept (или legacy S<N>.T<M>)?
         5. Есть ли в debug.md Extend-секции, ведущие к объёму работ без
            архитектурного сопровождения?
         6. Это всё ещё одна цель ЗНИ или уже 2+ независимых?

         ## Формат ответа (обязательно)

         YAML front-matter:
         mode: scope-coherence-audit
         scope.files: [...]
         scope.modules: [...]

         ### Verdict
         coherent | drift-warning | scope-violation

         ### Findings
         (по пунктам 1–6)

         ### Recommendations
         Конкретные действия: rebrick срезов, вынести в новую ЗНИ,
         удалить избыточные задачи, обновить proposal/Non-Goals.

         ### Closes
         Какие scope-drift триггеры этот отчёт закрывает.

         Simplicity Check не включать.

         Сохранить полный отчёт в:
         openspec/changes/<name>/reports/architecture-extend-coherence-YYYY-MM-DD.md",
  subagent_type="onec-code-architect"
)
```

---

## Architect — design challenge (verify Layer 4)

Used by `/opsx:verify` Layer 4 — независимый адверсариальный аудит, что `design.md` действительно решает проблему из `proposal.md` оптимальным способом. **Не** дублирует Architect Gate из `/opsx:new` (там — auctorial design); здесь — adversarial challenge.

Подробный протокол (адверсариальная установка, Three-Question Challenge, формат отчёта, маппинг вердикта) — см. `.cursor/agents/onec-code-architect.md` секция «Режим `design-challenge`».

**Модель.** По цепочке для архитектора из `.cursor/rules/model-selection.mdc`. **Режим запуска: `run_in_background=true`** (см. SKILL.md verify, секция «Запуск агентов verify»).

```
Task(
  description="Design Challenge [change-name]",
  prompt="mode=design-challenge

         ## Задача

         Независимый адверсариальный аудит `design.md` ЗНИ `<change-name>`.
         Атаковать решение по Three-Question Challenge (Q1 Problem-Solution
         Fit, Q2 Optimality vs ≥2 неупомянутых альтернатив, Q3 Fresh-Eye
         Approval). Полный протокол режима — см. .cursor/agents/onec-code-architect.md
         секция «Режим design-challenge».

         ## Артефакты (первичные источники истины)

         - proposal: openspec/changes/<change-name>/proposal.md
         - design: openspec/changes/<change-name>/design.md
         - specs: openspec/changes/<change-name>/specs/**/spec.md

         ## Запреты

         - Не опираться на собственные прошлые отчёты `reports/architecture-*.md`
           как на источник истины — каждый аргумент должен ссылаться на
           proposal.md / design.md / specs / файлы кода / вендорские стандарты.
         - Не повторять секции «Simplicity Check», «Found Patterns»,
           «Architecture» из режима design — это не дизайн-сессия.
         - Не молчать. Если не нашёл что атаковать — показать ≥3 альтернативы
           и обосновать, почему ни одна не лучше.

         ## Результат

         Сохранить отчёт в:
         openspec/changes/<change-name>/reports/design-challenge-YYYY-MM-DD.md

         YAML: verdict ∈ {APPROVE | CHALLENGE | REJECT}, confidence,
         scope.design_mtime.

         ## Final message to chat (HARD CONSTRAINT)

         Your final assistant message in this turn is a single line:
         \"Отчёт сохранён: openspec/changes/<change-name>/reports/design-challenge-YYYY-MM-DD.md\".

         Do NOT include verdict, severity, three-question challenge summary,
         layer name, alternative names, recommendations, или any other
         analysis в финальном сообщении. Full analysis goes ONLY to the
         saved markdown file.

         Reason: the user does NOT read your final message. The orchestrator
         reads the file and synthesizes a single message for the user.
         Anything you put в финальный assistant message becomes user-visible
         chat noise.

         Cursor will auto-generate a user_visible_high_level_summary card from
         your reasoning. To keep that card readable by the orchestrator (not by
         the user as raw text), AVOID in your reasoning steps:
         - artifact IDs without unfolding: S1.0, S1.T5, Option A, F1
         - engine layer names: Layer N, design-challenge, task-readiness, QC,
           PASS/FAIL/CHALLENGE/APPROVE/REJECT, slice coherence, snapshot
         - internal section names: Three-Question Challenge, Simplicity Check
         Use plain language: «первый ручной шаг», «приёмочный тест единственного
         шаблона», «выбранный вариант реализации».",
  subagent_type="onec-code-architect",
  model="<по цепочке для архитектора из model-selection.mdc>",
  run_in_background=true
)
```

---

## Architect — task readiness review (verify Layer 5)

Used by `/opsx:verify` Layer 5 (Implementation Readiness) — MANDATORY in every pre-apply verification. The architect evaluates holistic readiness: can the ЗНИ be implemented as-is by agents and users without returning for clarification?

```
Task(
  description="Ревью готовности задач [change-name]",
  prompt="mode=task-readiness
         
         ## Задача
         
         Оцени готовность ЗНИ `<name>` к реализации.
         Не детали кода — целостная оценка: можно ли по этим
         артефактам реализовать ЗНИ силами агентов (writer — BSL,
         включая программное создание элементов формы в модуле)
         и пользователя (ручная конфигурация в Конфигураторе)
         без возвратов на уточнение?

         ## Артефакты

         - proposal: <путь>
         - design: <путь>
         - tasks: <путь>
         - specs: <путь>
         - Чеклист ручной конфигурации (verify, Layer 2):
           <чеклист-таблица или «маркеров не найдено»>
         - Замечания механических проверок (verify, Layers 1–3):
           <список или «замечаний нет»>
         - Executability issues (verify, Layer 5 pre-screen): <список или «замечаний нет»>

         ## Критерии оценки

         Для каждого — вердикт (OK / GAP) и обоснование:

         1. **Реализуемость кодовых задач.** Может ли writer
            реализовать каждую задачу, имея только design + spec
            + текст задачи? Непонятно ЧТО или ГДЕ делать?

         2. **Реализуемость форм и метаданных.** Достаточно ли design
            для ручной конфигурации формы в Конфигураторе или
            программного создания элементов в модуле формы (BSL)?
            Может ли пользователь создать метаданные без вопросов?
            Описаны ли элементы формы, UX-сценарий?

         3. **Разрешённость решений.** Все ли «или»/«/» разрешены?
            Неопределённые контракты возврата? Задачи с двумя
            путями без выбора?

         4. **Полнота покрытия.** Покрывают ли задачи все
            requirements из spec? Пробелы?

         5. **Согласованность.** Противоречия tasks↔design?
            tasks↔spec? «создать» vs реальное состояние репо?

         6. **Связность кода и порядок задач.** Может ли исполнитель
            (агент или пользователь) выполнить задачу при текущем
            состоянии других задач и порядке в файле?
            - Кодовые задачи: все ли объекты/процедуры, на которые
              ссылается задача, созданы или реализованы
              предшествующими задачами (в том же срезе или в принятых
              зависимостях)?
            - Slice-gate: для каждого среза — финальная приёмочная
              задача (`S<N>.accept` для новой модели или legacy
              `S<N>.T<M>`) присутствует ровно одна? Маркер
              `<!-- slice-gate -->` корректен? Граф зависимостей
              `**Зависимости:**` в метаданных среза согласован с
              остальными срезами?

         7. **Архитектурная эстетика (Design Smells).** Нет ли в проекте
            архитектурных запахов?
            - Over-engineering: переусложнение (например, новый регистр там, где хватит реквизита).
            - Invasiveness: высокая инвазивность (необоснованный &ИзменениеИКонтроль вместо &После или подписок).
            - Reinventing the wheel: игнорирование существующих механизмов или БСП.

         ## Out of scope

         Не оценивай: выполним ли приёмочный чеклист пользователем сейчас;
         нужны ли тестовые документы; есть ли эталон ИБ до перехвата;
         smoke-сценарии. Это apply/archive. **Structural user-spike в
         `S<N>.<M>` — in scope** (критерий 8), не путать с отсутствием
         тестовых данных.

         ## Формат ответа

         ### Вердикт
         ГОТОВО / ГОТОВО С ЗАМЕЧАНИЯМИ / НЕ ГОТОВО

         ### Оценка по критериям
         | # | Критерий | Вердикт | Обоснование |
         |---|----------|---------|-------------|
         | 1 | Реализуемость кодовых задач | OK/GAP | ... |
         | 2 | Реализуемость форм и метаданных | OK/GAP | ... |
         | 3 | Разрешённость решений | OK/GAP | ... |
         | 4 | Полнота покрытия | OK/GAP | ... |
         | 5 | Согласованность | OK/GAP | ... |
         | 6 | Связность кода и порядок задач | OK/GAP | ... |
         | 7 | Архитектурная эстетика (Design Smells) | OK/SUBOPTIMAL | ... |
         | 8 | User Task Contract | OK/GAP | user runtime-spike в `S<N>.<M>`? |

         ### Пробелы (только при GAP или SUBOPTIMAL)
         Для каждого GAP:
         - Задача / артефакт
         - Что отсутствует / неоднозначно
         - Рекомендация (что дополнить, где)
         - Если GAP связан с отсутствием пути, подсистемы или пропущенной задачей из design, предоставь готовый сниппет (строку) для автоматической вставки в артефакт, чтобы оркестратор мог применить его как авто-исправление Layer 1.

         НЕ НУЖНО: ревью архитектуры, оценка рисков,
         альтернативные подходы.
         Только: можно ли реализовать as-is.

         Результат: сохранить в
         openspec/changes/<change-name>/reports/task-readiness-review-YYYY-MM-DD.md.

         ## Final message to chat (HARD CONSTRAINT)

         Your final assistant message in this turn is a single line:
         \"Отчёт сохранён: openspec/changes/<change-name>/reports/task-readiness-review-YYYY-MM-DD.md\".

         Do NOT include verdict, severity, gap summary, layer name,
         simplicity check, recommendations, или any other analysis в финальном
         сообщении. Full analysis goes ONLY to the saved markdown file.

         Reason: the user does NOT read your final message. The orchestrator
         reads the file and synthesizes a single message for the user.
         Anything you put в финальный assistant message becomes user-visible
         chat noise.

         Cursor will auto-generate a user_visible_high_level_summary card from
         your reasoning. To keep that card readable by the orchestrator (not by
         the user as raw text), AVOID in your reasoning steps:
         - artifact IDs without unfolding: S1.0, S1.T5, Option A, F1
         - engine layer names: Layer N, design-challenge, task-readiness, QC,
           PASS/FAIL/CHALLENGE/APPROVE/REJECT, slice coherence, snapshot
         - internal section names: Three-Question Challenge, Simplicity Check
         Use plain language: «первый ручной шаг», «приёмочный тест единственного
         шаблона», «выбранный вариант реализации».",
  subagent_type="onec-code-architect",
  run_in_background=false
)
```

---

## Architect — slice transition review (slice-gate)

Used on the slice boundary (option «пересмотр» on `/opsx:apply` slice-gate, or `/opsx:verify` pre-apply scoped to the transition between slices). Assesses whether upcoming slices are still valid given what was implemented in the accepted slice.

```
Task(
  description="Slice transition review [change-name]",
  prompt="mode=slice-transition
         
         ## Задача
         
         Оцени актуальность задач и сценария следующего среза
         (S<N+1>) ЗНИ `<name>` после принятия среза S<N>.

         ## Контекст

         - tasks: <путь> (задачи принятого S<N> с [x],
           задачи следующих срезов с [ ])
         - design: <путь> (с секцией ## Slices)
         - debug.md: <путь или «отсутствует»>,
           секция `## Slice Gate Decisions` — решения по принятым срезам
         - openspec/changes/<change-name>/reports/slice-acceptance-S<N>-*.md: <пути>
         - Недавние отчёты: <пути к openspec/changes/<change-name>/reports/*.md>

         ## Критерии

         1. **Актуальность сценария.** Пользовательский сценарий
            следующего среза (из блока метаданных) остаётся
            валидным после принятия S<N>? Нет ли дублирования с
            уже принятым сценарием?

         2. **Актуальность задач.** Описания задач S<N+1> соответствуют
            текущему состоянию кода? Нет ли ссылок на подходы, которые
            изменились при реализации S<N>?

         3. **Design drift.** Отличается ли реализация S<N> от того,
            что описано в `## Slices` и прочем design? Если да —
            какие срезы затронуты (их файлы, их сценарии)?

         4. **Связь со spec.** Scenarios из spec, на которые ссылается
            S<N+1>, не покрыты полностью принятым S<N>? (Иначе срез
            S<N+1> можно удалить.)

         5. **Необходимость перестройки среза.** Нужно ли объединить
            следующий срез с одним из более поздних, разделить его
            на два, перенести задачи в другой срез?

         6. **Новые срезы/задачи.** Выявились ли при реализации S<N>
            пользовательские сценарии, не предусмотренные в текущем
            `## Slices`? Нужен ли новый срез?

         7. **Граф зависимостей.** Зависимости, объявленные в S<N+1>
            (`Зависимости:`), выполнены? Если нет — это блокер.

         ## Формат ответа

         ### Вердикт
         СРЕЗ S<N+1> АКТУАЛЕН / ТРЕБУЕТ КОРРЕКТИРОВКИ / УСТАРЕЛ

         ### Оценка
         | # | Критерий | Вердикт |
         |---|----------|---------|
         | 1 | Актуальность сценария | OK/DRIFT/DUPLICATE |
         | 2 | Актуальность задач | OK/DRIFT |
         | 3 | Design drift | OK/DRIFT |
         | 4 | Связь со spec | OK/EMPTY/INCOMPLETE |
         | 5 | Перестройка среза | НЕ НУЖНА/РЕКОМЕНДУЕТСЯ |
         | 6 | Новые срезы/задачи | НЕТ/ЕСТЬ |
         | 7 | Граф зависимостей | OK/BLOCKED |

         ### Рекомендации (при DRIFT/РЕКОМЕНДУЕТСЯ/ЕСТЬ/BLOCKED)
         Конкретные изменения в tasks.md и/или design.md ## Slices.

         Результат: сохранить в
         openspec/changes/<change-name>/reports/slice-transition-S<N+1>-YYYY-MM-DD.md.",
  subagent_type="onec-code-architect"
)
```

---

## Architect — slice restructuring (ремедиэйшн после verify / `/opsx:extend`)

Вызывается при **перестройке** `tasks.md` в вертикальные срезы: когда verify обнаружил несовместимый с срезами формат (**нет** заголовков `# Срез` при slice-ориентированной ЗНИ) или маркеры `<!-- phase-gate -->`, и пользователь запустил **`/opsx:extend <name>`** с architect slice restructuring (или аналогичный явный триггер). Restructures an existing flat or phase-based tasks.md into vertical slices. Preserves existing tasks, their numbering, and `[x]`/`[ ]` statuses; re-IDs задач с префиксом среза ТОЛЬКО если явно указано.

```
Task(
  description="Slice restructuring [change-name]",
  prompt="mode=slice-restructuring
         
         ## Задача
         
         Перестрой tasks.md ЗНИ `<name>` в структуру вертикальных
         срезов (см. .cursor/rules/vertical-slices.mdc).
         Исходный tasks.md может быть плоским, либо организован
         по фазам P0–P4 (устаревший формат) — оба случая мигрируем.

         ## Артефакты

         - tasks: <путь> (существующие задачи, часть может быть [x])
         - design: <путь> (если есть секция ## Slices — использовать её;
           если нет — сгенерировать новую секцию и включить в ответ)
         - proposal: <путь>
         - specs: <путь к specs/ — для сопоставления Scenarios срезам>

         ## Требования

         1. **Сохранить задачи.** Все существующие задачи остаются
            as-is по тексту и статусу [x]/[ ]. Нумерация может
            измениться: ID задач получают префикс среза (см. п.4).
            Оригинальные номера сохранить в комментарии или оставить
            без префикса, если переход без ломания трассируемости
            невозможен.
         2. **Сгруппировать задачи по срезам.** Каждый срез маппится
            на 1–3 пользовательских сценария из spec. Слои (метаданные,
            форма, BSL, визы) группируются внутри среза вокруг сценария.
         3. **Заголовки срезов:** `# Срез S<N>: <имя>` (H1).
            Под каждым — блок метаданных:
            ```
            **Сценарий:** <пользовательский сценарий>
            **Приёмка:** <ручной тест / авто / проверка в ИБ>
            **Связь со spec:** <Requirement «…», Scenario «…»>
            **Зависимости:** <нет | S<K>, S<L>>
            ```
         4. **ID задач с префиксом среза:** `- [ ] S<N>.<M> <текст>`.
            Для уже завершённых задач (`[x]`) — сохранить галочку.
            Приёмочная задача — **ровно одна** на срез:
            ```
            - [ ] S<N>.accept Принять срез S<N> «<имя>» — <бизнес-результат>:
              - Scenario «<имя 1 буквально из spec>»: <одна строка>
              - Scenario «<имя 2>»: <одна строка>
            ```
            Если в исходном tasks.md есть legacy-формат `S<N>.T<M>`
            (один или несколько на срез) — при реструктуризации можно
            **сохранить как есть** (legacy остаётся работоспособным)
            или объединить в один `S<N>.accept` со чеклистом, если
            пользователь явно об этом попросил при миграции.
            Если в исходном tasks.md приёмки нет вовсе — создать
            `S<N>.accept` с чеклистом по Scenarios из «Связь со spec».
         5. **Маркер slice-gate.** После последней задачи среза:
            `<!-- slice-gate: <критерий приёмки, одно предложение> -->`
         6. **Удалить** заголовки `# Фаза N`, маркеры `<!-- phase-gate -->`,
            упоминания P0–P4, правила упорядочивания по слоям.
         7. **Legacy-срез.** Если часть задач принадлежит принятому
            ранее функционалу (все связанные задачи `[x]`), допустимо
            поместить их в срез-контейнер `S0: Legacy (read-only)` с
            пометкой «миграция из фазовой структуры». Новые срезы
            начинаются с S1.
         8. **Покрытие Scenarios.** После реструктуризации каждый
            `#### Scenario:` из spec покрыт хотя бы одним срезом.
            Если пробел обнаружен — добавить срез или задачу в
            существующий.
         9. **Граф зависимостей** — описать внутри ответа (список или
            mermaid); циклы недопустимы.
         10. **Remediation: ошибочный fix-срез на непринятом S<K>.**
            Если в `tasks.md` уже есть `# Срез S<N+1>` с признаками
            исправления (подстрока «Исправление»/«Fix» в заголовке **или**
            в метаданных `**Зависимости:** S<K>`) и при этом приёмка
            `S<K>.T<M>` у зависимого среза = **`[ ]`** — это нарушение
            `.cursor/rules/vertical-slices.mdc` (**ИНВАРИАНТ: Defect placement**).
            В ответе **обязательно** дать пошаговый план миграции:
            перенести задачи `S<N+1>.<M>` внутрь `S<K>` как `S<K>.<M_new>`
            **перед** `S<K>.T<M>`, удалить блок `# Срез S<N+1>` целиком,
            поправить `design.md` `## Slices` (убрать строку лишнего среза)
            и при необходимости `debug.md` (`Решение: inside-slice rework`).
         11. **Remediation: non-vertical / foundation срез (`slice-not-vertical`,
             `slice-foundation-with-gate`, QC критерии 8–9).**
            Если срез `S<K>` содержит только подготовительный код (функция,
            API, хелпер) и programmatic-only accept (вызов функции,
            проверка возвращаемого значения, код-ревью контракта), а
            следующий срез `S<K+1>` **подключает** этот код и даёт
            user-journey accept — **свернуть** S<K> и S<K+1> в один
            срез: код-задачи S<K> → `S<M>.<M>` внутри объединённого
            среза; implementation Scenario → **agent** verification
            `S<M>.<M>` («по коду», не user IB) с пометкой «(verification,
            не accept)»; приёмка S<K+1> →
            `S<M>.accept` с black-box чеклистом; удалить блок
            `# Срез S<K>`; синхронизировать `design.md` `## Slices`.

         ## Формат ответа

         1. Полный обновлённый tasks.md (весь файл).
         2. Если design.md не содержал `## Slices` — сниппет этой
            секции для вставки в design (таблица срезов + граф
            зависимостей + покрытие Scenarios).
         3. Summary миграции:
            - Сколько срезов получилось.
            - Какие задачи переименованы (старый ID → новый ID), если
              были переименования.
            - Если сработал п.10 (ошибочный fix-срез) — явно: что удалено,
              куда перенесены задачи, какие артефакты (`design.md`, `debug.md`)
              синхронизировать.
            - Какие Scenarios из spec оказались без покрытия (если
              такие есть — явно перечислить).

         Формат срезов: .cursor/rules/vertical-slices.mdc",
  subagent_type="onec-code-architect"
)
```

---

## Architect — slice decomposition (декомпозиция на вертикальные срезы)

Used BEFORE task decomposition: architect produces the `## Slices` section of `design.md`. Входит в Slice Generation Gate скилла `openspec-new-change`.

```
Task(
  description="Декомпозиция на срезы [feature]",
  prompt="mode=slice-decomposition
         
         Декомпозируй ЗНИ на вертикальные пользовательские срезы.

         Артефакты:
         - proposal: [путь]
         - design: [путь без секции ## Slices]
         - specs: [путь к specs/ — все Requirements и Scenarios]

         Что такое срез (Slice) — см. .cursor/rules/vertical-slices.mdc:
         Минимальная единица, которую пользователь может принять
         независимо end-to-end (открыл продукт → выполнил сценарий →
         получил ожидаемый результат).

         Правила декомпозиции на срезы:
         1. Один срез = 1–3 связанных пользовательских сценария.
            Каждый срез SHALL маппиться на ≥1 #### Scenario: из spec.
         2. Полный стек внутри среза: метаданные, форма, BSL, общий
            модуль, визы, миграция — всё, что нужно для приёмки.
         3. Независим при приёмке: срез S<N> принимается без S<N+1..>.
            Зависимости только «назад»: S<N> может требовать S<K>, K<N.
         4. В каждом срезе — как минимум 1 приёмочный сценарий
            (описывается в поле «Приёмка»).
         5. Покрытие: каждый #### Scenario: из spec SHALL быть покрыт **primary-срезом**
            (буллет `S<N>.accept` где код даёт UX) **или** **agent** verification-задачей
            `S<N>.<M>` («верифицировать по коду» / static) без отдельного accept.
            User IB/runtime spike **запрещён** (User Task Contract).
            **Не** 1:1 Requirement → Slice. Отдельный срез с accept — только для
            самостоятельного user outcome. Risks с «runtime-verify» → Assumptions +
            проверка в Primary accept, не user-задача.
         6. Размер порога (см. `.cursor/rules/vertical-slices.mdc` «ТРИГГЕРЫ И ПОРОГИ»):
            - ≤ 5 задач (Lite) — срезы опциональны (1 срез-контейнер).
            - 6–15 задач (Standard) — **1 срез по умолчанию**; второй+ только при
              независимых пользовательских outcomes.
            - ≥ 16 задач (Full) — минимум 2 среза **при 2+ outcomes**; иначе 1 срез
              + группы `## N.` внутри.
         7. Срез не может быть чисто подготовительным слоем
            (одни метаданные / одна функция без UX-эффекта) — это не
            срез. Подготовительный код SHALL включаться в primary-срез,
            где код **вызывается** и даёт наблюдаемый результат.
            Option B (новый API + потребитель) → **один срез**: API =
            первые задачи `S1.1`, один `S1.accept` на все Scenario.
            «Ревью контракта», «код-ревью», static verification — agent
            задачи `S<N>.<M>`, не `S<N>.accept`. User runtime (ИБ, отладчик,
            консоль) — только `S<N>.accept` на границе среза, не spike
            посередине. См. QC критерии 8–11 и User Task Contract в
            `.cursor/rules/vertical-slices.mdc`.
         8. НЕ использовать классификацию P0–P4, «# Фаза N»,
            «<!-- phase-gate -->» — эти концепции отменены.
         9. **Primary acceptance + Acceptance Checklist Coverage** (правило среза 6 в
            `.cursor/rules/vertical-slices.mdc`):
            - В каждом срезе **ровно один** blocking journey — **Primary acceptance**
              (поле metadata + mandatory sub-bullet в `S<N>.accept`).
            - Остальные Scenario — optional sub-bullets или задачи `S<N>.<M>`.
            - Self-check: «Может ли заказчик пройти Primary за ~5–15 мин на ИБ
              без отладчика?» — если нет, merge с consumer-срезом.
            - Запрещён programmatic-only Primary (diff, API, отладчик).
            - Инварианты/NFR — `design.md#Assumptions` или `S<N>.<M>`, не accept.
         10. **Self-check перед выводом `## Slices`:** для каждого среза —
            «Может ли пользователь принять этот срез без следующего
            **в терминах Why/proposal**?» Если нет — объединить срезы.
            Запрещён `foundation-slice-with-gate` (QC критерии 8–9).

         Формат вывода — готовый блок для вставки в design.md:

         ## Slices

         | Срез | Пользовательский сценарий | Primary acceptance | Scenarios из spec | Файлы/модули | Приёмка |
         |---|---|---|---|---|---|
         | S1: <имя> | <сценарий> | <один Given→When→Then> | <Scenario A, B> | <файлы> | <ручной тест> |

         ### Граф зависимостей
         - S1 → нет
         - S2 → S1
         - S3 → S1, S2
         (или mermaid-диаграмма при 4+ срезах)

         ### Критерии приёмки по срезам
         - **S1** SHALL: <критерий slice-gate, одно предложение>
         - **S2** SHALL: <критерий slice-gate>
         - ...

         ### Покрытие Scenarios (проверка)
         | Scenario (spec) | Покрыт срезом |
         |---|---|
         | <Scenario A> | S1 |
         | <Scenario B> | S1 |
         | <Scenario C> | S3 |

         Если какой-то Scenario не покрыт — это дефект декомпозиции,
         добавить срез или перенести в существующий.

         ### Чеклист приёмки (Acceptance Checklist)

         Для каждого среза — Primary (mandatory) + optional/verification mapping.

         | Срез | S<N>.accept (бизнес-результат) | Primary (mandatory) | Optional / S<N>.<M> |
         |---|---|---|---|
         | S1 | <бизнес-результат> | <шаги Primary> | Scenario «B» (опционально) / S1.5 verify |

         Инварианты, NFR и код-уровневые ревью в чеклист НЕ вносятся —
         они попадают либо в `design.md#Assumptions`, либо в обычные
         задачи `S<N>.<M>`, либо в блок `## Follow-up` (правило среза 6
         в `.cursor/rules/vertical-slices.mdc`).",
  subagent_type="onec-code-architect"
)
```

---

## Architect — slice-aware task decomposition (задачи по срезам)

Used AFTER slice decomposition: architect produces `tasks.md` with H1 slice headers using the approved `## Slices` section from design.md.

```
Task(
  description="Декомпозиция задач по срезам [feature]",
  prompt="mode=task-decomposition
         
         Декомпозируй design.md на атомарные задачи, сгруппированные
         по срезам (`## Slices` из design).

         Артефакты:
         - proposal: [путь]
         - design: [путь — обязано содержать секцию ## Slices]
         - specs: [путь к specs/]
         - template: [шаблон из openspec instructions]

         Правила декомпозиции задач:
         1. Каждая задача — атомарная (1 файл, 1 аспект).
         2. Критерии приёмки для каждой задачи (что проверить).
         3. Зависимости между задачами (что сначала) — внутри среза.
         4. Группировка по файлам (для параллелизации) внутри среза.
         5. Для каждой задачи: путь к файлу + что менять.

         Требования к структуре по срезам:
         6. tasks.md начинается с H1-заголовков срезов (БЕЗ `# Фаза`):
            `# Срез S<N>: <имя>`
         7. Под каждым H1 — блок метаданных (из design ## Slices):
            ```
            **Сценарий:** <из design>
            **Primary acceptance:** <из design — один blocking journey>
            **Приёмка:** <из design>
            **Связь со spec:** <Requirement «…», Scenario «…»>
            **Зависимости:** <нет | S<K>, S<L>>
            **Режим apply:** mechanical  <!-- опционально, migration-only -->
            ```
         8. Внутри среза задачи объединяются в группы H2
            `## N. <Группа>` (по слоям для читаемости): Метаданные,
            Форма, Объектный модуль, Общий модуль, Визы, Миграция,
            Приёмка. Группы опциональны при <5 задачах в срезе.
         9. ID задач имеют префикс среза: `- [ ] S<N>.<M> <описание>`.
         10. Последняя группа среза — Приёмка: **ровно одна** задача
             формата `- [ ] S<N>.accept Принять срез S<N> «<имя>» —
             <бизнес-результат>:` со sub-bullets чеклиста сценариев
             в теле. См. правило 10.1 ниже.
         10.1. **Primary acceptance — формат `S<N>.accept`**
             (правило среза 6 + «ФОРМАТ S<N>.accept» в
             `.cursor/rules/vertical-slices.mdc`):
             - Ровно одна `S<N>.accept`; `[x]` = Primary пройден.
             - Первый sub-bullet — **Primary (обязательно):** из
               `**Primary acceptance:**` metadata.
             - Остальные Scenario — «(опционально)» или `S<N>.<M>`.
             - Каждый Scenario spec покрыт Primary, optional, или agent
               `S<N>.<M>` (static verification only).
             - **Запрещено** mandatory programmatic-only в accept.
             - Non-scenario: Assumptions / agent `S<N>.<M>` / Follow-up.
         10.2. **User Task Contract** (`.cursor/rules/vertical-slices.mdc`):
             - От пользователя в `S<N>.<M>`: только ручное конфигурирование.
             - Unknown API/runtime → design § Assumptions + кодовая задача агента.
             - **BAD:** `S<N>.<M>` На тестовой ИБ верифицировать вызов API и
               зафиксировать порядок в debug.md.
             - **OK:** `S<N>.<M>` В `Module.bsl` cf проследить контракт при пустом
               параметре; допущение в design § Assumptions; observable outcome —
               Primary accept «поле пусто после Прервать обработку».
             - **Пример OK:**
               ```
               - [ ] S3.accept Принять срез S3 — автосопоставление входящих:
                 - **Primary (обязательно):** открыть С1 → запустить автосопоставление → строка РС совпадает с эталоном
                 - Scenario «Несколько документов С1 без строки регистра» (опционально): прогнать автосопоставление на пакете ≥2 С1 — одно событие ЖР с перечнем пропусков
               ```
             - **BAD:** все Scenario mandatory без Primary — overload.
             - **BAD:** Primary = «вызвать API в отладчике» — programmatic.
         11. Маркер конца среза сразу после последней задачи:
             `<!-- slice-gate: <критерий приёмки из design> -->`
         12. НЕ использовать: `# Фаза N`, P0–P4, `<!-- phase-gate -->`,
             правило упорядочивания P0→P1→P2→P3→P4.
         13. Группы внутри среза упорядочиваются по зависимости:
             метаданные → форма → код → приёмка. Это внутренний
             порядок, НЕ классификация фаз.
         14. Для каждой задачи с зависимостью от другой — указать
             явно: «Зависимости: S<N>.<M>» (допустимы только задачи
             того же среза или принятых предыдущих срезов).
         15. **Читаемость задачи (Task Readability).** Формулировка
             задачи — по шаблону:
             `<Глагол> <файл/модуль/процедура>: <что> <зачем> [(ссылка на D<N>/OQ<N>/ADR)]`.
             Файл/процедура и бизнес-результат SHALL быть в первых
             12 значимых словах. **ЗАПРЕЩЕНО** начинать задачу с
             «Реализовать инвариант D<N>», «Обеспечить D<N>»,
             «Закрыть OQ<N>», «Выполнить <команда>» без контекста,
             «Обновить» / «Проверить» без объекта. Ссылки на
             инварианты/решения/ADR — **в скобках** после
             формулировки, не в начале. Исключения:
             - `S<N>.accept` — заголовок описывает бизнес-результат
               среза, тело — буллет-чеклист сценариев (по правилу 10.1);
             - prerequisites — явное имя файла/объекта метаданных
               (например, `Выгрузить BusinessProcesses/X.xml`);
             - Follow-up — префикс `Follow-up:` + ссылка на ЗНИ.
             Правило: `.cursor/rules/task-readability.mdc`.

         Используй переданный шаблон для структуры вывода.
         Формат срезов: .cursor/rules/vertical-slices.mdc.
         Читаемость задач: .cursor/rules/task-readability.mdc.

         Результат: полный tasks.md готовый к /opsx:apply.",
  subagent_type="onec-code-architect"
)
```

---

## Architect — fix quality review (debug шаг 5.5)

Used by `/opsx:explore` (профиль bug) or `/opsx:extend --from-report` when Architect Gate triggers fire. The architect evaluates the proposed fix against root cause, UX scenario, alternatives, and band-aid signs. Orchestrator passes RCA (from debug.md or reports), proposed fix plan, and paths to trace/exploration reports.

```
Task(
  description="Ревью качества фикса [краткое описание]",
  prompt="mode=fix-quality
         
         ## Контекст
         Change: [name]. RCA: [путь к debug.md или секция в нём].
         Отчёты: [пути к trace-analysis-*.md, exploration-*.md]

         ## Корневая причина
         [из debug.md: Verified facts + Root cause — цепочка «Почему»]

         ## Предложенный фикс
         [из debug.md: Fix plan — что менять, где, как]

         ## Fix Quality чеклист (ОБЯЗАТЕЛЬНО ответить на каждый пункт)

         1. **Корневая причина vs симптом.** Фикс устраняет корневую причину
            или улучшает поведение неправильного сценария?
         2. **UX-сценарий.** Что видит/делает пользователь ПОСЛЕ фикса?
            Описать полный UX-путь от действия до результата.
            Является ли этот UX-путь целевым или промежуточным?
         3. **Альтернативы.** Есть ли подход с меньшим числом условий/ветвлений?
            Можно ли решить задачу изменением в другой точке?
         4. **Прототип.** Как аналогичный сценарий реализован в прототипе
            (ДО3 Демо / ЭДО ПАО / существующий паттерн в коде)?
         5. **Признаки заплатки.** Обходной путь? Флаг/параметр «для этого
            случая»? Дублирование? Фикс симптома?
         6. **Placement (OpenSpec slices).** Куда попадут задачи фикса в tasks.md
            по `.cursor/rules/vertical-slices.mdc` (**ИНВАРИАНТ: Defect placement**,
            decision tree)? Для целевого среза S<K>: статус `S<K>.T<M>` ([ ] vs [x]);
            фикс cross-slice (≥2 среза) или нет. Подтвердить default=**inside-slice**
            (задачи перед `S<K>.T<M>`) или обосновать исключение (**fix-срез**
            `S<N+1>` только при frozen-slice или cross-slice).

         ## Формат ответа

         ### Fix Quality
         | # | Критерий | OK/WARNING | Обоснование |
         | 1 | Корневая причина vs симптом | ... | ... |
         | 2 | UX-сценарий | ... | ... |
         | 3 | Альтернативы | ... | ... |
         | 4 | Прототип | ... | ... |
         | 5 | Признаки заплатки | ... | ... |
         | 6 | Placement (slices) | ... | ... |

         ### Рекомендация
         ПРИНЯТЬ / ПЕРЕФОРМУЛИРОВАТЬ / ОТКЛОНИТЬ
         [При ПЕРЕФОРМУЛИРОВАТЬ: конкретное предложение по другому подходу]
         [При ОТКЛОНИТЬ: что исследовать дополнительно]

         Project paths: base=[путь из openspec/project.md], extensions=[пути]",
  subagent_type="onec-code-architect"
)
```

---

## Architect — ADR extraction (извлечение архитектурных решений)

Used at archive time when `reports/architecture-*.md` exists and user confirms ADR extraction.

```
Task(
  description="Извлечь ADR из [architecture report]",
  prompt="mode=adr-extraction
         
         Извлеки архитектурные решения из отчёта для создания ADR.

         Отчёт архитектора: [путь к architecture-*.md]
         Design: [путь к design.md]
         Proposal: [путь к proposal.md]

         Задача: из architecture report извлечь РЕШЕНИЯ (не анализ,
         не валидацию, не обследование) и оформить как ADR.

         Критерии ADR-worthy решения:
         - Влияет на несколько changes / будущие задачи
         - Выбран из 2+ альтернатив с неочевидным trade-off
         - Устанавливает контракт, паттерн, архитектурный принцип
         - Ограничивает будущие возможности (trade-off принят осознанно)

         НЕ извлекать как ADR:
         - Точечную валидацию (подтверждение, что точка расширения работает)
         - Решения без альтернатив
         - Чисто тактические решения для одной ЗНИ

         Формат вывода для КАЖДОГО ADR (может быть 0, 1 или несколько):

         ---
         **Заголовок:** [краткое название решения]
         **Область:** [домен/подсистема]

         ## Контекст
         [1-3 абзаца — ситуация и ограничения]

         ## Решение
         [Что решено, какой подход, контракт]

         ## Альтернативы
         | Вариант | Плюсы | Минусы | Почему отклонён |
         |---------|-------|--------|-----------------|
         | ... | ... | ... | ... |

         ## Последствия
         - Положительные: ...
         - Отрицательные: ...

         ## Связи
         - Specs: [если есть]
         - Changes: [имя change]
         ---

         Если в отчёте нет ADR-worthy решений — вернуть:
         'Нет решений, подходящих для ADR. Причина: [объяснение].'",
  subagent_type="onec-code-architect"
)
```

---

## Architect — multisampling arbiter

Used for Critical tasks after 2-3 parallel design runs.

```
Task(
  description="Арбитраж архитектурных решений [feature]",
  prompt="mode=design
         
         Ты выступаешь в роли арбитра. Проведи арбитраж предложенных архитектурных решений.
         
         Решение 1: [путь к architecture-run1.md]
         Решение 2: [путь к architecture-run2.md]
         
         Задача:
         1. Сравни подходы (Alternatives matrix).
         2. Выбери лучший или синтезируй гибридный.
         3. Сформируй итоговый Architecture Document (с YAML front-matter, Tier=Critical).",
  subagent_type="onec-code-architect"
)
```

