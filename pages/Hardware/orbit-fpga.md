---
title: Orbit FPGA
parent: Hardware
layout: default
---

## FPGA options

A good one Ive seen is [this](https://au.mouser.com/ProductDetail/Intel-Altera/HLDC-DDR4-4GB-A?qs=GedFDFLaBXF8IbKp2YF4lg%3D%3D)

- 4GB DDR4 RAM
- Prob enough LBAs to simulate a CPU. Not an entire modern SoC though

### Simulating an SoC

Will prob need a few FPGA chips or boards.

- Prob tons of ways of doing it too
- Main thing is latency and speed of IO from one chip or board to another
- Also topology of connections

## Simulating Spectre SoC

Ok. So we can use multiple FPGA chips by xilinx in parallel. We will have to use our own software to interface with it. E.g. JTAG.

1. Either multiple FPGA chips on a custom PCB
2. Or a buyable FPGA board with 4-8GB RAM and 10000-100000s of LBAs

## Orbit Design

Orbit implements the spectre style of SoC port comms. It has a lot of GPIO which can be used for power in, UART, SPI, I2C, etc.

It also has 4-8 Xilinx FPGA chips connected in a spectre soc like topology.

- The MPU system can be >4 chips. The general and compute subsystems are onnected by high speed buses of at least 128bits parallel
- The RTU system can also be >4 chips. Like the compute subsystem, we may need to split up several 'thread blocks' or 'clusters' into each chip. And connect each cluster with high speed buses

In theory, we should have everything grouped together as close as possible. If I can find a chip with very high LBA and speed then maybe I'll use that.
