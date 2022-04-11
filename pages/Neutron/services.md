---
layout: default
title: Services
parent: Neutron
---

## List of Services

Although there is no need to think of everything as a file, the concept of a file being openable, closable, readable, writable is quite good for many things

- For most devices, we can mount(dev_id) as a file and then open(filename) to initialise it
- Then we can `read` and `write` like usual, e.g. to a wifi card (socket), disk, speaker, mic, display, gpu. CPU and RAM access is defined implicitly in the code itself

## API

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
