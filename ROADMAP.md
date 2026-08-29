# Anvil Roadmap

## Назначение

Этот документ фиксирует порядок развития проекта, основные milestones и зависимости между фазами.

Он должен оставаться кратким и обновляться после завершения исследовательских узлов и архитектурных решений.

## Текущая последовательность

### Phase -1 — Golden Device Capture

Получить заведомо рабочий Android baseline и снять:

- boot chain;
- partition images;
- VINTF;
- Binder/HAL inventory;
- kernel/device tree;
- firmware;
- hardware test baseline.

### Phase 0 — Debian Host

Загрузить Debian userspace на known-good Android kernel:

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

### Phase 1A — Android Core in LXC

Запустить:

```text
Android init
Binder
APEX
zygote
system_server
PackageManager
```

Главный milestone:

```text
sys.boot_completed = 1
```

### Phase 1B — Vendor / HAL Reattachment

Подключить original vendor services и проверить HAL по сравнению с Golden Android.

### Phase 2 — APK Lifecycle

Обеспечить install / launch / stop Android-приложений.

### Phase 3 — Graphics

Сначала functional VirtualDisplay → Wayland, затем dma-buf / zero-copy.

### Phase 4 — Desktop Integration

Input, launcher, notifications, clipboard, file/URL/MIME integration.

### Phase 5 — Hardware Services

Последовательно:

- audio;
- camera;
- sensors/GNSS;
- wireless;
- modem/telephony;
- power/thermal/battery.

### Phase 6 — Generic / Newer Android Framework

Перейти от original framework к generic AOSP/Lineage/GSI при сохранении совместимости с vendor.

### Phase 7 — Mainline / Hybrid

Переносить аппаратные подсистемы на native Linux там, где mainline support достаточно зрелый.

## Правило обновления roadmap

Каждый research node после завершения должен:

1. подтвердить или скорректировать соответствующую фазу;
2. указать блокирующие зависимости;
3. добавить measurable acceptance criteria;
4. при необходимости предложить новый ADR.
