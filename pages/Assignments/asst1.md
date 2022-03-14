---
layout: default
title: Asst1 - 22S1
parent: Assignments
---
## Assignment 1: CPU, Bootloaders and Memory <strong>[v1]</strong>

Starts Feb. 25th. Due in 2 metric weeks.

| Feature | To be Assessed | % |
| --- | ----------- | --- |
| *RISCV64GC ISA* | 4 different types of instruction encoding. Code each type in rust. | 30% |
| *UEFI Bootloader* | Code in C. No C++, Rust or anything else. Needs to interface with QEMU `virt` and detect all bootable EFI partitions in a `-disk` drive specified. Needs to be able to boot Neutron. | 40% |
| *Paging* | Code in C only. No C++, Rust, etc. Needs to interface with CSR VM register (CSR2) equivalent in riscv. Pages are 4K each and constitute a 48bit virtual memory/52-bit physical memory scheme. A specified linked list allocator in `autotest` must be able to use your paging scheme. | 30% |

## Tools

You will need:

- Test: `autotest` -> `cargo install autotest-6900`
- Run: `qemu`
- Build: `rustup`, `gcc-x86_64`, `gcc-riscv64`
- Platform Tools: git for windows or bash on any unix like OS

I will post a script that will detect your OS and download and install the required dependencies. Note you will need a bash shell to run it.

```bash
#!/bin/bash

if [ -os Windows ] then
    # RUST
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    source <RUST>
    rustup component add rust-src
    rustup target add riscv64gc-unknown-none-elf

    # MSYS64
    # install msys64
    MSYS2_LINK="https://repo.msys2.org/distrib/x86_64/"
    wget $MSYS2_LINK ~/Downloads
    exec ~/Downloads/$MSYS2_LINK
    # wait for it to complete install
    # start it, should be added to path
    source ~/.bashrc
    exec msys64 << pacman -Syu && expect "Y\n"
    exec msys64 << pacman -S mingw && expect "Y\n"

    # QEMU
    winget install --id=SoftwareFreedomConservancy.QEMU  -e

    # GCC-RISCV
    wget <gcc-riscv64> ~/Downloads
    mkdir -p ~/tools/riscv
    mv <gcc-riscv64> ~/tools/riscv
    echo "export PATH=$PATH:~/tools/riscv/<gcc-riscv64>/bin" >> ~/.bashrc

    echo "DONE!"
fi

if [ -os Ubuntu ] then
    sudo apt install rustup riscv64-unknown-elf-bintuils riscv64-unknown-elf-gcc qemu
fi

if [ -os Mac ] then
    brew install rustup riscv64-unknown-elf-bintuils riscv64-unknown-elf-gcc qemu
fi
```

## Assessment

As the dictator of the course, I will be deciding the final marks. The `autotest` script should decide 99.99% of the marks.

### Criteria

| Feature | Does it pass the autotests? |
| --- | --- |
| *RISCV64GC ISA* | Yes = 100%, No = 0% |
| *UEFI Bootloader* | Yes = 100%, No = 0% |
| *Paging* | Yes = 100%, No = 0% |

## Sample Solution

Here is my sample in rei:

```
# In riscv64gc.rei

/*
 r format
*/

// 64IMA CSR FENCE

// F32,64

// C -> compress instructions to 16bit
// when imm is small
// one of reg is zero or usual
// src = dest reg
// 8 most popular reg x1-x8
// NOTE: must align on 16bit boundary in the .code section, so:
//  [16bit-instruction | 16bit-padding | 32-bit instruction ]
// should also be loaded into the RAM .text section at 16bits too

/*
 i format
*/

/*
 s format
*/

/*
 sb format
*/

/*
 u format
*/

/*
 uj format
*/

# In paging.rei

// no commas in rei, only for for loops and decorators
object Page {
    address_start: u48
    size_bytes: u32
}

const n_page_entries = 1024

class PageTable {
    constructor() {}

    // note, labels by themselves are statements
    // this by itself can be a substatement of let, const, etc
    entries: [PageEntry; n_page_entries]

    @deduce
    fn add_new_page(type: PageType) {
        // get the next page from the process' page table
        let last_page = entries.type(type).last()
        lew new_page = Page {address_start: last_page.next()}

        entries.push(new_page)
    }
}

```
