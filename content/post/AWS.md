+++
date = "2016-04-21T09:20:18+01:00"
draft = true
title = "AWS notebook"
tags = ["aws"]

+++
## EC2
### T2 instances
**A CPU Credit provides the performance of a full CPU core for one minute.** T2 instances provide a baseline level of CPU performance with the ability to burst above that baseline level. The baseline performance and ability to burst are governed by CPU credits.  

Other combinations of vCPUs, utilization, and time are also equal to one CPU credit; for example, one vCPU running at 50% utilization for two minutes or two vCPUs running at 25% utilization for two minutes.  

Each T2 instance starts with a healthy initial CPU credit balance and then continuously (at a millisecond-level resolution) receives a set rate of CPU credits per hour, depending on instance size.  

Initial CPU credits do not expire, but they are used first when an instance uses CPU credits. Unused earned credits from a given 5 minute interval expire 24 hours after they are earned, and any expired credits are removed from the CPU credit balance at that time, before any newly earned credits are added. Additionally, the CPU credit balance for an instance does not persist between instance stops and starts; stopping an instance causes it to lose its credit balance entirely, but when it restarts it will receive its initial credit balance again.  

If your instance uses all of its CPU credit balance, performance remains at the baseline performance level. If your instance is running low on credits, your instance’s CPU credit consumption (and therefore CPU performance) is gradually lowered to the base performance level over a 15-minute interval, so you will not experience a sharp performance drop-off when your CPU credits are depleted.
## Autoscaling groups
### Scaling Plans
1. **Maintain current instance levels at all times:** you can configure your Auto Scaling group to maintain a minimum or specified number of running instances at all times. To maintain the current instance levels, Auto Scaling performs a periodic health check on running instances within an Auto Scaling group. When Auto Scaling finds an unhealthy instance, it terminates that instance and launches a new one.
2. **Manual scaling:** you only need to specify the change in the maximum, minimum, or desired capacity of your Auto Scaling group. Auto Scaling manages the process of creating or terminating instances to maintain the updated capacity.
3. **Scale based on a schedule:** scaling by schedule means that scaling actions are performed automatically as a function of time and date.
4. **Scale based on demand (dynamic scaling):** A more advanced way to scale your resources, scaling by policy, lets you define parameters that control the Auto Scaling process. This is useful when you can define how you want to scale in response to changing conditions, but you don't know when those conditions will change. You can set up Auto Scaling to respond for you.  
Note that you should have two policies, one for scaling in (terminating instances) and one for scaling out (launching instances), for each event to monitor.

#### Dynamic Scaling
An Auto Scaling group uses a combination of alarms and policies to determine when the conditions for scaling are met. An alarm is an object that watches over a single metric (for example, the average CPU utilization of the EC2 instances in your Auto Scaling group) over a specified time period. When the value of the metric breaches the threshold that you defined, for the number of time periods that you specified, the alarm performs one or more actions (such as sending messages to Auto Scaling). A policy is a set of instructions that tells Auto Scaling how to respond to alarm messages.

##### Scaling Adjustment Types
When a scaling policy is executed, it changes the current capacity of your Auto Scaling group using the scaling adjustment specified in the policy. A scaling adjustment can't change the capacity of the group above the maximum group size or below the minimum group size.

Auto Scaling supports the following adjustment types:
- **ChangeInCapacity:** increase or decrease the current capacity of the group by the specified number of instances. A positive value increases the capacity and a negative adjustment value decreases the capacity.
- **ExactCapacity:** change the current capacity of the group to the specified number of instances. Note that you must specify a positive value with this adjustment type.
- **PercentChangeInCapacity:** increment or decrement the current capacity of the group by the specified percentage. A positive value increases the capacity and a negative value decreases the capacity.

##### Scaling Policy Types
When you create a scaling policy, you must specify its policy type. The policy type determines how the scaling action is performed. Auto Scaling supports the following policy types:
- **Simple scaling:** increase or decrease the current capacity of the group based on a single scaling adjustment.
- **Step scaling:** increase or decrease the current capacity of the group based on a set of scaling adjustments, known as step adjustments, that vary based on the size of the alarm breach.

# Security
- Create groups instead of applying permissions to multiple users
- CIS benchmark
- Configure password rotation policy
- Use cloudfront as API endpoint to hide your real API and mask the real endpoints. Also useful for DDOS protection

# Links
* [http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/t2-instances.html)
