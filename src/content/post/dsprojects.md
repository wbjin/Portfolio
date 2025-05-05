---
title: "Distributed System Projects"
description: "Projects I did in class for EECS 491 Intro to Distributed Systems @ UMich"
publishDate: "May 2025"
tags: ["go", "class"]
---

Throughout the course, I implemented three key value servers based on different strategies

## Primary Backup
For this project, I implemented a fault-tolerant key/value store using Go, based on primary/backup replication. The system supports Put, Get, and Append operations and maintains consistency between a primary and backup server using RPC-based communication and periodic heartbeats. I designed a replication protocol that ensures at-most-once semantics and handles network partitions and crash-recovery scenarios. A centralized view service tracks server availability and coordinates role changes. Key takeaways include designing resilient distributed protocols, managing view changes safely, and enforcing synchronization to avoid race conditions.

## Paxos
For this project, I implemented a fault-tolerant distributed key-value store using the Paxos consensus algorithm to ensure strong consistency and high availability without a centralized coordinator. I developed a custom Paxos library in Go that supports multiple concurrent agreement instances, handles unreliable networks, and implements memory cleanup using a Done() mechanism. Building on top of this, I implemented a replicated state machine (PaxosRSM) and a KVPaxos server that applies client operations (Put, Append, Get) in a globally agreed-upon order. The key/value servers guarantee linearizable single-copy semantics, and deduplicate client requests using unique client IDs and request IDs. This project deepened my understanding of distributed consensus, concurrency, and fault tolerance, while reinforcing the value of layered design for simplifying complex systems.

## Sharded Paxos
In this project, I designed and implemented a fault-tolerant, sharded key/value storage system built on top of Paxos consensus groups. Each key/value server group replicated its shard state using Paxos, while a separate Paxos-based shardmaster coordinated shard-to-group assignments. I engineered dynamic reconfiguration logic to support consistent shard migration across groups during joins, leaves, and rebalancing events. To ensure linearizable consistency and at-most-once semantics, I tracked client operations and coordinated configuration changes through replicated logs. Technologies used include Go, RPC, and concurrency-safe state management. This project deepened my understanding of distributed systems, consensus protocols, and sharding mechanisms in modern key/value stores.
