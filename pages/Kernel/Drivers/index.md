---
layout: default
title: Drivers
has_children: true
parent: Kernel
---

## Driver Ops

Drivers must use physical addrs to copy the block to and from process memory and disk. You pass a virtual addr to the kernel with `read()`. This gets translated to via the process' TLB/PT to some PAddr in userspace. But in kspace, read() actually translates the vaddr as you go into the kernel. So the driver routines actually work properly without TLB (set paging to false) while reading and writing. This seems pretty unsafe because it is, so the kernel has to ensure the process isnt trying to read too much or write too much / at the wrong place.

A terminal program catches keyboard and mouse inputs. KB inputs are redirected to tty0, which is then directed to the output framebuffer of the `kterm` program. The `kterm` program converts that char into a vector/bitmap form and uses the graphics driver or cpu to render the next image, then overwrite the framebuffer location with the new memory vals. This should happen pretty quickly. IDK if there are ways to conserve energy or speed up. Such as maybe keeping the prev texture and simply adding the new char on. And just changing that specific mem addr on the framebuffer mmio.

## TO READ

[ARM64 ACPI Tables](https://www.kernel.org/doc/html/latest/arm64/acpi_object_usage.html)

- they have to implement DSDT, FADT, GTDT, MADT, MCFG, RSDP, SPCR, XSDT. And a few extra recommended stuff. The others are not as useful. Same for riscv
- for x86, a different story. Maybe more, maybe less. I think maybe less cause has more abstractions esp windows and more complex BIOSes

[QEMU Boot Steps](https://www.qemu.org/2020/07/03/anatomy-of-a-boot/)

[Device tree overview](https://elinux.org/Device_Tree_What_It_Is)

[Driver dev -> ACPI and MMIO](https://forum.osdev.org/viewtopic.php?f=1&t=32210)

## What is a driver?

Great question. A driver is software that drives hardware. A driver for a device can be implemented as a bunch of files that take specific OS service calls and translates them to low level MMIO device calls.

### Example: `open()`

So if you wanted to `open()` a file, the OS needs to translate that to a device call on a disk and a partition, via a driver for that disk. The driver would prob intercept the `open()` call in its call queue. Then it would see that `open` was requested on a file belonging to disk A, partition P. Then it will look up a table of disks numbers - memory addresses in memory. It will then make the right MMIO calls to BIOS. To access an SSD on a RISCV BIOS, you would set the appropriate registers and call INT 13Hx42H to read from a disk > 8GB in capacity.

Then the driver will wait on the firmware to satisfy the request, stream it into a driver kernel buffer in RAM. When satisfied, a hardware interrupt is generated. The OS service recognises the interrupt is for them. Since the driver sends a signal `REQ_COMPLETE(p_id, *data)` which the service process intercepts in the IPC bus. Kernel can set permission flags to ensure the `*data` in the frames can only be accessed by the process with `p_id`.

The service process sees that the request has been complete. The signal should basically tell it to `RESUME` on that request thread and send the data to the callee process, e.g. file browser or word processor. If everything goes well, the app should then be able to see that the data it wanted is placed in its userspace data buffer within `HEAP` or `OPEN_FILES`. Usually it will also have a `RESUME` call that tells it to start e.g., rendering the contents of the directory or file after `await let f = open(file_path)`.

## `close()`

Same idea as open(). Also read(), write() later.

## `fork()`

Quite important for a POSIX compliant OS.
