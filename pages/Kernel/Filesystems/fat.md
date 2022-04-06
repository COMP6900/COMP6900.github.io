---
layout: default
title: FAT Filesystem
parent: Filesystems
grand_parent: Kernel
---

## FAT

FAT or File Allocation Table FS was originally built for 8-bit computing in 1977.

### Layout

1. Boot Sector
2. Info Sector
3. **Optional** Reserved Sectors
4. File Allocation Table 0
5. **Optional** File Allocation Table 1..N
6. Root directory (size = N*32/1000B/Sector)
7. Data region (size = N clusters * sectors/cluster)
