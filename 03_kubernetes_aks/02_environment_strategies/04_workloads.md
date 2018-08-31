# Workloads

After deciding on a general strategy for laying out your evironments, you should consider what types of workloads you are going to be running. This will help you decide what node pools you may need to define.

#### Labeled Nodes
When a new node joins a Kubernetes cluster, it can optionally designate additional **Labels** that can be used for scheduling workload later. 

For instance, if you have a set of micro-services which consist of a web server and a service that makes use of GPU processors, you could have a `general` node pool, with adequate memory and cpu for running web server loads and a `gpu` node pool suitable for running the other service. This approach can also be followed for hosts which may have differing disk types, cpu to memory ratios, networking configurations, locality, etc. 

In a cloud landscape, these pools are generally defined as different scaling groups/sets. In the managed services model, sometimes these are referred to as node pools.

#### Operating System
With the rise of Windows containers, some users may find it necessary to run Windows Server nodes, and ensure that Windows-specific workload are scheduled onto those nodes. 

Windows Server-based nodes are not available in AKS at this time. You can, however, use Virtual Kubelet to schedule Windows containers on Azure Container Instances and manage them as part of your AKS cluster. For more information, see Use Virtual Kubelet with AKS.

#### Pod Resources
One thing that is generally neglected for fist-time deployments are resource "requests" and "limits". These are parameters within a Pod or PodTemplate, at the container level, that tell Kubernetes how much hardware is necessary to run the set of containers. 

If resource "requests" and "limits" are undefined, Kubernetes will allow workloads to run unbounded, and will be unable to tell the difference between a low utilization Pod (that may fit anywhere) and a high utilization Pod (that may not be able to co-habitate with other high utlization pods) at the time of scheduling.

If the container exceeds the limits, it will be automatically restarted by Kubernetes. If a single pod needs, at minimum ("requests"), almost the entire capacity of a node, its a sign that the nodes are too small:

- Consider a node pool with each member having 4 cpu and 16Gb memory. If your application "requests" 3 cpu and 12Gb of memory, the limit, even if undefined, will not be much higher with cluster services also running on the node, leaving no room to expand vertically, for more cpu/mem intense jobs. It may also leave a gap, wherein I don't have any other workloads that will fit in the remaining space, which could lead to under utilized horizontally scaled nodes. 

- Consider If the nodes were double that, 8 cpu and 32 Gb of memory, we could maintain the request, and schedule an additional 2-3 cpu job(s) onto the node, leaving each instance more room to expand vertically if necessary. If 6/8 cpus were requested, and no other workloads were scheduled, each individual instance could vertically scale to use the additional 2 cores and extra memory, of course not at the same time.

While these smaller sizes may be ok for development (orders of half cpus and mb of ram), its still worth considering how your workloads will consume the resources your cluster is managing. We will discuss this further in later sections around scaling and site reliability engineering.

#### Scheduling
When you first start populating workloads its important to make use of labels via `nodeSelector` and `affinity`, even if you only have one node pool, since you may want to expand without having existing workloads scheduled onto the newer node pool. It may not be easy to update every pod/deployment missing these scheduling conditions. We will discuss this in more detail, when we stand up a cluster and start deploying services, in the next sections.