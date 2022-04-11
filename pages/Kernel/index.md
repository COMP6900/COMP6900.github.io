---
layout: default
title: Kernel
has_children: true
---

## Kernel Services

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

## Services Control Access to Hardware

The main hardware we want to control access to are the: CPU, RAM, GPU & VRAM, Disk and Specific IO like USB Storage & sensitive pluggables. Other components like Mouse + Keyboard are less at risk and are mostly input devices rather than output.

### Access to Disk and Storage

This is one of the main things. If we have sensitive stuff on an SSD or USB drive, we def do not want anyone or any random code to be able to read from it. We can do things like:

- set a disk or partition to a user. As if they 'own' the partition. If user_2 tries to access a partition owned uniquely by user_1, the kernel should return `-1` or tell the program it cannot access the partition. `user_1` must explicitly give access to `user_2` for that partition. Or the root can do it. In Quanta, there doesnt need to be a distinction between an admin or the root, but for UNIX sake and to protect the avg user from screwing up things randomly its not a bad idea
- wrap functions around the basic MMIO to read/write to disk with driver code. Then only allow the kernel to execute the driver code. Expose access to the drivers through syscalls on files within a partition. Programs dont even need to know the drivers exist, just interact with the VFS built on the partition's FS. In many cases they would be using a high level language like C/Rust with a `std` library that wraps around all this stuff for you

### Access to System Info

We dont necessarily want processes to know everything about the system. It can know some basics but it doesnt need to know exactly how many USB devices are connected, list of users, etc.

- make `get_sys_info() -> SysInfo` a system call requiring the user to be at least an admin. Cannot be used within a `guest` container
- make programs with a low `privilege_level` or `trust_level` be unable to use the syscall for potentially hacktatious reasons. Implement a `trust_level` from 0 - 10, with 10 being the highest and being able to request any syscall. 0 being very untrustworthy and unable to make a syscall request or even run its program in the first place
- all unverified apps/executables can have a `trust_level = 3` or something while verified apps have a trust level of `7-10` depending on their latest update
- allow the user to grant access to system calls to those apps when they try to `syscall`. A GUI popup could be displayed and the user can either click yes or no, or temporarily for this session only

### Access to CPU

In RISC-V, we have 3 modes that automatically do most of the work for us. We prevent usermode apps from executing unwanted or unsafe code by ensuring they arent able to get out of U Mode. And ensuring all restricted things are doable only on request to the kernel through restricted, defined, interrupts.

### Access to GPU

This is can be a bit more easy to do if we have MMIO. We set certain portions of physical RAM address space to map to GPU operations. Like reading a block of memory from/to the GPU, telling it to execute a block of code (shader), telling it to output a frame. We just have to make sure only the kernel has access to those physical frames (i.e. virtual pages that ensure those addresses cannot be accessed by user programs). Then we can wrap around the basic MMIO functions with more complex drivers that implement an API like vulkan.

### Access to Networking Devices

We dont processes to be able to randomly send out stuff with the Wifi or Ethernet, to any random IP with any message it wants. We also dont want it free access to bluetooth or any extra stuff like NFC and location detection.

- Abstract access to the Internet and Networking capabilities via sockets which can be created with syscalls. If a process has permission to use the internet, it can make and close sockets it owns. And freely send out data/requests. If we dont quite trust a process, we can give it a rating `network_trust` in the same manner to prevent them from trying to hog the bandwidth. A general process `priority` will also be able to control how much system resources a process can use, including networking
- Allow a process to manage network state through syscalls, which the user must grant specifically as an `NetworkManagement` permission

### Access to General IO and Peripherals

With general IO, we prob dont want a program to be able to tell devices/controllers connected to them what to do directly. Same with keyboard and mouse peripherals and headphone/mic/speakers.

- For audio we can do something like networking where we open a channel of communication with the mic or speaker. The channel must be opened through a syscall from a trustworthy process. Then it stream data to it through high level functions like `audio_play` which allows a process to copy the data it wants to play to the buffer. Other processes using the same function to play can also copy their data to the buffer. The kernel driver code (on another thread) can then multiplex the incoming inputs either through hardware or a software FFT to combine the signals together and output to the MMIO speaker address
- Same idea for MIC, except we have to copy the data from the kernel buffer to each process listening on the MIC

For mouse and keyboard, we are primarily interested in capturing the inputs. But we dont want any program to be able to do that, e.g. a keylogger.

- The KB + Mouse can be discovered through ACPI and routed into a device table like the rest of the devices. But then the bootloader/kernel drivers can wrap around their MMIO and create listeners. Then the SBI or something can expose these listener interfaces for the Desktop Environment/ Window API to detect mouse movements and keyboard presses and make changes on a queued cycle
- Processes that want to be able to listen on mouse input must be `trusted` and open a communication/listener channel

## Notes

Only the user can change the `trusted` status of an application or executable. An process attempting a `syscall` without required permissions can then tell the Kernel to prompt the user to give the process the required permissions. It is up the user then. If they dont accept, the kernel returns an error and the program should be able to handle it (if they are using high level constructs). On a lower level though, they may crash if they then try to read from a nonexistent virtual address they thought they would have
