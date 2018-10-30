---
title: ARM cpu and memory performance measures
date: 2018-10-30
tags: arm performance benchmark base-hw
---

Recently, I recognized a regression in our (Genode's) microkernel when running
it on top of certain ARM Cortex A9 boards. It came out that due to code
restructuring the quite special off-core L2 cache was not enabled anymore -
except for the Pandaboard. More annoying than the regression itself was the fact
that we did not recognized it in time. Instead it land on ours feet when
actually investigating the belowground performance of our Java port on top of
the Nitrogen6 SoloX board. As it turned out, treating our runtime environment
for the Java port was only half of the puzzle. The CPU and memory performance
were already degraded significantly.

That was the straw that broke the camel's back. So I wrote two easy tests to
measure the CPU speed and memory throughput. Actually, just measuring would not
be sufficient. I first needed a baseline. Therefore, I run both tests on top of
Linux on all of our ARM test-boards. That was the real work: to boot Linux with
a build environment on all that different, mostly older ARM boards. After some
disappointments with Yocto builds and Debian installers, which failed at least
to install the bootloader correctly on an SD-card, I decided to boot all boards
with a self-compiled Linux kernel over network, and use one and the same Debian
based root-filesystem via NFS. That approach worked straight forward.

So here are the results for Arndale, Pandaboard, Rasperry Pi 1, i.MX53
Quickstart board and Wandboard Quad measured running Linux, Genode/HW,
Genode/sel4, and Genode/Fiasco.OC. It is assumed that Linux configures the
device in an optimal fashion regarding the clocking, voltage, and caching
attributes to gain maximum cpu and memory speed. On Linux the test times were
measured on-target. On Genode I've measures timing off-target to circumvent
wrongly configured clocks.

The bogomips test measures bogus instructions analoque to the measurement in the
Linux kernel. It performs 2 billions of increment operations in a register. So it
mainly does not touch memory. Thereby we gain a value consistent with the cpu clock
speed. If we measure lower values here, it is clear that we are not running with
full cpu speed.

All other tests work on a 8MB memory region which gets continously written in a loop
1K times. Thereby I've used the quite simple memcpy and memset implementations of the
Genode base-library, the C-library memcpy and memset, and finally the Genode
base-library memcpy again to read and write uncached memory. The C-library is not
really comparable as Debian uses the highly optimized glibc and on Genode we use a
port of the Freebsd libc without architecture-specific optimization.

## Arndale board

Test / Kernel   | Linux baseline | HW         | Fiasco.OC
----------------|----------------|------------|-----------
bogomips        | 1327 BMIPS     | 1259 BMIPS | 1259 BMIPS
----------------|----------------|------------|-----------
bytewise memcpy | 1097 MB/s      |  901 MB/s  |  900 MB/s
----------------|----------------|------------|-----------
base-lib memcpy | -              | 2194 MB/s  | 2209 MB/s
----------------|----------------|------------|-----------
base-lib memset | -              |  710 MB/s  |  708 MB/s
----------------|----------------|------------|-----------
libc memcpy     | 5821 MB/s      |  998 MB/s  | 1007 MB/s
----------------|----------------|------------|-----------
libc memset     | 5903 MB/s      | 4501 MB/s  | 4518 MB/s
----------------|----------------|------------|-----------
uncached write  | -              | 1886 MB/s  | 1879 MB/s
----------------|----------------|------------|-----------
uncached read   | -              |  249 MB/s  |  230 MB/s

## Panda board

Test / Kernel   | Linux baseline | HW        | Fiasco.OC
----------------|----------------|-----------|----------
bogomips        |  527 BMIPS     | 532 BMIPS | 533 BMIPS
----------------|----------------|-----------|----------
bytewise memcpy |  374 MB/s      |  83 MB/s  |  83 MB/s
----------------|----------------|-----------|----------
base-lib memcpy | -              | 128 MB/s  | 129 MB/s
----------------|----------------|-----------|----------
base-lib memset | -              |  69 MB/s  |  69 MB/s
----------------|----------------|-----------|----------
libc memcpy     | 1702 MB/s      |  73 MB/s  |  73 MB/s
----------------|----------------|-----------|----------
libc memset     | 1768 MB/s      | 276 MB/s  | 276 MB/s
----------------|----------------|-----------|----------
uncached write  | -              | 140 MB/s  | 141 MB/s
----------------|----------------|-----------|----------
uncached read   | -              | 117 MB/s  | 117 MB/s


## Wandboard Quad

Test / Kernel   | Linux baseline | HW        | sel4
----------------|----------------|-----------|----------
bogomips        |  656 BMIPS     | 528 BMIPS | 527 BMIPS
----------------|----------------|-----------|----------
bytewise memcpy |  447 MB/s      | 125 MB/s  | 184 MB/s
----------------|----------------|-----------|----------
base-lib memcpy | -              | 195 MB/s  | 174 MB/s
----------------|----------------|-----------|----------
base-lib memset | -              | 471 MB/s  | 468 MB/s
----------------|----------------|-----------|----------
libc memcpy     | 1899 MB/s      | 154 MB/s  | 125 MB/s
----------------|----------------|-----------|----------
libc memset     | 1998 MB/s      | 662 MB/s  | 477 MB/s
----------------|----------------|-----------|----------
uncached write  | -              | 87 MB/s   | 58 MB/s
----------------|----------------|-----------|----------
uncached read   | -              | 192 MB/s  | 156 MB/s


## i.MX53 Quickstart board

Test / Kernel   | Linux baseline | HW
----------------|----------------|-----------
bogomips        | 497 BMIPS      | 499 BMIPS 
----------------|----------------|-----------
bytewise memcpy | 343 MB/s       | 168 MB/s   
----------------|----------------|-----------
base-lib memcpy | -              | 200 MB/s  
----------------|----------------|-----------
base-lib memset | -              | 302 MB/s  
----------------|----------------|-----------
libc memcpy     | 726 MB/s       | 175 MB/s   
----------------|----------------|-----------
libc memset     | 818 MB/s       | 808 MB/s  
----------------|----------------|-----------
uncached write  | -              | 197 MB/s  
----------------|----------------|-----------
uncached read   | -              |  45 MB/s  


## Raspberry PI

Test / Kernel   | Linux baseline | HW        | Fiasco.OC
----------------|----------------|-----------|----------
bogomips        |  222 BMIPS     | 140 BMIPS | 233 BMIPS
----------------|----------------|-----------|----------
bytewise memcpy |  83 MB/s       | 49 MB/s   | 49 MB/s
----------------|----------------|-----------|----------
base-lib memcpy | -              | 103 MB/s  | 103 MB/s
----------------|----------------|-----------|----------
base-lib memset | -              |  71 MB/s  |  88 MB/s
----------------|----------------|-----------|----------
libc memcpy     | 1257 MB/s      | 81 MB/s   | 81 MB/s
----------------|----------------|-----------|----------
libc memset     | 1309 MB/s      | 284 MB/s  | 355 MB/s
----------------|----------------|-----------|----------
uncached write  | -              | 103 MB/s  | 102 MB/s
----------------|----------------|-----------|----------
uncached read   | -              | 100 MB/s  | 100 MB/s
