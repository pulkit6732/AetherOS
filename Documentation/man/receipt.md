# receipt

## Name

receipt - make a DEMO receipt (pid 1..999, default 1)

## Synopsis

```
receipt [pid]
```

## Description

Builds a 128-byte signed receipt for a process and saves receipt-<pid>.bin.
DEMO ONLY: the binary hash covers an 8-byte stub, not a running program.
Receipts do not prove what executed; see 'about'.

## Mechanism

Legacy v1 DEMO receipt (128 bytes, FNV-1a binary hash). Signed with the
per-boot v1 key; use 'receipts' for real v2 run receipts.

## See also

[receipts](receipts.md), [verify-receipt](verify-receipt.md)
