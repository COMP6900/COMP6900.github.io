---
layout: default
title: Video Memory
parent: Memory
---
## Basically RAM but for GPU
There is also virtual memory for different processes, and thus page tables and stuff. Like CPU programs, GPU programs (shaders) can be streamed from disk to VRAM via DMA at a certain offset/virtual memory.

Like the CPU, GPU memory hierarchy is also L1 Cache -> L2 -> L3 -> VRAM -> Disk. Can think of GPU as a CPU that is optimised for processing data rather than branches/if statements.

## Memory Layout
Like RAM, it is a good idea to specify a virtual memory layout for each process using VRAM.
The kernel also uses the top half of RAM to make it simpler for processes. The DE will usually use Userspace VRAM. Kernel VRAM is good for simpler things like startup and metadata.

| Space | Range |
| --- | ----------- |
| Kernel (2GB)| `0xffffffff ffffffff - 0xffff8000 00000000` |
| Userspace (Whole Shebang) | `0xffff8000 00000000 - 0x0` |

