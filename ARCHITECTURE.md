# Anvil Architecture

## Назначение

Этот документ является краткой актуальной картой архитектуры проекта Anvil.

Он не должен дублировать подробные research nodes и ADR. Его задача — показывать текущее согласованное устройство системы на одном уровне абстракции и служить точкой входа для понимания проекта.

## Текущая базовая модель

```text
                         Debian host
                              │
             ┌────────────────┴────────────────┐
             │                                 │
       Linux applications              Android applications
             │                                 │
       Linux services                  Android Framework
             │                                 │
             └──────────────┬──────────────────┘
                            │
                       Anvil Runtime
                            │
                  Android system services
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
     Android BSP       Mainline Linux       Hybrid
          │                 │                 │
          └─────────────────┼─────────────────┘
                            │
                         hardware
```

## Принятые архитектурные положения

- Основная host-платформа: Debian.
- Runtime core не должен зависеть от Droidian, Ubuntu Touch, Sailfish OS или конкретной mobile shell.
- Первый container backend: LXC.
- Android BSP используется как основной hardware backend первого поколения.
- Для mainline-supported устройств предусматривается native Linux backend.
- Долгосрочно должен поддерживаться hybrid backend.
- Wayland используется как основная графическая интеграция с Linux host.
- PipeWire используется как основной Linux media/audio integration layer.
- Android Runtime должен быть отделён от shell-specific integration.

## Что должно появляться здесь по мере развития проекта

После завершения research nodes сюда переносятся только подтверждённые архитектурные решения:

- границы компонентов;
- основные data flows;
- ownership аппаратных подсистем;
- host/runtime interfaces;
- hardware backend model;
- краткие ссылки на ADR и спецификации.

Подробности реализации должны оставаться в соответствующих research/design/spec документах.
