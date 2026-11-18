# uptime

## Name

uptime - time since power-on

## Synopsis

```
uptime
```

## Description

Uses the CPU timestamp counter, calibrated once against the PIT timer.

## Mechanism

rdtsc divided by a PIT-calibrated TSC frequency.

## See also

[status](status.md), [uname](uname.md)
