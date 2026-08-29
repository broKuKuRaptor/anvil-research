# RULES — worklog

`worklog/` хранит append-only историю разработки.

## Правила
- один stage = один stage worklog file;
- stage files создаются по факту начала этапа;
- план iteration записывается до изменяющих действий;
- unexpected state фиксируется до продолжения;
- прошлое не переписывается;
- raw private logs в Git не помещаются;
- после stage closure обновляется `INDEX.md`.

## Имена
`stage-00-<short-name>.md`, `stage-01-<short-name>.md`, ...

## Минимальная структура
```text
Goal
Expected outcome
Initial state
Dependencies
Accepted baseline
Completion criteria
Stop conditions
Rollback strategy
Iterations
Stage summary
```

Использовать статусы из `WORKFLOW.md`.
