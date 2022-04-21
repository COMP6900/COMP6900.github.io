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

### Nodes

Each node in a btrfs btree starts with a header. It contains the node's level. So Level > 0 is an internal node. Each internal node stores pointers to child nodes (address start offsets) elsewhere on disk.

Header of the node also contains the number of items the node contains. For an internal node, that is the number of KeyPtrs to child nodes. For leaf nodes, we instead have BtrfsItems. This is basically where to actually find the payload in the leaf node.

BtrfsItems can represent files or file blocks prob.

```rust
// a data item that belongs to a node
object BtrfsItem {
    key: BtrfsKey
    // offset of the payload on disk
    offset: u32
    // size of the payload on disk
    size: u32
}
```

Nodes can have basically infinite number of children. But usually we dont want that. We want fast log(n) search, addition, deletion operations. So we use BTrees to auto balance the entire thing if a node gets too big (too many children).

The superblock has a single root pointer to the root BtrfsNode. The core root node has a BtrfsHeader like any other node. It also has BtrfsItems and BtrfsRootItems.

- note the root pointer is the logical address of the root tree node. We dont have to convert that into an actual physical disk addr since the microcontroller does it for us

The core root node is technically a leaf node. So it has BtrfsItems which contain data items that point to payloads. These payloads are either 'normal' items, FS tree roots or extent tree roots.

- apparently the core root node doesnt actually have BrtfsItem, rather it just has BtrfsRootItems. The first one being the rootfs. The next ones being pointers to extent trees of available space.

### Dirs and Files

The core root node contains a single BtrfsItem which points to the Root Filesystem payload on disk. The rootfs payload `/` is the main stuff that we use for our kernel and prob initramfs. Other filesystems like a FAT32 partition can be mounted separated on `/mnt` as extentfs pointers or something that the kernel/userspace handles itself. RAID and stuff would use chunk and device trees.  The extent trees stores available space that isnt being used for the partition (not entire disk).

BtrfsDirItem represents a single entry in a directory. Not the directory itself. The name of the dir is stored in each DirItem. So we enumerate all the BtrfsDirItem in the filesystem (0-n dirs) up to 2^64.

```rust
object BtrfsDirItem {
    location: BtrfsKey
    trans_id: u64
    data_length: u16
    // technically 2^16. But the max filename len. is actually 2^8
    name_length: u16
    d_type: DType
}

object BtrfsInodeRef {
    // inode number, 2^64 possible files
    index: u64
    name_length: u16
}
```

The BtrfsInodeRef is a helper data structure. It helps link inode numbers to BtrfsDirItems. So basically it links inodes to files (contained within directories).

### Item Type

Each BtrfsItem (not root) can be either:

```rust
enum BtrfsItemType {
    InodeItem,
    InodeRef,
    DirItem,
    ExtentData,
    // lesser
    DirIndex,
    XAttrItem
}
```

A way to walk the tree for the files, is to simply parse all the leaf nodes. So we loop over the leaf nodes and only care about `DirItem`. If we get to one, we fetch that item from disk. And we get the `BtrfsDirItem`.

If the `d_type` of that dir item is `RegularFile` then we know its a file. If it is `0`, that signals corruption and we should prob do something about that item. Maybe go back to a prior snapshot.

```rust
enum DType {
    Unknown = 0
    RegularFile
    Dir
    CharDevice
    BlockDevice
    Fifo
    Socket
    Symlink
    // extended attributes, used on disk and not visible to the user
    // prob good for permissions
    XAttr
}
```

We can then get the name of the file. Note the name is only the name only, not the full path. To get the filepath, we have to find it.

Basically, we get the parent of the DirItem. Apparently the InodeRef contains the index to the parent of that file. So then we go to that index and get that parent's parent, iteratively. Until we get back to the core root node.

### Checksums

Each logical block (payload in the leaf node) contains a checksum field. We can make it a SHA256 checksum of the entire block contents + header and store it into the `checksum` field. Its very quick to verify as we just use the verify alg. But to compute it, takes some time. I think its worth and its pretty strong anyway. Maybe if theres a CPU alg for it that would be great.

- the Btrfs header actually contains a 20 Byte checksum of the node (not the payloads). The header also contains a pointer to the parent, an 8 Byte val of the parent's BtrfsKey value
- since the pointers are sorted by keys, it might be easier to do or something. At least as quick rewrites

### Summary

[Great Resource](https://dxuuu.xyz/btrfs-internals-2.html)

So you have a single superblock at the start. This stores the root filesystem. The superblock is basically an array of (Key, Chunk, Stripe) tuples.

- the superblock is at 0x10000
- there are usually 2 duplicate superblocks for redundancy

Everything is a BTree. All of them are linked together at a single root.

- the core root node contains a reference to every other tree
- BENEFIT: things are very dynamic, unlike having an array of inodes and blocks. We instead make everything a tree

BtrfsKey is the foundational type for all BTrees on disk. A 'node' has one or more data items. And each data item are ordered by their key (ascending).

BtrfsStripe represents a single stripe. One or more may be present depending on the RAID level. If not using RAID, then prob just one Stripe and one disk.

BtrfsChunk is the data stored on disk that describes a 'chunk'. So basically a data item I guess. In the superblock though, it describes the BtrfsStripe.

For RAID configs on multiple devices/partitions, we use both the chunk tree and the device tree.

- chunk tree maps logical offsets to one or more physical offsets on the disk
- device tree does the inverse of that
- BENEFIT: the btrfs fs can grow and shrink without unmounting. Because we can relocate chunks on the fly by remapping the chunk tree

So:

1. superblock -> core root node
2. core root node -> root filesystem (in use) + extent payloads ~4KiB in size each (not in use)
3. root filesystem -> node, node, node...

Doesnt matter what the nodes are. Each leaf node simply points to some used payloads on disk. Those payloads can represent data on any file or (dir?) in any kind of order.

In order to read an entire specific file into RAM, we have to get the inode of the file from its InodeRef and BtrfsDirItem. To do so we have to get the right item types.

Definitely a good idea to map the entire FS into RAM. Esp since it doesnt take that much space.
