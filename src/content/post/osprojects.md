---
title: "Operating System Projects"
description: "Projects I did in class for EECS 482 Intro to Operating Systems @ UMich"
publishDate: "May 2024"
tags: ["c++", "class"]
---

## Thread library

For this project, I implemented a user-level thread library in C++17 on Linux, building core primitives such as threads, mutexes, and condition variables from scratch using ucontext.h. I managed context switching via getcontext, makecontext, setcontext, and swapcontext, and ensured FIFO scheduling across all queues. My implementation handled both synchronous and asynchronous timer interrupts, supported multiple CPUs, and used std::atomic to coordinate inter-processor interrupts and maintain atomicity with a cpu guard. I rigorously tested the library with custom test cases, explored low-level concurrency control, and enforced robustness through careful memory management, exception handling, and assertions. This project deepened my understanding of OS-level thread scheduling, context switching, and synchronization in multicore environments.

## External Pager
For this project, I implemented a user-space virtual memory manager in C++ that simulates an external pager, handling page faults, virtual-to-physical memory translation, and disk swapping. I built core components such as vm_extend, vm_fault, vm_create, and vm_destroy, and managed both private and shared memory pages across processes. I implemented FIFO with second-chance (clock) page replacement to handle physical memory eviction, and maintained per-process page tables and a global free list for memory and disk blocks. I optimized for deferred zero-fills and avoided unnecessary disk I/O by tracking reference and dirty bits in software. This project gave me a deep understanding of demand paging, arena-based memory allocation, and page replacement algorithms at the OS level.

## Network File Server
For this project, I built a multi-threaded network file server in C++11 that handles remote client requests using a custom file system over TCP sockets. The server supports operations like file creation, reading, appending, and deletion, and enforces session-based authentication using sequence numbers to prevent replay attacks. I implemented fine-grained synchronization using reader-writer locks and mutexes to enable safe concurrent access across clients. The file system is hierarchical and manages metadata using inodes and directory entries stored on a simulated disk. I also designed a robust disk I/O strategy that maintains on-disk consistency even under crashes by ordering writes carefully. This project deepened my understanding of socket programming, concurrency control, and file system design.


