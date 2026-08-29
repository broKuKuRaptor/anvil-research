# Anvil Research Workspace

Этот каталог содержит независимые research nodes проекта Anvil.

Каждый `.md` файл является самостоятельным исследовательским заданием. Его можно прорабатывать в отдельной сессии, но итоговые решения должны учитывать зависимости, зафиксированные в корневом `DEPENDENCIES.md` и внутри самих узлов.

Исследовательский узел должен завершаться не только обзором существующих решений, но и конкретным результатом для Anvil:

- design recommendation;
- сравнением вариантов и reusable components;
- PoC-планом;
- implementation/adaptation plan;
- acceptance/readiness criteria;
- рисками и открытыми вопросами;
- источниками и provenance.

## Текущие приоритетные потоки

Первая волна должна сформировать основу первого PoC и инфраструктуру безопасных экспериментов:

1. `10_device/10_device_capture.md`
2. `10_device/11_boot_debian_host.md`
3. `10_device/12_device_profile.md`
4. `10_device/13_image_tooling.md`
5. `20_runtime/21_android_container_core.md`
6. `20_runtime/23_vendor_hal_integration.md`
7. `20_runtime/24_android_image_framework_strategy.md`
8. `40_testing/40_diagnostics_testing.md`
9. `40_testing/41_device_lab_mcp.md`

## Параллельные runtime/integration потоки

Могут исследоваться параллельно после фиксации базовых assumptions:

- `20_runtime/20_runtime_manager.md`;
- `20_runtime/22_graphics_display_input.md`;
- `30_integration/30_networking.md`;
- `30_integration/31_application_management.md`;
- `30_integration/32_desktop_integration.md`.

## Development infrastructure

Эти узлы формируют инфраструктуру воспроизводимой разработки и должны развиваться вместе с первыми физическими экспериментами:

- `40_testing/41_device_lab_mcp.md`;
- `40_testing/42_experiment_automation.md`;
- `40_testing/43_failure_analysis_regression.md`;
- `60_platform/62_reproducible_builds.md`;
- `10_device/13_image_tooling.md`.

## Hardware / platform wave

После появления устойчивого Android Runtime и vendor/HAL boundary:

- audio;
- camera/media;
- telephony/modem;
- sensors/GNSS;
- wireless;
- power/thermal/battery;
- security hardening;
- packaging/images/updates.

## Правило ведения research node

Перед началом глубокой проработки проверить:

- входные документы и assumptions;
- зависимости от соседних узлов;
- существующие project knowledge и source registry;
- необходимость внешнего research.

После завершения узла:

1. обновить сам research document;
2. перенести подтверждённые архитектурные выводы в accepted project docs;
3. актуализировать `DEPENDENCIES.md`, `ROADMAP.md`, `RISKS.md` и `CURRENT.md`, если выводы на них влияют;
4. зарегистрировать новые источники в `sources/`;
5. перенести reusable решения и методы в `knowledge/`;
6. обновить `research_nodes/INDEX.md` при изменении структуры.

Общие правила разработки и исследования определены в корневом `WORKFLOW.md`.
