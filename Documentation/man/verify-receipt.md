# verify-receipt

## Name

verify-receipt - check a saved receipt's signature

## Synopsis

```
verify-receipt [pid|p80]
```

## Description

Reads receipt-<pid>.bin (default receipt-p80.bin, made at boot) and checks its
signature. VALID means the bytes match the signature, nothing more.

## Mechanism

Strict Ed25519 verification (rejects weak keys and malleable signatures).

## See also

[receipt](receipt.md), [tamper-probe](tamper-probe.md)
