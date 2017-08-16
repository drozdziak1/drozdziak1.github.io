---
layout: post
title: "Kernel Safari #3.2: Security Fest 2017 CTF - the 2bright challenge
writeup"
author: Stan Drozd
date: 2017-06-02 17:00:00 +0200
categories: kernel-safari
excerpt: "Solving CTF challenges turns out to be very exciting! Let's see what
Security Fest 2017 had in store for the interesting challenge which almost
no-one could solve"
tags: ctf securityfest reverse engineering
---
Long time no see! I finally got myself to write something for the blog in a
```c
long long time;
```

As you can see, this post has a different prefix, and that's because the [Daj
Się Poznać](http://www.dajsiepoznac.pl) (a.k.a. "Get Noticed!") competition is
over. I didn't make it till the end, but starting this blog was a great
experience for me, and it wouldn't ever happen if not for the competition. From
now on I'll be posting under the label of **Kernel Safari**.

Let's see what the `Security Fest` 2017 CTF's `2bright` challenge was about, but
first...

# What's a CTF?
"Capture The Flag"s (or CTFs for short) are sets of computer security challenges in
which teams of participants have to obtain passwords (flags) hidden in various
digital shapes and forms - be it insecure websites which you have to hack,
ordinary files with hidden content, encrypted media or executables. They aim to
hone the skills of pentesters, reverse engineers, digital forensics experts,
hackers and other security enthusiasts in a perfectly safe and legal way. 

CTFs are often tied up with conferences and security events. Some are even meant
to sieve potential candidates for [security
jobs](https://join.eset.com/en/challenges/malware-analyst).

A couple weeks ago, I joined a team called
[OpenToAll](https://www.reddit.com/r/OpenToAllCTFteam/) which, as the name
suggests, is open for anyone without a team who would like to participate in a
CTF. Two days ago, I finally seized the opportunity to join OpenToAll in my
first CTF, hosted by [Security
Fest](https://twitter.com/securityfest). Our team got the 3rd place!

# The challenge
My contribution to our score was through `2bright`, a challenge consisting of a
`*.tar.gz` archive with one file inside. Upon downloading the file we're given a
hint:

> In ancient times, giants ruled the world. Thought long gone, some giants has
> once again appeard, and might have even been here all the time. And though old
> still shine bright - maybe too bright? Note: Flag does not follow the format

# My solution
Let's untar the file! Doing that leaves us with a file called `2bright`, which
when passed to `file` was recognized as a `MMDF mailbox`. After a quick look at `man
mmdf` I was quite confident that `file`'s diagnosis was incorrect. Into the hex editor it goes!

The file starts with a sequence that seems to specify it's total (16KB) size:
```plain
[0x00000000 0% 2128 Pobrane/2bright]> xc @ fcn.00000000
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF  comment
0x00000000  0108 0c08 0a00 9e31 3633 3834 0000 0000  .......16384....
0x00000010  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x00000020  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x00000030  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x00000040  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x00000050  0000 0000 0000 0000 0000 0000 0000 0000  ................
```
*My hex editor of choice is* `radare2`*, check it out, it's awesome!*

After a bunch of zeros, at offset `0x1801`, a weird sequence of descending bytes
appeared:
```plain
[0x000017f0 40% 2128 Pobrane/2bright]> xc
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF  comment
0x000017f0  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x00001800  00fe 7d80 ed7e 7bea 7b78 e774 75e4 0d0a  ..}..~{.{x.tu...
0x00001810  e172 6ffe 6f6c fb68 69f8 6566 f566 9de2  .ro.ol.hi.ef.f..
0x00001820  e1e0 dfde dddc dbda d9d8 d7d6 d5d4 d3d2  ................
0x00001830  d1d0 cfce cdcc cbca c9c8 c7c6 c5c4 c3c2  ................
0x00001840  c1c0 c0fe bdfc fbba f9f8 b7f6 f5b4 cbf2  ................
0x00001850  b1f0 efae edec abea e9a8 e7e6 a5e4 dca2  ................
0x00001860  a1a0 9f9e 9d9c 9b9a 9998 9796 9594 9392  ................
0x00001870  9190 8f8e 8d8c 8b8a 8988 8786 8584 8382  ................
0x00001880  81fc 037e fffe 7bf8 fb78 f5f6 458a ef42  ...~..{..x..E..B
0x00001890  f3f2 6fec ef5c e9e8 59ea e566 e718 6362  ..o..\..Y..f..cb
0x000018a0  6160 5f5e 5d5c 5b5a 5958 5756 5554 5352  a`_^]\[ZYXWVUTSR
0x000018b0  5150 4f4e 4d4c 4b4a 4948 4746 4544 4342  QPONMLKJIHGFEDCB
0x000018c0  4140 4042 3d7c b93a 79ba 3776 b734 4bcc  A@@B=|.:y.7v.4K.
0x000018d0  3170 ad2e 6dae 2b6a ab28 67a4 2564 a122  1p..m.+j.(g.%d."
0x000018e0  2120 1f1e 1d1c 1b1a 1918 1716 1514 1312  ! ..............
0x000018f0  1110 0f0e 0d0c 0b0a 0908 0706 0504 0302  ................
0x00001900  013e 81be f5bd bbf2 b8b8 ffb7 b5fc 8db2  .>..............
0x00001910  f9a0 afe6 a5ac e3ae a9e0 a5a6 dba5 9ce2  ................
0x00001920  e1e0 dfde dddc dbda d9d8 d7d6 d5d4 d3d2  ................
0x00001930  d1d0 cfce cdcc cbca c9c8 c7c6 c5c4 c3c2  ................
0x00001940  c1fe c33c b53e 39b2 3b3a bf36 37bc 2f4c  ...<.>9.;:.67./L
0x00001950  b932 2da6 2f2e a328 2ba0 2524 9bd8 21a2  .2-./..(+.%$..!.
0x00001960  a1a0 9f9e 9d9c 9b9a 9998 9796 9594 9392  ................
0x00001970  9190 8f8e 8d8c 8b8a 8988 8786 8584 8382  ................
0x00001980  817e 7f00 6d7c 7a6a 7979 6776 7464 736c  .~..m|zjyygvtdsl
0x00001990  6170 6e7e 6d6d 7b6a 6878 6767 7564 1d62  apn~mm{jhxggud.b
0x000019a0  6160 5f5e 5d5c 5b5a 5958 5756 5554 5352  a`_^]\[ZYXWVUTSR
0x000019b0  5150 4f4e 4d4c 4b4a 4948 4746 4544 4342  QPONMLKJIHGFEDCB
0x000019c0  417e 7e40 5c7d 7a6b 7879 6608 747d 3b4c  A~~@\}zkxyf.t};L
0x000019d0  7838 7f6b 2564 6e22 6d6b 2f64 1b2c 6222  x8.k%dn"mk/d.,b"
0x000019e0  2120 1f1e 1d1c 1b1a 1918 1716 1514 1312  ! ..............
0x000019f0  1110 0f0e 0d0c 0b0a 0908 0706 0504 0302  ................
0x00001a00  0100 0000 0000 0000 0000 0000 0000 0000  ................
0x00001a10  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x00001a20  0000 0000 0000 0000 0000 0000 0000 0000  ................
```
*The hell?*

More and more zeroes, and...

```plain
[0x00003800 93% 3528 Pobrane/2bright]> xc
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF  comment
0x00003800  00a9 018d 20d0 8d21 d0a9 178d 18d0 a200  .... ..!........
0x00003810  a920 9d00 049d 0005 9d00 069d 0007 e8d0  . ..............
0x00003820  f1a2 00a9 019d 00d8 9d00 d99d 00da 9d00  ................
0x00003830  dbe8 d0f1 a21c a000 bd28 43c9 6030 02e9  .........(C.`0..
0x00003840  6099 0604 c8ca 10f0 a200 a000 985d 0020  `............].
0x00003850  9d00 2098 5d00 219d 0021 c8ca d0ee a257  .. .].!..!.....W
0x00003860  8e00 d0a2 6f8e 02d0 a287 8e04 d0a2 9f8e  ....o...........
0x00003870  06d0 a2b7 8e08 d0a2 cf8e 0ad0 a2e7 8e0c  ................
0x00003880  d0a2 ff8e 0ed0 a080 8c01 d08c 03d0 8c05  ................
0x00003890  d08c 07d0 8c09 d08c 0bd0 8c0d d08c 0fd0  ................
0x000038a0  a980 8df8 07a9 818d f907 a982 8dfa 07a9  ................
0x000038b0  838d fb07 a984 8dfc 07a9 858d fd07 a986  ................
0x000038c0  8dfe 07a9 878d ff07 a901 8d27 d08d 28d0  ...........'..(.
0x000038d0  8d29 d08d 2ad0 8d2b d08d 2cd0 8d2d d08d  .)..*..+..,..-..
0x000038e0  2ed0 a9ff 8d15 d078 a90c 8d14 03a9 418d  .......x......A.
0x000038f0  1503 0e19 d0a9 7b8d 0ddc a981 8d1a d0a9  ......{.........
0x00003900  1b8d 11d0 a910 8d12 d058 4c09 410e 19d0  .........XL.A...
0x00003910  ad11 42d0 12ae 1042 bd28 428d 01d0 ee10  ..B....B.(B.....
0x00003920  42ad 1242 8d11 42ce 1142 ad14 42d0 12ae  B..B..B..B..B...
0x00003930  1342 bd28 428d 03d0 ee13 42ad 1542 8d14  .B.(B.....B..B..
0x00003940  42ce 1442 ad17 42d0 12ae 1642 bd28 428d  B..B..B....B.(B.
0x00003950  05d0 ee16 42ad 1842 8d17 42ce 1742 ad1a  ....B..B..B..B..
0x00003960  42d0 12ae 1942 bd28 428d 07d0 ee19 42ad  B....B.(B.....B.
0x00003970  1b42 8d1a 42ce 1a42 ad1d 42d0 12ae 1c42  .B..B..B..B....B
0x00003980  bd28 428d 09d0 ee1c 42ad 1e42 8d1d 42ce  .(B.....B..B..B.
0x00003990  1d42 ad20 42d0 12ae 1f42 bd28 428d 0bd0  .B. B....B.(B...
0x000039a0  ee1f 42ad 2142 8d20 42ce 2042 ad23 42d0  ..B.!B. B. B.#B.
0x000039b0  12ae 2242 bd28 428d 0dd0 ee22 42ad 2442  .."B.(B...."B.$B
0x000039c0  8d23 42ce 2342 ad26 42d0 12ae 2542 bd28  .#B.#B.&B...%B.(
0x000039d0  428d 0fd0 ee25 42ad 2742 8d26 42ce 2642  B....%B.'B.&B.&B
0x000039e0  ce46 43ad 4643 d023 ad47 438d 4643 ae45  .FC.FC.#.GC.FC.E
0x000039f0  43bd 4843 a01d 9906 d888 10fa ee45 43ad  C.HC.........EC.
0x00003a00  4543 c962 d005 a900 8d45 4368 a868 aa68  EC.b.....ECh.h.h
0x00003a10  4000 0101 1001 0120 0101 3801 0140 0101  @...... ..8..@..
0x00003a20  5001 0160 0101 7001 0196 9593 918f 8d8b  P..`..p.........
0x00003a30  8987 8583 817f 7d7c 7a78 7674 7371 6f6d  ......}|zxvtsqom
0x00003a40  6c6a 6867 6564 6261 5f5e 5d5b 5a59 5756  ljhgedba_^][ZYWV
0x00003a50  5554 5352 5150 4f4e 4d4d 4c4b 4b4a 4949  UTSRQPONMMLKKJII
0x00003a60  4848 4847 4747 4747 4746 4747 4747 4747  HHHGGGGGGFGGGGGG
0x00003a70  4848 4849 494a 4b4b 4c4d 4d4e 4f50 5152  HHHIIJKKLMMNOPQR
0x00003a80  5354 5556 5759 5a5b 5d5e 5f61 6264 6567  STUVWYZ[]^_abdeg
0x00003a90  686a 6c6d 6f71 7374 7678 7a7c 7d7f 8183  hjlmoqstvxz|}...
0x00003aa0  8587 898b 8d8f 9193 9596 9799 9b9d 9fa1  ................
0x00003ab0  a3a5 a7a9 abad afb0 b2b4 b6b8 b9bb bdbf  ................
0x00003ac0  c0c2 c4c5 c7c8 cacb cdce cfd1 d2d3 d5d6  ................
0x00003ad0  d7d8 d9da dbdc ddde dfdf e0e1 e1e2 e3e3  ................
0x00003ae0  e4e4 e4e5 e5e5 e5e5 e5e6 e5e5 e5e5 e5e5  ................
0x00003af0  e4e4 e4e3 e3e2 e1e1 e0df dfde dddc dbda  ................
0x00003b00  d9d8 d7d6 d5d3 d2d1 cfce cdcb cac8 c7c5  ................
0x00003b10  c4c2 c0bf bdbb b9b8 b6b4 b2b0 afad aba9  ................
0x00003b20  a7a5 a3a1 9f9d 9b99 9774 6867 694c 7269  .........thgiLri
0x00003b30  6146 2066 6f20 6e61 6d68 6374 6157 2079  aF fo namhctaW y
0x00003b40  6220 6564 6f43 0001 0301 070f 0c0b 0900  b edoC..........
0x00003b50  0009 0b0c 0f07 0101 0101 0101 0101 0101  ................
0x00003b60  0101 0101 0101 0101 0101 0101 0101 0101  ................
0x00003b70  0101 0101 0101 0101 0101 0101 0101 0101  ................
0x00003b80  0101 0101 0101 0101 0101 0101 0101 0101  ................
0x00003b90  0101 0101 0101 0101 0101 0101 0101 0101  ................
0x00003ba0  0101 0101 0101 0101 0101 01ff ffff ffff  ................
0x00003bb0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
0x00003bc0  ffff ffff ffff ffff ffff ffff ffff ffff  ................
```
*At last, a string!*

...a weird, reverse string appears at `0x3b29`:

> Code by Watchman of FairLight

*FairLight... FairLight... I heard that name somewhere!*

Yes! The challenge was written by [Watchman](http://csdb.dk/scener/?id=1090), a
Commodore 64 scener associated with the famous
[FairLight](http://csdb.dk/group/?id=20) group. Let's try a C64 emulator!

# Running the code
When I fired up `VICE` and hit Alt+A, the emulator wouldn't initially load the
file, but after some trial and error I found that you need to hit `Autostart`
after choosing the file, not `Open` (it probably was some kind of raw executable
format). After loading for a couple seconds, the screen filled with white and
flashing letters forming the text we saw a while ago:

![VICE screen 1]({{ site.url }}/assets/vice_screenshot_1.png)
*So... that's it?*

# And then it hit me
At this point both the challenge name and the hint started to make perfect
sense. The flag text must've been hidden by painting it the same color as the
background, which was too bright!

I believe that the giant from the hint was C64 itself - demoscene of this
insanely popular platform is still very active and doesn't seem to be affected
by time at all.

# Unveiling the flag
So, I read on the VICE emulator debugging options and C64 itself, and found that
the device holds the background color value at memory offset `$d021`. I hit
`Alt+H` to enable VICE's Monitor (a fancy name for a debugger that is) and wrote
a `02` byte to the offset:
```plain
AUTOSTART: `/home/drozdziak1/2bright' recognized as snapshot image.
Main CPU: RESET.
Drive 8: RESET.
AUTOSTART: Done.
AUTOSTART: Restoring snapshot.
Sound: Closing device `pulse'
Sound: Opened device `pulse', speed 44100Hz, fragment size 1,5ms, buffer size
100ms
reSID: MOS8580, filter on, sampling rate 44100Hz - resampling, pass to 19845Hz
Drive 8: RESET (For undump).
(C:$4109) > d021 02
```
*The last line changes the background color to brown.*

The one thing left to do was continue program execution with `return`:

![VICE screen 2]({{ site.url }}/assets/vice_screenshot_2.png)
*The flag is actually meant to be entered without spaces.*

# Conclusion
So there you have it. My first writeup from my first CTF. I had a ton of fun and
adding to the very low solve count was quite a confidence booster, I can't wait
for the CTFs to come! Feel free to speak your mind in the comments.

```nasm
mov     rax, 0
pop     rbp
ret             ; See ya!
```
