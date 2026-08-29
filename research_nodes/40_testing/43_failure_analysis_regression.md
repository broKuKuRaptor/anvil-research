# 43 — Failure Analysis / Regression Engine

## Статус

**Research task prepared**

Этот узел объединяет автоматический triage ошибок, сравнение Golden Android с Anvil runtime, capability diff и формирование компактных regression reports.

## Назначение компонента

Device Lab может собрать:

```text
dmesg
logcat
journal
pstore
recovery logs
getprop
service list
lshal
mounts
/dev
sysfs
VINTF
```

Но передача всего этого агенту напрямую дорогостояща и плохо масштабируется.

Цель:

> детерминированно нормализовать, сравнивать и предварительно классифицировать диагностические данные до передачи их AI-agent.

## Основная модель

```text
raw artifacts
     ↓
normalization
     ↓
event extraction
     ↓
timeline correlation
     ↓
baseline comparison
     ↓
failure classification
     ↓
compact report
```

Агент получает:

```yaml
failure_stage: android_framework

primary_event:
  type: apex_mount_failure

related:
  - binder context manager unavailable
  - selinux_denials: 17

regressions:
  missing_services:
    - android.hardware.camera.provider
```

а не мегабайты сырых логов.

## Capability Diff

Сравнивать:

```text
Golden Android
      ↓
capability snapshot

Anvil
      ↓
capability snapshot

      ↓
structured diff
```

Минимальные категории:

- properties;
- Binder services;
- HIDL/AIDL HAL;
- VINTF;
- mounts;
- devices;
- sysfs;
- firmware;
- kernel modules;
- network interfaces;
- SELinux state;
- processes;
- package/framework state.

## Failure Signatures

Предварительные категории:

```text
kernel panic
Oops
watchdog
mount failure
firmware missing
Binder failure
APEX failure
linker failure
SELinux denial
service crash loop
zygote failure
system_server failure
vendor HAL crash
network failure
```

## Задание на исследование

1. Найти Android/Linux log analysis и crash triage frameworks.
2. Изучить approaches к:
   - log normalization;
   - event signatures;
   - timeline correlation;
   - baseline diff;
   - regression reports.
3. Определить общий diagnostic event schema.
4. Определить capability snapshot schema.
5. Спроектировать Golden-vs-runtime diff.
6. Определить boot-stage classifier.
7. Спроектировать rule-based failure signatures.
8. Определить, что должен анализировать deterministic engine, а что оставлять AI-agent.
9. Спроектировать artifact queries:
   - read relevant range;
   - search by event;
   - compare runs.
10. Интегрировать с Device Lab experiment bundles.
11. Интегрировать с HIL CI.
12. Сформировать MVP и implementation roadmap.

## Что должно появиться после проработки

- diagnostic event schema;
- capability snapshot schema;
- Golden-vs-runtime diff format;
- boot/failure stage taxonomy;
- initial signature library;
- timeline correlation design;
- compact report schema;
- artifact query API;
- integration с Device Lab;
- integration с Experiment Automation;
- PoC plan;
- roadmap;
- источники.

## Зависимости

Вход:

- `40_diagnostics_testing.md`
- `41_device_lab_mcp.md`

Используется:

- `42_experiment_automation.md`

Сравнивает данные, полученные из:

- Device Capture;
- Golden Android;
- Android Container Core;
- Vendor/HAL Integration.

## Обязательная структура итогового документа

```text
1. Purpose
2. Existing analysis tools
3. Diagnostic data taxonomy
4. Event schema
5. Capability snapshot
6. Baseline diff
7. Boot stage classification
8. Failure signatures
9. Timeline correlation
10. AI vs deterministic analysis boundary
11. Report/API design
12. Device Lab integration
13. HIL CI integration
14. PoC
15. Implementation roadmap
16. Risks
17. Sources
```
