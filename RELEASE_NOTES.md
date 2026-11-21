# AetherOS 0.92.0 - research preview

Maintainer: Pulkit Srivastava | Project: https://github.com/pulkit6732/AetherOS | Security: see SECURITY.md

AetherOS is a from-scratch operating system for **verifiable execution**: only signed
programs run, each runs through the AetherBridge translator in a sandbox with a fuel
limit, and every finished run produces a signed, hash-chained receipt that anyone can
check offline. This is a preview for developers and researchers, not a daily-use OS.

## What is in this release

- Boots in **VirtualBox (BIOS firmware)** and QEMU to the `aether>` shell.
- **Signed apps run end to end**: `apps`, `run <app>` (hello, sum, fib, primes, spin).
  Each run: Ed25519 check against the release key, AetherBridge translation (user code
  confined to a 4 MiB sandbox arena), contained execution, fuel limit, v2 receipt.
- **Receipts (AetherProof v2, 280 bytes)** bind the program's SHA-256, the SHA-256 of the
  x86-64 code AetherBridge produced, the kernel build id, the control-flow hash and the
  previous receipt. `receipts` shows a private summary; `receipts export` prints them; `receipts save` writes them
  to `/home/receipts.txt`.
- **Shell**: `help`, `man <command>`, `about arch|security|trust|limits`, `license`,
  files under `/home` (`ls cd pwd mkdir touch cat echo > rm cp mv`), `history` with
  Up/Down, `uptime`, `uname -a`, `reboot`, `shutdown`, `lang en|hinglish|genz`.
- Keyboard: PS/2 with Shift and Caps Lock (VirtualBox default).

## Verify your download

```sh
openssl pkeyutl -verify -pubin -inkey release.pub.pem -rawin -in SHA256SUMS -sigfile SHA256SUMS.sig
sha256sum -c SHA256SUMS
```
Both must succeed. `release.pub.pem` is the key the kernel trusts (`about trust` shows
its first bytes).

## Run it in VirtualBox

Quickest: `./vbox-vm.sh aetheros-0.92.0.iso` creates and starts a correctly configured VM.
By hand:


1. New VM: type **Other/Unknown (64-bit)**, 512 MB RAM, **EFI off** (BIOS).
2. Network: NAT, adapter type **Intel PRO/1000 MT Desktop (82540EM)**.
3. Serial port 1: **enabled, mode "Raw File"**, path e.g. `aetheros-serial.txt`.
4. Attach `aetheros-0.92.0.iso` as the optical disk and start the VM. Type `help`.

Linux hosts with Secure Boot must sign VirtualBox's kernel module once (MOK), and KVM
must be unloaded while VirtualBox runs (`sudo modprobe -r kvm_intel kvm`).

## Check receipts on another machine

1. In the VM, run some apps, then `receipts export`. The output also goes to the serial
   file from step 3 above.
2. On the host, take the `boot signer public key` line and the `APR2 ...` lines:
   ```sh
   grep -a '^APR2 ' aetheros-serial.txt | cut -d' ' -f2 | tr -d '\r\n' | xxd -r -p > receipts.bin
   ./aetherproof-verify-x86_64 receipts.bin --pubkey <boot signer public key>
   ```
   (No `xxd`? `python3 -c "import sys;sys.stdout.buffer.write(bytes.fromhex(sys.stdin.read()))"`.)
   Expect `chain of N receipts: VALID`.

## Known limits

- QEMU and VirtualBox in BIOS mode only; no UEFI, no real-PC boot yet.
- Files under `/home` live in RAM and are lost at power-off (no AHCI disk driver yet).
- Receipt keys are generated per boot (no TPM-sealed device key yet), so pin the key
  printed at each boot.
- The control-flow hash is global, not per process; exact re-execution is future work.
- VELA-3 instructions are not yet implemented in AetherBridge (finding F-011).
- No GUI, no user accounts, no networking applications.
- Open findings: `tests/findings/FINDINGS.md`. Security review: `docs/reviews/`.

## Licence

Kernel MPL-2.0; ISA, tools and tests Apache-2.0. Third-party code is limited to vetted
cryptography (ed25519-dalek, sha2, ml-dsa).
