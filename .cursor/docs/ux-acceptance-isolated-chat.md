# Acceptance: isolated chat (регрессия UX)

**Gate перед merge** правил Chat Surface Contract и verify Repair Loop.

## Зачем изолированный чат

Оркестратор не должен опираться на prior turns («да», контекст extend, якорь брифа). Проверка UX — **только** по первому ответу в **новом** чате без истории.

SSOT контракта: `.cursor/docs/opsx-output-style.md` §2.6.

## Протокол

1. **Подготовка:** change в известном состоянии (fixture или git tag «до repair»).
2. **Новый чат (Composer/Agent):** единственное сообщение — команда under test.
3. **Pass/fail** — только по **первому** ответу оркестратора.

## Сценарии A–F

| ID | Команда | Fixture | Pass |
|----|---------|---------|------|
| **A** | `/opsx:verify <name>` | ZNI с repair-only (текстовые пробелы, нет decision) | **1** сообщение; GO; `**Следующий шаг:**` + `/opsx:apply`; нет «Подтвердить?», `/opsx:extend`, списка файлов |
| **B** | `/opsx:verify <name>` | ZNI с CHALLENGE / A-B | **1** блок вопроса (проблема→последствия→варианты); `**Следующий шаг:**` + «ответьте»; END TURN |
| **C** | `/opsx:verify <name>` | post-GO, артефакты не менялись | ≤6 строк; «статус прежний: можно apply»; `**Следующий шаг:**` + команда по `verify_mode` |
| **D** | `/opsx:new <name>` | новый change | T-CONFIRM; **одна** команда next: verify; без перечня файлов |
| **E** | `/opsx:apply <name>` | завершённый срез | handoff: **Primary acceptance** одной строкой; user-action next step; без meta pipeline |
| **E2** | `/opsx:apply <name>` | кодовая задача без критерия приёмки (mechanical slice) | одна строка «реализована»; нет «пошаговая пауза»; нет diff-чеклиста; нет «автопроверки пройдены» |
| **E3** | `/opsx:apply <name>` | slice-gate handoff | Primary acceptance; ≤7 строк в «Что проверить СЕЙЧАС»; T-HANDOFF §2 без перечня всех задач |
| **F** | `/opsx:explore` | симптом | бриф **B3** карточка (`brief-card.md`: **Симптом**, **Маршрут** или **Варианты**, **Вопрос**, Подтвердить?); финал — блок `## Постановка ЗНИ` или один вопрос |
| **F2** | `/opsx:extend <change>` | чёткое уточнение scope | бриф **B1**: слот **Изменение**; нет `Как буду искать`/`Что понял`; END TURN на «Подтвердить?» |
| **F3** | `/opsx:extend <change>` | расширение с drift-warning | бриф **B2**: **Варианты** нумерованно; `Drift-check` не в чате |

## Сценарии G — verify anti-fatigue (multi-round)

| ID | Шаг | Pass |
|----|-----|------|
| **G1** | Новый чат: `/opsx:verify diadok-mchd-before-pack` после backfill ledger | GO **или** одна развилка; при GO — `**Следующий шаг:**` + user-action команда; **0** agent-keys; нет reopen mount/cleanup |
| **G2** | Ответ на развилку → extend | Handoff на языке эффекта; ledger в debug.md |
| **G3** | Повторный `/opsx:verify` в **той же** сессии | Блок «Уже зафиксировано»; **0** reopen закрытых решений |

**Hard gate:** G1+G3 (или G1 GO без G2). Fixture: `diadok-mchd-before-pack` с `debug.md` § Verify decision ledger.

## Anti-patterns (fail)

- «Как в прошлой сессии» / «вы уже подтвердили»
- Internal-команды с путями (`/opsx:extend --from-verify …`)
- Перечень изменённых файлов как handoff
- Explore-бриф на internal repair
- «Ничего не требуется» + «Подтвердить?»
- GO verify без `**Следующий шаг:**` + user-action команды
- «Что нужно от вас: ничего» на GO без `/opsx:apply` или `/opsx:archive`
- Перегруз bold/backticks; решение требует открыть файл
- «пошаговая пауза» / «Ваш шаг (» в заголовках apply
- Meta-статус pipeline: «автопроверки пройдены», «diff не обязателен», «reviewer PASS»

## Fixtures (рекомендуемые)

| Сценарий | Change / примечание |
|----------|---------------------|
| A, C | `do2-pavlik-predzapolnenie-viz-shablony` — после repair или tag «до repair» |
| B | тот же change с открытым CHALLENGE (контракт схемы КП) |
| D | любой новый запуск new из explore-блока |
| E, E3 | любой change с готовым срезом к приёмке (Primary в metadata) |
| E2 | mechanical slice (`**Режим apply:** mechanical`) или change без `**Приёмка:**` в рабочих задачах |
| F | произвольный симптом ДО2 |
| F2 | `signing-contract-sign-verify-bytes` + уточнение «только облачные УНЭП» |
| F3 | тот же change + запрос локального УНЭП (drift-warning) |
| G1–G3 | `diadok-mchd-before-pack` — ledger backfill в debug.md |
| **H** *(опционально)* | post-apply verify (все `[x]`) — next step = `/opsx:archive`, не apply |

Опционально: `temp/ux-fixtures/README.md` с git tag на commit «до repair».

## Чеклист регрессии (gate)

- [ ] A — repair-only verify в новом чате
- [ ] B — decision verify в новом чате
- [ ] C — silent_ok в новом чате
- [ ] D — new hint verify в новом чате
- [ ] E — apply handoff (Primary)
- [ ] E2 — apply mechanical task (тихий успех)
- [ ] E3 — apply slice-gate handoff
- [ ] F — explore бриф B3 / финал
- [ ] F2 — extend B1 в новом чате
- [ ] F3 — extend B2 drift в новом чате
- [ ] G1/G2/G3 — verify anti-fatigue на `diadok-mchd-before-pack`
