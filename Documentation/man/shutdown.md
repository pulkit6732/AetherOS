# shutdown

## Name

shutdown - power off (QEMU, VirtualBox)

## Synopsis

```
shutdown
```

## Description

Writes the ACPI power-off ports of QEMU, Bochs and VirtualBox, then halts.

## Mechanism

ACPI PM1a writes for QEMU/Bochs/VirtualBox; no AML interpreter yet.

## See also

[reboot](reboot.md)
