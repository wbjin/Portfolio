---
title: "Zeus"
description: "My contributions to Zeus, a library for measuring and optimizing energy consumption
under Deep learning workloads"
publishDate: "January 2025"
tags: ["python", "rust", "work"]
---

<a href="https://github.com/ml-energy/zeus/">
  <img
    src="/github-mark-white.svg"
    alt="GitHub"
    class="w-20 h-20"
    style="filter: brightness(0) invert(1);"
  />
</a>


## Zeus
Zeus is a library for measuring and optimizing energy consumption built by [Jae-Won Chung](https://github.com/jaywonchung). It is primarily written in Python but has components built in Rust and C++.

## Distributed Energy Tracing
Coming soon...

## CPU and DRAM monitoring
I integrated CPU and DRAM monitoring to the library using the RAPL framework that Intel processors exposed. My PR [commit](https://github.com/ml-energy/zeus/commit/5a419c4ece448838b66e7aac4246ecb8a9b925d9) has been merged and is a part of the library currently.

Intel processors expose RAPL metrics through model specific registers. Root user privileges are required to interface with RAPL but because, it is undesirable to run training or inference with elevated privileges, Zeus also provides a Zeus Daemon that contains the bare minimum interfacing needed with the operating system in order to provide energy measurements. Zeus Daemon is run with elevated privileges while the python library interfaces with Zeusd to get measurement values.

Zeusd, architected by Jae-Won, is built for low latency and maximum concurrency.
![image](/zeusdarch.png)
Each request is handled by it's own request handler that interfaces with separate asynchronous tokio tasks that are in charge of measuring each CPU socket. This architecture allows for many concurrent requests to be served.

The biggest trouble with energy measurement through RAPL is integer overflow. RAPL counters are unsigned 32 bit integer microjoule energy measurements. Therefore, the counter overflows fairly often. In order to handle this integer overflow, each async task in charge of measuring a CPU socket spawns off two more tasks that monitor integer overflow. This task counts the number of overflows which can then be used to caclulate the correct energy measurement.

For more details, check out the presentation for this project.
<object data="/zeusdpres.pdf" type="application/pdf" width="700px" height="800px">
    <embed src="/zeusdpres.pdf">
        <p>This browser does not support PDFs.</p>
    </embed>
</object>
