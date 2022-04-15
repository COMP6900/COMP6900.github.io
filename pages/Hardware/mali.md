---
layout: math
title: Mali GPU
parent: Hardware
---

## Mali T880

Only supports vulkan 1.0.

### AMBA 4

A specification for more stuff like ACE, AXI Coherency Extensions

## AXI

### ACE

AXI = Advanced Extensible Interface

### Write Signals

xVALID handshake is put into BVALID write response channel.

xREADY handshake signal is put into BREADY write response channel.

User defined data is put into BUSER write response channel.

Write response status of the burst is put intpo BRESP channel.

Write response ID is put into BID channel.

### Handshake Mechanism


