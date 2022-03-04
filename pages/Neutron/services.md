---
layout: default
title: Services
parent: Neutron
---

## Neutron Services

A kernel sits between untrustworthy apps and trustworthy hardware. Not all hardware can be trusted, e.g. peripheral firmware, but that is also software.

- To ensure apps dont do anything stupid or malicious, we need ways to structure and control access to hardware. Bad software cause bad harware execution and possibly corruption of data and the hardware itself. Worse, it could cause a physical hazard, e.g. hospital software
- **Memory Control**: we want to control access to RAM and VRAM. We dont want processes to be able to write over other process's data, or worse, the kernel's data which they will then be able to do anything. So we employ virtual memory semantics to `virtualise` the view of RAM to each process instead of allowing direct access.
  - Processes have their own view of RAM and VRAM. For a 64-bit address space, they should see RAM addresses from 0xHALF - 0x0 as "their memory". The other stuff above are not accessible and are reserved for the privileged processes. This means processes should not allocate too much memory and that they have only 256GB of memory to work with. A process that would need more should use the `Neutron Workstation` edition that restricts the kernel's virtual address space to a minimum and allows maximal use of virtual memory space for processes.
- **CPU Instructions Control**: we dont want processes being able to execute any instruction they want, esp the ones that can cause changes to other hardware like the disk, GPIO and peripherals.
  - CPU Rings: 0-3 in increasing order of privilege (RISCV). Ring 3 is 'Machine Mode' which allows all instructions to be executed. 1-2 are lower modes, 2 is 'Supervisor Mode'. Supervisor mode seems to also allow all instructions, though in a more controlled manner setup by the bios firmware. Ring 0 is the least privileged, allowing instructions like `scause` to be uncallable, if called, an interrupt should be generated with the `UNPERMITTABLE IN RING 0` error code. Kernel should then inform the process of such and maybe choose to terminate it immediately or warn it/tally once and let it resume. Then terminate if 3 warnings are generated
  - MMIO: instead of having CPU instructions, we instead setup MMIO within the hardware itself and initialise it within the BIOS firmware. As the kernel boots up, it should then setup virtual memory and control access to these MMIO via services/RPC/IPC that causes the kernel to spawn a new thread to handle the request within the kernel context (may still to have switch a hardware thread if all of them are being used)
- **GPU Control**: like the CPU, the GPU is an entire processor with its own ISA and possible RAM.
  - Like the CPU, has its own privilege vs user instruction set. In software, devs normally write GPU code with shaders and compile them with a properitary compiler. This prevents a lot of stupidity from occuring. For potential malicious activity, the GPU can also employ virtual memory and user vs privilege mode MMIO accesses via kernel code. They will be accessible via drivers in userspace, and if using Rust or a high level programming language, as a userspace library
  - Note: processes have every right to use as much resources as they want. Its the kernel's job to restrict and prioritise certain processes and see which ones are hogging the bandwidth. If the system logger detects a process has been using a lot of resources for a while (e.g. 30min), it could do certain things like slow it down, deprioritise its priority index, etc
- **Firmware Control**: to prevent firmware from executing potentially stupid or malicious code, the kernel can isolate devices in their own containers and allow them to be interfaced through driver code.
  - The driver code enforces correct back and forths communication between device firmware and system software.
  - Normally, drivers are memory mapped and placed in a restricted address space which processes can call but not edit. If they try to edit the address space where the drivers live, an `UNPERMITTED PAGE ACCESS` should be generated. This applies to all MMIO services setup/granted by the kernel

## Services Control Access to Hardware

The main hardware we want to control access to are the: CPU, RAM, GPU & VRAM, Disk and Specific IO like USB Storage & sensitive pluggables. Other components like Mouse + Keyboard are less at risk and are mostly input devices rather than output.

### Access to Disk and Storage

This is one of the main things. If we have sensitive stuff on an SSD or USB drive, we def do not want anyone or any random code to be able to read from it. We can do things like:

- set a disk or partition to a user. As if they 'own' the partition. If user_2 tries to access a partition owned uniquely by user_1, the kernel should return `-1` or tell the program it cannot access the partition. `user_1` must explicitly give access to `user_2` for that partition. Or the root can do it. In Quanta, there doesnt need to be a distinction between an admin or the root, but for UNIX sake and to protect the avg user from screwing up things randomly its not a bad idea
- wrap functions around the basic MMIO to read/write to disk with driver code. Then only allow the kernel to execute the driver code. Expose access to the drivers through syscalls on files within a partition. Programs dont even need to know the drivers exist, just interact with the VFS built on the partition's FS. In many cases they would be using a high level language like C/Rust with a `std` library that wraps around all this stuff for you

### Access to System Info

We dont necessarily want processes to know everything about the system. It can know some basics but it doesnt need to know exactly how many USB devices are connected, list of users, etc.

- make `get_sys_info() -> SysInfo` a system call requiring the user to be at least an admin. Cannot be used within a `guest` container
- make programs with a low `privilege_level` or `trust_level` be unable to use the syscall for potentially hacktatious reasons. Implement a `trust_level` from 0 - 10, with 10 being the highest and being able to request any syscall. 0 being very untrustworthy and unable to make a syscall request or even run its program in the first place
- all unverified apps/executables can have a `trust_level = 3` or something while verified apps have a trust level of `7-10` depending on their latest update
- allow the user to grant access to system calls to those apps when they try to `syscall`. A GUI popup could be displayed and the user can either click yes or no, or temporarily for this session only

### Access to CPU

In RISC-V, we have 3 modes that automatically do most of the work for us. We prevent usermode apps from executing unwanted or unsafe code by ensuring they arent able to get out of U Mode. And ensuring all restricted things are doable only on request to the kernel through restricted, defined, interrupts.

### Access to GPU

This is can be a bit more easy to do if we have MMIO. We set certain portions of physical RAM address space to map to GPU operations. Like reading a block of memory from/to the GPU, telling it to execute a block of code (shader), telling it to output a frame. We just have to make sure only the kernel has access to those physical frames (i.e. virtual pages that ensure those addresses cannot be accessed by user programs). Then we can wrap around the basic MMIO functions with more complex drivers that implement an API like vulkan.

### Access to Networking Devices

We dont processes to be able to randomly send out stuff with the Wifi or Ethernet, to any random IP with any message it wants. We also dont want it free access to bluetooth or any extra stuff like NFC and location detection.

- Abstract access to the Internet and Networking capabilities via sockets which can be created with syscalls. If a process has permission to use the internet, it can make and close sockets it owns. And freely send out data/requests. If we dont quite trust a process, we can give it a rating `network_trust` in the same manner to prevent them from trying to hog the bandwidth. A general process `priority` will also be able to control how much system resources a process can use, including networking
- Allow a process to manage network state through syscalls, which the user must grant specifically as an `NetworkManagement` permission

### Access to General IO and Peripherals

With general IO, we prob dont want a program to be able to tell devices/controllers connected to them what to do directly. Same with keyboard and mouse peripherals and headphone/mic/speakers.

- For audio we can do something like networking where we open a channel of communication with the mic or speaker. The channel must be opened through a syscall from a trustworthy process. Then it stream data to it through high level functions like `audio_play` which allows a process to copy the data it wants to play to the buffer. Other processes using the same function to play can also copy their data to the buffer. The kernel driver code (on another thread) can then multiplex the incoming inputs either through hardware or a software FFT to combine the signals together and output to the MMIO speaker address
- Same idea for MIC, except we have to copy the data from the kernel buffer to each process listening on the MIC

For mouse and keyboard, we are primarily interested in capturing the inputs. But we dont want any program to be able to do that, e.g. a keylogger.

- The KB + Mouse can be discovered through ACPI and routed into a device table like the rest of the devices. But then the bootloader/kernel drivers can wrap around their MMIO and create listeners. Then the SBI or something can expose these listener interfaces for the Desktop Environment/ Window API to detect mouse movements and keyboard presses and make changes on a queued cycle
- Processes that want to be able to listen on mouse input must be `trusted` and open a communication/listener channel

## Notes

Only the user can change the `trusted` status of an application or executable. An process attempting a `syscall` without required permissions can then tell the Kernel to prompt the user to give the process the required permissions. It is up the user then. If they dont accept, the kernel returns an error and the program should be able to handle it (if they are using high level constructs). On a lower level though, they may crash if they then try to read from a nonexistent virtual address they thought they would have

# List of Services

Although there is no need to think of everything as a file, the concept of a file being openable, closable, readable, writable is quite good for many things

- For most devices, we can mount(dev_id) as a file and then open(filename) to initialise it
- Then we can `read` and `write` like usual, e.g. to a wifi card (socket), disk, speaker, mic, display, gpu. CPU and RAM access is defined implicitly in the code itself

## Rei

A generalised service interface using `extern rei`:

| Service | Rei API | Notes |
| --- | --- | --- |
| mount | `@restricted_level(5) fn mount(dev_id: u64, filepath: &str) -> ServiceStatus` | Can be used on almost any device with fd semantics |
| dismount | `@restricted_level(8) fn dismount(filepath: &String) -> ServiceStatus` | |
| open | `@restricted_level(7) fn open(filepath: String, flags: u8) -> (ServiceStatus, <fd>)` | Status = -1 on fail, e.g. no permissions. -2 on nonexistent filepath. flags -> RDONLY, RD_WRITE, APPEND, etc. |
| close | `@restricted_level(7) fn close(fd: u64) -> (ServiceStatus)` | Status = -1 on fail if fd doesnt exist. -2 if trying to close a fd not owned by the user or an untrustworthy process |
| read | `@restricted_level(7) fn read(fd: u64, nbytes: u64, buf: &String) -> ServiceStatus` | Status = -1 if no more space. String can grow dynamically. -2 if trying to overread or do something fishy |
| write | `@restricted_level(7) fn write(fd: u64, buf: &String, size: u64) -> ServiceStatus` | Same idea as read for trying to overwrite the current size of buf |
| lseek | `@restricted_level(7) fn lseek(fd: u64, offset: i64, type: SeekType) -> ServiceStatus` | Seek an open file descriptor to a new offset, useful mostly for storage devices |
| stat | `@restricted_level(3) fn stat(filepath: &String) -> (ServiceStatus, <FileStat>)` | |
| dup | `@restricted_level(7) fn dup(fd: u64) -> (ServiceStatus, <new_fd>)` | Duplicate an fd. Useful for two processes/threads reading/writing to the same file or resource |
| socket | `@restricted_level(3) fn socket() -> (ServiceStatus, <socket_fd>)` | Represents a endpoint for a channel of communication, usually between a server-client or just two peers |
| bindsocket | `@restricted_level(7) fn bindsocket(socket_fd, addr: SocketAddr) -> ServiceStatus` | Bind a name to the socket |
| send | `@restricted_level(7) fn send(socket_fd, buf: &String, flags: SendFlags) -> ServiceStatus` | Send an ASCII message stored in a userspace buffer to the address connected to by the socket. Needs either socket_bind or connect beforehand |
| connect | | Attempt to connect to an address, usually ipv4/6. Can be localhost:8000 for example |
| accept | | Accept an incoming request. Usually used in a loop with a queue of requests |
| spawn | `@restricted_level(7) fn spawn(executable_path: &String, args: &Vec<String>) -> (ServiceStatus, <pid>)` | Spawn a new process by creating a userspace container environment based on the executable's trustworthy level. execute it (elf64 only) and give it a priority of `PriorityDefault` |
| nice | `@restricted_level(10) fn nice(pid: u64, nice_level: i64) -> ServiceStatus` | Should be used by the user only. Kernel management should be relied on in most cases |
| time | `@restricted_level(1) fn time() -> (ServiceStatus, <u64>)` | Returns the number of seconds elapsed since 1970-01-01 00:00 |
| gettimeofday | `@restricted_level(1) fn gettimeofday() -> (ServiceStatus, <Timestamp>)` | Should be used with a high level construct like rust-std, which should use the VDSO version rather than the syscall |
| symlink | `@restricted_level(5) fn symlink(oldpath: &String, newpath: &String) -> ServiceStatus` | |
| chmod | `@restricted_level(10) fn chmod(path: &String, flags: ChownFlags) -> ServiceStatus` | |
| chown | `@restricted_level(10) fn chown(path: &String, u_id: u64) -> ServiceStatus` | |
| chdir | `@restricted_level(5) fn chdir(new_dir_path: &String) -> ServiceStatus` | |
| mmap | `@restricted_level(5) fn mmap(addr: *data, length: u64, flags: MMapFlags) -> ServiceStatus` | Maps a file to RAM (the process's vm). Works for a device using its `dev_id` too. If you want to `open` to read/write efficiently, can use mmap. Kernel handles writing back changes at idle time |
| heapalloc | `@restricted_level(3) fn heapalloc(new_dir_path: &String) -> ServiceStatus` | Basically brk() and sbrk() combined for anything malloc() related. For stack based data, the program should be able to as much as it wants directly `<= stack_limit` which is usually quite high or `unlimited` in virtual memory  |
| ulimit | `@restricted_level(1) fn ulimit() -> (ServiceStatus, ULimit)` | Often used to see the usage limits like RAM/stack space available to the current user/session |

### Notes

- some less contentious and safer services can be implemented as a VDSO
- `ServiceStatus` combines the generic return value of linux syscalls with the more helpful errno. Also includes a message in `ServiceStatus.message`
- the fd's are actually wrapped around Option<> but I cant be bothered writing it in, so Im shortcutting them as `<type>` instead of `Option<Type>`
- the semantic FS is mostly a userspace construct. It is a zip-file kind of thing with key: val `file_name: &EmberFile` relation

## RISC-V

A call interface with register level schemes using `extern asm`:

| Service | Asm API |
| --- | --- |
| mount | |
| open | |
| close | |
| read | |
| write | |
