# Anvil Research Dependencies

## Назначение

Этот документ показывает зависимости между исследовательскими узлами и помогает запускать проработку параллельно, не создавая скрытых блокировок.

Он должен описывать не только runtime-компоненты, но и development infrastructure, которая обеспечивает безопасные, воспроизводимые и анализируемые эксперименты.

## Основной runtime/device граф

```text
Device Capture
      │
      ▼
Device Profile
      │
      ├───────────────┬────────────────┐
      ▼               ▼                ▼
Boot / Debian     Vendor / HAL     Image Tooling
      │               │                │
      └───────┬───────┘                │
              ▼                        │
      Android Container Core           │
              │                        │
    ┌─────────┼──────────┬─────────────┤
    ▼         ▼          ▼             ▼
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

## Android image/framework strategy

`24_android_image_framework_strategy` связывает:

```text
Golden Android / Device Capture
           │
           ▼
Android Container Core
           │
           ├───────────────┐
           ▼               ▼
Vendor / HAL        Packaging / Updates
           │               │
           └───────┬───────┘
                   ▼
     Framework / image strategy
```

Его итог влияет на:

- initial Android userspace;
- APEX/linker/VINTF model;
- persistence/overlay strategy;
- upgrade/rollback design;
- возможность перехода к generic/newer Android framework.

## Development infrastructure graph

```text
Device Capture
      │
      ▼
Device Profile
      │
      ▼
Image Tooling
      │
      ▼
Device Lab
      │
      ├───────────────┐
      ▼               ▼
Diagnostics      Experiment Automation
      │               │
      ▼               ├───────────────┐
Failure Analysis      │               ▼
 / Regression         │          HIL CI / Bisect
      ▲                │
      └────────────────┘
                       │
                       ▼
              Reproducible Builds
```

Точные отношения:

- `13_image_tooling` использует Device Capture и Device Profile и обслуживает Device Lab, Experiment Automation и Packaging.
- `41_device_lab_mcp` использует Device Profile, Image Tooling и Diagnostics.
- `42_experiment_automation` строится поверх Device Lab и Diagnostics, использует Image Tooling и связывается с Reproducible Builds.
- `43_failure_analysis_regression` использует Diagnostics и Device Lab и отдаёт структурированные результаты Experiment Automation/HIL CI.
- `62_reproducible_builds` связан с Image Tooling, Experiment Automation и Packaging и должен использоваться всеми build-producing узлами.

## Сквозные направления

Эти области могут исследоваться параллельно почти всему проекту, но их выводы могут изменить несколько узлов одновременно:

```text
Diagnostics / Testing
Security Model
Packaging / Images / Updates
Image Tooling
Reproducible Builds
Device Lab / Experiment Infrastructure
```

## Рекомендуемые параллельные потоки сейчас

Можно независимо начинать или углублять:

1. `10_device_capture`
2. `11_boot_debian_host`
3. `12_device_profile`
4. `13_image_tooling`
5. `20_runtime_manager`
6. `21_android_container_core`
7. `22_graphics_display_input`
8. `23_vendor_hal_integration`
9. `24_android_image_framework_strategy`
10. `40_diagnostics_testing`
11. `41_device_lab_mcp`

После появления первых конкретных experiment/build artifacts приоритет получают:

12. `42_experiment_automation`
13. `43_failure_analysis_regression`
14. `62_reproducible_builds`

## Правило

Research node должен явно указывать:

- какие документы и artifacts он использует как вход;
- какие решения он блокирует;
- какие interfaces должен определить для соседних компонентов;
- какие assumptions требуют проверки экспериментом;
- какие development-infrastructure компоненты должны обслуживать его PoC/validation.

После завершения research node этот граф пересматривается и актуализируется, если фактические зависимости отличаются от предварительных.
