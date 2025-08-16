---
title: "Ethereum Foundation"
description: "My project during my internship at Ethereum Foundation"
publishDate: "August 2025"
tags: ["go", "rust", "work"]
---

## [Ethereum Foundation](https://ethereum.foundation/)

The Ethereum Foundation (EF) is an organization that supports and works towards
the long‑term success of the Ethereum network . It maintains key components of
the network and also funds research, infrastructure, education, tooling, and
community programs to advance Ethereum’s scalability, security, and adoption.

## My work
I was on the Peer to Peer (P2P) Networking Team. P2P networking is the
networking abstraction that supports communication between nodes in the network
using `libp2p`. The main theme of my work was simulation and benchmark of the
network.

## 1) Simulation and Benchmarking

### Motivation
The networking layer of Ethereum, `libp2p` have various implementations of in
different languages by different groups. Testing within a single implementation
can be done relatively throughly but the challenge is **testing between different
implementations**. There is also the challenge of Ethereum nodes not being managed
by a central entity. Therefore, it is hard to test certain behaviors and
scenarios that may occur simply because you don't control all the nodes.

### Solution
In order to test across different implementations, we can define specific
scenarios, simulate the interactions between the implementations, and check
that the expected behavior happened. This means that we need two pieces to
simulate effectively: a tool for simulating the network interactions and a way
to understand the state of the Ethereum network. I worked on both the
simulation tool and created a tool for understanding the network.

### Shadow
Shadow was the tool used for simulating network interactions. It is a discrete
event simulator meaning it is based on events such as a message being sent or a
system call being made. Shadow simulates network interactions by intercepting
the system calls made by application programs using techniques like
`LD_PRELOAD` and `seccomp`. What I did was to implement various system calls
and socket options that were not present in Shadow to allow for QUIC support.
These system calls were `recvmmsg` and `sendmmsg` which allow programs to
receive and send multiple UDP datagrams in one system call. This was
interesting for me because I got to look at the implementation of these
syscalls in the Linux kernel and understand how different flags and
functionalities work.

### Kazami
In order to understand the state of the Ethereum network, I built a tool called
`kazami`. `kazami` does three things: 1) Collect latency data 2) Collect
Ethereum node data 3) Generate network graph and characteristics to feed into
Shadow.

- Latency Data: Subscribe to RIPE Atlas's streaming API and continuously ingest
results of `ping` experiments of probes all across the world. Collect latency
data between cities, countries, and continents.
- Node Data: Use `nebula`, a crawler built by Probelab. Take parts that are
needed just for crawling Ethereum consensus layer with discv5 and collect
information such as IP address and geo location of Ethereum nodes.
- Mapping: Cross list these two data sources to generate network graphs and
network characterisitcs like latency to use for a shadow simulation.

## 2) Telemetry and Analytics of libp2p

### Motivation
The Ethereum Foundation and many of the clients already had a robust
observability framework that exported telemetry data from client nodes to a
central data store. However, there wasn't as much analytics done on the
telemetry data. Basically, we weren't really looking into the data.

### Solution
Make cool graphs

### My work
Analyze the robust telemetry data to identify any patterns in the network.
Things we looked for were
- Any irregularities
- Average and tail latencies
- Network graph of mainnet
- Health of the network

Here are some visualizations
<img src="/ef_graph_geo.png" width="700">
<img src="/ef_graph_duplicates.png" width="700">
