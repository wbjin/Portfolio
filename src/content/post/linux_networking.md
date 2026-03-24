---
title: "Linux Networking"
description: "A dive into the Linux Networking subsystem"
publishDate: "March 2026"
tags: ["presonal"]
---

<a href="https://github.com/wbjin/learningcpp/tree/main/understanding_linux_kernel/networking">
  <img
    src="/github-mark-white.svg"
    alt="GitHub"
    class="w-20 h-20"
    style="filter: brightness(0) invert(1);"
  />
</a>

This is a deep dive into the Linux Networking stack using `ftrace`. It looks at
things like
- `sk_buff`
- `qdisc`
- Hardware Interrupts
- I/O and Local APIC
- Software IRQs
- Infiniband

The GitHub repository contains a guide on how to reproduce the results and the
`ftrace` traces. It also contains some analysis of the specific functions that
are involved in the Linux networking stack that was done using LLMs.

The slides below contains some diagrams. Take a look at [this google
slides](https://docs.google.com/presentation/d/1UfRkB6ZZADTQnQxyeRoXOrRPmSokM2oICsm7lz2cqwg/edit?slide=id.p#slide=id.p)
if you want animations as well.

<object data="/LinuxNetworking.pdf" type="application/pdf" width="900px" height="800px">
  <embed src="/LinuxNetworking.pdf"> <p>This browser does not support PDFs.</p> </embed>
</object>
