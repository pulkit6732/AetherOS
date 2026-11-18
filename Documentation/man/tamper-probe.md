# tamper-probe

## Name

tamper-probe - flip one byte and show it is rejected

## Synopsis

```
tamper-probe [pid|p80]
```

## Description

Verifies a receipt, flips one signature bit (expect INVALID), restores it
(expect VALID). Works on a copy in memory; the stored file is not changed.

## Mechanism

Flips one byte of a saved receipt and re-verifies; must be rejected.

## See also

[verify-receipt](verify-receipt.md)
