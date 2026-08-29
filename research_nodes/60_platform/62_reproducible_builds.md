# 62 — Reproducible Builds / Build Provenance

## Статус

**Research task prepared**

Этот узел определяет воспроизводимую build environment и происхождение бинарных артефактов Anvil.

## Назначение компонента

В проекте будут появляться:

```text
boot.img
initramfs
kernel
kernel modules
Debian rootfs
Android images
device packages
runtime binaries
```

Через некоторое время необходимо уметь ответить:

> Из каких исходников, каким toolchain и с какими параметрами был получен этот конкретный image?

И желательно:

> Можно ли получить идентичный artifact повторной сборкой?

## Основная модель

```text
source commit
+
dependency revisions
+
toolchain
+
build environment
+
configuration
+
SOURCE_DATE_EPOCH
        ↓
      build
        ↓
artifact + provenance
```

Каждый artifact должен иметь минимум:

```text
SHA256
source commit
build ID
toolchain versions
device profile version
build configuration
timestamp policy
dependency manifest
```

## Что требуется проработать

- reproducible builds.
- hermetic build environments.
- containers/chroots/Nix-like approaches как reference.
- Debian package reproducibility.
- AOSP reproducible build issues.
- kernel reproducibility.
- Android boot image reproducibility.
- build manifests.
- artifact attestations.
- SBOM.
- build cache.
- source dependency pinning.
- toolchain pinning.
- signing separation.
- deterministic timestamps.
- artifact naming/versioning.

## Задание на исследование

1. Изучить Debian Reproducible Builds.
2. Изучить AOSP/Android reproducibility tooling.
3. Проанализировать kernel reproducible build requirements.
4. Определить минимальную reproducibility policy для Anvil.
5. Определить build environment:
   - container;
   - chroot;
   - VM;
   - other hermetic approach.
6. Определить toolchain/version pinning.
7. Спроектировать build manifest.
8. Спроектировать artifact provenance metadata.
9. Определить relation provenance ↔ experiment ID.
10. Определить SBOM/attestation applicability.
11. Разделить build reproducibility и artifact signing.
12. Спроектировать reproducibility test.
13. Определить CI integration.
14. Сформировать поэтапный implementation plan.

## Что должно появиться после проработки

- reproducibility requirements;
- comparison of build environment options;
- selected build environment;
- dependency/toolchain pinning policy;
- build manifest schema;
- artifact provenance schema;
- source→artifact relationship;
- experiment integration;
- reproducibility verification procedure;
- signing policy boundaries;
- CI plan;
- PoC;
- roadmap;
- источники.

## Зависимости

Связан с:

- `61_packaging_images_updates.md`
- `42_experiment_automation.md`
- `13_image_tooling.md`

Результаты должны использоваться всеми build-producing узлами.

## Обязательная структура итогового документа

```text
1. Purpose
2. Reproducibility requirements
3. Existing ecosystems/tools
4. Debian reproducible builds
5. Android/AOSP reproducibility
6. Kernel/images
7. Build environment
8. Dependency/toolchain pinning
9. Build manifest
10. Artifact provenance
11. SBOM/attestations
12. Signing boundary
13. Experiment integration
14. CI verification
15. PoC
16. Implementation roadmap
17. Risks
18. Sources
```
