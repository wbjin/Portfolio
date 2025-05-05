---
title: "Computer Networking Projects"
description: "Projects I did in class for EECS 489 Computer Networks @ UMich"
publishDate: "December 2024"
tags: ["c++", "class"]
---

## Adaptive Vidoe Streaming
For this project, I built an adaptive video streaming system in C++ that emulates how modern content delivery networks (CDNs) handle video traffic. I implemented miProxy, a multithreaded HTTP proxy that intercepts MPEG-DASH video requests, estimates per-client throughput using an exponentially weighted moving average (EWMA), and dynamically rewrites segment requests to select the optimal bitrate. The proxy supports concurrent client connections using select()-based polling and socket programming, parses raw HTTP headers, and adapts streaming quality based on real-time network conditions. I also implemented a load balancer that returns video server addresses using either round-robin or geographic routing based on Dijkstra's algorithm, parsing a weighted graph topology from a config file. This project deepened my understanding of adaptive bitrate streaming, socket multiplexing, throughput estimation, and the architecture behind large-scale video CDNs.

## Reliable Transport
For this project, I implemented a reliable transport protocol, WTP, over UDP. I developed both a sender (wSender) and receiver (wReceiver) that ensured in-order, loss-resilient data transfer using a sliding window protocol, 32-bit CRC checksums, and custom packet headers with sequence numbers. I extended the system with an optimized version (wSenderOpt, wReceiverOpt) that reduced unnecessary retransmissions by tracking individual packet ACKs and timers. The project involved low-level socket programming, file I/O, and handling packet-level network issues such as loss, duplication, and reordering. This experience deepened my understanding of reliable transport mechanisms and protocol design beyond TCP.

## Static router
For this project, I built a static router in C++ that processes raw Ethernet frames and routes real packets across a Mininet-emulated network. The router uses a static routing table to forward IP packets and handles low-level protocols such as ARP, IP, ICMP, and Ethernet. I implemented longest prefix match logic for routing decisions, ARP caching with expiration and retry logic, and ICMP error reporting for TTL expiry, unreachable hosts, and ping replies. The project involved integrating with the POX controller using Boost.Asio and Protocol Buffers to communicate with the Mininet switch. This hands-on experience deepened my understanding of how real routers handle packet forwarding, address resolution, and protocol-level error signaling.
