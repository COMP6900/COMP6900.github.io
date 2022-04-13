---
layout: default
title: Microkernels
nav_order: 2
parent: Dumb Stuff
grand_parent: Kernel

---

Although not as bad as monolithic kernels, we still have some really weird ideas of how to make something good

## Place more things in userspace = win?

What? Really? Yes place drivers in userspace, great idea. While we're at it why dont we just have a single address space for everything.
No need for virtual memory = faster right? Well I guess no need for a TLB, which isnt terrible. So then you wont have to store extra page table/paging bookkeeping stuff and instead have some 0-999 pid permissions for each frame. Processes wont have a coherent view of memory like in vm, but rather be told what memory blocks and stack they have. Yes.

## Still kind of a good idea

The concept of having a minimal kernel itself is a good idea. But you shouldnt be putting things in userspace imo. Have a "monolithic" kernel like macos but simplify and debloat it as much as possible and rely more on kernel modules like FreeBSD.
