# Mobile Android Runtime — Research Workspace

Этот архив подготовлен для параллельной проработки архитектурных компонентов проекта.

Каждый `.md` файл является самостоятельным исследовательским заданием. Его можно открывать в отдельной сессии и прорабатывать независимо, кроме узлов, которые по смыслу уже объединены в один документ.

## Основные параллельные потоки

Рекомендуется начать одновременно с:

1. `10_device/10_device_capture.md`
2. `10_device/12_device_profile.md`
3. `20_runtime/21_android_container_core.md`
4. `20_runtime/22_graphics_display_input.md`
5. `20_runtime/23_vendor_hal_integration.md`
6. `20_runtime/20_runtime_manager.md`
7. `40_testing/40_diagnostics_testing.md`

## Вторая волна

После первых архитектурных выводов:

- audio;
- camera/media;
- telephony/modem;
- sensors/GNSS;
- wireless;
- power;
- security hardening;
- packaging/update design.

## Правило

Каждый исследовательский поток должен завершаться конкретным design recommendation, PoC-планом и перечнем необходимых доработок существующих решений.
