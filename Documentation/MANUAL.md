# AetherOS User and Developer Manual

Version 0.92.0 (research preview)

This manual describes AetherOS as it is in release 0.92.0. Features that are planned but
not yet available are marked **Planned** and are listed separately in chapter 12.

## Contents

1. What AetherOS is
2. Core concepts
3. Installing and starting
4. Using the console
5. Command reference
6. Files and directories
7. Running programs
8. Receipts and verification
9. Writing your own program
10. Building from source
11. Security model
12. Interfaces: today and planned
13. Troubleshooting
14. Known limits
15. Glossary



## 1. What AetherOS is

AetherOS is an operating system written from scratch for one purpose: **making it
possible for anyone to check what a computer actually ran.**

On a conventional system, the record of what happened (logs, audit trails) is kept by
the same machine and the same administrators who could alter it. AetherOS changes this
in three ways:

- **Only signed programs run.** A program is accepted only if it carries a valid
  signature from the key the system trusts.
- **Programs run through a checked path.** Every program is translated by AetherBridge,
  confined to a sandbox, and stopped if it runs too long.
- **Every finished run produces a receipt.** The receipt is signed by the kernel and
  linked to the previous one, so removing, reordering or altering records is detected.
  Anyone can check receipts on another computer with a small, independent tool.

AetherOS is intended as a **trusted component** that runs next to a conventional system,
not as a replacement for a desktop or server operating system. Typical uses are
recording automated or AI-driven decisions, payment and approval steps, and any process
whose record must hold up when it is disputed.

**Status:** research preview. It runs in QEMU and VirtualBox (BIOS firmware). It is not
yet suitable for protecting real data. See chapter 14.

## 2. Core concepts

| Term | Meaning |
|---|---|
| **VELA** | The instruction set AetherOS programs are written in: fixed 32-bit instructions, registers R1 to R13, R0 always zero. |
| **AELF** | The signed program file format: a header, the code section and a 64-byte Ed25519 signature over the whole file. |
| **Admission gate** | The checks every program passes before it runs: signature, structure, and a scan that rejects privileged instructions. |
| **AetherBridge** | The translator that turns VELA code into x86-64 machine code and inserts runtime checks (for example the shadow-stack check before every return). |
| **Sandbox (SFI)** | User programs can read and write only a fixed 4 MiB memory area. They cannot reach kernel memory. |
| **Fuel** | Every backward jump uses one unit of fuel. A run that uses its whole budget (20 000 000 units) is stopped; the system keeps running. |
| **Receipt** | A 280-byte signed record of one finished run (chapter 8). |
| **Build id** | The SHA-256 of the kernel's source tree, build settings and compiler version. It identifies exactly which kernel issued a receipt. |
| **Release key** | The key the release kernel trusts for program admission. Its private half is kept by the maintainer, outside the source tree. |

## 3. Installing and starting

### 3.1 Check the download

A release contains:

| File | Purpose |
|---|---|
| `aetheros-0.92.0.iso` | The bootable system |
| `aetherproof-verify-x86_64` | The receipt verifier for Linux (static binary) |
| `release.pub.pem` | The release public key |
| `BUILD_ID` | The kernel build id |
| `RELEASE_NOTES.md` | Release notes |
| `SHA256SUMS`, `SHA256SUMS.sig` | Checksums and their signature |

Verify before use. Both commands must succeed:

```sh
openssl pkeyutl -verify -pubin -inkey release.pub.pem -rawin -in SHA256SUMS -sigfile SHA256SUMS.sig
sha256sum -c SHA256SUMS
```

### 3.2 VirtualBox

1. Create a virtual machine: type **Other/Unknown (64-bit)**, 512 MB memory.
2. System settings: **Enable EFI off** (AetherOS boots with BIOS firmware).
3. Network: attached to NAT, adapter type **Intel PRO/1000 MT Desktop (82540EM)**.
4. Serial port 1: enabled, port mode **Raw File**, a file path such as
   `aetheros-serial.txt`. Everything the system prints is also written there.
5. Storage: attach `aetheros-0.92.0.iso` to the optical drive.
6. Start the machine. After a few seconds the `aether>` prompt appears.

The same steps from a terminal:

```sh
VBoxManage createvm --name AetherOS --ostype Other_64 --register
VBoxManage modifyvm AetherOS --memory 512 --firmware bios --nic1 nat --nictype1 82540EM \
  --uart1 0x3F8 4 --uartmode1 file "$HOME/aetheros-serial.txt"
VBoxManage storagectl AetherOS --name IDE --add ide
VBoxManage storageattach AetherOS --storagectl IDE --port 0 --device 0 --type dvddrive \
  --medium aetheros-0.92.0.iso
VBoxManage startvm AetherOS --type gui
```

### 3.3 QEMU

```sh
qemu-system-x86_64 -cdrom aetheros-0.92.0.iso -m 512M -serial stdio -nic user,model=e1000
```

## 4. Using the console

After start-up the system shows a banner and the prompt:

```
aether>
```

Type a command and press Enter. In VirtualBox, click inside the window first so it
receives the keyboard.

| Key | Action |
|---|---|
| Backspace | Delete the last character |
| Ctrl+U | Clear the line |
| Ctrl+C | Abandon the line |
| Up / Down | Step through the last 16 commands |
| Shift, Caps Lock | Capital letters and symbols (US layout) |

**Language.** `lang hinglish` or `lang genz` changes the banner, the help header and error
hints. Commands themselves are always English. `lang en` switches back.

## 5. Command reference

`help` lists the commands available in your build, grouped by topic. `man <command>`
shows the full page for one command: name, synopsis, description, the mechanism that
does the work, and related commands.

### 5.1 System

| Command | Description |
|---|---|
| `help [command]` | List commands, or show one command's page |
| `man <command>` | Full manual page for a command |
| `about [overview\|arch\|security\|trust\|limits]` | What the system is, how it works, its keys and its limits |
| `license` | Licences of AetherOS and the third-party cryptography it uses |
| `ver` | Version and build information |
| `status` | Live summary of processes, files and memory |
| `uname [-a]` | System name; `-a` adds version, architecture and build id |
| `uptime` | Time since power-on |
| `ps` | List processes |
| `history` | Last 16 commands |
| `clear` | Clear the screen |
| `lang [en\|hinglish\|genz]` | Change the console language |
| `reboot` | Restart the machine |
| `shutdown` | Power off (QEMU, VirtualBox) |

### 5.2 Files

| Command | Description |
|---|---|
| `pwd` | Show the current directory |
| `cd [dir]` | Change directory; with no argument, go to `/home` |
| `ls [dir]` | List a directory; each file is marked `local` or `signed system file` |
| `mkdir <dir>` | Create a directory under `/home` |
| `touch <file>` | Create an empty file under `/home` |
| `cat <file>` | Show a file (first 4 KiB) |
| `echo <text>` | Print text |
| `echo <text> > <file>` | Write text to a file under `/home`, replacing a local file of that name |
| `cp <source> <destination>` | Copy a file into `/home` |
| `mv <source> <destination>` | Move a local file within `/home` |
| `rm <file\|dir>` | Remove a local file or an empty directory |
| `vfs-save` | Write the file system to a virtio disk (QEMU only; VirtualBox has no such disk) |

### 5.3 Programs and receipts

| Command | Description |
|---|---|
| `apps` | List the signed programs included in this build |
| `run <app>` | Verify, translate, run and receipt a program; prints its result |
| `receipts` | Private summary: how many runs were receipted and whether the chain is intact |
| `receipts export` | Print the signer key and the receipts for checking elsewhere |
| `receipt [pid]` | Create a legacy v1 demonstration receipt |
| `verify-receipt [pid]` | Check a saved v1 receipt's signature |
| `tamper-probe [pid]` | Change one byte of a saved receipt and show that it is rejected |

### 5.4 Diagnostics

| Command | Description |
|---|---|
| `mem` | Memory layout and allocator state |
| `monitor`, `stress [n]` | Performance counters and a stress test; only in builds made with `--features monitor` |

### 5.5 Examples

```
aether> mkdir notes
aether> cd notes
aether> echo meeting at nine > today.txt
aether> cat today.txt
meeting at nine
aether> run primes
result (R1): 168
  verified + translated: 151 B x86-64, 1 runtime checks inserted
  receipt #1 issued (see 'receipts')
aether> receipts
receipts this boot: 1  (last 1 kept in memory)
chain: intact (every receipt signed, none missing or reordered)
```

## 6. Files and directories

- All user files live under **`/home`**. The system starts in `/home`.
- Files you create are **local**: they belong to you and can be changed or removed.
- Files shipped with the system are **signed system files**: they can be read, but no
  command can change, move or remove them.
- Paths can be relative (`notes/a.txt`), absolute (`/home/notes/a.txt`) and use `.` and `..`.
  Names may contain letters, digits, `.`, `_` and `-`; a full path is at most 48 characters.
- Each file is checked against stored hashes when it is read.
- **In 0.92 files are kept in memory and are lost at power-off.**

## 7. Running programs

`run <app>` performs the complete program path:

1. **Admission.** The program's signature is checked against the trusted key, its
   structure is parsed, and every instruction is scanned. A program that fails any check
   is refused and does not run.
2. **Translation.** AetherBridge translates the program to x86-64, confines its memory
   access to the sandbox and inserts runtime checks.
3. **Execution.** The translated code runs. A CPU fault stops the program and is
   reported; the system continues.
4. **Fuel.** A program that does not finish within its fuel budget is stopped.
5. **Receipt.** A receipt is issued for the run.

The value a program leaves in register R1 is printed as its result. The included
programs are:

| App | Result | Shows |
|---|---|---|
| `hello` | 42 | The smallest program |
| `sum` | 5050 | A loop |
| `fib` | 6765 | Several registers |
| `primes` | 168 | Nested loops, division |
| `spin` | stopped by fuel | A program that never ends |

## 8. Receipts and verification

### 8.1 What a receipt contains

| Field | Meaning |
|---|---|
| Sequence number | Position in this boot's chain |
| Process id, syscall count, timestamp | Facts about the run |
| Program SHA-256 and length | Exactly which signed program ran |
| AetherBridge SHA-256 | Exactly which machine code was produced from it |
| Build id | Exactly which kernel issued the receipt |
| Control-flow hash | A fingerprint of the path the program took |
| Previous receipt SHA-256 | Links the chain |
| Signer public key and signature | Who signed it, and proof the bytes are unchanged |

### 8.2 What a receipt proves, and what it does not

A valid chain proves that the holder of the signing key recorded these runs, in this
order, with nothing removed, inserted or changed. Together with the build id it names the
kernel that recorded them.

It does **not** yet prove which physical machine signed it: in 0.92 the signing key is
created fresh at every start-up from hardware randomness, and AetherOS does not read real
TPM measurements yet, so no receipt claims a TPM-backed key. Pin the key printed at start-up (`receipts export`
shows it). A hardware-sealed device key is planned.

### 8.3 Checking receipts on another computer

1. In AetherOS: `receipts export`. With the serial port set up as in 3.2, the output is
   also in the serial file.
2. On the other computer:

```sh
grep -a '^APR2 ' aetheros-serial.txt | cut -d' ' -f2 | tr -d '\r\n' | xxd -r -p > receipts.bin
./aetherproof-verify-x86_64 receipts.bin --pubkey <boot signer public key>
```

The verifier prints each receipt and `chain of N receipts: VALID`. It exits with 0 for
valid, 1 for invalid (and names the first bad receipt), 2 for a usage or file error.
`--binary <file>` additionally checks that a given program file matches a receipt.

## 9. Writing your own program

Programs are written in VELA assembly. Example (`tools/apps/sum.aeis`):

```
; sum: 1 + 2 + ... + 100 = 5050
_start:
    MOVI    R1, 0           ; total
    MOVI    R2, 100         ; i
loop:
    ADD     R1, R1, R2
    ADDI    R2, R2, -1
    BNEZ    R2, loop
    RET
```

Rules:

- The result is the value in **R1** when the program returns.
- Registers are **not** cleared at start: set every register before reading it.
- **R0 is always zero**; `BEQZ R0, label` is an unconditional jump.

Steps (development build):

```sh
tools/asm/target/release/aether-asm -o kernel/apps/myapp.code tools/apps/myapp.aeis
tools/run-qemu-linux.sh        # the build packs and signs every file in kernel/apps/
```

Then `run myapp` in the console. For a release, `tools/release.sh` signs all programs with
the release key.

## 10. Building from source

Requirements: Linux, Rust nightly (pinned by `kernel/rust-toolchain.toml`), QEMU,
`python3`, `openssl`, and for the ISO `grub-pc-bin xorriso mtools`.

| Command | Result |
|---|---|
| `tools/run-qemu-linux.sh` | Development build with all self-checks, console on the terminal |
| `tools/run-qemu-linux.sh --test` | The same, without a window; fails on any failed check |
| `tools/release-key.sh` | Once: create the release key in `~/.aetheros-keys` (passphrase protected) |
| `tools/build-iso.sh` | Release ISO (requires the release public key) |
| `tools/release.sh` | Full signed release in `build/release/` |
| `tools/export-public.sh` | Clean public source tree with a fresh history |

Development builds trust a **published test key** and run the self-checks at start-up.
Release builds trust only the release key, contain no test key and start directly at the
prompt. Scripts never delete earlier output; it is moved to `build/old/`.

## 11. Security model

**Trusted:** the kernel image (identified by its build id), the release public key built
into it, and the cryptography libraries (ed25519-dalek, sha2, ml-dsa).

**Not trusted:** every program. Programs are checked at admission, confined at run time
and recorded afterwards.

| Protection | Effect |
|---|---|
| Signed-only admission | Unsigned or modified programs are refused |
| Instruction scan | Privileged instructions in user programs are refused |
| Sandbox | Programs cannot reach memory outside their 4 MiB area |
| Write-xor-execute | Translated code is never writable and executable at the same time |
| Shadow-stack check | A return to a manipulated address stops the program |
| Fuel limit | Endless programs are stopped |
| Kernel page isolation | Separate page tables for kernel and user mode |
| Receipt chain | Missing, reordered, inserted or altered records are detected |

Report security problems as described in `SECURITY.md`.

## 12. Interfaces: today and planned

### 12.1 Today (0.92)

AetherOS has a **text console**, shown on the screen and copied to the first serial
port. The serial copy is how you move output, such as exported receipts, to another
computer. There is no network service yet; receipts leave the machine through the serial
file.

### 12.2 Planned

These are design goals, not features of 0.92.

| Planned item | Description |
|---|---|
| Graphical console | A pixel display with the AetherMono typeface drawn for AetherOS, replacing the text mode |
| Desktop | A monochrome window system with a pointer: terminal, file viewer and receipt viewer |
| Mouse | PS/2 mouse support for VirtualBox |
| Persistent disk | An AHCI driver so files and receipts survive a restart in VirtualBox |
| UEFI start-up | Booting on current PCs and from USB media |
| Connecting to other systems | Sending receipts to a public, append-only log over the network, and a gateway mode in which a conventional system sends jobs to AetherOS and receives results with receipts |
| Quantum-safe receipts | A post-quantum signature (ML-DSA) on the receipt chain |
| Device key | A key sealed in the TPM so receipts identify the machine |

## 13. Troubleshooting

| Problem | Cause and solution |
|---|---|
| VirtualBox: "kernel driver not installed" or vboxdrv not loaded | Secure Boot blocks the unsigned module. Sign it (MOK) and run `sudo /sbin/vboxconfig`, or disable Secure Boot |
| VirtualBox: "VT-x is being used by another hypervisor" | Unload KVM before starting: `sudo modprobe -r kvm_intel kvm` |
| Black screen, no prompt | EFI is enabled. Switch the virtual machine to BIOS |
| Typing does nothing | Click inside the VirtualBox window so it captures the keyboard |
| VirtualBox starts an old session | Discard the saved state: `VBoxManage discardstate AetherOS` |
| `vfs-save` reports no device | Expected in VirtualBox (no virtio disk) |
| `run` reports "refused" | The program's signature does not match this build's trusted key |

## 14. Known limits

- Runs in QEMU and VirtualBox with BIOS firmware only.
- Files are lost at power-off.
- Receipt signing keys are generated at each start-up.
- The control-flow hash is shared by all processes; exact re-execution of a run is not
  yet possible.
- Some VELA-3 instructions are not yet supported by AetherBridge (finding F-011).
- No graphical interface, user accounts or network services.
- The full list of open findings is in `tests/findings/FINDINGS.md`.

## 15. Glossary

| Term | Definition |
|---|---|
| Build id | SHA-256 identifying the exact kernel source and build |
| Chain | Receipts linked by the hash of each previous receipt |
| Ed25519 | The signature scheme used for programs and receipts |
| Fuel | Run-time budget counted in backward jumps |
| ML-DSA | A post-quantum signature scheme (FIPS 204) |
| SFI | Software fault isolation: confining a program's memory accesses |
| W^X | Memory is either writable or executable, never both |
