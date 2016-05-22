+++
date = "2016-04-13T09:20:18+01:00"
draft = false
title = "Nomad notebook"
tags = ["cluster"]

+++

# Glossary
 - **Job**: a Job is a specification provided by users that declares a workload for Nomad. A Job is a form of desired state; the user is expressing that the job should be running, but not where it should be run.
 - **Task Group**: a Task Group is a set of tasks that must be run together. For example, a web server may require that a log shipping co-process is always running as well. A task group is the unit of scheduling, meaning the entire group must run on the same client node and cannot be split.
 - **Task**: a Task is the smallest unit of work in Nomad. Tasks are executed by drivers, which allow Nomad to be flexible in the types of tasks it supports. Tasks specify their driver, configuration for the driver, constraints, and resources required.
 - **Allocation**: an Allocation is a mapping between a task group in a job and a client node. A single job may have hundreds or thousands of task groups, meaning an equivalent number of allocations must exist to map the work to client machines.
 - **Client**: a Client of Nomad is a machine that tasks can be run on. All clients run the Nomad agent. The agent is responsible for registering with the servers, watching for any work to be assigned and executing tasks. The Nomad agent is a long lived process which interfaces with the servers.
 - **Server**: Nomad servers are the brains of the cluster. There is a cluster of servers per region and they manage all jobs and clients, run evaluations, and create task allocations. The servers replicate data between each other and perform leader election to ensure high availability. Servers federate across regions to make Nomad globally aware.

# Basic concepts
A datacenter represents something like an AWS availability zone, a physical datacenter location, or some other fault domain or boundary.  
Regions may contain multiple datacenters. Servers are assigned to regions and manage all state for the region and make scheduling decisions within that region. Requests that are made between regions are forwarded to the appropriate servers. As an example, you may have a US region with the us-east-1 and us-west-1 datacenters, connected to the EU region with the eu-fr-1 and eu-uk-1 datacenters.  

A client is a very lightweight process that registers the host machine, performs heartbeating, and runs any tasks that are assigned to it by the servers. The agent must be run on every node that is part of the cluster so that the servers can assign work to those machines.  

![Single Region](https://www.nomadproject.io/assets/images/nomad-architecture-region-a5b20915.png)

Within each region, we have both clients and servers. Servers are responsible for accepting jobs from users, managing clients, and computing task placements. Each region may have clients from multiple datacenters, allowing a small number of servers to handle very large clusters.  

Regions are fully independent from each other, and do not share jobs, clients, or state. They are loosely-coupled using a gossip protocol, which allows users to submit jobs to any region or query the state of any region transparently. Requests are forwarded to the appropriate server to be processed and the results returned.  

The servers in each datacenter are all part of a single consensus group. This means that they work together to elect a single leader which has extra duties. The leader is responsible for processing all queries and transactions. Nomad is optimistically concurrent, meaning all servers participate in making scheduling decisions in parallel. The leader provides the additional coordination necessary to do this safely and to ensure clients are not oversubscribed.  

Clients are configured to communicate with their regional servers and communicate using remote procedure calls (RPC) to register themselves, send heartbeats for liveness, wait for new allocations, and update the status of allocations. A client registers with the servers to provide the resources available, attributes, and installed drivers. Servers use this information for scheduling decisions and create allocations to assign work to clients.  

Users make use of the Nomad CLI or API to submit jobs to the servers. A job represents a desired state and provides the set of tasks that should be run. The servers are responsible for scheduling the tasks, which is done by finding an optimal placement for each task such that resource utilization is maximized while satisfying all constraints specified by the job. Resource utilization is maximized by bin packing, in which the scheduling tries to make use of all the resources of a machine without exhausting any dimension. Job constraints can be used to ensure an application is running in an appropriate environment. Constraints can be technical requirements based on hardware features such as architecture and availability of GPUs, or software features like operating system and kernel version, or they can be business constraints like ensuring PCI compliant workloads run on appropriate servers.

## Jobs and Scheduling
There are four primary "nouns" in Nomad; jobs, nodes, allocations, and evaluations. Jobs are submitted by users and represent a desired state. A job is a declarative description of tasks to run which are bounded by constraints and require resources. Tasks can be scheduled on nodes in the cluster running the Nomad client. The mapping of tasks in a job to clients is done using allocations. An allocation is used to declare that a set of tasks in a job should be run on a particular node. Scheduling is the process of determining the appropriate allocations and is done as part of an evaluation.  

![Evaluation Flow](https://www.nomadproject.io/assets/images/nomad-evaluation-flow-7629d361.png)

The lifecycle of an evaluation beings with an event causing the evaluation to be created. Evaluations are created in the pending state and are enqueued into the evaluation broker. There is a single evaluation broker which runs on the leader server. The evaluation broker is used to manage the queue of pending evaluations, provide priority ordering, and ensure at least once delivery.  

Nomad servers run scheduling workers, defaulting to one per CPU core, which are used to process evaluations. The workers dequeue evaluations from the broker, and then invoke the appropriate scheduler as specified by the job. Nomad ships with a service scheduler that optimizes for long-lived services, a batch scheduler that is used for fast placement of batch jobs, a system scheduler that is used to run jobs on every node, and a core scheduler which is used for internal maintenance. Nomad can be extended to support custom schedulers as well.  

When planning is complete, the scheduler submits the plan to the leader which adds the plan to the plan queue. The plan queue manages pending plans, provides priority ordering, and allows Nomad to handle concurrency races. Multiple schedulers are running in parallel without locking or reservations, making Nomad optimistically concurrent. As a result, schedulers might overlap work on the same node and cause resource over-subscription. The plan queue allows the leader node to protect against this and do partial or complete rejections of a plan.  

As the leader processes plans, it creates allocations when there is no conflict and otherwise informs the scheduler of a failure in the plan result. The plan result provides feedback to the scheduler, allowing it to terminate or explore alternate plans if the previous plan was partially or completely rejected.  

Once the scheduler has finished processing an evaluation, it updates the status of the evaluation and acknowledges delivery with the evaluation broker. This completes the lifecycle of an evaluation. Allocations that were created, modified or deleted as a result will be picked up by client nodes and will begin execution.  

# Job
## Service Discovery
Nomad integrates with Consul for service discovery. A service block represents a routable and discoverable service on the network.  

Nomad schedules workloads of various types across a cluster of generic hosts. Because of this, placement is not known in advance and you will need to use service discovery to connect tasks to other services deployed across your cluster. Nomad integrates with Consul to provide service discovery and monitoring.

There is no need of a service discovery if the job doesn't need to receive any incoming connection.

## Networking
This only applies to services that want to listen on a port. Batch jobs or services that only make outbound connections do not need to allocate ports, since they will use any available interface to make an outbound connection.

# Snippets
**Read stdout and stderr**
```bash
nomad fs cat "${ALLOC_ID}" alloc/logs/"${TASK_NAME}".stderr.0
```

**Manage jobs**
```bash
# Start a job
nomad run -verbose test_nomad
nomad status "${JOB_NAME}"
# Pick an ID in the Allocations output, match it with the Node ID
# On the Node
```



# Links
- [https://www.nomadproject.io](https://www.nomadproject.io)
- [https://github.com/hashicorp/nomad/issues/304](https://github.com/hashicorp/nomad/issues/304)
