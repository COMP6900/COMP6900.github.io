---
layout: default
title: ACPI Device Tables
parent: ACPI
grand_parent: Drivers
has_children: true
---

## Overview

There are like 100 different tables in the ACPI spec. Some of them are arch and kernel specific. Some of them are pretty optional.

But for arm, there are a specific few tables that are absolutely great.

[Best place](https://ethv.net/workshops/osdev/notes/notes-4.html)

- another good place is just the uefi spec pdf
- for acpi only, go to [uefi.org/specs/acpi](https://uefi.org/specs/ACPI/6.4/)

### RSD Pointer

The RSDP is a pointer that is always stored in the same location in RAM (uefi standard).

It references the RSD Table (Root Data Structure) somewhere else in memory.

### HPET

High precision event timer. Basically a 64 bit counter from 0-2^64.

- programmed by MMIO. The main reg is `MAIN_COUNTER_VAL` that counts up and keeps track of the time since 1970
- base address of HPET Table can be found via the RSD Table

## ARM: Required Tables

DSDT and etc.
