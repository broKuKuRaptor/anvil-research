# 44 — Virtual Test Bed / VM Pre-flight

## Статус

**Initial research completed / Research task prepared**

Этот узел исследует, какую часть разработки и отладки Anvil можно перенести с физического телефона на виртуальный стенд, где заканчивается полезность виртуализации и как встроить VM-testing в общий Experiment Automation / HIL workflow.

---

# 1. Цель

Снизить количество физических flash/boot cycles и использовать реальное устройство только для тех проверок, которые действительно зависят от Android BSP, vendor services, firmware или аппаратуры.

Главная идея:

```text
source change
    ↓
static checks
    ↓
Linux VM / container tests
    ↓
ARM64 virtual tests
    ↓
Android virtual-device tests
    ↓
physical Device Lab / HIL
```

Каждый следующий уровень должен запускаться только если предыдущий прошёл или если конкретный test scenario принципиально требует более высокого уровня.

---

# 2. Вывод исследования

Виртуальный стенд для Anvil не только возможен, но и архитектурно желателен.

При этом не рекомендуется ставить целью полную эмуляцию конкретного телефона и его Android BSP.

Причина: QEMU хорошо эмулирует стандартные виртуальные платформы и CPU architectures, но device-specific Android stack зависит от большого количества компонентов, которые не представлены в generic virtual hardware:

- downstream kernel drivers;
- vendor firmware;
- GPU/DSP/NPU;
- modem;
- camera ISP;
- secure world / TEE;
- power/thermal hardware;
- fingerprint;
- vendor-specific shared memory и IPC;
- bootloader-specific behavior.

Эмуляция такого телефона быстро превращается в самостоятельный проект по моделированию SoC и периферии.

Поэтому рекомендуемая модель — **layered virtual pre-flight**, а не «виртуальный OnePlus/Pixel».

---

# 3. Что можно тестировать без реального hardware

## 3.1. Debian host

Можно полноценно тестировать:

- systemd units;
- service ordering;
- udev rules, если используются virtual fixtures;
- filesystem layout;
- mounts;
- namespaces;
- cgroups;
- networking;
- D-Bus interfaces;
- PipeWire integration на уровне userspace;
- logging;
- package install/update;
- Runtime Manager.

Для этой категории preferred backend — обычная Linux VM с KVM на той же architecture, что и host CI.

---

## 3.2. LXC / container lifecycle

В VM можно проверять:

- создание/запуск/остановку container;
- mount namespace;
- bind mounts;
- device exposure policy;
- cgroup configuration;
- environment/property injection;
- container filesystem layout;
- host/runtime IPC;
- failure handling;
- restart/recovery logic.

Отдельно необходимо учитывать, что реальный Android downstream kernel может отличаться от kernel VM по Binder, cgroups, namespaces, SELinux и другим возможностям.

Поэтому успешный VM test не доказывает compatibility с target device kernel.

---

## 3.3. Binder

Современный Linux kernel поддерживает `binderfs`, позволяющий создавать независимые наборы Binder devices в отдельных binderfs instances.

Это делает VM подходящей для тестирования:

- Binder device setup;
- Binder namespace strategy;
- Runtime Manager integration;
- access/ownership policy;
- многократного запуска независимых Android environments.

Важно:

> binderfs в VM проверяет Anvil userspace/container architecture, но не является доказательством того, что конкретный старый Android kernel target device предоставляет те же возможности.

Для старых downstream kernels может потребоваться classic `/dev/binder`, `/dev/hwbinder`, `/dev/vndbinder` model.

---

## 3.4. Android userspace и framework logic

В зависимости от image и kernel support можно тестировать:

- init sequencing;
- property service;
- linker/linkerconfig;
- APEX mount logic;
- zygote;
- system_server;
- PackageManager;
- ActivityManager;
- APK lifecycle;
- parts of framework services;
- Binder-facing Anvil integration.

Но Android framework boot без original vendor environment может потребовать:

- virtual HAL implementations;
- disabled/optional services;
- mock properties;
- synthetic VINTF;
- virtual devices.

Поэтому этот test bed должен иметь отдельный profile и не притворяться exact copy Golden Android.

---

# 4. QEMU/KVM как Linux VM backend

QEMU system emulation предоставляет generic machine model и поддерживает KVM acceleration на Linux для совпадающей host/guest architecture.

Рекомендуемый быстрый backend:

```text
x86_64 Linux host
       │
      KVM
       │
 Debian x86_64 VM
       │
 Anvil host services
       │
      LXC
       │
 test Android/runtime fixtures
```

Преимущества:

- высокая скорость;
- дешёвый CI;
- snapshot/reset;
- deterministic storage/network;
- легко запускать несколько экземпляров;
- подходит для unit/integration/system tests Anvil userspace.

Ограничение:

- x86_64 VM не проверяет ARM64 ABI и ARM-specific binaries.

---

# 5. ARM64 QEMU

QEMU TCG умеет system-emulate AArch64 на x86_64 host.

Generic `virt` machine предоставляет стандартную ARM virtual platform, а не модель Android phone SoC.

Рекомендуемая роль:

```text
x86_64 CI host
      │
 QEMU TCG
      │
ARM64 virt machine
      │
Debian arm64
      │
Anvil + ARM64 Android userspace tests
```

Подходит для:

- ARM64 ELF/linker checks;
- ARM64 runtime binaries;
- architecture-specific packaging;
- Android arm64 userspace;
- page-size assumptions;
- bootstrapping generic ARM64 kernels/rootfs;
- cross-build validation.

Минусы:

- существенно медленнее KVM when host architecture differs;
- не моделирует target SoC;
- не воспроизводит vendor drivers/firmware.

Если CI host сам ARM64, ARM64 guest можно запускать через KVM и получить намного более быстрый стенд.

---

# 6. Android Emulator

Android Virtual Device / Emulator полезен прежде всего как Android application/framework reference environment.

Он позволяет:

- запускать AOSP system images;
- использовать `adb`;
- использовать root с AOSP images;
- тестировать apps и framework behavior;
- автоматизировать instrumented tests.

Но он не воспроизводит vendor stack целевого устройства.

Для Anvil Android Emulator рекомендуется использовать как:

```text
reference Android framework target
```

а не как основной Anvil host/runtime emulator.

---

# 7. Cuttlefish

Cuttlefish — наиболее интересный Android virtual-device backend для Anvil research.

AOSP использует Cuttlefish как virtual Android device для platform development. Доступны x86_64 и ARM64 targets, а Android documentation показывает возможность запуска custom kernels и проверки полного Android boot до `sys.boot_completed = 1`.

Потенциально полезные области:

- Android Framework boot;
- APEX;
- Binder/AIDL/VINTF;
- framework modifications;
- generic Android Runtime experiments;
- app lifecycle;
- virtual display/input;
- custom kernel testing;
- compatibility experiments с будущими Android releases.

Особенно важно, что AOSP сам использует Cuttlefish как development target для новых framework/VINTF requirements.

Однако Cuttlefish имеет собственный virtual hardware/vendor model.

Поэтому:

> успешная загрузка Android Runtime в Cuttlefish не доказывает compatibility с original vendor stack target phone.

Cuttlefish полезен для generic Android side Anvil, а Golden Device + physical HIL остаётся source of truth для BSP compatibility.

---

# 8. Virtual HAL и mock backends

Для отделения Anvil core от hardware-dependent слоя рекомендуется определить test interfaces.

Пример:

```text
Anvil service
     │
hardware/backend interface
     │
 ┌───┴───────────────┐
 │                   │
Mock backend     Android HAL backend
 │                   │
VM tests         physical device
```

Кандидаты для mock backends:

- sensors;
- GNSS;
- modem state;
- battery/charger;
- audio routing;
- camera frames;
- display events;
- input events;
- network state.

Mock должен моделировать не только happy path, но и failure states:

- service unavailable;
- timeout;
- reconnect;
- invalid data;
- device disappearance;
- permission denied;
- delayed initialization.

---

# 9. Golden Snapshot replay

Golden Device Capture может использоваться не только как архив baseline, но и как источник test fixtures.

Из snapshot можно сформировать sanitized machine-readable fixtures:

```text
fixtures/<device>/<snapshot>/
├── properties.json
├── vintf/
├── binder-services.json
├── hal-services.json
├── mounts.json
├── devices.json
├── sysfs.json
├── firmware.json
├── kernel-features.json
└── capabilities.json
```

VM tests могут использовать эти данные для проверки assumptions Anvil без подключения телефона.

Примеры:

- проверка, что Runtime Manager ожидает реально существующие services;
- проверка VINTF names/versions;
- validation mount plans;
- сравнение expected devices;
- генерация mock HAL/service topology;
- regression test against previous Golden Snapshot.

Критическое ограничение:

> snapshot replay моделирует наблюдаемое состояние, но не воспроизводит поведение hardware/driver во времени.

Это fixture/replay mechanism, а не hardware emulator.

---

# 10. Предлагаемая иерархия test levels

## Level 0 — Static

Не требует VM.

Проверять:

- config/schema;
- image format;
- partition compatibility;
- VINTF parsing;
- ELF dependencies;
- systemd/LXC config;
- source formatting/lint;
- package metadata;
- AVB/image structure;
- Device Profile consistency.

Стоимость: минимальная.

---

## Level 1 — Native Architecture VM

Предпочтительно KVM.

Проверять:

- Debian host services;
- systemd;
- networking;
- D-Bus;
- LXC lifecycle;
- cgroups/namespaces;
- Binder setup на современном kernel;
- Anvil APIs;
- failure/restart logic;
- Experiment Automation.

Стоимость: низкая.

---

## Level 2 — ARM64 Generic VM

QEMU TCG на x86_64 или KVM на ARM64 host.

Проверять:

- ARM64 binaries;
- Android ARM64 userspace;
- linker/APEX assumptions;
- architecture-specific packaging;
- generic ARM64 boot environment.

Стоимость: средняя/высокая при TCG.

---

## Level 2A — Android Virtual Device

Cuttlefish или Android Emulator.

Проверять:

- generic Android framework;
- Binder/VINTF/APEX;
- package/application lifecycle;
- virtual display/input;
- framework modifications;
- future generic/newer Android strategy.

Это не замена Level 2, а параллельный Android-focused backend.

---

## Level 3 — Physical Device / HIL

Anvil Device Lab.

Обязательно для:

- target downstream kernel;
- real boot chain;
- AVB/slot behavior;
- vendor services;
- HAL compatibility;
- firmware;
- modem;
- GPU;
- camera;
- sensors;
- GNSS;
- Wi-Fi/Bluetooth hardware;
- power/suspend/resume;
- charging;
- thermal;
- fingerprint;
- TEE/secure services;
- real display stack.

Стоимость и риск: максимальные.

---

# 11. Test promotion policy

Рекомендуемый pipeline:

```text
commit
  │
  ▼
L0 static
  │ PASS
  ▼
L1 native VM
  │ PASS
  ├───────────────┐
  ▼               ▼
L2 ARM64       L2A Cuttlefish
  │               │
  └───────┬───────┘
          │ PASS / relevant
          ▼
     L3 Device Lab
```

Не каждый commit должен проходить Level 3.

Device Lab запускается если:

- изменён target kernel/image;
- затронут vendor/HAL interface;
- требуется device capability validation;
- изменён boot/partition logic;
- изменён hardware ownership;
- завершён milestone;
- VM test выявил behavior, который невозможно доказать виртуально.

---

# 12. Unified Experiment backend

Experiment Automation не должно быть связано только с physical Device Lab.

Предлагается общий abstraction:

```text
Experiment Runner
       │
       ├── local/static
       ├── qemu-kvm
       ├── qemu-arm64
       ├── cuttlefish
       └── device-lab
```

Experiment DSL должен уметь выбирать backend:

```yaml
name: runtime-core-smoke

target:
  backend: qemu-kvm
  profile: debian-testing

expect:
  - runtime-manager-active
  - binder-ready
```

или:

```yaml
name: vendor-hal-smoke

target:
  backend: device-lab
  device_profile: enchilada

expect:
  - android-framework-booted
  - vendor-hal-baseline-match
```

Результат желательно нормализовать в одну experiment schema независимо от backend.

---

# 13. Что нельзя автоматически переносить между VM и phone

Не следует считать equivalent следующие проверки:

| Проверка | VM | Physical device |
|---|---:|---:|
| systemd/LXC lifecycle | высокая применимость | подтверждение |
| D-Bus/network logic | высокая | подтверждение |
| Binder architecture | средняя/высокая | обязательно для target kernel |
| ARM64 ELF/APEX | высокая на ARM64 VM | подтверждение |
| Android Framework generic behavior | высокая в Cuttlefish | vendor-specific не доказано |
| VINTF generic compatibility | высокая | exact vendor compatibility требует device artifacts |
| vendor HAL | низкая без vendor environment | обязательно |
| firmware/drivers | практически отсутствует | обязательно |
| modem/camera/GPU/power | mock only | обязательно |
| bootloader/slots/AVB | частично моделируется | обязательно |

---

# 14. Nested virtualization

Если Anvil CI сам выполняется внутри VM, возможен nested KVM.

Linux KVM поддерживает nested virtualization, в частности nested VMX на Intel. Однако доступность и производительность зависят от CI provider и host configuration.

Поэтому:

- не делать nested virtualization обязательным требованием базового CI;
- уметь использовать dedicated/bare-metal runners;
- иметь fallback на TCG для функциональных ARM64 tests;
- маркировать performance-sensitive tests отдельно.

---

# 15. Рекомендация по реализации

## Этап A — быстрый VM smoke test

Создать reproducible Debian VM image и проверить:

- systemd;
- LXC;
- cgroup;
- binderfs;
- D-Bus;
- Runtime Manager skeleton;
- networking;
- test artifact collection.

Это должен быть первый virtual backend.

## Этап B — Golden fixtures

Определить export schema из Device Snapshot и использовать fixtures в VM tests.

## Этап C — ARM64 VM

Добавить generic QEMU `virt` profile для ARM64 packages/runtime components.

## Этап D — Cuttlefish

Использовать Cuttlefish для Android-framework-specific tests и generic/newer Android research.

## Этап E — Unified Experiment Runner

Один experiment definition должен по возможности запускаться на нескольких backends.

## Этап F — CI promotion

Автоматически запускать дешёвые gates до physical HIL.

---

# 16. PoC

Минимальный PoC виртуального стенда должен доказать:

```text
Debian VM boots
    ↓
systemd ready
    ↓
binderfs available
    ↓
LXC starts
    ↓
Anvil Runtime test container starts
    ↓
structured smoke tests run
    ↓
experiment result bundle produced
```

Вторая часть PoC:

```text
same experiment framework
      │
      ├── qemu-kvm backend
      └── device-lab backend
```

Цель — доказать, что backend является свойством test target, а не отдельным automation stack.

---

# 17. Критерии успеха

Research direction можно считать подтверждённым, когда:

- определён минимальный VM image/profile;
- L1 smoke tests воспроизводимы;
- experiment result format одинаков для VM и Device Lab;
- Golden Snapshot fixtures имеют schema/version;
- минимум один ARM64-specific test проходит в QEMU;
- минимум один Android framework test проходит в Cuttlefish;
- promotion policy определяет, какие changes требуют phone;
- VM failures автоматически блокируют ненужный physical flash/test.

---

# 18. Риски

## R-VT-001 — False confidence

VM может скрывать device-kernel/vendor проблемы.

Mitigation: явно хранить `test_level` и `backend`; VM PASS не повышать автоматически до hardware acceptance.

## R-VT-002 — Mock drift

Mock HAL может перестать соответствовать real device behavior.

Mitigation: генерировать часть fixtures из Golden Snapshot и регулярно сравнивать с physical baseline.

## R-VT-003 — Cuttlefish coupling

Anvil tests могут случайно начать зависеть от Cuttlefish-specific vendor implementation.

Mitigation: Cuttlefish должен быть optional backend/reference target.

## R-VT-004 — ARM64 TCG cost

Full-system ARM64 emulation на x86_64 может быть слишком медленной для каждого commit.

Mitigation: разделить smoke/full tests, использовать ARM64 KVM runners при возможности.

## R-VT-005 — VM kernel differs from target kernel

Современный VM kernel может предоставлять BinderFS/cgroups/features, которых нет в старом Android downstream kernel.

Mitigation: capability-based tests и обязательный target-device gate для kernel-sensitive функций.

---

# 19. Open questions

Нужно отдельно определить:

1. какой Debian release/profile использовать для canonical VM;
2. нужен ли отдельный minimal Anvil test kernel;
3. какая часть Android userspace реально может стартовать в generic LXC VM без virtual HAL layer;
4. стоит ли использовать Cuttlefish напрямую или через отдельный backend adapter;
5. формат Golden Snapshot fixtures;
6. как versioning Device Profile связывается с virtual fixtures;
7. какие test scenarios можно запускать одинаково в VM и Device Lab;
8. требуется ли dedicated ARM64 CI host на ранней стадии.

---

# 20. Влияние на соседние research nodes

## `40_diagnostics_testing.md`

Diagnostics должны уметь работать с VM и physical experiment artifacts.

## `41_device_lab_mcp.md`

Device Lab остаётся physical-device backend и не должен разрастаться до общего VM orchestrator.

## `42_experiment_automation.md`

Experiment Automation должен стать общим orchestration layer над virtual и physical backends.

## `43_failure_analysis_regression.md`

Failure Analysis должен сравнивать результаты между backends и учитывать уровень доказательства.

## `62_reproducible_builds.md`

VM images и Cuttlefish inputs должны иметь version/provenance так же, как physical artifacts.

---

# 21. Рекомендуемое архитектурное решение

Принять многоуровневую validation model:

```text
L0 Static
  ↓
L1 Native VM/KVM
  ↓
L2 ARM64 QEMU    L2A Cuttlefish
        \          /
         \        /
          ▼      ▼
        L3 Device Lab
```

И зафиксировать правило:

> Физическое устройство является source of truth для hardware/vendor compatibility, но не должно использоваться для проверки того, что можно надёжно и дешевле доказать на virtual pre-flight уровне.

---

# 22. Источники

Primary/official sources зарегистрированы также в `sources/documentation.md`.

- QEMU — System Emulation Introduction: https://www.qemu.org/docs/master/system/introduction.html
- QEMU — Emulation / TCG architecture support: https://www.qemu.org/docs/master/about/emulation.html
- QEMU — ARM `virt` machine: https://www.qemu.org/docs/master/system/arm/virt.html
- Linux kernel — Android binderfs filesystem: https://kernel.org/doc/html/next/admin-guide/binderfs.html
- Linux kernel — Nested VMX/KVM: https://www.kernel.org/doc/html/latest/virt/kvm/x86/nested-vmx.html
- Android — Create and manage virtual devices: https://developer.android.com/studio/run/managing-avds
- AOSP — Cuttlefish ARM64 / custom kernel example: https://source.android.com/docs/core/architecture/16kb-page-size/getting-started-cf-arm64-pgagnostic
- AOSP — Stable AIDL, Cuttlefish as development tool: https://source.android.com/docs/core/architecture/aidl/stable-aidl
