# CURRENT — текущее состояние проекта Anvil

## Назначение
Краткий canonical snapshot текущего состояния проекта. Не является историей.

## Текущая фаза
**Research / architecture / development workflow preparation**

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

## Подготовленные направления
Research nodes уже определены для device, boot, runtime, vendor/HAL, graphics, integration, hardware, testing, Device Lab, image tooling, experiment automation, failure analysis, security, packaging и reproducible builds.

## Текущий инфраструктурный приоритет
1. workflow;
2. worklog;
3. knowledge base;
4. source registry;
5. glossary;
6. provenance/experiment discipline.

## Следующий логический шаг
После принятия структуры начать независимые research sessions и переносить подтверждённые результаты в accepted docs.

## Открытые gates
- язык Device Lab Core не выбран;
- формат Golden Snapshot не специфицирован;
- Runtime API не специфицирован;
- production source tree не создан;
- конкретный image strategy первого PoC требует отдельного решения.

## Правило обновления
После существенной итерации проверить изменение accepted state.
После закрытия этапа обновление этого файла обязательно.
