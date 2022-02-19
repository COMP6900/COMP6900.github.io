---
layout: default
title: Graphics Driver
---
Graphics drivers are prob the most complex stuff in coding existence. AMDGPU has like 12m lines of code itself. So yea.

## What is in a graphics driver?
A bunch of stuff. You'll also want a vulkan -> gpu binary compiler. Usually the driver is bound to the kernel interface and OS features. Windows and Nvidia for example, stuff like optimus, ray tracing uses DX12.

So you'll want to be able to manipulate the GPU to do mostly high level stuff from drivers. But there is just so much functionality I guess. Most of the code th runs on the GPU is simply a shader binary or a CUDA/OPENCL binary that uses the GPU ISA compiler. You'll need to tell the GPU to actually start executing and read from VRAM. Or DMA to RAM.
