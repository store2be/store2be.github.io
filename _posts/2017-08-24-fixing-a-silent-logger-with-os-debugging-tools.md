---
date: 2017-08-24
categories:
- Debugging
- Kubernetes
- Rust
title: Fixing a silent logger with OS debugging tools
author_staff_member: 03_tom
medium_link: https://medium.com/store2be-tech/fixing-a-silent-logger-with-os-debugging-tools-b4106ac45f54
practical_dev_link: https://dev.to/tomhoule/fixing-a-silent-logger-with-osdebugging-tools
---

One of the features we wanted in the recent push to release a [0.2 version of
Papers](https://github.com/store2be/pape-rs/releases/tag/0.2.0) was debugging
output uploaded to an S3 bucket for generated documents. One of the major
things we want in that debug archive is the logs from the service itself - this
means having a second sink for our logs on a per-document-generation basis,
ideally with a higher log level. Fortunately, this is possible in a
boilerplate-free way with [slog](https://github.com/slog-rs/slog) 2.0, since
loggers are now also sinks.

So we decided to update to slog 2 and rework our logs configuration, which is
just a few lines anyway. There is even a nice high level wrapper around slog
called [sloggers](https://github.com/sile/sloggers) for logging out to files or
the terminal with nice colors.

It went very smoothly and the result was impeccable, but one does not simply
containerize an app that logs to the terminal: in our test deployment, the
service was running but there was absolutely no terminal output.

After checking that yes, release binaries do log in the same docker image
locally, and investigating several suspects, including fluentd and slog
behaving differently in release builds (this is [documented
behaviour](https://docs.rs/slog/2.0.9/slog/#notable-details)), I resorted to
one of my favourite debugging tools:
[strace](https://en.wikipedia.org/wiki/Strace).

`strace <your program>` runs your program and prints whatever system calls it
makes to the terminal, allowing you to inspect all IO operations. For Papers,
you will get something like that (excerpt):

```
open("/usr/lib/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\340\5\2\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1985472, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5f6ca3d000
mmap(NULL, 3823824, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f5f6b57b000
mprotect(0x7f5f6b718000, 2093056, PROT_NONE) = 0
mmap(0x7f5f6b917000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x19c000) = 0x7f5f6b917000
mmap(0x7f5f6b91d000, 14544, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f5f6b91d000
close(3)                                = 0
open("/usr/lib/libm.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0`]\0\0\0\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1120216, ...}) = 0
mmap(NULL, 3215384, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f5f6b269000
mprotect(0x7f5f6b37a000, 2093056, PROT_NONE) = 0
mmap(0x7f5f6b579000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x110000) = 0x7f5f6b579000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5f6ca3a000
arch_prctl(ARCH_SET_FS, 0x7f5f6ca3a8c0) = 0
mprotect(0x7f5f6b917000, 16384, PROT_READ) = 0
mprotect(0x7f5f6b579000, 4096, PROT_READ) = 0
mprotect(0x7f5f6bb36000, 4096, PROT_READ) = 0
mprotect(0x7f5f6bd50000, 4096, PROT_READ) = 0
mprotect(0x7f5f6bf5c000, 4096, PROT_READ) = 0
mprotect(0x7f5f6c160000, 4096, PROT_READ) = 0
mprotect(0x7f5f6c5b7000, 122880, PROT_READ) = 0
mprotect(0x7f5f6c842000, 20480, PROT_READ) = 0
mprotect(0x55873cd6f000, 1167360, PROT_READ) = 0
mprotect(0x7f5f6ca6f000, 4096, PROT_READ) = 0
munmap(0x7f5f6ca41000, 185208)          = 0
set_tid_address(0x7f5f6ca3ab90)         = 13360
set_robust_list(0x7f5f6ca3aba0, 24)     = 0
rt_sigaction(SIGRTMIN, {sa_handler=0x7f5f6bb3d740, sa_mask=[], sa_flags=SA_RESTORER|SA_SIGINFO, sa_restorer=0x7f5f6bb497e0}, NULL, 8) = 0
rt_sigaction(SIGRT_1, {sa_handler=0x7f5f6bb3d7e0, sa_mask=[], sa_flags=SA_RESTORER|SA_RESTART|SA_SIGINFO, sa_restorer=0x7f5f6bb497e0}, NULL, 8) = 0
rt_sigprocmask(SIG_UNBLOCK, [RTMIN RT_1], NULL, 8) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
readlink("/etc/malloc.conf", 0x7ffcda2ec870, 4096) = -1 ENOENT (No such file or directory)
brk(NULL)                               = 0x55873d340000
mmap(NULL, 2097152, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5f6b069000
munmap(0x7f5f6b069000, 2097152)         = 0
mmap(NULL, 4190208, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5f6ae6a000
munmap(0x7f5f6ae6a000, 1662976)         = 0
munmap(0x7f5f6b200000, 430080)          = 0
open("/sys/devices/system/cpu/online", O_RDONLY|O_CLOEXEC) = 3
read(3, "0-3\n", 8192)                  = 4
close(3)                                = 0
rt_sigaction(SIGPIPE, {sa_handler=SIG_IGN, sa_mask=[PIPE], sa_flags=SA_RESTORER|SA_RESTART, sa_restorer=0x7f5f6b5ae940}, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=0}, 8) = 0
mmap(NULL, 2097152, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5f6ae00000
open("/proc/self/maps", O_RDONLY|O_CLOEXEC) = 3
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
fstat(3, {st_mode=S_IFREG|0444, st_size=0, ...}) = 0
read(3, "55873c5e7000-55873cb6f000 r-xp 0"..., 1024) = 1024
read(3, "libc-2.25.so\n7f5f6b917000-7f5f6b"..., 1024) = 1024
read(3, "pthread-2.25.so\n7f5f6bd52000-7f5"..., 1024) = 1024
read(3, "so.1.1\n7f5f6c5b7000-7f5f6c5d5000"..., 1024) = 1024
read(3, "000-7ffcda2f0000 rw-p 00000000 0"..., 1024) = 316
close(3)                                = 0
sched_getaffinity(13360, 32, [0, 1, 2, 3]) = 16
mmap(0x7ffcd9af0000, 4096, PROT_NONE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7ffcd9af0000
rt_sigaction(SIGSEGV, {sa_handler=0x55873ca20260, sa_mask=[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_SIGINFO, sa_restorer=0x7f5f6bb497e0}, NULL, 8) = 0
rt_sigaction(SIGBUS, {sa_handler=0x55873ca20260, sa_mask=[], sa_flags=SA_RESTORER|SA_ONSTACK|SA_SIGINFO, sa_restorer=0x7f5f6bb497e0}, NULL, 8) = 0
sigaltstack(NULL, {ss_sp=NULL, ss_flags=SS_DISABLE, ss_size=0}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f5f6ca6d000
sigaltstack({ss_sp=0x7f5f6ca6d000, ss_flags=0, ss_size=8192}, NULL) = 0
getcwd("/home/tom/src/papers", 512)     = 21
open("/home/tom/src/papers/.env", O_RDONLY|O_CLOEXEC) = 3
ioctl(3, FIOCLEX)                       = 0
read(3, "PAPERS_AWS_ACCESS_KEY=<REDACTED>"..., 8192) = 228
getrandom("", 0, GRND_NONBLOCK)         = 0
getrandom("\x3d\xfd\x3c\x08\x48\x6d\x03\x13", 8, GRND_NONBLOCK) = 8
getrandom("\x80\x11\x02\x43\x2e\xa6\x6f\x1b", 8, GRND_NONBLOCK) = 8
read(3, "", 8192)                       = 0
close(3)                                = 0
stat("/home/tom/.terminfo", 0x7ffcda2eb7d0) = -1 ENOENT (No such file or directory)
stat("/etc/terminfo", 0x7ffcda2eb7d0)   = -1 ENOENT (No such file or directory)
stat("/lib/terminfo", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
stat("/lib/terminfo/s/screen", {st_mode=S_IFREG|0644, st_size=1587, ...}) = 0
open("/lib/terminfo/s/screen", O_RDONLY|O_CLOEXEC) = 3
ioctl(3, FIOCLEX)                       = 0
read(3, "\32\1*\0+\0\20\0i\1z\2screen|VT 100/ANSI X"..., 8192) = 1587
close(3)                                = 0
stat("/home/tom/.terminfo", 0x7ffcda2eb790) = -1 ENOENT (No such file or directory)
stat("/etc/terminfo", 0x7ffcda2eb790)   = -1 ENOENT (No such file or directory)
stat("/lib/terminfo", {st_mode=S_IFDIR|0755, st_size=4096, ...}) = 0
stat("/lib/terminfo/s/screen", {st_mode=S_IFREG|0644, st_size=1587, ...}) = 0
open("/lib/terminfo/s/screen", O_RDONLY|O_CLOEXEC) = 3
ioctl(3, FIOCLEX)                       = 0
read(3, "\32\1*\0+\0\20\0i\1z\2screen|VT 100/ANSI X"..., 8192) = 1587
close(3)                                = 0
ioctl(1, TCGETS, {B38400 opost isig icanon echo ...}) = 0
futex(0x7f5f6c161048, FUTEX_WAKE_PRIVATE, 2147483647) = 0
mmap(NULL, 2101248, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS|MAP_STACK, -1, 0) = 0x7f5f6abff000
mprotect(0x7f5f6abff000, 4096, PROT_NONE) = 0
clone(child_stack=0x7f5f6adfee30, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7f5f6adff9d0, tls=0x7f5f6adff700, child_tidptr=0x7f5f6adff9d0) = 13361
epoll_create1(EPOLL_CLOEXEC)            = 3
pipe2([4, 5], O_NONBLOCK|O_CLOEXEC)     = 0
epoll_ctl(3, EPOLL_CTL_ADD, 4, {EPOLLIN|EPOLLET, {u32=4294967295, u64=18446744073709551615}}) = 0
socket(AF_INET, SOCK_STREAM|SOCK_CLOEXEC, IPPROTO_IP) = 6
setsockopt(6, SOL_SOCKET, SO_REUSEADDR, [1], 4) = 0
```

It looks scary at first, but you absolutely don't need to know what every
system call does to gain insight from it (if you wonder, they all have manpages
so you can go `man futex`).

You can follow the program linearly, and it's easy to spot things like [the
moment we call `dotenv` in
`config.rs`](https://github.com/store2be/pape-rs/blob/1f4aeb791f14662ce5bcfe84aa17e4e58a7f3bbe/src/config.rs#L60)
In this strace output it's:

```
open("/home/tom/src/papers/.env", O_RDONLY|O_CLOEXEC) = 3
```

We also have, at the end of the trace, the moment where the server starts
listening (it's binding to a TCP socket). `epoll` is used by the tokio event
loop, so our logger configuration is happening before that, and after dotenv.

A lot of calls in-between are for reading the terminfo database to figure out
the capabilities of the terminal.

This was missing in the strace output inside the container in our kubernetes
cluster.

...

And that was it: when we set the `TERM` environment variable, our log output is back.

This is because when not specified, kubernetes does not allocate a TTY to
containers, so `TERM` is not set, and since sloggers does not set up terminal
output without it, nothing got printed.

Our solution was to set `TERM` to `screen`, although it can also be set to
`xterm` or `dumb`. Another possible solution would be to configure the
deployment to allocate a TTY to the container (similar to the `-t` flag in
`docker run` and `docker exec`), but this is not necessary for simple logging.

This post is a testimony to the usefulness of the OS debugging tools and strace
especially. There is a very good series of posts by Julia Evans on the many
tracing tools for Linux, I would recommend them to anyone writing server-side
software:

- [A zine about strace](https://jvns.ca/blog/2015/04/14/strace-zine/)
- [Linux tracing systems and how they fit together](https://jvns.ca/blog/2017/07/05/linux-tracing-systems/)
- [ftrace: trace your kernel functions](https://jvns.ca/blog/2017/03/19/getting-started-with-ftrace/)
