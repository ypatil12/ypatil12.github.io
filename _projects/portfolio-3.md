---
title: "Distributed Systems Class Papers"
collection: projects
---

System designs on a contract tracing system and an asynchronous KV store. 

# A Delay & Disruption Tolerant Key-Value Store
This [paper](https://ypatil12.github.io/files/AsyncKeyValue.pdf) was an exercise in creating a key-value store with BASE transaction capability that optimized optimized for availability in the presence of network partitions and intermittent network connectivity. Naturally, I proposed building such a system on top of [Bayou](https://people.eecs.berkeley.edu/~brewer/cs262b/update-conflicts.pdf), an eventually consistent data store, and borrowed heavily from elements in Paxos. The writeup modifies the entropy-exchange protocol in Bayou to allow for transaction-level guarantees through compromising on rolling-back many transactions when replicas are disconnected from the primary. 

# TravelLog: A Scalable Service for Contact Tracing 
A brief [system design overview](https://ypatil12.github.io/files/TravelLog.pdf) of a service that can contract trace individuals during a pandemic (doesn't discuss obvious privacy concerns) The paper discusses the cloud sevices, infrastructure, data processing, and scalibility concerns of maintaining such a system, including updating a user's location, finding paths of users that opt-in to location tracking in a municipality, and flagging close contacts.

