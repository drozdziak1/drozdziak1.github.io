---
layout: post
title: "Kernel Safari #4: The (source) Tree of the Knowledge of Good and Evil"
author: Stan Drozd
date: 2017-07-31 08:00:00 +0200
categories: kernel-safari
excerpt: "Will you dare to seize the forbidden fruit?"
tags: tutorial kernel modules c compilation tree signing
---

Hiya! It trully is good to be back. I took a small hiatus from the blog for a
couple weeks, and I spent that time primarily on learning about some of the cool
stuff that happens outside the kernel realm. I believe that too much
specialization can do to a brain what an undiversified diet does to a body.

Today we're going to talk about the kernel source directory and how to find your
way around it. We'll cover the basic purpose of every top-level directory and
talk more in-depth about the ones which you'll be peeking into the most often.
Also, it's a good idea to talk about the patterns occuring in the directory
layout in different places around the source tree.

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
sphinx-generated RST docs available under the `*docs` Make targets (see `make
help | grep docs` for more details) or one of the hosted instances like
[https://www.kernel.org/doc/html/latest](https://www.kernel.org/doc/html/latest).
To view a freshly compiled batch of docs, see the `Documentation/output`
directory in your source tree.

# arch/
`arch/` is responsible for all things architecture-specific. Also, whenever you
build a kernel, it's often where you'll find the final kernel image resulting
from your build, e.g.  `arch/x86/boot/bzImage` for a typical x86 defconfig
build.

# block/
Block devices stuff

# certs/
`certs/` holds the code responsible for [module
signing](https://www.kernel.org/doc/html/latest/admin-guide/module-signing.html)
\- a safety feature that prevents the kernel from loading unauthorized modules.

# crypto/
`crypto/` is the home of the kernel's cryptographic API, which consists of
different cipher implementations and the code to make use of them; it's where
hardware crypto capabilities get hooked up with your OS.

> :information_source: Note:
>
> The kernel crypto API also implements different algorithms than ciphers, e.g.
> compression algs.

# drivers/
Probably the most important directory in the whole project, if not only the
biggest (at whopping 513MB of source code, nearly half of the whole codebase).
This is where all the hardware/software chat is going, where lower-half drivers
meet with their upper-half counterparts, where the magic of hardware abstraction
happens.

# firmware/
`firmware/` is the home of firmware blobs, one of the few places where
there is no human-readable source code in the kernel codebase. If you understand
what the bit about blobs means, feel free to scroll to the next dir. If not,
prepare to learn a thing or two :slightly_smiling_face:

Imagine you have a USB stick - be it an LTE modem, a WiFi card or a DVB tuner.
Your device has a couple on-board chips, among which there's a write-protected
flash die for storing the device's firmware. Nothing unusual about that, huh?

**But!** What if your developers make a mistake or discover a security hole and
thousands of devices were to become vulnerable, with no other way out than
discarding them?

**But!** What if a wireless technology which your device works with is growing
very fast and receiving frequent updates? What if those would normally force you
to release new versions of your hardware more often than you can afford?

**But!** What if your hardware needs to work with tens of different
configurations depending e.g. on the target country? Like WiFi cards do with
respect to restricted radio frequency
[regulations](https://wireless.wiki.kernel.org/en/developers/Regulatory)? Would
all of those regulations stay unchanged over the years? Could you afford a flash
so big that all the configs fit on a single device?

What if you could design your hardware so that **the OS can load a firmware
on-demand**? To answer the qeustions above, many manufacturers go even further
and choose to make it so that **the device's memory is volatile and it's the
OS's job to find and load the correct firmware image onto the device**, while
the hardware in itself only provides the basic mechanisms for firmware loading.

#### But why does firmware have its source closed?
There's one thing you need to know about hardware manufacturing: source code can
reveal how your hardware works, which inherently is how you get cheap knock-offs
from the competition and unsolicited reverse engineering of your product.

# fs/
This place is special, because it describes the implementation of all the
different filesystems that Linux supports. Some of them are not tied to real
drives (they're called *pseudofilesystems*, e.g. `devtmpfs` used for `/dev/` or
`sysfs` used for `/sys/`), some Filesystems are implemented here, along with
their common abstraction layer known as VFS (Virtual File System).
