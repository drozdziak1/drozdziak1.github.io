---
layout: post
title:  "DSP-17 #1: The kernel's role in an OS"
author: Stan Drozd
date: 2017-03-02 22:00:00 +0100
categories: dsp17
tags: kernel introduction anatomy tux penguin compilation dsp17
---
![Hey kids, wanna recompile some kernels?]({{ site.url }}/assets/recompile_some_kernels.jpg)
# So... you wanna do some Kernel Hacking?
If your answer (to both questions, preferably) is "yes" then you're probably in
the right place. But before we do anything, we'd better find out what it is that
we want to hack.

# What the hell is a kernel?
There's a couple definitions we could put up with, but most prominently, a
kernel is the centerpiece of every OS, it's the program that has the ultimate
control over all hardware and software present in a system. It uses this power
to provide a whole array of functionalities vital for an operating system to...
operate.

If we were to bake a cake from all things that make a computer work, we'd find
our kernel somewhere in the second layer - that sour jam supporting all the
fluffy software cream and covering its crusty bottom - the hardware, if you will
:upside_down_face:. If your OS was a country's government, the kernel would be
the president.

# What does it do, again?
My definition can sound a bit vague, but that's because kernels come in various
shapes and sizes - and so do their responsibilities. Typically, a kernel
supplies everything above it with things like process scheduling, memory
management, device drivers, access to filesystems or power management. Depending
on a kernel's design, the span of its jurisdiction might be something more than
that... or less!

When your machine is running, all the higher level, "regular" software governed
by your kernel does its thing in a place called **user space**, **user mode** or
**userland**.  The kernel prevents user space from getting involved in any funky
business, like writing to/reading from invalid memory locations or accessing
files and resources without sufficient permissions.

On the other hand, the environment in which the kernel executes is called
**kernel space** or **kernel mode**. In order to understand it, one should know
about a couple pitfalls and restrictions, which I'll describe in a moment,  but
first, let's have a look at the common types of kernels.

# Kernel architectures
Some of the most popular kernel designs involve **microkernels**, **monolithic**
kernels and **unikernels**. Apart from implementation-level details, the
three breeds mainly deviate in how much they actually *do* on behalf
of userland:

* **Microkernels** - a microkernel generally aims to push the most functionality
  out to the user space, thus usually making it the most modular of the three.
  Everything a microkernel doesn't do is performed by special user space
  services. An important goal of a microkernel is to be able to recover from
  driver errors by taking advantage of replaceable services that can be restarted
  when needed. A nice example is the famous Mach microkernel (yes, the same one
  Linus Torvalds refused to work on when Apple tried to hire him), widely known
  for being an ancestor to the kernels that power macOS and GNU [Hurd][hurd] (A
  Linux alternative from GNU, optionally available in Debian and Gentoo).

* **Monolithic kernels** - a monolithic kernel is the standard for UNIX-based
  operating systems. All of its subsystems exist within the same address space,
  thus minimizing the time spent on switches to user space and back. However,
  the common address space still allows modularity to an extent. A popular
  practice is to allow people to write loadable kernel modules, which can then
  be compiled and linked against a running kernel on the fly. The operation is
  reversible and will let us write a simple "Hello, world!" module in a future
  post. The most popular example of a monolithic kernel and the subject of this
  tutorial series is obviously Linux. There's also the FreeBSD kernel and the
  somewhat microkernel-ish NT kernel that powers Windows.

* **Unikernels** - this highly specialized kind of kernel could be thought of as
  the most controlling one. In fact, a unikernel in itself is nothing more than
  a library, against which an application is directly linked at compile time,
  thus creating a **uni**form OS image. Unikernels have the least flexibility of
  the three, for which they make up through outstandingly low resource
  consumption.

# Why do people think kernel development is difficult?
I wouldn't describe it as straight up difficult, but the job needs to be
performed by people who know what they're doing and have the creativity and
imagination to deal with situations where debugging options are limited. Here's
a couple examples of things to consider when dealing with kernel code:
* **The kernel space is mostly allergic to floating point variables** -
  actually, it's one of a kernel's many tasks to deal with floating point
  numbers, and with each such number mentioned somewhere in the code, the kernel
  has to pass it to an **FPU** (FPUs are secondary processors specialized in
  floating point arithmetics), a similar facility (like Intel's SSE CPU
  extensions) or emulate the operation entirely (expensive! :money_with_wings:).
  All three options take extra time for a mere convenience, with most use cases
  for floating point being completely replaceable. One easy way to avoid them is
  to designate some bits in an integer to the decimal part, which in fact is
  what a lot of hardware does anyway
* **Kernel space requires top-notch error handling** - one critical error (like
  a segfault) in the kernel is enough to compromise the whole system and force
  the user to reboot their machine. Not the nicest thing to happen to a server,
  right?
* **Pointers in kernel mode are not guaranteed to be valid in the userland** -
  When working with data passed by reference from or back to the user, it's
  crucial to consider a kernel's address translation API for the two modes.
  Failing to acknowledge that can lead to bugs that easily hide in plain sight.
* **Backdoors are lethal** - kernel backdoors make for the ultimate security
  threats. Imagine a USB driver bug which would let anyone escalate their
  privileges by burning a special sequence of bytes onto a pen drive and then
  plugging it into the target machine. All computers and USB-equipped embedded
  systems (which can often be impossible to upgrade) using that kernel would
  then be affected and prone to the attack (Wouldn't want my Wi-Fi router
  vulnerable to this kind of thing!)

# Penguin Anatomy 101, or what makes Linux Linux?:penguin::skull::mag:
Linux is a monolithic, UNIX-like kernel written mostly in C, which came to life
in the 90's under the fingertips of a Finnish Computer Science student by the
name of Linus Torvalds. He based his creation on the kernel present in the
[MINIX][minix] operating system. Upon sharing his kernel on the Internet, Linus
wrote a very, *Very*, **VERY** modest note:

```
    From: torvalds@klaava.Helsinki.FI (Linus Benedict Torvalds)
    Newsgroups: comp.os.minix
    Subject: What would you like to see most in minix?
    Summary: small poll for my new operating system
    Message-ID: <1991Aug25.205708.9541@klaava.Helsinki.FI>
    Date: 25 Aug 91 20:57:08 GMT
    Organization: University of Helsinki

    Hello everybody out there using minix –

    I’m doing a (free) operating system (just a hobby, won’t be big and
    professional like gnu) for 386(486) AT clones. This has been brewing
    since april, and is starting to get ready. I’d like any feedback on
    things people like/dislike in minix, as my OS resembles it somewhat
    (same physical layout of the file-system (due to practical reasons)
    among other things).

    I’ve currently ported bash(1.08) and gcc(1.40), and things seem to work.
    This implies that I’ll get something practical within a few months, and
    I’d like to know what features most people would want. Any suggestions
    are welcome, but I won’t promise I’ll implement them 🙂

    Linus (torvalds@kruuna.helsinki.fi)

    PS. Yes – it’s free of any minix code, and it has a multi-threaded fs.
    It is NOT protable (uses 386 task switching etc), and it probably never
    will support anything other than AT-harddisks, as that’s all I have :-(.
```

About the same time, [GNU][gnu], the original Free Software
nexus and also an operating system project, was in fact in need of a kernel.
However, their own creation - [Hurd][hurd] - was not yet ready to sufficiently
bring the project to life. Thus, GNU/Linux was born. Within a couple years, the
first distributions - [Slackware](slackware) and [Debian](debian) - appeared,
with many others to come. Tux the Penguin has become its official mascot:

![Tux](http://isc.tamu.edu/~lewing/linux/sit3-shine.7.gif)

# Conclusion
Linux proved itself over the years to be a reliable piece of software that
doesn't even cost a dime - the one FLOSS project to rule them all. What is more,
its extreme popularity managed, in a way, to shadow [GNU][gnu] itself. It even
resulted in the acknowledgement of the GNU and Linux' independence becoming a
meme:

![May I interject for a moment? ...](http://s2.quickmeme.com/img/b9/b91afe13fc7e1b79898b1f65a12b4d23a25d5083ec0410185ff563fdf8ce8a87.jpg)

*Actually, the text above is not an official quote on GNU's founder Richard
Stallman, it probably emerged from some Internet forum.*

The kernel supports a tiny bit more hardware than just [x86 and
AT-harddisks](http://www.linux-drivers.org/) and its distros run a little  more
software than [Bash 1.08 and GCC 1.40][popcon-stats].

I hope you liked what you read (and learned something). If you have anything on
your mind, please leave me a comment. Your feedback is very welcome. 

:penguin: :penguin: :penguin:

[minix]:https://pl.wikipedia.org/wiki/MINIX
[gnu]:https://www.gnu.org/
[hurd]:https://www.gnu.org/software/hurd/hurd.html
[debian]:www.debian.org/
[slackware]:www.slackware.com/
[popcon-stats]:http://popcon.debian.org/
