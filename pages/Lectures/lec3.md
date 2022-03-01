---
layout: default
title: Lecture 3
parent: Lectures
---

## Lec 3

Collectivism is like a sugar pill for fools. It makes you feel like you're going to be okay in the instant, and as if you are part of something big. But in reality you are simply overlooking your own weaknesses and relying on numbers to win rather than enlightened action.

# Hardware Design

## PCB Design

Use something like KiCad. You can start off with a schematic to place down all the small components in place, positioned relative to each other. Then you can go into PCB mode and drag and drop the components onto a board in a similar position. But in a more calculated form for easier production, physical stability, heat generation, etc. Although a lot of the heat generation can also be done in lower levels component-pcb wise or higher within the case design.

- Idk how much it would take to design an entire motherboard or SoC but Im guessing a lot of it is done for economics sake. They are optimising for a range of factors like size, heat, cost vs performance.
- I optimise for size, heat and performance only in the smallest package available. I like to think from negatives. If we go too far then we can scale back.
- If we think about it the otherway around, if we can still keep going then increase, I feel like its not as good since we can fail at an increased stage, but we cant if we are scaling back instead.

## FPGA Design

It is quite hard to make something high speed and something that 'just works' out of the box. A lot of the IDEs seem quite messy though so it might not be too bad as long as you arent thinking in 2000s terms and in modern rust-20s terms.

- It is great having an FPGA to test out a hardware design without actually having to make a PCB for it, manufacture it and put it in place. Just use the FPGA chip itself to simulate an uploaded CPU and connect it to RAM and IO.
- Spectro FPGA! Just came to my mind. Based on TinyFPGA and Xilinx's normal sized FPGAs we can simulate a small 4-8 core RISC CPU. Though GPUs will take up more space, as well as on chip RAM.

## Scala-Chisel3

Chisel is a mid-high level hardware design library for scala. We can write a specification for a hardware device and simulate it directly and test it with scala's great tools. The clean syntax makes it smooth and cool to use.

- We can also compile from chisel to C++ and do further testing on a lower level. Even better we can simulate it quite fast (although software simulation) with verilator that converts the C++ to a verilog.

# RAM & Memory Management

We use page tables and stuff to manage memory. It is a given thing for kernels.

In RAM, there are a bunch of things we want to keep track of:

- Page tables
- Any global descriptors, e.g. GDT, IDT, etc.
- Memory-mapped objects like VDSOs
  - can be used for frequently used syscalls
  - can be used for virtual drivers (virtually addressed driver functions mapped to userspace virtual addresses). Or even used in kernel
  - when used in kernel, can be used for MMIO. Kernel mode only though and should be in kernel-only frames/virtual addresses. Just map something like a framebuffer to memory and manipulate the bits directly to manipulate the framebuffer. Or for something like audio, manipulate the bits to tell the sound device to output certain sound waves (digital signal)

In Video RAM, we could have the same thing. Although memory-mapped objects dont make as much sense and can be mostly ignored.

## MMIO

If we use an MMIO scheme, we would prob have to use kernel drivers. If we wanted to use user-manipulatable drivers, we can try something like virtual-user drivers through SR-IOV

### Video MMIO

We would prob use a framebuffer mapped to RAM or VRAM. The hardware controller can then listen on those addresses for any changes, and update the hardware

- for frambuffers, we could have N framebuffers for N supported output displays. Usually at least 1 output display is available and can be enabled by quering ACPI
- some applications may have higher level abstractions and theirr own pseudo framebuffers like KDE. We would want to reserve the main framebuffer for rendering the entire desktop and auxiliary pseudo buffers in their own window processes to manage any extra information like dual-framebuffers
- it can be quite slow to 'copy' a whole bunch of memory like an entire framebuffer byte by byte to another address block. There are ways to speed it up like 'moving' the memory view somehow. But since you have to manipulate the framebuffer's physical address values themselves, it may not be possible
- so instead we prob just link the GPU to the output itself. The GPU can decide when to output based on its currently running shader code or a signal from the CPU that tells it to output once its done

### Audio MMIO

Stuff:

- <http://midi.teragonaudio.com/tech/mmio.htm>
- <http://www.edm2.com/0403/mmio.html>
- <https://www.quora.com/What-is-the-difference-between-direct-memory-access-and-memory-mapped-I-O-systems>

### SR-IOV

Could we perhaps place drivers or driver interfaces in userspace for direct use, rather than doing syscalls and stuff indirectly to e.g. read/write from disk or to get the mouse position at the next tick.

- Some like mouse position can be abstracted pretty well by GLAD or something similar so its not a big deal. But having fine grained control over the drivers could also be useful if a program wants to
