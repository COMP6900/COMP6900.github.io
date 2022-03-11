---
layout: default
title: Lecture 4
parent: Lectures
---

## FAT (File Allocation Table)

When you format a partition with FAT, you would first add an entry to the GPT or MBR. Then for those contiguous sectors or LBAs that you want to partition, you place the filesystem's metadata into the first LBA used.

### Nodes

Most filesystems usually have a metadata 'header' table that stores an index to each file. Usually needs only 1 LBA, though more can be used.

### The "Table"

The table refers to a list of entries in the first LBA (not an actual table).

Entry 0-1 contains the volume info for the partition/filesystem. Entry 2-..n refers to a 'cluster'.

- Note a FAT32 fs supports up to a 2TB partition. But each file can only be up to 4GB each
- to increase the size, you could use a utility to split up a larger file to smaller 4GB files

The table can just be thought of as a 'table of contents' for the disk. (Not the 'filesystem'). Each cluster is either part of a single file, reserved for the OS (e.g. swap space or boot/recovery), free for use. Sometimes a cluster may also be unavailable if it is part of a bad sector.

- when a file points to an entry (cluster), then it is in a linked list structure. If we follow the linked list, we can go through each cluster of the file. All done in the table itself
- each cluster of a file may not be right next to each other on the disk itself, but the following scheme still works. To 'open' a file we go through the linked list and return the clusters in order. To read, similar idea but we just go through it until we get to the offset. For writes though, we may need to reallocate based on how much space the new write/file takes
- write reallocation is probably handled quite well by the disk driver. Like git, you would have to make additions and removals and if there are quite large changes, then maybe you just use another function that completely deallocs the prev one and allocs a new cluster list

Each file's metadata is 32B and stores:

```
filename -> 0-10
attributes/permissions -> 11
reserved for NT -> 12
creation_time_second -> 13
creation_timestamp -> 14-15
creation_date -> 16-17
last_accessed
first_cluster_number_high16 -> 20-21
last_modified_time
last_modified_date
first_cluster_number_low16 -> 26-27
file_size -> 28-31 //in bytes
```

- there is also a 'long' version but dont worry about it. Basically if the filename exceeds 11B, we can add another header right afterwards that stores the full filename
- the metadata is stored with the file's physical data, usually right before its contents. Files are contiguous so if you know the start and the size, you know exactly where the file is on disk

The above is actually an entry for the 'directory table'. The directory table is another header structure (tree like) stored along with the FAT.

## Secondary GPT

Note a disk formatted with GPT will usually have a secondary GPT that mirrors the primary GPT, but stored in the higher end LBAs.

- useful for recovery. Should not be touched by software at all

## Ext4

The 'extension 4' filesystem is a partition scheme used widely on linux systems. Mainly for root and general storage partitions/filesystems.

- the 4th extension to the original linux filesystem scheme. Ext 1,2,3 are all extensions as well

## Filesystem Layers

Here is an overview:

![](/assets/img/FS_Layers.gif)

- got this somewhere on google, I think it was from a company or university, thanks to the guy who made it

# QEMU

QEMU is an emulator. It emulates a specific ISA through 'dynamic binary translation'.

- basically interpreted instructions, e.g. riscv64gc -> x86_64 step by step
- latency can be quite high if interpreting step by step. So if possible, translate the entire .img into one that can be run on the host ISA in a containerised environment

TODO

- [Best documentation](https://wiki.qemu.org/Documentation)
- [OSDev](https://wiki.osdev.org/QEMU)

## QEMU BIOS

We can write a bios for qemu.
[TODO READ](https://www.qemu.org/docs/master/specs/acpi_hw_reduced_hotplug.html)

- hotplug on the bios = device can be added or removed without shutting down the system. So we can "hot plug" a usb during runtime

## QEMU Devices

[TODO READ](https://qemu.readthedocs.io/en/latest/system/device-emulation.html)

VIRTIO -> virtualised IO

QObject -> QEMU Object Model. Allows us to register a type that we create. Like we can create a custom device that we want to emulate with the components like cpu, maybe gpu?

### Disks

Its a great idea to create a qemu disk as a virtual disk for your image or app. Just do:

```
qemu-img create myimage.img mysize
```

which creates a file of size `mysize`. Then just specify that when you want to launch an OS/vm with that disk `--disk=myimage.img`.


## QEMU Usermode Emulation

Kind of like Wine. We have a containerised environment for translating syscalls across platforms. Given the same architecture and usually ELF64 img format.

- creates a host process container. Then runs that stuff in there with a syscall translation layer. Im guessing
- supports linux and bsd systems. For bsd, freebsd, openbsd, etc. and possibly macos. So basically popular modern UNIX systems

To run an img compiled for another platform, just do the usual `qemu /path/to/img`. Replace `qemu` with `qemu_x86-64` or `qemu-i386` if your on x86. And etc. for arm and risc.

- if you try to run an img compiled for another arch. It prob wont run since the opcodes are different, or the header says an unsupported platform

