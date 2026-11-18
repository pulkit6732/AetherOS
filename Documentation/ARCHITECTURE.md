# AetherOS architecture

This document describes the structure of AetherOS 0.92.0: its layers, the path a program
takes, the trust model and the boot sequence. Detailed design notes and status history
are in `ARCHITECTURE.html`.

## 1. Layers

```mermaid
flowchart TB
    subgraph User["User space (untrusted)"]
        APP[Signed VELA programs]
    end
    subgraph Core["Kernel (trusted, Rust, no_std)"]
        SHELL[Shell]
        GATE[Admission gate]
        BRIDGE[AetherBridge translator]
        PROOF[aetherproof-core]
        VFS[IronVFS]
        SCHED[Scheduler and fuel]
        MEM[Memory manager]
        DRV[Drivers]
    end
    subgraph HW["Platform"]
        CPU[x86-64 CPU]
        DEV[PS/2, PCI, e1000, virtio-blk, serial, VGA]
    end
    APP --> GATE --> BRIDGE
    SHELL --> GATE
    SHELL --> VFS
    BRIDGE --> MEM
    GATE --> PROOF
    SCHED --> CPU
    MEM --> CPU
    DRV --> DEV
```

| Component | Responsibility | Source |
|---|---|---|
| Admission gate | Ed25519 signature check, AELF parsing, instruction scan | `kernel/src/exec.rs` |
| AetherBridge | VELA to x86-64 translation, sandbox confinement, runtime checks, fuel gates | `kernel/src/bridge.rs` |
| aetherproof-core | Receipt format, hash chain and verification, shared with the host verifier | `aetherproof-core/` |
| IronVFS | In-memory file system with per-block integrity hashes | `kernel/src/fs/ironvfs.rs` |
| Memory manager | Page tables, write-xor-execute, kernel page isolation, sandbox arena | `kernel/src/mem/`, `kernel/src/arch/vela1/kmap.rs` |
| Shell | Commands and manual pages | `kernel/src/shell/` |

## 2. Program path

```mermaid
sequenceDiagram
    participant U as User
    participant S as Shell
    participant G as Admission gate
    participant B as AetherBridge
    participant X as Contained execution
    participant P as aetherproof-core
    U->>S: run primes
    S->>G: signed AELF image
    G->>G: verify signature, parse, scan
    G->>B: verified code
    B->>B: translate, confine, add checks and fuel gates
    B->>X: machine code
    X->>X: run until return, fault or fuel stop
    X->>P: run facts and hashes
    P->>P: sign receipt, link to previous
    P-->>S: receipt number
    S-->>U: result (R1) and receipt number
```

## 3. Trust model

```mermaid
flowchart LR
    RK[Release private key<br/>kept by maintainer] -->|signs| APPS[Program images]
    RK -->|signs| SUMS[SHA256SUMS]
    RPK[Release public key] -->|built into| K[Kernel]
    K -->|admits only programs signed by| APPS
    K -->|generates at start-up| BK[Receipt key]
    BK -->|signs| RCPT[Receipts]
    BID[Build id<br/>SHA-256 of source and toolchain] -->|stamped into| RCPT
```

- The kernel trusts one admission key, fixed at build time. Release builds refuse to be
  built with the published development key.
- Receipt keys are generated at each start-up from hardware randomness. A key sealed in
  the TPM, which would identify the machine, is planned.
- Receipts name the kernel by its build id, so a verifier can rebuild the kernel from
  source and compare.

## 4. Boot sequence

```mermaid
flowchart TB
    A[GRUB loads kernel<br/>Multiboot2] --> B[Long mode, page tables]
    B --> C[Kernel map hardening<br/>W^X, NX heap]
    C --> D[Interrupts, timer, PCI scan]
    D --> E[Drivers: serial, VGA, PS/2, e1000, virtio-blk]
    E --> F[Late initialisation<br/>IronVFS, SMP, KPTI, TPM probe, receipt key]
    F --> G[Shell prompt]
```

Development builds run the self-test suite (about 700 checks) between steps E and F.
Release builds go directly to the prompt.

## 5. Receipt format (v2, 280 bytes)

| Bytes | Field |
|---|---|
| 0-3 | Magic `APR2` |
| 4 | Version (2) |
| 5 | Flags (bit 0: key derived from measured TPM PCRs, never set in 0.92; bit 1: weak entropy) |
| 6-7 | Reserved (0) |
| 8-15 | Sequence number |
| 16-19 | Process id |
| 20-23 | Syscall count |
| 24-31 | Timestamp |
| 32-39 | Program length |
| 40-47 | Control-flow hash |
| 48-55 | Memory hash |
| 56-87 | Program SHA-256 |
| 88-119 | AetherBridge output SHA-256 |
| 120-151 | Kernel build id |
| 152-183 | Previous receipt SHA-256 |
| 184-215 | Signer public key |
| 216-279 | Ed25519 signature over a fixed domain string and bytes 0-215 |
