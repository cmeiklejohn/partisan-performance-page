+++
date = "2015-12-2T14:10:00+03:00"
draft = false
title = "Motivation"
weight = 1
+++

Partisan is the the design of an alternative runtime system for improved
scalability and reduced latency in actor applications.

Partisan provides:

* **Better scalability** by leveraging different network topologies for communication
* **Reduced latency** through efficient parallel message scheduling for actor-to-actor communication

Partisan is provided as a user library in Erlang and achieves up to an order
of magnitude increase in the number of nodes the system can scale to through
runtime overlay selection, up to a 34.96x increase in throughput, and up to a
13.4x reduction in latency over Distributed Erlang.