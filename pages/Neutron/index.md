---
layout: default
title: Neutron
has_children: true
---
Something that isnt as dumb. Well tries to fix the things that are dumb from first ideas

## Virtual Memory Layout

Based on: <https://www.kernel.org/doc/html/latest/riscv/vm-layout.html>. <br/>
On a 64-bit system with 48-bit virtual memory, we have:

| Space | Range |
| --- | ----------- |
| Kernel img (2GB)| `0xffffffff ffffffff - 0xffff8000 00000000` |
| Kernel modules (2GB) | `0xffffffff 7fffffff - 0xffffffff 00000000` |
| Kernel serviced mmio (224GB) | `0xfffffffe ffffffff - ffffffc6 fee00000` |
| Kernel services (16,000GB) | `0xffffffbf ffffffff - 0x40 00000000` |
| Userspace (256GB) | `0x3f ffffffff - 0x0` |

### Kernel Img

Coontains the key kernel code for setting up filesystems, drivers, process management, exposing MMIO for key services and syscalls.

### Kernel Modules

Extra kernel modules that can be linked and loaded in the bootloader stage. I actually dont have much to say about it rn.

### Kernel MMIO

Key space where driver and low level service ABI can be called by higher level service code running in Kernel Service space or Userspace.

### Kernel Services

Stuff like initd, devicemanagerd, moused, cpud, networkd, ipcd, filesystemd etc.

### Userspace

Your apps and very high level services that run in usermode. HTTP servers and stuff will user kernel networkd sockets. Apps like umbral word will use filesystemd through mmio ipc or syscalls.
