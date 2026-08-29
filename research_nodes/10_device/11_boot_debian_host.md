# Boot / Debian Host

## Статус

**Research task prepared.**

Этот файл предназначен для отдельной самостоятельной проработки Phase 0.

## Назначение узла

Определить надёжный и переносимый способ загрузки Debian host на устройстве, используя заведомо рабочий Android kernel и максимально сохраняя original boot/vendor hardware foundation.

Цель:

```text
bootloader
   ↓
known-good Android kernel
   ↓
Linux-oriented initramfs
   ↓
Debian rootfs
   ↓
systemd
```

## Что требуется проработать

- Android boot image formats.
- `boot`, `vendor_boot`, `init_boot`.
- GKI / non-GKI.
- DTB / DTBO.
- AVB.
- A/B и Virtual A/B.
- Dynamic partitions.
- Initramfs strategy.
- Debian rootfs bootstrap.
- Kernel modules.
- Firmware.
- Device nodes.
- Early boot logging.
- Recovery / rollback.
- Возможность сохранять golden Android slot.
- Применимость решений Droidian, Halium, Mobian, postmarketOS.

## Задание на исследование

1. Найти и изучить существующие способы загрузки Debian/Linux userspace на Android devices.
2. Разделить решения по поколениям Android boot architecture.
3. Определить, как сохранить kernel bytes и device-specific boot artifacts без изменений на первом PoC.
4. Определить минимальный Linux initramfs.
5. Определить структуру Debian rootfs для Phase 0.
6. Проанализировать kernel requirements:
   - namespaces;
   - cgroups;
   - binderfs;
   - filesystems;
   - modules;
   - firmware.
7. Определить handling AVB и unlocked bootloader.
8. Сформировать безопасную rollback strategy.
9. Предложить не менее двух вариантов boot flow, если это оправдано.
10. Выбрать рекомендуемый вариант для первого PoC.
11. Составить пошаговый эксперимент и acceptance criteria.

## Что должно появиться в этом файле после проработки

- обзор Droidian/Halium/Mobian/postmarketOS boot approaches;
- карта Android boot generation → Anvil boot strategy;
- рекомендуемая boot architecture;
- initramfs design;
- Debian rootfs requirements;
- kernel config/module/firmware checklist;
- AVB strategy;
- A/B/rollback plan;
- PoC procedure;
- критерии успешного Phase 0;
- список device-specific unknowns;
- прямые ссылки на исходники и официальную документацию.

## Обязательная структура итогового документа

```text
1. Цель
2. Android boot architecture
3. Existing solutions
4. Comparison
5. Recommended boot flow
6. Initramfs
7. Debian rootfs
8. Kernel/modules/firmware
9. AVB and slots
10. Recovery/rollback
11. Interfaces with Device Profile
12. PoC
13. Acceptance criteria
14. Risks/open questions
15. Sources
```

## Зависимости

Вход:

- Device Capture;
- Golden Android snapshot.

Выход используется:

- Device Profile;
- Runtime Manager;
- Android Container Core.
