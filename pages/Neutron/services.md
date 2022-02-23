---
layout: default
title: Services
parent: Neutron
---

## Neutron Services

A kernel sits between untrustworthy apps and trustworthy hardware. Not all hardware can be trusted, e.g. peripheral firmware, but that is also software.

- To ensure apps dont do anything stupid or malicious, we need ways to structure and control access to hardware. Bad software cause bad harware execution and possibly corruption of data and the hardware itself. Worse, it could cause a physical hazard, e.g. hospital software
- **Memory Control**: we want to control access to RAM and VRAM. We dont want processes to be able to write over other process's data, or worse, the kernel's data which they will then be able to do anything. So we employ virtual memory semantics to `virtualise` the view of RAM to each process instead of allowing direct access.
  - Processes have their own view of RAM and VRAM. For a 64-bit address space, they should see RAM addresses from 0xHALF - 0x0 as "their memory". The other stuff above are not accessible and are reserved for the privileged processes. This means processes should not allocate too much memory and that they have only 256GB of memory to work with. A process that would need more should use the `Neutron Workstation` edition that restricts the kernel's virtual address space to a minimum and allows maximal use of virtual memory space for processes.
- **CPU Instructions Control**: we dont want processes being able to execute any instruction they want, esp the ones that can cause changes to other hardware like the disk, GPIO and peripherals.
  - CPU Rings: 0-3 in increasing order of privilege (RISCV). Ring 3 is 'Machine Mode' which allows all instructions to be executed. 1-2 are lower modes, 2 is 'Supervisor Mode'. Supervisor mode seems to also allow all instructions, though in a more controlled manner setup by the bios firmware. Ring 0 is the least privileged, allowing instructions like `scause` to be uncallable, if called, an interrupt should be generated with the `UNPERMITTABLE IN RING 0` error code. Kernel should then inform the process of such and maybe choose to terminate it immediately or warn it/tally once and let it resume. Then terminate if 3 warnings are generated
  - MMIO: instead of having CPU instructions, we instead setup MMIO within the hardware itself and initialise it within the BIOS firmware. As the kernel boots up, it should then setup virtual memory and control access to these MMIO via services/RPC/IPC that causes the kernel to spawn a new thread to handle the request within the kernel context (may still to have switch a hardware thread if all of them are being used)
- **GPU Control**: like the CPU, the GPU is an entire processor with its own ISA and possible RAM.
  - Like the CPU, has its own privilege vs user instruction set. In software, devs normally write GPU code with shaders and compile them with a properitary compiler. This prevents a lot of stupidity from occuring. For potential malicious activity, the GPU can also employ virtual memory and user vs privilege mode MMIO accesses via kernel code. They will be accessible via drivers in userspace, and if using Rust or a high level programming language, as a userspace library
  - Note: processes have every right to use as much resources as they want. Its the kernel's job to restrict and prioritise certain processes and see which ones are hogging the bandwidth. If the system logger detects a process has been using a lot of resources for a while (e.g. 30min), it could do certain things like slow it down, deprioritise its priority index, etc
- **Firmware Control**: to prevent firmware from executing potentially stupid or malicious code, the kernel can isolate devices in their own containers and allow them to be interfaced through driver code.
  - The driver code enforces correct back and forths communication between device firmware and system software.
  - Normally, drivers are memory mapped and placed in a restricted address space which processes can call but not edit. If they try to edit the address space where the drivers live, an `UNPERMITTED PAGE ACCESS` should be generated. This applies to all MMIO services setup/granted by the kernel
