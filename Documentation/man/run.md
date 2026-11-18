# run

## Name

run - run an app: verify, translate, execute, receipt

## Synopsis

```
run <app>
```

## Description

Runs the full AetherOS program path and prints the result (R1),
the AetherBridge output size and the receipt number.

## Mechanism

exec::run_aelf: Ed25519 strict check against the trusted key, parse,
preflight, AetherBridge translation (user code confined to the SFI
arena), execution under fault containment, then a v2 receipt that binds
the image hash, the AetherBridge output hash and the kernel build id.

## See also

[apps](apps.md), [receipts](receipts.md), [about arch](about.md)
