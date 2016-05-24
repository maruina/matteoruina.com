+++
date = "2016-04-21T09:20:18+01:00"
draft = true
title = "RabbitMQ"
tags = ["queues", "rabbimq"]

+++
# Highly Available
## Highly Available Queues
Queues can optionally be made mirrored across multiple nodes. Each mirrored queue consists of one master and one or more slaves, with the oldest slave being promoted to the new master if the old master disappears for any reason.

Messages published to the queue are replicated to all slaves. Consumers are connected to the master regardless of which node they connect to, with slaves dropping messages that have been acknowledged at the master.  

Queue mirroring therefore enhances availability, but does not distribute load across nodes (all participating nodes each do all the work).
## Clustering
For two nodes to be able to communicate they must have the same shared secret called the Erlang cookie. The cookie is just a string of alphanumeric characters. It can be as long or short as you like. Every cluster node must have the same cookie.  

RabbitMQ clustering has several modes of dealing with network partitions, primarily consistency oriented. Clustering is meant to be used across LAN. It is not recommended to run clusters that span WAN.  

Every queue in RabbitMQ has a home node. That node is called queue master. All queue operations go through the master first and then are replicated to mirrors. This is necessary to guarantee FIFO ordering of messages.

Queue masters can be distributed between nodes using several strategies.

A node may join a cluster at any time. Depending on the configuration of a queue, when a node joins a cluster, queues may add a slave on the new node. At this point, the new slave will be empty: it will not contain any existing contents of the queue. Such a slave will receive new messages published to the queue, and thus over time will accurately represent the tail of the mirrored queue. As messages are drained from the mirrored queue, the size of the head of the queue for which the new slave is missing messages, will shrink until eventually the slave's contents precisely match the master's contents. At this point, the slave can be considered fully synchronised, but it is important to note that this has occurred because of actions of clients in terms of draining the pre-existing head of the queue.  

However, there is currently no way for a slave to know whether or not its queue contents have diverged from the master to which it is rejoining (this could happen during a network partition, for example). As such, when a slave rejoins a mirrored queue, it throws away any durable local contents it already has and starts empty. Its behaviour is at this point the same as if it were a new node joining the cluster.  

## Network partitions
In pause-minority mode RabbitMQ will automatically pause cluster nodes which determine themselves to be in a minority (i.e. fewer or equal than half the total number of nodes) after seeing other nodes go down. It therefore chooses partition tolerance over availability from the CAP theorem. This ensures that in the event of a network partition, at most the nodes in a single partition will continue to run. The minority nodes will pause as soon as a partition starts, and will start again when the partition ends.  

Choose `pause_minority` when you have clustered across 3 AZs in EC2, and you assume that only one AZ will fail at once. In that scenario you want the remaining two AZs to continue working and the nodes from the failed AZ to rejoin automatically and without fuss when the AZ comes back.

The Erlang VM on the paused nodes will continue running but the nodes will not listen on any ports or do any other work. They will check once per second to see if the rest of the cluster has reappeared, and start up again if it has.

Note that nodes will not enter the paused state at startup, even if they are in a minority then. It is expected that any such minority at startup is due to the rest of the cluster not having been started yet.

Also note that RabbitMQ will pause nodes which are not in a strict majority of the cluster - i.e. containing more than half of all nodes. It is therefore not a good idea to enable `pause-minority` mode on a cluster of two nodes since in the event of any network partition or node failure, both nodes will pause. However, `pause_minority` mode is likely to be safer than `ignore` mode for clusters of more than two nodes, especially if the most likely form of network partition is that a single minority of nodes drops off the network.

Finally, note that pause_minority mode will do nothing to defend against partitions caused by cluster nodes being suspended. This is because the suspended node will never see the rest of the cluster vanish, so will have no trigger to disconnect itself from the cluster.
