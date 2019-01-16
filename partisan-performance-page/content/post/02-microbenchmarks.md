+++
date = "2015-12-2T14:10:00+03:00"
draft = false
title = "Microbenchmarks"
weight = 2
+++

Each of the microbenchmarks run multiple configurations of Partisan under
both increasing latency and payload size, with a fixed number of 10,000
messages per experiment. At the start of each experiment for our
microbenchmarks, N actors are spawned on each of two instances of the Erlang
VM, based on the desired concurrency level. Each actor will send a single
message to an actor on the other node and wait for acknowledgement before
proceeding. Experiments were run using the full-mesh overlay, but the
optimizations are implemented for all overlays. Latency is reported as the
time to send a single message from the source to the destination.

We start by showing a baseline configuration of Distributed Erlang compared
with Partisan with all optimization enabled. Our results show that leveraging
additional connections and affinitizing communication increases performance
regardless of concurrency. With 128 actors, 512KB payload, and 1ms RTT,
Partisan with affinitized parallelism performs 1.9x better than Distributed
Erlang.

<img src="img/microbenchmarks.JPG" alt="Microbenchmarks" class="graph" />

What happens if we don’t have such favorable network conditions? What if our
application is spread out between two AWS availability zones and suffers from
RTTs closer to 20ms instead? We show the effects of running our earlier
experiment this time with a 20ms RTT latency between actors located on
different nodes. As we can see, as the latency increases, the system can take
advantage of more communications channels to parallelize inter-actor
communication on the network. With 128 actors, 512KB payload, and 20ms RTT,
Partisan with affinitized parallelism performs 13.4x better than Distributed
Erlang.

<img src="img/microbenchmarks-highlatency.JPG" alt="Microbenchmarks: High Latency" class="graph" />

In the Erlang community, large message sizes are not uncommon. Consider
[Riak](http://www.github.com/basho/riak), the distributed key-value store
which could contain user-stored and arbitrary-sized data. An Erlang message
then could contain a user-provided piece of data megabytes in size. However,
it’s well-known in the Erlang community that Distributed Erlang doesn’t
handle large message sizes well. In fact, the Riak documentation suggests to
avoid storing objects larger than 1-2MB due to the performance degradation
that occurs due to Distributed Erlang. Cognizant of this, we turn our
attention to question of how large payload size affects performance in
Partisan. Can Partisan overcome some of the performance issues faced by
Distributed Erlang in the face of large payloads?

We explore the effects of increasing payload size on Partisan as compared to
Distributed Erlang. Keeping in line with the community-observed limits of
Distributed Erlang, we vary the message size from 512kb (below the 1MB
performance degradation threshold) to 8MB (far above the 1MB performance
degradation threshold). With 128 actors, 8MB payload, and 1ms RTT, Partisan
with affinitized parallelism performs 3.7x better than Distributed Erlang!

<img src="img/microbenchmarks-largepayload.JPG" alt="Microbenchmarks: Large Payload" class="graph" />