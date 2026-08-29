# 42 — Experiment Automation / Provenance / HIL CI

## Статус

**Research task prepared**

Этот узел объединяет автоматизацию физических экспериментов, provenance артефактов, Hardware-in-the-Loop CI, автоматический bisect и декларативное описание test scenarios.

## Назначение компонента

После появления Anvil Device Lab возникает следующий уровень автоматизации:

```text
source commit
    ↓
build
    ↓
artifact
    ↓
physical device
    ↓
boot / flash
    ↓
test
    ↓
collect
    ↓
classify
    ↓
rollback
```

Цель — превратить физический телефон из ручного отладочного объекта в программируемый экспериментальный стенд.

## Основная идея

Каждый эксперимент должен быть воспроизводим и связан с:

```text
git commit
build configuration
toolchain
device profile
image hashes
target slot
Device Lab workflow
test results
logs
rollback result
```

Пример experiment identity:

```text
exp-0042
├── source commit
├── build manifest
├── boot.img SHA256
├── device profile version
├── tested slot
├── test scenario
├── result
└── artifact bundle
```

## Подзадачи

### Experiment Provenance

Определить, как связать build artifacts, исходники и физический test run.

### Hardware-in-the-Loop CI

Исследовать pipeline:

```text
git push / PR
    ↓
build
    ↓
Device Lab
    ↓
physical phone
    ↓
smoke tests
    ↓
report
    ↓
rollback
```

### Automatic Bisect

Автоматизировать:

```text
git bisect
  ↓
build candidate
  ↓
boot candidate
  ↓
GOOD / BAD
  ↓
repeat
```

### Experiment Specification DSL

Исследовать декларативное описание сценария:

```yaml
name: boot-anvil-linux

prepare:
  state: bootloader
  slot: inactive

flash:
  boot: artifacts/boot.img

boot:
  timeout: 90

expect:
  transport: ssh

collect:
  - dmesg
  - journal

on_failure:
  enter: recovery
  collect:
    - pstore

finally:
  rollback: true
```

## Задание на исследование

1. Изучить существующие Hardware-in-the-Loop CI systems.
2. Найти device farm / flashing automation проекты с self-healing.
3. Проанализировать provenance подходы:
   - build manifests;
   - artifact attestations;
   - SBOM;
   - reproducible builds metadata.
4. Спроектировать experiment ID и artifact relationships.
5. Определить immutable experiment record.
6. Спроектировать Experiment Specification DSL.
7. Определить workflow execution model поверх Device Lab.
8. Спроектировать automatic retry / timeout / recovery policy.
9. Определить automatic git bisect workflow.
10. Спроектировать HIL CI integration.
11. Определить concurrency и device reservation model.
12. Определить lifecycle артефактов и retention policy.
13. Сформировать MVP и поэтапный implementation roadmap.

## Что должно появиться после проработки

- обзор HIL CI и device farm решений;
- experiment provenance model;
- experiment directory/schema;
- experiment DSL;
- workflow state model;
- automatic bisect design;
- HIL CI architecture;
- device reservation model;
- artifact retention policy;
- retry/recovery policy;
- PoC plan;
- implementation roadmap;
- источники.

## Зависимости

Критические:

- `41_device_lab_mcp.md`
- `40_diagnostics_testing.md`

Использует:

- `13_image_tooling.md`
- `12_device_profile.md`

Связан с:

- `43_failure_analysis_regression.md`
- `62_reproducible_builds.md`

## Обязательная структура итогового документа

```text
1. Purpose
2. Existing HIL/device farm solutions
3. Provenance requirements
4. Experiment identity/model
5. Artifact model
6. Experiment DSL
7. Workflow engine integration
8. Retry/recovery
9. HIL CI
10. Automatic bisect
11. Device reservation/concurrency
12. Retention
13. PoC
14. Implementation roadmap
15. Risks
16. Sources
```
