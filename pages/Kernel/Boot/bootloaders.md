---
layout: default
title: Bootloaders
parent: Boot
grand_parent: Kernel
---

## Overview

So we want a relatively low level abstraction of the BIOS functions like ACPI for managing devices and disks. We want to be able to allow the user to search for compatible and "good" kernel images.

- stuff like multiboot compatible kernels

## Multiboot

It is possible to have multiple OSes on the same system, e.g. installed in different partitions across several disks.
Usually, its prob better to just use a type-1 hypervisor and run OS images in higher level containers. That way we can simplify the bootloader-kernel header layer.

### Installing and using a bootloader

With a traditional Von Neumann-OS system, we always start with the BIOS firmware that either executes as its separate controller or as a complete elf-like program.

Next the BIOS should look for a bootloader partition. Usually theres only one, or perhaps two if we have are dual booting and stuff.

The bootloader partition on UEFI should be an elf image that can be loaded into RAM and executed. Usually it will have pointers to other LBAs that store the rest of the bootloader image.

The bootloader takes control. A bootloader must be of the same arch (e.g. elf64/riscv) as the system. Usually a BIOS of a particular device will have its own MMIO mappings. On older devices, you may have to just use IO ports and CPU instructions.

To be able to do more useful things with the hardware components like disk, mouse, display, etc. We should have a higher level abstraction of the devices. So usually the BIOS will implement ACPI (v1 or v2) and expose those tables, registers and stuff at 0x0 or some standard agreed upon RAM/physical address.

### UEFI Bootloader

[Here](https://wiki.osdev.org/UEFI_App_Bare_Bones) is a bootloader that uses the UEFI interface.

### After Bootloader

The bootloader should have setup paging structures, e.g. 4KiB. And the associated bookkeeping. For RISCV, it should also implement SBI and expose the functions at their standard virtual addresses for the kernel code (supervisor mode) to call on the fly.

- so SBI functions are the second form of mmio functions. First being BIOS MMIO exposed in the ACPI tables
- SBI tables may also exist or something
