# Security policy

AetherOS is a research preview. Do not use it to protect real data yet.

## Reporting a vulnerability

Report privately; do not open a public issue for an unfixed vulnerability.
Include the version (`ver` in the shell), the build id (`about trust`), steps to
reproduce and the impact you expect. A contact address will be published with
the first public release.

## What is in scope

- program admission (signature check, preflight, AetherBridge translation)
- isolation (W^X, KPTI, capability checks)
- AetherProof receipts (forgery, chain breaks, key handling)
- the release build and signing process

## Known, tracked weaknesses

Open findings are listed in `tests/findings/FINDINGS.md` and in
`ARCHITECTURE.html` section 41. Reports that duplicate them are still welcome
when they add a working exploit or a better fix.

## Keys

- Release kernels trust only the release public key; its private key never
  enters the repository or the image.
- The DEV key in the source tree is a published **test** key. Dev builds trust
  it; release builds refuse to be built with it.
- Receipt signing keys are generated per boot and are never displayed.
