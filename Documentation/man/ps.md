# ps

## Name

ps - list processes

## Synopsis

```
ps
```

## Description

Shows PID, priority and state for every process slot in the scheduler.

## Mechanism

Walks the scheduler process table (64-PID bitmap) under its lock.

## See also

[status](status.md), [receipts](receipts.md)
