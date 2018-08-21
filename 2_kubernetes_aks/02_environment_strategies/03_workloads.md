# Workloads

After deciding on a general strategy for laying out your evironments, you should consider what types of workloads you are going to be running. This will help you decide what node pools you may need to define.

#### Labeled Nodes
When a new node joins a Kubernetes cluster, it can optionally designate additional **Labels** that can be used for scheduling workload later. 

For instance, if you have a set of micro-services which consist of a web server and a service that makes use of GPU processors, you could have a `general` node pool, with adequate memory and cpu for running web server loads and a `gpu` node pool suitable for running the other service. This approach can also be followed for hosts which may have differing disk types, cpu to memory ratios, networking configurations, locality, etc. 

In a cloud landscape, these pools are generally defined as different scaling groups/sets. In the managed services model, sometimes these are referred to as node pools.

#### Scheduling
When you first start populating workloads its important to make use of labels via `nodeSelector` and `affinity`, even if you only have one node pool, since you may want to expand without having existing workloads scheduled onto the newer node pool. It may not be easy to update every pod/deployment missing these scheduling conditions.

We will discuss this in more detail, when we stand up a cluster and start deploying services, in the next sections.