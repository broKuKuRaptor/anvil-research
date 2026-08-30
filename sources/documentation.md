# Documentation Sources

## Формат

```text
Name
Status
URL
Version/date
Scope
Used by
Notes
```

---

## QEMU — System Emulation Introduction

Status: `PRIMARY`

URL: https://www.qemu.org/docs/master/system/introduction.html

Version/date: QEMU master documentation, проверено 2026-08-30

Scope:
- system emulation;
- host/guest architecture model;
- accelerator model;
- distinction between virtual machine and target physical hardware.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- первичная документация QEMU;
- используется для обоснования роли generic VM backend.

---

## QEMU — Emulation support

Status: `PRIMARY`

URL: https://www.qemu.org/docs/master/about/emulation.html

Version/date: QEMU master documentation, проверено 2026-08-30

Scope:
- architecture emulation;
- TCG;
- cross-architecture emulation.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- используется для ARM64-on-x86_64 test model.

---

## QEMU — ARM `virt` platform

Status: `PRIMARY`

URL: https://www.qemu.org/docs/master/system/arm/virt.html

Version/date: QEMU master documentation, проверено 2026-08-30

Scope:
- generic ARM/AArch64 virtual machine;
- virtual hardware model.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- подтверждает, что QEMU `virt` является generic virtual platform, а не моделью Android phone SoC.

---

## Linux kernel — binderfs

Status: `PRIMARY`

URL: https://kernel.org/doc/html/next/admin-guide/binderfs.html

Version/date: Linux kernel documentation, проверено 2026-08-30

Scope:
- Android Binder filesystem;
- dynamic Binder device creation;
- independent binderfs instances.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- применяется для оценки Binder testing внутри VM;
- не доказывает наличие binderfs на конкретном downstream Android kernel.

---

## Linux kernel — Nested VMX / KVM

Status: `PRIMARY`

URL: https://www.kernel.org/doc/html/latest/virt/kvm/x86/nested-vmx.html

Version/date: Linux kernel documentation, проверено 2026-08-30

Scope:
- nested virtualization;
- nested VMX/KVM requirements.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- используется только как optional CI deployment reference;
- nested KVM не должен быть обязательным требованием Anvil CI.

---

## Android Developers — Create and manage virtual devices

Status: `PRIMARY`

URL: https://developer.android.com/studio/run/managing-avds

Version/date: Android Developers documentation, проверено 2026-08-30

Scope:
- Android Virtual Device;
- Android Emulator targets;
- virtual Android testing environment.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- Android Emulator рассматривается как Android framework/application reference target, а не emulator vendor stack целевого телефона.

---

## AOSP — Cuttlefish ARM64 / custom kernel workflow

Status: `PRIMARY`

URL: https://source.android.com/docs/core/architecture/16kb-page-size/getting-started-cf-arm64-pgagnostic

Version/date: AOSP documentation, проверено 2026-08-30

Scope:
- Cuttlefish ARM64;
- custom kernel testing;
- Android boot validation;
- `sys.boot_completed` verification.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- Cuttlefish рассматривается как generic Android platform-development backend;
- его virtual vendor/hardware model не эквивалентен vendor stack целевого телефона.

---

## AOSP — Stable AIDL

Status: `PRIMARY`

URL: https://source.android.com/docs/core/architecture/aidl/stable-aidl

Version/date: AOSP documentation, проверено 2026-08-30

Scope:
- stable AIDL interfaces;
- platform/vendor interface development;
- Cuttlefish as Android platform development/testing target.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

Notes:
- используется для оценки роли Cuttlefish в generic framework/VINTF/interface testing.
