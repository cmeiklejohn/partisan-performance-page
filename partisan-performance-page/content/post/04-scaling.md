+++
date = "2015-12-2T14:10:00+03:00"
draft = false
title = "Cluster Scalability: Lasp"
weight = 4
+++

Lasp is a programming framework designed for large scale coordination-free
programming. Applications in Lasp are written using shared state; this shared
state is stored in an underlying key-value store and is replicated between
all nodes. Applications modify their own replica and propagate the effects of
their changes to their peers. Lasp ensures that applications converge to the
same result on every node through the use of data structures known as
Conflict-Free Replicated Data Types, combined with monotone programming. For
our Lasp evaluation, the application is a simulated advertisement counter,
modeled after the Rovio advertisement counter scenario for Angry Birds. In
this application, each client keeps a replica of a grow-only distributed
counterissuing increment operations when an advertisement is displayed. Once
a certain number of impressions is reached, the advertisement is disabled.
The increment interval was fixed at 10s, and the propagation interval for the
counter was fixed at 5s. The total number of impressions was configured to
ensure that the experiment would run for 30 minutes under all configurations.
The evaluation is performed on both the client-server and peer-to-peer
overlays for different cluster sizes, ranging from 32 all the way up to 1,024
node clusters.

For both overlays, the system propagates the full state of the counter to the
node’s peers at each propagation interval. Note that since the Rovio
advertisement counter scenario was designed for mobile applications, we do
not run the fullmesh topology because it would be unrealistic. That is, in
the context of mobile apps, clients would not connect to all other nodes, nor
will they have knowledge of who all of the clients in the system are. Rather,
either mobile apps will communicate with some number of nearby peers
(peer-to-peer) or they will communicate through a server (client-server).
Clientserver also serves as the standard model of deploying mobile
applications today. Thus, we designed our experiments to reflect this—we
examine client-server and peer-to-peer overlays for this application in our
experiments.

We present the total data transmission required for the experiment to finish
as we scale the size of the cluster from 32 to 1024 nodes. For smaller
clusters of nodes, client-server is the more efficient overlay in terms of
the amount of data that must be transmitted to finish the experiment. This
improved efficiency comes at a cost, however, as the client-server
configuration is unable to scale beyond 256 nodes. Beyond 256 nodes, the
cilent-server experiments could not be completed; as the Erlang VM has
unbounded queues, when the server cannot process incoming messages quickly
enough, the VM allocates all available memory, leading to termination of the
Erlang VM by the Linux OOM Killer. Peer-to-peer is more resilient in the face
of a node failure allowing it to support larger clusters of nodes—up to 1024!
However, peer-to-peer is less efficient due to this—the redundancy of
communication links used by the overlay causes it to transmit more data in
order to complete the experiments.

<img src="img/scale.JPG" alt="Cluster Scalability" class="graph" />

Perhaps the most interesting takeaway from the results of this real-world
large-scale experiment is that the experiment was even possible at all with
Erlang. As Distributed Erlang permits one to only use a full-mesh overlay,
it’s possible that the previous results observed by Ericsson on the maximum
size of Erlang clusters–only 200 nodes–are due to this full-mesh-only
restriction. This experiment suggests that Partisan may enable the
development of new applications with actors systems that have not been
previously possible by enabling the application developer to dynamically
change the pattern of communication between nodes, without altering
application semantics. Perhaps the lack of mobile applications written using
distributed actor systems is a symptom of the full-mesh-only restriction.
Similarly, this observation may also hold true for the emerging class of
“Internet of Things” applications.