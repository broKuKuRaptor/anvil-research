# WORKFLOW — правила ведения разработки Anvil

## 1. Назначение

Этот документ является нормативным описанием процесса разработки Anvil. Он определяет правила планирования, исследований, экспериментов, rollback, ведения worklog, базы знаний, реестра источников, документации и commit history.

`WORKFLOW.md` отвечает на вопрос **как ведётся работа**.
`CURRENT.md` фиксирует **текущее accepted состояние**.
`worklog/` хранит **историю**.
`knowledge/` хранит **повторно применимые подтверждённые знания**.
`sources/` хранит **provenance источников и артефактов**.

## 2. Основные принципы

### 2.1. Plan before action
До любой нетривиальной итерации зафиксировать:
- цель;
- гипотезу;
- исходное состояние;
- зависимости;
- входные артефакты;
- planned changes;
- порядок действий;
- критерии успеха;
- stop conditions;
- риски;
- rollback.

### 2.2. Одна итерация — одна основная гипотеза
Изменять минимальное число независимых переменных. Если изоляция невозможна, это явно фиксируется.

### 2.3. Сначала read-only/static, потом изменение
Предпочтительный порядок:
```text
existing knowledge
→ local evidence
→ external research
→ static analysis
→ artifact validation
→ read-only preflight
→ controlled change
→ measurement
```

### 2.4. Evidence before conclusion
Разделять:
- `Observed` — непосредственно измерено;
- `Inferred` — логически выведено;
- `Unverified` — требует проверки;
- `Accepted` — принято после достаточной проверки.

### 2.5. Отсутствие наблюдения не доказывает отсутствие компонента
Учитывать права, namespaces, lifecycle state и ограничения инструментов.

### 2.6. Stop on unexpected state
Если предпосылки плана нарушены:
```text
STOP
→ record
→ preserve evidence
→ diagnose
→ revise hypothesis
→ revise plan
→ continue
```
Не продолжать destructive sequence по инерции.

### 2.7. Research before guessing
Если локальные evidence не дают достаточно сильной следующей гипотезы, до новой изменяющей итерации провести внешний поиск:
1. official docs;
2. upstream source;
3. похожие реализации;
4. issue trackers;
5. patches/commits;
6. device-specific reports;
7. community reports.

Для найденного решения оценивать:
- что оно решает;
- target environment;
- assumptions;
- applicability;
- требуемую адаптацию;
- риски;
- степень доверия.

### 2.8. Rollback — часть эксперимента
Для device-changing iteration:
```text
accepted baseline
→ test change
→ measurement
→ evidence
→ rollback
→ baseline verification
```
Успешная команда rollback без проверки baseline недостаточна.

### 2.9. Provenance обязателен
Для repositories, tools, patches, images и значимых источников фиксировать origin/version/commit/hash/status.

### 2.10. Tool output — evidence, а не истина
Учитывать assumptions, target platform, версию и false positives.

### 2.11. История append-only
Не переписывать прошлые действия так, будто новая гипотеза существовала заранее. Исправления добавлять отдельной correction/superseding записью.

## 3. Иерархия доверия
1. runtime capture текущего устройства;
2. exact artifacts принятого baseline;
3. official upstream source нужной версии;
4. official documentation нужной версии;
5. verified reference implementation;
6. historical device-specific implementation;
7. maintainer/issue discussion;
8. community report;
9. unverified local artifact.

Статусы source registry:
`PRIMARY`, `VERIFIED`, `REFERENCE`, `HISTORICAL`, `COMMUNITY`, `UNVERIFIED`, `REJECTED`.

## 4. Project state

### CURRENT.md
Агент читает его после `WORKFLOW.md` в начале рабочей сессии. Он содержит только актуальное состояние, не историю.

### worklog/
Один этап разработки = один stage worklog file. Файлы создаются по факту начала этапа.

### knowledge/
Только подтверждённые reusable решения, методы, failure patterns, tool limitations и workarounds.

### sources/
Реестр происхождения внешних источников и binary artifacts.

## 5. Жизненный цикл этапа
Stage должен содержать:
- цель;
- expected outcome;
- initial state;
- dependencies;
- accepted baseline;
- completion criteria;
- stop conditions;
- rollback strategy;
- iterations;
- stage summary.

## 6. Жизненный цикл итерации
```text
PLAN
→ PREFLIGHT
→ EXECUTE
→ OBSERVE
→ PRESERVE EVIDENCE
→ ANALYZE
→ EXTERNAL RESEARCH if needed
→ CONCLUDE
→ ROLLBACK if applicable
→ VERIFY BASELINE
→ UPDATE DOCUMENTATION
→ UPDATE NEXT PLAN
→ UPDATE KNOWLEDGE
→ UPDATE SOURCES
→ CLOSE
```

## 7. Шаблон итерации
```markdown
### Итерация N.M. Название
Статус: **PLANNED**

#### Назначение
#### Гипотеза
#### Исходное состояние
#### Входные данные и зависимости
#### Планируемые изменения
#### Подробный план действий
#### Критерии успеха
#### Условия остановки
#### Риски
#### Rollback
#### Preflight
#### Выполненные действия
#### Наблюдения
#### Отклонения и непредвиденные ситуации
#### Evidence / artifacts
#### Внешнее исследование
#### Анализ
#### Принятые выводы
#### Оставшиеся неизвестные
#### Влияние на документацию
#### Влияние на knowledge base
#### Следующая итерация
```

## 8. Статусы
`PLANNED`, `RUNNING`, `SUCCESS`, `PARTIAL`, `FAILED`, `BLOCKED`, `CANCELLED`, `SUPERSEDED`.

## 9. Planned changes
Перед build/device experiment явно указать ожидаемый delta, например:
```text
kernel: changed
ramdisk: unchanged
vendor: unchanged
dtbo: unchanged
vbmeta: unchanged
slot: boot_b only
userdata: read-only
```
Actual delta после теста сопоставляется с planned delta.

## 10. Preflight Gate
До изменения физического устройства проверить:
- identity;
- transport/state;
- slot;
- bootloader state;
- partition size при необходимости;
- hashes;
- rollback artifact;
- recovery/bootloader availability;
- питание;
- однозначность target partition.

## 11. Artifact Gate
Проверять применимое:
- SHA-256;
- size;
- image type;
- boot header;
- architecture;
- partition/slot compatibility;
- AVB;
- DTB/DTBO;
- filesystem consistency;
- expected semantic delta.

## 12. Классы риска операций
`READ_ONLY`, `TEST_BOOT`, `FLASH_SLOT`, `DESTRUCTIVE`, `BOOTLOADER_CRITICAL`.

`BOOTLOADER_CRITICAL` по умолчанию запрещён без отдельного решения.

## 13. Device experiment
Перед:
1. worklog plan;
2. checkpoint commit при существенном риске;
3. artifact gate;
4. device preflight;
5. rollback;
6. stop conditions.

Во время:
- не расширять scope скрытно;
- фиксировать state transitions;
- сохранять timeline/stdout/stderr;
- останавливаться при unexpected state;
- не уничтожать forensic evidence повторным boot без необходимости.

После:
- evidence collection;
- rollback;
- baseline verification;
- анализ.

## 14. Внешнее исследование
Если root cause неясен, research record должен фиксировать:
- Problem;
- Observed symptoms;
- Search terms;
- Solutions found;
- Target environment;
- Applicability;
- Conflicts/risks;
- Selected hypothesis;
- Rejected alternatives;
- Sources.

## 15. Актуализация после итерации
Итерация административно не закрыта, пока проверено влияние на:
- `CURRENT.md`;
- stage worklog;
- architecture/docs;
- related research;
- test docs;
- планы зависимых итераций;
- `knowledge/`;
- `sources/`.

Если следующая итерация зависит от изменившейся предпосылки, её план обновляется до выполнения.

## 16. Закрытие этапа
Checklist:
- [ ] completion criteria reviewed
- [ ] accepted conclusions recorded
- [ ] remaining unknowns recorded
- [ ] accepted limitations recorded
- [ ] artifacts listed
- [ ] CURRENT.md updated
- [ ] ARCHITECTURE.md reviewed
- [ ] ROADMAP.md reviewed
- [ ] DEPENDENCIES.md reviewed
- [ ] RISKS.md reviewed
- [ ] GLOSSARY.md reviewed
- [ ] knowledge/ reviewed
- [ ] sources/ updated
- [ ] dependent next-stage plans reviewed
- [ ] final stage checkpoint commit created

`reviewed` не означает обязательное изменение.

## 17. Knowledge management
Переносить только reusable подтверждённые знания:
- solution;
- diagnostic technique;
- failure signature;
- tool limitation;
- verified workaround;
- compatibility rule.

Knowledge entry должна содержать:
```text
Problem
Symptoms
Root cause
Confirmed environment
Solution
Why it works
Limitations
Related experiments
Sources
```

## 18. Source registry
При первом существенном использовании нового источника добавить его в:
- `sources/documentation.md`
- `sources/repositories.md`
- `sources/tools.md`
- `sources/patches.md`
- `sources/images.md`

## 19. Язык проекта
Все project-owned `.md` файлы — на русском языке.
Технические англицизмы допустимы.
Commit messages — на английском языке.

## 20. Терминологическая стабильность
Использовать canonical terms из `GLOSSARY.md`.
При переименовании термина обновлять активные docs, historical worklog не переписывать.

## 21. Raw evidence и приватность
```text
raw evidence → local ignored storage
sanitized conclusions → Git
```
Не коммитить без необходимости serial, IMEI/IMSI, MAC/BSSID/SSID, IP, аккаунты, номера телефонов, tokens и private filesystem data.

## 22. Commit messages
Для значимых commits:
```text
<type>: short summary

Context paragraph.

Changes:
- ...

Validation:
- ...

Known limitations:
- ...

Next:
- ...
```
Commit должен помогать восстановить контекст: why / what / validation / limitations / next.

## 23. Checkpoint commits
Рекомендуются:
- перед destructive operation;
- перед architecture transition;
- перед baseline change;
- перед рискованным device experiment;
- после stage closure.

## 24. Reproducibility
Различать:
- `Byte reproducible`;
- `Semantically reproducible`.

Допустимые nondeterministic differences должны быть известны заранее.

## 25. Baseline
Accepted baseline должен иметь:
- known source state;
- hashes;
- known device state;
- tested functionality;
- known limitations;
- rollback path.

## 26. Что читает агент в начале сессии
```text
1. WORKFLOW.md
2. CURRENT.md
3. GLOSSARY.md при необходимости
4. worklog/INDEX.md
5. текущий stage worklog
6. relevant knowledge/*
7. relevant sources/*
8. related docs/research nodes
```

## 27. Перед новой итерацией агент обязан
1. определить accepted baseline;
2. проверить предыдущий result;
3. убедиться, что документация актуализирована;
4. проверить assumptions;
5. сформулировать одну основную hypothesis;
6. определить минимальный experiment;
7. сформировать stop/rollback conditions;
8. записать план до изменяющих действий.

## 28. После итерации агент обязан
1. сохранить evidence;
2. отделить facts от interpretation;
3. оценить criteria;
4. провести external research, если причина неясна;
5. записать accepted conclusions;
6. rollback/verify baseline, если применимо;
7. обновить docs;
8. пересмотреть зависимый plan;
9. обновить CURRENT.md;
10. добавить reusable knowledge;
11. зарегистрировать sources;
12. закрыть статус.

## 29. Запрещённая скрытая импровизация
Агент не должен:
- делать дополнительные flash/erase/set_active после неожиданной ошибки без нового анализа;
- применять patch без applicability analysis;
- менять несколько независимых подсистем «для ускорения»;
- считать визуальный симптом root cause;
- скрывать limitations;
- выдавать tool failure за device failure;
- автоматически исправлять все warnings checker;
- переписывать historical worklog.

## 30. Минимально разрушительный тест
Предпочитать:
- temporary boot перед permanent flash;
- inactive slot перед active, если это валидно;
- read-only recovery mounts перед read-write.

## 31. Forensic evidence
До повторного boot учитывать потерю:
- pstore;
- volatile journal;
- last_kmsg;
- temporary logs;
- USB state evidence.

## 32. Связь с будущей автоматизацией
```text
Image Tooling → Artifact Gate
Device Lab → state/preflight/flash/rollback
Experiment Automation → lifecycle
Failure Analysis → triage
Reproducible Builds → provenance
```

## 33. Минимальная бюрократия
Не дублировать информацию.
- `CURRENT.md` — состояние.
- `worklog/` — история.
- `knowledge/` — reusable knowledge.
- `sources/` — provenance.
- `docs/` — accepted technical documentation.
- `research_nodes/` — открытые исследования.

## 34. Главное правило
> Изменяющее действие должно быть объяснимо до выполнения, наблюдаемо во время выполнения, проверяемо после выполнения и обратимо там, где это технически возможно.
