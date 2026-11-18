# history

## Name

history - last 16 commands (Up/Down arrows recall them)

## Synopsis

```
history
```

## Description

Duplicates of the previous line and empty lines are not stored.

## Mechanism

16-entry ring buffer in the shell; arrows come from the PS/2 0xE0 codes.

## See also

[help](help.md)
