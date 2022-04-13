---
layout: default
title: Asst2 - 22S1
parent: Assignments
---

## Assignment 2: Drivers, PCI, MMIO, Graphics & Networking **[Draft]**

How does a graphics driver work you ask? That is such a good question. Even better, how does a wifi driver work? These devices usually connect over PCI and with ACPI and MMIO in mind.

We use DMA and MMIO for most low level control over the hardware. Usually a GPU connected via PCI exposes a set of registers that can be controlled. And its video memory (usually 8GB+) can be mapped to the virtual RAM address ranges through DMA.

So if we copy a block of memory from RAM -> GPU it will be quite easy.
