---
layout: default
title: Lecture 6
parent: Lectures
---

## Overview

I dont know if Im even following the course structure. But I think I updated it to look more at my amazing code instead of some generic stuff. And you of course will learn how I think.

So today we'll learn more about device drivers.

### Wnere do Drivers live?

Drivers can be written for any part of the boot stage. Usually, the bulk of it lives in userspace, e.g. Mesa. The stuff in the kernel usually deals with allocating memory blocks and submitting commands to a device.

- its good to use the userspace since we have the c library. So we have higher level abstractions to help us do more complex things more easily
- and let the kernel deal with its own RAM management. We are mostly concerned with how data is copied, e.g. double buffering and the mechanisms like DMA

### DMA

Direct Memory Access is a very cool idea. Instead of going through the CPU to copy stuff from RAM to a device's memory and the other way, we can instead do it directly.

- we use a DMA controller on the board
- usually the device's memory is mapped to memory. Not only that we can also map its IO to memory, so stuff like setting a BAR can be done by setting a virtual memory address from an app

Then we can set permissions based on the app. And maybe syscalls to handle some of the stuff instead of allowing apps to directly write to the MMIO. So `ioctl` and `read(driver_fd)` syscalls that are handled by the kernel instead.

## ACPI

APIC is usually at `0xfec00000` on x86. We would use ACPI tables (uefi standard) to find this mmio address.

We use the RSDP to find the RSDT. The RSDT has entries to each other table somewhere in memory, which are usually in a certain range big enough for expansion. Can waste a bit of virtual mem if we make the range too big but its fine on 48/52-bit virtual addressing.

### APIC Table

The APIC tables store info on programmable interrupt controllers on the system.

- intel standard. Replaces PIC and are used in all recent Intel processors
- more sophisticated interrupt redirection and things that werent possible in PIC

It starts with a header.

- string: "APIC"
- length of header: u32
- APIC_BASE MSR: u32
- byte40: flags and legacy 8259 PIC mode
- byte44: packed (c-like) structs containing info about the APICs on the system. IO APIC, local APIC, etc
