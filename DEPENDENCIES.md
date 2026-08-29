# Anvil Research Dependencies

## Назначение

Этот документ показывает зависимости между исследовательскими узлами и помогает запускать проработку параллельно, не создавая скрытых блокировок.

## Основной граф

```text
Device Capture
      │
      ▼
Device Profile
      │
      ├───────────────┐
      ▼               ▼
Boot / Debian     Vendor / HAL
      │               │
      └───────┬───────┘
              ▼
      Android Container Core
              │
    ┌─────────┼──────────┬───────────┐
    ▼         ▼          ▼           ▼
Graphics   Networking   Apps     Runtime Manager
    │
    ├─────────────┬───────────────┐
    ▼             ▼               ▼
Audio         Camera/Media    Desktop Integration

Vendor / HAL
    ├──────────────┬──────────────┬──────────────┐
    ▼              ▼              ▼              ▼
Audio        Telephony       Sensors/GNSS     Wireless
    │              │              │              │
    └──────────────┴──────────────┴──────────────┘
                           │
                           ▼
                Power/Thermal/Battery
```

## Сквозные направления

Эти узлы могут прорабатываться параллельно почти всем остальным:

```text
Diagnostics / Testing
Security Model
Packaging / Images / Updates
```

## Рекомендуемые параллельные потоки сейчас

Можно независимо запускать:

1. `10_device_capture`
2. `11_boot_debian_host`
3. `12_device_profile`
4. `20_runtime_manager`
5. `21_android_container_core`
6. `22_graphics_display_input`
7. `23_vendor_hal_integration`
8. `24_android_image_framework_strategy`
9. `40_diagnostics_testing`

## Правило

Research node должен явно указывать:

- какие документы он использует как вход;
- какие решения он блокирует;
- какие interfaces должен определить для соседних компонентов;
- какие предположения требуют проверки экспериментом.
