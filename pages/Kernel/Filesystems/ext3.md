---
layout: default
title: EXT3
parent: Filesystems
grand_parent: Kernel
---

## Additions

Basically:

1. journalling
2. optional HFS dir indexing (not file blocks)
3. online support

While being mostly backwards compatible with ext2, and uses the same set of tools like `e2fsprogs` and `fsck`.

## Directory Indexing

Turn on the `dir_index` flag to use it on linux.

So we have HTree structures for each directory. A directory inode has an inode array 

## Extent Tree

[Btrfs Trees Overview](https://btrfs.wiki.kernel.org/index.php/Trees)
