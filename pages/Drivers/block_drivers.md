---
layout: default
title: Block Drivers
parent: Drivers
---

## What are Block Drivers?

Usually disks. We transfer data to and from in 'blocks' rather than sequences. For sequential (char) transfer, we would have stuff like keyboard, mice, and other volatile devices usually without state.

- Char driver performance/speed isnt really too important as they arent as signifcant in overall system performance. Latency is a bigger issue. But for block drivers, performance is critical. We dont want to have to wait along time for a file to open or for a file to save.
- Ramdisks are of interest as we can make a lot of abstractions within RAM and use those structures for interfacing with disk.

## Registering a Block Driver

We register a Bdriver like Char driver. Esp in linux, where we use `register_blkdev(u32 id, char* name)`.

- To unregister, we use `unregister_blkdev` in the same way

## Operations

Like a char device, block devices have the base operations `open, release, ioctl`. And use a `block_device_operations` struct. Extra methods include:

`media_changed(gd: Gendisk) -> Int` - check whether user has changed the media in the drive. Returns positive value if changed.

`revalidate_disk(gd: Gendisk) -> Int` - called automatically when media is changed. Maybe a disk was unplugged or a new disk was plugged in.

`let owner = *Module` - pointer to the module that owns this structure.

## Gendisk

Linux's representation of a single disk device. Basically 'generic disk'.

```
Gendisk = {
    major: u32,
    first_minor: u32,
    minors: u32,
    disk_name: [char; 32],
    block_dev_operations: *FileOps,
    request_queue: *Queue,
    flags: u32,
    // usually set with self.set_capacity(u64)
    capacity: u64,
    // growable data on demand for a block drive implementation to use for their own internal constructs
    private_data: Box<Data>
}

fn alloc_disk(minors: u32) -> &Gendisk;
fn del_gendisk(gd: &Gendisk);
```

- `Gendisk` uses `ARC` to count its refeerences. We use `get_disk, put_disk` to read and increment/decrement the ref count
- should only call `add_disk` to add a new disk plugged in on the system, when the disk driver has completely initialised

## Driving an SSD


