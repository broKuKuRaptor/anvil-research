# Anvil

**Android Hardware Runtime for Linux**

Anvil — экспериментальный проект по созданию Linux-системы для мобильных устройств, в которой Android используется не как основная операционная система, а как **runtime для Android-приложений и слой совместимости с Android hardware stack**.

Цель проекта — получить систему, где основной host работает на Debian/Linux, а один Android Runtime предоставляет:

- запуск Android-приложений;
- доступ к Android Framework;
- использование vendor services и Android HAL;
- совместимость с существующим Android BSP устройства;
- постепенный переход к native/mainline Linux hardware support там, где он доступен.

## Идея

Современные Mobile Linux решения часто используют два независимых Android-окружения:

```text
Linux host
├── Halium / Android vendor environment
└── Waydroid / Android application runtime
```

Anvil исследует возможность объединить эти роли:

```text
                         Debian host
                              │
             ┌────────────────┴────────────────┐
             │                                 │
       Linux applications              Android applications
             │                                 │
       Linux services                  Android Framework
             │                                 │
             └──────────────┬──────────────────┘
                            │
                      Anvil Runtime
                            │
                  Android system services
                            │
                 vendor HAL / Linux backend
                            │
                         hardware
```

Основной принцип:

> На устройстве должен существовать один Android hardware/runtime world, которым могут совместно пользоваться Android-приложения и Linux-host.

## Базовая host-платформа

В качестве основной архитектурной базы выбран **Debian**.

Планируемый базовый стек:

```text
Debian
├── systemd
├── udev
├── LXC
├── cgroup v2
├── binderfs
├── D-Bus
├── Wayland
└── PipeWire
```

Runtime не должен зависеть от конкретной mobile shell.

Потенциальные UI:

- Phosh;
- Plasma Mobile;
- Lomiri;
- другие Wayland compositors.

## Hardware backends

Anvil должен поддерживать несколько моделей работы с оборудованием.

### Android BSP backend

Для устройств, где основной hardware support находится в Android vendor stack:

```text
Debian
  │
Anvil Runtime
  │
Android Framework
  │
vendor services
  │
Android HAL
  │
Android kernel
  │
hardware
```

Это основной путь для первого поколения проекта.

### Native Linux backend

Для устройств с хорошей mainline Linux поддержкой:

```text
Debian / Mobian / postmarketOS
            │
      native Linux stack
            │
         hardware
```

Android Runtime использует стандартные Linux interfaces.

### Hybrid backend

Долгосрочно наиболее интересный вариант:

```text
Display      → DRM/KMS/Mesa
Audio        → ALSA/PipeWire
Wi-Fi        → Linux
Bluetooth    → BlueZ

Camera       → Android HAL
Modem        → Android RIL
Fingerprint  → Android HAL
```

То есть Android BSP используется только там, где native Linux support отсутствует или существенно хуже.

## Начальная стратегия разработки

Первый этап проекта строится вокруг **Golden Android baseline**.

На устройство устанавливается заведомо полностью рабочая Android/LineageOS система.

До любых изменений снимаются:

- boot chain;
- kernel;
- system/vendor/odm images;
- dynamic partition layout;
- VINTF;
- Binder/HAL services;
- firmware;
- device tree;
- kernel configuration;
- `/dev` и sysfs inventory;
- hardware test baseline.

После этого выполняется контролируемый переход:

```text
Known-good Android
        ↓
Golden Device Snapshot
        ↓
тот же kernel
        ↓
Debian host
        ↓
LXC
        ↓
тот же Android userspace
        ↓
тот же vendor stack
```

На раннем этапе желательно не менять одновременно:

- kernel;
- vendor;
- Android framework.

Это позволяет проверять влияние Linux-host и container boundary отдельно от проблем hardware compatibility.

## Планируемая реализация

Проект предполагается развивать поэтапно.

### Phase -1 — Device Capture

Создать Golden Device Snapshot полностью рабочего Android-устройства.

### Phase 0 — Debian Host

Загрузить Debian userspace на исходном working Android kernel.

```text
bootloader
   ↓
known-good Android kernel
   ↓
Linux initramfs
   ↓
Debian rootfs
   ↓
systemd
```

### Phase 1 — Android Runtime

Перенести исходный Android userspace внутрь LXC:

```text
Debian
  │
 LXC
  │
Android init
  │
Binder / APEX
  │
zygote
  │
system_server
```

Первый важный критерий:

```text
sys.boot_completed = 1
```

### Phase 2 — Android applications

Обеспечить:

- PackageManager;
- APK install/uninstall;
- ActivityManager;
- Android application lifecycle.

### Phase 3 — Graphics

Организовать вывод Android через:

```text
SurfaceFlinger
    ↓
Virtual Display
    ↓
GraphicBuffer
    ↓
Wayland
```

После функционального PoC перейти к `dma-buf` / zero-copy.

### Phase 4 — Linux integration

Добавить:

- input;
- application launcher;
- notifications;
- clipboard;
- file sharing;
- URL/MIME integration.

### Phase 5 — Hardware services

Последовательно прорабатывать:

- audio;
- camera;
- sensors;
- GNSS;
- Wi-Fi;
- Bluetooth;
- modem/telephony;
- power management.

### Phase 6 — Generic Android Runtime

После доказательства архитектуры:

```text
original Android framework
        ↓
modified framework
        ↓
generic AOSP / Lineage
        ↓
newer Android framework
```

Совместимость framework/vendor проверяется через Treble/VINTF/GSI/VTS.

### Phase 7 — Mainline / Hybrid

Для устройств с хорошей mainline Linux поддержкой Android-specific hardware stack постепенно заменяется Linux-native реализациями.

## Основные компоненты проекта

Предварительно проект разделяется на следующие направления:

```text
Anvil
├── Device Capture
├── Device Profile
├── Boot / Debian Host
├── Runtime Manager
├── Android Container Core
├── Vendor / HAL Integration
├── Graphics / Display / Input
├── Networking
├── Application Management
├── Desktop Integration
├── Diagnostics / Testing
├── Audio
├── Camera / Media
├── Telephony / Modem
├── Sensors / GNSS
├── Power / Thermal / Battery
├── Wireless
├── Security
└── Packaging / Updates
```

Каждый компонент прорабатывается отдельным design/research документом.

## Reference projects

Anvil не является форком этих проектов, но использует их как инженерную и архитектурную базу:

- Halium;
- libhybris;
- Droidian;
- Ubuntu Touch;
- Waydroid;
- Sailfish OS;
- Mobian;
- postmarketOS;
- AOSP / Project Treble / GSI.

Особое внимание уделяется повторному использованию существующих решений вместо разработки аналогичных механизмов с нуля.

## Текущий статус

Проект находится на стадии архитектурного проектирования и исследования существующих решений.

Текущие основные задачи:

1. определить границы отдельных компонентов;
2. исследовать существующие реализации;
3. выбрать переиспользуемые технологии;
4. определить необходимые изменения;
5. подготовить PoC-планы;
6. перейти к реализации Device Capture, Debian boot и Android Container Core.

## Принцип проекта

Anvil не ставит целью создать ещё одну отдельную мобильную ОС.

Цель — создать переносимый runtime и hardware compatibility layer:

```text
                   Android applications
                           │
                         Anvil
                           │
              ┌────────────┼────────────┐
              │            │            │
         Android BSP    Mainline      Hybrid
              │            │            │
              └────────────┼────────────┘
                           │
                        hardware
                           ▲
                           │
                         Debian
```

Детальные архитектурные решения, исследования и планы реализации находятся в отдельных документах проекта.
