# 13 — Image / Partition Tooling

## Статус

**Research task prepared**

Этот файл предназначен для отдельной самостоятельной проработки инструментария анализа, валидации и преобразования Android/Linux image-артефактов, используемых Anvil.

## Назначение компонента

Во время разработки Anvil постоянно потребуется выполнять однотипные операции:

```text
inspect boot.img
inspect vendor_boot.img
inspect init_boot.img
inspect vbmeta.img
inspect super.img
extract logical partitions
inspect DTB/DTBO
check AVB
compare images
verify partition sizes
prepare image for flash
```

Если каждый раз выполнять эти операции вручную через набор независимых CLI-инструментов, агент и разработчик будут тратить много времени на повторяющуюся рутину и нормализацию вывода.

Цель — определить единый слой:

```text
anvil-image
```

который предоставляет стабильный структурированный API поверх существующих Android tooling.

## Что требуется проработать

- Android boot image formats.
- `boot`, `vendor_boot`, `init_boot`.
- `vbmeta` и AVB descriptors.
- `super` и dynamic partitions.
- sparse/raw images.
- `lpdump`, `lpunpack`, `simg2img`.
- `unpack_bootimg`, `mkbootimg`.
- `avbtool`.
- DTB / DTBO / `dtc`.
- partition metadata.
- image checksums.
- image diff.
- safe validation before flashing.
- mapping image → target partition.
- compatibility with A/B slots.

## Основная идея

Вместо:

```text
unpack_bootimg ...
avbtool ...
lpdump ...
lpunpack ...
dtc ...
file ...
sha256sum ...
```

получить:

```bash
anvil-image inspect boot.img
anvil-image inspect super.img
anvil-image verify boot.img --device profile.yaml
anvil-image diff old.img new.img
```

с machine-readable output.

Пример:

```yaml
type: boot
header_version: 4
kernel:
  sha256: ...
  size: ...
ramdisk:
  present: false
avb:
  signed: true
target:
  partition: boot
  slot_aware: true
```

## Задание на исследование

1. Найти и изучить существующие Android image tools и библиотеки.
2. Определить, какие функции можно вызывать как библиотеки, а какие придётся оборачивать через CLI.
3. Сравнить:
   - Android platform-tools;
   - AOSP build tools;
   - Magisk boot image tooling;
   - `liblp`;
   - `avbtool`;
   - community boot image unpackers;
   - Droidian/Halium/postmarketOS tooling.
4. Определить универсальную internal image model.
5. Спроектировать structured output schema.
6. Определить pre-flash validation:
   - image type;
   - partition size;
   - slot;
   - architecture;
   - boot header;
   - AVB;
   - device compatibility.
7. Определить safe transformations, которые Anvil может выполнять автоматически.
8. Сформировать PoC CLI.
9. Определить интеграцию с Device Capture, Device Profile и Device Lab.
10. Проверить лицензии существующих реализаций.

## Что должно появиться после проработки

- обзор существующих image/partition tools;
- reuse/license matrix;
- internal image metadata model;
- CLI/API design;
- structured output schema;
- pre-flash validation rules;
- dynamic partition workflow;
- AVB workflow;
- boot image workflow;
- DTB/DTBO workflow;
- PoC plan;
- implementation roadmap;
- источники.

## Зависимости

Вход:

- `10_device_capture.md`
- `11_boot_debian_host.md`
- `12_device_profile.md`

Используется:

- `41_device_lab_mcp.md`
- `42_experiment_automation.md`
- `61_packaging_images_updates.md`

## Обязательная структура итогового документа

```text
1. Purpose
2. Existing tools
3. Reuse/license analysis
4. Android image taxonomy
5. Internal metadata model
6. CLI/API
7. Boot/vendor_boot/init_boot
8. Super/dynamic partitions
9. AVB
10. DTB/DTBO
11. Validation before flash
12. Device Profile integration
13. Device Lab integration
14. PoC
15. Implementation roadmap
16. Risks
17. Sources
```
