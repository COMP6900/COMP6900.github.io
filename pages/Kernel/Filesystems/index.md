---
layout: default
title: Filesystems
has_children: true
parent: Kernel
---

## Overview

Filesystems are a software level abstraction to more effectively manipulate the lower level byte-addressing on disk.

## fsck

File system consistency check. It is a great tool for looking for any corrupted checksums, dangling blocks and such. Prob not for defraging, which would require something like gparted.

[Got info from here](https://www.cs.unc.edu/~porter/courses/cse506/s16/slides/ext4.pdf)

- can be quite slow since we are going through the entire filesystem twice and one by one

### How it works

1. walks through the directory tree. Each reachable node is marked as `allocated`
2. for each inode, check ref count. Ensure all blocks that are referenced by the inode is marked as allocated
3. double check all allocated blocks and inodes are reachable

## lsblk

Great tool for viewing block storage devices. Not any device, e.g. char and network devices,

## gparted

Apparently the most versatile tool on unix. Resize, create, delete, clone, defrag partitions. Most partitions supported. Also frontends available in gnome.

## fdisk

Useful for MBR partitions and only for disks <2TB. For GPT, use gdisk.

## gdisk

Great!

## parted

Also great. Esp gparted with the GUI.
