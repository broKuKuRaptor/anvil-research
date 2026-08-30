# Tool Sources

## Формат

```text
Tool name
Status
Source
Version
Distribution
SHA-256
Local path
Purpose
Known limitations
Used by
```

---

## QEMU

Status: `PRIMARY`

Source: https://www.qemu.org/

Version: не зафиксирована; research-level reference

Distribution:
- Debian package или upstream build;
- exact version должна быть зафиксирована при появлении canonical VM environment.

SHA-256: N/A на стадии исследования

Local path: N/A

Purpose:
- system virtualization/emulation;
- KVM-accelerated native-architecture VM;
- TCG ARM64 emulation;
- generic ARM `virt` platform.

Known limitations:
- generic virtual hardware не моделирует target Android SoC/vendor stack;
- ARM64 TCG на x86_64 может быть слишком медленным для каждого commit.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

---

## KVM

Status: `PRIMARY`

Source: https://www.kernel.org/doc/html/latest/virt/kvm/index.html

Version: host Linux kernel dependent

Distribution:
- Linux kernel virtualization subsystem.

SHA-256: N/A

Local path: N/A

Purpose:
- hardware-accelerated VM execution when host/guest architecture is compatible;
- preferred backend для fast Level 1 VM tests.

Known limitations:
- availability depends on host hardware, kernel and CI environment;
- nested KVM availability is provider/configuration dependent.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

---

## Android Emulator / AVD

Status: `REFERENCE`

Source: https://developer.android.com/studio/run/emulator

Version: не зафиксирована; research-level reference

Distribution:
- Android SDK emulator + system images.

SHA-256: N/A на стадии исследования

Local path: N/A

Purpose:
- Android framework/application reference tests;
- generic AOSP virtual device behavior.

Known limitations:
- не воспроизводит vendor stack конкретного target device;
- не является основным Anvil host/runtime test backend.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`

---

## Cuttlefish

Status: `REFERENCE`

Source: https://source.android.com/docs/devices/cuttlefish

Version: зависит от AOSP branch/build; exact revision должна фиксироваться при внедрении backend

Distribution:
- AOSP virtual device tooling/images.

SHA-256: N/A на стадии исследования

Local path: N/A

Purpose:
- Android platform/framework virtual-device testing;
- ARM64/x86_64 Android targets;
- custom kernel/framework/VINTF experiments.

Known limitations:
- использует собственный virtual hardware/vendor model;
- PASS в Cuttlefish не доказывает compatibility с target phone vendor/HAL stack.

Used by:
- `research_nodes/40_testing/44_virtual_test_bed.md`
