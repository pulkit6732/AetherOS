# reboot

## Name

reboot - restart the machine

## Synopsis

```
reboot
```

## Description

Pulses the keyboard-controller reset line; falls back to a CPU triple fault.

## Mechanism

8042 reset (port 0x64 <- 0xFE), then triple fault.

## See also

[shutdown](shutdown.md)
