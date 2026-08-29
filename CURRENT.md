# CURRENT — текущее состояние проекта Anvil

## Назначение
Краткий canonical snapshot текущего состояния проекта. Не является историей.

## Текущая фаза
**Research / architecture / development workflow established**

Production source code пока отсутствует.

## Accepted architecture baseline
- Host: Debian.
- Android используется как runtime/hardware compatibility layer.
- Первый container backend: LXC.
- Hardware backends: Android BSP / Native Linux / Hybrid.
- Первый PoC должен максимально сохранять known-good Android kernel/vendor foundation.
- Для устройства предусматривается Golden Android / Device Snapshot.
- Runtime не должен зависеть от конкретной mobile shell.
- Device experimentation планируется автоматизировать через Anvil Device Lab.
- MCP — frontend к Device Lab Core, а не место основной orchestration logic.

## Development workflow baseline
- `WORKFLOW.md` принят как нормативный документ разработки.
- История этапов ведётся в `worklog/` по принципу one stage = one file.
- Повторно применимые подтверждённые знания сохраняются в `knowledge/`.
- Provenance внешних inputs ведётся в `sources/`.
- Канонические термины фиксируются в `GLOSSARY.md`.
- Project-owned Markdown ведётся на русском языке.
- Commit messages ведутся на английском языке и для существенных изменений должны позволять восстановить контекст.

## Подготовленные направления
Research nodes определены для:

- device capture;
- Debian boot;
- device profile;
- image tooling;
- runtime manager;
- Android container core;
- graphics/display/input;
- vendor/HAL integration;
- Android image/framework strategy;
- networking;
- application management;
- desktop integration;
- diagnostics/testing;
- Device Lab;
- experiment automation / provenance / HIL CI;
- failure analysis / regression;
- audio;
- camera/media;
- telephony/modem;
- sensors/GNSS;
- power/thermal/battery;
- wireless;
- security;
- packaging/images/updates;
- reproducible builds.

## Текущий приоритет
Перейти от подготовки структуры к focused research sessions по наиболее критичным узлам первого PoC и development infrastructure.

В первую очередь:

1. Device Capture / Golden Snapshot;
2. Device Profile;
3. Debian boot;
4. Image Tooling;
5. Android Container Core;
6. Vendor / HAL Integration;
7. Android Image / Framework Strategy;
8. Diagnostics / Device Lab.

## Следующий логический шаг
Начать приоритетные research nodes и последовательно переносить подтверждённые результаты в accepted project docs.

По мере появления первых реальных build/device experiments подключать:

- Experiment Automation;
- Failure Analysis / Regression;
- Reproducible Builds.

## Открытые gates
- язык Device Lab Core не выбран;
- формат Golden Snapshot не специфицирован;
- Runtime API не специфицирован;
- production source tree не создан;
- конкретный image strategy первого PoC требует отдельного решения;
- ownership сложных hardware subsystems в hybrid backend остаётся открытым до соответствующих исследований.

## Не считать принятым без отдельного решения
- конкретный Android framework image первого PoC;
- окончательный modem/camera/audio ownership;
- окончательный security model;
- окончательный image/update format;
- язык core-компонентов.

## Правило обновления
После существенной итерации проверить изменение accepted state.
После закрытия этапа обновление этого файла обязательно.
