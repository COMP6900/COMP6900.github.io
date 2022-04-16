---
layout: default
title: ARM
has_children: true
parent: Hardware
---

## Features

### AMBA 4

A protocol defines how 'functional' blocks communicate with each other. It defines the entire on chip interconnect and management of functional blocks like the cores ad DMA.

- AMBA usually specifies a coherent or noncoherent bus for groups of blocks
- the main bus is prob a coherent interconnect

## AXI

An IP like protocol that defines the interface of IP blocks. Does not define the interconnect itself.

- optimises for high bandwidth between manager and subordinate, similar to PCI
- basically an AMBA sub protocol for block interfaces

![](/assets/img/arm/AXI-master-subordinate.svg)

### ACE

AXI = Advanced Extensible Interface

### Write Signals

xVALID handshake is put into BVALID write response channel.

xREADY handshake signal is put into BREADY write response channel.

User defined data is put into BUSER write response channel.

Write response status of the burst is put intpo BRESP channel.

Write response ID is put into BID channel.

### Handshake Mechanism
