# AetherOS Architecture Overview

## High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│                      User Programs                       │
│              (VELA ISA bytecode, signed AELF)            │
└────────────────────────┬────────────────────────────────┘
                         │ exec() syscall
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    AetherBridge                          │
│              (Mandatory JIT Translator)                  │
│                                                          │
│  1. Verify Ed25519 signature                             │
│  2. Translate VELA → x86-64                              │
│  3. Emit security guards inline                          │
│  4. Prove safe guards redundant → elide them             │
└────────────────────────┬────────────────────────────────┘
                         │ x86-64 machine code
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    Kernel Services                       │
├──────────────┬──────────────┬──────────────┬────────────┤
│   IronVFS    │  Scheduler   │  SyscallGate │  ChronoTag │
│  (crypto FS) │  (RR, 4-core)│  (25 syscalls│  (mem mon) │
└──────────────┴──────────────┴──────────────┴────────────┘
```

## Component List

### AetherBridge (JIT Compiler)
- Input: VELA-1/2/3 32-bit fixed-width instructions
- Output: x86-64 machine code with inline security guards
- Security features: CFI, SHADOW.CHK, capability checks, time-bound values

### VELA ISA
- **VELA-1:** 32-bit RISC scalar (R0–R15, ~70 opcodes)
- **VELA-2:** SSE2 vector extension (VR0–VR7 = XMM0–XMM7, 11 opcodes)
- **VELA-3:** Security opcodes — capability-gated, CHAOS polymorphism, execution fingerprint

### IronVFS
- 64-slot RAM filesystem, virtio-blk persistence
- Per-file Ed25519 signatures
- Per-block FNV-1a hash integrity
- Capability-gated open (Fs cap prefix must match filename)

### Memory Management
- 4-level paging, 2MB HHDM
- Bitmap frame allocator
- linked_list_allocator heap
- Slab allocator for fixed-size kernel objects
- **ChronoTag:** per-physical-page write counter (exfiltration detection)

### Drivers (Phase 70–75)
| Driver | Interface | Status |
|--------|-----------|--------|
| UART 16550 | I/O port | ✅ Full |
| VGA textmode | memory-mapped | ✅ Full |
| PS/2 keyboard | I/O port, SPSC ring | ✅ Full |
| Intel e1000 NIC | PCI/MMIO, polling | ✅ Full |
| virtio-blk | PCI legacy, polling | ✅ Full |
| virtio-net | PCI legacy | ✅ Stub |
| PCI scanner | config space | ✅ Full |

### Platform Support
| Platform | Boot | Drivers | Status |
|----------|------|---------|--------|
| x86-64 BIOS/QEMU | ✅ | ✅ Full | **Primary** |
| x86-64 real hardware | ✅ USB boot | ✅ | Tested |
| AArch64 / RPi4 | ✅ | PL011, GICv2 | Partial |

## Test Coverage

518 assertions across 75 phases run at boot time.  
Tests cover: memory subsystems, syscalls, JIT translation, VFS operations,
driver I/O, cryptographic primitives, SMP, networking, security enforcement.

See `serial-out.txt` for full test output from last QEMU run.
