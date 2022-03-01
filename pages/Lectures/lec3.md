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
