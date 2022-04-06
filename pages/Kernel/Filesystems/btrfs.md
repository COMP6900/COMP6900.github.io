---
layout: default
title: Btrfs
parent: Filesystems
grand_parent: Kernel
---

## Overview

Btrfs or "btree fs" was released in 2013 and developed by many major companies like FB, Intel, Linux Foundation, Red Hat, Oracle.

The main idea is to use a CoW filesystem so that you dont actually mutate your existing blocks. Rather you remake the entire btree of an inode so that you get better safety with concurrent access and builtin snapshots of your entire partition (inode/block tables).

- another thing is that btrfs uses pooling as a way to manage resources
- it also defines checksums for data integrity

### Pros over EXT4

- newer design ideas, including modern tech and standards
- better scalability
- better reliability (checksums instead of just journalling)
- easier to manage (idk about this actually)

## Theory

### B Trees

B Trees are self balancing trees that try to make the height of leaf nodes the same. This allows a tree to be as round as possible for THETA(log(n)) reads, writes, etc.

Now, B+ Trees are a 'better' version of a normal b tree where each node only has keys and not `key:val` pairs. It also links leaf nodes. They are often used for on-disk DBs since they have very high fanout/balancing so finding a block requires fewer IO ops. But they are not great at CoW, which is not great if we want easy, strong concurrency management.

So what if we modify the standard BTree to have ensure that leaf nodes are not linked together and give a refcount to each node. But store each refcount in a free map. The free map would be created on the fly. The B Tree itself would also have less rigorous balancing algorithms. In all, this allows CoW to be implemented effectively as snapshots. Each snapshot would be created on each modification and be stored in a certain data structure.

### Proposal

So we want a snapshot capable fs that would use the above data structure exclusively for metadata and file data (blocks). And also recursively, to track space allocation of the trees themselves.

Mason: we would be able to funnel all traversal and modifications through a single code path. So CoW, checksumming, mirroring only need to be implemented once to benefit the entire filesystem/substructures.

## Design

Btrfs is structured as a few layers of the data structure. That is layers of B Trees ontop of each other all using the same implementation.

- each tree stores "generic item" nodes. These items are sorted by their keys, which are 136 bits in length
- the most significant 64 bits is the Unique `Object` ID. The next 8 bits is its `type`, and can be used for tree lookups / item filters
- an `Object` can have multiple items of multiple types
- the last 64 bits are specific to the type. Items for the same object end up adjacent to each other in a tree. Items themselves are also grouped by their type

1. objects can put items of the same type in a certain order if we choose a certain key

An interior node is just a list of key-pointer pairs.

- each pointer is the logical block number of a child node
- leaf nodes contain item keys packed into the front of the node. Item data is packed at the end of the node. The data grows up towards the key. If more keys are needed/more key data, it will also grow towards the data
- so a leaf node is full when the keys/data cant grow anymore, and some method needs to be used to solve it/insert node/rebalance

### Filesystem Tree

Within each dir, dir entries appear as 'directory items'. The least sig bits of their keys are a hash of their file name.
Their data is a location key. So it is the key of the inode it points to. (An Inode Item)

So Directory Items can act as an index for a path-to-inode lookup. They are not used for iteration since their hashes are random.

- but this means we would need many more disk seeks between non adj files. Other fs like ext4 actually have TEA-hashed filename keys and are used for iteration. They are quite fast in that regard
- we thus make each dir entry have a `directory index` item. Its key is set to a per directory counter that increments with each new dir entry
- iterating over these should return entries that are in order like on disk, though maybe not 100% of the time

### Hard links

Files with hard links in multiple dirs have multiple reference items. One for each parent dir.

- but files with multiple hard links in the same dir simply packed all the links' filenames into the same reference item. This was a problem and set a hard limit of the amount of hard links possible
- so we fixed it by using `spillover extended ref items` to hold hard link filenames that are too long
