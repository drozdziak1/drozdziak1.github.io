---
layout: post
title: "Kernel Safari #4: The Source Tree of the Knowledge of Good and Evil"
author: Stan Drozd
date: 2017-08-16 17:00:00 +0200
categories: kernel-safari
excerpt: "I had an inner debate about whether Genesis or Dante's Inferno would
suit this topic better"
tags: tutorial kernel modules c compilation tree signing tour
---

> :information_source: Note:
>
> Treat this post as a map. Don't bother reading or memorizing all of this at
> once. Just skim through and find something interesting - it's about making
> things more approachable, not forcing them down your throat :smile:

Hiya! It's really good to be back. I took a small hiatus from the blog for a
couple weeks. I spent that time primarily learning about some of the cool stuff
that happens outside the kernel realm. I believe that too much specialization
can do to a brain what an undiversified diet does to a body. But now I'm back on
track!

Today we're going to talk about the kernel source directory and how to find your
way around it. We'll cover the basic purpose of every top-level directory and
talk more in-depth about some of them. Behold!

```plain
-rw-r--r--   1 drozdziak1 drozdziak1  18693 2016-09-07  COPYING
-rw-r--r--   1 drozdziak1 drozdziak1  98465 07-27 07:41 CREDITS
drwxr-xr-x 121 drozdziak1 drozdziak1  12288 07-27 07:41 Documentation
-rw-r--r--   1 drozdziak1 drozdziak1   2258 07-27 07:41 Kbuild
-rw-r--r--   1 drozdziak1 drozdziak1    252 2016-09-07  Kconfig
-rw-r--r--   1 drozdziak1 drozdziak1 420362 07-27 07:41 MAINTAINERS
-rw-r--r--   1 drozdziak1 drozdziak1  60210 07-27 07:41 Makefile
-rw-r--r--   1 drozdziak1 drozdziak1    722 02-27 10:30 README
drwxr-xr-x  32 drozdziak1 drozdziak1   4096 07-27 09:16 arch
drwxr-xr-x   3 drozdziak1 drozdziak1   4096 07-27 09:16 block
drwxr-xr-x   2 drozdziak1 drozdziak1   4096 07-27 09:16 certs
drwxr-xr-x   4 drozdziak1 drozdziak1  12288 07-27 09:16 crypto
drwxr-xr-x 132 drozdziak1 drozdziak1   4096 07-27 09:16 drivers
drwxr-xr-x  36 drozdziak1 drozdziak1   4096 07-27 09:16 firmware
drwxr-xr-x  74 drozdziak1 drozdziak1  12288 07-27 09:16 fs
drwxr-xr-x  28 drozdziak1 drozdziak1   4096 07-27 09:16 include
drwxr-xr-x   2 drozdziak1 drozdziak1   4096 07-27 09:16 init
drwxr-xr-x   2 drozdziak1 drozdziak1   4096 07-27 09:16 ipc
drwxr-xr-x  17 drozdziak1 drozdziak1  12288 07-27 09:16 kernel
drwxr-xr-x  12 drozdziak1 drozdziak1  20480 07-27 09:16 lib
drwxr-xr-x   3 drozdziak1 drozdziak1  12288 07-27 09:16 mm
drwxr-xr-x  69 drozdziak1 drozdziak1   4096 07-27 09:16 net
drwxr-xr-x  27 drozdziak1 drozdziak1   4096 07-27 07:41 samples
drwxr-xr-x  14 drozdziak1 drozdziak1   4096 07-27 09:16 scripts
drwxr-xr-x  10 drozdziak1 drozdziak1   4096 07-27 09:16 security
drwxr-xr-x  24 drozdziak1 drozdziak1   4096 07-27 09:16 sound
drwxr-xr-x  31 drozdziak1 drozdziak1   4096 07-27 07:41 tools
drwxr-xr-x   2 drozdziak1 drozdziak1   4096 07-27 09:16 usr
drwxr-xr-x   4 drozdziak1 drozdziak1   4096 07-27 09:16 virt
```

# Documentation/
It's hard to miss this one when going through the sources, `Documentation/`
houses the better part of Linux docs. Its topics span from development
environment tips, the [kernel development
process](https://www.kernel.org/doc/html/latest/process/development-process.html)
and patch exchange rules, all the way to the intricacies of how the actual code
works. As far as docs viewing goes, the modern approach is to use the
sphinx-generated documentation available under the `*docs` Make targets (see
`make help | grep docs` for more details) or one of the hosted instances like
[https://www.kernel.org/doc/html/latest](https://www.kernel.org/doc/html/latest).
To view a freshly compiled batch, see the `Documentation/output` directory in
your source tree.

# arch/
`arch/` is responsible for all things architecture-specific. Also, whenever you
build a kernel, it's where you'll most likely find the final kernel image
resulting from your build, e.g.  `arch/x86/boot/bzImage` for a typical x86
`defconfig` build.

# block/
This is the home of the Linux block layer and the related generic
implementations of block manipulation, I/O handling, scheduling, prioritization,
the relevant ioctl() requests etc.

# certs/
`certs/` holds the code responsible for [module
signing](https://www.kernel.org/doc/html/latest/admin-guide/module-signing.html)
\- a safety feature that lets the kernel verify authenticity of modules.

# crypto/
`crypto/` is the home of the kernel's cryptographic API, which consists of
different cipher implementations. Hardware-accelerated solutions can also use
`crypto/`'s common
[interface](http://elixir.free-electrons.com/linux/v4.12.5/source/crypto/algapi.c).

> :information_source: Note:
>
> The kernel crypto API also features algorithms other than ciphers, e.g.
> compression algs.

# drivers/
Probably the most significant directory in the whole project, if not only the
biggest (nearly half of the whole codebase). This is where all the
hardware/software chat is going, with lower-half drivers talking to their
upper-half counterparts and where the magic of hardware abstraction happens.

# firmware/
`firmware/` is just a big bag of firmware blobs, one of the few places where
there is no human-readable source code in the kernel codebase. If you understand
what loadable firmware is about, feel free to scroll to the next dir. If not,
prepare to learn a thing or two :slightly_smiling_face:

Imagine you have a USB stick - be it an LTE modem, a WiFi card or a DVB tuner.
Your device has a couple on-board chips, among which there's a write-protected
flash die for storing the device's firmware. Nothing unusual about that, huh?

**But!** What if your developers make a mistake or discover a security hole and
thousands of devices are to become vulnerable, with no other way out than
discarding them?

**But!** What if a wireless technology which your device works with grows very
fast and receives frequent updates? What if those would normally force you to
release new versions of your hardware more often than you can afford?

**But!** What if you're using a protocol that has its frequencies and signal
strength regulated in different countries? Would all of those regulations stay
the same forever? Could you afford a flash so big that all the configs
fit on a single device?

What if you could design your hardware so that **the OS can load a firmware
on-demand**? To answer the questions above, many manufacturers go even further
and choose to make it so that **the device's memory is volatile and it's the
OS's job to find and load the correct firmware image onto the device**, while
the hardware in itself only provides the basic mechanisms for firmware loading.

#### But why does firmware have its source closed?
Open-source software is all nice and dandy, but things can look different in
hardware manufacturing - source code can reveal how your hardware works, which
sooner or later will give you cheap knock-offs from the competition and
unsolicited reverse engineering of your product.

Given all that, Linux doesn't accept new firmware blobs into the source tree
anymore. If you have to use one, you'll usually have to supply it yourself and
keep it [in the
userspace](https://www.kernel.org/doc/html/latest/driver-api/firmware/direct-fs-lookup.html)
or build it into Linux [at
compile-time](https://www.kernel.org/doc/html/latest/driver-api/firmware/built-in-fw.html).

# fs/
In `fs/` you'll find the implementation of all the different filesystems that
Linux supports. Some of them are not tied to real drives (they're called
*pseudofilesystems*, e.g. `devtmpfs` used for `/dev/` or `sysfs` used for
`/sys/`) and exist purely in RAM. But pseudo- or not, their common denominator
is the interface known as VFS (Virtual File System) - the part of Linux that
lets you browse files on different partitions as if they were a part of a single
hierarchy.

# include/
Headers - And lots of them. If you've seen enough C/C++ projects, you should
roughly know what to look for in here. The public headers expose the APIs for
interaction with the kernel both from the inside and userspace.

# init/
Generic kernel startup code (the platform-specific stuff lies in
`arch/<your_architecture>`). This is where the kernel startup routines live,
like `startup_kernel()` - the kernel function that takes over after your machine
is done with architecture-specific provisioning.

# ipc/
`ipc/` is where inter-process communication code lives (what a twist! :smile:).
Pipes, shared memory, message queues is what you'll find there.

# kernel/
This is the home of all things that make Linux a real kernel:
* process scheduling, prioritization
* cgroups
* synchronization primitives
* high-level power management
* timekeeping
* debugging and profiling (of both the kernel and user programs)
* logging
* error handling
* module loading
* user and groups permissions management
* stuff specific to multicore machines

...and more!

`kernel/` has the potential to give you a great deal of insight about how things
are done under the hood.

# lib/
Helper functions - kernels in general have no use for the standard library.
Linux is no different here and it had to develop some functions of its own. If
you're looking for a generic implementation of a common operation, `lib/` is the
place to go. Things like [string manipulation
functions](http://elixir.free-electrons.com/linux/latest/source/lib/string.c),
[hash function
implementations](http://elixir.free-electrons.com/linux/v4.12.5/source/lib/sha1.c)
or [compression
algorithms](http://elixir.free-electrons.com/linux/v4.12.5/source/lib/lzo) can
be found inside. Some ciphers and compression algorithms hooked up to the crypto
API (`crypto/`) have their logic implemented here.

An interesting example of an algorithm from `lib/` are red-black trees, which
are a common data structure used in different process schedulers,
[kmemleak](https://www.kernel.org/doc/html/v4.10/dev-tools/kmemleak.html) and
more!

# mm/
Memory management - once you understand the acronym, `mm/`'s contents are no
longer a mystery. This directory holds the code for different memory allocators,
paging implementation, swap implementation, memory sharing mechanisms, memory
compression, talking to backing devices, DMA etc.

> :information_source: Note:
>
> `mm/` is also where the [Dirty
> COW](https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails)
> vulnerability was discovered.

# net/
Networking - every network protocol supported by Linux is kept here. But apart
from that, there's also the firewall infrastructure, the UNIX sockets
implementation, DNS cache, network statistics etc.

# samples/
Various code samples useful for testing different kernel APIs, like:
* [HID](http://elixir.free-electrons.com/linux/latest/source/samples/hidraw/hid-example.c)
* [Hardware
  breakpoints](http://elixir.free-electrons.com/linux/latest/source/samples/hw_breakpoint/data_breakpoint.c)
* [The kernel
  debugger](http://elixir.free-electrons.com/linux/latest/source/samples/kdb/kdb_hello.c)
* [Video](http://elixir.free-electrons.com/linux/latest/source/samples/v4l/v4l2-pci-skeleton.c)
* [The packet filter
  format](http://elixir.free-electrons.com/linux/latest/source/samples/bpf/README.rst)
* [The
  watchdog](http://elixir.free-electrons.com/linux/latest/source/samples/watchdog/watchdog-simple.c)
* Various system calls

# scripts/
Helper scripts for making the work around the project a little easier
on the developer. Prominent examples include:
* `checkpatch.pl` - a nagging friend of every Linux kernel developer, its
  purpose is to find obvious patch bloopers like coding style violations or the
  commit message format in patches or changes to the repo tracked by git
* `get_maintainer.pl` - a helper script for finding the maintainers and
  mailing lists connected to the specified file(s). When you find a bug or add
  code which you'd like to send a patch for, it'll tell you where to send it
  and who to CC
* `coccicheck` - a
  [Coccinelle](https://www.kernel.org/doc/html/latest/dev-tools/coccinelle.html)
  script for semantic analysis of the kernel sources.

# security/
Security modules - this directory contains the different means for OS hardening
that Linux supports, including
[SELinux](https://www.kernel.org/doc/html/latest/admin-guide/LSM/SELinux.html),
[AppArmor](https://www.kernel.org/doc/html/latest/admin-guide/LSM/apparmor.html)
and [others](https://www.kernel.org/doc/html/latest/admin-guide/LSM/index.html).

# sound/
The home of all the major sound systems supported by Linux, including ALSA, OSS
and Jack.

# tools/
Userland helper tools and test programs - this directory is very similar to
`samples/` but its contents are more focused on usable solutions than examples
of how the underlying kernel code works.

# usr/
You may remember this directory from the time when we [generated a kernel image
for KVM]({% post_url 2017-03-14-DSP17-2 %}). For some reason, it was decided
that only the tool for generating initcpio's should live here.

# virt/
Host-side KVM implementation.

## Where them syscalls/namespaces at?
Some parts of Linux don't have a centralized location in the codebase, with
system calls and namespaces being two notable examples.

You probably know what system calls are, but in case you don't, they're
basically a collection of special procedures that userspace programs use to make
each and every request from the operating system. There's about 300 of them, and
to call one, your Linux program sets a designated register to the system call
number and then specifies the call's arguments according to the architecture's
system call [calling
convention](https://en.wikipedia.org/wiki/Calling_convention) (see the
"Architecture calling conventions" section of `man 2 syscall` for more
information). Finally, the process requests a context switch to kernelspace,
e.g. in x86 by invoking software interrupt No. 128:

```assembly
movl    eax, 4          ; Choose the write() system call number
movl    ebx, 1          ; Choose file descriptor 1 a.k.a. stdout
movl    ecx, 0xdeadbeef ; Choose a pointer to a string at address 0xdeadbeef
movl    edx, 12         ; Specify the string's length as 12 bytes
int     0x80            ; Invoke interrupt 128, the kernel takes over now
```
*write(1, "some message", 12)*

Because they appeal to many different subsystems, syscalls are defined all over
the kernel sources using a special set of macros which combine them into a list
that pairs each one up with a syscall number.

> :information_source: Note:
>
> If you're curious about the system calls used by your program, check out
> `strace` - a tool for dumping all syscalls used by your application as they're
> called. The program is especially great for reverse engineering of
> closed-source applications and low-level debugging.

Namespaces together with cgroups form a foundation for Linux OS-level
virtualization (containers). Namespaces work by exposing only the selected
resources to a process that we want to isolate. To date, the namespace-ified
subsystems include:
```plain
Namespace   Constant          Isolates
Cgroup      CLONE_NEWCGROUP   Cgroup root directory
IPC         CLONE_NEWIPC      System V IPC, POSIX message queues
Network     CLONE_NEWNET      Network devices, stacks, ports, etc.
Mount       CLONE_NEWNS       Mount points
PID         CLONE_NEWPID      Process IDs
User        CLONE_NEWUSER     User and group IDs
UTS         CLONE_NEWUTS      Hostname and NIS domain name
```
*See* `man 7 namespaces` *for more details*

# Conclusion
This post required a lot of effort, but here it is! Working on this was real fun
and it also gave me a reason to learn more about the source tree myself. For the
next post, I'm planning to do some bug hunting in the kernel and write up a
little report.

:deciduous_tree: :evergreen_tree: :palm_tree:
