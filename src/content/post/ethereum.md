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
using `libp2p`. The main theme of my work was improving observability of the
network layer of Ethereum. This came with three major workflows:

## 1) Simulation and Benchmarking

### Motivation
The networking layer of Ethereum, `libp2p` have various implementations of in
different languages by different groups. Testing within a single implementation
can be done relatively throughly but the challenge is **testing between different
implementations**. There is also the challenge of Ethereum nodes not being managed
by a central entity. Therefore, it is hard to test certain behaviors and
scenarios that may occur simply because you don't control all the nodes.

### Solution
The foundation has a crawler that crawls nodes on mainnet and exports data about
a node such as which client/implementation it is running, the IP address, geo
location, autnomous system number, etc. Using this list of nodes in the network
and other data sources such as latencies between autnomous systems/geo locations
and bandwidth limitations of a node, it is possible to replicate the network
graph of mainnet **locally**. This is powerful because you can create certain
scenarios that you wouldn't be able to on the actual network and test the
interoperability of differnet implementations of `libp2p` throughly. Luckily,
there is already a tool for simulating called `shadow`. `shadow`, from its
documentation, "is a discrete-event network simulator that directly executes real
application code, enabling you to simulate distributed systems with thousands of
network-connected processes in realistic and scalable private network
experiments using your laptop, desktop, or server running Linux."

### My work
- Source up to date and reliable inter-autnomous system latency measurements.
  RIPE Atlas has streaming API that continously streams ping measurements between
  probes across the world. I built a program that subcribes to this stream and
  exports the data to a data store. https://github.com/wbjin/PingStreamer
- Create a network graph using the list of source nodes and latency measurements
  that can be fed into `shadow`. Because there are limitations on how many
  processes you can realistically run for a `shadow` simulation, I had to come up
  with a way to as closely replicate mainnet's network graph without blowing a
  computer up

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

## 3) Expanding telemetry collection of the networking layer

### Motivation
All of the above is powered by robust telemetry data. While the current
telemetry pipeline has a lot insightful data, it doesn't capture all of the
available metrics from `libp2p` implementations. Getting even more telemetry
will allow more robust and a more complete view of the network.


### Solution
There are three different ways telemetry data is collected: crawlers, nodes that
sit in the network and act as a regular node but also export telemetry data,
full clients that export telemetry data.

### My work
- Create new gRPC service for collecting node resource usage such. Key metrics
  include CPU usage, memory usage, jitter.
- Expanding existing gRPC services and writing new kafka translations to collect more `libp2p` metrics.
