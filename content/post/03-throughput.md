+++
date = "2015-12-2T14:10:00+03:00"
draft = false
title = "Scaling Riak Core"
weight = 3
+++

To look at read-world applicability, we ported the distributed systems
framework, [Riak Core](http://www.github.com/basho/riak_core), to Partisan
and built two example applications: (i) a simple echo service – an
application that’s designed to only be bound by the speed of the actor
receiving messages and the network itself; and (ii) a memory-based key-value
store that operates using read/write quorums – more representative of a
workload where more data is being transmitted and more CPU work has to occur.

Riak Core is a distributed programming framework written in Erlang and based
on the Amazon Dynamo system that influenced the design of the distributed
database Riak, Apache Cassandra, and the distributed actor framework Akka. In
Riak Core, a distributed hash table is used to partition a hash space across
a cluster of nodes. These virtual nodes—the division of the hash space into N
partitions—are claimed by a node in the cluster, and the resulting ownership
is stored in a data structure known as the ring that is periodically gossiped
to all nodes in the cluster. Requests for a given key are routed to a node in
the cluster based on the current partitioning of virtual nodes to cluster
nodes in the ring structure using consistent hashing, which minimizes the
impact of reshuffling when nodes join and leave the cluster. Background
processes are used for cluster maintenance; ownership handoff, (transferring
virtual node ownership) metadata anti-entropy (an internal KVS for
configuration metadata) and ring gossip (information about the cluster’s
virtual node to node mapping).

Our first application is a simple echo service, implemented on a three node
Riak Core cluster. For each request, we generate a binary object, uniformly
select a partition to send the request to, and wait for a reply containing
the original message before issuing the next request. Binary objects are
generated for two payload sizes, 512KB and 8MB. Concurrency is increased
during the test execution and parallelism is configured at 16. We test two
latency configurations: 1ms (representative of single-AZ, single-region) and
20ms (representative of multi-AZ, single region.) We run a fixed duration of
120 seconds.

We first demonstrate that with 128 actors, 1ms RTT, and large payloads (8MB),
Partisan is 3.10x faster than Distributed Erlang. With smaller payloads
(512KB), Partisan is on par with Distributed Erlang (1.01x).

<img src="img/echo-1ms.JPG" alt="Echo Service: 1ms" class="graph" />

We also demonstrate that with 128 actors, 20ms RTT, and larger payloads
(8MB), Partisan is 34.96x faster than Distributed Erlang (which achieves only
5 ops/second before reaching peak throughput). With smaller payloads (512KB),
Partisan is 7.21x faster than Distributed Erlang.

<img src="img/echo-20ms.JPG" alt="Echo Service: 20ms" class="graph" />

Our second application is a memory-based key-value store, similar to the Riak
database, implemented on a three node Riak Core cluster.

Each key is hashed and mapped to a virtual node using the ring structure that
is gossiped in the cluster. The virtual node that the key is hashed to, along
with that virtual nodes’s two clockwise neighbors on the ring, represent the
three virtual nodes that contain the three replicas for the data item. Each
request (either a get operation or put operation) to the keyvalue store uses
a quorum request pattern, where requests are made to these three replicas,
and the response is returned to the user when a majority (2 out of 3)
replicas reply.

This pattern involves multiple nodes in the request path, and each partition
simulates a 1ms storage delay in the request path. We reuse the
aforementioned benchmarking strategy: test execution is fixed at 120 seconds.

For each request, we draw a key from a normal distribution across 10,000 keys
and run the key through Riak Core’s consistent hashing algorithm for
placement. The consistent hashing placement algorithm aims for uniform
partitioning of keys across the cluster. We use a 10:1 read/write ratio for
the experimental workload. Concurrency is varied in our experiments (x-axis)
and parallelism is configured at 16. We test two latency configurations: 1ms,
and 20ms.

We present the results of the 1ms RTT KVS experiment. Under workloads that
are CPU-bound, where the network is not congested nor contended for,
Distributed Erlang can outperform Partisan for small payloads, but
underperforms Partisan for large payloads. Distributed Erlang can overperform
Partisan in the case of small payloads because of (i) the impact of
scheduling overhead for Partisan; and (ii) its placement in the Erlang VM,
which minimizes the cost of a given message send. However, as the size of
both payloads and latency increases, the benefits of Partisan becomes
apparent. In this experiment, with 128 actors, 1ms RTT, and large payloads
(8MB), Partisan is 1.4x faster than Distributed Erlang, while with smaller
payloads (512KB), Partisan on par with Distributed Erlang (0.96x). However,
these dynamics change markedly in less favorable network conditions.

<img src="img/kvs-1ms.JPG" alt="KVS Service: 1ms" class="graph" />

We present the results of the 20ms RTT KVS experiment. Here, we see that
Partisan optimizations allow for better performance in terms of latency and
throughput, whereas Distributed Erlang loses performance as concurrency
increases. In this experimet, with 128 actors, 20ms RTT and smaller payloads
(512KB), Partisan is 1.8x faster than Distributed Erlang. As we increase the
payload size, Distributed Erlang surprises us. Under larger payloads (8MB),
Partisan far exceeds the performance of Distributed Erlang, achieving 95
ops/second; Distributed Erlang can only complete 1 operation during the
entire 120s execution with 128 actors, and only 3 operations during the
entire execution with 64 actors!

<img src="img/kvs-20ms.JPG" alt="KVS Service: 20ms" class="graph" />