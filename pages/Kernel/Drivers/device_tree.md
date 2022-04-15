---
layout: default
title: Device Tree
parent: Drivers
grand_parent: Kernel
---

## Overview

The device tree interface is a legacy thing. It is hierarchical.

- it is implemented by the BIOS, who goes through each port and scans the stuff
- either in the source format (dts/dtsi) or a 'flattened' format (fdt)

[Good Reference](https://elinux.org/Device_Tree_Reference)

## What information is contained?

- Device characteristics, e.g. how much RAM, clock speed range
- Connections between devices. Basically a list of buses
- Device configuration information. Maybe any saved states or loaded states
