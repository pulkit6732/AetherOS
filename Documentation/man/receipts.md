# receipts

## Name

receipts - receipt summary; export prints them for a verifier

## Synopsis

```
receipts [export]
```

## Description

Without arguments: a private summary (how many runs were receipted this
boot and whether the chain is intact). Receipts reveal which programs ran
and when, so their contents are not shown by default.
receipts export: prints the signer public key and the last 32 receipts
as APR2 hex lines, to save and check on another machine with
aetherproof-verify <file> --pubkey <key>

## Mechanism

AetherProof v2: one 248-byte receipt per finished program run, issued at
sys_exit by aetherproof-core. Binds SHA-256 of the signed image, SHA-256
of the AetherBridge x86 output, PROOF.PATH hash, syscall count, and the
SHA-256 of the previous receipt (hash chain). Ed25519, per-boot key.

## See also

[receipt](receipt.md), [verify-receipt](verify-receipt.md), [about trust](about.md)
