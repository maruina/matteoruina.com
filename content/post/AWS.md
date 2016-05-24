+++
date = "2016-04-21T09:20:18+01:00"
draft = true
title = "AWS notebook"
tags = ["aws"]

+++
# EC2
## T2 instances
**A CPU Credit provides the performance of a full CPU core for one minute.** T2 instances provide a baseline level of CPU performance with the ability to burst above that baseline level. The baseline performance and ability to burst are governed by CPU credits.  

Other combinations of vCPUs, utilization, and time are also equal to one CPU credit; for example, one vCPU running at 50% utilization for two minutes or two vCPUs running at 25% utilization for two minutes.  

Each T2 instance starts with a healthy initial CPU credit balance and then continuously (at a millisecond-level resolution) receives a set rate of CPU credits per hour, depending on instance size.  

Initial CPU credits do not expire, but they are used first when an instance uses CPU credits. Unused earned credits from a given 5 minute interval expire 24 hours after they are earned, and any expired credits are removed from the CPU credit balance at that time, before any newly earned credits are added. Additionally, the CPU credit balance for an instance does not persist between instance stops and starts; stopping an instance causes it to lose its credit balance entirely, but when it restarts it will receive its initial credit balance again.  

If your instance uses all of its CPU credit balance, performance remains at the baseline performance level. If your instance is running low on credits, your instance’s CPU credit consumption (and therefore CPU performance) is gradually lowered to the base performance level over a 15-minute interval, so you will not experience a sharp performance drop-off when your CPU credits are depleted.
# Security
- Create groups instead of applying permissions to multiple users
- CIS benchmark
- Configure password rotation policy
- Use cloudfront as API endpoint to hide your real API and mask the real endpoints. Also useful for DDOS protection

# Links
* [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html)
