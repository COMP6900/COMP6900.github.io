---
layout: default
title: Lecture 3
parent: Lectures
---

## Lec 3

Collectivism is like a sugar pill for fools. It makes you feel like you're going to be okay in the instant, and as if you are part of something big. But in reality you are simply overlooking your own weaknesses and relying on numbers to win rather than enlightened action.

## Hardware Design

### PCB Design

Use something like KiCad. You can start off with a schematic to place down all the small components in place, positioned relative to each other. Then you can go into PCB mode and drag and drop the components onto a board in a similar position. But in a more calculated form for easier production, physical stability, heat generation, etc. Although a lot of the heat generation can also be done in lower levels component-pcb wise or higher within the case design.

- Idk how much it would take to design an entire motherboard or SoC but Im guessing a lot of it is done for economics sake. They are optimising for a range of factors like size, heat, cost vs performance.
- I optimise for size, heat and performance only in the smallest package available. I like to think from negatives. If we go too far then we can scale back.
- If we think about it the otherway around, if we can still keep going then increase, I feel like its not as good since we can fail at an increased stage, but we cant if we are scaling back instead.

### FPGA Design

It is quite hard to make something high speed and something that 'just works' out of the box. A lot of the IDEs seem quite messy though so it might not be too bad as long as you arent thinking in 2000s terms and in modern rust-20s terms.

- It is great having an FPGA to test out a hardware design without actually having to make a PCB for it, manufacture it and put it in place. Just use the FPGA chip itself to simulate an uploaded CPU and connect it to RAM and IO.
- Spectro FPGA! Just came to my mind. Based on TinyFPGA and Xilinx's normal sized FPGAs we can simulate a small 4-8 core RISC CPU. Though GPUs will take up more space, as well as on chip RAM.

### Scala-Chisel3

Chisel is a mid-high level hardware design library for scala. We can write a specification for a hardware device and simulate it directly and test it with scala's great tools. The clean syntax makes it smooth and cool to use.

- We can also compile from chisel to C++ and do further testing on a lower level. Even better we can simulate it quite fast (although software simulation) with verilator that converts the C++ to a verilog.

## RAM & Memory Management

We use page tables and stuff to manage memory. It is a given thing for kernels.

In RAM, there are a bunch of things we want to keep track of:

- Page tables
- Any global descriptors, e.g. GDT, IDT, etc.
- Memory-mapped objects like VDSOs
  - can be used for frequently used syscalls
  - can be used for virtual drivers (virtually addressed driver functions mapped to userspace virtual addresses). Or even used in kernel
  - when used in kernel, can be used for MMIO. Kernel mode only though and should be in kernel-only frames/virtual addresses. Just map something like a framebuffer to memory and manipulate the bits directly to manipulate the framebuffer. Or for something like audio, manipulate the bits to tell the sound device to output certain sound waves (digital signal)

In Video RAM, we could have the same thing. Although memory-mapped objects dont make as much sense and can be mostly ignored.

### Page Tables

The core structure that translates between a process' virtual address to an actual physical address. Then can dereference e.g. a 32-bit or 64-bit value at that address to load into a register. Or vice versa, store a 64-bit value from a register to a virtual-physical address.

### PTE

A page table entry is the core feature of a PT. It contains:

```rust
@FFI(C)
object PageTableEntry {
  @range(4096)
  frame_number: u64,
  // if page is in RAM
  present: bool,
  // RD_ONLY, RD_WRITE, etc.
  protection: u4,
  // was it modified in any way since creation/swap?
  // basically modified and has not been saved to storage (disk) yet
  // if we e.g. want to write a block of memory to swap space, we have to check if it is dirty, and if so, write it
  // if not, then we can just discard the page as the swap space contains the up to date version
  dirty: bool,
  // allow page to be cached in TLB. If cachable, the cache may not have the latest data?
  // more like if a page is really important you might not want to risk caching it
  cache_enable: bool,
  // was it used in a recent CPU cycle? May be useful for things like LRU heuristics and time measuring of what pages keep being used
  referenced: bool,
}
```

more on [dirty bit](https://en.wikipedia.org/wiki/Dirty_bit)

### MMIO

If we use an MMIO scheme, we would prob have to use kernel drivers. If we wanted to use user-manipulatable drivers, we can try something like virtual-user drivers through SR-IOV

### Video MMIO

We would prob use a framebuffer mapped to RAM or VRAM. The hardware controller can then listen on those addresses for any changes, and update the hardware

- for frambuffers, we could have N framebuffers for N supported output displays. Usually at least 1 output display is available and can be enabled by quering ACPI
- some applications may have higher level abstractions and theirr own pseudo framebuffers like KDE. We would want to reserve the main framebuffer for rendering the entire desktop and auxiliary pseudo buffers in their own window processes to manage any extra information like dual-framebuffers
- it can be quite slow to 'copy' a whole bunch of memory like an entire framebuffer byte by byte to another address block. There are ways to speed it up like 'moving' the memory view somehow. But since you have to manipulate the framebuffer's physical address values themselves, it may not be possible
- so instead we prob just link the GPU to the output itself. The GPU can decide when to output based on its currently running shader code or a signal from the CPU that tells it to output once its done

### Audio MMIO

Stuff:

- [MMIO 1](http://midi.teragonaudio.com/tech/mmio.htm)
- [MMIO 2](http://www.edm2.com/0403/mmio.html)
- [Difference between DMA and MMIO](https://www.quora.com/What-is-the-difference-between-direct-memory-access-and-memory-mapped-I-O-systems)

### SR-IOV

Could we perhaps place drivers or driver interfaces in userspace for direct use, rather than doing syscalls and stuff indirectly to e.g. read/write from disk or to get the mouse position at the next tick.

- Some like mouse position can be abstracted pretty well by GLAD or something similar so its not a big deal. But having fine grained control over the drivers could also be useful if a program wants to

## RISC-V SV48

SV48 is the virtual memory extension for riscv64.

- 2^36 page table entries when using 4K pages. We calculate this from 2^48/2^12 = 2^36. This is around 69bn maximum entries in the entire table

![](/assets/img/riscv/RV64-RV48-extension-virtual-memory.png)

- Note that PPN[3] is actually 17bits instead of 9. This allows the physical address space to be 2^52 -> 2^4 times larger than the virtual address space

```rust
Fields = {
  // reserved for supervisor code (usually extra kernel bookkeeping)
  RSW: u2,
  dirty: bool,
  accessed: bool,
  global: bool,
  // if this page is a userspace page, otherwise reserved for kernel
  user: bool,
  // permissions for this page, for userspace (kernel doesnt care)
  // is this page executable? Does it contain code (32bits each) that we should execute
  executable: bool,
  writable: bool,
  readable: bool,
  // not scrapped/invalidated and
  // matches the page on disk, if page to disk is enabled
  valid: bool,
}
```

### Why 48-bits?

Because that is all thats needed, 256TB is a lot of space for the entire OS, apps, etc.

- We usually also only have 16-32GB of RAM on physical machines. Having way more virtual memory than physical memory available doesnt make too much sense either
- If we wanted to implement 64-bit addresses, we just the rest of the 16bits instead of setting them all the the same bit as bit 47. Then we would have to implement CPU microarches to actually support it, i.e. TLB and CSRs now read 64-bits instead of up to 48 for virtual and 52 for physical. We simply translate a 48-bit address to a 52-bit address (backwards comaptible)

# Filesystems and Disks

[What are filesystems?](https://en.wikipedia.org/wiki/File_system)

For certain filesystems like ext4, we can have a lot of external fragmentation since we allocate new files randomly and store pointers to them. For NTFS, its more compact at the beginning though requires frequent defraging for internal fragmentation.

- NTFS is relatively simple but works. So as ext4, which tries to improve on ext3.
- Other filesystems like btrfs and zfs are more enlightened in their features and usages. I'll talk a lot about these as well as the basics like FAT (File allocation table).

## Solid State Drives

They use LBA (logical block addressing). A driver should be able to see how the ssd works and abstract away the logic to allow read() and write() by the kernel interface.

They are NAND Flash so are quite efficient and fast. Unlike a mechanical storage unit like a hard drive or floppy disk or an optical one like a CD (compact disk).

They usually have a small microcontroller and firmware embedded into the device itself, or perhaps the interface like NVMe or SATA. This allows relatively more complex calls to be decoded into simpler calls to address LBA's and stream data back into memory, i.e. via DMA.

### Controller

Within the SSD is a controller chip that directs requests (at each clock cycle) to actual operations on the flash chips that store the information.

![](/assets/img/SSD-controller-large.png)

- a SATA-3 SSD 2.5in form factor. Image taken from [Storage Review](https://www.storagereview.com/wp-content/uploads/2010/04/SSD-controller-large.png)

![](/assets/img/nvme-ssd-controller_2.webp)

![](/assets/img/nvme-ssd-controller_1.png)

- Controller architectures for NVMe SSDs. Images taken from WD and EKWB

SLC = Single Level Cell\
MLC = Multi Level Cell\
TLC = Thhree Level Cell\
QLC = Quad Level Cell

Each increasing level allows lower heat generation, higher efficiency and speed. But higher cost due to expensive technological manufacturing processes.

Notice the 'SLC Cache' in the WD SSD. This is a static NAND flash which is used for frequently looked up LBAs. Since fast SRAM is quite expensive, it would be a problem to make it too fast. But if we have e.g. 30GB of SLC cache we can store a lot of data for much faster lookups, e.g. 2s of cache lookup vs 50s of normal lookup on the same instruction.

- The SLC cache can take a write request and write it to the cache instead in a very fast amount of time. Then during downtime, the controller can tell the cache to start writing to the memory. Now if we're doing this right when we want to do something else with the SSD, that can be a bit of a problem but at least we can usually use the downtime for something good like this
- The cache itself is SLC, but the rest of the storage is TLC. This allows the cache to be not too expensive as if we were using CPU-type SRAM.

### Low Level Firmware

The SSD controller is still a microcontroller and hence can be programmed, usually in some kind of properitary code.

[Great link](https://www.extremetech.com/extreme/210492-extremetech-explains-how-do-ssds-work)

![](/assets/img/ssd_firmware.png)

- The firmware contains components to manage the entire SSD. Stuff like garbage collection (relocating existing data to new locations, erasing prexisting data), flash translation layer management (for mapping LBAs to flash memory banks), metadata manager (for the entire drive e.g. UEFI GPT, partition metadata is managed by the fs). The NVMe manager usually handles the interface routing and communication requests to and from the SSD.
- Note that NAND flash drives need to first erase the memory banks, then write a new set of stuff (e.g. a file) to them. To do this they can move a block of memory to somewhere else. Then utilise GC to invalidate/clear the preexisting data.
- The backend NAND cells may be managed by a 'microcoded' channel controller that directly manages each bank of memory and provides itself some level of abstraction with channels to handle more bits at once. The firmware decodes instructions down to microcode level for the channel controller to make the final scans or writes. A scan of a block would be returned to the firmware, which may then choose to cache or stream back directly through PCIe. A write could be done like RAM where we have rows and cols and write bytes at a time to each [row, col] address. These writes could be structured as streams of pairs of `[row, col]: byte`.

![](/assets/img/SSD-Controller-Elements.png)

- An II SSD could be implemented as a mini environment. It may have its own embedded processor in 16/32 bits with its own small ISA like RISC-V. Would be great to couple it to the CPU arch for the CPU to more easily manipulate it, unless using MMIO which is recommended.
- Additional settings embedded on-chip in the EEPROM. Can configure the SSD through the Chip Config interface for speed control, LEDs, debug IO, clock control.
- Almost like a mini motherboard sometimes. Has a bus that can directly access main memory addresses. And the rest is pretty properitary and choice'd in terms of implementation. GPIO as well if you want to program the SSD to do specific things.

### General Flash NAND

SSDs, USB-3 Flash Drives, SD Cards, EMMC are all types of NAND Flash, but with differing implementations, quality and therefore performance. They are usually all quite reliable, and should have ECC stuff on chip.

![](/assets/img/flash_nand_diagram.jpg)

## FAT

[Wiki for FAT](https://en.wikipedia.org/wiki/File_Allocation_Table)

### Basics

- use a linked list to store files
- have a single header at the top of the partition that lays out the pointers to each inode start

## Btrfs

The B-Tree filesystem was made by a whole bunch of dudes, namely the people over at linux & linux partners, facebook, intel.

### Features

- A lotta storage -> 16 Exibytes altogether and 2^64 max number of files indexable (if each file = 1B each)
- Snapshots of the entire partition like git, and the ability to revert to earlier snapshots
- Subvolumes -> allow extra namespacing for virtual filesystems. Snapshots are basically subvolumes, and with CoW, is quite quick to make and takes little space
- Also good features like online/cloud transactions, RAID0, swap files/partitions support, mostly lossless compression per file/volume
- Problematic: 255 char pathname limit (due to byte storage)

### How does it work?

[BTRFS Internals](https://dxuuu.xyz/btrfs-internals-3.html)

[BTRFS in Linux](https://btrfs.wiki.kernel.org/index.php/Btrfs_design)

## Neutron File System

A modified version of btrfs that takes out the unimportant stuff.

- supported by the NFSDriver in Arcboot, will search for and prioritise NFS partitions to see whether a neutron kernel image exists in their `/boot` directories
- if so, will list those first

Things BTRFS doesnt need => extended safety protocols

NFS makes things easier to manage. Creates and deallocates files and dirs quickly.

- a userspace abstraction "Ember" builds upon it by exposing a semantic key-value store of files. Useful for most generic files a user deals with
- for the most part, NFS is used for system files, app files, executables, links, etc. But they are generally a thing for devs and the OS rather than the user
