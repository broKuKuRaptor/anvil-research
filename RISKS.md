# Anvil Risk Register

## Назначение

Этот файл содержит сквозные технические риски проекта.

Он не заменяет разделы `Risks` внутри research nodes. Здесь фиксируются только риски, способные повлиять сразу на несколько компонентов или изменить архитектуру проекта.

## Текущий предварительный список

### R-001 — Android framework внутри container не воспроизводит штатный boot

Возможные причины:

- init/mount assumptions;
- Binder;
- APEX;
- cgroups;
- SELinux;
- uevent/device ownership.

Связанные узлы:

- Android Container Core;
- Vendor/HAL Integration;
- Security.

### R-002 — SurfaceFlinger и Linux compositor конкурируют за display stack

Требуется nested/virtual display model.

Связанные узлы:

- Graphics/Display/Input;
- Vendor/HAL Integration.

### R-003 — Vendor HAL зависит от среды, отсутствующей внутри LXC

Например:

- boot properties;
- sysfs;
- kernel command line;
- linker namespaces;
- vendor init ordering.

Связанные узлы:

- Device Profile;
- Container Core;
- Vendor/HAL Integration.

### R-004 — SELinux требует слишком глубоких изменений

Ранний permissive PoC может скрыть архитектурные проблемы.

Связанные узлы:

- Security;
- Container Core;
- Vendor/HAL.

### R-005 — Camera HAL не допускает совместное владение

Потребуется единый owner и bridge architecture.

### R-006 — Power management работает функционально, но неприемлемо по автономности

Особенно опасны:

- wakelocks;
- suspend/resume;
- thermal;
- modem;
- background Android services.

### R-007 — Framework/vendor version mismatch ограничивает обновление Android

Treble/VINTF может не позволить использовать существенно более новый framework с original vendor.

### R-008 — Android downstream kernel конфликтует с современным Debian userspace/cgroup model

Может потребоваться kernel configuration/patching.

### R-009 — Device-specific код начинает проникать в runtime core

Это приведёт к потере переносимости.

### R-010 — Существующие решения нельзя переиспользовать из-за лицензирования или архитектурной связанности

Необходимо проверять license и coupling до копирования кода.

## Что добавлять после исследований

Для каждого подтверждённого риска:

- вероятность;
- влияние;
- mitigation;
- owner/research node;
- критерий закрытия;
- статус: Open / Mitigated / Accepted / Closed.
