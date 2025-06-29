---
title: "SMBC Nikko Securities"
description: "My project during my internship at SMBC"
publishDate: "June 2025"
tags: ["c", "work"]
---

## [SMBC Nikko Securities](https://www.smbcnikko.co.jp/en/)

Nikko is a broker dealer firm that trades in the Japan equities markets. The Equity
department at Nikko operates an electronic trading platform called SNET that processes millions of
trades from institutional and retail clients.


## My work
I was on the infrastructure team supporting SNET working on network infrastructure. I built a packet
tracing tool that measured latency at the application and the network interface card (NIC) level to
profile latency at various points in the packet send and receive life cycle. The tool was used to
understand the kernel network stack but to also understand and profile a different approach in
kernel bypass. The tool is capable of measuring latency between the application and the NIC on
transmit (transmit path), the NIC and the application on receive (receive path), RTT, oneway
latency, and packet throughput. The tool relies heavily on various other technologies already
impelmented in the network infrastructure or is provided by hardware. A lot of what I did was
dealing with hardware such as clocks and interface cards. Below is a rough timeline of what I did
through the internship.

### Intel Processor Trace
A key part of my tool is being able to reliably tell exactly what is going on when you make various
function calls. For example, what is happening when you call `send` or `recv`. What is hapenning
when you call `clock_gettime`, etc... This was important for a latency tracing tool because being
able to reliably get the current time is the crux of latency measrument. Getting inaccurate time or
taking too long to get time makes your measurements unreliable. In order to understand what is
happening in these C library functions, I used tools such as `strace` and `perf trace` that allows
you to inspect the system calls that are being made in your program. While this was a good way to
profile how long `send` and `recv` took, it wasn't a perfect solution. In the case of
`clock_gettime`, I was not able to inspect the underlying system call, the reason being `vdso`.

#### vdso
From the man pages: "Virtual Dynamic Share Object (vdso) is a small shared library that the kernel
automatically maps into the address space of all user-space applications." This is a performance
optimization that the linux kernel uses to avoid making system calls to access kernel functions. It
removes the need for a context switch and applications can directly access some kernel functions, in
this case `clock_gettime`. vdso does make performance faster but at the cost of observability. Now
you can no longer understand what is happening because you cannot inspect system calls.

### Back to Intel PT
Now, because we cannot see the system calls that are made by the C library function, I could no
longer apply the tools such as `strace` and `perf trace` to understand what was happening. Luckily,
I was working on a processor that supported the relatively new tool **Intel Processor Trace**. This
tool was probably one of the coolest things that I used during the internship. It is a tool that
lets you trace the assembly instructinos executed by a program at the processor level. Along with
`xed` (x86 Encoder Decoder), it lets you see the exact instruction and symbol that the instruction
was executed for. It also provides timestamps generated with CYC packets (don't really understand
this but it essentially uses cpu cycles to timestamp at the hardware level). Now this is cool
because you can see exactly what your program is doing at the lowest level. All the information you
need to debug is provided. Using this tool, you can see the exact instructions you execute and how
long each instruction takes which also means you can see how long each function call takes. Now, I
was able to see exactly how long a `clock_gettime` took.

### Clocks on a machine
Again, measuring how long it took to take time was super important. Reading CLOCK_REALTIME or
CLOCK_MONOTONIC was typically very fast (nanosecond range). However, the problem arose when I was
trying to read time from devices other than the Real Time Clock. Why was I even trying to do this?
Because I was trying to make sense of hardware timestamps

### Hardware timestamps
Hardware timestamps are capabilities on some interface cards. It is essentially timestamps on
packets just before transmitting on the wire and just after receiving it from the wire. It uses
clocks on the interface cards themselves called PHC. You can extract these timestamps by enabling
them on your interface card, setting the correct socket options for the timestamps, using `struct
msghdr`, `recvmsg`, and parsing the msghdr struct. You can extract the hardware timestamps like this
but in order get latency measurements, you need to have a point of comparison. I was initially using
CLOCK_REALTIME as this point of comparison but this caused latency measurements to be in the
negatives or way too high. Why? Because CLOCK_REALTIME comes from the RTC and the hardware
timestamps come from the PHC and unfortunately, these clocks were drifted too far apart. Therefore,
I tried measuring time from the same clock as the NIC, the PHC. This way, the timestamps would be
from the same time source and would be comparable. `clock_gettime` also allows you to get time from
a clock device by using its file handler. Using this method, the latency measurements were no longer
in the negatives. However, the problem with this was that it took too long to retrieve time from the
PHC  (around 100 microseconds). Therefore, this was not a viable method. This is where PTP comes in.

### PTP
The P in PHC stands for Percision Time Protocol. The clock on the NIC is a PTP Hardware Clock and is
being synced with a time master in the datacenter. Because the network is incredibly fast and
because thanks to this incredible complex protocol that I don't understand, we are able to get the
PHC in sync the real time at nanosecond granularity. Additionally, the RTC (source of
CLOCK_REALTIME) can be synced with the PHC very fast because it is on the same machine. Using this
protocol, we can now compare CLOCK_REALTIME with the hardware timestamps without having to worry
about clock drift. Using the same method, clocks across different machines can be really in sync as
well. This way, we can now get rtt, oneway, and receive and transmit latencies.

### Kernel Bypass
This is where kernel bypass was incorporated. Kernel bypass, as it sounds like, is a way of
bypassing the kernel when interacting with the NIC. With the traditional network stack, the
application asks the kernel to send and receive packets to the interface card. This makes networking
much easier for the application but comes at the cost of latency. In order for the kernel to run,
there needs to be a context switch to it. Additionally, the kernel is subject to jitter, the
deviation from intended time a task was supposed to run and when it actually ran. This can cause a
lot of deviation in latency between the app and the NIC within a machine. Kernel bypass can bypass
this by interfacing with the NIC directly. Using the tool that I built so far, we can compare the
RTT and oneway latencies of the kernel network stack and kernel bypass. We can also see where the
latency improvement is coming from by looking at how long it takes for a packet to go back and forth
from the app to the NIC.

### Other things I did
- Build a machine from scratch: I built a machine from scratch by accessing an ILO. Configured the
operating system, interface cards, and disk storage. Managed packages with yum and rpm
- Build and ship a rpm: Built an produced an rpm package that can be installed on RHEL 8.10 and RHEL
  9.5 systems
- Measure jitter on a system using sysjitter
- Use multicast


## Leassons learned

- Premature optimization: While I learned a lot and was able to produce a usable product, I couldn't
  achieve all that I set out to. This was mostly because I was focused on trying to things perfectly
  the first time. I was trying to solve a problem when I myself didn't truly understand the problem.
  I wasted a lot of time adding features and functionality that wasn't aligned to the final goal
- Asking the right questions: Throughout the internship, I learned that the hard part wasn't
 answering the question but rather, learning what the right questions to ask were.
- Writing beautiful git commits (50/72) rules
