+++
date = "2016-04-13T09:20:18+01:00"
draft = true
title = "Nomad"
tags = ["cluster"]

+++

# Basic concepts
A client is a very lightweight process that registers the host machine, performs heartbeating, and runs any tasks that are assigned to it by the servers. The agent must be run on every node that is part of the cluster so that the servers can assign work to those machines.  
