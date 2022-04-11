---
layout: default
title: Parallelism
parent: Kernel
---

## Parallelism

So we have symmetric multiprocessing and ideas of implicit and explicit parallelism.

- Also very long instruction word. This allows an instruction to contain multiple subinstructions

## Superscalar

Basically means we duplicate all the components on the CPU. So two ALUs, two FPUs, etc. per core. Hence this allows us to send 2 instructions down a CPU core's pipeline instead of only one, and have basically 2 execution units per core.

- We fetch 2 instructions in a single clock cycle
- We also need 2 fetch and decode units. And after execution we have 2 write back units. But the writeback unit interfaces with the same core-level memory controller for L1 I/D cache. The rest is up to the cluster design, e.g. route the L1 cache directly to L2
- NOTE: in a write through cache scheme, we would write back to RAM as well as the local core caches. This is a simpler design although means there may be more traffic on thee CPU - RAM bus

A standard scalar CPU takes 2x less silicon to make, compared to a 2-way superscalar CPU.

## VLIW

Instead of a superscalar CPU with 2 of everything, we make each instruction longer. Instead of a single 32-bit instruction, we can have a 128-bit instruction that encodes up to 2 integer64 instructions and 2 float32 instructions simulataneously. Then have the fetch/decode and dispatch figure out how to execute and predict branches, etc.

- Compilers can compile source code into VLIW supported arches. Given that the microarch has correctly implemented VLIW supported, we dont need to hand optimise or code VLIW per se. But we can do certain things like code in a certain way to ensure most of our logic is 'close' together without too many branches, function calls (stack overhead), mallocing (should be all done at once). To reduce some of these overheads we can also inline as much as possible given the ICache size, have smaller loops, have similar matching code executing together, i.e. spatial locality. It may be somewhat harder to maintain temporal locality if we are offering an application with a lot of randomness. But most code usually follows the same branches, by pareto's principle
- Much simpler to implement than a superscalar arch. Though maybe not optimise by hand or even with software tools. It is quite hard to make full or good use of VLIW
- Some research on variable length VLIW. But rn a simple scalar CPU isnt a terrible idea esp if your software code quality is good. I could then play around with the hardware-compiler choices to see the differences in performance, latency, cost through implementing superscalar vs VLIW
