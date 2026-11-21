# Security Policy

## Project status

AetherOS is experimental software distributed as a public research preview. It has not undergone a complete independent security audit and must not be used to protect sensitive data, production workloads, or critical systems.

## Supported version

| Version | Status |
|---|---|
| 0.92.0 | Research Preview |

## Reporting a vulnerability

Please report suspected security vulnerabilities privately. Do not publish details of an unpatched vulnerability in a public issue.

**Private reporting channel:** [https://www.linkedin.com/in/pulkit-srivastava-2033942a6/) OR ENABLE GITHUB PRIVATE VULNERABILITY REPORTING]

Include:
- Affected AetherOS version.
- Build ID (`about trust`).
- Environment and virtualisation platform.
- Steps to reproduce.
- Expected and observed behaviour.
- Potential security impact.
- Proof-of-concept material, where safe to provide.

## Scope

Security reports are welcome for:
- Program admission and signature verification.
- AELF parsing and instruction validation.
- AetherBridge translation and execution.
- Sandbox boundaries and memory protections.
- Fuel limits and runtime checks.
- AetherProof receipt generation, signatures, and chain verification.
- Release builds and signing procedures.

## Known limitations

AetherOS has documented open findings and incomplete security properties. Consult `RELEASE_NOTES.md` and the architecture documentation before testing.

Reports that overlap with known findings are still welcome when they provide a reproducible exploit, additional impact, or a better remediation.

## Cryptographic keys

- The release admission public key is used by release builds.
- The release private key must never be committed to the repository or included in the ISO.
- Development signing keys are test keys and must not be treated as production trust anchors.
- Receipt signing keys are generated at startup; they are not TPM-sealed hardware identities.

## Responsible disclosure

Please allow the maintainer reasonable time to investigate and address a report before publicly disclosing an unfixed vulnerability.
