---
layout: default
title: ACPI Device Tables
parent: Drivers
---

We can implement ACPI into our UEFI BIOS so the bootloader can get a view of the preliminarily connected devices during boot and set up data structures for them with SBI. Then new devices that are connected after boot will also use SBI to register

## ACPI

ACPI was meant to be just a way to set power states and sleep states for a device. You tell the device to go into S0 or P0 for example and its firmware handles it.

- but it can also be used for device discovery. Usually the BIOS firmware implements and ACPI view
- [good resource](https://www.usenix.org/legacy/publications/library/proceedings/usenix02/tech/freenix/full_papers/watanabe/watanabe_html/node4.html)

So we have tables that the BIOS sets up, a small acpi subsystem in the BIOS, and registers that the devices themselves implement to the ACPI spec.

### Tables

Unlike the device tree, we use a bunch of tables. Each entry points to a device. USB and PCI are easily enumerated and should work easily. But other port standards, prob not. HDMI, USB/2, etc. may need either more config or a device tree.

### RSD PTR



## Device Discovery

The BIOS interrogates all the devices that are connected to it during boot. Note, can be and should be done on the fly, not always though by specific OEMs.

- if the device is responsive and can be interacted with, BIOS will ask/determine how much memory space they need for all their ops
- then it sets up base address registers (BAR) for each device. Each device may have up to 6 32-bit BARs (PCIe 3.1 spec). Or better, just 3 64-bit BARs
- the address decoder then translates loads and stores to those addresses to actual data reads/writes to those device's and functionality
- [more on mmio setup](https://superuser.com/questions/595672/how-is-memory-mapped-to-certain-hardware-how-is-mmio-accomplished-exactly)

## PCI Configuration Space

PCI devices must have a certain set of registers called the 'config space' if we want to set them up with MMIO

![](/assets/img/pcie/Pci-config-space.svg)

### Device ID 16-bit Register

The DID register associates the device a unique ID

### Vendor ID 16-bit Register

The VID register + DID allows a completely unique fingerprint of the device

### Bar 32-bit

A 32-bit BAR:

```
@length = 1
enum RegionType: Memory = 0, IO = 1

BAR_32 = {
    region_type: RegionType
}

BAR_32_Memory {
    region_type = Memory,
    // should be 2 on 64-bit platforms
    locatable: u2,
    prefetchable: bool,
    // should be 16B aligned, e.g. 0x10 minimum, multiple of
    base_addr: u27
}

BAR_32_IO {
    region_type = IO,
    // should be 4B aligned, e.g. 0x04 minimum, multiple of
    base_addr: u31
}
```
