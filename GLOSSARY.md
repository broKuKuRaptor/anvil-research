# GLOSSARY — глоссарий Anvil

## Назначение
Канонические термины проекта. Project-owned `.md` документы ведутся на русском, но привычные технические термины могут оставаться английскими.

## Anvil
Android Hardware Runtime for Linux — проект Linux-host среды, где Android используется как runtime и hardware compatibility layer.

## Host
Основная Linux userspace environment. Для Anvil baseline — Debian host.

## Runtime
Управляемая среда выполнения. В Anvil обычно означает Android Runtime.

## Android Runtime
Android userspace environment с Framework/services/apps и доступом к vendor stack. Не путать с ART.

## Android Framework
System services, framework APIs, package/application lifecycle и связанный Android userspace слой.

## Android BSP
Downstream kernel, drivers, firmware, HAL, vendor services и device-specific configuration конкретного Android device.

## Android BSP backend
Backend, использующий Android BSP/vendor stack для hardware access.

## Native Linux backend
Backend на стандартных Linux drivers/APIs без Android vendor HAL.

## Hybrid backend
Часть hardware обслуживается native Linux, часть Android vendor/HAL stack.

## Backend
Конкретный механизм реализации runtime/hardware interface.

## Golden Android
Заведомо рабочая Android/LineageOS конфигурация устройства, используемая как эталон.

## Golden Snapshot
Device Snapshot, снятый с Golden Android и принятый как known-good reference.

## Device Snapshot
Набор images, metadata и runtime capture конкретного состояния устройства.

## Device Capture
Процесс получения Device Snapshot.

## Device Profile
Структурированное описание partition layout, slots, boot format, USB IDs, recovery strategy, firmware requirements и safe/forbidden operations устройства.

## Accepted baseline
Проверенное состояние, принятое как рабочая точка дальнейших экспериментов.

## Candidate baseline
Кандидат на baseline, ещё не прошедший acceptance gates.

## Baseline
По умолчанию означает accepted baseline; иначе использовать `candidate baseline`.

## Artifact
Файл или набор файлов — вход/результат build или experiment.

## Artifact Gate
Проверки artifact перед использованием: checksum, size, format, compatibility, AVB и semantic delta.

## Evidence
Данные, на основании которых делается инженерный вывод: logs, hashes, properties, service lists, pstore и т.д.

## Observed
Непосредственно измеренный факт.

## Inferred
Логически выведенный результат, не измеренный напрямую.

## Unverified
Гипотеза, требующая проверки.

## Accepted
Вывод, прошедший достаточную проверку и принятый проектом.

## Experiment
Контролируемая проверка hypothesis с известными inputs, planned changes, criteria, stop conditions и evidence.

## Experiment Bundle
Набор артефактов одного experiment run.

## Experiment ID
Уникальный идентификатор experiment run.

## Iteration
Минимальная управляемая единица инженерной работы; обычно проверяет одну основную hypothesis.

## Stage
Крупный этап, содержащий несколько iterations.

## Planned changes
Явный ожидаемый delta по subsystems/artifacts в рамках iteration.

## Stop condition
Условие немедленной остановки текущей последовательности до анализа.

## Preflight
Проверка состояния непосредственно перед experiment.

## Rollback
Возврат к known-good state с последующей baseline verification.

## Checkpoint commit
Git commit перед рискованным изменением или важной границей этапа.

## Provenance
Прослеживаемое происхождение source/artifact/tool и exact identifiers.

## Source Registry
Каталог `sources/` с provenance документации, repositories, tools, patches и images.

## Knowledge Base
Каталог `knowledge/` с подтверждёнными reusable знаниями.

## Capability Snapshot
Структурированный снимок services, HAL, devices, mounts, firmware, properties и других capabilities.

## Capability diff
Структурированное сравнение двух Capability Snapshot.

## Regression
Утрата ранее подтверждённой функциональности.

## Failure signature
Повторяемый набор симптомов/evidence, характерный для типа отказа.

## Failure triage
Первичная классификация и локализация проблемы.

## PoC
Proof of Concept — минимальная проверка жизнеспособности решения.

## Device Lab
Сокращение от Anvil Device Lab.

## Anvil Device Lab
Automation subsystem для ADB/Fastboot/recovery, states, slots, flashing, diagnostics, rollback и artifacts.

## HIL CI
Hardware-in-the-Loop CI — автоматическое тестирование builds на физическом устройстве.

## Image
Binary filesystem/partition/boot artifact.

## Boot image
Android boot partition image с kernel/ramdisk/metadata в зависимости от поколения boot format.

## Slot
Один из A/B наборов разделов, обычно `a` или `b`.

## Inactive slot
Slot, не выбранный для текущего normal boot.

## Bootloader
Низкоуровневый загрузчик устройства.

## Fastboot
Protocol/tool bootloader management и partition flashing.

## Fastbootd
Userspace fastboot environment. Не путать с bootloader fastboot.

## Recovery
Отдельная boot environment обслуживания и восстановления.

## Destructive operation
Операция, способная существенно или необратимо изменить устройство/данные.

## Golden-state protection
Правила/automation для сохранения known-good state при экспериментах.

## Binder
Android IPC stack.

## BinderFS
Filesystem для динамического создания Binder devices; не обязателен для старых kernels с classic binder nodes.

## HAL
Android Hardware Abstraction Layer.

## HIDL
Наследуемый Android HAL interface model.

## AIDL
Android Interface Definition Language, включая stable HAL interfaces.

## Vendor
Уточнять по контексту: vendor partition, vendor services или vendor implementation.

## Vendor stack
Vendor partition/services/HAL и связанные device-specific Android components.

## VINTF
Android mechanism описания/проверки framework/vendor compatibility.

## Treble
Архитектурное разделение Android framework и vendor interfaces.

## AVB
Android Verified Boot.

## APEX
Android modular system component format/runtime mechanism.

## LXC
Linux Containers — выбранный первый system-container backend Anvil.

## System container
Container для полноценной userspace OS environment с init/services.

## cgroup
Linux control groups.

## cgroup v2
Unified cgroup hierarchy.

## Wayland
Основной Linux display protocol для host graphics integration.

## PipeWire
Основной Linux multimedia routing layer для host integration.

## dma-buf
Linux shared buffer mechanism, важный для graphics/media zero-copy.

## Zero-copy
Data path без лишнего копирования между memory regions.

## Reproducible build
Build, воспроизводимый из зафиксированных sources/toolchain/configuration.

## Byte reproducible
Повторные builds дают byte-identical artifacts.

## Semantically reproducible
Допустимы заранее известные metadata differences, но значимое содержимое идентично.
