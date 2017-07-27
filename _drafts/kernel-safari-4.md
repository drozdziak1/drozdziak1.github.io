---
layout: post
title: "Kernel Safari #4: The Linux kernel source tree explained"
author: Stan Drozd
date: 2017-07-31 08:00:00 +0200
categories: dsp17
excerpt: "Climb the tree, seize the fruit"
tags: dsp17 tutorial kernel modules c compilation
---

Hiya! At this point you must have noticed that I took a small hiatus from the
blog for a couple weeks. I spent that time primarily on learning about some of
the cool stuff that happens outside the kernel realm. I believe that too much
specialization can do to a brain what an undiversified diet does to a body.

Today we're going to talk about the kernel source directory and how to find your
way around it. We'll cover the basic purpose of every top-level directory and
talk more in-depth about the ones which you'll be peeking into the most often.

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
works. As far as docs viewing goes, the modern approach is to use the Sphinx
generated RST docs available under the `*docs` Make targets (see `make help |
grep docs` for more details) or one of the hosted instances like
[https://www.kernel.org/doc/html/latest](https://www.kernel.org/doc/html/latest)

# arch/
You probably won't be surprised to hear that `arch/` is responsible for all
things architecture-specific. This is where
