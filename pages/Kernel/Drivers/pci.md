---
layout: default
title: PCI
parent: Drivers
grand_parent: Kernel
---

## PCI Overview

A 'local' computer bus for attaching hardware devices. Part of the PCI local bus standard.

- allows hardware to be attached independent of the system's arch, whether it be x86, arm, riscv, powerpc, mips, etc.
- a network like topology for inter hardware communication between PCI devices and PCI-motherboard devices like the CPU (bridge) and RAM (mmio controller)

### Slot

The PCI hardware slot is either 3.3V or 5V. It has 2,3 or 4 key positions to attach into the adapter.

![](/assets/img/uefi/PCI_hardware_slot_spec.png)

### Bus Transitions

On a board there is a 'bus arbitrater'. Any PCI device may initiate a transaction, e.g. it wants to send a single message to another device.

1. It will first request permission from a PCI bus arbiter, which should queue the request
2. Arbiter grants permission to one of the requesting devices (e.g. FIFO)
3. Initiator begins address phase by broadcasting a 32 bit address + 4 bit command code. It then `sleep()` or busy waits for a response
4. Each other device is sent the address to see if it is directed at them. Only one device should fit the address and respond a few cycles later

### 64-bit Addressing

The bus may only be 32bits wide on a 32-bit CPU system. Hence the initiator will first broadcast the least sig 32bits + a 'dual' command code to signal 64bits. The next cycle it broadcasts the next 32bits + the actual command.

Then the transaction happens identically.

NOTE: it is possible for bytes to be 'enabled'. These bytes are 'significant' and the other bytes may be 'nop' instructions to pad space.

### Command Codes

We have 4 bits for the command code right after the address. Here are the possible ones:

```rust
enum PCICommandCode {
    InterruptAcknowledge = 0x0
    SpecialCycle = 0x1
    IORead = 0x2
    IOWrite = 0x3
    Reserved = 0x4, 0x5, 0x8, 0x9 // pretty much nop
    MemoryRead = 0x6
    MemoryWrite = 0x7
    ConfigRead = 0x10
    ConfigWrite = 0x11
    MemoryReadMultiple = 0x12
    DualAddressCycle = 0x13
    MemoryReadLine = 0x14
    MemoryWriteCacheInvalidate = 0x15
}
```

- the 'line' refers to cache stuff in the pci config space

### Bus Latency

Target must be able to complete the initial data phase.
