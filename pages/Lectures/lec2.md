---
layout: default
title: Lecture 2
parent: Lectures
---

## Lec 2

What is life other than to suffer?

## Bootloaders

Bootloaders are a magnificent beast.

## Supervisor Binary Interface (SBI)

The RISCV SBI sits between the bootloader and the kernel. It is used as a platform agnostic way of doing basic setup code before loading a kernel image into memory and handing off control to it

In riscv fashion, the bootloader should set up an SEE (Supervised Execution Environment) for which the kernel code can run in. The kernel is simply a program and if it just ran on bare metal with hardly any abstractions, it wouldnt be great to code for. So instead the bootloader sets up some abstractions in memory and inspects ACPI for devices to create data structures which are more helpful for the kernel code to use.

THe kernel interacts with the SBI interface through these fields:

```c
// return value of all SBI functions
struct sbiret {
    long error; // 0,-1,...-8
    long value;
}
```

- when calling `ecall`, store the SBI extension ID within `a7`. Should also store the SBI function ID in `a6`

There is an implementation of SBI that runs on M mode, [here](https://github.com/rustsbi/rustsbi)

- should be used as a dependency for your bootloader/kernel

### SBI Extensions

A bunch of very nice extensions like system timers, console reads and writes, fence instructions, shutdown the system, and functions to get the SBI extensions available for a kernel to use

- around 11 total extension sets available. Only 7 or so seem noteworthy for a standard implementation. Some of them are experimental and some are made for vendors to extend themselves. There are also some that are specific to the firmware itself
- a legacy set exists for kernels that have older designs. Its not a bad idea to implement
- The most important ones seem to be: **Base**, **Timer**, **IPI** (interprocess interrupt), **RFENCE** (Instruction execution or memory accesses are forced to execute in a specific order, without out-of-order stuff), **HART state management** (start, stop, status, suspend), **reset system**, **performance monitoring** of hardware like disk, cpu, ram, gpu, etc (stuff like list of events, counters)

## RISC-V Interrupt Controllers

An interrupt controller directs hardware generated interrupts to the CPU as a formatted request.

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

```
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

- CSR2 for x86. Supervisor mode only, no "machine mode"
- For RISCV, `m` instructions like `mie` to enable interrupts and `mcause` to specify an interrupt code
- `s` for supervisor mode, i.e. kernel mode. Basically machine mode but with virtual addressing available. MMU is functional and TLB flushing is possible
- `h` for hypervisor mode. Basically a minimal translation scheme with minimal overhead, much better than a full software emulator. Also an extra layer for address translations. The `h` instructions would be able to efficiently use the host resources without affecting other guests. They do so by 'almost' running on bare metal, in a specialised "hypervisor" environment with extra containerisation. More info on [hardware assisted virtualisation](https://en.wikipedia.org/wiki/Hardware-assisted_virtualization).

## Interrupt Instruction

For a userspace application to invoke a syscall in riscv, they use the `ecall` instruction which jumps into supervisor mode. ECALL = Executive Call.

- This normally starts by storing the registers (app) on the stack. Note by making an `ecall` instruction, we went from virtual addressing to physical addressing within the kernel. So we are storing the app's registers on the kernel stack, usually starting at a relatively high frame addr. Note, some services wont need to use registers at all and only manipulate memory directly. In this case the registers do not need to be saved.
- It then captures specific registers like `pc` and the current privilege level, and set the cause of the interrupt from the syscall API register to the `mcause` register. Then it checks if the interrupt was a page fault, which means `mtval` reg needs to hold the virtual address which the process tried to access.
- Now it can turn off interrupts and look up an IDT for the interrupt number and handler function. Then it simply jumps to that interrupt handler function
- Within the interrupt handler function, it would do what it needs to, usually some IO task. From first principles, it is possible to wait on the IO to finish on another kthread and resume execution of the userspace program if it has other things to do too. Or else simply call the service on another user thread, which would be handled by a different kernel thread
- After the service is completed, the kthread restores the app's registers from the kernel stack and transfers execution back to the user thread that called the service. It would need to also restore the PC and other things that have been changed within the kernel

There can also be conventions for programs to not use a specific register at all or expect to be always temporary so a quick service can simply use that one. Hence the handler would not have to store all the registers

## Hypervisors

For VMs, we could instead code a userspace hypervisor that provides a minimal translation layer from the VM syscalls to the host syscalls. This has the advantage of being simpler to code and run since you already have a running OS that you know is working.

Note `h` is built for type 1 hypervisors which benefit from a minimal host layer and almost bare metal. A vm running on a type 1 hypervisor will still have to make syscalls and interrupt into a privileged mode, so its not a complete do-all kind of thing. For some applications like XBox, it could make sense to run multiple apps as separate vm containers on a type 1 hypervisor. But on a normal PC being used by a single person for multitasking and performance related things, and only cares about speed and reliability, its not really something that makes sense to replace over a `supervisor` mode.

### Type 1 Hypervisors

- A type 1 hypervisor is more efficient and possibly more secure since there is a very limited gap between the VM and the hardware that an attack can exploit to compromise the system
- But it is usually more of a hassle to set up and may require another machine to manage it well. Windows Hyper-V is great at doing this on its own platform, (I'll add more on this later)

### Type 2 Hypervisors

- Basically an app in userspace. Can be quicker and easier to setup/access than type 1 vms. For normal users that may want extra safety/security for certain tasks and if they want to test out something. Would be great to test out an OS on a type 2 hypervisor as it is quite safe (Note "safety" doesnt equal "security")
- Even though in practice would be safer and even more secure in some areas, has more gaps between it and the hardware as an attacker can go for the host OS as well. Also never as fast as a type 1 hypervisor, should expect considerable latency with services and devices

## What a bootloader and kernel not have to expose?

The kernel's internal ABI for managing processes and communicating with other kernel modules can be completely obfuscated and randomised. But if we want to expose services to userspace processes or ask the bootloader for a service, we will have to use a clearly defined structure of communication

- an interrupt with a0..7 registers and x0..7 registers. Can maybe expect t0..7 registers to be reset
- perhaps a memory mapped 'virtual dynamic library' which you can link your ELF program to before running
- C-repr structs to hold data for back and forths communication, like JSON

In most cases, user-kernel communication occurs through interrupts. For really 'quick' and 'safer' services we could use memory mapped objects that contain the function definition

- for user-user communication, we would prob use some userspace bus and a C-repr struct. The bus would be managed by a kernel module or sparx. Sending a request would be basically an interrupt to the kernel sparx on another thread

For kernel-bootloader communication, we wouldnt have a direct interrupt but rather a direct function call of the SBI functions using `call sbi_function` within kernel code. The structs would also be C-repr, but mostly because we want a stable interface

## RISC-V Architecture

Lets talk about RISC-V architectural features

- HART = "Hardware Thread" in RISC-V

## Assembly Line Architecture

They call it 'Pipelining', but in COMP6900, I will be referring to it as an 'Assembly Line'

### 5 Stage RISC Pipeline

Each HART (hardware execution thread) is pipelined

![](/assets/img/RISC_Pipeline.webp)

- Image taken from [All About Circuits](allaboutcircuits.com)

This means each stage is running at the same time. While an instruction is being executed in the ALU, another instruction waiting can be fetched. The previously fetched instruction can be decoded. And the oldest instruction in the assembly line may be undergoing writeback.

### Hardware View

Inside a processor thread, the pipeline looks like:

![](/assets/img/RISC_Pipeline_hardware.jpg)

- Image taken from [GA Tech](faculty.cc.gatech.edu/~hyesoon/fall10/prog1.html)

Within a HART, the program counter increments or jumps to an instruction address. It then goes to its L1 Instruction Cache and queries for an address of PC. If exists, then can continue onto decoding.

- If not in the I-Cache (e.g., program too big to fit inside cache), then tell the Fetcher to get the 32-bit instruction at address PC from RAM. This can take some time as other stages of the pipeline may have already finished doing what they are doing, so if it does happen, we call this a cache-miss due to either poor spatial/temporal locality

Next, the 32-bit instruction is sent to the decoder, which figures the format, i.e. UJ, B, etc. and the source/destination registers. If any immediates, then consider that too since we wont be doing arithmetic on register-register by register-line.

- Before execution, we have to check if the register file is ready (i.e. no current operations). If not ready, will have to stall/busy wait until ready
- Note within the CPU itself, the lines are very high speed, prob a magnitude higher than cpu-ram bus. Also with a risc processor we always have load-store unlike x86 which is technically RISC at the hardware level too, just that the decoder does more to load from RAM into a register, then work on it. This means most instructions will prob do something to 1-2 HART registers

The execution stage may take more cycles if we have a branch instruction or load/store from/to memory instruction. There should be a single ALU(64) and maybe FPU(32) to handle float ops. These units will handle the arithmetic and return the result to a register. If an instruction would compare registers then branch, the execution part may need to send the source register values to the ALU for comparison, e.g. `=` or `>=`. Then depending on the result, a branch is chosen, either stay or jump to the dest address in another register. If jumping, then a branch unit will take that register's value and handle the branch.

- Branch prediction will also be done in this stage. Most programs will choose one brancgh 90% of the time. The processor can see this and give more weight to a certain branch (instruction address)
- Branch prediction is very useful since you wont have to 'slow down' the entire pipeline to wait for the branch to take place. This is because you would have to wait for all previous instructions to complete so you can make the full jump.
- Instead you can just branch immediately on the fly based on past branches. If you get it right, the pipeline can keep going as the instructions are correct (given right out of order execution implementation)
- But if you get it wrong, that can be a problem. You will have to flush the entire pipeline and start from fresh at the other branch. You will get rid of at least 4 assembly cycles, but at least you have the right branch now and the pipeline didnt 'halt' per se, rather just reset
- If you get it right >=90% of the time, it is pretty good in practice. Much better than halting execution for the other instructions to complete, then branching.

After the instruction has been executed, the result should be in a certain `dest` register. If the instruction was to branch, the branch target address can be forwarded to the PC here.

- If the instruction was to store a value into RAM, the L1 Data Cache will be accessed in order to see whether any addresses matches the `dest` address (value). If so, then the data can be written to the cache instead. If not, then we can check any higher order caches like L2 and L3, and write to those.
- If those caches dont contain the requested RAM address, then we can send the least recently used entry to L2/L3/writeback/RAM and add the RAM address to L1. L2/L3 should cascade in the same way based on least recently used. As you can see, it would prob be a good idea to have good temporal locality and a program that uses only a small amount of variables in RAM to do what it does.
- For programs that store complex structures and really long lists, then maybe its not worth it to cache the array/database in the CPU. Or maybe only cache the most used ones.
- If the instruction was to load a value from RAM into the register, it should be done in the execution stage but interact with the `mem` stage as well like a dual-stage instruction

Finally, we reach the writeback stage. Here the instruction has been done processing. The results

### Cache Entries

To ensure we always have the most up to date entries, we also associate a timestamp of when an entry was updated. In cases of small programs, we prob dont need the RAM at all, just use the data cache.

- There should be no risk of overwriting any new data in RAM since we are doing everything in the CPU, which should know which RAM addresses are being touched.
- It might a bit more complex with multiple HARTs since we will have to check each cache, and use L2/L3 caches to duplicate the data in the local L1 Data caches
- Even more if we are using virtual addresses. Each process has their own virtual address space. So on process switch, we will have to flush the data caches to RAM. The instruction caches can just be cleared
- Also applies when we start a new process or end a process. This also assumes one process per hardware thread. But maybe we are using two hardware threads on a single program. This means two HARTs share the same virtual address space. Should be fine if L2/L3 is updated correctly and mutexes are done properly.

TBH Im not that enlightened here

### Memory Management Unit

The MMU is reponsible for translating virtual addresses to physical addresses. Assumes some page table-like structure in memory with CS registers that store a pointer to its physical address.

- Interacts with the TLB beforehand to see whether a virtual - physical mapping already exists. If so, then use that.
- If a page isnt in there, it will have to fetch from RAM's page table and place the entry into the TLB using LRU heuristics.
- If allocating/deallocating pages, will prob need to update the TLB. Maybe flush it always.
- Used on load/store instructions as well as any program counter instruction lookups. Can be quite expensive so having more TLB entries does make sense.

## Rings

There are rings 0-3 from least privileged to most privileged.

- This is different to x86 where the rings go from most privileged to least privileged
- Usually, only ring 3 and 0 are used, i.e. kernel-mode and user-mode
- This is a bit different to actual "modes"

## Execution Modes

There are 4 'modes' of execution which kind of correspond to rings.

Mode M -> most privileged. Any execution can be executed.

Mode S -> kinda actually the most privileged. Has extra structures and concepts to allow virtual addressing.

Mode U -> user mode, least privileged.

Mode H -> hypervisor mode, gives you more instructions to translate from a virtual machine's addresses.
