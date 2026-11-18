# apps

## Name

apps - list the signed apps shipped in this build

## Synopsis

```
apps
```

## Description

Each app is a VELA program signed with this build's trusted key.

## Mechanism

Static table built by kernel/build.rs from tools/apps (packed by pack.py).

## See also

[run](run.md), [receipts](receipts.md)
