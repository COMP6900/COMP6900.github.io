---
layout: default
title: SSD Filesystems
parent: Filesystems
grand_parent: Kernel
---

## SSDs vs HDDs

Hard disk drives and SSDs are similar in that they both prefer sequential access. HDDs even more so since they are mechanical with magnetic strips.

HDDs are separated into sectors. Each sector is a sector of the disk circle. It usually coveres 10 tracks. The head can only scan a single track at a time. It can move onto another track. It can also read/write to other disk circles stacked vertically at the same time, but I think its a bit hard to drive it like that.

- the earliest hdds used horizontal recording. So the charges were either left/right and the head has to scan each bit for either a left or right charge. We used a 'ring' head element to read/write horizontally
- the left/right charge were bad because temperature changes can affect the charges, flipping them. This affected hdds with sector sizes smaller than a certain size
- so instead we use monopole head. This allows to record stuff in perpendicular chages. We also need an extra zero charge layer underneathe as insulation. However we still have increased senstivity to read errors and external magnetic fields. So more accurate read/write arms needed to be developed as well

A SSD works similary to RAM where there are rows and cols of bytes. We specify a grid reference `[row, col]` to get that byte stored at that location. Kind of like a memory address.

- SSDs have no mechanical parts. We simply use capacitors for a single bit. >50% charge is high/1 and <50% is low. In reality its prob more like 40-60 so we can employ ecc or something, and see that the ssd has worn out
- so there is minimal latency since the electrons move almost near light speed
- there is 3D nand to further improve the size/efficiency. And lower heat generation and the distance to get and store the data
- however, cost is still a problem

Normally, an SSD has a microcontroller that 'sees' the storage in terms of pages (4KiB). OSes nowadays sees disk chunks as LBAs or 'sectors' from the old HDD days.

An LBA is like a sector. It replaced the outdated cylinder head sector scheme and was used in HDDs too.

### LBA

[Great read](https://www.elinfor.com/knowledge/overview-of-ssd-structure-and-basic-working-principle2-p-11204)

- An LBA is generally 512B. The OS should access disks in 4KiB (drivers that read and write 4KiB at a single time). On a PCIe 4.0 x4 we have a speed of 7.9GB/s. We transfer back/forth 4 bits at a time in parallel, at a very high rate. So to read a 4K block/page, we have to mak e arequest to read 4096/4 = 1024 cycles from ssd -> ram
- the driver should be able to convert that 4KiB block in LBA space to physical NAND host page space
- the microcontroller fetches from the physical NAND page by looking up the Map Table

At the highest level of the microcontroller we see a map table. With `n-2` entires. Each entry points

A host page is 4KiB. There are 64 million host pages in a 256GB SSD.
So the map table needs to have 64 million entries. Each with size 4B to store the physical address of the host page. So we need a 256MB map table.

- For SSDs, we can read the map table into main RAM for faster access. This can happen in the kernel boot stage or the bootloader stage. Usually 256MB isnt too bad for 16GB. When you have multiple disks you can use GBs of RAM though, but its prob worth

### Microcontroller Lookup

So the OS wants to access a file on ext4. It looks up the path and the inode associated with it. By requesting the partition on the SSD, the superblock and inode table at their usual offsets given the amount of block groups.

So the driver queues up the requests for those 4KiB pages representing the 4KiB blocks on the filesystem. It prob has associated RAM locations to tell the SSD where to write it to before it asks for the each page, e.g. some 64bit physical RAM address on each page read request.

When the SSD finishes each request (DMA), it sends an interrupt and goes to the next one. The driver keeps track of the queue and sends a software interrupt (trap) when the entire queue is complete. This will then send a signal or wake the userspace code that prob paused on IO read.

Now the app has access to the memory mapped file, in the heap or process container memory.
