---
layout: default
title: Graphics Driver
parent: Drivers
---

Graphics drivers are prob the most complex stuff in coding existence. AMDGPU has like 12m lines of code itself. So yea.

## What is in a graphics driver?

A bunch of stuff. You'll also want a vulkan -> gpu binary compiler. Usually the driver is bound to the kernel interface and OS features. Windows and Nvidia for example, stuff like optimus, ray tracing uses DX12.

So you'll want to be able to manipulate the GPU to do mostly high level stuff from drivers. But there is just so much functionality I guess. Most of the code th runs on the GPU is simply a shader binary or a CUDA/OPENCL binary that uses the GPU ISA compiler. You'll need to tell the GPU to actually start executing and read from VRAM. Or DMA to RAM.

## Bootloader driver code

Prob dont need much or any at all. Can just use the CPU to manipulate a memory mapped framebuffer. And link that framebuffer directly to VGA out/HDMI/DP. So no need to mess with the GPU at all. Just detect that there is a GPU and create some informative wrappers around it for the kernel to load appropriate drivers for them. And use shader code if a supported gpu exists on the system and is usable.

## Kernel driver code

The kernel would usually have access to a higher level abstraction over the devices. E.g. a device tree for RISC or acpi device tables.

It seems like a good idea to implement some level of abstraction in the bootloader. Some higher level view of the GPU MMIO in terms a low level drawing API that wraps common ops like how gl libraries do stuff:

```
expose __common_gl_shader_function
expose __common_gl_verify_function
expose __common_gl_do_function
```

Then the kernel drivers themselves simply implement stuff that allocates GPU memory (VRAM or unified), submit a command queue, and other close-to-hardware stuff.

## Userspace driver code

A lot of the logic goes in the userspace code since we dont need to bloat the kernel with such complexity. With libc we can also make our lives easier.

The idea is to be able to implement any given API, vulkan, opengl, metal, etc. In a userspace-agnostic way so to be portable.

Here we have ways to compile shader code and DMA it to the GPU. We also wrap around the driver functions like `read, write` with code that translates from VkImage, etc.
