# AetherOS

> A security-first operating system kernel where security enforcement is proven at translation time, not checked at runtime.

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/yourusername/aetheros)
[![Tests](https://img.shields.io/badge/tests-518%2F518-brightgreen)](https://github.com/yourusername/aetheros)
[![Phase](https://img.shields.io/badge/phase-75%2F75-blue)](https://github.com/yourusername/aetheros)
[![Platform](https://img.shields.io/badge/platform-x86--64%20%7C%20AArch64-lightgrey)](https://github.com/yourusername/aetheros)

---

## What is AetherOS?

AetherOS is a from-scratch OS kernel written in Rust targeting x86-64 (and partially AArch64/RPi4). Every user program executes through **AetherBridge** — a mandatory JIT translation gate. Security guards are emitted into the output binary at translation time. The kernel mathematically proves, using interval analysis on register value ranges, which guards are redundant and elides them.

The result: security that cannot be bypassed at runtime because it was proven correct before the code ever ran.

**Not a simulation. Boots on real x86-64 BIOS hardware from a USB stick.**

---

## Core Architecture

```
User Program (VELA ISA bytecode)
         │
         ▼
  ┌─────────────────┐
  │  AetherBridge   │  ◄── JIT translation gate (mandatory)
  │  (JIT Compiler) │       emits x86-64 with security guards
  └────────┬────────┘
           │ Value-Range analysis (elides provably-safe checks)
           ▼
  ┌─────────────────┐
  │  x86-64 native  │  ◄── security is a translation property,
  │  machine code   │       NOT a runtime check
  └─────────────────┘
```

### Key Properties

| Property | Description |
|---|---|
| **Mandatory translation** | No direct native execution — all code passes through AetherBridge |
| **VELA ISA** | Custom 32-bit RISC instruction set (VELA-1 scalar + VELA-2 SSE2 vectors) |
| **Capability system** | Process capabilities gate filesystem access, network, crypto opcodes |
| **IronVFS** | Cryptographic RAM filesystem — Ed25519 per-file signatures, per-block FNV hash |
| **ChronoTag** | Physical memory write-count tracking; detects data exfiltration at kernel level |
| **ISA-ASLR** | Per-process randomised register→x86 mapping; defeats ROP chain reuse |
| **SMP** | 4-core LAPIC bringup, IPI shootdown, PCID-based TLB flush |

---

## Status: Phase 75 Complete

| Component | Status |
|---|---|
| x86-64 boot (multiboot2) | ✅ Boots on real hardware |
| 64-bit paging (4-level) | ✅ |
| Heap allocator | ✅ |
| VELA-1 JIT (scalar ops) | ✅ ~70 opcodes |
| VELA-2 SSE2 vectors | ✅ 11 opcodes (V.ZERO–V.GET.W) |
| VELA-3 security opcodes | ✅ CHAOS polymorphism, TIME.BIND, PROOF.PATH |
| 25 POSIX-compatible syscalls | ✅ |
| IronVFS + Ed25519 signatures | ✅ |
| virtio-blk disk persistence | ✅ |
| Intel e1000 Ethernet | ✅ (polling) |
| VGA textmode + PS/2 keyboard | ✅ |
| SMP 4-core LAPIC | ✅ (APs park at idle) |
| AArch64 / RPi4 partial port | ✅ (boots to shell) |
| **Total verified assertions** | **518/518** |

---

## Building

### Prerequisites

```bash
# Install Rust nightly with bare-metal target
rustup install nightly
rustup target add x86_64-unknown-none
rustup component add rust-src --toolchain nightly

# QEMU for testing
# Ubuntu/Debian: sudo apt install qemu-system-x86
# macOS: brew install qemu
```

### Build & Run

```bash
git clone https://github.com/yourusername/aetheros
cd aetheros/kernel

# Build (dev — includes sig_bypass for testing)
cargo build

# Build hardened (production — no bypasses)
cargo build --release --no-default-features

# Run in QEMU (requires aetheros-kernel ELF + disk image)
qemu-system-x86_64 \
  -kernel target/x86_64-unknown-none/debug/aetheros-kernel \
  -drive file=disk.img,format=raw,if=virtio \
  -serial stdio -display none -m 128M
```

### Run Tests

All 518 assertions run at boot. Check serial output for `[OK]` / `[FAIL]`:

```
cargo build && qemu-system-x86_64 -kernel ... -serial file:serial-out.txt -display none
grep -c '\[OK\]' serial-out.txt   # should print 518
grep '\[FAIL\]' serial-out.txt    # should be empty
```

---

## VELA ISA Quick Reference

VELA is a 32-bit fixed-width RISC ISA. Instruction format:

```
 31      24 23    18 17    12 11     6 5      0
 ┌─────────┬────────┬────────┬────────┬───────┐
 │ opcode  │   Rd   │  Rs1   │  Rs2   │  imm6 │
 └─────────┴────────┴────────┴────────┴───────┘
```

**Scalar (VELA-1):** R0–R15 → x86-64 GPRs. R0 = zero register. Arithmetic, memory, branch, syscall.  
**Vector (VELA-2):** VR0–VR7 → XMM0–XMM7 (SSE2). V.ADD.D/Q, V.AND/OR/XOR, V.LOAD/STORE, V.SCALAR.W, V.GET.W.

---

## Security Model (Summary)

1. **Code cannot execute without translation** — AetherBridge is enforced at the syscall boundary.
2. **CFI (Control Flow Integrity)** — indirect calls are checked against a valid target bitmap.
3. **SHADOW.CHK** — stack frame integrity guard emitted into every function prologue.
4. **Capability-gated opcodes** — filesystem, network, and crypto opcodes require explicit capability tokens.
5. **Ed25519 binaries** — executables are signed; kernel verifies before loading.
6. **ChronoTag** — kernel-level write-count monitor for physical pages (AI weight protection).

> **Note on gaps:** TCP is a stub. Filesystem data is signed but not encrypted at rest. FIPS 140-3 validation is not done. These are planned for the next phase.

---

## Directory Structure

```
kernel/
├── src/
│   ├── arch/vela1/     # x86-64 boot, GDT, IDT, TSS, LAPIC, syscalls
│   ├── drivers/        # uart, vga, ps2, pci, e1000, virtio-blk/net
│   ├── mem/            # frame allocator, paging, heap, slab, ChronoTag
│   ├── proc/           # scheduler, process table, context switch
│   ├── fs/             # IronVFS cryptographic filesystem
│   ├── translate.rs    # AetherBridge JIT compiler (VELA→x86-64)
│   ├── bridge.rs       # Translation entry point
│   ├── exec.rs         # AELF binary loader + Ed25519 verify
│   ├── net/            # ephemeral network stack (stub TCP)
│   ├── crypto/         # ChaCha20, Ed25519 wrappers
│   ├── disk/           # DarkDisk encrypted block layer
│   └── main.rs         # Kernel entry + 518 test assertions
├── Cargo.toml
└── x86_64-unknown-none.json  # bare-metal target spec
```

---

## Research Background

AetherOS explores one core question:

> *Can security checks be moved from runtime enforcement to translation-time proof, eliminating the class of attacks that bypass runtime checks?*

The answer: partially yes. The Value-Range Coloring (VRC) algorithm proves at translation time that specific memory accesses are safe, and elides the CFI guard for those accesses. This is verified in the test suite. The gap is that VRC currently only handles simple interval cases — a complete solution requires a full abstract interpreter.

---

## License

The public interface, documentation, and test harness in this repository are released under **MIT License**.

The core JIT translation algorithms (AetherBridge internals, VRC, ChronoTag) are **Patent Pending**. Contact for licensing.

---

## Contact

Built by [Pulkit Srivastava](mailto:pulkitsrivastavae@gmail.com) — Durgapur, West Bengal, India.

*Feedback, collaboration, and pilot deployments welcome.*
