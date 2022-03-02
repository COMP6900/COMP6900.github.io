---
layout: default
title: Bootloaders
parent: Boot
has_children: true
---

## Overview

So we want a relatively low level abstraction of the BIOS functions like ACPI for managing devices and disks. We want to be able to allow the user to search for compatible and "good" kernel images.

- stuff like multiboot compatible kernels

## Multiboot

It is possible to have multiple OSes on the same system, e.g. installed in different partitions across several disks.
Usually, its prob better to just use a type-1 hypervisor and run OS images in higher level containers. That way we can simplify the bootloader-kernel header layer.
