# stress

## Name

stress - run an n-iteration stress test (default 1000)

## Synopsis

```
stress [n]
```

## Description

Runs the AetherMon stress loop n times. Available only in builds made with `--features monitor`.

## Mechanism

Runs n iterations per subsystem and reports the timer ticks each took.

## See also

[monitor](monitor.md), [mem](mem.md)
