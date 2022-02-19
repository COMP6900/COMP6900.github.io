---
layout: default
title: Asst1 - 22S1
parent: Assignments
---
## Assignment 1: CPU, Bootloaders and Memory <strong>[Draft]</strong>
Starts Jan. 25th.

| Feature | To be Assessed | % |
| --- | ----------- | --- |
| *RISCV64GC ISA* | 4 different types of instruction encoding. Code each type in rust. | 30% |
| *UEFI Bootloader* | Code in C. No C++, Rust or anything else. Needs to interface with QEMU `virt` and detect all bootable EFI partitions in a `-disk` drive specified. | 40% |
| *Paging* | Code in C only. No C++, Rust, etc. Needs to interface with CSR VM register (CSR2) equivalent in riscv. Pages are 4K each and constitute a 48bit virtual memory/52-bit physical memory scheme. A specified linked list allocator in `autotest` must be able to use your paging scheme. | 30% |

## Tools
You will need:
- Test: `autotest` -> `cargo install comp6900-autotest`
- Run: `qemu`
- Build: `rustup`, `gcc-x86_64`, `gcc-riscv64`
- Platform Tools: git for windows or bash on any unix like OS

I will post a script that will detect your OS and download and install the required dependencies. Note you will need a bash shell to run it.

```bash
#!/bin/bash
#[draft]

if [ -os Windows ] then
    # RUST
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    source <RUST>
    rustup component add rust-src
    rustup target add riscv64gc-unknown-none-elf

    # MSYS64
    # install msys64
    wget <msys64> ~/Downloads
    exec ~/Downloads/<msys64>
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

if [ -os Unix ] then
    <same as windows except no msys64 but brew or download with apt/pacman/fedora/etc>
fi
```

## Assessment
As the dictator of the course, I will be marking. So you better not send something with less than 1000 lines of code.
