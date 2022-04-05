---
layout: default
title: Filesystems
has_children: true
parent: Kernel
---

## ext2

ext2 or "extended 2 fs" is an old fs published in 1993 for Linux.

- designed in a similar way to UFS/BSD fast fs. Ideas include using boot blocks (bootloader only), a superblock (metadata), and 'cylinder groups'
- cylinder groups contain a backup of the superblock. And a header that contains statistics, free nodes, etc. Then we have our inodes and data blocks as the body

![](/assets/img/kernel/Ext2-inode.svg)

### Facts

- max partition size = 32TiB
- max file size = 2TiB
- max number of files (of min size) = 10^18
- max filename length (not including path, which is auto calc'd) = 255B

### Inode

An inode is an 'index node'. We use one to index into a data block or another list of pointers.

- every file or dir is represented by a single inode. A 1-1 relationship

An inode contains a list of 15 pointers.

- pointers 1-12 each point to a data block
- pointer 13 points to a list of 128 pointers. Each of which point to a data block
- pointer 14 is like pointer 13. But each pointer of the list of 128 pointers also points to another list of 128 pointers
- pointer 15 is the same idea, a triply indirect pointer

```rust
@packed
object inode {
    // main header
    mode: Flags[Permissions, InodeMode]
    uid: u16
    size_lower: u32
    last_accessed: u32
    created_timestamp: u32
    last_modified: u32
    // for recovery. usually 0
    deleted_timestamp: u32
    gid: u16
    n_hard_links: u16
    n_disk_sectors: u32
    flags: InodeFlags
    os_reserved_1: u32
    
    // pointers
    direct_pointer_0: u32
    ...
    direct_pointer_11: u32
    indirect_pointer_single: u32
    indirect_pointer_double: u32
    indirect_pointer_triple: u32

    // more metadata
    // mostly used for NFS (new filesystem)
    generation_num: u32
    reserved_1: u32
    reserved_2: u32
    // as an offset of the partition or disk
    block_addr: u32
    os_reserved_2: u32
}

enum InodeMode(u4) {
    FIFO = 0x1000
    CHAR_DEV = 0x2000
    DIR = 0x4000
    BLOCK_DEV = 0x6000
    REGULAR_FILE = 0x8000
    SYM_LINK = 0xA000
    UNIX_SOCKET = 0xC0000
}

// each value can be turned on and off
flag Permissions(u12) {
    OTHER_EXE = 0x1
    ...
    GROUP_EXE = 0x8
    ...
    USER_EXE = 0x40
    ...
    STICKY = 0x200
    GID_SET = 0x400
    UID_SET = 0x800
}

flag InodeFlags(u32) {
    SECURE_DELETE = 0x1
    KEEP_COPY_ON_DELETE = 0x2
    COMPRESSED = 0x4
    SYNC_IMMEDIATELY = 0x8
    IMMUTABLE = 0x10
    APPEND_ONLY = 0x20
    NO_DUMP = 0x40
    NO_UPDATE_LAST_ACCESSED = 0x80
    RESERVED = Bits(8,25)
    HASH_INDEX_DIR = 0x10000
    AFS_DIR = 0x20000
    JOURNAL_ON = 0x40000
}

// used in table of directories
object DirEntry {
    inode: u32
    total_size_entry: u16
    length_of_name_lower8bits: u8,
    type_indicator: TypeIndicator
    name: *char // actually [char; N] would do
}
```

### Superblock

A superblock is the header of the entire fs. It contains how many inodes and blocks there are.

- how many inodes/blocks are in use, how many are free
- other versioning stuff, modification time and mount time

### Symlinks

Basically a file. So an inode.

- technically it can have as many blocks as a file. But all a symlink is, is a string that contains the path of the file/inode it points to
- so we can usually store a symlink's data in the inode header itself. Has to be less than 60 chars and we have to set some flags for it

If the inode/path the symlink is pointing to does not exist, than it is dangling.
Note a symlink cant be incorrect if the path exists, as it does not differentiate between file and dir.

## Driver

1. An ext2 partition starts with a reserved/boot block from 0-1024B.
2. The first block group (block group 0)
3. The superblock lies in 1024-2048B
4. The rest of the block group contains at least 20 blocks and at most 8192 blocks including the superblock. Each the size of 1KiB (can be increased to 2KiB or 4KiB)
5. Block 2 is the BGT (data on number of free blocks, inodes, used dirs, etc.)
6. Block 3 is the block bitmap (each bit denotes whether block `i` is free or not)
7. Block 4 is the inode bitmap (each bit denotes if inode `i` is free or not)
8. Blocks 5-214 contains the entire inode table
9. Blocks 219-8192 contains the actual data blocks. At most 7974 data blocks

So to allocate a new block, e.g. when you want to write a 8000B to a file from 0B, you need 8 blocks. So you find the first 8 blocks available from the block bitmap, set them to 0 to be used, and allocate them to the file's/dir's inode.

- use one of the first 12 pointers, the next one that is free
- if none are free, then go into indirect block
- a file/dir inode is structured in order, to access offset 8000 we need block 7, which should be pointer 7
- when editing (writing), we need a driver that converts the additions or removals to changes to blocks. Either we create more, less, or just modify existing ones
- removing an entire file, removes the blocks first then the file
- removing lines removes the affected data in their blocks. If the file size then shrinks so that we dont need as many blocks to fit the data, we can remove the blocks at the end. Then rewrite the blocks from RAM -> disk as a memory mapped file
- adding lines adds data to their affected blocks. When we run out of space/bytes at an offset, we can use another block
- each block should contain the file's starting and end offset that it contains, so in reality it can only hold 1016B, not 1024B

So at the end of the day, looks like the driver should implement it in a way to reduce internal and external fragmentation.

## Setbacks

- prob not as fast as newer fs like ntfs, hfs+, ext4, zfs and btrfs. Does not use b tree to structure a file/inode's blocks. Just kinda has a table that points to all them, meaning lseek takes O(n) to find the right block
- not as feature rich and contains extra stuff that you dont need in a good OS. Does not have snapshots, journalling (flags in every block or change), larger sizes,
- speaking about inefficiency, to search a dir for a file, you have to search in O(n) front-to-back. So okay for smaller dirs but not as n > 1000 when log2(1000) = 10
