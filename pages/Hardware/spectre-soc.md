---
title: Spectre SoC
parent: Hardware
layout: default
---

## Specs

### MPU Complex

General Compute:

- 1x 'High Core'
- 3-7x 'Standard Cores'
- 4-8x 'Efficiency Cores'

CUDA Compute:

- 3000-4000x 'Streaming FP32 Cores'
- 700-1000x 'Special Function Cores'
- 700-1000x 'Tensor FP16 Cores'

### RTU Complex

- 3000-4000x 'RT FP12 Cores'

### Unified Memory

- 8-16GB LPDDR5X

## Ports

### Power

1. 3.3V Input for standard clocks
2. 5V Input for boost clocks and overclock

Only one of the above can be used at a time. If both ports have positive max voltages, SoC will lock.

### PCIe

- 1x PCIe 4.0 x4

### Video

- 1x HDMI 2.1
- 1x eDP

### USB

- 2x USB 4.0

### SODIMM Expansion

- 2x SODIMM Slots, each 16GB max

Very high speed link to the unified memory port. For non-phone/tablet devices only.

### GPIO

- 10x GPIO pins
  - 2x UART receive ports
  - 2x UART transmit ports
  - 6x General pins
  - 1x SPI recieve
  - 1x SPI transmit
  - 1x I2C recieve
  - 1x I2C transmit

Useful for all kinds of things like networking and etc. Mainly used to hook onto extra functionality not already supported by the SoC.

### Media

- 1x Stereo Out
- 1x Mic In

## Notes

<i>CUDA Compute subsystem is useful for all kinds of things. Highly parallelisable algorithms, deferred rendering and post processing of frames from RTU, ML model training and querying and General 2D/3D Rendering.</i>

<i>I2C is optimised for IC-IC comms (serial). SPI is optimised for communicating with peripherals (serial). UART is good for transforming parallel to serial data for IC-IC comms as well.</i>
