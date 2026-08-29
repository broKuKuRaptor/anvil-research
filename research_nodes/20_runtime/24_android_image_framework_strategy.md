# Android Image / Framework Strategy

## Статус

**Research task prepared.**

Этот файл предназначен для отдельной самостоятельной проработки стратегии Android system/framework image в Anvil.

## Назначение узла

Определить, какой Android userspace должен использовать Anvil на разных стадиях:

```text
original device Android
        ↓
modified original framework
        ↓
generic Lineage/AOSP
        ↓
GSI / newer Android framework
```

При этом original vendor stack должен по возможности оставаться совместимым.

## Что требуется проработать

- `system`.
- `system_ext`.
- `product`.
- `vendor`.
- `odm`.
- APEX.
- linkerconfig.
- VINTF.
- Treble.
- GSI.
- VNDK/vendor compatibility.
- Android framework upgrade path.
- Read-only image model.
- Writable `/data`.
- Overlay approaches.
- Waydroid image model.
- LineageOS/AOSP image generation.
- Image versioning и compatibility metadata.

## Задание на исследование

1. Сравнить:
   - original OEM/Lineage system image;
   - modified original image;
   - AOSP;
   - LineageOS generic build;
   - GSI;
   - Waydroid images.
2. Определить, какие части Android userspace нужно сохранить для первого PoC.
3. Проанализировать framework/vendor compatibility через VINTF/Treble.
4. Определить, насколько реально использовать более новый Android framework поверх старого vendor.
5. Изучить APEX и linker namespace implications для container runtime.
6. Определить storage model:
   - RO images;
   - `/data`;
   - overlays;
   - upgrades;
   - rollback.
7. Сформировать несколько вариантов image strategy.
8. Выбрать рекомендуемую последовательность от Golden Android к generic runtime.
9. Составить PoC-план замены framework без изменения vendor.
10. Определить compatibility tests и критерии отказа от конкретной версии Android.

## Что должно появиться в этом файле после проработки

- сравнительная таблица image strategies;
- рекомендуемый initial image;
- рекомендуемый generic image path;
- VINTF/Treble compatibility model;
- APEX/linker considerations;
- image filesystem layout;
- `/data` persistence strategy;
- upgrade/rollback design;
- план перехода к newer Android;
- PoC;
- список ограничений по версиям framework/vendor;
- источники.

## Обязательная структура итогового документа

```text
1. Цель
2. Android partition/image model
3. Existing image approaches
4. Original vs AOSP vs Lineage vs GSI vs Waydroid
5. Framework/vendor compatibility
6. APEX/linker/VINTF requirements
7. Storage and persistence model
8. Upgrade/rollback strategy
9. Recommended sequence
10. PoC
11. Acceptance criteria
12. Risks/open questions
13. Sources
```

## Зависимости

Тесно связан с:

- Vendor / HAL Integration;
- Android Container Core;
- Packaging / Images / Updates.

Может исследоваться параллельно им, но итоговое решение должно быть согласовано между этими узлами.
