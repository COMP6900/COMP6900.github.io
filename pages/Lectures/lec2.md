---
layout: default
title: Lecture 2
parent: Lectures
---

## Lec 2

What is life other than to suffer?

# Bootloaders

Bootloaders are a magnificent beast.

## RISC-V Interrupt Controllers

We start with an interrupt controller which directs hardware generated interrupts to the CPU as a formatted request.

### Local Interrupt Controllers

SiFive provides two designs: a 'core local interrupter' or a 'core local controller'. Both require the `mtime`, `mtimecmp` registers for timer interrupts, and `msip` to configure software generated interrupts.

Core local interrupters or CLINT:

- a compact, fixed priority design. Support for preemptive interrupt controls from high privilege levels of HART
- simple interrupter for software and timer interrupts, does not control other local interrupts

Core local interrupt contoller or CLIC:

- fully featured with config options. Supports programmable interrupt levels and priorities
- supports nested, preemptive interrupts. Based on interrupt level and priority configuration. And within a given privilege level

### Global Interrupt Controllers

The PLIC provides flexibility to multiple CPUs on a system. For multiple HARTs, you will need a PLIC to handle higher level interrupts that affect multiple threads or have complex functionality like randomised handling.

- Runs on a different clock compared to the Local controllers
- Each global interrupt has a priority programmable register. The registers have a unique ID to map to an interrupt.

## Interrupt Handler

Example: Within the Controller Firmware

```arm
.align 2
.global handler_table_entry

handler_table_entry:

// Push x1..x31 onto the stack (of caller)
// REGBYTES => 4/8 Bytes depending on 32I/64I
addi sp, sp, -32 * REGBYTES
STORE x1, 1 * REGBYTES(sp)
STORE x2, 2 * REGBYTES(sp)
...
STORE x30, 30*REGBYTES(sp)
STORE x31, 31*REGBYTES(sp)

//---- call C Code Handler ----
call software_handler
//---- end of C Code Handler ----

// Reload x1..x31 from stack
LOAD x1, 1*REGBYTES(sp)
LOAD x2, 2*REGBYTES(sp)
...
LOAD x30, 30*REGBYTES(sp)
LOAD x31, 31*REGBYTES(sp)
addi sp, sp, 32*REGBYTES
// return to jalr instruction
mret
```

Handler for CLINT:

```c
#define MCAUSE_INT_MASK 0x80000000
#define MCAUSE_CODE_MASK 0x7FFFFFFF

void software_handler() {
    unsigned long mcause_value = read_csr(mcause);
    if (mcause_value & MCAUSE_INT_MASK) {
        // Branch to interrupt handler here
        // Index into 32-bit array containing addresses of functions
        async_handler[(mcause_value & MCAUSE_CODE_MASK)]();
    } else {
        // Branch to exception handler
        sync_handler[(mcause_value & MCAUSE_CODE_MASK)]();
    }
}
```

## Interrupt CSR
Control and Status Registers are heavily used for interrupt logic and setup.
- CSR2 for x86

# RISC-V Architecture

Lets talk about RISC-V architectural features

## Assembly Line Architecture
They call it 'Pipelining', but in COMP6900, I will be referring to it as an 'Assembly Line'


