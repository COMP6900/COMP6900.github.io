---
layout: default
title: UEFI
parent: Boot
grand_parent: Kernel

---

## What is UEFI?

Unified Extensible Firmware Interface => Unified version of the original EFI by Intel.

- Basically, an interface between the kernel and platform firmware, like the BIOS ROM.

## GPT

GPT = GUID Partition Table

A GUID is a 'globally unique' identifier. It is a 128-bit label with the format:

```rust
0x aaaa aaaa - bbbb - cccc - dddd - eeee eeee eeee eeee
```

{% highlight javascript %}
UUID = {
    time_low: u32,
    time_mid: u16,
    time_high_and_version: u16,
    clock_seq: u16,
    node: u48
{% endhighlight %}

E.g., a Linux Root partition on x86_64 would have a GUID of `4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709`. This is standard across all linux root partitions.

- So a BIOS or Bootloader would be able to detect this GUID hardcoded into its program, and do useful things like booting a bootable partition.

### GPT Header

When a disk is partitioned with GPT, we install a header in LBA-1.

{% highlight javascript %}
GPTHeader = {
    // for little or big endian
    signature: u64,
    uefi_version: u32,
    header_size: u32,
    // cyclic redundancy check when updated
    crc32: u32,
    reserved: u32,
    // where this header is located, usually 1
    current_lba: u64,
    // lba of a single backup header
    backup_lba: u64,
    // if partitioning this drive, this is the first free block
    first_usable_lba: u64,
    last_usable_lba: u64,
    // maybe little-endian first half, big-endian second half or just random mixture
    disk_guid: u128,
    // usually LBA 2, and ends at 33 at most (or maybe just reserved completely)
    starting_lba_of_array_entries: u64,
    n_partition_entries: u32,
    // usually 128 bits/16B
    size_of_entry: u32,
    crc32_partition_entries: u32,
    // usually 420Bytes or 3360bits (for 512B sectors), or more
    reserved2: >= u3360
}
{% endhighlight %}

So what is in LBA 0 one may wonder? Well it can either not be used or be used for MBR/compatibility. When used as a protective MBR, we have:

### GPT Entry

It is possible to have 2^32 = 4.2bn entries. But idk how much we actually have if 128-bit entry. Then for 31 LBA of 512B each, we have 15,872B of entry space. Hence 15,872/16 = 992 Entries.

{% highlight javascript %}
GPTEntry = {
    // FAT, NTFS, EXT4, etc
    partition_type_guid: u128,
    uuid: u128,
    // where the partition begins
    first_lba: u64,
    // assumes a partition is contiguous across LBAs, otherwise kinda problematic
    last_lba: u64,
    // bit60 = read_only, e.g. boot or recovery partitions
    flags: u64,
    // up to 36 UTF-16 (lower chars)
    partition_name: u576
}
{% endhighlight %}

- Note: UTF-16 is a 16 or 32-bit character encoding. They are usually little endian (depends on the system's endianess). We can encode all 1 million unicode characters in a variable length encoding. Unlike UTF-8 where we can have 1-4 8-bit values for a single char.

For the flags, we can specify a whole bunch of metadata associated with that partition. So:

{% highlight javascript %}
Flags = {
    // does the partition require specialised software/platforms to maintain it? E.g. an OEM partition
    require_platform: bool,
    ignore_this_partition: bool,
    legacy_bootable: bool,
    reserved: u45,
    partition_defined: u16,
    read_only: bool,
    // is this partition a copy of another partition on the same disk
    shadow_copy: bool,
    hidden: bool,
    // tells bootloader/kernel to automount or not
    automount: bool
}
{% endhighlight %}
