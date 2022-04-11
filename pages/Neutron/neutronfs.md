---
layout: default
title: NeFS - Neutron File System
parent: Neutron
---

## Overview

I want it to be similar to apple's fs and btrfs. But minimalised and optimised for neutron syscalls, services and rust/rei apps.

## Design

Sample code for driving an NeFS partition:

```rust
// abstractions
struct NeFSDir; // can be root
struct NeFSFile; // most things are files. A dir is a file with `., ..` and extras

// block objects
struct NeFSSuperBlock;
struct NeFSInodeTable;
struct NeFSInode; // includes b tree of blocks
struct NeFSBlock;
```

### Installer

Neutron comes with NeFS drivers loaded by default. Any new Neutron/Quantii installs should partition a drive with a single root NeFS partition that fills up the disk. The installer can also look at other available drives and format them with extra NeFS partitions.

## Flashing Neutron/Quantii

When we format a drive for Neutron, we should use several partitions. Technically we can partition at any point in the drive but assume we have a clean 200GB PCIe SSD.

We would download the neutron kernel img and the arcboot img (both ELF, arcboot contains logic from linker scripts/asm to start at certain positions/UEFI). We can package all the logic needed to format and make 2/3 partitions on the USB into the ISO. The flashing utility will then copy the files onto the USB.

If compression is turned on, we can instead copy a compressed kernel img onto the USB and just copy the full arcboot files instead. It should be like any other process, since arcboot contains an `.EFI` file in its own filesystem partition, the UEFI BIOS should be able to recognise that partition (from the GPT entries) as bootable.

- NOTE, the BIOS UEFI/GPT should see the disk in terms of LBA. So 512B chunks. Or maybe 4KiB? Anyway its the reason why the OS also sees things in LBA
- A GPT entry that points to the start of a bootable partition simply has the boot flag turned on. And a UUID to identify the type of filesystem. Maybe NeFS wouldnt be supported so we can just store `/boot` on a FAT32 partition

## Neutron Filesystem Hierarchy

Like Linux and Unix derived OSes, NeFS places system files, binaries, static and shared libraries (for dev or runtimes), newly installed apps in their own separate dirs.

Base hierarchy:

```
/
    /desktop
    /documents
    /sys
    /packages
    /dev
    /mount
    /live
```

Explanations of each dir:

```
# quick access stuff and workspaces
/desktop
    /workspace_0
        .workspace_settings
        /widgets
# user stuff (default single user)
/documents
    /downloads
    /images
# system config files, kinda like windows registry and /etc
# combines config for /usr into the system itself, no distinction
# note, no "login manager" per se, just a screen to ensure you are who you say you are
/sys
    /process
    /logs
    /config # basically /etc and windows reg type thing (userspace editor)
# installed packages using Neutron Bundler (NB)
# userland should search this dir for the dock and app drawer apps and executables
# executables are auto namespaced by their package name and can be searched from the searcher like "package_name/<some_exe>"
# .apps are kinda like archives with a single executable and an icon + preferences and settings
/packages
    /package_name
        /some_app.app
        /some_app2.app
        /some_exe.elf
        /lib
    # shared libraries, mainly for executables in packages
    /shared_libraries
# mounted devices and null dev for throwing away stuff
/dev
    null
    mouse_<uuid>
    kb_<uuid>
    usb_drive<uuid>
/mount
    /nvme0p1
    /nvme0p2
    /usb_flash0
# a VFS mainly for storing temporary process data and live logs
/live
```

## Neutron Bundler

The recommended way to install apps. Packages in quantii should be bundled as `.package` archives.

A package can contain zero or more apps. It needs to have at least one 'object' which is either a:

- executable app file `.app` that contains its own thumbnail, config settings, and files that are small
- executable `.executable` files that may not be self contained, requiring extra folders on the fs to run

If developing an app or library, use the language's packaging system instead. E.g. `cargo`, `npm`, `pip`, `bundle`. These should create their own dirs and package things their own way.

- as well as setting any env variables. It is usually not recommended to install executables through a language's package manager
- recommended to only install deps (usually libs) to the project dir
- any extra tools should be installed by `neutron bundler` and be versioned automatically. To use a tool of a specific version, neutron tools should be executed like `<tool> -use <version>`
