---
layout: default
title: Memory
has_children: true
---
## Memory = RAM

When I talk about "memory", 99% of the time, I mean Random Access Memory. So basically the ~8-16GB memory accessible to the CPU. We always assume 64bit OSes running on 64bit INT instructions and FLOAT32 instructions. QUAD and DOUBLE are supported on riscvgc, but we dont use it for now.

When I say VRAM, I mean Video RAM. When I say "unified ram/memory", I mean RAM shared by both CPU and GPU. When I say "cache", I mean CPU caches L1-3 99% of the time. TLB is cache specfically for virtual memory. ICache = Instruction cache. DCache = data cache. When I talk about ICache, 99% that means L1 instruction cache in a "cpu core". A cpu core in riscv is usually single threaded, i.e. no "hyperthreading" as RISC doesnt take advantage of it as well as CISC.
