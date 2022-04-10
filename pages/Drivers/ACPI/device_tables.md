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

### RSD Pointer

The RSDP is a pointer that is always stored in the same location in RAM (uefi standard).

It references the RSD Table (Root Data Structure) somewhere else in memory.

## ARM: Required Tables

DSDT and
