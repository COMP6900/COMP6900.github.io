---
layout: default
title: USB
parent: Drivers
grand_parent: Kernel
---

## Overview

USB Devices can be plugged in and out of USB ports. These ports should be found the acpi or device tree on firmware boot.

- the thing with USB is you have a host controller that manages all the connections. This is to allow easy plug and play, and is why it is used for many things
- the controller asks each connected device if they want to send any data, so it works preemptively by polling every now and then

### USB Subsystem

A USB system is arranged in a hierarchical structure:

- ground
- power
- 2x signal wires

The thing with the USB protocol is that it only defines the communication channel between device and host. It does not define any meaning or structure to the data that goes back and forth.

- so basically we can just pass a `*void` data block by streaming it on demand. And end the data block with some terminal char

### Standards

USB defines a set of standards for each type of device. If each device follows that standard, then we dont need any extra drivers for it except USB itself.
