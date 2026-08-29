# 41 — Anvil Device Lab MCP

## Статус

**Initial research completed / Research task prepared**

Этот файл предназначен для дальнейшей самостоятельной проработки подсистемы **Anvil Device Lab** и её MCP-интерфейса.

Документ уже содержит результаты первичного анализа идеи и найденные reference projects. Следующая проработка должна углубить архитектуру, API, state machine, безопасность и план реализации.

---

# 1. Назначение компонента

Во время разработки Anvil регулярно повторяется один и тот же цикл работы с физическим Android-устройством:

```text
подготовить test image
        ↓
определить состояние устройства
        ↓
ADB / bootloader / fastbootd / recovery
        ↓
выбрать или переключить slot
        ↓
загрузить временный image или выполнить flash
        ↓
reboot
        ↓
ждать появления устройства
        ↓
определить результат boot
        ↓
снять properties / services / logs
        ↓
если boot failed:
    перейти в recovery
    получить pstore / recovery logs
    сохранить состояние
        ↓
при необходимости rollback
```

Если все эти действия выполняет AI-agent напрямую, возникают:

- большое количество однотипных tool calls;
- повторяющееся планирование ADB/Fastboot workflows;
- большой объём stdout/logcat в контексте LLM;
- высокая стоимость анализа рутинных состояний;
- риск ошибки при выборе slot или partition;
- риск выполнения опасных fastboot operations;
- сложность воспроизводимости эксперимента;
- необходимость каждый раз заново определять текущее состояние телефона.

Цель компонента:

> вынести низкоуровневую работу с устройством, контроль состояний, flashing workflows, сбор диагностических материалов и rollback в отдельную automation subsystem.

MCP является одним из интерфейсов к этой подсистеме, но не должен быть местом, где живёт основная orchestration logic.

---

# 2. Предлагаемое название

Подсистема:

```text
Anvil Device Lab
```

MCP frontend:

```text
anvil-device-mcp
```

Предпочтительная архитектура:

```text
                 AI agent
                    │
                   MCP
                    │
            anvil-device-mcp
                    │
               Device Lab Core
          ┌─────────┼──────────┐
          │         │          │
        CLI        MCP        CI/API
                    │
                    ▼
             Workflow Engine
                    │
             Device State Machine
                    │
       ┌────────────┼────────────┐
       │            │            │
      ADB        Fastboot     Recovery
                                 │
                              optional
                                SSH
```

Главное архитектурное правило:

> MCP server должен быть тонким adapter-слоем поверх обычного Device Lab Core.

Это позволит использовать ту же automation logic из:

- AI agent;
- CLI;
- test scripts;
- CI;
- device farm;
- developer tools.

---

# 3. Почему MCP здесь применим

MCP особенно полезен не для оборачивания отдельных `adb` команд, а для предоставления агенту **высокоуровневых, структурированных операций**.

Плохой уровень API:

```text
adb_command()
fastboot_command()
shell()
```

Такой API практически не снижает стоимость работы агента.

Предпочтительный уровень:

```text
device.status
device.ensure_state
slot.status
image.boot
experiment.run_boot_test
experiment.collect_failure
experiment.rollback
```

В этом случае Device Lab самостоятельно выполняет десятки низкоуровневых операций и возвращает агенту короткий structured result.

---

# 4. Найденные MCP-проекты

Первичный поиск показывает, что Android/ADB MCP ecosystem уже существует, но найденные проекты в основном ориентированы на управление Android через ADB, UI automation и diagnostics.

Готового зрелого MCP-сервера, ориентированного именно на:

```text
flash experimental image
→ slot management
→ reboot
→ state detection
→ boot failure recovery
→ recovery/pstore collection
→ rollback
```

на момент первичного анализа не найдено.

## 4.1 MrNewDelhi/adb-mcp

Repository:

https://github.com/MrNewDelhi/adb-mcp

Проект предоставляет широкий typed MCP API вокруг ADB.

Найденные возможности:

- device discovery;
- wait states;
- USB/TCP/IP;
- raw shell;
- reboot modes;
- root/remount;
- APK management;
- file push/pull;
- logcat;
- dumpsys;
- getprop/setprop;
- input events;
- intents;
- forwarding;
- bugreport;
- compact diagnostics bundle;
- raw `adb_command` escape hatch.

### Применимость к Anvil

Полезен как reference для:

- MCP tool taxonomy;
- ADB backend;
- diagnostics API;
- raw escape hatch;
- structured Android device operations.

Недостаток:

- практически не решает fastboot/recovery orchestration;
- ориентирован на уже доступное ADB-устройство.

---

## 4.2 us-all/android-mcp-server

Repository:

https://github.com/us-all/android-mcp-server

Проект предоставляет крупный набор Android diagnostics tools.

По найденной документации:

- десятки ADB tools;
- logcat;
- dumpsys;
- package analysis;
- process diagnostics;
- battery/memory/cpu/network;
- aggregation tools;
- MCP Prompts;
- read-only по умолчанию;
- отдельные security gates для write и shell operations.

### Особенно полезная идея

Разделение операций по степени риска.

Для Anvil эта модель должна быть развита дальше на flashing/bootloader operations.

### Применимость

Reference для:

- diagnostics aggregation;
- безопасного MCP API;
- permission gates;
- compact reports вместо необработанного log stream.

---

## 4.3 httprunner/adb-mcp

Repository:

https://github.com/httprunner/adb-mcp

Go-based MCP server для ADB.

Найденные особенности:

- structured logging;
- configurable command timeout;
- stdio MCP transport;
- device discovery;
- reboot в system / bootloader / recovery / fastboot;
- UI operations;
- длинный stdout/stderr сохраняется в log files вместо переполнения MCP stdout.

### Особенно применимо к Anvil

Идея:

> long-running command output → artifact/log store, а не MCP context.

Это следует считать одним из базовых принципов Device Lab.

---

# 5. Найденные non-MCP reference projects

Для fastboot/slot/flashing orchestration полезнее оказались обычные Android toolkits.

## 5.1 Pixel-Kit

Repository:

https://github.com/not-GIANT/Pixel-Kit

Найденные возможности:

- ADB operations;
- Fastboot operations;
- partition flashing;
- A/B slot management;
- получение device info;
- временный boot `.img` без permanent flash;
- reboot modes.

### Применимость

Reference для:

- slot management;
- fastboot workflow;
- live boot;
- partition operations;
- UX вокруг device state.

---

## 5.2 AutoIMG

Repository:

https://github.com/BlassGO/AutoIMG

Проект автоматизирует Android installation/flashing workflows.

Найденные особенности:

- Fastboot и FastbootD;
- slot-aware flashing;
- автоматический выбор `_a` / `_b`;
- работа с dynamic partitions;
- проверка размера image и partition;
- scripted configuration flows;
- recovery-aware installation.

### Применимость

Очень полезный reference для:

- generic flashing workflow;
- slot-aware partition resolution;
- dynamic partition handling;
- Fastboot/FastbootD distinction;
- scriptable installation state machine.

---

# 6. Основная архитектурная гипотеза

Device Lab должен быть не просто набором wrappers, а **stateful device orchestrator**.

Предлагаемая структура:

```text
Device Lab Core
├── Device Discovery
├── State Machine
├── Transport Backends
├── Workflow Engine
├── Safety Policy
├── Artifact Store
├── Experiment Database
├── Diagnostics Parsers
├── CLI
└── MCP Adapter
```

---

# 7. Device State Machine

Телефон может находиться в различных состояниях:

```text
UNKNOWN
DISCONNECTED
OFFLINE

ANDROID_ADB
LINUX_ADB
LINUX_SSH

RECOVERY_ADB
SIDELOAD

BOOTLOADER_FASTBOOT
FASTBOOTD
```

В перспективе могут появиться:

```text
EDL
DOWNLOAD_MODE
SERIAL_CONSOLE
```

но они не должны входить в первый PoC.

State определяется комбинацией:

```text
adb devices
fastboot devices
USB VID/PID
adb getprop
fastboot getvar
fastboot getvar is-userspace
optional SSH probe
```

Пример:

```text
fastboot device exists
+
is-userspace=yes

→ FASTBOOTD
```

---

# 8. State transitions

Device Lab должен самостоятельно знать допустимые переходы.

Пример:

```text
ANDROID_ADB
    │
    │ adb reboot bootloader
    ▼
BOOTLOADER_FASTBOOT
    │
    │ fastboot reboot fastboot
    ▼
FASTBOOTD
```

или:

```text
ANDROID_ADB
    │
    │ adb reboot recovery
    ▼
RECOVERY_ADB
```

Вместо того чтобы агент решал последовательность команд, API должен позволять:

```text
device.ensure_state("recovery")
```

Workflow engine самостоятельно выбирает путь.

---

# 9. Transport Backends

Минимальный набор:

```text
ADB backend
Fastboot backend
Recovery ADB backend
```

Дополнительно:

```text
SSH backend
```

для Linux host после Phase 0.

Позднее:

```text
Serial backend
EDL backend
```

если появится необходимость.

---

# 10. Уровни API

## 10.1 Low-level

Используется в основном для отладки Device Lab:

```text
adb.shell
adb.push
adb.pull

fastboot.getvar
fastboot.flash
fastboot.boot

recovery.shell
```

Низкоуровневый API должен существовать, но не быть основным интерфейсом агента.

## 10.2 Mid-level

```text
device.list
device.status
device.ensure_state
device.reboot
device.wait

slot.status
slot.activate

image.inspect
image.boot
image.flash

logs.collect
snapshot.collect
```

## 10.3 Workflow-level

Основной интерфейс для AI-agent:

```text
experiment.run_boot_test
experiment.run_flash_test
experiment.collect_failure
experiment.rollback
experiment.restore_golden
```

Именно этот уровень должен максимально сокращать tool calls.

---

# 11. Пример boot experiment

Agent вызывает:

```text
experiment.run_boot_test(
    image = boot-anvil-042.img,
    target_slot = inactive
)
```

Device Lab выполняет:

```text
1. identify device
2. determine current state
3. record current slot
4. verify image checksum
5. determine whether temporary boot is supported
6. reboot to bootloader
7. verify fastboot state
8. boot or flash requested image
9. change slot if required
10. reboot
11. monitor USB/device state transitions
12. wait for ADB/SSH
13. classify boot result
14. collect standard diagnostics
15. if boot failed:
       enter recovery if possible
       collect pstore/recovery logs
16. preserve experiment artifacts
17. optionally restore previous state
18. return concise structured result
```

---

# 12. Structured result

Агент должен получать короткий результат, например:

```json
{
  "experiment_id": "exp-0042",
  "result": "boot_failed",
  "previous_slot": "a",
  "tested_slot": "b",
  "last_state": "recovery",
  "adb_seen": false,
  "fastboot_seen": true,
  "recovery_seen": true,
  "failure_stage": "kernel_to_userspace",
  "artifact_bundle": "experiments/exp-0042/"
}
```

Полные логи не должны автоматически передаваться в LLM context.

---

# 13. Experiment Artifact Store

Каждый test run должен быть воспроизводимым.

Пример:

```text
experiments/
└── exp-0042/
    ├── request.json
    ├── result.json
    ├── timeline.jsonl
    ├── device-before.json
    ├── device-after.json
    │
    ├── images/
    │   ├── boot.img.sha256
    │   └── metadata.json
    │
    ├── fastboot/
    │   ├── getvar-all.txt
    │   └── slot-status.txt
    │
    ├── adb/
    │   ├── getprop.txt
    │   ├── service-list.txt
    │   ├── logcat.txt
    │   └── dmesg.txt
    │
    └── recovery/
        ├── dmesg.txt
        ├── pstore/
        └── mounts.txt
```

MCP возвращает:

- experiment ID;
- status;
- summary;
- notable events;
- paths/resources к артефактам.

---

# 14. Diagnostics preprocessing

Device Lab должен по возможности делать первичную фильтрацию логов.

Искать:

```text
kernel panic
Oops
watchdog
avc: denied
binder errors
init failures
mount failures
APEX failures
firmware missing
SELinux
service crash loops
```

Результат:

```text
notable_events:
  - type: mount_failure
  - type: selinux_denial
  - type: missing_firmware
```

Агент затем может запросить только relevant log region.

---

# 15. Recovery as first-class backend

Recovery нельзя рассматривать только как special-case ADB.

В recovery Device Lab должен уметь:

- определить recovery environment;
- mount partitions read-only;
- читать pstore;
- читать `/proc/last_kmsg`, если доступно;
- получать recovery logs;
- inspect slots;
- pull files;
- reboot в bootloader/system.

Операция:

```text
experiment.collect_failure
```

сама должна выбирать доступный источник forensic data.

---

# 16. Safety model

Flashing automation является потенциально опасной.

Предлагается классификация операций:

```text
READ_ONLY
TEST_BOOT
FLASH_SLOT
DESTRUCTIVE
BOOTLOADER_CRITICAL
```

## READ_ONLY

```text
adb getprop
service list
fastboot getvar
collect logs
```

## TEST_BOOT

```text
fastboot boot image.img
```

если устройство поддерживает temporary boot.

## FLASH_SLOT

```text
fastboot flash boot_b ...
set active slot
```

## DESTRUCTIVE

Например:

```text
erase userdata
erase metadata
partition resize
```

## BOOTLOADER_CRITICAL

По умолчанию должны быть запрещены:

```text
flash bootloader
flash xbl
flash abl
flash rpm
flash tz
flash modem
erase persist
flashing lock
```

Конкретный список должен быть device-profile aware.

---

# 17. Golden-state protection

Device Lab должен знать Golden Device Snapshot и до flashing операций сохранять:

```text
current slot
partition hashes
active boot image
device state
recovery availability
rollback procedure
```

Желательно поддерживать policy:

```text
golden_slot = a
experimental_slot = b
```

если hardware/partition layout это позволяет.

---

# 18. Интеграция с Device Profile

`12_device_profile.md` должен предоставлять Device Lab данные:

```text
device codename
USB IDs
slots
partition names
dynamic partitions
boot image layout
supported transports
safe flash partitions
forbidden partitions
recovery strategy
ADB properties
expected device states
```

Device Lab не должен содержать hard-coded знания о конкретном телефоне.

---

# 19. Интеграция с Device Capture

`10_device_capture.md` и Device Lab частично пересекаются.

Предлагаемая граница:

```text
Device Capture
    ↓
создаёт Golden Snapshot

Device Lab
    ↓
использует Snapshot
для экспериментов,
verification и rollback
```

Возможно, позднее capture-tool станет workflow внутри Device Lab Core.

---

# 20. Интеграция с Diagnostics

`40_diagnostics_testing.md` определяет:

- какие показатели собирать;
- какие тесты выполнять;
- какие значения сравнивать с Golden Android.

Device Lab отвечает за:

- получение данных с реального устройства;
- transport/state handling;
- сохранение artifacts;
- запуск diagnostic workflows.

Таким образом:

```text
Device Lab
    │
data acquisition
    ▼
Diagnostics Framework
    │
analysis/comparison
```

---

# 21. Предварительный MCP API

Для первой версии желательно не создавать сотни tools.

Минимальный набор:

```text
device.list
device.status
device.ensure_state
device.reboot
device.wait

slot.status
slot.activate

image.inspect
image.boot
image.flash

snapshot.collect

experiment.run_boot_test
experiment.run_flash_test
experiment.collect_failure
experiment.rollback

artifact.read
```

Дополнительно один escape hatch:

```text
debug.exec
```

с явно указанным backend:

```text
adb
fastboot
recovery
ssh
```

---

# 22. Чего не следует делать

Не рекомендуется строить API в виде сотен прямых оболочек:

```text
adb_getprop
adb_logcat
adb_dumpsys
fastboot_flash
fastboot_getvar
...
```

как основной интерфейс для AI.

Такие primitives могут существовать внутри Device Lab Core, но основной MCP API должен быть ориентирован на **намерение и workflow**.

---

# 23. Предварительный план реализации

## Stage 1 — Device State Detector

Реализовать:

```text
device.list
device.status
device.wait
```

Поддержать:

```text
ADB
BOOTLOADER_FASTBOOT
FASTBOOTD
RECOVERY_ADB
DISCONNECTED
```

## Stage 2 — State Transitions

```text
device.ensure_state
device.reboot
```

Покрыть переходы:

```text
ADB → bootloader
ADB → recovery
bootloader → fastbootd
bootloader → system
recovery → bootloader
recovery → system
```

## Stage 3 — Slot Management

```text
slot.status
slot.activate
```

С safety checks.

## Stage 4 — Image Operations

```text
image.inspect
image.boot
image.flash
```

Добавить:

- SHA-256;
- partition size validation;
- slot validation;
- temporary boot preference.

## Stage 5 — Artifact Store

Каждая операция получает:

```text
operation_id
timestamp
device identity
timeline
stdout/stderr artifacts
```

## Stage 6 — Boot Experiment

Реализовать:

```text
experiment.run_boot_test
```

Это первый основной high-level workflow.

## Stage 7 — Recovery Diagnostics

```text
experiment.collect_failure
```

С pstore/recovery capture.

## Stage 8 — Rollback

```text
experiment.rollback
experiment.restore_golden
```

## Stage 9 — MCP Adapter

После стабилизации Device Lab Core добавить компактный MCP frontend.

Не наоборот.

---

# 24. Ключевые вопросы для дальнейшего исследования

Необходимо дополнительно определить:

1. На каком языке реализовать Device Lab Core.
2. Использовать ли прямые Android platform-tools binaries или библиотеки.
3. Есть ли зрелая библиотека Fastboot, пригодная для orchestration.
4. Как надёжно различать bootloader fastboot и fastbootd.
5. Как учитывать vendor-specific recovery.
6. Как хранить operation/experiment state после crash MCP server.
7. Как реализовать exclusive device lock.
8. Как поддерживать несколько подключённых телефонов.
9. Как обрабатывать USB disconnect/re-enumeration.
10. Как ограничить опасные flashing operations.
11. Как формально описать rollback guarantees.
12. Как интегрировать Device Profile.
13. Как представить artifacts через MCP resources.
14. Какие operations требуют явного подтверждения пользователя.
15. Как тестировать Device Lab без постоянного риска brick физического устройства.

---

# 25. Что требуется проработать далее

Следующая отдельная research-сессия должна:

1. Глубоко изучить найденные MCP проекты:
   - API;
   - архитектуру;
   - лицензии;
   - активность;
   - возможность reuse.
2. Найти дополнительные Fastboot/recovery automation libraries.
3. Проанализировать Android platform-tools internals.
4. Спроектировать formal device state machine.
5. Спроектировать safety policy.
6. Определить Device Lab Core API.
7. Отделить Core API от MCP API.
8. Определить experiment artifact schema.
9. Спроектировать recovery/failure collection workflow.
10. Определить MVP.
11. Составить detailed implementation roadmap.
12. Определить automated test strategy.

---

# 26. Что должно содержаться в итоговом документе

После полной проработки этот файл должен включать:

```text
1. Purpose and scope
2. Existing MCP solutions
3. Existing Fastboot/Recovery automation
4. Reuse/license analysis
5. Device Lab architecture
6. Device state machine
7. Transport backend interfaces
8. Workflow engine
9. Safety model
10. Golden-state protection
11. Artifact/experiment model
12. Device Profile integration
13. Diagnostics integration
14. Core API
15. MCP API
16. CLI design
17. PoC
18. Implementation roadmap
19. Testing strategy
20. Risks
21. Open questions
22. Sources
```

---

# 27. Первичный вывод

Создание Anvil Device Lab представляется технически оправданным.

Главная ценность не в использовании MCP как такового, а в переносе повторяющейся логики:

```text
detect
transition
flash
wait
classify
collect
rollback
```

из AI-agent в детерминированный orchestration layer.

Ожидаемая модель:

```text
AI:
    "проверь boot image"

Device Lab:
    выполняет 10–30 низкоуровневых операций

AI получает:
    concise structured result
    +
    ссылки на diagnostic artifacts
```

Это должно:

- уменьшить число agent tool calls;
- сократить объём LLM context;
- сделать эксперименты воспроизводимыми;
- уменьшить риск flashing errors;
- упростить regression testing;
- позволить использовать ту же infrastructure без AI.

---

# 28. Источники первичного анализа

## MCP / Android

MrNewDelhi adb-mcp  
https://github.com/MrNewDelhi/adb-mcp

us-all Android MCP Server  
https://github.com/us-all/android-mcp-server

httprunner adb-mcp  
https://github.com/httprunner/adb-mcp

## Fastboot / flashing workflow references

Pixel-Kit  
https://github.com/not-GIANT/Pixel-Kit

AutoIMG  
https://github.com/BlassGO/AutoIMG

## Базовые технологии

Android Debug Bridge documentation  
https://developer.android.com/tools/adb

Android source / platform-tools  
https://android.googlesource.com/platform/packages/modules/adb/

Model Context Protocol  
https://modelcontextprotocol.io/
